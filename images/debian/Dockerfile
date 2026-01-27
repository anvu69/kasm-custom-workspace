FROM kasmweb/core-debian-bookworm:1.18.0

USER root

# --- PHẦN 1: CÀI ĐẶT PACKAGE ---
# Đã loại bỏ 'fzf' khỏi apt để cài thủ công bên dưới
RUN apt update && \
    apt install -y --no-install-recommends \
        build-essential git curl wget python3 python3-pip nodejs npm golang-go \
        fonts-noto fonts-dejavu fonts-noto-cjk \
        fcitx5 fcitx5-unikey fcitx5-config-qt fcitx5-frontend-gtk2 fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 \
        software-properties-common apt-transport-https \
        gnupg ca-certificates \
        zsh thefuck fontconfig \
        libgbm1 libxshmfence1 libasound2 \
        chromium \
        x11-utils && \
    # Setup VSCode
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /usr/share/keyrings/microsoft-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list && \
    # Setup WezTerm
    curl -fsSL https://apt.fury.io/wez/gpg.key | gpg --dearmor -o /usr/share/keyrings/wezterm-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/wezterm-archive-keyring.gpg] https://apt.fury.io/wez/ * *" > /etc/apt/sources.list.d/wezterm.list && \
    apt update && \
    apt install -y code wezterm && \
    # Fonts P10k
    mkdir -p /usr/local/share/fonts && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf && \
    fc-cache -fv && \
    apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# --- PHẦN 2: WRAPPER SCRIPTS (CLEAN LOGS) ---

# VSCode: Thêm 2>/dev/null để giấu warning 'unity-launch'
RUN sed -i 's|^Exec=.*|Exec=/usr/share/code/code --no-sandbox --unity-launch %F|' /usr/share/applications/code.desktop && \
    sed -i 's|Icon=.*|Icon=com.visualstudio.code|' /usr/share/applications/code.desktop && \
    mv /usr/bin/code /usr/bin/code-orig && \
    echo '#!/bin/bash' > /usr/bin/code && \
    echo '/usr/bin/code-orig --no-sandbox --unity-launch "$@" 2>/dev/null' >> /usr/bin/code && \
    chmod +x /usr/bin/code

# Chromium: Thêm --test-type để giấu warning 'unsupported command-line flag'
RUN sed -i 's|Exec=/usr/bin/chromium %U|Exec=/usr/bin/chromium --no-sandbox --test-type %U|' /usr/share/applications/chromium.desktop && \
    mv /usr/bin/chromium /usr/bin/chromium-orig && \
    echo '#!/bin/bash' > /usr/bin/chromium && \
    echo '/usr/bin/chromium-orig --no-sandbox --test-type "$@" 2>/dev/null' >> /usr/bin/chromium && \
    chmod +x /usr/bin/chromium

RUN usermod -s /usr/bin/zsh kasm-user

USER 1000

# --- PHẦN 3: USER SETUP (ZSH, FZF, P10K) ---

ENV ZSH=/home/kasm-user/.oh-my-zsh
# 3.1 Cài OMZ và Plugins
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH}/custom/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH}/custom/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH}/custom/plugins/zsh-autocomplete && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH}/custom/plugins/zsh-completions && \
    git clone https://github.com/rupa/z ${ZSH}/custom/plugins/z && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH}/custom/themes/powerlevel10k

# 3.2 Cài FZF Thủ công (Fix lỗi warning đường dẫn Debian)
RUN git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && \
    ~/.fzf/install --bin

# 3.3 Pre-download Gitstatusd (Fix lỗi fetching... khi khởi động)
# Tải binary gitstatusd cache sẵn vào thư mục theme để P10k không phải tải lại
RUN mkdir -p ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin && \
    wget -O ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz https://github.com/romkatv/gitstatus/releases/download/v1.5.5/gitstatusd-linux-x86_64.tar.gz && \
    tar -xzf ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz -C ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/ && \
    rm ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz

# 3.4 Config .zshrc
# Lưu ý: Đã xóa 'fzf' khỏi plugins=(...) để tránh xung đột với bản cài tay
RUN echo 'export ZSH=$HOME/.oh-my-zsh' > ~/.zshrc && \
    echo 'ZSH_THEME="powerlevel10k/powerlevel10k"' >> ~/.zshrc && \
    echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck)' >> ~/.zshrc && \
    echo 'source $ZSH/oh-my-zsh.sh' >> ~/.zshrc && \
    echo 'export LANG=en_US.UTF-8' >> ~/.zshrc && \
    echo 'export LC_ALL=en_US.UTF-8' >> ~/.zshrc && \
    echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc && \
    echo '[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh' >> ~/.zshrc && \
    # Source FZF cài tay
    echo '[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh' >> ~/.zshrc

RUN curl -fsSL https://raw.githubusercontent.com/romkatv/powerlevel10k/master/config/p10k-lean.zsh -o ~/.p10k.zsh && \
    sed -i 's/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon /' ~/.p10k.zsh

RUN echo 'export GTK_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export QT_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export XMODIFIERS=@im=fcitx' >> ~/.xprofile && \
    echo 'export SDL_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'fcitx5 -d &' >> ~/.xprofile

USER root

# --- PHẦN 4: SYNC ---
RUN cp -r /home/kasm-user/.oh-my-zsh /home/kasm-default-profile/.oh-my-zsh && \
    cp -r /home/kasm-user/.fzf /home/kasm-default-profile/.fzf && \
    cp /home/kasm-user/.zshrc /home/kasm-default-profile/.zshrc && \
    cp /home/kasm-user/.p10k.zsh /home/kasm-default-profile/.p10k.zsh && \
    cp /home/kasm-user/.xprofile /home/kasm-default-profile/.xprofile && \
    # Sync cả file config fzf sinh ra
    cp /home/kasm-user/.fzf.zsh /home/kasm-default-profile/.fzf.zsh && \
    chown -R 1000:1000 /home/kasm-default-profile

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
