# Use an official slim Python image as the base.
# Provides python3, pip, and a small Debian userspace.
FROM python:3.12-slim

# ------------------------------------------------------------
# INSTALL BASE TOOLING
# ------------------------------------------------------------
# Install minimum tooling needed for development:
#  - git      : version control
#  - curl     : simple HTTP tool for testing endpoints
#  - vim      : quick in-container editing if needed
#  - docker   : docker environment
# Clean apt caches afterward to keep the image small.
RUN apt-get update && apt-get install -y \
    git \
    curl \
    vim \
    #docker.io \
    #docker-compose-plugin \
    && rm -rf /var/lib/apt/lists/*

# ------------------------------------------------------------
# SET WORKING DIRECTORY
# ------------------------------------------------------------
# This is where the repository will be mounted inside the container.
WORKDIR /workspace

# ------------------------------------------------------------
# DEFAULT COMMAND
# ------------------------------------------------------------
# Container stays alive so VS Code can attach to it.
# We don’t run Python or other services here — this container
# is only the development shell, not runtime stack.
CMD ["sleep", "infinity"]