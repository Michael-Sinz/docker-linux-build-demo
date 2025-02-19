# Example of using docker as a dev/build environment

Docker is a great way to build/bundle a set of files as if they are
a single versioned tool - much like static linking but across a rich
and diverse set of potential tools and technologies.

What we are doing here is to build the build environment as a docker
image.  This allows significant benefits to packaging the build tools
into a consistent dependency managed group that is then universally
available to all without impacting their normal ambient environment
on their personal environment.

The only requirements is that the user has rights build / run docker
images.

Why this way?  It simplifies the pre-build requirements to not impact
the user's machine.  Thus, I can still build projects that require
C++11 in one but C++13 in another without having to figure out how to
switch my environment correctly for each one.

In the past, things like LD_LIBRARY_PATH and LD_PRELOAD plus changing
the PATH and other environment variables were the tricks that were used.
These are a great start but can still lead to ambient configuration leaking
into the build environment and thus not producing consistent result
across all users.

The best past processes would include chroot jails to help isolate things
but that brought with it a lot of other overheads and complexities.

Docker (or containerd) came out of the chroot jail concept and some
automation/generalization of the processes to build the chroot environments.
This has grown to a very rich toolset over the years where very few
would think of manually building chroot jails unless they already had
them from before.

The trick here is that we are not building a docker image as the goal but
building a docker image as the tool with which we build our main project.

## Requirements on the environment

The basic requirements is that the user have git to get the source code
and docker to run the tools.  Docker needs rights to pull some base images
but that should be about the limit of what is needed.

The rest should be contained in the docker images and the specifications
to build those images.

The benefit is that the big part of the build - the building of the image
with the set of tools and libraries needed - can be done in one place.
This makes the bootstrap and setup process as simple as running a single
command (or so)

## This Example

For this example, we did something really simple.  We have a minimal
dev-base container that is made that has all of the tools (gcc/vi/etc)
in it such that it can compile the linux kernel.  Note that we likely want
to have even more or different items (or versions) installed but this
is for the demo.

Also, there is nothing that says we just use one of these - we could
have additional images, maybe based on this or completely fresh - that
incorporate the items we have built (or just mount them into the image,
which is something I have done in some other projects)

### Prerequisites

The demo depends on:

* Linux (maybe even other OSes that can run Docker and bash and git - untested)
* bash
* docker.io
* git

Also, the user must be added to the docker group such that the user
can run docker commands.

## Directories:

### src

This is where all of the source lives - it is never modified in the
containers and is assured of that by being mounted read-only.

We do not need to do this but that is a best practices type of thing.

In order to keep this consistent across all users, this will be mounted
under `/src` inside the container (it also simplifies scropting since it
would know where it is, no matter the user name)

For the demo, I have checked out the linux v6.13 source from git such that
we can build it.  Note that it is absolutely clean checkout with no extra
files or local modifications.

### out

This is the output directory - all build artifacts will live here.
This is the external persistent directory that will keep the results

Again, as part of the consistency and simplification concept, the directory
is mounted into the container under `/out`  Not technically needed but
it makes life so much easier.

### home

This directory is read-only mounted into the dev container environment
This is useful for setting up some common initialization that may be
wanted beyond what would have been default in the dev container.

Things like .bashrc/etc.

This does not need to be here as we can just let the container's empty
home directory be there but this could be a useful hook for some additional 
behaviors/features.

It is mounted read-only as it is part of the source tree and when doing
builds, the source tree is always read-only.  Only output artifacts are
read-write.

What I have in it here is the build-kernel script (trivial) and the linux
kernel config we want (for this demo).

## Example

A very simple example that builds the kernel with the config given
and has the results in the out directory (specifically `out/linux/`)

The script named `build` first builds the dev-base container image,
then the dev-personal personalized image and then uses that personalized
image to actually build the kernel.

Due to docker caching, the first step is rather quick as there is nothing
to do after the first time it was built.  In a real world example, this
base image would be more globally built and stored in a container registery
for all to use and thus not something anyone would do under normal conditions.
However, if someone needs to update dependencies, this is easily done
as on any local machine (which is what I am showing) - this is on a Windows VM
in Azure running WSL2 Ubuntu Linux with docker.io and docker-buildx installed
in Linux (not Docker Desktop on Windows).

Running this again, once the whole build has completed, with all of the
steps happening again, takes a total of 6.18 seconds on my VM, of which 4.77
seconds is the linux kernel checking that it has all of its dependencies
compiled and up-to-date.

If I just do the last step (assuming the container images are already
built) the time it takes is 5.47 seconds, just under 1 second of overhead.
(Unfortunately, the overhead for starting and stopping and cleaning up
a docker container is non-zero but it is reasonable relative to the
actual work.)

