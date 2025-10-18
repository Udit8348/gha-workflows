# Apptainer GitHub Actions Demo

## Overview
This is a demo github actions repo used to experiment with Apptainer based workflows.

## Background
The basic Apptainer architecture uses a `.def` file to describe the contents of the container and how to build it. This is very similar to a Dockerfile, and a `.def` file can be generated from one too. This generates a `.sif` which is self-contained container image. Unlike docker that stores images in immutable / mutable layers everything is included in a singularity image file. This is most practical in HPC environments where you might be moving from node to node and need to keep everything packaged together. There is a slight size overhead, but nothing impractical.

<!-- ## Apptainer for Github Actions Approaches
There are two main approaches for to enable Apptainer in GitHub Actions. -->

### Workflow Comparison: PPA Install vs. Marketplace Action

This repository includes two viable GitHub Actions workflows that automate building and running Apptainer containers.  
Both achieve the same end goal — building a `.sif` image from a `.def` file — but differ in how Apptainer is installed and how much system overhead each approach incurs.

| Aspect | **PPA Installation** | **Marketplace Action (`setup-apptainer`)** |
|:-------|:----------------------|:-------------------------------------------|
| **Installation method** | Adds the official `ppa:apptainer/ppa` repository and installs via `apt`. | Uses the [eWaterCycle/setup-apptainer](https://github.com/eWaterCycle/setup-apptainer) GitHub Action to download a pre-built Apptainer binary (`.deb`) directly from the official GitHub releases. |
| **Privileges required** | Requires `sudo` privileges to install system packages. | Runs fully in user space; no `sudo` access or repository configuration needed. |
| **Version control** | Controlled by the version available in the Ubuntu PPA (may lag behind latest). | Explicitly specified via the `apptainer-version` input (e.g., `1.3.6`). Find them here: [Apptainer Releases](https://github.com/apptainer/apptainer/releases) |
| **Integration with system** | Installs Apptainer and dependencies into `/usr/bin` using the system package manager. | Adds a user-local binary to the GitHub Actions `$PATH` without touching system packages. |
| **Performance and setup time** | Slightly slower because it updates APT sources, installs dependencies, and runs as root. | Faster — downloads and unpacks a prebuilt binary; avoids apt updates entirely. |
| **Reproducibility** | Matches production or HPC environments that also use the Ubuntu PPA packages. | More isolated and portable; ideal for lightweight CI pipelines. |
| **Failure modes** | Can fail if the PPA is down or network access to `apt` is restricted. | Can fail if GitHub releases for the specified version are unavailable. |
| **Recommended use** | Use when testing full system integration or mirroring HPC cluster setups. | Use when prioritizing speed, reproducibility, or running without elevated privileges. |

In summary, both workflows build and run the same container image, but the **PPA-based workflow** mirrors a realistic Ubuntu installation environment, while the **`setup-apptainer`** action provides a lightweight, fast-install alternative for CI testing and reproducible builds.

## Refinements
In Vortex Repo there are some [misc scripts](https://github.com/vortexgpgpu/vortex/tree/master/miscs/apptainer) that are hard dependencies for building the .`def` files. If possible these should be streamlined either into one script. Or placed directly in the `.def` or Vortex's [`ci/install_dependencies.sh`](https://github.com/vortexgpgpu/vortex/blob/master/ci/install_dependencies.sh) file.

Recall, `install_dependencies.sh` is system dependencies and was previously installed in the Github Actions workflow, prior to building Vortex. Similarly, we need these dependencies met in the Apptainer.

Optimally, the misc scripts and `install_dependencies.sh` are merged and pruned for overlap.

### Pipelining Suggestions
If you later want to download these same artifacts in a different workflow (e.g., multi-job pipeline), you can use. Would you like me to extend one of these workflows into a two-job version (where one job uploads these dependencies and another downloads + builds)? That’s the standard way to isolate build staging from container creation.


## Results



### Third-Party Notice

This project uses the GitHub Action [eWaterCycle/setup-apptainer](https://github.com/eWaterCycle/setup-apptainer),
licensed under the Apache License 2.0.
Copyright © eWaterCycle contributors.