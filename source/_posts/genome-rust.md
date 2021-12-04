---
title: StyleGAN2 generation with Genome
date: 2020-06-24 23:12:37
copyright_author_href: https://github.com/r1cebank
categories:
- Programming
tags:
- programming
- rust
cover: /covers/genome-rust.jpeg
---

Previously I have written a simple library called genome using Javascript [repo](https://github.com/r1cebank/genome-legacy). The idea came to me when I wanted to build something like cryptokittie (crypto cat breeding and trading). I wanted a way to generate random values that can be used to generate random things, also the ability to merge these values. The project is really crude, it basically stores 4 markers and 1 influence as number. When merging two DNA together, there is random change that one gene might mutate. There are 5 type of mutations:

* delete (set to zero)
* reversal (reverse)
* duplication
* shift
* new value

I also added the ability to tell if one genome is probably related with another. But the project didn't take off and the library is quickly rendered useless. I just have it sitting there, until now.

I've been experimenting with StyleGAN implementations from Nvidia, it has some amazing abilities to generate life like images.

{% asset_img 688100.jpeg GAN generated images %}

Another interesting thing about it is the latent vector that is feeded to the generator can be used to control the style of the output. So why not use the genome library and generate latent vectors for the model?

So since I am learning rust now, I decided to build this using Rust. The final repo is here: [repo](https://github.com/r1cebank/genome)

It basically using the same idea as the JS one, but this time I added the ability to turn the DNA into latent vector for use in the GAN.

```rust
use genome::DNA;
let dna = DNA::new(2, 2);
let dna_string = dna.to_string();
```

The above code give you a DNA sequence with 2 genes and each gene have 2 markers (+1 influence), this gives you a string like this:

`c02395b6000200023e416a34bf22dcea3e8fb5893f79ae22bfafbdddbffef0d4`

I am still training the network, I am not sure if this will work with StyleGAN but lets keep fingers crossed.
