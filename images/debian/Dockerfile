FROM kasmweb/core-debian-bookworm:1.18.0

USER root

RUN apt update && \
    apt install -y --no-install-recommends \
        build-essential git curl wget python3 python3-pip nodejs npm golang-go \
        fonts-noto fonts-dejavu fonts-noto-cjk \
        fcitx5 fcitx5-unikey fcitx5-config-qt fcitx5-frontend-gtk2 fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 \
        software-properties-common apt-transport-https \
        wget gnupg ca-certificates zsh fzf thefuck && \
    # VS Code repo
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /usr/share/keyrings/microsoft-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list && \
    # WezTerm repo
    curl -fsSL https://apt.fury.io/wez/gpg.key | gpg --dearmor -o /usr/share/keyrings/wezterm-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/wezterm-archive-keyring.gpg] https://apt.fury.io/wez/ * *" > /etc/apt/sources.list.d/wezterm.list && \
    apt update && \
    apt install -y code wezterm && \
    apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

USER 1000

# Install Oh My Zsh + plugins + theme Powerlevel10k + bypass wizard
RUN touch /home/kasm-user/.zshrc && \
    # Install Oh My Zsh (user kasm-user)
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    # Install plugins
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-autocomplete && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-completions && \
    git clone https://github.com/rupa/z ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/z && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/themes/powerlevel10k && \
    # Config .zshrc: theme + plugins
    sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' /home/kasm-user/.zshrc && \
    sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck fzf)/' /home/kasm-user/.zshrc && \
    # Bypass wizard zsh-newuser-install and config Powerlevel10k
    echo '# Bypass zsh-newuser-install wizard' >> /home/kasm-user/.zshrc && \
    echo '[[ ! -f ~/.p10k.zsh ]] && p10k configure --non-interactive' >> /home/kasm-user/.zshrc && \
    echo '[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh' >> /home/kasm-user/.zshrc && \
    echo '[[ -f /usr/bin/zsh ]] && exec zsh' >> /home/kasm-user/.bashrc

USER root

# Set env vars fcitx5 cho gõ tiếng Việt (lưu ở home user)
RUN echo "export GTK_IM_MODULE=fcitx" >> /home/kasm-user/.xprofile && \
    echo "export QT_IM_MODULE=fcitx" >> /home/kasm-user/.xprofile && \
    echo "export XMODIFIERS=@im=fcitx" >> /home/kasm-user/.xprofile && \
    echo "export SDL_IM_MODULE=fcitx" >> /home/kasm-user/.xprofile && \
    echo "export GLFW_IM_MODULE=fcitx" >> /home/kasm-user/.xprofile && \
    rm -rf /tmp/* /var/tmp/*

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
