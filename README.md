# nice-view-mod
A copy of the nice!view shield from the official ZMK firmware as a ZMK module for the purposes of easily customizing.
As provided, it should function exactly like the current nice!view shield present in the ZMK firmware.

This module is meant to be added to an existing forked customized keymap repo like [this one for the Urchin board](https://github.com/duckyb/zmk-urchin) or [this one for the Chocofi](https://github.com/beekeeb/zmk-config-corne-chocofi-with-niceview) with build actions set up to build your firmware with github actions and is of course meant for boards with the nice!view. Please check for such forkable repos for your board if you do not currently have one.

This repo is meant only to serve as a starting point for customization such as adding your own art or animations. Simply fork or clone this repo and make the edits you would like and you can easily share your customizations and animations with others as they can easily include your module in thier firmware builds with only a few changes to thier west file and build.yaml.

Replace the url-base and shield name below with whatever you customize in your fork/clone.

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
    - name: nice-view-mod                 #new entry
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
