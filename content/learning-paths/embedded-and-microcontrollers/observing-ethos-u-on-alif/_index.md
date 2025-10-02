---
title: Observing Ethos-U85 on an Alif E8 Board, Built on Arm

minutes_to_complete: 120

who_is_this_for: This is an introductory topic for developers and data scientists new to Tiny Machine Learning (TinyML), who want to observe ExecuTorch performance on a physical device.

learning_objectives:
    - Identify suitable physical Arm-based devices for TinyML applications.
    - Optionally, configure physical embedded devices.
    - Deploy a TinyML ExecuTorch model to Alif's Ensemble E8 Series board.
    - Observe model execution on a display connected to the E8 board.

prerequisites:
    - Purchase of an Alif [Ensemble E8 Series Development Kit](https://alifsemi.com/ensemble-e8-series/) (you will need to [contact Alif Sales](https://alifsemi.com/support/sales-support/) directly).
    - x2 USB Type-C cables.
    - Basic knowledge of Machine Learning concepts.
    - A computer running Windows, Linux or macOS.

author: Waheed Brown

### Tags
skilllevels: Introductory
subjects: ML
armips:
    - Cortex-A
    - Cortex-M
    - Ethos-U

operatingsystems:
    - Linux
    - macOS
    - Windows

tools_software_languages:
    - Baremetal
    - Python
    - PyTorch
    - ExecuTorch
    - Arm Compute Library
    - GCC

further_reading:
    - resource:
        title: Visualize Ethos-U NPU performance with ExecuTorch on Arm FVPs
        link: /learning-paths/embedded-and-microcontrollers/visualizing-ethos-u-performance/
        type: blog
    - resource:
        title: Arm Machine Learning Resources
        link: https://www.arm.com/developer-hub/embedded-and-microcontrollers/ml-solutions/getting-started
        type: documentation
    - resource:
        title: Arm Developers Guide for Cortex-M Processors and Ethos-U NPU
        link: https://developer.arm.com/documentation/109267/0101
        type: documentation




### FIXED, DO NOT MODIFY
# ================================================================================
weight: 1                       # _index.md always has weight of 1 to order correctly
layout: "learningpathall"       # All files under learning paths have this same wrapper
learning_path_main_page: "yes"  # This should be surfaced when looking for related content. Only set for _index.md of learning path content.
---