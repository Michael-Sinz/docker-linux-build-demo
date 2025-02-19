# Normally, we would publish the base image into some public
# container registry and select a specific version that matched
# the source tree state (such that we have direct control over
# the tooling for the builds of our source tree)
FROM dev-base as dev-personal

# The UID and GID should be arguments at build time
# albeit it is a bit scary to do this as the build time user
# could set a strange user but then it will just make things not
# work very well in the system (or, even more likely, fail to
# build the docker image)
ARG _GID=9999
ARG _GROUP=unknown

ARG _UID=9999
ARG _USER=unknown

# First add the group since it must be there before the user
RUN groupadd -g ${_GID} ${_GROUP}

# Now define a user with the given UID and GID and name.
# We can also include the user in some other groups, such as the
# AzureLinux defined "users" group.  (Do as needed)
# Finally, the user will be defined to have a home directory as /home/build
# as we want it consistent across all use cases
RUN useradd -u ${_UID} -g ${_GID} -G users -d /home/build -m ${_USER}

# As an example here, we add this user into the sudoers list.
# Since the user has no password, we need to allow the user
# to sudo without one.  The hope is that no sudo is ever needed,
# in which case is could be just ignored (and sudo not even installed)
RUN echo "${_USER} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${_USER}

# Finally, we set the docker image to default run as the user we
# just defined.  This really has to happen here at the end as the
# above steps had to run as root within the image build context.
USER ${_USER}:${_GROUP}
