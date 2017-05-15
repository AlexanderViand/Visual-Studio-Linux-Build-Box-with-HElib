# Visual Studio Linux Build Box with HElib
This repository contains instructions (and the required [Dockerfile](./Dockerfile)) for setting up a Visual Studio development environment for [HElib](https://github.com/shaih/HElib/)-based applications. See the documentation for the general [Visual Studio workflow](./readme_visualstudio.md) and for [using the dockerfile](./readme_dockerfile.md).

This is based on a [docker image](https://hub.docker.com/r/ducatel/visual-studio-linux-build-box/) ([mirror](https://github.com/AlexanderViand/Visual-Studio-Linux-Build-Box)) from [David Ducatel](https://github.com/Ducatel). However, instead of basing off a default ubuntu 16.04 LTS image, this is based off of my [HElib docker image](https://hub.docker.com/r/alexanderviand/helib/) ([github](https://github.com/AlexanderViand/HElib)).

## How to use this:
1. Install and setup docker according to your platform. (See the official Docker documentation for this)
1. Pull the image: `docker pull alexanderviand/visual-studio-linux-build-box-with-helib` (PowerShell is recommedned for Docker)
1. To run a container, execute `docker run -d -p 12345:22 --security-opt seccomp:unconfined --name someshortname repo/name:latest` with the appropriate image name, e.g. `docker run -d -p 12345:22 --security-opt seccomp:unconfined --name buildbox alexanderviand/visual-studio-linux-build-box-with-helib:latest`
1. This will "expose" the ssh server on `http://localhost:12345` with username `root` and password `toor` (defined by the VS linux build image)
1. Use that login information as the "remote connection" for Visual Studio 2017
1. To get all the include files needed for proper IntelliSense support in visual studio, navigate to a suitable folder and run `docker cp someshortname:/usr/local/include/ .` , e.g. `docker cp buildbox:/usr/local/include/ .` This will copy the include files for GMP, NTL and HElib into the current directory where it can be used as "Additional Include Directories" for Visual Studio





