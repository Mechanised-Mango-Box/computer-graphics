# 3 Rendering

Looking from above is awfully boring so time to do what we really came here to do which is to render this in 3D.

## 3.1 Theory

Here I will go through the steps from converting a 3D world into a 2D image using a method called raycasting.

### 3.1.1 Everything Is Cooler with Lasers

We already know our world can be represented in 2D as a top-down "flat world" so lets think about it that way for now. Imagine we have a 2D radar that shoots lazers from a single pinhole, where we project out rays at incremental angles, which can tell us how **far** and what **colour** is an ***object it hit*** (if it hit one at all)

![](./theory-projection.svg)

Which to the radar, looks like this:

![](./theory-radar.svg)

Which we can process by removing the borders between each "vertical pixel"

![](./theory-radar-cleaned.svg)

### 3.1.2 ~~New~~ Fake Horizons

Right now it doesn't look too convincing so we can add a (fake) horizon, sky and floor to the image.

![](./theory-fake-horizons.svg)

Looks a bit better but still awkward right?

### 3.1.3 You Are Looking at It Wrong: It’s All About Perspective

Remember I mentioned distance before? Well here we scale the vertical pixel's height with how far away the object was before a ray hit it.

![](./theory-perspective.svg)

Here is the layout for reference:
![](./theory-projection.svg)

### 3.1.4 RTX On (what if we just had more pixels)

To make it look even better, here's what it would look like with much more pixels, instead of having only a screen with ~30 pixels wide. On top of that we can add some light to sides facing an arbitrary room light (shading).

![](./theory-interpolated.svg)

Looks pretty good right? (I cheated a bit by using polygons but I am not drawing that many pixels)

## 3.2 FPS Inputs

So far we have been playing the game from the top down, and moving in global space, where `W/A/S/D` corresponds to North/West/South/East respectively.

![](./input-move-global.svg)

But as a shooter, the control scheme should be using relative directions of forward and strafe/right (which direction is forward when rotation is 0 is arbitrary):

![](./input-move-local.svg)

We do this by rotating the input to be relative to the player's direction. We can also use our mouse to rotate. 

```js
const move = (player, axis_fwd, axis_strafe, delta) => {
    /**
     *     -strafe
     *       |
     *     --@----- > +forwards
     *       |
     *       v
     *     +strafe
     */
    if ((axis_fwd != 0 || axis_strafe != 0)) {
        const length = Math.sqrt(axis_fwd * axis_fwd + axis_strafe * axis_strafe);
        const local_x = axis_fwd / length;
        const local_y = axis_strafe / length;

        const looking_x = Math.cos(player.rot);
        const looking_y = Math.sin(player.rot);

        const move_x = local_x * looking_x + local_y * looking_y;
        const move_y = local_x * looking_y - local_y * looking_x;
        const velocity = player.move_speed * delta;
        player.pos_x += move_x * velocity;
        player.pos_y += move_y * velocity;
    }
}

let sens = 0.35;
let axis_strafe = 0;
let axis_forwards = 0;

const tick = (delta) => {
    //strafe
    axis_strafe = 0
    if (input.keys.a) { axis_strafe += -1; }
    if (input.keys.d) { axis_strafe += 1; }
    //forwards
    axis_forwards = 0
    if (input.keys.s) { axis_forwards += -1; }
    if (input.keys.w) { axis_forwards += 1; }
    //look
    if (input.mouse.dx) {
        player.rot += sens * -input.mouse.dx * delta; // -ve = non-inverted mouse
        input.mouse.dx = 0; // consume the delta after reading otherwise it will continue with no input
    }

    move(player, axis_forwards, axis_strafe, delta)
}
```

## 3.3 Raycasting

### 3.3.1 Rays on Grids

To implement the rays, we can make use of a variation to the DDA, accounting for the position being a decimal position within a cell. We also scale the coordinates by the tilemap scale so that the calculations apply at the right distances. 

> https://lodev.org/cgtutor/raycasting.html does a far better job of explaining it.

```js
const DEG2RAD = Math.PI / 180;
const FOV_DEFAULT_DEG = 90 // source-like default
let fov = FOV_DEFAULT_DEG * DEG2RAD;

/**
 * @param{Tilemap} tilemap
 * @param{number} ox       
 * @param{number} oy      
 * @param{number} dx
 * @param{number} dy
 * @param{number} max_dist
 */
const raycast_dda = (tilemap, ox, oy, dx, dy, max_dist) => {
    // Scale positions to tilemap coords
    ox /= tilemap.grid_scale;
    oy /= tilemap.grid_scale;
    dx /= tilemap.grid_scale;
    dy /= tilemap.grid_scale;
    max_dist /= tilemap.grid_scale;

    // Cell positions are always ints
    let cx = Math.floor(ox);
    let cy = Math.floor(oy);

    // Diagonal distance from the edges of x to x+1 (or y to y+1)
    const diag_dist_x = Math.abs(1 / dx);
    const diag_dist_y = Math.abs(1 / dy);

    // Distance to nearest edge along said axis
    let edge_dist_x;
    let edge_dist_y;

    // Direction to step in
    let step_x;
    let step_y;

    // Calculate initial step (to a cell edge)
    if (dx < 0) {
        step_x = -1;
        edge_dist_x = (ox - cx) * diag_dist_x;
    } else {
        step_x = 1;
        edge_dist_x = (cx + 1.0 - ox) * diag_dist_x;
    }

    if (dy < 0) {
        step_y = -1;
        edge_dist_y = (oy - cy) * diag_dist_y;
    } else {
        step_y = 1;
        edge_dist_y = (cy + 1.0 - oy) * diag_dist_y;
    }

    let side = 0;
    let perp_dist = 0;

    // DDA loop
    while (perp_dist <= max_dist) {
        // Choose the axis to "snap" to which requires moving along the diagonal the least
        if (edge_dist_x < edge_dist_y) {
            perp_dist = edge_dist_x;
            edge_dist_x += diag_dist_x;
            cx += step_x;
            side = 0;
        } else {
            perp_dist = edge_dist_y;
            edge_dist_y += diag_dist_y;
            cy += step_y;
            side = 1;
        }

        const bi = bi_s(cx, cy, tilemap.w, tilemap.h);
        if (bi == undefined) {
            return null;
        }

        const cell = tilemap.buffer[bi];
        if (cell == '#') {
            return {
                side: side,
                cell: cell,
                cell_pos_x: cx,
                cell_pos_y: cy,
                perp_dist: perp_dist,
            };
        }
    }
    return null;
};
```

