# 1. Setup
The first order of business is create the tick/update cycles. This is copied from an earlier project, but with support for the render and physics ticks to be on seperate frames (i.e. when the physics is locked to a certain tick rate)

```js
const draw = (delta, viewport) => {

}

const tick_rate = 60;
const render_fps = null;

let physics_last_frame = performance.now();
let render_last_frame = performance.now();
let physics_delta = 0;
let render_delta = 0;

const tick = (delta) => {

}


const viewport = setup_viewport();
viewport.set_resolution(1280, 720);

const update = () => {
    const current_frame = performance.now();

    physics_delta = (current_frame - physics_last_frame) / 1000;
    if (tick_rate == null || physics_delta >= 1 / tick_rate) {
        tick(physics_delta);
        physics_last_frame = current_frame;
    }

    render_delta = (current_frame - render_last_frame) / 1000;
    if (render_fps == null || render_delta >= 1 / render_fps) {
        draw(render_delta, viewport);
        viewport.render()
        render_last_frame = current_frame;
    }

    requestAnimationFrame(() => update(viewport));
};

requestAnimationFrame(() => update());
```

Next will be to create the viewport throught which the virtual world should be rendered. This will be done by drawing individual pixels to a frame buffer each render frame (as opposed to before where I have somewhat cheated by drawing using the `SVG`-like functions of a `canvas`).

```js
const DEFAULT_VIEWPORT_W = 400;
const DEFAULT_VIEWPORT_H = 300;
const setup_viewport = () => {
    const canvas = document.getElementById("viewport");
    const ctx = canvas.getContext("2d");

    let viewport_w = DEFAULT_VIEWPORT_W;
    const get_viewport_w = () => viewport_w;
    let viewport_h = DEFAULT_VIEWPORT_H;
    const get_viewport_h = () => viewport_h;

    let imageData;
    let framebuffer;

    const set_resolution = (w, h) => {
        canvas.width = w;
        canvas.height = h;

        imageData = ctx.createImageData(w, h);
        framebuffer = imageData.data;
    };

    const set_pixel = (x, y, r, g, b, a = 255) => {
        const i = (y * get_viewport_w() + x) * 4;
        framebuffer[i] = r;
        framebuffer[i + 1] = g;
        framebuffer[i + 2] = b;
        framebuffer[i + 3] = a;
    };
    const render = () => ctx.putImageData(imageData, 0, 0);

    return {
        get_viewport_w,
        get_viewport_h,
        set_resolution,

        set_pixel,
        render,
    };
}
```
