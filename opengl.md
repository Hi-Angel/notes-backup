EGL + GBM, and a little of opengl https://blogs.igalia.com/elima/tag/egl/

`glGenTextures()`: The generated textures have no dimensionality; they assume the dimensionality of the texture target to which they are first bound (see glBindTexture).

# OpenGL object

Generic term talking about almost anything in OpenGL.

# renderbuffer

Holds a single image. Can't be worked on by a shader, unless you put it to a FB.

# Framebuffer

Needs to be generated, and attached to the context.

The target​ parameter for this object can take one of 3 values: GL_FRAMEBUFFER, GL_READ_FRAMEBUFFER, or GL_DRAW_FRAMEBUFFER. The last two allow you to bind an FBO so that reading commands (glReadPixels, etc) and writing commands (all rendering commands) can happen to two different framebuffers. The GL_FRAMEBUFFER binding target simply sets both the read and the write to the same FBO.

When an FBO is bound to a target, glReadBuffer selects which buffer of the framebuffer object to read and glDrawBuffer selects which buffer to write to. The available buffer names depends on whether the default framebuffer is bound or not. If the default framebuffer is bound the valid buffer names are GL_FRONT, GL_BACK, GL_AUXi, GL_ACCUM, and so forth. Otherwise the valid buffer names are GL_NONE or GL_COLOR_ATTACHMENTi.

Instead, FBOs have a different set of image names. Each FBO image represents an attachment point, a location in the FBO where an image can be attached.

# Attach
To connect one object to another. This term is used across all of OpenGL, but FBOs make the most use of the concept. Attachment is different from binding. Objects are bound to the context; objects are attached to one another

# textures

Texture can hold one or more images of *the same format*. Texture can be used both as render source and render target.

Target of a texture is not a context, but something like `GL_TEXTURE_2D`. Dunno if the target in its turn implicitly belongs to the context, but parameters of the target can be changed, e.g. `glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)`.

# image load

OpenGL doesn't have native ways of loading a texture from a format. Use libs.

## sdl2_image example code


    #include "SDL2/SDL_image.h"
    #include <cstdio>

    int main(int argc, char *argv[])
    {
        SDL_Surface* res_texture = IMG_Load("earth.png");
	    if (!res_texture) {
		    printf("IMG_Load: %s\n", SDL_GetError());
		    return 1;
	    }
        SDL_FreeSurface(res_texture);
        return 0;
    }

# glossary

uniform — a constant inside GLSL *(doesn't have to be a constant outside it tho)*.

U and V — X and Y acc. NIH syndrome probably. It's for real used, e.g. `color = texture( myTextureSampler, UV ).rgb;` in GLSL.

stride — single vertex size in bytes. E.g. if a vertex represented by uint32_t of x, y, z, the stride is 12.

# VBO

Stores vertices data. There are various ways of updating them, here's a useful link https://www.khronos.org/opengl/wiki/Buffer_Object_Streaming

In general, you call `glBufferData()` for allocating a buffer. It will either have the data copied from where you've pointed, or just uninitialized bytes *(if you pass a null)*. Then parts of the buffer can be updated through `glBufferSubData()`. Alternative way of updating is to use multiple buffer objects, constantly (un)mapping them.

Yet another alternative is `glMapBuffer()` to map the buffer to a usual address space, thus to read/write it directly.

# VAO

Stores vertex attribute configuration, a bound VBO and EBO. The data is drawed by `glDrawArrays()`

To use a VAO you have to bind it using `glBindVertexArray`. Next bind/configure the corresponding VBO(s) and attribute pointer(s), and unbind the VAO for later use. For drawing an object, simply bind the VAO with the prefered settings before drawing the object.

# EBO

Determines offsets of points of the primitive in the buffer data *(so you need both the indices and a VBO data)*. When used, a draw command should be not `glDrawArrays()` but `glDrawElements()`.

# vertex shader

`layout (location = 0) in vec3 myVariable` — specifies a variable consisting of 3 floats, whose data is from a VBO. In the application code the location needs to be configured with the first parameter to `glVertexAttribPointer()`. This allows to pass data from multiple buffers to a single shader. Part of a code from here https://computergraphics.stackexchange.com/questions/4637/can-one-vao-store-multiple-calls-to-glvertexattribpointer

    rendering-loop
    {
        glBindVertexArray(vao);

        glBindBuffer(GL_ARRAY_BUFFER, points_vbo);
        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, NULL);
        glBindBuffer(GL_ARRAY_BUFFER, colours_vbo);
        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 0, NULL);

        glDrawArrays(GL_TRIANGLES, 0, 3);
    }

# some examples

    /* The function called when our window is resized */
    void ReSizeGLScene(int Width, int Height)
    {
        if (Height==0)				// Prevent A Divide By Zero If The Window Is Too Small
            Height=1;

        glViewport(0, 0, Width, Height);		// Reset The Current Viewport And Perspective Transformation

        glMatrixMode(GL_PROJECTION);
        glLoadIdentity();

        gluPerspective(45.0f,(GLfloat)Width/(GLfloat)Height,0.1f,100.0f);
        glMatrixMode(GL_MODELVIEW);
    }

## VAO + VBO + shaders = triangle

https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/2.1.hello_triangle/hello_triangle.cpp

# todo
## Mesa Alternate Frame Rendering

App is called with MESA_AFR=1 set. In general what happens:

There's a hypotetic generic function where glDraw\* calls are end up; let's call it `mesa_draw()`. In alike way, another one to swap buffers, let's call it `mesa_swap_buf()`.

1. App calls `mesa_draw()`.
2. `mesa_draw()` copies the data, then:
    1. if there's a spare GPU, immediately returns.
    2. otherwise, proceeds as usual
3.  App calls `mesa_swap_buf()`, which in turn
    1. if prev. frame happened in current vblank, block till vblank
    2. if there're new frames, swap with the oldest one from one of GPUs
    3. otherwise block *(ideally till a GPU finish rendering)*, then go to 3.2↑
    4. schedule as a GPU to be used for `mesa_draw()` the one from 3.2

## Skipping driver calls

Absolutely crazy idea: how about making tuples `(hash state, CS)`, and passing the command stream instead of processing the state again and again? To not over-junk it, statistics allows to leave only the most used ones, and as a future feature maybe allows cross draw-calls optimizations.

Main question is: whether state passed to `*draw_vbo` in one-to-one correspndence with the produced CS; and if not — is there a point where this optimization would make sense.
