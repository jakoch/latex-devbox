# Latex DevBox [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/jakoch/latex-devbox/release.yml?branch=main&style=flat&logo=github&label=Image%20published%20on%20GHCR)](https://github.com/jakoch/latex-devbox)

A Docker development box for documentation creation using LaTeX.

**Debian Linux 13 Trixie with LaTeX (TexLive + TexLS), Pandoc, AsciiDoc, Docbook.**

## What is this?

This repository maintains a Dockerfile for generating a container image based on Debian Linux.

The image includes TexLive on Debian 13 Trixie, along with other essential documentation tools like Pandoc, AsciiDoc, and Docbook.

The image is published to the Github Container Registry (GHCR).

The purpose of the image is to set up a comprehensive LaTeX development environment within Visual Studio Code using a [devcontainer config](https://github.com/jakoch/latex-devbox#fetching-the-prebuild-container-images-using-a-devcontainer-config). This setup allows for seamless LaTeX document creation, editing, and compilation directly within a Docker container, ensuring a consistent and reproducible environment.
