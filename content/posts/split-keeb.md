---
title: "My split keyboard setup"
date: "2026-08-05"
summary: "My Go60 split keyboard setup."
description: "Configuration used for my split keyboard"
toc: false
autonumber: false
math: false 
tags: ["moergo", "go60", "split-keyboards"]
showTags: true
hideBackToTop: true
hidePagination: true
---

## Swedish keys on US locale 
Personally, I find it easier to program using US layout, but to properly register the Swedish keys such as: 
- `å` 
- `ä`
- `ö`

I have had to bind them to a different layer. I'm using the symbol layer for those keys, but have bound them to the same key positions as you would expect for a Swedish keyboard.

- `å -> C6R2`
- `ä -> C6R3`
- `ö -> C5R3`

This is configured using what MoErgo layout editor calls `Custom defined behaviors`.
Add the following and then bind those key positons to the definitions:
```
behaviors {
  sv_ao: sv_ao {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&macro_press &kp LALT>,
               <&macro_tap &kp A>,
               <&macro_release &kp LALT>;
  };

  sv_ae: sv_ae {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&macro_press &kp LALT>,
               <&macro_tap &kp U>,
               <&macro_release &kp LALT>,
               <&macro_wait_time 50>,
               <&macro_tap &kp A>;
  };

  sv_oe: sv_oe {
    compatible = "zmk,behavior-macro";
    #binding-cells = <0>;
    bindings = <&macro_press &kp LALT>,
               <&macro_tap &kp U>,
               <&macro_release &kp LALT>,
               <&macro_wait_time 50>,
               <&macro_tap &kp O>;
  };
};
```



### Updating keyboard layout
Finger key positions are referred to using column/row combinations. 

E.g C3R3 refers to the key on the position `column 3, row 3`

Thumb key positons are defined using `T`. 
E.g `T3` refers to the 3rd thumb key, i.e the one furthest "out".

#### Right half first
1. Turn off the device
2. Plug in the MoErgo supplied USB cable from computer to the right half
3. Hold `T3` button and `C3R3` while also pressing down the "turn on" button.
4. MoErgo Go60 should show up in your Finder now. 
5. Drag/Transfer the `uf2` file you generated in the go60 layout editor.
6. You should get some alert about "device getting ejected" or something. This is expected.

#### Left half
Follow exact same steps as above.
