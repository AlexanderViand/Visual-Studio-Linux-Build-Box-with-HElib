Setting up Visual Studio 2017 to work with HElib-based applications.
=====

Thanks to the introduction of ["Visual C++ for Linux"](http://aka.ms/vslinux), it has become feasible to use Visual Studio for C++ development on linux. This was first introduced as a plugin for Visual Studio 2015, but is now available natively as the "Linux development with C++" workload in Visual Studio 2017.
The system uses remote compilation on a linux machine, however this can also be a locally running virtual machine.


General Workflow 
---
With Visual C++, the code is written and developed locally in Visual Studio with full IntelliSense support. For compilation and debugging, the system automatically connects o the build machine via ssh, copies the sources files to the remote linux machine, and compiles and executes/debugs them there. The output of this process is fed back to Visual Studio, integrating to various degrees with the standard IDE functionality. Compiler errors, for example, will display just like local compilations, while linker errors are not reported well by default ([workaround](#debugging-linker-errors-and-other-build-failures)). The default settings are suitable for working with self-contained projects, however to integrate well with a library, especially during debugging, some additional setup is required.

The linux build machine
---
A docker image with all the required tools to work with Visual C++ out of the box is [available from a third party](https://hub.docker.com/r/ducatel/visual-studio-linux-build-box/) ([mirror](https://github.com/AlexanderViand/Visual-Studio-Linux-Build-Box)). The dockerfile in this repo is based on this, but includes HElib and its dependencies (via the [HElib image](https://hub.docker.com/r/alexanderviand/helib/)).
Alternatively, advanced users can set up an environment manually. Refer to the [Visual C++ documentation](http://aka.ms/vslinux) and the [HElib installation instructions](https://github.com/shaih/HElib/blob/master/INSTALL.txt) ([mirror](https://github.com/AlexanderViand/HElib/blob/master/INSTALL.txt)) for detailed information.
The following will assume that you have a valid "remote" machine available.


Setting up the Visual Studio Project & Solution
---
Firstly, make sure that the "Linux development with C++" workload is installed. This should be a simple matter of opening the Visual Studio Installer and selecting it from the list of available workloads, but details can be found on the [official Visual C++ for Linux site](http://aka.ms/vslinux).

Create a new "Linux Console Application" project (under Cross-platform->Linux, but again, refer to the official documentation for details). This should create a simple "Hello World"-style program. Test the connection to your linux build machine by trying to run/debug the program. If this is the first time you're using Visual C++ for Linux, a window will open and ask you for your connection details. See ["Managing remote connections"](#managing-remote-connections) to learn how to add, change and remove connections.

Now, we need to set up the project to work with the external library (in our case, HElib). In the project properties,
under "Linker->Input->Library Dependencies" we need to add the libraries we require, seperated by semicolons. Note that the order is important ([explanation](http://stackoverflow.com/questions/45135/why-does-the-order-in-which-libraries-are-linked-sometimes-cause-errors-in-gcc)). In our case, we need to have `fhe;ntl;gmp;m;pthread` as library dependencies.

Project specific note: The FastFaceMinimal project uses two macros for in-depth debugging, `VERBOSE` and `VERIFY` which increase the amount of debug output and the amount of checks, respectively. (As of 2017-05-12_10:47MEZ the latter is not yet implemented). These can be enabled by providing them to the compiler, by setting Project Properties "C/C++ -> All Options -> Additional Options" to `-D VERBOSE -D VERIFY %(AdditionalOptions)`.

Under "Debugging->Debugging Mode", we recommend selecting `gdb` rather than `gdbserver`. This is because the latter seems to have issues stepping into HElib code because it doesn't know where to find the sources. It might also be helpful to add `-enable-pretty-printing` under "Debugging->Additional Debugger Commands".

For debugging to properly step into the library sources, we need to make them available locally (e.g. clone HElib into a local repository). In the *Solution* Properties under "Common Properties -> Debug Source Files" add the folder that contains HElib.

In order for IntelliSense to properly recognize classes and functions from the libraries we use, we need to make the include files available to Visual Studio. If you have copied the includes to a folder `DIR` on the same level as your solution, then the include files will be in `DIR/include`. I.e. you should have `$(SolutionDir)..\DIR\include;$(IncludePath)` in the Project Properties "VC++ Directories->Include Directories".

 

Managing remote connections
---
Remote connections can be managed under "Tools->Options->Cross Platform->Connection Manager". To change which remote a project uses, go to Project Properties "General->Remote Build Machine" and select the desired machine from the dropdown. It is also possible to create additional Configurations (Project Properties -> Configuration Manager) and to have different remotes for different Configurations. This can be useful when using a local VM to debug but a remote system to evaluate and benchmark.


Debugging Linker Errors and other build failures
---
By default, the Output of the build system is very minimal. Should a build fail without appropriate reasons, this is most likely a linker error. To see these and other errors, go to "Tools->Options->Projects and Solutions->Build and Run" (or type "MSBuild" into the Options search window) and change the "MSBuild project build output verbosity" from Minimal to Normal. Keep increasing this until you find the error (Normal should be sufficient for linker errors).
