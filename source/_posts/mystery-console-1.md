---
title: 神秘的游戏机 (part 1)
copyright_author_href: 'https://github.com/r1cebank'
date: 2022-01-20 15:16:53
categories:
- Gaming
- Portable
tags:
- soc
- reverse-engineering
cover: /covers/mystery-console-1.jpg
---

2021年的生日我收到了一个有趣的礼物，当时女朋友瞒着我说给我买了一个很好玩的游戏机。我的第一反应除了当时谁都买不到的PS5就是之前我有给她看过的我还没有收集到的游戏机：

* Sega Saturn
* Neo Geo Color
* Sega Gamegear
* Nintendo 64

当然在收到礼物之前我就有猜到也许她会给我买一个几百合一的模拟器游戏机，问题是是哪个呢？当时最害怕的就是什么都不懂的她会在网上被骗花了很多钱买了一个原本廉价的游戏机。

后来拿到的时候其实也没有什么惊讶，虽然不是什么主流主机，这个小东西游戏倒是真的蛮多的。而且还包括一些我之前在模拟器上无法模拟的游戏。

{% asset_img photo_2022-01-20_19-10-58.jpg Game console %}

当然，任何一个工程师第一反应就是拆开看一看这东西是怎么运行的。但是你可能会问：这玩意有什么特殊的啊？就是一个ARM处理器加上一个linux内核加上一堆开源模拟器，就算你拆开也就是一个SoC，也不会有什么值得研究的。

{% note warning simple %}
私自拆开任何电子产品都有危险，请勿模仿，本博客对可能造成的人身和财产损失概不负责。
{% endnote %}

拆开之后，原本准备就老老实实装回去，但是这个游戏机并没有我之前想的那么简单：

{% asset_img photo_2022-01-20_19-27-26.jpg Game console PCB %}

看起来挺无聊，不是吗？一个SoC然后加上一堆按钮。但是仔细观察，这个SoC的型号都被激光抹掉了，我们可以看到的就是代表PIN1的圆点而已。

{% asset_img photo_2022-01-20_15-21-44.jpg Game console PCB %}

这个神秘的SoC立刻勾起了我的兴趣，因为一般这种状态SoC可以代表几件事情：

* 回收芯片
* 工程样本

工程样本一般也不会直接把全部信息抹掉，一般都会把ES字样或者“保密”字样抹掉，有的也会直接抹掉序列号。回收芯片的确有的会直接抹掉全部信息，所以这块芯片让我觉得可能是比较老的ARM SoC （虽然这个时候我没有任何证据说它肯定是ARM，MIPS的可能性也很高，但是我对ARM SoC有些了解就这样猜了）

仔细观察了一下PCB我发现整个PCB的设计并不算复杂，而且几个重要的芯片都有标识。

{% asset_img photo_2022-01-20_19-29-46.jpg Game console PCB labeled %}

* 未知 SoC
* XM25QH64A SPI FLASH
* 24Pin LCD
* 未知芯片和晶振
* SC8002B 功放IC

这里我最好奇的就是这块SPI FLASH。这里也许存储着固件信息，或者是SoC启动需要的first stage bootloader。把这块芯片内容读取出来之后，先放到binwalk检查一下是否有什么有趣的发现：

