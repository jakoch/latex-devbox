# Latex DevBox [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/jakoch/latex-devbox/release.yml?branch=main&style=flat&logo=github&label=Image%20published%20on%20GHCR)](https://github.com/jakoch/latex-devbox)

A Docker development box for documentation creation using LaTeX.

**Debian Linux 13 Trixie with LaTeX (TexLive + TexLS), Pandoc, AsciiDoc, Docbook.**

## What is this?

This repository maintains a Dockerfile for generating a container image based on Debian Linux.

The image includes TexLive on Debian 13 Trixie, along with other essential documentation tools like Pandoc, AsciiDoc, and Docbook.

The image is published to the Github Container Registry (GHCR).

The purpose of the image is to set up a comprehensive LaTeX development environment within Visual Studio Code using a [devcontainer config](https://github.com/jakoch/latex-devbox#fetching-the-prebuild-container-images-using-a-devcontainer-config). This setup allows for seamless LaTeX document creation, editing, and compilation directly within a Docker container, ensuring a consistent and reproducible environment.

## Prerequisites

You need the following things to run this:

- Docker
- Visual Studio Code

## How to run this?

There are two ways of setting the container up.

Either by building the container image locally or by fetching the prebuild container image from the Github container registry.

### Building the Container Image locally using VSCode

- **Step 1.** Get the source: clone this repository using git or download the zip

- **Step 2.** In VSCode open the folder in a container (`Remote Containers: Open Folder in Container`):

   This will build the container image (`Starting Dev Container (show log): Building image..`)

   Which takes a while...

   Then, finally...

- **Step 3.**  Enjoy! :sunglasses:

### Fetching the prebuild container images using Docker

This container image is published to the Github Container Registry (GHCR).

You may find the package here: https://github.com/jakoch/latex-devbox/pkgs/container/latex-devbox

**Command Line**

You can install the container image from the command line:

```bash
docker pull ghcr.io/jakoch/latex-devbox:trixie-latest
```
**Dockerfile**

You might also use this container image as a base image in your own `Dockerfile`:

```bash
FROM ghcr.io/jakoch/latex-devbox:trixie-latest
```
### Fetching the prebuild container images using a .devcontainer config

**Devcontainer.json**

You might use this container image in the `.devcontainer/devcontainer.json` file of your project:


```
{
    "name": "LaTeX Dev Container",
    "image": "ghcr.io/jakoch/latex-devbox:trixie-latest"
}
```
