---
# User change
title: "Create Build Environment"

weight: 6 # 1 is first, 2 is second, etc.

# Do not modify these elements
layout: "learningpathall"
---

## Create a Linux Container

1. Install and start [Docker Desktop](https://www.docker.com/)

2. Create a RTOS project directory:

   ```bash
   mkdir alif-rtos-starter
   ```

3. Create a `dockerfile` in the `alif-rtos-starter` directory:

   ```bash
   cd alif-rtos-starter
   touch dockerfile
   ```

4. Open the `dockerfile`:
   ```bash
   nano dockerfile
   ```

   and add the following commands:

   ```dockerfile
   FROM arm64v8/ubuntu:24.04

   # Basics
   FROM arm64v8/ubuntu:24.04

   # Basics
   RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
       build-essential cmake ninja-build git python3 python3-pip \
       wget unzip curl ca-certificates pkg-config && \
       rm -rf /var/lib/apt/lists/*

   # Arm GNU toolchain for the Alif Ensemble E8 Cortex-M
   RUN apt-get update
   RUN apt-get install -y --no-install-recommends gcc-arm-none-eabi

   # cmsis-toolbox (optional, handy for packs/csolution projects)
   RUN wget -q https://github.com/Open-CMSIS-Pack/cmsis-toolbox/releases/download/2.11.0/cmsis-toolbox-linux-arm64.tar.gz \
       && tar -xzf cmsis-toolbox-linux-arm64.tar.gz -C /opt \
       && ln -s /opt/cmsis-toolbox/csolution /usr/local/bin/csolution

   WORKDIR /work
   ```

   {{% notice Arm GNU Toolchain Compatibility %}}

   ###### The Alif Ensemble E8 needs a very specific [Arm GNU Toolchain](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain):
   * The Micro-Controller Unit (MCU) you are targeting is Cortex-M55
   * The Cortex-M55’s Arm architurecture version is [Armv8.1-M](https://developer.arm.com/documentation/107656/0101/Introduction-to-Armv8-M-architecture)
   * This is a 32-bit instruction set (AArch32) for bare-metal deployment

   ###### You need to compile software especially for the Cortex-M55 build target:
   * You need to cross-compile any code that you want to run on the Ensemble E8 board, to the AArch32, bare-metal Cortex-M55 target
   * Since your build container is AArch64 Linux, you will need a cross-compiler that runs on "AArch64 Linux", building for an "AArch32 bare-metal" target
   * The specific [Arm GNU Toolchain Download](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) you want is:
     * Dev environment: "aarch64 Linux hosted cross toolchains"
     * Build target: "AArch32 bare-metal target (arm-none-eabi)"
     * Download file: [arm-gnu-toolchain-14.3.rel1-aarch64-arm-none-eabi.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu/14.3.rel1/binrel/arm-gnu-toolchain-14.3.rel1-aarch64-arm-none-eabi.tar.xz)
	
   ###### Just use apt-get in your dockerfile
   * It's far easier to just install the correct toolchain in your dev container using `apt-get install`, like in the above `dockerfile`:
   ```bash
   apt-get install -y --no-install-recommends gcc-arm-none-eabi
   ```

   {{% /notice %}}

5. Create the `alif-rtos-starter` container:

   ```bash
   docker build -t alif-rtos-starter .
   ```

6. Run the `alif-rtos-starter`:

   ```bash { output_lines = "2-3" }
   docker run --rm -it -v "$PWD:/work" alif-rtos-starter bash
   # Output will be the Docker container prompt
   root@<CONTAINER ID>:/work#
   ```

   [OPTIONAL] If you already have an existing container:
   - Get the existing CONTAINER ID:
     ```bash { output_lines = "2-4" }
     docker ps -a
     # Output
     CONTAINER ID  IMAGE                    COMMAND      CREATED        STATUS                       PORTS  NAMES
     0123456789ab  alif-rtos-starter  "/bin/bash"  27 hours ago   Exited (255) 59 minutes ago.        container_name
     ```
   - Log in to the existing container:
     ```bash
     docker start 0123456789ab
     docker exec --rm -it -v "$PWD:/work" alif-rtos-starter bash
     ```

## Make an Arm Toolchain Build File (CMake)

1. From inside the `alif-rtos-starter` container