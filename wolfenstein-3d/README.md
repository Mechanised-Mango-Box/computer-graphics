# Wolfenstein 3D

The lesser known predecessor to Doom.

## Demo
[Link](./5_mobile/wolfenstein-3d.html)

## Walkthrough
### Core
1. [Setup](./1_setup/README.md)
2. [Flatworld (Minimap)](./2_flatworld/README.md)
3. [Rendering](./3_rendering/README.md)
4. [Physics](./4_physics/README.md)

### Extras
5. [Mobile Controls + Movement](./5_mobile/README.md)
6. [Multiple Textures](./6_more_texture/)

### WIP
- Scope In / Run Effect (FoV Tricks)

## Q&A
> What is this?

A walkthrough of me learning how to make a Wolfenstein 3D like game from bare pixels with interactive diagrams along the way.

It *could* work as a tutorial but I can't guarantee it will work out for you that way as **it is more a snippet of what I and doing and how/why rather than a step by step on recreating what I have done**, so if you follow along the code might not look like mine (although I do leave a copy of how my code looks at the end of each chapter). I also rename some things to be easier to type/read which is not reflected in the commentary (i.e. `buffer_index_safe()` -> `bi_s()`)

I also leave links to things that I found really helpful along the way since this stuff doesn't always show up when you search for them (if you even know what to look for).

> Why did you do this?

I am moreso visual thinker so what other programming problem is more visual than graphics but other than that... idk. was bored ig. `¯\_(ツ)_/¯`

> Why use JS?

I was going to use `raylib` with `C` or `Odin` but setting it up on Windows took longer than expected (fixing it was going to take a while and I was going to switch to Linux anyways). On the bright side it (kinda) just works on the web.

> Is JS a mistake?

Also yes. How did anyone think silent errors (not even just logging it) was a good idea is beyond me. Also no types is most definitely a choice of all time.

> Can I get the actual Wolfenstein 3D game?

- Original Source Code (not very helpful): https://github.com/id-Software/wolf3d
- Playable Emulation (unless you have a MS-DOS somehow): https://archive.org/details/wolfenstein-3d