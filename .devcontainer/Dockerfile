FROM node:24-bookworm

RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g pnpm

RUN echo "node ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/node \
    && chmod 0440 /etc/sudoers.d/node

# RUN npm install -g @anthropic-ai/claude-code

# Install Trivy (vulnerability + secret + misconfig scanner)
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
    | sh -s -- -b /usr/local/bin

# Install Gitleaks (git history secret scanner)
RUN curl -sSfL https://raw.githubusercontent.com/gitleaks/gitleaks/master/scripts/install.sh \
    | sh -s -- -b /usr/local/bin

# Install Semgrep (SAST scanner — version matches CI: semgrep/semgrep:1.157.0)
RUN apt-get update \
    && apt-get install -y --no-install-recommends python3-pip \
    && pip3 install --break-system-packages semgrep==1.157.0 \
    && apt-get purge -y python3-pip \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/*

USER node

# Pre-create auth paths so bind-mounts work even if host files don't exist yet
RUN mkdir -p ~/.claude && touch ~/.claude.json

WORKDIR /workspace
