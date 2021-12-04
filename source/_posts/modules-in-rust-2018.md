---
title: Modules in Rust 2018
copyright_author_href: 'https://github.com/r1cebank'
date: 2020-11-09 20:03:53
categories:
- Programming
tags:
- programming
- rust
cover: /covers/rustlang.bmp
---
## Modules in Rust 2018

https://doc.rust-lang.org/nightly/edition-guide/rust-2018/module-system/path-clarity.html

#### Rust 2015 
.
├── lib.rs
└── foo/
    ├── mod.rs
    └── bar.rs

#### Rust 2018
.
├── lib.rs
├── foo.rs
└── foo/
    └── bar.rs

In top level, the same level of the `main.rs`

```rust
// module.rs

pub fn some_module_fn() {}
```

Then if you want to add more stuff in the module, helper functions, sub-modules, add them in the folder named the same name as your module.

```rust
// In ./module/shared.rs

pub fn shared_fn() {}
```

To make sure the sub module can be accessed by others, you need to add the module in the top level file as well:

```rust
// module.rs
mod shared; // This imports the sub module
```
