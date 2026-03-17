FROM fedora:latest

# Layer 1 - System packages
RUN dnf install -y \
    git zsh curl wget \
    dnf-plugins-core \
    python3 python3-devel \
    nodejs npm \
    golang \
    php php-cli php-common \
    gcc gcc-c++ make cmake \
    findutils procps-ng \
    sudo \
    jq ripgrep fd-find bat fzf htop tmux neovim \
    && dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo \
    && dnf install -y docker-ce-cli \
    && dnf clean all

# System-wide PATH for dev tools (persists when user mounts their own dotfiles)
RUN cat >> /etc/zshenv << 'EOF'

# Dev tools PATH
export PATH="$HOME/.local/bin:$HOME/.cargo/bin:$PATH"
export GOPATH="$HOME/go"
export PATH="$GOPATH/bin:$PATH"
EOF

# Layer 2 - UV (Python package manager)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh

# Layer 3 - Rust via rustup
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    && . /root/.cargo/env && rustup default stable

# Layer 4 - Non-root user (UID/GID 1000 for host permission compatibility)
RUN groupadd -g 1000 dev \
    && useradd -m -s /bin/zsh -u 1000 -g 1000 -G wheel dev \
    && echo "dev ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/dev

# Install starship binary (as root, to /usr/local/bin)
RUN curl -sS https://starship.rs/install.sh | sh -s -- -y

# Move root-installed tools to dev user
RUN mv /root/.cargo /home/dev/.cargo \
    && mv /root/.rustup /home/dev/.rustup \
    && mv /root/.local /home/dev/.local \
    && chown -R dev:dev /home/dev/.cargo /home/dev/.rustup /home/dev/.local

# Layer 5 - Oh My Zsh (as dev user)
USER dev
WORKDIR /home/dev

RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Create minimal fallback .zshrc (will be overwritten by mounted config)
RUN cat > /home/dev/.zshrc << 'EOF'
# Fallback zshrc - typically overwritten by mounted user config
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="robbyrussell"
plugins=(git)
source $ZSH/oh-my-zsh.sh
eval "$(starship init zsh)"
EOF

CMD ["/bin/zsh"]
