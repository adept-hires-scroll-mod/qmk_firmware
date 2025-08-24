# Ploopy Adept High-Resolution Scroll Mod

This is a fork of [QMK Firmware](https://github.com/qmk/qmk_firmware) with changes to make high-resolution scrolling practical on the [Ploopy Adept](https://ploopy.co/adept-trackball/) trackball, along with other scrolling-related quality-of-life improvements.

Though [official builds](https://blog.ploopy.co/high-resolution-scrolling-for-the-adept-257) for the Ploopy Adept have been shared that enable high-resolution scrolling, there are some issues affecting usability that this mod aims to correct:

* Scrolling data needs to be sent using 16-bit values rather than 8-bit values to avoid overflows that can cause jittery/slow scrolling.
* When high-resolution scrolling is enabled, scrolling data can be sent to the host at a very high rate if not throttled. This can lead to the host OS and/or application software being unable to process the high quantity of scrolling messages generated. Angry beeps and poor scrolling experience will result.

Fixes for these issues are implemented here. Credit for identifying them as necessary belongs to [@eynsai](https://github.com/eynsai) (along with the addition of high-resolution scrolling to QMK, making any of this possible), in [this comment](https://github.com/qmk/qmk_firmware/issues/17585#issuecomment-2287758137) and [this comment](https://github.com/qmk/qmk_firmware/issues/17585#issuecomment-2298792294), along with the warnings included in [the docs for QMK high-resolution scrolling](https://docs.qmk.fm/features/pointing_device#high-resolution-scrolling).

Like the official builds (and other firmware using the current implementation of QMK high-resolution scrolling), Mac OS is not currently supported. Though Linux should work, it has not been tested.

## New custom keycodes to tune scrolling

Some additional features are implemented to improve flexibility and work around scenarios in which high-resolution and/or two-axis scrolling is not handled well by software. New custom keycodes have been added to access these features, but are not assigned to any buttons by default.

Note that these custom keycodes are not currently known by [VIA](https://usevia.app/), as they are not standard for PloopyCo official firmware. In order to use them in VIA, you must select SPECIAL -> Any, and enter `CUSTOM(n)`, where `n` is the ordinal value of the custom keycode. These are specified below, along with the keycode identifier to be used if building firmware with a static QMK keymap.

|Keycode                        |VIA Keycode  |Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`SCROLL_DIV_CONFIG`|`CUSTOM(2)`|Cycles through a few scrolling sensitivity presets. Selected sensitivity will be maintained through power cycles.|
|`HIRES_SCROLL`|`CUSTOM(3)`|Toggles between high-resolution and low-resolution scrolling (effectively). It will always default to high-resolution on power up. This can be used to work around software that doesn't respect scrolling resolution (for example, scrolling may be way too fast).|
|`SCROLL_SNAP_V`|`CUSTOM(4)`|Toggles vertical scroll snap on and off. When on, horizontal scrolling is disabled. When turned on, horizontal scroll snap will be turned off.|
|`SCROLL_SNAP_H`|`CUSTOM(5)`|Toggles horizontal scroll snap on and off. When on, vertical scrolling is disabled. When turned on, vertical scroll snap will be turned off.|

## Activate drag scroll using scroll lock

One additional feature is to activate drag scroll as long as the Adept detects that scroll lock is active. This is a simple way to offload the drag scroll button from the Adept, which can reduce hand strain without resorting to drag scroll toggle.

Repurposing the states of keyboard "LEDs" such as caps lock, num lock, scroll lock, etc., is one of the few ways to externally signal a QMK device without resorting to custom raw HID messaging (which requires software running on the host to pass messages between devices). Scroll lock has few actual uses in the modern age, therefore it makes sense to use it for scrolling with the Adept.

This feature is enabled by default when cloning the repository, but can be disabled by removing the following line from [madromys/config.h](keyboards/ploopyco/madromys/config.h):

```c
#define PLOOPY_DRAGSCROLL_SCROLLOCK
```

If you use additional QMK devices (such as a keyboard), it is possible to set up a custom keycode to activate scroll lock only while a button is held down, effectively implementing momentary drag scroll using a button external to the Adept. This does require building with custom code. For example, you could add this to your keymap.c, and then assign `SCRL_HOLD` to a key:

```c
enum custom_keycodes {
    SCRL_HOLD = SAFE_RANGE,
};

bool process_record_user(uint16_t keycode, keyrecord_t *record) {
    if (keycode == SCRL_HOLD) {
        if (record->event.pressed) {
            if (!host_keyboard_led_state().scroll_lock) {
                tap_code(KC_SCRL);
            }
        } else {
            if (host_keyboard_led_state().scroll_lock) {
                tap_code(KC_SCRL);
            }
        }
        return false;
    }
    return true;
}
```

Another option is to use something like [AutoHotkey](https://www.autohotkey.com/) to perform the same task. Here is an example keybinding to activate scroll lock while holding down caps lock, but activate caps lock normally when tapping it:

```ahk
CapsLock::
    start_ms := A_TickCount
    SetScrollLockState, On
    KeyWait, CapsLock
    SetScrollLockState, Off
    if ((A_TickCount - start_ms) < 500) {
        SetCapsLockState, % GetKeyState("CapsLock", "T") ? "Off" : "On"
    }
    Return
```

## Firmware binaries

See the [latest release](https://github.com/adept-hires-scroll-mod/qmk_firmware/releases/latest) for binaries to flash. These will be in the form of ".uf2" files. See [instructions for flashing](#flashing-firmware-to-your-ploopy-adept) below.

There are binaries with and without the [scroll lock](#activate-drag-scroll-using-scroll-lock) feature enabled.

## Building firmware from source

If you are new to QMK, see the [setup guide](https://docs.qmk.fm/newbs_getting_started). You won't need to install QMK Toolbox for flashing the Adept, but you will need QMK MSYS. In order to automatically clone this repository and use it in your QMK build environment, run the setup command as:

```shell
qmk setup adept-hires-scroll-mod/qmk_firmware
```

If you already have a QMK build environment set up, you can add this fork as a new remote and check out the master branch (or merge the master branch with your own as desired). For example:

```shell
git remote add adept_hires https://github.com/adept-hires-scroll-mod/qmk_firmware.git
git fetch adept_hires
git checkout adept_hires/master
```

Though standard VIA keymaps have been removed from the QMK firmware repository (they are [available still](https://github.com/the-via/qmk_userspace_via) as a [userspace](https://docs.qmk.fm/newbs_external_userspace)), this fork includes an Adept VIA keymap (copied from the standard one in the external userspace) for convenience.

To build with VIA support:

```shell
qmk compile -kb ploopyco/madromys/rev1_001 -km via
```

To build without VIA support (and default keymap):

```shell
qmk compile -kb ploopyco/madromys/rev1_001 -km default
```

You may also create your own static keymap using standard QMK methods, of course. Refer to the [QMK documentation](https://docs.qmk.fm) for help.

For additional firmware configuration options (drag scroll toggle, scroll inversion, additional DPI or scroll sensitivity settings, etc.), see [here](keyboards/ploopyco/).

## Flashing firmware to your Ploopy Adept

This part is easy. Your firmware binary will be a ".uf2" file, which you have either built yourself or downloaded. Unplug your Adept from USB. Plug it back in while holding down the lower-left button. You may release the button after plugging it in. Your computer should recognize that a mass storage device was connected. Copy your ".uf2" file into the root of the mass storage device. If successful, it will auto-eject the mass storage device and the Adept will reboot with your new firmware running.

More detailed instructions with additional tips are [here](https://ploopyco.github.io/adept-trackball/appendices/programming) if you get stuck.
