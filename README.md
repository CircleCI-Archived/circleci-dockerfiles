# circleci-dockerfiles
Dockerfiles for [CircleCI's convenience Docker images](https://hub.docker.com/r/circleci), built via https://github.com/circleci/circleci-images

This repository's `master` branch tracks images pushed to the official CircleCI Docker Hub organization:

- https://hub.docker.com/r/circleci

Its `staging` branch tracks images pushed to [CircleCI's staging environment](https://hub.docker.com/r/ccistaging); any other branches track images pushed to [CircleCI's test environment](https://hub.docker.com/r/ccitest).

## Usage
The docker build command builds an image from a Dockerfile and a context. The build’s context is the set of files at a specified location PATH or URL. The PATH is a directory on your local filesystem. The URL is a Git repository location.

A context is processed recursively. So, a PATH includes any subdirectories and the URL includes the repository and its submodules. This example shows a build command that uses the current directory as context:

$ docker build .
Sending build context to Docker daemon  6.51 MB
...
The build is run by the Docker daemon, not by the CLI. The first thing a build process does is send the entire context (recursively) to the daemon. In most cases, it’s best to start with an empty directory as context and keep your Dockerfile in that directory. Add only the files needed for building the Dockerfile.

Warning: Do not use your root directory, /, as the PATH as it causes the build to transfer the entire contents of your hard drive to the Docker daemon.

To use a file in the build context, the Dockerfile refers to the file specified in an instruction, for example, a COPY instruction. To increase the build’s performance, exclude files and directories by adding a .dockerignore file to the context directory. For information about how to create a .dockerignore file see the documentation on this page.

Traditionally, the Dockerfile is called Dockerfile and located in the root of the context. You use the -f flag with docker build to point to a Dockerfile anywhere in your file system.

$ docker build -f /path/to/a/Dockerfile .
You can specify a repository and tag at which to save the new image if the build succeeds:

$ docker build -t shykes/myapp .
To tag the image into multiple repositories after the build, add multiple -t parameters when you run the build command:

$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
Before the Docker daemon runs the instructions in the Dockerfile, it performs a preliminary validation of the Dockerfile and returns an error if the syntax is incorrect:

$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
The Docker daemon runs the instructions in the Dockerfile one-by-one, committing the result of each instruction to a new image if necessary, before finally outputting the ID of your new image. The Docker daemon will automatically clean up the context you sent.

Note that each instruction is run independently, and causes a new image to be created - so RUN cd /tmp will not have any effect on the next instructions.

Whenever possible, Docker will re-use the intermediate images (cache), to accelerate the docker build process significantly. This is indicated by the Using cache message in the console output. (For more information, see the Build cache section in the Dockerfile best practices guide):

$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
Build cache is only used from images that have a local parent chain. This means that these images were created by previous builds or the whole chain of images was loaded with docker load. If you wish to use build cache of a specific image you can specify it with --cache-from option. Images specified with --cache-from do not need to have a parent chain and may be pulled from other registries.

When you’re done with your build, you’re ready to look into Pushing a repository to its registry.

## Stay informed about CircleCI image changes/announcements
As part of regular maintenance, changes are occassionally made to various images, from updating images' contents, to changing how image variants are tagged. With the exception of bugfixes or security patches, these changes will always be announced in advance. Changes are posted in the Announcements section of CircleCI Discuss; relevant posts will always have a `convenience-images` tag:

- https://discuss.circleci.com/c/announcements
- https://discuss.circleci.com/tags/convenience-images

By creating a Discuss account, you can subscribe to these posts, in order to receive notifications via email:

https://discuss.circleci.com

### This repository is automated and is not actively managed or maintained on its own
Information about CircleCI's convenience images is available in the following locations:

- https://circleci.com/docs/2.0/circleci-images
- https://github.com/circleci/circleci-images

## Submitting issues and pull requests
Please file any CircleCI convenience image issues, feature requests, or pull requests via the **circleci-images** repository:

- https://github.com/circleci/circleci-images/issues
- https://github.com/circleci/circleci-images/pulls
