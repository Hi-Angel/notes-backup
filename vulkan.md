# Incomplete steps for rendering with Vulkan

0. make window, e.g. with glfw
1. make Vulkan *(TODO here)* instance, initialize struct (in particular the sType)
2. (optional) Enable validation layers. Basically, check for the string in the layers list.
3. Select physical device, maybe check for its properties to see if it's suitable for us.
4. Choose the queue type we want.
5. Create a queue
6. Create a logical VK device
7. Use WSI to connect Vulkan and window. Make sure `VK_KHR_surface` is enabled *(part of the list of `glfwGetRequiredInstanceExtensions` though)*.
8. Create a window surface. As a note: Vulkan does not require creating one, and you can just do an off-screen rendering *(whereas with OpenGL that requires a hack with invisible window)*.
9. Check that device supports presenting to the surface *(it may not)*
10. Create the presentation queue
11. Create swap chain queue *(check for support)*. The right settings has to be chosen.
12. We skipped some related to the swap chain.
13. Create image views.
*. destroy instance and window

# Misc

* `VkRenderPass`: basically tells GPU about how the image data will be used, so e.g. is it compute or graphics, what to do or not to with the loaded memory, etc. It will be part of FBs information.
* `Image view`: a projection of an image that allows for different formats, types, etc.
* `Command buffer`: where the instructions will be placed. Various Vulkan functions take these as an arg.
* `Logical device`: is a Vulkan concept allowing to have multiple rendering instances in a single process that are completely unaware of each other.
* `fixed` vs `programmable` functions: `fixed` is a graphics-processing stage where there's pre-set number of actions you can apply to a primitive. You can parametrize them, but not much besides. "Programmable" OTOH gives flexibility in what you can do, typically by using shaders.
* `Frame buffer` *(FB)*: a collection of buffers which store the rendering result *(yes, sounds weird but the singular form of the term refers to a collection)*. Buffers of "default FB" are usually what gets displayed on the screen.
* `Swap chain`:
* `Swap chain`: basically series of virtual FBs, so e.g. two of them is "double-buffering", etc. OpenGL has no support for swap chain. Requires `VK_KHR_swapchain` extension. Vulkan doesn't have a "default framebuffer", so the chain has to be created explicitly.
* `SPIR-V` is a bytecode shaders presentation, which supposed to resolve ambiguity of GLSL textual presentation.
* `Shader`: a program that gets executed on each of its thousands of inputs by getting parallelized on GPU. E.g. a vertex shader gets executed over each vertex.
* `Primitive`: usually a "base primitives", i.e. a point, line, triangle, patches *(for tesselation)*. Not to be confused with "full primitive assembly" that is fragments. A `patch` is a group of vertices to be tesselated and in turn may be "isolines", "triangles", "quads".
* `Input assembler`: groups vertices into geometric primitives.
* `Vertex shaders`: mainly transforms vertex positions from model space to screen space; but may also do unrelated calculations before passing further down the pipeline, such determining the color. Inputs are called "vertex attributes", may be arrays, matrices, doubles, or combination thereof. Usually invariant to its inputs, which allows to cache the output.
* `Mesh`: like a model but only geometric structure described. May include multiple VBOs, an IBO, etc.

    After VS the vertices are grouped into primitives to be fed to further shaders.
* `Tesselation shader`: accepts "patches" and can generate new vertices. has adjacency information and can be used to e.g. smooth-out surfaces.
* `Geometry shader` input and output are geometric primitives, not necessarily same ones. E.g. may take points but produce triangles. If number of output vertices not enough to make a primitive *(e.g. 2 for a triangle)*, in Vulkan they're dropped. GS kind of may be used for limited tesselation.
* `Rasterizer`: breaks the geometry input such triangles into "fragments" before passing through to FG. It also clips fragments that are off screen, and may run depth-testing to remove those hidden by other fragments.
* `Fragment`: a data necessary to generate a pixel in the FB, examples of which: depth, raster position, stencil, alpha, interpolated attributes…
* `Fragment shaders`: also called `pixel shaders` because "fragments" are the inputs and "pixels" are the outputs. Output the fragment color.
* `stencil buffer`: allows "to stencil" the rendering. An example: you're making racing game and you have a rear-view mirror of what's behind. Stencil buffer aside, you'd render the scene behind and clip the image to the size of the mirror. Now, having stencil buffer simplifies that as it allows you to create a "mask" to masks out bits during the rendering stage, so you simply render the scene shaped to the mirror. It's also an optimization as GPU has less to render.
* `uniform`: a constant shader variable that is typically initialized from the outside the shader, may also be set to an explicit value within the shader when declared. Can't be changed by shader.
* `Transform feedback`: an optional cache of the output of vertex processing *(whereas vertex processing ends with the last shader before fragment one)*

## GLSL

* `VAO`: a collection of `VBO`s
* `layout (location = 0) …`: an index choosing a VBO from VAO. A `gl_VertexID` may be preferred.
* `gl_VertexIndex` and `gl_VertexID`: *(1st is for Vulkan, 2nd is for OpenGL)* the Vertex the VS is running over.

# Debugging

* `RenderDoc` is an open source app that allows to step through various shader stages.
