# Update the VARIANT arg in docker-compose.yml to pick an Elixir version: 1.9, 1.10, 1.10.4
ARG VARIANT="1.16.0"
FROM elixir:${VARIANT}

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Update the “remoteUser” property in
# devcontainer.json to use it. More info: https://aka.ms/vscode-remote/containers/non-root-user.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Options for common package install script
ARG INSTALL_ZSH="true"
ARG UPGRADE_PACKAGES="true"
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/v0.233.0/script-library/common-debian.sh"
ARG COMMON_SCRIPT_SHA="ab4609258940b032d02ed08d4c776e9e66e1a2a21dd2f3313a8fc1c4fee607d0"

# Optional Settings for Phoenix
ARG PHOENIX_VERSION="1.7.10"

ARG NODE_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/node-debian.sh"
ARG NODE_SCRIPT_SHA="dev-mode"
ARG NODE_VERSION="none"
ENV NVM_DIR=/usr/local/share/nvm
ENV NVM_SYMLINK_CURRENT=true
ENV PATH=${NVM_DIR}/current/bin:${PATH}

# [Choice] Node.js version: none, lts/*, 16, 14, 12, 10
ARG NODE_VERSION="none"

# Install needed packages and setup non-root user. Use a separate RUN statement to add your own dependencies.
RUN apt-get update \
    # Install common utilities 
    && apt-get -y install --no-install-recommends curl apt-utils dialog 2>&1 \
    # Install bash tree
    && apt-get -y install --no-install-recommends tree \

    # Install git iproute2, procps, lsb-release (useful for CLI installs)
    && apt-get -y install --no-install-recommends git iproute2 procps iproute2 lsb-release ca-certificates \

    && curl -sSL ${COMMON_SCRIPT_SOURCE} -o /tmp/common-setup.sh \
    && ([ "${COMMON_SCRIPT_SHA}" = "dev-mode" ] || (echo "${COMMON_SCRIPT_SHA} */tmp/common-setup.sh" | sha256sum -c -)) \
    && /bin/bash /tmp/common-setup.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${UPGRADE_PACKAGES}" \
    #
    # [Optional] Install Node.js for use with web applications
    && if [ "$NODE_VERSION" != "none" ]; then \
    curl -sSL ${NODE_SCRIPT_SOURCE} -o /tmp/node-setup.sh \
    && ([ "${NODE_SCRIPT_SHA}" = "dev-mode" ] || (echo "${COMMON_SCRIPT_SHA} */tmp/node-setup.sh" | sha256sum -c -)) \
    && /bin/bash /tmp/node-setup.sh "${NVM_DIR}" "${NODE_VERSION}" "${USERNAME}"; \
    fi \
    #
    # Install dependencies
    && apt-get install -y build-essential \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /tmp/common-setup.sh /tmp/node-setup.sh

RUN su ${USERNAME} -c "mix local.hex --force \
    && mix local.rebar --force \
    && mix archive.install --force hex phx_new ${PHOENIX_VERSION}"

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update \
#     && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment this line to install additional package.
# RUN  mix ...