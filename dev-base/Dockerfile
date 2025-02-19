# I am doing this with AzureLinux as a base but
# you could do similar with Ubuntu or Debian or RedHat or...
FROM mcr.microsoft.com/azurelinux/base/core:3.0 AS dev-base

# First build up all of the tools/packages we want as the toolset
# Note that this is obviously not the complete list you may need
# but it is an example

RUN tdnf install --assumeyes \
    autoconf \
    awk \
    bc \
    binutils \
    bison \
    elfutils-devel \
    fakeroot \
    flex \
    gcc \
    git \
    glibc-devel \
    gpgme \
    gpgme-devel \
    gzip \
    kernel-headers \
    libcxxabi-devel \
    make \
    ncurses-devel \
    openssl \
    openssl-devel \
    openssl-libs \
    rpm-build \
    rpm-devel \
    rsync \
    rust \
    sed \
    sudo \
    tar \
    time \
    vi \
    && echo 'Done'