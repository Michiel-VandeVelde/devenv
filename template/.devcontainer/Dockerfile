FROM node:22
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install basic development tools
# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends less man-db sudo curl nano \
    && rm -rf /var/lib/apt/lists/*

# Ensure default `node` user has access to `sudo`
ARG USERNAME=node
RUN echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

USER node
RUN curl -fsSL https://gh.io/copilot-install | bash
RUN curl -fsSL https://claude.ai/install.sh | bash
RUN git config --global alias.gitm 'commit -am' \
    && git config --global push.autoSetupRemote true \
    && git config --global core.editor nano \
    && echo "alias gitm='git commit -am'" >> /home/node/.bashrc \
    && echo "alias claude-yolo='claude --dangerously-skip-permissions'" >> /home/node/.bashrc \
    && echo "alias copilot-yolo='copilot --yolo'" >> /home/node/.bashrc
