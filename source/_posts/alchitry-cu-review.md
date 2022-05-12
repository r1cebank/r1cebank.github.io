---
title: Alchitry Cu Review
copyright_author_href: https://github.com/r1cebank
date: 2022-05-11 15:21:53
categories:
- Technology
- Reviews
tags:
- fpga
- hardware
cover: /covers/alchitry-cu.jpg
---
Around the time of the pandemic, or even before, I acquired a couple FPGA board with the intend to learn more about FPGA programming.

{% asset_img orders.png orders from sparkfun %}

The Alchitry Au is a development board based on the Xilinx Artix 7. At that stage, I wasn't really into doing coding in the Xilinx toolchain since its big and not open source.

The Alchitry Cu is a lower powered board based on the Lattice iCE40, even lattice offers their own toolset called iCECube, there are open source alternatives. (yosys, nextpnr, etc)
{% asset_img alchitry-cu.jpg alchitry cu %}

## Overview

Features:

* Lattice iCE40-HX8K FPGA - 7680 logic elements
* 79 IO pins (3.3V logic level)
* USB-C to configure and power the board
* Eight general purpose LEDs
* One button (typically used as a reset)
* 100MHz on-board clock (can be multiplied internally by the FPGA)
* Powered with 5V through USB-C port, 0.1" holes, or headers
* USB to serial interface for data transfer (up to 12Mbaud)
* Qwiic Connector
* Dimensions of 65mm x 45mm

The Alchitry io board is also a nice addition, including DIP switches, LED and segmented LED that you can use to test your circuit design.

{% asset_img alchitry-io.jpg alchitry io %}

Features:

* 4x 7-segment LED digits
* 5x momentary push buttons
* 24x LEDs
* 24x DIP switches

Lately, I've been following the tutorial from Digi-Key, this is a great introduction about FPGA, from building blinky to RISC-V. (Since I am using Alchitry-Cu, the pcf and some of the logic design has to be tailored for the Alchitry-Cu)

https://www.youtube.com/playlist?list=PLEBQazB0HUyT1WmMONxRZn9NmQ_9CIKhb

## Review

While following the show, and programming the FPGA, I have encountered couple issues with the board, and I will highlight them here.

I will first start with the pros for the board:

* Small footprint (very small, credit card sized)
* USB-C (yeah, I care about this a lot)
* Schematic is available (same can be said for most dev board)
* Program with USB
* Can be used with open source toolchain (apio)

And the cons for the board:

* iCE40-HX8K is a bit limited in terms of embedded ram
* Not enough existing peripheral could be used
* Alchitry io has hardware flaw (fixed in new v2 version)

## IO Shield Flaw

When working on the project for the push button, I realized there is no pull-up-down resisters on the board. The board internally has pull-up resistors but the switch is also hooked up to the VCC so there is no sense to use the internal pull-up resistor.

{% asset_img io_v1.png io v1 flaw %}

https://cdn.alchitry.com/docs/alchitry_io_sch.pdf

This has created a lot of pain for us, since a separate pull down simulator has to be implemented to simulate the pull down.

```verilog
// A module to emulate pull down resistors by occasionally pulling outputs low.
// Built for use with alchitry CU and early gen alchitry IO boards which didn't
// have pull down resistors in hardware

`ifndef _ALCHITRY_EMULATE_PULL_DOWN_
`define _ALCHITRY_EMUALTE_PULL_DOWN_

`default_nettype none

module emulate_pull_down #(parameter WIDTH = 1) (
    input clk,
    inout [WIDTH-1:0] in,
    output reg [WIDTH-1:0] out
  );
  
  localparam SIZE = 3'h5;
  reg [WIDTH-1:0] IO_in_enable;
  wire [WIDTH-1:0] IO_in_read;
  reg [WIDTH-1:0] IO_in_write;
  genvar GEN_in;
  generate
    for (GEN_in = 0; GEN_in < WIDTH; GEN_in = GEN_in + 1) begin
      assign in[GEN_in] = IO_in_enable[GEN_in] ? IO_in_write[GEN_in] : 1'bz;
    end
  endgenerate
  assign IO_in_read = in;
  
  
  reg [3:0] M_flip_d, M_flip_q = 1'h0;
  reg [WIDTH-1:0] M_saved_d, M_saved_q = 1'h0;
  
  always @* begin
    M_saved_d = M_saved_q;
    M_flip_d = M_flip_q;
    
    M_flip_d = M_flip_q + 1'h1;
    IO_in_write = 1'h0;
    IO_in_enable = {3'h5{M_flip_q == 1'h0}};
    if (M_flip_q > 2'h2) begin
      M_saved_d = IO_in_read;
    end
    out = M_saved_q;
  end
  
  always @(posedge clk) begin
    M_flip_q <= M_flip_d;
    M_saved_q <= M_saved_d;
  end
  
endmodule


`endif // _ALCHITRY_EMULATE_PULL_DOWN_
```

What this does is everytime we want to read the pin, we pull it to low, this would allow the pin to exit the floating state and able to function exactly like a correctly pulled down input.

In the updated version of the IO sheld, this issue has been fixed and proper pull down has been added.

{% asset_img io_v2.png io v2 flaw %}

This change can also be seen on the PCB as well

{% asset_img io_pcb.png io v2 flaw %}

If you happened to be using Alchitry Cu with the outdated IO shield. Here are some utilities I wrote during my learning, also please feel free to make any suggestions.

https://github.com/r1cebank/alchitry-cu-utils