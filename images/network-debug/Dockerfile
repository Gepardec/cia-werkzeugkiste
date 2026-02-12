# Base image with essential debugging tools. Also provided by this repo at images/base.
FROM ghcr.io/gepardec/debug-images/base-debug:latest

# Switch to the root user to install new packages and binaries.
USER root

# Install additional, more advanced networking tools.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    iputils-ping \
    traceroute \
    nmap \
    netcat-traditional \
    tcpdump \
    mtr-tiny && \
    # Clean up apt cache
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Switch back to the non-root user for security.
USER debuguser

# Set the working directory.
WORKDIR /home/debuguser

# Default command to start a shell.
CMD [ "/bin/bash" ]