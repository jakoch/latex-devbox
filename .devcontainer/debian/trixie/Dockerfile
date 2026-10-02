# SPDX-FileCopyrightText: 2024 Jens A. Koch
# SPDX-License-Identifier: MIT
# This file is part of https://github.com/jakoch/latex-devbox

# +-----------------------------+
# | Builder Stage               |        See https://hub.docker.com/_/debian
# +-----------------------------+

FROM debian:trixie-slim AS builder

# Set automatically by BuildKit/buildx.
ARG TARGETARCH

# Upstream releases are pinned.
# Bumping a download means bumping its version and its checksum together.
# Nothing is fetched from a moving "latest" URL.
ARG JAVA_VERSION=25.0.4
ARG LTEX_VERSION=18.7.0
ARG TEX_FMT_VERSION=0.5.7

ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NOWARNINGS="yes" \
    CURL_OPTIONS="--silent --show-error --retry 5 --location"

WORKDIR /tmp

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "Builder Stage" && \
    apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates \
      curl && \
    rm -rf /var/lib/apt/lists/* /var/tmp/* && \
    # +-------------------------------------+
    # | Architecture mapping                  |
    # +-------------------------------------+
    # Each upstream project names its release assets differently,
    # so map Docker's TARGETARCH onto the three naming schemes used below:
    # Oracle JDK, ltex-ls-plus and tex-fmt.
    case "${TARGETARCH}" in \
      amd64) JDK_ARCH=x64;     LTEX_ARCH=x64;     TEX_FMT_ARCH=x86_64  ;; \
      arm64) JDK_ARCH=aarch64; LTEX_ARCH=aarch64; TEX_FMT_ARCH=aarch64 ;; \
      *) echo "Unsupported TARGETARCH: '${TARGETARCH}' (supported: amd64, arm64)"; exit 1 ;; \
    esac && \
    # +-------------------------------------+
    # | Checksums                           |
    # +-------------------------------------+
    # sha256 sums of the releases pinned in the ARGs above.
    # Upstream publishes no checksum file for these,
    # so they are verified against the recorded sum.
    case "${TARGETARCH}" in \
      amd64) \
        LTEX_SHA256=1e16df6c578dc76ff97d644445d126ba6fba5c2e8e174178ab86372652fd7612; \
        TEX_FMT_SHA256=d0e30b9b811fdc1e66a9328d5c95e9d8e585019288ca39c07f643e0796bcc4a3 ;; \
      arm64) \
        LTEX_SHA256=3a92a4dd22ea87ff5d4de4891581ff41bacc7210256ebe9d0e496f1da8382f54; \
        TEX_FMT_SHA256=ad5a6c8a84f61de711f2d7659191cd7e813ad1881b384abed60df420e1eabdd1 ;; \
    esac && \
    echo "TARGETARCH=${TARGETARCH} -> JDK_ARCH=${JDK_ARCH} LTEX_ARCH=${LTEX_ARCH} TEX_FMT_ARCH=${TEX_FMT_ARCH}" && \
    # +-------------------------------------+
    # | OpenJDK                             |
    # +-------------------------------------+
    # https://www.oracle.com/java/technologies/downloads/
    # Early access downloads: https://jdk.java.net/
    # Patched releases live in the "archive" directory, the checksum file is
    # published next to the tarball and verified below.
    JDK_URL="https://download.oracle.com/java/${JAVA_VERSION%%.*}/archive/jdk-${JAVA_VERSION}_linux-${JDK_ARCH}_bin.tar.gz" && \
    curl $CURL_OPTIONS -o jdk.tar.gz -L "${JDK_URL}" && \
    curl $CURL_OPTIONS -o jdk.tar.gz.sha256 -L "${JDK_URL}.sha256" && \
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
    LTEX_URL="https://github.com/ltex-plus/ltex-ls-plus/releases/download/${LTEX_VERSION}/ltex-ls-plus-${LTEX_VERSION}-linux-${LTEX_ARCH}.tar.gz" && \
    echo "Download URL: \"$LTEX_URL\"" && \
    curl $CURL_OPTIONS -o ltex.tar.gz -L "${LTEX_URL}" && \
    # Verify checksum
    echo "${LTEX_SHA256}  ltex.tar.gz" | sha256sum -c --status || { \
        echo "ltex-ls-plus checksum verification FAILED"; \
        ls -l ltex.tar.gz; \
        exit 1; \
    } && \
    # Extract to /opt/ltex
    tar -xzf ltex.tar.gz && \
    mv -v "ltex-ls-plus-${LTEX_VERSION}" /opt/ltex && \
    # Cleanup
    rm -vf ltex.tar.gz && \
    # +-------------------------------------+
    # | TEX-FMT                             |
    # +-------------------------------------+
    TEX_FMT_URL="https://github.com/wgunderwood/tex-fmt/releases/download/v${TEX_FMT_VERSION}/tex-fmt-${TEX_FMT_ARCH}-linux.tar.gz" && \
    curl $CURL_OPTIONS -o tex-fmt.tar.gz -L "${TEX_FMT_URL}" && \
    # Verify checksum
    echo "${TEX_FMT_SHA256}  tex-fmt.tar.gz" | sha256sum -c --status || { \
        echo "tex-fmt checksum verification FAILED"; \
        ls -l tex-fmt.tar.gz; \
        exit 1; \
    } && \
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
    org.opencontainers.image.authors='Jens A. Koch <jakoch@web.de>' \
    org.opencontainers.image.url='https://github.com/jakoch/latex-devbox' \
    org.opencontainers.image.source='https://github.com/jakoch/latex-devbox' \
    org.opencontainers.image.title='ghcr.io/jakoch/latex-devbox' \
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

# Note: apt/dpkg settings are exported inside the RUN steps that need them,
# so that they do not end up in the environment of the final image.
RUN echo "Packages" && \
    export DEBIAN_FRONTEND=noninteractive DEBCONF_NOWARNINGS="yes" && \
    apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    # sudo
    sudo \
    make \
    git git-lfs \
    jq \
    ripgrep \
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
    rm -rf /var/lib/apt/lists/* /var/tmp/* && \
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

ENV TERM=xterm-256color

# Oh My Zsh and its plugins come from pinned commits.
ARG OH_MY_ZSH_COMMIT=4d4cfc287e9d887b81242c0e431b5f49f9cec5c1
ARG OH_MY_ZSH_SHA256=d3af1bf5f3c59e84f1f3051d3a994b557b8ffd3cbdf64b2811ac532c38f6b5ee
ARG ZSH_AUTOSUGGESTIONS_COMMIT=85919cd1ffa7d2d5412f6d3fe437ebdbeeec4fc5
ARG ZSH_AUTOSUGGESTIONS_SHA256=20665e469e720e674964a66854d5534b9af78cdfd438512746ecd1ade8e819ce
ARG ZSH_COMPLETIONS_COMMIT=b5e8e0a22deb05a807bb8655aef809a29f3cffed
ARG ZSH_COMPLETIONS_SHA256=2a7a140e2698f39fb215a0ea86694c8c59ad51c97133cf3f2709c46dfe28bf34
ARG ZSH_HISTORY_SUBSTRING_SEARCH_COMMIT=a0bdb0d47dbaba31dba2db7af8c48a5d9c74049a
ARG ZSH_HISTORY_SUBSTRING_SEARCH_SHA256=b9edc11a4a04d2a5cc23bb7bafd8f02935240a45efe8a2fa61605b7636ecb22f

# Setup zsh for the root user
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN echo "ZSH" && \
    # Install zsh and the fonts required by the powerline prompt
    export DEBIAN_FRONTEND=noninteractive DEBCONF_NOWARNINGS="yes" && \
    apt-get update && apt-get install --no-install-recommends --assume-yes \
    zsh fontconfig fonts-powerline && \
    # Install a zsh plugin from a pinned commit, verifying its checksum
    install_zsh_plugin() { \
      name="$1"; commit="$2"; checksum="$3"; \
      curl -fsSL --retry 5 -o plugin.tar.gz -L "https://github.com/zsh-users/${name}/archive/${commit}.tar.gz" && \
      echo "${checksum}  plugin.tar.gz" | sha256sum -c --status || { \
        echo "${name} checksum verification FAILED"; exit 1; \
      } && \
      mkdir -p "/root/.oh-my-zsh/custom/plugins/${name}" && \
      tar -xzf plugin.tar.gz -C "/root/.oh-my-zsh/custom/plugins/${name}" --strip-components=1 && \
      rm -f plugin.tar.gz; \
    } && \
    # Install Oh My Zsh
    curl -fsSL --retry 5 -o omz.tar.gz -L "https://github.com/ohmyzsh/ohmyzsh/archive/${OH_MY_ZSH_COMMIT}.tar.gz" && \
    echo "${OH_MY_ZSH_SHA256}  omz.tar.gz" | sha256sum -c --status || { \
      echo "oh-my-zsh checksum verification FAILED"; exit 1; \
    } && \
    tar -xzf omz.tar.gz && \
    mv "ohmyzsh-${OH_MY_ZSH_COMMIT}" /root/.oh-my-zsh && \
    rm -f omz.tar.gz && \
    cp /root/.oh-my-zsh/templates/zshrc.zsh-template /root/.zshrc && \
	# Add Zsh plugins
    install_zsh_plugin zsh-autosuggestions "${ZSH_AUTOSUGGESTIONS_COMMIT}" "${ZSH_AUTOSUGGESTIONS_SHA256}" && \
    install_zsh_plugin zsh-completions "${ZSH_COMPLETIONS_COMMIT}" "${ZSH_COMPLETIONS_SHA256}" && \
    install_zsh_plugin zsh-history-substring-search "${ZSH_HISTORY_SUBSTRING_SEARCH_COMMIT}" "${ZSH_HISTORY_SUBSTRING_SEARCH_SHA256}" && \
    # Update .zshrc
    sed -i \
      -e 's/^plugins=(git.*)/plugins=(git zsh-completions zsh-autosuggestions zsh-history-substring-search)/' \
      -e 's/^ZSH_THEME="[^"]*"/ZSH_THEME="robbyrussell"/' \
    /root/.zshrc && \
    # Disable the update check, which prompts on every new shell
    echo "zstyle ':omz:update' mode disabled" >> /root/.zshrc && \
	# Cleanup
	apt-get autoremove -y && \
    apt-get clean autoclean && \
    rm -rf /var/lib/apt/lists/* /var/tmp/*

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
    rm -rf /var/lib/apt/lists/* /var/tmp/*

# +-----------------------------+
# | Set Locale + Timezone       |
# +-----------------------------+

# Default working directory.
# This is a fallback for plain "docker run" usage, overridden by devcontainer.
WORKDIR /workspace

ENV TZ=Europe/Berlin

RUN echo "Locale & Timezone" && \
    export DEBIAN_FRONTEND=noninteractive DEBCONF_NOWARNINGS="yes" && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata && \
    sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    sed -i -e 's/# de_DE.UTF-8 UTF-8/de_DE.UTF-8 UTF-8/' /etc/locale.gen && \
    # generate the enabled locales, then set the default
    dpkg-reconfigure -f noninteractive locales && \
    update-locale LANG=de_DE.UTF-8 && \
    echo "export PATH=$PATH" > /etc/environment

# Note: LC_ALL is deliberately not set here. It would override the locale a user
# picks inside the container. LANG/LANGUAGE provide the default.
ENV LANG=de_DE.UTF-8 LANGUAGE=de_DE:de

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

# setup tlmgr for user
USER $USERNAME
RUN tlmgr init-usertree && \
    # pre-generate the font names database
    luaotfload-tool --update --force
