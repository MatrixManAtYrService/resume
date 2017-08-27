### Econolite, Software Engineer, 07/2011 - Present

Econolite sells traffic systems software and hardware.
The Colorado Springs office (where I work) is primarily concerned with the developing our [ATMS](https://en.wikipedia.org/wiki/Advanced_Traffic_Management_System) product, which is written primarily in C# (.Net).
The Anaheim office has traditionally handled device firmware: C++ on embedded Linux.

For my first four years with Econolite I aided the Colorado Springs office's development efforts by performing maintenance bug fixes or adding minor features.
In mid 2015 I became a team lead on an R&D project.
We were tasked with creating a product with the following goals:

 - Interpret trajectory-based vehicle detection from a variety of sensors.
 - Optimize traffic flow and provide advice to the signal controller.
 - Maintain a list of vehicle trajectories and make related analytics available to stakeholders.
 - Handle [V2V](https://www.its.dot.gov/itspac/october2012/PDF/data_availability.pdf) and [V2I](https://www.its.dot.gov/presentations/pdf/Fok_SPaT.pdf) communications.
 - Be able to run in the cloud or on the street (parameterized operation to account for latency differences).

Hardware wise, a signal controller is like a flip phone, and the platform for my project is like a smartphone.
This style of embedded linux was outside of the comfort zone for most of the Colorado Springs office.
The firmware engineers in Anaheim could make it work, but their workflow did not to take advantages of certain conveniences that come with more capable hardware (e.g. installing things instead of building them).

My familiarity with desktop Linux (and raspberry pi) allowed me to find a workflow that could more easily leverage existing open-source tools and to create the infrastructure necessary to to develop and deploy on this new platform.
Along the way I learned how to use Python, CMake, and Javascript while continuing to sharpen my knowledge of Linux and C++.

The remainder of this section describes the automation that I created in order to support this product's transition from idea to beta.

### Development Environment Setup

The traditional way to handle library dependencies in a C++ project is to leave them for to the user to build.
This makes sense for C++, but my office was primarily a Microsoft shop and had different expectations.

I integrated our dependencies into our repository (as git submodules) and then wrote a handful of Bash (and occasionally Powershell) scripts which would:
 - Build the project's dependencies (boost, sqlite, openssl) if they weren't already present.
 - Generate a build environment of the user's choice (Visual Studio, Eclipse, or GNU make).
 - In the case of Windows:
   - If our traffic simulation software was present, configure it (via environmental variables) to use the output of our build
   - scan for absolute-paths in simulation files and adjust them for the local machine

This automation encouraged the use of certain development practices (e.g. Dependency Injection).
I was later asked to present on some of these to other departments, I created and [this](https://github.com/MatrixManAtYrService/hello-cpp-linwin) project for use during that presentation.
  
#### Containerized Build

To isolate the software from environmental differences between developers I containerized the application and the build using docker.  There wasn't a tremendous amount of documentation on how best to do this with C++, so I provided notes on what I did [here](https://stackoverflow.com/a/43641999/1054322).

This required configuring a docker daemon local to the build process (Windows) to run the container on a remote linux VM.  After overcoming a few issues I left some breadcrumbs [here](https://github.com/docker/machine/issues/3239#issuecomment-274189959).

#### Continuous Integration

The rest of the office uses Microsoft Team Foundation Server extensively and it was a requirement that our builds also be launched from this hub.
We were able to use Microsoft's [build agent](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-linux) to automate the linux buils on x86.

No such agent was available for ARMv7, so I wrote some bash scripts to:
 - ask git to parse .gitignore and provide a list of relevant files
 - rsync the files to the ARM build server
 - execute the build
 - propogate success/failure information (and logs) back to TFS
 
The result was that it appeared as if we had integrated TFS with our target hardware, when in fact we had automated a remote build (launch from x86, build on ARMv7) and were just executing that automation from TFS.

##### Deployment Server

Part of deploying this product involves preparing a SD card with intersection-specific configuration.
I automated this process in two phases.  Unless otherwise specified, the tooling in use is bash.

###### Build OS Image

Launched from TFS, this phase:
 - mounts a copy of the [OS image](https://boundarydevices.com/ubuntu-xenial-mx67-boards-august-2016-kernel-4-1-15/) provided by the board manufacturer
 - enters a chroot environment within the mounted image
   - updates packages to their latest version via `apt`
   - installs chef
   - runs a chef recipe which:
     - confiures user, password, VLANs, and other non-intersection-specific details
   - stores the deployment chef recipe which gets used later
 - writes the updated pre-deployment image to disk on the deployment server
 - writes the deployment script to disk on the deployment server
 
 This allowed us to control the configuration and deployment of the image entirely in code.
 
 ###### Prepare SD card
 
 Launched from the deployment server, this phase:
  - prompts the user for intersection-specific details
  - generates a ssl certificate for this device
    - contacts the CA server to have it signed
  - mounts a copy of the pre-deployment image generated by the previous phase
  - modifies the deployment chef recipe to include intersection specific details
  - enters a chroot environment within the mounted image
    - runs a chef recipe which:
      - installs the latest version of the product
      - configures intersection-specific details (IP address, intersection geometry, etc)
      - install the ssl certificate for this device
      - configures ssh to accept only Econolite-signed ssh keys
  - Writes the image to an SD card
  - Resizes the image to fill all available space

  The ability to plug an SD card in and run a script has freed my teem up to focus on more interesting tasks, and has introduced a level of determinism that did not exist when we were doing this by hand.
