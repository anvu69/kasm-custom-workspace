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

# --- PHẦN 2: WRAPPER SCRIPTS ---
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
ENV ZSH=/home/kasm-user/.oh-my-zsh

# --- PHẦN 3: USER SETUP ---
# 3.1 OMZ & Plugins
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH}/custom/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH}/custom/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH}/custom/plugins/zsh-autocomplete && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH}/custom/plugins/zsh-completions && \
    git clone https://github.com/rupa/z ${ZSH}/custom/plugins/z && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH}/custom/themes/powerlevel10k

# 3.2 FZF Manual
RUN git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && \
    ~/.fzf/install --bin

# 3.3 Gitstatusd (v1.5.4)
RUN mkdir -p ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin && \
    wget -O ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz https://github.com/romkatv/gitstatus/releases/download/v1.5.4/gitstatusd-linux-x86_64.tar.gz && \
    tar -xzf ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz -C ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/ && \
    rm ${ZSH}/custom/themes/powerlevel10k/gitstatus/usrbin/gitstatusd-linux-x86_64.tar.gz

# 3.4 Wallpaper & Assets
RUN mkdir -p /home/kasm-user/Pictures && \
    wget -O /home/kasm-user/Pictures/wallpaper.jpg https://images.unsplash.com/photo-1477346611705-65d1883cee1e?q=80\&w=1920\&auto=format\&fit=crop

