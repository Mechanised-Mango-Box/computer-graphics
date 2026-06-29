# 5 Mobile Controls 

> [Demo](./wolfenstein-3d.html)

The extras are just some changes that might not need a full writeup (or are not that interesting).

With that said, it is not great to show this off only to tell someone that they need to whip out a keyboard and mouse to try it...

## 5.1 Adding Buttons
Nothing really special, just another dictionary of input that gets processed all the same:
```js
// Virtual Controls
const bind_virtual_button = (button, bind) => {
    // Button
    button.addEventListener("touchstart", e => {
        e.preventDefault();

        input.virtual[bind] = true;
    }, { passive: false })
    button.addEventListener("touchend", e => {
        e.preventDefault();

        input.virtual[bind] = false;
    }, { passive: false })

    // Mouse
    button.addEventListener("mousedown", e => {
        input.virtual[bind] = true;
        e.preventDefault();
    });

    button.addEventListener("mouseup", e => {
        input.virtual[bind] = false;
    });

    button.addEventListener("mouseleave", () => {
        input.virtual[bind] = false;
    });
}
const button_move_forward = document.getElementById("virtual-dpad-forward"); bind_virtual_button(button_move_forward, "dpad_forward")
const button_move_backward = document.getElementById("virtual-dpad-backward"); bind_virtual_button(button_move_backward, "dpad_backward")
const button_move_left = document.getElementById("virtual-dpad-left"); bind_virtual_button(button_move_left, "dpad_left")
const button_move_right = document.getElementById("virtual-dpad-right"); bind_virtual_button(button_move_right, "dpad_right")

const button_look_left = document.getElementById("virtual-look-left"); bind_virtual_button(button_look_left, "look_left")
const button_look_right = document.getElementById("virtual-look-right"); bind_virtual_button(button_look_right, "look_right")
```

And adapt the input processing to this new parallel input set.
```js
let sens = 0.001;
let virtual_sens_multiplier = 2000;
const input = {
    kbm: {
        keys: {},
        mouse: {
            dx: 0, dy: 0,
            buttons: {}
        },
    },
    virtual: {
        dpad_forward: false,
        dpad_backward: false,
        dpad_left: false,
        dpad_right: false,

        look_left: false,
        look_right: false
    }
};

const process_input = (delta) => {
    //strafe
    player.wish_strafe = 0
    if (input.kbm.keys.a || input.virtual.dpad_left) { player.wish_strafe += -1; }
    if (input.kbm.keys.d || input.virtual.dpad_right) { player.wish_strafe += 1; }
    //forward
    player.wish_forward = 0
    if (input.kbm.keys.s || input.virtual.dpad_backward) { player.wish_forward += -1; }
    if (input.kbm.keys.w || input.virtual.dpad_forward) { player.wish_forward += 1; }
    //look
    if (input.kbm.mouse.dx) {
        player.rot += sens * -input.kbm.mouse.dx; // -ve = non-inverted mouse
        input.kbm.mouse.dx = 0; // consume the delta after reading otherwise it will continue with no input
    }
    else {
        if (input.virtual.look_left) {
            player.rot += sens * virtual_sens_multiplier * delta;
        }
        if (input.virtual.look_right) {
            player.rot -= sens * virtual_sens_multiplier * delta;
        }
    }
}
```

## 5.2 Acceleration Based Movement

I prefer a more Source-like movement which uses acceleration based movement.

```js
let is_moving_input = false;
/**
 * @param {float} delta 
 */
const move = (delta) => {
    /**
     *     -strafe
     *       |
     *     --@----- > +forward
     *       |
     *       v
     *     +strafe
     */

    is_moving_input = (player.wish_forward != 0 || player.wish_strafe != 0)

    const speed = Math.sqrt(player.vx * player.vx + player.vy * player.vy);

    if (is_moving_input) {
        // accel
        const length = Math.sqrt(player.wish_forward * player.wish_forward + player.wish_strafe * player.wish_strafe);
        const local_x = player.wish_forward / length;
        const local_y = player.wish_strafe / length;

        const looking_x = Math.cos(player.rot);
        const looking_y = Math.sin(player.rot);

        const move_x = local_x * looking_x + local_y * looking_y;
        const move_y = local_x * looking_y - local_y * looking_x;

        player.vx += move_x * player.accel * delta;
        player.vy += move_y * player.accel * delta;

        const current_speed = Math.sqrt(player.vx * player.vx + player.vy * player.vy);
        if (current_speed > player.max_speed) {
            const clamp_scaler = player.max_speed / current_speed;
            player.vx *= clamp_scaler;
            player.vy *= clamp_scaler;
        }
    } else {
        // friction
        const speed = Math.sqrt(player.vx * player.vx + player.vy * player.vy);

        if (speed > 0) {
            const drop = speed * player.friction * delta;
            let new_speed = speed - drop;
            if (new_speed < 0) new_speed = 0;

            const friction_force = new_speed / speed;
            player.vx *= friction_force;
            player.vy *= friction_force;
        } else {
            player.vx = 0;
            player.vy = 0;
        }
    }

    player.x += player.vx * delta;
    player.y += player.vy * delta;
}
```