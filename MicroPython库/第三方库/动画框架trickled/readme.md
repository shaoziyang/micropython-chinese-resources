# LED 动画框架 trickLED

Micropython 的执行可寻址 LED (NeoPixel) 动画框架。它不如 FastLED for Arduino 快速或高效，但是创建自己的自定义动画非常容易。   
   
- [https://gitlab.com/scottrbailey/trickLED](https://gitlab.com/scottrbailey/trickLED)
   
   
TrickLED 示例
```py
import uasyncio as asyncio
import trickLED
from trickLED import animations
from trickLED import generators

# use TrickLED class instead of NeoPixel
tl = trickLED.TrickLED(machine.Pin(26), 200)
tl.fill((50, 50, 50), start_pos=0, end_pos=24) # fill first 25 pixels with white
tl.fill_gradient(0xDC143C, 0xF08080, 25) # fill remaining with red gradient
tl.write()
''' 
If you have a long strip and an animation that is expensive to calculate, you can 
set repeat_n and either stripe or mirror that section to the rest of the strip. 
Once it is set on the TrickLED object, animations will automatically just calculate
that section.
'''
tl.repeat_n = 50 
tl.repeat_mode = tl.REPEAT_MODE_MIRROR # stripe is default

# Python generators are awesome, ones that generate colors are awesomer
cwg = generators.striped_color_wheel(hue_stride=5, stripe_size=1)
tl.fill_gen(cwg) # fill our strip from the color generator
tl.write()
tl.scroll(1) # move pixels forward 1 position
tl.blend_to_color(0, 50) # blend the strip with black at 50%

```

动画示例
```py
ani = animations.Fire(tl, interval=40)
# settings can also be set by passing as keyword arguments to play()
asyncio.run(ani.play(500, sparking=64, cooling=15))

```