~~~
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
766124        0xBB0AC         JPEG image data, EXIF standard
766136        0xBB0B8         TIFF image data, big-endian, offset of first image directory: 8
766802        0xBB352         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
773521        0xBCD91         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
781806        0xBEDEE         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
808412        0xC55DC         JPEG image data, EXIF standard
808424        0xC55E8         TIFF image data, big-endian, offset of first image directory: 8
809090        0xC5882         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
816252        0xC747C         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
825009        0xC96B1         Copyright string: "Copyright (c) 1998 Hewlett-Packard Company"
6080726       0x5CC8D6        bix header, header size: 64 bytes, header CRC: 0x80030000, created: 1970-01-01 00:46:56, image size: 218107136 bytes, Data Address: 0x2000100, Entry Point: 0x37F00C00, data CRC: 0x47E02C20, OS: pSOS, image name: "$"
6302815       0x602C5F        Intel x86 or x64 microcode, sig 0x40052009, pf_mask 0xf8188404, 2004-10-11, rev 0x-381dffc, size 65536
7119048       0x6CA0C8        Zip archive data, at least v2.0 to extract, compressed size: 44921, uncompressed size: 131072, name: uni-bios.10
7164010       0x6D506A        Zip archive data, at least v2.0 to extract, compressed size: 51935, uncompressed size: 131072, name: uni-bios.11
7215986       0x6E1B72        Zip archive data, at least v2.0 to extract, compressed size: 53782, uncompressed size: 131072, name: uni-bios.12
7269809       0x6EEDB1        Zip archive data, at least v2.0 to extract, compressed size: 53781, uncompressed size: 131072, name: uni-bios.12a
7323632       0x6FBFF0        Zip archive data, at least v2.0 to extract, compressed size: 28825, uncompressed size: 131072, name: 236-bios.bin
7352499       0x7030B3        Zip archive data, at least v2.0 to extract, compressed size: 86802, uncompressed size: 262144, name: 271-bios.bin
7439343       0x7183EF        Zip archive data, at least v2.0 to extract, compressed size: 23097, uncompressed size: 131072, name: aes-bios.bin
7462482       0x71DE52        Zip archive data, at least v2.0 to extract, compressed size: 29337, uncompressed size: 131072, name: ks-ng005.bin
7491861       0x725115        Zip archive data, at least v2.0 to extract, compressed size: 32458, uncompressed size: 131072, name: ks-ng045.bin
7524361       0x72D009        Zip archive data, at least v2.0 to extract, compressed size: 31869, uncompressed size: 131072, name: neodbug2.bin
7556272       0x734CB0        Zip archive data, at least v2.0 to extract, compressed size: 214035, uncompressed size: 524288, name: ng-cd.bin
7770346       0x7690EA        Zip archive data, at least v2.0 to extract, compressed size: 45814, uncompressed size: 131072, name: ngcd-old.bin
7816202       0x77440A        Zip archive data, at least v2.0 to extract, compressed size: 27345, uncompressed size: 131072, name: unk-bios.bin
7843589       0x77AF05        Zip archive data, at least v2.0 to extract, compressed size: 2741, uncompressed size: 131072, name: unk-nglo.bin
7846372       0x77B9E4        Zip archive data, at least v2.0 to extract, compressed size: 22416, uncompressed size: 262144, name: unkngsm1.bin
7868830       0x78119E        Zip archive data, at least v2.0 to extract, compressed size: 27104, uncompressed size: 131072, name: usa_2slt.bin
7895976       0x787BA8        Zip archive data, at least v2.0 to extract, compressed size: 1403, uncompressed size: 65536, name: 000-lo.lo
7897418       0x78814A        Zip archive data, at least v2.0 to extract, compressed size: 27030, uncompressed size: 131072, name: asia-s3.rom
7924489       0x78EB09        Zip archive data, at least v2.0 to extract, compressed size: 83440, uncompressed size: 131072, name: mvsbios.rom
8007970       0x7A3122        Zip archive data, at least v2.0 to extract, compressed size: 35891, uncompressed size: 131072, name: neodebug.rom
8043903       0x7ABD7F        Zip archive data, at least v2.0 to extract, compressed size: 27925, uncompressed size: 131072, name: sp-j2.rom
8071867       0x7B2ABB        Zip archive data, at least v2.0 to extract, compressed size: 44991, uncompressed size: 131072, name: vs-bios.rom
8116899       0x7BDAA3        Zip archive data, at least v2.0 to extract, compressed size: 15363, uncompressed size: 131072, name: sfix.sfx
8132300       0x7C16CC        Zip archive data, at least v2.0 to extract, compressed size: 11245, uncompressed size: 131072, name: sm1.sm1
8143582       0x7C42DE        Zip archive data, at least v2.0 to extract, compressed size: 27284, uncompressed size: 131072, name: sp-e.sp1
8170904       0x7CAD98        Zip archive data, at least v2.0 to extract, compressed size: 27156, uncompressed size: 131072, name: sp-s2.sp1
8198099       0x7D17D3        Zip archive data, at least v2.0 to extract, compressed size: 26650, uncompressed size: 131072, name: sp-s.sp1
8226318       0x7D860E        End of Zip archive, footer length: 22
~~~

binwalk能发现这么多有趣的东西，而且在最后的Zip文档还发现了游戏机的bios文件名。我们可以初步判定固件没有被加密。

然后再放到010 Editor打开。Voila！发现好东西了

{% asset_img dump.png Editor view %}

我们可以看到文件的开头有很多ASCII，比如说：

* eGON.BT0
* SUNII

用strings跑一遍就发现了更多的字符串，有的看起来像是copyright，有的貌似是文件名。

但是这个文件比较大，足足有8MB，中间还有很大全是FF或是00的空隙，所以说需要真的理解和之后修改固件，我们需要的信息远远不够。

经过一番查找我现在可以得出以下结论：

eGON.BT0 是 allwinner 公司生产SoC启动时BROM来确定存储单元是否存在一级和二级启动程序使用的Magic Number。我们可以初步认定，这个神秘SoC是allwinner公司生产的SoC。

我原本猜测这块SoC应该是 [Allwinner F1C200s](https://linux-sunxi.org/F1C200s)，这款SoC的包装方式和游戏机里的这块是一样的，并且经过对比所有的PIN也都吻合。但是这块SoC是属于suniv平台，虽然不能排除跨平台运行sunii固件，但是在信息不足的情况下只能暂时排除掉。

SUNII所在的位置正好是启动程序header保存平台信息的位置，所以说现在我们已经可以确定这个SoC就是allwinner公司生产的SUNII系列SoC。但是上网查了一下发现SUNII系列不但不支持linux，所支持的Melis系统信息对这个平台也少之又小。

| Name<br>                                                                                                                                         | Platform name |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------- |
| [Boxchip F10](https://linux-sunxi.org/index.php?title=F10&action=edit&redlink=1 "F10 (page does not exist)") aka SoChip SC9800 aka Teclast T8100 | (sunii)       |
| [Boxchip F13](https://linux-sunxi.org/index.php?title=F13&action=edit&redlink=1 "F13 (page does not exist)")                                     | (sunii)       |
| [Boxchip F15](https://linux-sunxi.org/index.php?title=F13&action=edit&redlink=1 "F13 (page does not exist)") aka SoChip SC8600 aka Teclast T7200 | (sunii)       |
| [Boxchip F18](https://linux-sunxi.org/index.php?title=F18&action=edit&redlink=1 "F18 (page does not exist)")                                     | (sunii)       |

但是经过一番查找，这类SoC的信息少得可怜，都是提供给特别廉价产品的廉价SoC，在这里F10，F15，F18的包装方式都是QFP而我们的这块是QFN88。F13没有任何信息，就算是它我也不会知道。

但是这并不代表逆向工程就要停止了，剩下的时间除了研究这个8MB文件的信息，还可以试着用UART连接看看可不可以获得更多有用的信息。

好了不说了，我得继续写Kaitai Struct了。

{% asset_img kaitai.png Editor view %}