If an actual build is needed, it takes around 6 minutes 50 seconds to
complete in my WSL Linux Docker environment.

All of this would likely be faster if we had a real Linux machine and not
a WSL Linux.

### Example output after a completed build (see overheads)

Running the `build` script:
```log
+++ dirname ./build
++ cd .
++ pwd
+ _DIR=/home/msinz/docker-linux-build-demo
+ /home/msinz/docker-linux-build-demo/src/clone-linux

real	0m0.003s
user	0m0.002s
sys	0m0.000s
+ /home/msinz/docker-linux-build-demo/dev-base/build
++ dirname /home/msinz/docker-linux-build-demo/dev-base/build
+ exec docker build -t dev-base /home/msinz/docker-linux-build-demo/dev-base
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 833B done
#1 DONE 0.0s

#2 [internal] load metadata for mcr.microsoft.com/azurelinux/base/core:3.0
#2 DONE 0.0s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/2] FROM mcr.microsoft.com/azurelinux/base/core:3.0
#4 DONE 0.0s

#5 [2/2] RUN tdnf install --assumeyes     autoconf     awk     bc     binutils     bison     elfutils-devel     fakeroot     flex     gcc     git     glibc-devel     gpgme     gpgme-devel     gzip     kernel-headers     libcxxabi-devel     make     ncurses-devel     openssl     openssl-devel     openssl-libs     rpm-build     rpm-devel     rsync     rust     sed     sudo     tar     time     vi     && echo 'Done'
#5 CACHED

#6 exporting to image
#6 exporting layers done
#6 writing image sha256:6d6407bacd0fa1e8351f4c4c56b4dce9b8bdc568cdb7794d25d026027b2f9d94 done
#6 naming to docker.io/library/dev-base done
#6 DONE 0.0s

real	0m0.619s
user	0m0.102s
sys	0m0.112s
+ /home/msinz/docker-linux-build-demo/dev-personal/build
++ id -u
++ id -u -n
++ id -g
++ id -g -n
++ dirname /home/msinz/docker-linux-build-demo/dev-personal/build
+ exec docker build --build-arg _UID=1000 --build-arg _USER=msinz --build-arg _GID=1000 --build-arg _GROUP=msinz -t dev-personal /home/msinz/docker-linux-build-demo/dev-personal
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 1.65kB done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/dev-base:latest
#2 DONE 0.0s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/4] FROM docker.io/library/dev-base:latest
#4 DONE 0.0s

#5 [2/4] RUN groupadd -g 1000 msinz
#5 CACHED

#6 [3/4] RUN useradd -u 1000 -g 1000 -G users -d /home/build -m msinz
#6 CACHED

#7 [4/4] RUN echo "msinz ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/msinz
#7 CACHED

#8 exporting to image
#8 exporting layers done
#8 writing image sha256:fa719e66e8fd6354635a528722915c22914495e104230ab5ce1ee3ee29a72139 done
#8 naming to docker.io/library/dev-personal 0.0s done
#8 DONE 0.0s

real	0m0.568s
user	0m0.122s
sys	0m0.062s
+ /home/msinz/docker-linux-build-demo/dev /home/build/build-kernel
+ OUT_DIR=/out/linux
+ mkdir -p /out/linux
++ dirname /home/build/build-kernel
+ cp /home/build/kernel.config /out/linux/.config
+ cd /src/linux
++ nproc
+ /usr/bin/time -v make abs_output=/out/linux -j 16
make[1]: Entering directory '/out/linux'
  SYNC    include/config/auto.conf
  GEN     Makefile
  GEN     Makefile
mkdir -p /out/linux/tools/objtool && make O=/out/linux subdir=tools/objtool --no-print-directory -C objtool 
  INSTALL libsubcmd_headers
  CALL    /src/linux/scripts/checksyscalls.sh
Kernel: arch/x86/boot/bzImage is ready  (#2)
make[1]: Leaving directory '/out/linux'
	Command being timed: "make abs_output=/out/linux -j 16"
	User time (seconds): 12.21
	System time (seconds): 3.49
	Percent of CPU this job got: 314%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:04.99
	Average shared text size (kbytes): 0
	Average unshared data size (kbytes): 0
	Average stack size (kbytes): 0
	Average total size (kbytes): 0
	Maximum resident set size (kbytes): 35760
	Average resident set size (kbytes): 0
	Major (requiring I/O) page faults: 9
	Minor (reclaiming a frame) page faults: 727707
	Voluntary context switches: 5830
	Involuntary context switches: 2566
	Swaps: 0
	File system inputs: 0
	File system outputs: 3560
	Socket messages sent: 0
	Socket messages received: 0
	Signals delivered: 0
	Page size (bytes): 4096
	Exit status: 0

real	0m5.860s
user	0m0.025s
sys	0m0.026s
```