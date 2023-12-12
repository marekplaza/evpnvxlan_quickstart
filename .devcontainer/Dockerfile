# use AVD all-in-one container to get all AVD requirements
#FROM avdteam/avd-all-in-one:avd3.3.3_cvp3.3.1_debian
FROM avdteam/avd-all-in-one:latest

# _IN_CONTAINER env variable will be used by make shortcuts
# to detect if command must be executed directly or as part of the docker run
ENV _IN_CONTAINER="True"

# install latest containerlab
RUN bash -c "$(curl -sL https://get.containerlab.dev)"

# The eos-downloader can be used to import ceos-lab images from arista.com automatically
# That only works for Arista employees currently if "ARISTA_TOKEN" env variable is set on the system
# Otherwise the image must be downloaded and imported manually
ENV ARISTA_TOKEN=""
RUN pip3 install eos-downloader

# install additional avd-quickstart requirements
COPY requirements.txt /tmp/pip-tmp/requirements.txt
WORKDIR /tmp/pip-tmp/
RUN pip install --upgrade pip \
    && pip install -r requirements.txt \
    && sudo rm -rf /tmp/pip-tmp

# add alias that can be used to add more aliases for quick lab access
RUN echo 'PROMPT="[ %{$fg[cyan]%}Apius%{$fg[green]%}LAB%{$reset_color%} 🧪 ] # "' >> ${HOME}/.zshrc 
RUN echo "alias add_aliases='. ${HOME}/add_aliases.zshrc'">> ${HOME}/.zshrc
#CMD ["make"]
#CMD ["bash", "--init-file", "<(echo 'PS1=\"\u@\h:\w :rocket: \"')"]

WORKDIR /home/avd/projects
