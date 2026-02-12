FROM debian:12-slim

# Set environment variables to ensure non-interactive package installation
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    git \
    procps \
    net-tools \
    iproute2 \
    dnsutils \
    unzip \
    zip \
    openssl \
    vim-tiny && \
    # Clean up the apt cache to reduce image size
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a dedicated, non-root user and group to run applications.
# Using a fixed UID/GID helps avoid permission issues on platforms like OpenShift.
RUN groupadd --gid 1001 debuguser && \
    useradd --uid 1001 --gid 1001 --shell /bin/bash --create-home debuguser

# Switch to the non-root user. Any subsequent commands will run as 'debuguser'
USER debuguser

# Set the home directory as the default working directory.
WORKDIR /home/debuguser

# Set a default command to start a shell
CMD [ "/bin/bash" ]