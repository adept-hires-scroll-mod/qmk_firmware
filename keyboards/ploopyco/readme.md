# Ploopyco

* [Mouse](mouse/)
* [Trackball](trackball/)
* [Trackball Mini](trackball_mini/)
* [Trackball Nano](trackball_nano/)
* [Trackball Thumb](trackball_thumb/)
* [Adept/Madromys](madromys/)

# Customizing your PloopyCo Device

There are a number of behavioral settings that you can use to help customize your experience
|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`PLOOPY_IGNORE_SCROLL_CLICK`   |*not defined*|Ignores scroll wheel if it is pressed down.              |
|`PLOOPY_SCROLL_DEBOUNCE`       |`5`          |Number of milliseconds between scroll events.            |
|`PLOOPY_SCROLL_BUTTON_DEBOUNCE`|`100`        |Time to ignore scroll events after pressing scroll wheel.|

## DPI

You can change the DPI/CPI or speed of the trackball by calling `pointing_device_set_cpi` at any time. Additionally, there is a `DPI_CONFIG` macro that will cycle through an array of options for the DPI.  This is set to 1200, 1600, and 2400, but can be changed.  1600 is also set to the default.

To configure/set your own array, there are two defines to use, `PLOOPY_DPI_OPTIONS` to set the array, and `PLOOPY_DPI_DEFAULT`.

|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`PLOOPY_DPI_OPTIONS`|`{ 1200, 1600, 2400 }`|Sets the values that you want to be able to cycle through, and the order they are in.|
|`PLOOPY_DPI_DEFAULT`|`1`|The default index into `PLOOPY_DPI_OPTIONS` before any user selection.|

The `DPI_CONFIG` custom keycode will cycle through the values in the array, each time you hit it. And it stores this value in persistent memory, so it will load it the next time the device powers up.

## Drag Scroll

Drag Sroll is a custom keycode for the Ploopy devices that allow you to hold or tap a button and have the mouse movement translate into scrolling instead.

Nothing needs to be enabled to use this functionality.  Just add `DRAG_SCROLL` to your keymap.

### Drag Scroll Configuration

Standard from official PloopyCo firmware:
|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`PLOOPY_DRAGSCROLL_MOMENTARY`|*defined*|Makes the key into a momentary key, rather than a toggle.|
|`PLOOPY_DRAGSCROLL_INVERT`|*defined*|This reverses the direction that the scroll is performed. Note that "inverted" means "scroll down when rolling the ball down", which may not seem intuitively to be "inverted", but refers to the direction in which negation of y-axis values must be applied when translating between mouse movement and scrolling.|

New for the High-Resolution Scroll Mod:
|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`PLOOPY_DRAGSCROLL_H_COEF`|`1.0`|Scale applied to horizontal scrolling. For example, 0.5 would make horizontal scrolling half as sensitive as vertical scrolling.|
|`PLOOPY_SCROLL_DIV_OPTIONS`|`{ 0.5, 1.0, 1.5, 2.0, 4.0 }`|Array of divisors applied to movement for scrolling. In other words, scrolling sensitivity options, to be cycled through using the `SCROLL_DIV_CONFIG` custom keycode. Higher is less sensitive. Like `DPI_CONFIG`, the current setting is stored in persistent memory, so it will be saved across power cycles.|
|`PLOOPY_SCROLL_DIV_DEFAULT`|`0`|The default index into `PLOOPY_SCROLL_DIV_OPTIONS`, meaning the default scrolling sensitivity.|
|`PLOOPY_DRAGSCROLL_SCROLLOCK`|*defined*|Enables drag scroll as long as the Adept detects that scroll lock is active. This is a simple way to offload the drag scroll button from the Adept, which can reduce hand strain without resorting to drag scroll toggle.|

These are standard QMK, but are relevant:
|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`POINTING_DEVICE_HIRES_SCROLL_ENABLE`|*defined*|Needed to enable high-resolution scrolling in QMK.|
|`POINTING_DEVICE_HIRES_SCROLL_MULTIPLIER`|`120`|This is the scrolling resolution used by QMK when reporting scroll events to the host. It is the value that becomes equivalent to what was a single wheel tick with standard scrolling. This along with the selected scroll divisor from `PLOOPY_SCROLL_DIV_OPTIONS` determines scrolling sensitivity. We keep it at the QMK default of `120`, though note that the default scroll divisor is `0.5`, meaning we are actually multiplying mouse movements by 2 to increase sensitivity. This is to increase resolution with slower scroll sensitivity (we don't get to dynamically update the scroll multipler).|
|`WHEEL_EXTENDED_REPORT`|*defined*|Send wheel reports as 16-bit values instead of 8-bit values. This is essential for high-resolution scrolling.|

It is worth noting that these two defines were present in standard PloopyCo firmware, but have been removed here. Their function is handled instead by `PLOOPY_SCROLL_DIV_OPTIONS` and `PLOOPY_DRAGSCROLL_H_COEF`:

|Define                         |Default Value|Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|~~`PLOOPY_DRAGSCROLL_DIVISOR_H`~~|~~`8.0`~~|~~Sets the horizontal movement divisor to use when drag scroll is enabled.~~|
|~~`PLOOPY_DRAGSCROLL_DIVISOR_V`~~|~~`8.0`~~|~~Sets the vertical movement divisor to use when drag scroll is enabled.~~|

### Custom Keycodes Added by High-Resolution Scrolling Mod

Note that these custom keycodes are not currently known by VIA, as they are not standard for PloopyCo official firmware. In order to use them in VIA, you must select SPECIAL -> Any, and enter `CUSTOM(n)`, where `n` is the ordinal value of the custom keycode. These are specified below.

|Keycode                        |VIA Keycode  |Description                                              |
|-------------------------------|-------------|---------------------------------------------------------|
|`SCROLL_DIV_CONFIG`|`CUSTOM(2)`|Cycles through scroll sensitivty options with each press. State is saved to persistent memory, so will be saved across power cycles.|
|`HIRES_SCROLL`|`CUSTOM(3)`|Toggle high-resolution scrolling on and off. The QMK implementation of high-resolution scrolling currently does not allow for dynamic enable/disable of high-resolution scrolling (in other words, changing the scrolling resolution reported to the host dynamically). In order to emulate disablement of hires scrolling, the firmware will report scrolling events only in increments of the statically defined scrolling resolution.|
|`SCROLL_SNAP_V`|`CUSTOM(4)`|Toggles vertical scroll snap on and off. When on, horizontal scrolling is disabled. When turned on, horizontal scroll snap will be turned off.|
|`SCROLL_SNAP_H`|`CUSTOM(5)`|Toggles horizontal scroll snap on and off. When on, vertical scrolling is disabled. When turned on, vertical scroll snap will be turned off.|
