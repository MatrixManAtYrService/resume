### Econolite, Software Engineer, 07/2011 - Present

Econolite sells traffic systems software and hardware.

#### Sustaining Development

For my first four years with Econolite I performed maintenance bug fixes and added minor features to our [ATMS](https://en.wikipedia.org/wiki/Advanced_Traffic_Management_System) product (mostly C# and SQL).

Despite no prior knowlege of these technologies, I emerged as the subject matter expert for:

 - Using MEF to compose objects from different libraries
 - Unicycles
 - XAML and the MVVM design pattern.

Achievements in code:

 - The menu system
   - It seems trivial, but as the top of the call stack for any given module, a menu item is the appropriate place for dependency injection to start.
   - This insight allowed us to reuse large pieces of UI functionality from an earlier design and isolated much of the team from the complexities of dependency management at the top level.
   - Designed an API that allowed contributors to control menu item properties in a way that did not require an explicit layout for each customer configuration (there are many).
 - A translator that accepts the European standard for signal control (stages) and converts it to the US standard (phases).
 - A visualization of how time is allocated across phases in a timing plan that updates while the user configures the plan.
 - Widespread UI modifications to support a new signal controller feature (Flashing Yellow Arrow Overlaps)
 - Brought many unit tests up to snuff so we could enable test running as part of continuous integration

#### R&D

In mid 2015 I became the technical lead on an R&D project with the following goals:

 - Interpret trajectory-based vehicle detection from a variety of sensors.
 - Optimize traffic flow and provide advice to the signal controller.
 - Maintain a list of vehicle trajectories and make related analytics available to stakeholders.
 - Handle [V2V](https://www.its.dot.gov/itspac/october2012/PDF/data_availability.pdf) and [V2I](https://www.its.dot.gov/presentations/pdf/Fok_SPaT.pdf) communications.
 - Be able to run in the cloud or on the street (parameterized operation to account for latency differences).

Our first deployment will be on an signal controller expansion board that is similar in capability to a raspberry pi (embedded linux).
The traffic simulator (VISSIM) we use is Windows-only (as is the majority of the Colorado Springs office).
This required that we maintain a cross-platform workflow from the start.
Mostly this was achieved with CMake, which required [significant enocouragement](https://github.com/MatrixManAtYrService/hello-cpp-linwin/blob/master/CMakeLists.txt) to generate a Visual Studio solution that was organized appropriately.

The cross-compiling (ARM) that was performed when we started required us to maintain the build process for all third party code.
A desire to remove barriers to open source adoption lead us to switch to a native compilation workflow.
The time spent not-coding has far outweighed the extra time spent compiling.

Achievements in code:
 - VISSIM Geometry Extractor
   - Learned Python to take advantage of more capable graph theory library (networkx)
   - Examines VISSIM's file format and identifies:
     - topological features (nodes, edges -> lanes, maneuvers)
     - geometrical features (coordinates -> stop-line-setback, detection setback)
     - control features (smiulation parameters -> yield-to relationships, signal configurations)
   - Generates an intersection geometry object for use by our project.
   - Uses Boost::Python to call C++ code and validate that the resultant geometry is valid.
 - VISSIM Driver
   - Allows us to control vehicle behavior in c++ code
   - Uses Boost::Python to call into the extractor and resuse coordinate transformation functionality
 - IO library
   - HTTP REST API
   - Optional Websockets API with failover to HTTP
     - Ignores responses (supports PUT and POST only) because waiting for REST responses was too slow
   - Source/sink semantics hides transport-layer details from library developers
   - Use of functional programming concepts allows flexible configuration
 - Sensor-type-agnostic operation with plugin-based extensibility
 - [v2i-unofficial](https://github.com/MatrixManAtYrService/v2i-unofficial), a fork of the FHWA's open source message hub.
   - modified it to run on ARM
   - automated the build process
   - our project links to their binaries for message decoding to ensure compatibility with standards

This project is a step towards replacing an older product which had become too complicated to maintain.
I was chosen for this because of prior Linux and C++ knowledge, and also because I had demonstrated a grasp of object oriented design principles and a commitment to writing testable code.
I was instructed to provide an architecture that would be resistent to the kind of incremental degredation that the older product faced.
Based on the way the design has encouraged the junior developers on my team to do things the right way (even in my absence), I am optimistic about the longevity of our work.

Management asked me to give a presentation to the rest of the company in hopes that some of the design principles we used would see wider adoption.
[This repo](https://github.com/MatrixManAtYrService/hello-cpp-linwin/) was created for use during that presentation.
It resembles our project's structure, except with the proprietary functionailty replaced with "Hello World".

