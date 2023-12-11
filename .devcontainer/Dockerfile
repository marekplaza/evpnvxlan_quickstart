# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /ansible
RUN apt-get update && apt-get install -y \
    git \
    openssh-client \
    sshpass \
    vim \
    && rm -rf /var/lib/apt/lists/* 


# _IN_CONTAINER env variable will be used by make shortcuts
# to detect if command must be executed directly or as part of the docker run
ENV _IN_CONTAINER="True"

# install latest containerlab
RUN bash -c "$(curl -sL https://get.containerlab.dev)"

# The eos-downloader "ARISTA_TOKEN" env variable is set on the system Otherwise the image must be downloaded and imported manually
ENV ARISTA_TOKEN=""
RUN pip3 ansible install eos-downloader

# Install Ansible and necessary libraries

# Install Arista collections
RUN ansible-galaxy collection install arista.eos arista.cvp arista.avd


# install additional avd-quickstart requirements
COPY requirements.txt /tmp/pip-tmp/requirements.txt
WORKDIR /tmp/pip-tmp/
RUN pip install --upgrade pip \
    && pip install -r requirements.txt \
    && sudo rm -rf /tmp/pip-tmp

# add alias that can be used to add more aliases for quick lab access
RUN echo 'PROMPT="[ %{$fg[cyan]%}Apius%{$fg[green]%}LAB%{$reset_color%} 🧪 ] # "' >> ${HOME}/.zshrc 
RUN echo "alias add_aliases='. ${HOME}/add_aliases.zshrc'">> ${HOME}/.zshrc

WORKDIR /home/avd/projects
