# godot-gdnative-template
Actually usable GDNative template for Godot and C++

(Based on the Godot wiki guide)

## Features
- Auto-generates binding files for GDNative classes (.gdns)
- Auto-generates the godot entry point and library (entry.cpp and .gdnlib)
- Configurable through a config.toml file.

## Usage
1. Replace the `game` directory with your godot project and add a `bin` directory to it (names are configurable).
2. Write your class names in the config file `game.bindings` section.
3. Build the `godot-cpp` files:
    ```bash
    cd godot-cpp
    scons platform=<platform> generate_bindings=yes -j$(nproc)
    cd ..
    ```
4. Build the project:
    ```bash
    scons platform=<platform> -j$(nproc)
    ```
5. Click the play button on Godot.
6. Enjoy!