What is this?
===
This is a dockerfile intended to create a repeatable environment for code using HElib.
A pre-built image is available on docker hub [here](https://hub.docker.com/r/alexanderviand/visual-studio-linux-build-box-with-helib/). 
If you choose not to use that image (e.g. if you want to modify this image), then the following will cover how to manually build an image from this dockerfile.


Background
===
The docker image is inspired by the [visual-studio-linux-build-box](https://hub.docker.com/r/ducatel/visual-studio-linux-build-box/) image ([mirror](https://github.com/AlexanderViand/Visual-Studio-Linux-Build-Box)) ( used to allow "remote" compiling & development via Visual Studio's C++ for linux workload ).
It is based on Ubuntu 16.04 LTS and includes HElib and its depdencies via the [HElib image](https://hub.docker.com/r/alexanderviand/helib/) ([github](https://github.com/AlexanderViand/HElib)). If Visual Studio integration is not desired, use that image directly. However, additional setup is then required to get your application to execute (in the VS image, this is done via ssh'ing into the container).
A mirror of the visual-studio-linux-build-box can be found [here](https://github.com/AlexanderViand/Visual-Studio-Linux-Build-Box).

The dockerfile first installs some required packages, then downloads and compiles GMP and NTL (threadsafe) before downloading and compiling HElib (indirectly, via HElib image). Finally, it sets up the required ssh and gdb connections. 
It is configured to use the AlexanderViand/HElib repository, rather than the official HElib repository (shaih/HElib) because the official version does not provide a "make install" directive"

How to build this dockerfile
===

1. Install and setup docker according to your platform. (See the official Docker documentation for this)
1. Navigate to the directory with the dockerfile and run `docker build . -t somerepo/somename:latest` (the part after the -t needs to be formated as repo/name:version in all lower case. E.g. `alexanderviand/visual-studio-linux-build-box-with-helib:latest`)
1. Wait for the build to complete. This can take some time.
1. Note that because of the caching system built into docker, changes to the HElib repo will not be reflected by an image build if a previous (cached) image is available. Either modify the `git clone` `RUN` line (e.g. add a comment at the end) or remove the cached files directly (see docker documentation).
1. To run a container from this system, execute `docker run -d -p 12345:22 --security-opt seccomp:unconfined --name someshortname somerepo/somename:latest` with the appropriate image name, e.g. `docker run -d -p 12345:22 --security-opt seccomp:unconfined --name buildbox alexanderviand/visual-studio-linux-build-box-with-helib:latest`
1. This will "expose" the ssh server on `http://localhost:12345` with username `root` and password `toor` (defined by the VS linux build image)
1. If using Visual Studio C++ for linux, then use that login information as the "remote connection"
1. To get all the include files needed for proper IntelliSense support in visual studio, run `docker cp someshortname:/usr/local/include/ .` , e.g. `docker cp buildbox:/usr/local/include/ .` This will copy the include files for GMP, NTL and HElic into the current directory where it can be used as "Additional Include Directories" for Visual Studio