### 3.3.3 Rendering

As mentioned before the distance to the surface is used to draw the height of the wall. To ensure that we do not suffer from the fisheye effect, we use the perpendicular distance of the ray, to the walls are projected with straight edges. A max distance is used for sanity (although all Wolf3D maps are always "boxed in" so the ray length could be infinite).

If you noticed before, we kept track of the `side` (N/S or E/W) of the surface that was hit. This is used to give more of a 3D feel, as only 2 sides will ever be visible (one from each group). I also added some shading based on the distance the wall is so that near and far walls can be more easily distinguished.

> Again see: https://lodev.org/cgtutor/raycasting.html.

```js
const render_scene = (delta, viewport, max_render_dist) => {
    const w = viewport.get_viewport_w();
    const h = viewport.get_viewport_h();

    // Camera / player look direction  
    const look_dir_x = Math.cos(player.rot);
    const look_dir_y = Math.sin(player.rot);

    // The camera plane (a line in 2D) defined by a 2D vector
    //                      
    //                         plane width
    //                     +-----------------+ 
    //                     ___________________ 
    //                  +  \        |        /
    //                  |   \       |       /
    //                  |    \      |      /
    //                  |     \     |     /
    //   plane distance |      \    |    /
    //                  |       \   |   /
    //                  |        \  |  /   
    //                  |         \ | / \___ FoV   
    //                  |          \|/
    //                  +           v
    //                              @
    //                            
    //

    // Scaling the camera plane based on the fov (a ratio between the distance of the plane and the (half-)width)
    const fov_proportion = Math.tan(fov / 2)

    // The camera plane (x+ is the "forward" of the camera at rotation = 0)
    //
    //     y+
    //     ^            
    //     |          /
    //     +--> x+   / 
    //              /  
    //             /   
    //            /    
    //           @----->  look dir            + 
    //            \                           | camera plane
    //             \                          |
    //              \                         v
    //               \ 
    //                \
    //                 
    // Convert the camera plane from local space to world space
    const cam_plane_x = -look_dir_y * fov_proportion;
    const cam_plane_y = look_dir_x * fov_proportion;

    // Cache the calculation of the wall height scaling
    const wall_scale_factor = (h * tilemap.grid_scale) / fov_proportion;

    // Foreach vertical pixel
    for (let px = 0; px < w; ++px) {
        // Map the pixel to the offset from the center of the plane
        const offset_along_plane = 1 - (2 * px / w);
        // Take the raycast direction
        const dx = look_dir_x + cam_plane_x * offset_along_plane;
        const dy = look_dir_y + cam_plane_y * offset_along_plane;

        const hit = raycast_dda(tilemap, player.pos_x, player.pos_y, dx, dy, max_render_dist);
        if (hit != null) {
            // Height of the hit wall
            const line_height = Math.floor(wall_scale_factor / hit.perp_dist);

            let py_wall_bottom = (h - line_height) / 2;
            let py_wall_top = (h + line_height) / 2;

            const shade = 1 - Math.sqrt(hit.perp_dist / max_render_dist);
            const color = 220 * shade * (hit.side === 1 ? 0.6 : 1);

            // Draw
            for (let py = 0; py < h; ++py) {
                if (py < py_wall_bottom) {
                    // Floor
                    viewport.set_pixel(px, py, 39, 45, 45);
                } else if (py_wall_top < py) {
                    // Celing / Sky
                    viewport.set_pixel(px, py, 25, 40, 100);
                } else {
                    // Wall
                    viewport.set_pixel(px, py, color, color, color);
                }
            }
        } else {
            // ERROR - failed to hit tile
            for (let py = 0; py < h; ++py) {
                viewport.set_pixel(px, py, 255, 0, 255);
            }
        }
    }
};
```

After all that hard work the code should be able to do this:
![](./rendering-example.png)

## 3.4 Texture Mapping









## 3.4 Further Reading:
1. Lode's Computer Graphics Tutorial - Raycasting *(https://lodev.org/cgtutor/raycasting.html)*
2. Wikipedia - Digital Differential Analyzer *(https://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm))*
