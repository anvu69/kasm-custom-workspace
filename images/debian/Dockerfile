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
    # WezTerm repo (mới: apt.fury.io/wez)
    curl -fsSL https://apt.fury.io/wez/gpg.key | gpg --dearmor -o /usr/share/keyrings/wezterm-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/wezterm-archive-keyring.gpg] https://apt.fury.io/wez/ * *" > /etc/apt/sources.list.d/wezterm.list && \
    apt update && \
    apt install -y code wezterm && \
    # Oh My Zsh + plugins
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-autocomplete && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/zsh-completions && \
    git clone https://github.com/rupa/z ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/plugins/z && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-/home/kasm-user/.oh-my-zsh/custom}/themes/powerlevel10k && \
    sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' /home/kasm-user/.zshrc && \
    sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck fzf)/' /home/kasm-user/.zshrc && \
    chsh -s /usr/bin/zsh kasm-user && \
    # Env vars fcitx5
    echo "export GTK_IM_MODULE=fcitx\nexport QT_IM_MODULE=fcitx\nexport XMODIFIERS=@im=fcitx\nexport SDL_IM_MODULE=fcitx\nexport GLFW_IM_MODULE=fcitx" >> /home/kasm-user/.xprofile && \
    echo '[[ -f /usr/bin/zsh ]] && exec zsh' >> /home/kasm-user/.bashrc && \
    apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN touch /home/kasm-user/.zshrc && \
    # Force Powerlevel10k config tự động (không hỏi user)
    echo '[[ ! -f ~/.p10k.zsh ]] && p10k configure --non-interactive' >> /home/kasm-user/.zshrc && \
    # Đảm bảo load p10k ngay
    echo '[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh' >> /home/kasm-user/.zshrc

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
