FROM kasmweb/core-debian-bookworm:1.18.0

USER root

# --- PHẦN 1: CÀI ĐẶT PACKAGE HỆ THỐNG & FONTS (Dùng apt) ---
RUN apt update && \
    apt install -y --no-install-recommends \
        build-essential git curl wget python3 python3-pip nodejs npm golang-go \
        fonts-noto fonts-dejavu fonts-noto-cjk \
        fcitx5 fcitx5-unikey fcitx5-config-qt fcitx5-frontend-gtk2 fcitx5-frontend-gtk3 fcitx5-frontend-gtk4 \
        software-properties-common apt-transport-https \
        gnupg ca-certificates \
        zsh fzf thefuck fontconfig \
        # Thư viện cần thiết cho VSCode/Electron/Chromium
        libgbm1 libxshmfence1 libasound2 \
        # Cài Chromium (trình duyệt)
        chromium \
        x11-utils && \
    # --- Setup Repo cho VSCode ---
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /usr/share/keyrings/microsoft-archive-keyring.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft-archive-keyring.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list && \
    # --- Setup Repo cho WezTerm ---
    curl -fsSL https://apt.fury.io/wez/gpg.key | gpg --dearmor -o /usr/share/keyrings/wezterm-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/wezterm-archive-keyring.gpg] https://apt.fury.io/wez/ * *" > /etc/apt/sources.list.d/wezterm.list && \
    # Cập nhật lại và cài đặt Code + Wezterm
    apt update && \
    apt install -y code wezterm && \
    # --- Cài đặt Font MesloLGS NF (Bắt buộc cho Powerlevel10k) ---
    mkdir -p /usr/local/share/fonts && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf && \
    wget -P /usr/local/share/fonts https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf && \
    fc-cache -fv && \
    # Dọn dẹp (Dùng apt clean)
    apt clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# --- PHẦN 2: FIX LỖI SANDBOX CHO VSCODE VÀ CHROMIUM ---

# 2.1 Fix VSCode
# Sửa .desktop để chạy từ Menu
RUN sed -i 's|^Exec=.*|Exec=/usr/share/code/code --no-sandbox --unity-launch %F|' /usr/share/applications/code.desktop && \
    sed -i 's|Icon=.*|Icon=com.visualstudio.code|' /usr/share/applications/code.desktop
# Tạo wrapper script để chạy từ Terminal (lệnh 'code')
RUN mv /usr/bin/code /usr/bin/code-orig && \
    echo '#!/bin/bash' > /usr/bin/code && \
    echo '/usr/bin/code-orig --no-sandbox --unity-launch "$@"' >> /usr/bin/code && \
    chmod +x /usr/bin/code

# 2.2 Fix Chromium
# Sửa .desktop để chạy từ Menu
RUN sed -i 's|Exec=/usr/bin/chromium %U|Exec=/usr/bin/chromium --no-sandbox %U|' /usr/share/applications/chromium.desktop
# Tạo wrapper script để chạy từ Terminal (lệnh 'chromium')
RUN mv /usr/bin/chromium /usr/bin/chromium-orig && \
    echo '#!/bin/bash' > /usr/bin/chromium && \
    echo '/usr/bin/chromium-orig --no-sandbox "$@"' >> /usr/bin/chromium && \
    chmod +x /usr/bin/chromium

# --- PHẦN 3: THIẾT LẬP MÔI TRƯỜNG USER ---
# Đổi shell mặc định sang Zsh
RUN usermod -s /usr/bin/zsh kasm-user

USER 1000

# Cài đặt Oh-My-Zsh và Plugins
ENV ZSH=/home/kasm-user/.oh-my-zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended && \
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH}/custom/plugins/zsh-autosuggestions && \
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH}/custom/plugins/zsh-syntax-highlighting && \
    git clone https://github.com/marlonrichert/zsh-autocomplete.git ${ZSH}/custom/plugins/zsh-autocomplete && \
    git clone https://github.com/zsh-users/zsh-completions ${ZSH}/custom/plugins/zsh-completions && \
    git clone https://github.com/rupa/z ${ZSH}/custom/plugins/z && \
    git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH}/custom/themes/powerlevel10k

# Cấu hình .zshrc
RUN echo 'export ZSH=$HOME/.oh-my-zsh' > ~/.zshrc && \
    echo 'ZSH_THEME="powerlevel10k/powerlevel10k"' >> ~/.zshrc && \
    echo 'plugins=(git zsh-autosuggestions zsh-syntax-highlighting zsh-autocomplete zsh-completions z thefuck fzf)' >> ~/.zshrc && \
    echo 'source $ZSH/oh-my-zsh.sh' >> ~/.zshrc && \
    # Setting quan trọng
    echo 'export LANG=en_US.UTF-8' >> ~/.zshrc && \
    echo 'export LC_ALL=en_US.UTF-8' >> ~/.zshrc && \
    echo 'POWERLEVEL9K_DISABLE_CONFIGURATION_WIZARD=true' >> ~/.zshrc && \
    echo '[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh' >> ~/.zshrc

# Tải cấu hình P10k mẫu (Lean Style)
RUN curl -fsSL https://raw.githubusercontent.com/romkatv/powerlevel10k/master/config/p10k-lean.zsh -o ~/.p10k.zsh && \
    # Thêm icon OS vào prompt cho đẹp
    sed -i 's/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(/POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon /' ~/.p10k.zsh

# Cấu hình Auto-start Fcitx5 (Tiếng Việt)
RUN echo 'export GTK_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export QT_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'export XMODIFIERS=@im=fcitx' >> ~/.xprofile && \
    echo 'export SDL_IM_MODULE=fcitx' >> ~/.xprofile && \
    echo 'fcitx5 -d &' >> ~/.xprofile

USER root

# --- PHẦN 4: SYNC CẤU HÌNH (QUAN TRỌNG NHẤT CHO PERSISTENT PROFILE) ---
# Copy toàn bộ cấu hình đã tạo sang thư mục default profile
# Kasm sẽ dùng thư mục này để khởi tạo lại home user khi mount volume
RUN cp -r /home/kasm-user/.oh-my-zsh /home/kasm-default-profile/.oh-my-zsh && \
    cp /home/kasm-user/.zshrc /home/kasm-default-profile/.zshrc && \
    cp /home/kasm-user/.p10k.zsh /home/kasm-default-profile/.p10k.zsh && \
    cp /home/kasm-user/.xprofile /home/kasm-default-profile/.xprofile && \
    chown -R 1000:1000 /home/kasm-default-profile

USER 1000
WORKDIR /home/kasm-user
EXPOSE 5901
