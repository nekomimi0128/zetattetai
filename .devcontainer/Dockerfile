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
    fcitx-frontend-gtk3 \
    fcitx-frontend-qt5 \
    fcitx-ui-classic \
    dbus-x11 \
    xfce4 \
    xfce4-terminal \
    tigervnc-standalone-server \
    python3-websockify \
    novnc \
    curl \
    wget \
    gpg \
    && rm -f /etc/xdg/autostart/xfce4-screensaver.desktop /etc/xdg/autostart/xscreensaver.desktop \
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
RUN mkdir -p /root/.vnc /home/vscode/.vnc \
    && chown -R vscode:vscode /home/vscode/.vnc \
    && printf '%s\n' \
      '#!/bin/bash' \
      'export GTK_IM_MODULE=fcitx' \
      'export QT_IM_MODULE=fcitx' \
      'export XMODIFIERS=@im=fcitx' \
      'export DefaultIMModule=fcitx' \
      'setxkbmap -model pc105 -layout jp' \
      'eval "$(dbus-launch --sh-syntax --exit-with-session)"' \
      'xset s off' \
      'xfconf-query -c xfce4-screensaver -p /lock/enabled -n -t bool -s false 2>/dev/null || true' \
      'fcitx -d &' \
      'fcitx-remote -s mozc 2>/dev/null || true' \
      'exec startxfce4' > /home/vscode/.vnc/xstartup \
    && chmod +x /home/vscode/.vnc/xstartup \
    && printf '%s\n' \
      '#!/bin/bash' \
      'export GTK_IM_MODULE=fcitx' \
      'export QT_IM_MODULE=fcitx' \
      'export XMODIFIERS=@im=fcitx' \
      'export DefaultIMModule=fcitx' \
      'setxkbmap -model pc105 -layout jp' \
      'eval "$(dbus-launch --sh-syntax --exit-with-session)"' \
      'xset s off' \
      'xfconf-query -c xfce4-screensaver -p /lock/enabled -n -t bool -s false 2>/dev/null || true' \
      'fcitx -d &' \
      'fcitx-remote -s mozc 2>/dev/null || true' \
      'startxfce4 &' > /root/.vnc/xstartup \
    && chmod +x /root/.vnc/xstartup

RUN printf '%s\n' \
    '#!/bin/bash' \
    'mkdir -p /home/vscode/.vnc /home/vscode/.config/fcitx/conf' \
    'mkdir -p /home/vscode/.vnc' \
    'cat > /home/vscode/.vnc/xstartup <<"EOF"' \
    'export GTK_IM_MODULE=fcitx' \
    'export QT_IM_MODULE=fcitx' \
    'export XMODIFIERS=@im=fcitx' \
    'export DefaultIMModule=fcitx' \
    'setxkbmap -model pc105 -layout jp' \
    'eval "$(dbus-launch --sh-syntax --exit-with-session)"' \
    'xset s off' \
    'xfconf-query -c xfce4-screensaver -p /lock/enabled -n -t bool -s false 2>/dev/null || true' \
    'fcitx -d &' \
    'fcitx-remote -s mozc 2>/dev/null || true' \
    'exec startxfce4' \
    'EOF' \
    'chmod +x /home/vscode/.vnc/xstartup' \
    'if [ -f /home/vscode/.config/fcitx/config ]; then sed -i -E "s/^#?TriggerKey=.*/TriggerKey=Zenkaku_Hankaku/" /home/vscode/.config/fcitx/config; else printf "%s\\n" "[Hotkey]" "TriggerKey=Zenkaku_Hankaku" > /home/vscode/.config/fcitx/config; fi' \
    'rm -rf /tmp/.X11-unix/X1 /tmp/.X1-lock' \
    'export GTK_IM_MODULE=fcitx' \
    'export QT_IM_MODULE=fcitx' \
    'export XMODIFIERS=@im=fcitx' \
    'vncserver :1 -geometry 1280x720 -depth 24 -SecurityTypes None' \
    'nohup websockify --web=/usr/share/novnc 6080 localhost:5901 >/tmp/novnc.log 2>&1 &' > /usr/local/bin/start-desktop.sh \
    && chmod +x /usr/local/bin/start-desktop.sh

EXPOSE 6080