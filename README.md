# takeshijs and motivations

takeshi.js is a small TypeScript library targeting WebGL2.

takeshijs' primary use case is image processing, not rendering geometry. That said, using takeshijs with other WebGL2 code should be well supported.

The main goals are:
- A simple API.
- It should be easy to set up an operation taking multiple images as input and having multiple outputs.
- Regular WebGL2 code should not be needed unless for special situations.
- Getting WebGL2 textures in and out from takeshijs should be easy, for example to do post processing.

Future goals are:
- A way to write simpler kernels in ES6 and have them compiled.

Known limitations:
- You can not operate on image data in place: this is a restriction due to OpenGL where writing and reading from the same image is undefined behaviour.
- Using framebuffers is a must to support a kernel writing to multiple targets, and thus the user is beholden to all the restrictions and limitations they have.

# API sketch

~~~typescript
let context = takeshi.fromGLContext(gl);

let textureTypes = context.textureTypes;

let inputs = { 
    diffuse: context.createTexture([2, 2], textureTypes.RGB16F),
    z: context.createTexture([2, 2], textureTypes.R16F),
};

inputs.diffuse.loadData(new Float32Array([1,1,1, 2,2,2, 3,3,3, 4,4,4]));
inputs.z.loadData(new Float32Array([1,2,3,4]));

let outputs = {
    color: context.createRGBTexture([2, 2], textureTypes.RGB8)
};

let kernel = context.createKernel(`
    void kernel(vec2 pos, vec2 size) 
    {
        out_diffuse = texture(in_diffuse).rgb * z;
    }
`, inputs, outputs);

context.execute(kernel, inputs.diffuse, outputs.color);
~~~ 
