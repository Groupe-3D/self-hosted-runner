FROM ubuntu:24.04

ARG RUNNER_VERSION="2.328.0"
ARG DEBIAN_FRONTEND=noninteractive

# Install base dependencies
RUN apt-get update -y \
    && apt-get upgrade -y \
    && apt-get install -y \
                build-essential \
                ca-certificates \
                curl \
                iptables \
                jq \
                libffi-dev \
                libssl-dev \
                libyaml-dev \
                python3 \
                python3-dev \
                python3-pip \
                python3-venv \
                ssh \
                sudo

# Install Docker
RUN install -m 0755 -d /etc/apt/keyrings \
    && curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc \
    && chmod a+r /etc/apt/keyrings/docker.asc \
    # Add the repository to Apt sources:
    && echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
      tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt-get update \
    && apt install -y --no-install-recommends \
                   containerd.io \
                   docker-buildx-plugin \
                   docker-ce-cli \
                   docker-compose-plugin \
    && rm -rf /var/lib/apt/lists/*

# Add a user named runner and adding it to sudoers and Docker group
RUN useradd -m runner \
    && groupadd docker \
    && usermod -aG sudo root \
    && usermod -aG sudo runner \
    && usermod -aG docker runner \
    && echo "%sudo   ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers \
    && echo "Defaults env_keep += \"DEBIAN_FRONTEND\"" >> /etc/sudoers

# Set up the actions runner
RUN cd /home/runner \
    && mkdir actions-runner \
    && cd actions-runner \
    && curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# Change ownership to runner user and install dependencies
RUN chown -R runner /home/runner \
    && /home/runner/actions-runner/bin/installdependencies.sh
RUN mkdir /opt/hostedtoolcache \
    && chown -R runner /opt/hostedtoolcache

# Copy the start script and make it executable
COPY start.sh /start.sh
RUN chmod +x /start.sh

# Switch to runner user
USER runner

# Define the entrypoint
ENTRYPOINT ["/start.sh"]
