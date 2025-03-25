FROM ubuntu:24.04

ARG RUNNER_VERSION=2.317.0
ENV DEBIAN_FRONTEND=noninteractive

LABEL Author="cafanwi"
LABEL Email="sosotech2000@gmail.com"
LABEL GitHub="https://github.com/sosotechnologies"
LABEL BaseImage="ubuntu:20.04"
LABEL RunnerVersion=${RUNNER_VERSION}

# update the base packages + add a non-sudo user
RUN apt-get update -y && apt-get upgrade -y && useradd -m docker

# install the packages and dependencies along with jq so we can parse JSON (add additional packages as necessary)
RUN apt-get install -y --no-install-recommends \
    curl nodejs wget unzip vim git jq build-essential libssl-dev libffi-dev python3 python3-venv python3-dev python3-pip

RUN cd /home/docker && mkdir actions-runner && cd actions-runner && \
    curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz && \
    echo "9e883d210df8c6028aff475475a457d380353f9d01877d51cc01a17b2a91161d  actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz" | shasum -a 256 -c && \
    tar xzf ./actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz 

# install some additional dependencies
RUN chown -R docker ~docker && /home/docker/actions-runner/bin/installdependencies.sh

# add over the start.sh script
ADD runner-scripts/start.sh start.sh

RUN chmod +x start.sh
USER docker
ENTRYPOINT ["./start.sh"]