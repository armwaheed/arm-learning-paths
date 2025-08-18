---
# # User change
# title: "Build the ExecuTorch Runtime Files"

# weight: 7 # 1 is first, 2 is second, etc.

# # Do not modify these elements
# layout: "learningpathall"
---

Embedded systems like the NXP board require two ExecuTorch runtime components: a `.pte` file and an `exeuctor_runner` file.

**ExecuTorch Runtime Files for Embedded Systems**
|Component|Role in Deployment|What It Contains|Why It’s Required|
|---------|------------------|----------------|-----------------|
|**`.pte file`**  (e.g., `mv2_arm_delegate_ethos-u55-256.pte`)|The model itself, exported from ExecuTorch|Serialized and quantized operator graph + weights + metadata|Provides the neural network to be executed|
|**`executor_runner`**  (binary [ELF](https://www.netbsd.org/docs/elf.html) file)|The runtime program that runs the .pte file|C++ application that loads the .pte, prepares buffers, and calls the NPU or CPU backend|Provides the execution engine and hardware access logic|

<style>
.ascii-diagram {
  font-size: 12px; /* Or smaller, like 10px */
  line-height: 1.2;
  font-family: monospace;
  white-space: pre-wrap;
  overflow-x: auto;
}
</style>
<center>
<br>
<pre class="ascii-diagram">
┌───────────────────────────────────────────────────┐
│                                                   │
│                Host Development                   │
│         (e.g., Linux or macOS+Docker)             │
│                                                   │
│  [Model export / compilation with ExecuTorch]     │
│                                                   │
│     ┌───────────────────┐        ┌───────────┐    │
│     │                   │        │           │    │
│     │  executor_runner  │        │  .pte     │    │
│     │  (ELF binary)     │        │ (model)   │    │
│     │                   │        │           │    │
│     └───────────┬───────┘        └─────┬─────┘    │
│                 │                      │          │
└─────────────────┼──────────────────────┼──────────┘
       │ SCP/serial transfer  │
       │                      │
       ▼                      ▼
┌───────────────────────────────────────────────────┐
│                                                   │
│            NXP i.MX93 Embedded Board              │
│                                                   │
│                                                   │
│  ┌───────────────────────────────────────────┐    │
│  │   executor_runner (runtime binary)        │    │
│  │                                           │    │
│  │    ┌───────────────────────────────┐      │    │
│  │    │ Load .pte (model)             │      │    │
│  │    └───────────────┬───────────────┘      │    │
│  │                    │                      │    │
│  │                    ▼                      │    │
│  │    ┌───────────────────────────────┐      │    │
│  │    │ Initialize hardware (CPU/NPU) │      │    │
│  │    └───────────────┬───────────────┘      │    │
│  │                    │                      │    │
│  │                    ▼                      │    │
│  │    ┌───────────────────────────────┐      │    │
│  │    │ Perform inference             │      │    │
│  │    └───────────────┬───────────────┘      │    │
│  │                    │                      │    │
│  │                    ▼                      │    │
│  │    ┌───────────────────────────────┐      │    │
│  │    │ Output results                │      │    │
│  │    └───────────────────────────────┘      │    │
│  └───────────────────────────────────────────┘    │
│                                                   │
└───────────────────────────────────────────────────┘
</pre>
<i>ExecuTorch runtime deployment to an embedded system</i>
</center>

## Accept the Arm End User License Agreement

```bash
export ARM_FVP_INSTALL_I_AGREE_TO_THE_CONTAINED_EULA=True
```

## Set Up the Arm Build Environment

This example builds the [MobileNet V2](https://pytorch.org/hub/pytorch_vision_mobilenet_v2/) computer vision model. The model is a convolutional neural network (CNN) that extracts visual features from an image. It is used for image classification and object detection. The actual Python code for the MobileNet V2 model is in the `executorch` repo: [executorch/examples/models/mobilenet_v2/model.py](https://github.com/pytorch/executorch/blob/main/examples/models/mobilenet_v2/model.py).

You can read a detail explanation of the build steps here: [ARM Ethos-U Backend](https://docs.pytorch.org/executorch/stable/backends-arm-ethos-u.html).

1. Run the steps to set up the build environment: 
    
   ```bash
   ./examples/arm/setup.sh \
     --target-toolchain aarch64-none-linux-gnu
   ```
  
2. Update your `PATH` with the `aarch64-none-linux-gnu` toolchain:
   ```bash
   source examples/arm/ethos-u-scratch/setup_path.sh
   ```

## Build the ExecuTorch Runtime Files
Now you will build the `.pte` and `executor_runner` files, that will be used on the NXP board.

{{% notice Note %}}

The `ethos-u65-256` target is not yet available in the [list of targets](https://github.com/pytorch/executorch/blob/c778063c984a19d3b55b65ce23522d1eb463ae4c/examples/arm/aot_arm_compiler.py#L334-L347). The `ethos-u55-256` is close enough, for learning how to deploy ML models to the NXP board. Both targets have 256 Multiply-Accumulate Units (MACs).

{{% /notice %}}

1. Build the [MobileNet V2](https://pytorch.org/hub/pytorch_vision_mobilenet_v2/) ExecuTorch runtime files using [run.sh](https://github.com/pytorch/executorch/blob/main/examples/arm/run.sh):

   ```bash
   ./examples/arm/run.sh \
     --aot_arm_compiler_flags="--quantize --intermediates mv2_cortexa/ --debug" \
     --toolchain=aarch64-none-linux-gnu-gcc \
     --output=mv2_cortexa \
     --model_name=mv2 \
     --no_delegate \
     --extra_build_flags="-DEXECUTORCH_ENABLE_EVENT_TRACER=OFF -DCMAKE_TOOLCHAIN_FILE=examples/arm/executor_runner_aarch64_linux_gnu/aarch64-none-linux-gnu-gcc.cmake"
   ```

2. The two output files you need are `mv2_arm_delegate_ethos-u55-256.pte` and `arm_executor_runner`, which will show in the `run.sh` output:

   {{% notice Note %}}

   In the below sample outputs, the executorch directory path is indicated as /path/to/executorch. Your actual path will depend on where you cloned your local copy of the [executorch repo](https://github.com/pytorch/executorch/).

   {{% /notice %}}

   ```bash { output_lines="1-6" }
   ...
   PTE file saved as /path/to/executorch/mv2_u55/mv2_arm_delegate_ethos-u55-256.pte
   ...
   [100%] Built target arm_executor_runner
   # Saved to /path/to/executorch/mv2_u55/mv2_arm_delegate_ethos-u55-256/cmake-out/arm_executor_runner
   ...
   # The remaining output for running on a FVP is not required for this learning path
   ```
## Troubleshooting
**`setup.sh`**
- If you see the following error in the `setup.sh` output:
  ```bash { output_lines = "1-2" }
  Failed to build tosa-tools-v0.80
  ERROR: Could not build wheels for tosa-tools-v0.80, which is required to install pyproject.toml-based projects
  ```
  then:
  1. Increase the swap space to 8 GB:
     ```bash
     fallocate -l 8G /swapfile
     chmod 600 /swapfile
     mkswap /swapfile
     swapon /swapfile
     ```
     - [optional] Deallocate the swap space after you complete this learning path:
        ```bash
        swapoff /swapfile
        rm /swapfile
        ```

  {{% notice macOS %}}
  Increase the "Memory Limit" in Docker settings to 12 GB: 
  ![Increase the "Memory Limit" in Docker settings to 12 GB alt-text#center](./Increase%20the%20Memory%20Limit%20to%2012%20GB.jpg "Increase the Memory Limit in Docker settings to 12 GB")

  {{% /notice %}}

  2. Re-run `setup.sh`
     ```bash
     ./examples/arm/setup.sh --i-agree-to-the-contained-eula
     ```

- If you see the following error in the `setup.sh` output:
  ```bash { output_lines = "1-2" }
  Failed to build tosa-tools
  ERROR: Failed to build installable wheels for some pyproject.toml based projects (tosa-tools)
  ```
  then do the below troubleshooting steps.
   1. Install any missing build tools:
      ```bash
      apt update && apt install -y \
         cmake \
         build-essential \
         ninja-build \
         python3-dev \
         libboost-all-dev
      ```
   2. Re-run `setup.sh`
      ```bash
      ./examples/arm/setup.sh --i-agree-to-the-contained-eula
      ```
- If you see the following error in the `setup.sh` output:
   ```bash { output_lines = "1-8" }
   ...
   ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
   tosa-tools 0.80.2.dev1+g70ed0b4 requires jsonschema, which is not installed.
   tosa-tools 0.80.2.dev1+g70ed0b4 requires flatbuffers==23.5.26, but you have flatbuffers 24.12.23 which is incompatible.
   tosa-tools 0.80.2.dev1+g70ed0b4 requires numpy<2, but you have numpy 2.3.1 which is incompatible.
   ...
   ```
   then just re-run `setup.sh`
   ```bash
   ./examples/arm/setup.sh --i-agree-to-the-contained-eula

**`run.sh`**
- If you see the following error in the `run.sh` output:
  ```bash { output_lines = "1" }
  ...
  Missing /root/executorch/examples/arm/ethos-u-scratch/setup_path.sh. please refer to /root/executorch/examples/arm/setup.sh to properly install necessary tools.
  ```
  then re-run `setup.sh`
  ```bash
  ./examples/arm/setup.sh --i-agree-to-the-contained-eula
  ```