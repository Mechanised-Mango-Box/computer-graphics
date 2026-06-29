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
    }, { passive: false });

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

I also added a resolution and FoV slider although no all sizes are fully tested 

## 5.3 Acceleration Based Movement

I prefer a more Source-like movement which uses acceleration based movement.

```js
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

    const speed = Math.hypot(player.vx, player.vy);

    // Friction
    if (speed > 0) {
        // stop speed (stop scaling friction if lower than stop_speed - keeping friction high)
        const control = speed < player.stop_speed ? player.stop_speed : speed;

        const drop = control * player.friction * delta;

        const new_speed = Math.max(speed - drop, 0);
        const scale = new_speed / speed;

        player.vx *= scale;
        player.vy *= scale;
    }

    const f = player.wish_forward;
    const s = player.wish_strafe;

    const wish_len = Math.hypot(f, s);
    if (wish_len == 0) {
        player.x += player.vx * delta;
        player.y += player.vy * delta;
        return;
    }

    const lf = f / wish_len;
    const ls = s / wish_len;

    const cos = Math.cos(player.rot);
    const sin = Math.sin(player.rot);

    const wish_x = lf * cos + ls * sin;
    const wish_y = lf * sin - ls * cos;

    let wish_speed = player.max_speed;
    if (wish_speed > player.max_speed) {
        wish_speed = player.max_speed;
    }

    // Accel
    const current_speed_in_wish_dir = player.vx * wish_x + player.vy * wish_y;
    const add_speed = wish_speed - current_speed_in_wish_dir;

    if (add_speed > 0) {
        const accel = player.accel * delta * wish_speed;
        const applied = Math.min(accel, add_speed);

        player.vx += wish_x * applied;
        player.vy += wish_y * applied;
    }

    // Apply velocity
    player.x += player.vx * delta;
    player.y += player.vy * delta;
}
```