# 2. Rendering

============================================================TODO summary + clean up writeup
**Click here for a [Interactive Demo](./wolfenstein-3d.html)**

## 2.1 Things To Look At (Map)

- levels (copy from ray cast thingo)

How that we know how to fire a beam and check if it hits anything, we can try and see into the level with rays (recall the radar analogy). We will "cheat" a little and have only axis-aligned tiles for a map.

Wolfenstein used a tile based level, where each tile could be one of various objects (walls, doors, enemies, etc). So to start off simple let's go with this map:

```js
const tilemap = load_tilemap(`
###########
#   #     #
#   #  #  #
## ##     #
#         #
#     #####
#         #
# #       #
# ###     #
#      S  #
###########
`)
```

Which we will load into an array like so:

```js
const buffer_index = (x, y, w) => y * w + x
const create_rect_buffer = (w, h) => new Array(w * h);

const load_tilemap = (str) => {

    const rows = str.split("\n").map(row => row.trim()).filter(row => row.length > 0);

    const w = rows[0].length;
    const h = rows.length;

    const tilemap = create_rect_buffer(w, h);
    let i = 0;

    for (let y = 0; y < h; ++y) {
        if (h != rows[y].length) {
            console.error("Tilemap is of irregular size, must be a rectangle.")
            return null;
        }

        for (let x = 0; x < w; ++x) {
            tilemap[i] = rows[y][x]
            ++i;
        }
    }

    if (i != w * h) {
        console.error("Tilemap is of irregular size.")
        return null
    }
    return tilemap
}
```


## 2.2 How To Look At Things (Ray Casting)



## 2.3 Texture Mapping








## 2.X Further Reading:
1. Lode's Computer Graphics Tutorial - Raycasting *(https://lodev.org/cgtutor/raycasting.html)*
2. Wikipedia - Digital Differential Analyzer *(https://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm))*