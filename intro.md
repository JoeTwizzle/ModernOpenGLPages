# Welcome
Have you ever wanted to learn modern computer graphics, but struggled finding a good place to learn best practices? <br/> **Then look no further.**
This site will teach you modern computer graphics techniques in OpenGL, **no prerequisite knowledge**, other than simple programming skills are required.

# A brief overview of the past

To understand why things are the way that they are nowadays, it is best we take a look in toward the past, to understand the reasoning behind these decisions.

### Early Computer Graphics

> [!NOTE]  
>  This section mainly covers early home computers and abrieviates and abstracts many things. (This is NOT a complete or chronological re-telling of history)

The way hardware worked back then is that a dedicated display adapter card would be plugged into a computer, in order to convert stored characters of text, into video which could be displayed on a screen. This is a primitive and early form of what would become the modern GPU.

Even early on in the home computing days, people had a desire to display complex visuals on their screens. Whether it was for showing graphs and charts, or for entertainment purposes. People wanted to see full color ''high resolution'' images. Unfortunately the display adapter cards of the time simply did not have the hardware capabilities needed to pull this off.

Manufacturers took note of this demand, and saw the capabilities which arcade machines and home consoles of the time had.
This led many of them to modify their display adapter cards and add their own custom video modes. If you used these custom video modes, you had the ability to use whatever custom features the manufacturer decided to add. 
* More colors!
* Higher resolutions!
* Smoothly moving images!
* 3D-Rendering!

But there was one major issue, which prevented the widespread adoption of these features. <br/>
If you wanted to use these features, you had to specifically write you program to support the exact card that had the features you wanted. Want to support a different card? I hope you enjoy writing two versions of the same graphics rendering code. 

In order to solve these issues a unified standard was needed, which all manufacturers would adhere to. That way a programmer could write their code once and it would work on all cards which supported this standard.

### What ***exactly*** is a Graphics API?

A graphics API is what was needed. So in 1992 industry leader Silicon Graphics, Inc. (SGI) created an open standard defining what functionality should be supported. This graphics API was called **OpenGL**.

> [!NOTE]  
> This is a huge simplification, there were other graphics APIs before, however OpenGL was the first to reach widespread adoption.

# Ancient OpenGL

OpenGL featured an **immediate mode API**, this programming paradigm meant that the display adapter card, (by then more commonly called video card), did not retain any data. This meant that every rendering command (rendering is the act of drawing an image onto a screen/buffer) issued by the cpu would also have to pass along **all the information related to that command**. This included things such as light positions, object positions, surface colors, etc... each and **every time** any object was rendered. This may seem extremely wasteful in terms of resources, however it made it simple for programmers to have more **direct control over how and when** things were rendered, this ended up reducing the total amount of computation that needed to be done.

# Classic OpenGL

As computers and video cards became faster and higher detail 3D-models were more commonly used, it became clear that the old immediate mode API was a bottleneck for rendering speeds. As such the OpenGL specification was updated to include several **new APIs based on a retained mode** design philosophy. 

The retained mode API **allowed data to be stored in video memory**, which was located **directly on the video card**. A benefit of this was that reading said data was **much faster** than reading data from normal RAM. 

The newer APIs also introduced the concept of ***state***. The state is a representation of the current configuration of the video card. It holds information such as, what textures should be used to render the next object and what buffers should be used to render the next object.

### Pseudocode
```c
//Set as texture to use for the next object
glBindTexture(myTexture);
//Set as vertex buffer for the next object
glBindBuffer(myVertexBuffer);
//Render next object
glDrawArrays(/*bla bla*/);
```

This design made sense at the time. Almost all computers only had one CPU core, meaning that managing a global state was a relatively simple task, since there was no need to concern yourself with the possibility of cross-thread communication. 

However even at the time there were some apparent issues with this design. As everything shared a single global state, any changes made to that state would naturally persist until that state is written to again. This is a problem, it effectively means that calling any code that ***could*** have drawn something to the screen, results in you having to reset the state again, even if nothing was actually drawn.

Another large problem was that the state did not just serve the purpose of holding information to use for rendering objects. It was also used whenever you wanted to modify any property of any object.

