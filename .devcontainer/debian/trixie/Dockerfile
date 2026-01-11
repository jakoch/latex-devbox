# SPDX-FileCopyrightText: 2024 Jens A. Koch
# SPDX-License-Identifier: MIT
# This file is part of https://github.com/jakoch/latex-devbox

# +-----------------------------+
# | Builder Stage               |        See https://hub.docker.com/_/debian
# +-----------------------------+

FROM debian:trixie-slim AS builder

ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NOWARNINGS="yes" \
    CURL_OPTIONS="--silent --show-error --retry 5 --location"

WORKDIR /tmp

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "Builder Stage" && \
    apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates \
      curl \
      jq && \
    # +-------------------------------------+
    # | OpenJDK                             |
    # +-------------------------------------+
    # https://www.oracle.com/java/technologies/downloads/
    # Set JAVA_VERSION
    export JAVA_VERSION=25 && \
    # Download JDK and its SHA256 checksum file, verify before extracting
    curl $CURL_OPTIONS_BAR -o jdk.tar.gz -L "https://download.oracle.com/java/${JAVA_VERSION}/latest/jdk-${JAVA_VERSION}_linux-x64_bin.tar.gz" && \
    curl $CURL_OPTIONS_BAR -o jdk.tar.gz.sha256 -L "https://download.oracle.com/java/${JAVA_VERSION}/latest/jdk-${JAVA_VERSION}_linux-x64_bin.tar.gz.sha256" && \
    # Verify checksum file format and value
    if ! grep -qE "^[0-9a-fA-F]{64}" jdk.tar.gz.sha256; then \
        echo "Checksum file missing or malformed"; \
        cat jdk.tar.gz.sha256 || true; \
        exit 1; \
    fi && \
    # Verify checksum
    HASH=$(grep -oE "^[0-9a-fA-F]{64}" jdk.tar.gz.sha256 | head -n1) && \
    echo "$HASH  jdk.tar.gz" | sha256sum -c --status || { \
        echo "JDK checksum verification FAILED"; \
        ls -l jdk.tar.gz jdk.tar.gz.sha256; \
        exit 1; \
    } && \
    # Extract to /usr/local/jdk
    mkdir /usr/local/jdk && \
    tar -xvf jdk.tar.gz -C /usr/local/jdk --strip-components=1 && \
    # Cleanup
    rm -f jdk.tar.gz jdk.tar.gz.sha256 && \
    # +-------------------------------------+
    # | Latex Language Sever (ltex-ls-plus) |
    # +-------------------------------------+
    # Download
    LTEX_VERSION=$(curl -s https://api.github.com/repos/ltex-plus/ltex-ls-plus/releases/latest | jq -r '.tag_name') && \
    echo "Latest Version: \"$LTEX_VERSION\"" && \
    LTEX_URL="https://github.com/ltex-plus/ltex-ls-plus/releases/download/${LTEX_VERSION}/ltex-ls-plus-${LTEX_VERSION}-linux-x64.tar.gz" && \
    echo "Download URL: \"$LTEX_URL\"" && \
    curl $CURL_OPTIONS_BAR -o ltex.tar.gz -L ${LTEX_URL} && \
    # Extract to /opt/ltex
    tar -xzf ltex.tar.gz && \
    mv -v ltex-ls-plus-${LTEX_VERSION} /opt/ltex && \
    # Cleanup
    rm -vf ltex.tar.gz && \
    # +-------------------------------------+
    # | TEX-FMT                             |
    # +-------------------------------------+
    # Download
    TEX_FMT_VERSION=$(curl -s https://api.github.com/repos/wgunderwood/tex-fmt/releases/latest | jq -r '.tag_name') && \
    if [[ -z "$TEX_FMT_VERSION" || "$TEX_FMT_VERSION" == "null" ]]; then echo "Failed to fetch TEX_FMT_VERSION"; exit 1; fi && \
    echo "Latest Version: \"$TEX_FMT_VERSION\"" && \
    TEX_FMT_URL="https://github.com/wgunderwood/tex-fmt/releases/download/${TEX_FMT_VERSION}/tex-fmt-x86_64-linux.tar.gz" && \
    curl $CURL_OPTIONS_BAR -o tex-fmt.tar.gz -L ${TEX_FMT_URL} && \
    # Extract
    tar -xzvf tex-fmt.tar.gz && \
    mv -v tex-fmt /usr/local/bin && \
    # Cleanup
    rm -vf tex-fmt.tar.gz

# +-----------------------------+
# | BASE IMAGE                  |        See https://hub.docker.com/_/debian
# +-----------------------------+

FROM debian:trixie-slim AS latex-devbox-final

ARG USERNAME=texlive
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG USER_GECOS=TeXLive

# +-----------------------------+
# | METADATA                    |
# +-----------------------------+
# Standardized Docker Image Labels
# https://github.com/opencontainers/image-spec/blob/main/annotations.md

LABEL \
    maintainer="Jens A. Koch <jakoch@web.de>" \
    org.opencontainers.image.created="$(date --utc --iso-8601=seconds)" \
    org.opencontainers.image.authors='Jens A. Koch <jakoch@web.de>' \
    org.opencontainers.image.url='https://ghcr.io/jakoch/latex-devbox' \
    org.opencontainers.image.source='https://github.com/jakoch/latex-devbox'\
    org.opencontainers.image.title='ghcr.io/jakoch/cpp-devbox'\
    org.opencontainers.image.description="A Docker-based devcontainer for documentation development. Based on Debian with LaTeX (TexLive+TexLS), Pandoc, AsciiDoc, and Docbook." \
    org.opencontainers.image.licenses=MIT

# +-----------------------------+
# | Setup User                  |
# +-----------------------------+

# Create a non-root user
RUN echo "Create a non-root user" && \
    groupadd --gid $USER_GID $USERNAME && \
    useradd \
    -s /bin/bash \
    --uid $USER_UID \
    --gid $USER_GID \
    -c "$USER_GECOS" \
    -m $USERNAME

# Add sudo support for non-root user
RUN echo "Add sudo support for non-root user" && \
    mkdir -p /etc/sudoers.d && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

#
#  8888888b.                    888
#  888   Y88b                   888
#  888    888                   888
#  888   d88P  8888b.   .d8888b 888  888  8888b.   .d88b.   .d88b.  .d8888b
#  8888888P"      "88b d88P"    888 .88P     "88b d88P"88b d8P  Y8b 88K
#  888        .d888888 888      888888K  .d888888 888  888 88888888 "Y8888b.
#  888        888  888 Y88b.    888 "88b 888  888 Y88b 888 Y8b.          X88
#  888        "Y888888  "Y8888P 888  888 "Y888888  "Y88888  "Y8888   88888P'
#                                                      888
#                                                 Y8b d88P
#                                                  "Y88P"

ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NOWARNINGS="yes" \
    CURL_OPTIONS="--silent --show-error --retry 5 --location"

RUN echo "Packages" && \
    apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    # sudo
    sudo \
    make \
    git git-lfs \
    jq \
    grep ripgrep \
    zip unzip xz-utils xzdec \
    wget curl aria2 \
    # hunspell
    hunspell \
    hunspell-de-de \
    hunspell-tools \
    # asciidoc
    asciidoc \
    # pandoc
    pandoc \
    # docbook
    docbook \
    docbook-xml \
    docbook-xsl \
    xmlto \
    xsltproc \
    dblatex \
    # xml
    xsltproc \
    # locale + timezone
    locales \
    tzdata \
    # shell (installation below)
    #zsh \
    # perl
    perl \
    cpanminus \
    # texlive
    biber \
    chktex \
    ghostscript \
    latex-cjk-common \
    latexmk \
    texinfo \
    texlive \
    texlive-extra-utils \
    texlive-fonts-extra \
    texlive-fonts-recommended \
    texlive-lang-english \
    texlive-lang-german \
    texlive-latex-extra \
    texlive-latex-recommended \
    texlive-luatex \
    texlive-publishers \
    texlive-science \
    texlive-xetex \
    # Fonts
    fontconfig \
    fonts-jetbrains-mono \
    fonts-linuxlibertine \
    fonts-lmodern \
    fonts-noto-color-emoji \
    # Python
    python3 \
    python3-pip && \
	# Cleanup
	apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    # Install perl modules for latexindent via cpanm
    cpanm YAML::Tiny File::HomeDir

# +-------------------------------------+
# | Java via OpenJDK23                 |
# +-------------------------------------+

COPY --from=builder /usr/local/jdk /usr/local/jdk

RUN echo "OpenJDK Symlink" && \
    ln -s /usr/local/jdk/bin/java /usr/bin/java && \
    ln -s /usr/local/jdk/bin/javac /usr/bin/javac && \
    ln -s /usr/local/jdk/bin/jshell /usr/bin/jshell

ENV JAVA_HOME=/usr/local/jdk
ENV PATH=$JAVA_HOME/bin:$PATH

# +-----------------------------+
# | Latex Language Server       |
# +-----------------------------+

COPY --from=builder /opt/ltex /opt/ltex

ENV PATH="/opt/ltex/bin:${PATH}"

# +-----------------------------+
# | TEX-FMT                     |
# +-----------------------------+

COPY --from=builder /usr/local/bin/tex-fmt /usr/local/bin/tex-fmt

#
#  8888888888P  .d8888b.  888    888
#        d88P  d88P  Y88b 888    888
#       d88P   Y88b.      888    888
#      d88P     "Y888b.   8888888888
#     d88P         "Y88b. 888    888
#    d88P            "888 888    888
#   d88P       Y88b  d88P 888    888
#  d8888888888  "Y8888P"  888    888
#
#
# Setup zsh, ohmyzsh, fontconfig, powerline fonts.
# Setup zsh plugins: autosuggestions, autocompletions, history search.

ENV TERM=xterm-256color \
    ZSH_THEME=agnoster

# Setup zsh for the root user
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "ZSH" && \
    # Install fonts
    apt-get update && apt-get install --no-install-recommends --assume-yes \
    zsh fontconfig fonts-powerline && \
    # Download and install fonts-powerline
    # An installation via apt-get is not possible for trixie,
	# because the package is only available in unstable(sid).
    # https://packages.debian.org/sid/fonts-powerline
	# https://api.ftp-master.debian.org/madison?package=fonts-powerline&table=debian
    #FONTS_POWERLINE_VERSION=$(curl -s 'https://api.ftp-master.debian.org/madison?package=fonts-powerline&table=debian&s=unstable&f=json' | jq -r '.[0]["fonts-powerline"].unstable | keys[0]'); echo "Latest Version: \"$FONTS_POWERLINE_VERSION\"" && \
    #FONTS_POWERLINE_URL="http://ftp.de.debian.org/debian/pool/main/p/powerline/fonts-powerline_${FONTS_POWERLINE_VERSION}_all.deb" && \
    #curl -o fonts-powerline.deb -L ${FONTS_POWERLINE_URL} && \
    #dpkg -i fonts-powerline.deb && \
    #rm fonts-powerline.deb && \
    # Install Oh My Zsh
    curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh | sh && \
	# Add Zsh plugins
    git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions && \
    git clone --depth=1 https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search && \
    # Update .zshrc
    sed -i \
      -e 's/^plugins=(git.*)/plugins=(git zsh-completions zsh-autosuggestions zsh-history-substring-search)/' \
      -e 's/^ZSH_THEME="[^"]*"/ZSH_THEME="robbyrussell"/' \
      -e 's/^# *DISABLE_UPDATE_PROMPT="true"/DISABLE_UPDATE_PROMPT="true"/' \
      -e 's/^# *DISABLE_AUTO_UPDATE="true"/DISABLE_AUTO_UPDATE="true"/' \
    /root/.zshrc && \
	# Cleanup
	apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Setup zsh for the texlive user by copying the root configuration
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "ZSH config" && \
    cp -r /root/.oh-my-zsh /home/texlive/.oh-my-zsh && \
    cp /root/.zshrc /home/texlive/.zshrc && \
    sed -i "s/\/root/\/home\/texlive/g" /home/texlive/.zshrc && \
    chown -R texlive:texlive /home/texlive/.oh-my-zsh /home/texlive/.zshrc

# Set the default shell to zsh after it is installed
ENV SHELL=/bin/zsh

# +-----------------------------+
# | CLEANUP                     |
# +-----------------------------+

RUN echo "Cleanup" && \
    apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# +-----------------------------+
# | Set Locale + Timezone       |
# +-----------------------------+

ENV TZ=Europe/Berlin

RUN echo "Locale & Timezone" && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i -e 's/# de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="de_DE.UTF-8"' > /etc/default/locale && \
    dpkg-reconfigure -f noninteractive locales && \
    update-locale LANG=de_DE.UTF-8 && \
    echo "export PATH=$PATH" > /etc/environment

ENV LANG=de_DE.UTF-8 LANGUAGE=de_DE:de LC_ALL=de_DE.UTF-8

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# setup tlmgr for user
USER $USERNAME
RUN tlmgr init-usertree && \
    # pre-generate the font names database
    luaotfload-tool --update --force
