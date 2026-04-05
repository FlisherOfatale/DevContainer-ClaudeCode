FROM node:24-bookworm

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN echo "node ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/node \
    && chmod 0440 /etc/sudoers.d/node

RUN npm install -g @anthropic-ai/claude-code

USER node

# Pre-create auth paths so bind-mounts work even if host files don't exist yet
RUN mkdir -p ~/.claude && touch ~/.claude.json

WORKDIR /workspace
