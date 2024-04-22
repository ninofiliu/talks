---
theme: default
title: Weird Web APIs I love to use when making art in the browser
highlighter: shiki
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Weird Web APIs I love to use when making art in the browser

By Nino Filiu

[github.com/ninofiliu/talk-queerjs2024](https://github.com/ninofiliu/talk-queerjs2024)

---

# About me

I make video games and interactive art with computers and electronics + many other things

🧑 26yo

🏳️‍🌈 he/they

🎓 Telecom Paris + Technische Universität Berlin

🔗 [ninofiliu.com](https://ninofiliu.com)

<div style="display: flex; height: 200px">
  <img src="/nino_shader.jpg"/>
  <img src="/nino_sculpture.png"/>
  <img src="/nino_blender.jpeg"/>
  <img src="/nino_dj.jpg"/>
  <img src="/nino_mag.jpeg"/>
</div>

---

# Short company promo

We're hiring!

ℹ️ Toucan Toco

🚀 Dataviz

🧑‍💻 Healthy and modern codebase

💚 Safe environment

💬 [Welcome To The Jungle](https://www.welcometothejungle.com/fr/companies/toucan-toco)

---

layout: image-right
image: /canvas_line.png

---

# Canvas API

Drawing a dashed line on a blue background

```html
<canvas></canvas>
```

```ts
// create a canvas and a 2D context
const canvas = document.querySelector("canvas")!;
canvas.width = 300;
canvas.height = 200;
const ctx = canvas.getContext("2d")!;

// fill the whole canvas blue
ctx.fillStyle = "blue";
ctx.fillRect(0, 0, 300, 200);

// draws a dashed line
ctx.strokeStyle = "steelblue";
ctx.lineWidth = 10;
ctx.setLineDash([20, 20]);
ctx.moveTo(50, 50);
ctx.lineTo(250, 150);
ctx.stroke();
```

---

# Canvas API

Reading pixel data of images and videos

```html
<canvas></canvas> <img src="/my-pic" />
```

```ts
const canvas = document.querySelector("canvas")!;
const ctx = canvas.getContext("2d")!;

const img = document.querySelector("img");
canvas.width = img.width;
canvas.height = img.height;
ctx.drawImage(img, 0, 0, img.width, img.height);

const imageData = ctx.getImageData(0, 0, img.width, img.height);

imageData.width; // number
imageData.height; // number
imageData.data; // number[]
```

- 0..3 = rgba top left pixel
- 4..7 = rgba of the one next to it
- ...

---

# Canvas API

Paints the top left pixel half opaque red

```ts
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
imageData.data[0] = 255;
imageData.data[1] = 0;
imageData.data[2] = 0:
imageData.data[3] = 128:
ctx.putImageData(imageData, 0, 0);
```

---

# WebUSB

Allows the web to speak to USB devices

Groundbreaking in some industries, but it was such a hassle to use it IMO

- poor browsing support
- OS-dependent permissions
- "raw" USB protocol

If you can use another API, do it

---

# WebMIDI

Allows navigators to communicate with MIDI devices

MIDI: Musical Instrument Digital Interface

Simple protocol:

- input and output channels
- messages have three bytes
- specs defines their meaning: `[144, 60, 127]` = note on, middle C, full velocity

[WebMIDI debugger](https://moutend.github.io/web-midi-debug/)

From personal XP, it's usually easier to play around with the debugger and understand the patterns rather than reading the docs

Only one Chrome for now

---

# WebMIDI

Log all messages on all inputs

```ts
const access = await navigator.requestMIDIAccess();
for (const input of access.inputs) {
  input.addEventListener("midimessage", (evt) => {
    const [a, b, c] = evt.data;
  });
}
```

---

# WebMIDi

Play a note on all outputs

```ts
const access = await navigator.requestMIDIAccess();
for (const output of access.outputs) {
  output.send([144, 60, 127]);
}
```

---

# WebMIDI

Sacrifice Demo

---

# WebGL

What are they?

GPUs: many thousand times more cores than CPUs, but that can only run a few programs at a time

Useful for graphics programming (same programs for many pixels)

OpenGL: driver for GPU

GLSL: OpenGL shading language

WebGL: run OpenGL in a `<canvas>`

WebGPU: successor to WebGL, can ask the GPU to execute arbitrary tasks, not just graphics - also machine learning and fluid simulations

---

# WebGl

hello world

Drawing one triangle on the screen takes at least 70 lines :p

```ts
const vertexShaderSource = `#version 300 es
in vec4 a_position;
void main() {
  gl_Position = a_position;
}
`;

const fragmentShaderSource = `#version 300 es
precision highp float;
out vec4 outColor;
void main() {
  outColor = vec4(1, 0, 0.5, 1);
}
`;

const x = <T>(x: T | null | undefined): T => {
  if (x == null) throw new Error("should not be nullish");
  return x;
};

const createShader = (
  gl: WebGL2RenderingContext,
  type: number,
  source: string
) => {
  const shader = x(gl.createShader(type));
  gl.shaderSource(shader, source);
  gl.compileShader(shader);
  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS))
    throw new Error(`${gl.getShaderInfoLog(shader)}`);
  return shader;
};

const createProgram = (
  gl: WebGL2RenderingContext,
  vertexShader: WebGLShader,
  fragmentShader: WebGLShader
) => {
  const program = x(gl.createProgram());
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  gl.linkProgram(program);
  if (!gl.getProgramParameter(program, gl.LINK_STATUS))
    throw new Error(`${gl.getProgramInfoLog(program)}`);
  return program;
};

const canvas = document.createElement("canvas");
document.body.append(canvas);
const gl = x(canvas.getContext("webgl2"));

const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
const fragmentShader = createShader(
  gl,
  gl.FRAGMENT_SHADER,
  fragmentShaderSource
);
const program = createProgram(gl, vertexShader, fragmentShader);
const locations = {
  a_position: gl.getAttribLocation(program, "a_position"),
};
gl.bindBuffer(gl.ARRAY_BUFFER, gl.createBuffer());
gl.bufferData(
  gl.ARRAY_BUFFER,
  new Float32Array([0, 0, 0, 0.5, 0.7, 0]),
  gl.STATIC_DRAW
);
const vao = gl.createVertexArray();
gl.bindVertexArray(vao);
gl.enableVertexAttribArray(locations.a_position);
gl.vertexAttribPointer(locations.a_position, 2, gl.FLOAT, false, 0, 0);
gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
gl.clearColor(0, 0, 0, 0);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.useProgram(program);
gl.bindVertexArray(vao);
gl.drawArrays(gl.TRIANGLES, 0, 3);
```

---

# WebGL

Made with WebGL

- [ThreeJS](https://threejs.org/): de-facto standard lib for 3D stuff on the web
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber/getting-started/introduction): write ThreeJS stuff in JSX
- [Spline](https://spline.design/): 3D software that runs inside the browser
- [Womp](https://womp.com/index): same, but focused on smooth shapes
- [Cannon](https://pmndrs.github.io/cannon-es/): 3D physics library
- [Shadertoy](https://www.shadertoy.com/): shader social network

# Canvas API

# WebCodecs

# Gamepad API

# Web Audio

# Made with WebGPU: TensorflowJS
