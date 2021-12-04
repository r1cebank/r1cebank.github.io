---
title: How Pokemon Red implemented flashing stars
copyright_author_href: 'https://github.com/r1cebank'
date: 2020-12-07 19:55:07
categories:
- Programming
tags:
- programming
- rust
- gameboy
- emulator
cover: /covers/gameboy.bmp
---
I've been writing my gameboy emulator in rust and I had a lot of issues getting this startup screen to display properly, later I've realized it was achieved using the clever pallet swapping in runtime. 

{% asset_img ougKFey.gif gamefreak startup screen %}

For pokemon red, the start screen flashing start is made possible by changing palette during runtime.

```rust
let palette_num = if sprite.use_palette_1 {
                            self.op1
                        } else {
                            self.op0
                        };
```
By changing the pallet from op1 to op2, we are able to achieve that.

The emulator project (poorly written), can be found [here](https://github.com/r1cebank/rgb)