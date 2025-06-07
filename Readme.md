# Welcome!

VulkanHelloWorld, as the name suggests is a minimal graphics application utilizing the Vulkan API
to draw a rotating, textured static mesh.

The project's purpose was purely for the sake of my own education about the API, and is almost entirely based on
[this](https://vulkan-tutorial.com/) Vulkan tutorial.

## Compiling the project

VulkanHelloWorld is written primarily in C++17.

The project is set up using CMake, and requires the Vulkan SDK to be installed in order to be built,
which can be downloaded from [the LunarG Website](https://vulkan.lunarg.com/).

With the SDK in hand, the project can be compiled just like any project made with CMake from a terminal:

```shell
cmake -S . -B ./build
cd ./build
cmake --build .
```

Then installed with:

```shell
cmake --install .
```

After which the application can be run from the install location.

The program has to be installed this way to be run, since the installation process places all assets in the correct
tree structure for the program to load it.

There might be a case in which the provided shader binaries aren't functioning properly, in which case they should be re-compiled
using glslc (which should come included with teh VulkanSDK) like so:

```shell
cd ./shaders
glslc shader.vert -o vert.spv
glslc shader.frag -o frag.spv

```
