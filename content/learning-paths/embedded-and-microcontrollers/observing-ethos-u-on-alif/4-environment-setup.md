---
# User change
title: "Enviroment Setup"

weight: 5 # 1 is first, 2 is second, etc.

# Do not modify these elements
layout: "learningpathall"
---

For detailed instructions on setting up your ExecuTorch build environment, please see the official PyTorch documentation: [Environment Setup](https://docs.pytorch.org/executorch/stable/using-executorch-building-from-source.html#environment-setup)

{{% notice macOS %}}

Use a Docker container as your development environment:
* The [Arm GNU Toolchain](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain) currently does not have a "AArch64 GNU/Linux target" for macOS

1. Install and start [Docker Desktop](https://www.docker.com/)

2. Create a directory for building a `ubuntu-24-container`:

   ```bash
   mkdir ubuntu-24-container
   ```

3. Create a `dockerfile` in the `ubuntu-24-container` directory:

   ```bash
   cd ubuntu-24-container
   touch dockerfile
   ```

4. Add the following commands to your `dockerfile`:

   ```dockerfile
   FROM ubuntu:24.04

   # Basic build deps
   RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
       build-essential cmake ninja-build git python3 python3-pip \
       wget unzip curl ca-certificates pkg-config \
       && rm -rf /var/lib/apt/lists/*

   # Arm GNU toolchain (GCC) – change version if you prefer
   RUN curl -L -o /tmp/gcc.tar.xz https://developer.arm.com/-/media/Files/downloads/gnu/13.3.rel1/binrel/arm-gnu-toolchain-13.3.rel1-x86_64-arm-none-eabi.tar.xz \
       && mkdir -p /opt/arm-gnu && tar -xf /tmp/gcc.tar.xz -C /opt/arm-gnu --strip-components=1 \
       && ln -s /opt/arm-gnu/bin/arm-none-eabi-gcc /usr/local/bin/arm-none-eabi-gcc

   # cmsis-toolbox (for packs/csolution-style builds used across Alif samples)
   RUN wget -q https://github.com/Open-CMSIS-Pack/cmsis-toolbox/releases/download/2.4.0/cmsis-toolbox-linux-amd64.tar.gz \
      && tar -xzf cmsis-toolbox-linux-amd64.tar.gz -C /opt && ln -s /opt/cmsis-toolbox/csolution /usr/local/bin/csolution

   # Get Alif SDK meta and sample links
   WORKDIR /work
   ```

   The `ubuntu:24.04` container image includes Python 3.12, which will be used for this learning path.

5. Create the `ubuntu-24-container`:

   ```bash
   docker build -t ubuntu-24-container .
   ```

6. Run the `ubuntu-24-container`:

   ```bash { output_lines = "2-3" }
   docker run --rm -it -v "$PWD:/work" ubuntu-24-container bash
   # Output will be the Docker container prompt
   root@<CONTAINER ID>:/work#
   ```

   [OPTIONAL] If you already have an existing container:
   - Get the existing CONTAINER ID:
     ```bash { output_lines = "2-4" }
     docker ps -a
     # Output
     CONTAINER ID  IMAGE                    COMMAND      CREATED        STATUS                       PORTS  NAMES
     0123456789ab  ubuntu-24-container  "/bin/bash"  27 hours ago   Exited (255) 59 minutes ago.        container_name
     ```
   - Log in to the existing container:
     ```bash
     docker start 0123456789ab
     docker exec --rm -it -v "$PWD:/work" ubuntu-24-container bash
     ```

{{% /notice %}}

1. **Install ExecuTorch dependencies:**

   ```bash { output_lines = "1" }
   # Use "sudo apt ..." if you are not logged in as root
   apt update
   apt install -y \
     python-is-python3 python3.12-dev python3.12-venv \
     gcc g++ \
     make cmake \
     build-essential \
     ninja-build \
     libboost-all-dev
   ```

2. Clone ExecuTorch:
   ```bash
   git clone --depth 1 https://github.com/pytorch/executorch.git
   ```

3. Create a Virtual Environment:
   ```bash { output_lines = "4" }
   cd executorch
   python3 -m venv .venv
   source .venv/bin/activate
   # Your prompt will prefix with (.venv)
   ```

4. Configure your git username and email globally:
   ```bash
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   ```