# 3.5 Configs (.zshrc, .p10k, .xprofile)
RUN echo 'export ZSH=$HOME/.oh-my-zsh' > ~/.zshrc && \
    echo 'ZSH_THEME="powerlevel10k/powerlevel10k"' >> ~/.zshrc && \
    echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck)' >> ~/.zshrc && \
    echo 'source $ZSH/oh-my-zsh.sh' >> ~/.zshrc && \
    echo 'export LANG=en_US.UTF-8' >> ~/.zshrc && \
    echo 'export LC_ALL=en_US.UTF-8' >> ~/.zshrc && \
    echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc && \
    echo '[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh' >> ~/.zshrc && \
    echo 'source ~/.fzf/shell/key-bindings.zsh' >> ~/.zshrc && \
    echo 'source ~/.fzf/shell/completion.zsh' >> ~/.zshrc && \
    echo 'alias shot="flameshot gui"' >> ~/.zshrc

# 3.6 P10k Config
RUN curl -fsSL https://raw.githubusercontent.com/romkatv/powerlevel10k/master/config/p10k-lean.zsh -o ~/.p10k.zsh && \
    sed -i 's/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon /' ~/.p10k.zsh

# .xprofile
RUN echo 'export GTK_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export QT_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export XMODIFIERS=@im=fcitx' >> ~/.xprofile && \
    echo 'export SDL_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'fcitx5 -d &' >> ~/.xprofile

# 3.7 Config i3wm & Rofi
RUN mkdir -p /home/kasm-user/.config/i3 && \
    mkdir -p /home/kasm-user/.config/rofi && \
    # --- I3 CONFIG ---
    echo 'set $mod Mod4' > /home/kasm-user/.config/i3/config && \
    echo 'set $alt Mod1' >> /home/kasm-user/.config/i3/config && \
    echo 'font pango:MesloLGS NF 10' >> /home/kasm-user/.config/i3/config && \
    echo 'exec --no-startup-id picom -b --active-opacity 1.0 --inactive-opacity 0.95 --fade-in-step 0.05' >> /home/kasm-user/.config/i3/config && \
    echo 'exec_always --no-startup-id feh --bg-scale /home/kasm-user/Pictures/wallpaper.jpg' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Return exec wezterm' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $alt+space exec "rofi -show drun -show-icons"' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $alt+Tab exec "rofi -show window -show-icons"' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+q kill' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+d exec "rofi -show drun -show-icons"' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Tab exec "rofi -show window -show-icons"' >> /home/kasm-user/.config/i3/config && \
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
    echo 'bindsym $mod+Shift+c reload' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+r restart' >> /home/kasm-user/.config/i3/config && \
    echo 'bindsym $mod+Shift+e exec "i3-nagbar -t warning -m \"Exit?\" -B \"Yes\" \"i3-msg exit\""' >> /home/kasm-user/.config/i3/config && \
    echo 'gaps inner 10' >> /home/kasm-user/.config/i3/config && \
    echo 'gaps outer 0' >> /home/kasm-user/.config/i3/config && \
    echo 'smart_gaps on' >> /home/kasm-user/.config/i3/config && \
    echo 'default_border pixel 2' >> /home/kasm-user/.config/i3/config && \
    echo 'bar { status_command i3status position top colors { background #1e1e2e statusline #cdd6f4 separator #f5e0dc focused_workspace #f5c2e7 #f5c2e7 #1e1e2e } }' >> /home/kasm-user/.config/i3/config

# 3.8 Config Rofi Theme
RUN echo 'configuration { display-drun: "Applications:"; display-window: "Windows:"; show-icons: true; }' > /home/kasm-user/.config/rofi/config.rasi && \
    echo '@theme "glue_pro_blue"' >> /home/kasm-user/.config/rofi/config.rasi

USER root

# --- PHẦN 4: SYNC PROFILE ---
RUN mkdir -p /home/kasm-default-profile/.config && \
    mkdir -p /home/kasm-default-profile/.oh-my-zsh && \
    cp -r /home/kasm-user/.config/i3 /home/kasm-default-profile/.config/i3 && \
    cp -r /home/kasm-user/.config/rofi /home/kasm-default-profile/.config/rofi && \
    cp -r /home/kasm-user/.oh-my-zsh /home/kasm-default-profile/.oh-my-zsh && \
    cp -r /home/kasm-user/.fzf /home/kasm-default-profile/.fzf && \
    cp /home/kasm-user/.zshrc /home/kasm-default-profile/.zshrc && \
    cp /home/kasm-user/.p10k.zsh /home/kasm-default-profile/.p10k.zsh && \
    cp /home/kasm-user/.xprofile /home/kasm-default-profile/.xprofile && \
    chown -R 1000:1000 /home/kasm-default-profile

# PHẦN 5: SCRIPT KHỞI ĐỘNG THÔNG MINH (SELF-HEALING)
# Script này thay thế startxfce4, có thêm logic kiểm tra và hồi phục config
RUN echo '#!/bin/bash' > /usr/bin/startxfce4 && \
    echo 'LOG="/home/kasm-user/startup.log"' >> /usr/bin/startxfce4 && \
    echo 'echo "--- Startup $(date) ---" > $LOG' >> /usr/bin/startxfce4 && \
    # 1. Load môi trường
    echo 'if [ -r /home/kasm-user/.xprofile ]; then . /home/kasm-user/.xprofile; fi' >> /usr/bin/startxfce4 && \
    # 2. KIỂM TRA & TỰ SỬA LỖI (Quan trọng)
    # LOGIC KHÔI PHỤC
    echo 'if [ ! -f /home/kasm-user/.config/i3/config ]; then' >> /usr/bin/startxfce4 && \
    echo '    echo "MISSING CONFIG. RESTORING..." >> $LOG' >> /usr/bin/startxfce4 && \
    echo '    mkdir -p /home/kasm-user/.config' >> /usr/bin/startxfce4 && \
    echo '    cp -r /home/kasm-default-profile/.config/i3 /home/kasm-user/.config/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    cp -r /home/kasm-default-profile/.config/rofi /home/kasm-user/.config/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    cp -r /home/kasm-default-profile/.oh-my-zsh /home/kasm-user/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    cp /home/kasm-default-profile/.zshrc /home/kasm-user/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    cp /home/kasm-default-profile/.p10k.zsh /home/kasm-user/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    cp /home/kasm-default-profile/.xprofile /home/kasm-user/ >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    echo '    echo "Restore complete." >> $LOG' >> /usr/bin/startxfce4 && \
    echo 'fi' >> /usr/bin/startxfce4 && \
    echo 'echo "Fixing permissions..." >> $LOG' >> /usr/bin/startxfce4 && \
    echo 'chown -R 1000:1000 /home/kasm-user/.config /home/kasm-user/.oh-my-zsh /home/kasm-user/.zshrc /home/kasm-user/.p10k.zsh >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    # 3. Chạy i3
    echo 'echo "Starting i3..." >> $LOG' >> /usr/bin/startxfce4 && \
    # Dùng dbus-launch để đảm bảo môi trường desktop ổn định
    echo 'exec dbus-launch --exit-with-session /usr/bin/i3 -V >> $LOG 2>&1' >> /usr/bin/startxfce4 && \
    chmod +x /usr/bin/startxfce4

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
