FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=ja_JP.UTF-8
ENV LANGUAGE=ja_JP:ja
ENV LC_ALL=ja_JP.UTF-8

# 1. 基本パッケージ、日本語環境（Noto Fonts, Fcitx-Mozc）、Xfce4、VNCのインストール
RUN apt-get update && apt-get install -y \
    locales \
    fonts-noto-cjk \
    fcitx-mozc \
    dbus-x11 \
    xfce4 \
    xfce4-terminal \
    tigervnc-standalone-server \
    python3-websockify \
    novnc \
    curl \
    wget \
    gpg \
    && locale-gen ja_JP.UTF-8 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 2. Firefoxの非Snapインストール（Mozilla公式PPAを使用）
RUN install -d -m 0755 /etc/apt/keyrings \
    && wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null \
    && echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" | tee /etc/apt/sources.list.d/mozilla.list > /dev/null \
    && echo 'Package: *\nPin: origin packages.mozilla.org\nPin-Priority: 1000' | tee /etc/apt/preferences.d/mozilla \
    && apt-get update && apt-get install -y firefox \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 3. VNC / fcitx 環境設定および起動スクリプトの作成
RUN mkdir -p /root/.vnc \
    && echo "#!/bin/bash\nexport GTK_IM_MODULE=fcitx\nexport QT_IM_MODULE=fcitx\nexport XMODIFIERS=@im=fcitx\nfcitx -d &\nstartxfce4 &" > /root/.vnc/xstartup \
    && chmod +x /root/.vnc/xstartup

RUN echo '#!/bin/bash\n\
rm -rf /tmp/.X11-unix/X1 /tmp/.X1-lock\n\
vncserver :1 -geometry 1280x720 -depth 24 -SecurityTypes None\n\
novnc_server --vnc localhost:5901 --listen 6080 &\n\
tail -f /dev/null' > /usr/local/bin/start-desktop.sh \
    && chmod +x /usr/local/bin/start-desktop.sh

EXPOSE 6080