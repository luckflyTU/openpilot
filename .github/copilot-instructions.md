# Copilot Instructions for openpilot

Welcome, AI agent. This document will guide you in contributing to openpilot, an open-source driver assistance system. Understanding the architecture and conventions is key to being effective.

## Architecture Overview

openpilot is a complex system with components written in Python, C++, and Cython. The system is composed of multiple processes that communicate via the "cereal" messaging library.

- **`selfdrive/`**: The core of the self-driving functionality.
    - **`selfdrive/car/`**: Contains vehicle-specific interface code. When adding support for a new car, you'll be working here. Each car brand has its own folder (e.g., `toyota/`, `honda/`) with a `carcontroller.py` and `carstate.py`.
    - **`selfdrive/controls/`**: Implements the control loops for steering, acceleration, and braking. `controlsd.py` is the main entry point.
    - **`selfdrive/ui/`**: The user interface, written in Qt/C++. `selfdrive/ui/qt/onroad/hud.cc` is where the main on-road UI is rendered.
- **`cereal/`**: Defines the messaging schema using Cap'n Proto. All communication between processes happens through these message types. See `cereal/log.capnp` for many of the core data structures.
- **`panda/`**: Firmware for the panda OBD-II interface, which is the hardware link between the openpilot device and the car. It's written in C.
- **`common/`**: A collection of utility functions and classes used throughout the codebase. `common/params.py` is used for persistent parameter storage.

## Developer Workflows

### Building

The project uses SCons as its build system. Most Python code doesn't require a build step, but C++ and Cython components do.

- To build the entire project, run `scons -j$(nproc)`.
- Individual components can be built by specifying their path, e.g., `scons -j$(nproc) selfdrive/controls/controlsd`.

### Testing

Tests are written using `pytest`.

- Run all tests with `pytest`.
- Run tests for a specific component with `pytest selfdrive/car/tests/`.

### Debugging & Logging

- `selfdrive/swaglog.py` is the logging utility. Use `cloudlog.info()`, `cloudlog.warning()`, `cloudlog.error()` for logging.
- For debugging, you can replay routes using the tools in `tools/replay/`.

## Project Conventions

- **Messaging**: All inter-process communication must use the `cereal` messaging library. When adding new data to be passed between processes, you'll likely need to update a `.capnp` file and re-run SCons.
- **Car Interfaces**: When adding or modifying car support, follow the existing structure in `selfdrive/car/`. The `CarInterfaceBase` class in `opendbc/car/interfaces.py` defines the required methods.
- **State Management**: The `selfdrive.state` module contains important state information that is shared across processes.
- **Persistent Parameters**: Use the `Params` class from `common.params` to store and retrieve configuration parameters that should persist across reboots. For example, `Params().put("MyNewParam", "1")` and `Params().get_bool("MyNewParam")`.
- **UI Code**: The UI is in C++/Qt. When modifying the UI, you will be editing files under `selfdrive/ui/qt`.

## Coding Standards

- **C/C++ Declarations**: When adding global variables or functions in a `.cc` file, ensure they are properly declared in the corresponding `.h` file.
- **C/C++ Includes**: Ensure that any function or type used in a C++ file has its corresponding header `#include`d at the top of the file.
- **Python Imports**: Ensure that any function or module used in a Python file is properly imported using `import` or `from ... import` at the top of the file.

By following these guidelines, you'll be able to contribute effectively to the openpilot project.
