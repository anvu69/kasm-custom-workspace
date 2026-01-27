FROM kasmweb/core-debian-bookworm:1.18.0

USER root

# --- PHẦN 1: CÀI ĐẶT PACKAGE ---
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
        x11-utils \
        flameshot \
        i3 rofi picom feh i3status && \
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
# VSCode
RUN sed -i 's|^Exec=.*|Exec=/usr/share/code/code --no-sandbox --unity-launch %F|' /usr/share/applications/code.desktop && \
    sed -i 's|Icon=.*|Icon=com.visualstudio.code|' /usr/share/applications/code.desktop && \
    mv /usr/bin/code /usr/bin/code-orig && \
    echo '#!/bin/bash' > /usr/bin/code && \
    echo '/usr/bin/code-orig --no-sandbox --unity-launch "$@" 2>/dev/null' >> /usr/bin/code && \
    chmod +x /usr/bin/code

# Chromium
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

# 3.2 Cài FZF Thủ công
RUN git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && \
    ~/.fzf/install --bin

# 3.3 Pre-download Gitstatusd (Đã fix link v1.5.4)
RUN mkdir -p ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin && \
    wget -O ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz https://github.com/romkatv/gitstatus/releases/download/v1.5.4/gitstatusd-linux-x86_64.tar.gz && \
    tar -xzf ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz -C ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/ && \
    rm ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz

# 3.4 Config .zshrc
# Thay đổi quan trọng: Source trực tiếp file trong folder .fzf thay vì tìm file .fzf.zsh
RUN echo 'export ZSH=$HOME/.oh-my-zsh' > ~/.zshrc && \
    echo 'ZSH_THEME="powerlevel10k/powerlevel10k"' >> ~/.zshrc && \
    echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck)' >> ~/.zshrc && \
    echo 'source $ZSH/oh-my-zsh.sh' >> ~/.zshrc && \
    echo 'export LANG=en_US.UTF-8' >> ~/.zshrc && \
    echo 'export LC_ALL=en_US.UTF-8' >> ~/.zshrc && \
    echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc && \
    echo '[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh' >> ~/.zshrc && \
    # Source FZF trực tiếp từ repo (Fix lỗi copy không tìm thấy file)
    echo 'source ~/.fzf/shell/key-bindings.zsh' >> ~/.zshrc && \
    echo 'source ~/.fzf/shell/completion.zsh' >> ~/.zshrc

RUN curl -fsSL https://raw.githubusercontent.com/romkatv/powerlevel10k/master/config/p10k-lean.zsh -o ~/.p10k.zsh && \
    sed -i 's/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon /' ~/.p10k.zsh

RUN echo 'export GTK_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export QT_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export XMODIFIERS=@im=fcitx' >> ~/.xprofile && \
    echo 'export SDL_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'fcitx5 -d &' >> ~/.xprofile

# --- SETUP I3WM ---
# 1. Tạo thư mục config
RUN mkdir -p /home/kasm-user/.config/i3 && \
    mkdir -p /home/kasm-user/.config/rofi

# 2. Tạo file config cơ bản cho i3 (để tránh wizard lần đầu)
# Lưu ý: "mod4" là phím Windows.
RUN echo 'set $mod Mod4' > /home/kasm-user/.config/i3/config && \
    echo 'font pango:MesloLGS NF 10' >> /home/kasm-user/.config/i3/config && \
    echo 'exec --no-startup-id picom -b' >> /home/kasm-user/.config/i3/config && \
    echo 'exec --no-startup-id fcitx5 -d' >> /home/kasm-user/.config/i3/config && \
    # Set hình nền (Kasm mặc định có hình nền ở bg_default.png)
    echo 'exec_always --no-startup-id feh --bg-scale /usr/share/backgrounds/bg_default.png' >> /home/kasm-user/.config/i3/config && \
    # Phím tắt mở Terminal (Dùng WezTerm)
    echo 'bindsym $mod+Return exec wezterm' >> /home/kasm-user/.config/i3/config && \
    # Phím tắt tắt cửa sổ
    echo 'bindsym $mod+Shift+q kill' >> /home/kasm-user/.config/i3/config && \
    # Phím tắt mở Rofi (Launcher)
    echo 'bindsym $mod+d exec "rofi -show drun"' >> /home/kasm-user/.config/i3/config && \
    # Các điều hướng cửa sổ cơ bản
    echo 'bindsym $mod+Left focus left' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Down focus down' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Up focus up' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Right focus right' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+Left move left' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+Down move down' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+Up move up' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+Right move right' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+h split h' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+v split v' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+f fullscreen toggle' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+s layout stacking' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+w layout tabbed' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+e layout toggle split' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+space floating toggle' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+space focus mode_toggle' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+r restart' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+e exec "i3-nagbar -t warning -m \"Exit?\" -B \"Yes\" \"i3-msg exit\""' >> /home/kasm-user/.config/i3/config && \
    # Gaps (Khoảng cách giữa các cửa sổ - Modern Look)
    echo 'gaps inner 10' >> /home/kasm-user/.config/i3/config && \
    echo 'gaps outer 0' >> /home/kasm-user/.config/i3/config && \
    echo 'smart_gaps on' >> /home/kasm-user/.config/i3/config && \
    # Thanh trạng thái
    echo 'bar { status_command i3status position top }' >> /home/kasm-user/.config/i3/config

# 3. QUAN TRỌNG: Tạo file .xsession để Kasm chạy i3 thay vì XFCE
# Kasm VNC startup script sẽ ưu tiên file này
RUN echo '#!/bin/bash' > /home/kasm-user/.xsession && \
    echo 'export DESTKTOP_SESSION=i3' >> /home/kasm-user/.xsession && \
    echo 'exec i3' >> /home/kasm-user/.xsession && \
    chmod +x /home/kasm-user/.xsession

USER root

# --- PHẦN 4: SYNC ---
# Đã xóa dòng cp .fzf.zsh gây lỗi
RUN cp -r /home/kasm-user/.oh-my-zsh /home/kasm-default-profile/.oh-my-zsh && \
    cp -r /home/kasm-user/.fzf /home/kasm-default-profile/.fzf && \
    cp /home/kasm-user/.zshrc /home/kasm-default-profile/.zshrc && \
    cp /home/kasm-user/.p10k.zsh /home/kasm-default-profile/.p10k.zsh && \
    cp /home/kasm-user/.xprofile /home/kasm-default-profile/.xprofile && \
    cp -r /home/kasm-user/.config /home/kasm-default-profile/.config && \
    cp /home/kasm-user/.xsession /home/kasm-default-profile/.xsession && \
    chown -R 1000:1000 /home/kasm-default-profile

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
