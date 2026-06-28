# 5 Mobile Controls 

> [Demo](./wolfenstein-3d.html)

It is not great to show this off only to tell someone that they need to whip out a keyboard and mouse to try it...

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