### Pseudocode
```c
//Set texture as the active texture in the state
glBindTexture(myTexture);
//Set the filter mode to linear for the currently bound texture
glSetTextureFilterMode(GL_LINEAR);

//Set vertex buffer as the active buffer in the state
glBindBuffer(myVertexBuffer);
//Upload data to the currently bound buffer
glUploadBufferData(myVertexDataArray);
```

This meant that it was often necessary to change the global state, even in methods that have nothing to do with rendering objects. Something like loading 3D-model data from disk could end up modifying this state, resulting in other code needing to reset the state.

As such it was nearly impossible to maintain a clean separation of concerns in code, resulting in many redundant writes to the state, in order to ensure that it would always be in a known good condition.

# Traditional OpenGL

As time went on video card gained the ability to execute limited programs on them. These programs, known as shaders, could modify how vertices were positioned on the screen and how pixels were colored. Video cards with these capabilites were called Graphics Processing Units (GPUs).

Over time, as hardware got more capabilites, OpenGL was naturally updated to include these new features. First vertex and fragment(pixel) shader support was added. Then Geometry shader support, able to operate on whole groups of triangles. This was followed by tesselation shaders, able to dynamically generate more triangles, the closer you were to the surface of a mesh. Lastly and most importantly were compute shaders. These allowed any arbitrary computations to be performed in parallel on the GPU. 

# Modern OpenGL

With compute shaders becoming widely supported, a new paradigm for rendering emerged. 

### GPU driven rendering

Traditionally, if you wanted to render an object, or groups of objects which happened to use the same shaders, you had to issue a render command from the CPU. This meant that the information on what objects to draw always had to be transfered from the CPU to the GPU for each and every render command.

Since compute shaders were now able to perform arbitrary computations on the GPU, the possibility of having the GPU generate its own render commands existed. On its own this capability had little impact on renderering perfomance. However, the added flexibility allowed other processes, such as determining what objects to render, to also be offloaded to the GPU. This was impossible before, as Transfering data from GPU to CPU and then back to GPU would result in terrible performance.

### Direct state access

These new features allowed OpenGL to be a capable cross platform API, which could be used to implement even the most cutting edge techniques at the time. However there was one annoyance that still persisted from previous versions of OpenGL which continued to inhibit writing performant, well structured code. 

The statefull, ***bind to modify*** model was the culprit. Thus seeing the issue this API presented, the OpenGL standard was updated to introduce a new set of APIs, which allowed directly modifying properties on OpenGL objects. 
These sets of APIs are known as **Direct state access (DSA)** APIs.

# Overview of OpenGL Objects and Features

| Object            | Description                                                         |
| --------------    | -----------------------------------------------------------------   |
| Buffers           | Holds arbitrary data visible to the GPU. <br/> Many different flags define how and where data is stored and where data is visible from. <br/> Triangle positions and other attributes are read from buffers. |
| Textures          | Textures store image data in GPU memory.<br/> Image data may have 1 to 4 color channels (called components).<br/> Image data may have differernt color depths (e.g. 8-Bit 16-Bit 32-Bit).<br/> Textures may contain many smaller versions of the same image (called mipmaps). <br/> When rendering the image with the size closest to the triangles size on the screen is chosen, in order it increase performance and reduce aliasing artifacts.         |
| Samplers          | Define how textures are sampled when rendering objects.<br/> When getting the color for a given position on a triangle, the returned color may be a mix of neighbouring pixels or just the clostest pixel, depending on sampling parameters.                      |
| Texture Views     | A view on an existing texture that may be of another color depth, or of a smaller mip-level.                                                                        |
| Framebuffers      | A Framebuffer holds the rendered image data, via attached color textures and/or an attached depth texture.                                                                                  |
| Shaders           | A shader is an intermediate object that is used to create execuable GPU programs. It represents a single 'stage' of a final executable GPU program.                                                                                  |
| Programs          | A program (also called shader program) is an  program. It is created by attaching, linking and then compiling one or more shader objects, for each 'stage' of the rendering pipeline.                                                                                 |
| Vertex Arrays     | A Vertex Array Object contains information on the layout of Vertices stored in a Buffer object, which is attached to the Vertex Array. <br/> A vertex array has one or more buffers attached to it, which store Vertex data or index data, used for rendering triangle meshes.                                                                          |
