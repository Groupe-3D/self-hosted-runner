FROM ubuntu:24.04

ARG RUNNER_VERSION="2.328.0"
ARG DEBIAN_FRONTEND=noninteractive

# Update and upgrade the system
RUN apt update -y && apt upgrade -y

# Add a user named runner
RUN useradd -m runner

# Install necessary packages
RUN apt install -y --no-install-recommends \
    curl build-essential libssl-dev libffi-dev python3 python3-venv python3-dev python3-pip jq libyaml-0 libgmp
RUN apt-get -yqq install ssh
# Set up the actions runner
RUN cd /home/runner && mkdir actions-runner && cd actions-runner \
    && curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz \
    && tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# Change ownership to runner user and install dependencies
RUN chown -R runner /home/runner && /home/runner/actions-runner/bin/installdependencies.sh
RUN mkdir /opt/hostedtoolcache && chown -R runner /opt/hostedtoolcache

# Copy the start script and make it executable
COPY start.sh /start.sh
RUN chmod +x /start.sh

# Switch to runner user
USER runner

# Define the entrypoint
ENTRYPOINT ["/start.sh"]
