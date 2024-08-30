# Urchin Peripheral Animation

This repo will attempt to serve as both an example and a tutorial on adding custom art and animations to your zmk firmware nice!view equipped keyboard. 

For the purposes of this tutorial I will assume you have a split keyboard with two nice!views, and like me, you've slowly tired of the balloon/mountain artwork (though beautiful) that statically displays on your right keeb and you want to spice things up a little. 

The right side of your split keyboard is called the peripheral side. For the time being, (PR's in the works?) it's pretty dumb and doesn't have access to as much data as the left side does which is why the right side only has a battery indicator, connectivity indicator and some art. This is also why the battery lasts much longer, it's simply doing much less. So without access to much data, displaying art / animations is kinda the best we can do with the peripheral side. I say this to hamper any "can I put a bongo cat on the right side that reacts to wpm?" like questions which were my first questions too. Unfortunatyl no, that's left hand business. The peripheral isn't going to react to inputs because it simply can't.

If you just want to use my custom urchin animation and don't care how to make your own, you can jump straight down to [Usage](#usage)

---

#### Table of Contents
1. [What are we making](#what-are-we-making)
2. [Getting Started](#getting-started)
3. [Making the Art](#making-the-art)
4. [Converting Your Art](#converting-your-art)
5. [Brief Explanation of the Module](#brief-explanation-of-the-module)
6. [Adding a Static Image](#adding-a-static-image)

---


## What are we making?

Ultimately we are going to implement a 12 frame animation of the unofficial Rust-lang unsafe code mascot, Corro the sea urchin.

Why? Because my main keyboard right now is [duckyb's variant of the ferris sweep that uses the nice!view displays, called the urchin](https://github.com/duckyb/urchin). It's emblazoned with a cute little corro graphic on the pcb with silk-screening 


![urchin board](https://raw.githubusercontent.com/duckyb/urchin/master/gallery/main/main-top.png)

---

*Sidenote: I bought mine from [beekeeb](https://shop.beekeeb.com/product/soldered-urchin-wireless-split-keyboard/) and I would recommend them for your nice!view equipped keyboard needs. I'm in no way affiliated with them, just a happy customer.*

---

So, I took this little dude and made a little 1bit animation that looks like this:

![urchingif](./assets/Sprite.gif)

and then I put it on the urchin board like this:

![urchinvid](./assets/20240829_081212.mp4)

If that seems like something you'd like to do, keep reading...


## Getting Started

Fork or Clone this nice!view module [https://github.com/GPeye/nice-view-mod](https://github.com/GPeye/nice-view-mod)

That repo is an exact copy of the same nice!view shield found in the official ZMK firmware repo, just wrapped up in a module.

**What's a module?** In a nutshell, it's a way of extending the ZMK firmware, allowing you to inject during build, new shields, boards, behaviors, features, drivers and more.

**But the nice!view shield is already in the firmware, why make it a module?** This way we can edit our forked/cloned module and just swap out the real nice!view shield for our custom one and we don't have to edit the official firmware.

Once you have your fork/clone, we'll need some art...


## Making the Art

*For learning the code part of this, feel free to use the assets under the assets folder in this repo and skip this step*

The current nice!view peripheral mountain and balloon images are **68px wide by 140px tall** (sort've*) so, to easily swap out for some other art, any image we want to make or use should also be that size. I use aseprite which is a paid software, but pixel art at this size can easily be made in gimp or ms paint for free, or even using in browser tools like [piskel](https://www.piskelapp.com/p/create/sprite) or [pixilart](https://www.pixilart.com/draw) which even have helpers for creating animation.

Use **only** black and white to create your art as the nice!view does not support grayscale or any colors. You may want to get familiar with dithering. Here are a couple good resources: [what is it](https://pixelparmesan.com/dithering-for-pixel-artists/) and [a massive library of 1bit patterns](https://dev.crankit.app/tools/gfxp/)

⚠️ *forewarning, converting your art to the necessary C arrays is somewhat tedious so, maybe don't make a lot of animation frames. My urchin is 12 frames and it's was bordering on annoying to do* 😅

Regardless of the tool you are using, you will want to export your art to a png format. If you created an animation you will want to export each frame as an individual png rather than as a spritesheet.

Check out the `/assets/urchinpngart` folder for my example urchin 68x140 1 bit pixel art.

## Converting Your Art

**The first step is** to take your 68x140 art and make it 140x68 by **rotating it 90 degrees clockwise**.

**Why?** the nice!view is not a vertical display, it's actually a horizontal display that's in a vertical orientation usually because it fits better on a keyboard that way. If you've ever built your firmware with the default ZMK display status screen you may have noticed this. 

Once you have rotated all your art, we need to convert it to a lvgl C array by heading to https://lvgl.io/tools/imageconverter and selecting your images, one at a time, and converting them by selecting LVGL v8, Color Format: `CF_ALPHA_1_BIT` and output format as `C array`

![lvglsetting](./assets/lvgl.png)

In the end you should end up with a bunch of C files, one for each image or frame of animation you have.

Check out the `/assets/urchincart` folder for my example c array files for the urchin pixel art.

## Brief Explanation of the Module

Feel free to skip this if you don't care but I thought it might be helpful to some to understand the files in this shield module and what they do.

<details>
<summary>Expand to see explanation</summary>

### nice_view_custom.zmk.yml
This file is the module manifest and lets the build know where to look for relevant files. In our case it just points to the boards directory where our shield is

### Kconfig.shield & Kconfig.defconfig
These files define and use the config flage used by this shield in order to determine if it applies the defined widgets. Line 5 of Kconfig.shield is where we check for the shield build flag "nice_view_custom" in order to set the config flag "SHIELD_NICE_VIEW_CUSTOM" which is then used in Kconfig.defconfig to toggle on additional configs like "NICE_VIEW_WIDGET_STATUS" which is then used in the CMakeLists.txt and custom_status_screen.c that either uses the nice!view status widgets or defaults to the built in ZMK ones.

### custom_status_screen.c
This is the entrypoint to our custom shield and if it's running the primary right hand build it runs the status.c and if it's running the peripheral right hand build it runs peripheral_status.c

### peripheral_status.c
This is the main file we are interested in. This defines the widgets and art that are used on the right hand peripheral nice!view screen

### art.c
This file exists just to hold the c array data that makes up our art for reference by the perepheral_status.c file. By default, it has the balloon and mountain image data.

</details>

## Adding a Static Image

*Even if you want to add an animation, it's good to cover how to swap out a static image first and build off that knowledge when implementing an animation*

Open the covnerted C file for the art you want and copy everything **After** the 

```c
#ifndef LV_ATTRIBUTE_MEM_ALIGN
#define LV_ATTRIBUTE_MEM_ALIGN
#endif
```
block

and paste it at the end of the `/boards/shields/nice_view_custom/widgets/art.c` file.

Near the top of what you pasted, find the following section

```c
~~~~
const LV_ATTRIBUTE_MEM_ALIGN LV_ATTRIBUTE_LARGE_CONST LV_ATTRIBUTE_IMG_CORRO01 uint8_t corro01_map[] = {
  0xff, 0xff, 0xff, 0xff, 	/*Color of index 0*/
  0x06, 0x06, 0x06, 0xff, 	/*Color of index 1*/

  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf0,
~~~~
```

and copy the #if CONFIG_NICE_VIEW_WIDGET_INVERTED block from either the balloon or the mountain art section and replace the first two "/\*Color of index\*/" lines. This will maintain the nice!view's ability to invert colors via config.

```c
~~~~
const LV_ATTRIBUTE_MEM_ALIGN LV_ATTRIBUTE_LARGE_CONST LV_ATTRIBUTE_IMG_CORRO01 uint8_t corro01_map[] = {
  #if CONFIG_NICE_VIEW_WIDGET_INVERTED
        0xff, 0xff, 0xff, 0xff, /*Color of index 0*/
        0x00, 0x00, 0x00, 0xff, /*Color of index 1*/
  #else
        0x00, 0x00, 0x00, 0xff, /*Color of index 0*/
        0xff, 0xff, 0xff, 0xff, /*Color of index 1*/
  #endif

  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf0,
~~~~
```

Next, find the name of your image const, which should be based off your original image name, by scrolling to the very bottom and finding the section that looks like this

```c
const lv_img_dsc_t corro01 = {
  .header.cf = LV_IMG_CF_INDEXED_1BIT,
  .header.always_zero = 0,
  .header.reserved = 0,
  .header.w = 140,
  .header.h = 68,
  .data_size = 1232,
  .data = corro01_map,
};
```

In this case the name is `corro01`. Copy this name for use in the next step.

Open the peripheral_status.c file and on line 28, add a new line and copy line 27, swapping out `mountain` for your new art's name. In my case it is `corro01`.

```c
#include "peripheral_status.h"

LV_IMG_DECLARE(balloon);
LV_IMG_DECLARE(mountain);
LV_IMG_DECLARE(corro01); // new line

static sys_slist_t widgets = SYS_SLIST_STATIC_INIT(&widgets);
```


## Usage

To use this module, first add it to your config/west.yml by adding a new entry to remotes and projects:

```yml
manifest:
  remotes:
      # zmk official
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: gpeye                         #new entry
      url-base: https://github.com/GPeye  #new entry
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: urchin-peripheral-animation   #new entry
      remote: gpeye                       #new entry
      revision: main                      #new entry
  self:
    path: config
```

Now simply swap out the default nice_view shield on the board for the custom one in your build.yaml file.

```yml
---
include:
  - board: nice_nano_v2
    shield: urchin_left nice_view_adapter  nice_view_custom #custom shield
  - board: nice_nano_v2
    shield: urchin_right nice_view_adapter nice_view_custom #custom shield
```
