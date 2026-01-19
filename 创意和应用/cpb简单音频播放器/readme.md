# Adafruit Circuit Playground Bluefruit 简单音频播放器

用 Adafruit Circuit Playground Bluefruit 制作简单音频采样播放器。

![](fms5nyumkcun900.webp)
   
CircuitPython 拥有丰富的音频支持库，覆盖多个硬件。音频采样播放器使用 `audiocore` 库，根据主板选择 `audiopwmio` 或 `audioio`。播放 wav 文件非常简单，下面展示了 REPL 的一个示例。

```
>>> import board, digitalio, audiocore, audiopwmio
>>>
>>> speaker_enable = digitalio.DigitalInOut(board.SPEAKER_ENABLE)
>>> speaker_enable.direction = digitalio.Direction.OUTPUT
>>> speaker_enable.value = True
>>>
>>> audio_out = audiopwmio.PWMAudioOut(board.SPEAKER)
>>> audio_out.play(audiocore.WaveFile("Evillaugh.wav"))
```

音频库更多，比如 `audiomp3`（仅限 CPB）、`audiomixer`、`audiobusio`（用于 I2S 输出）和 `synthio`。adafruit\_circuitplayground 高级库提供了一些简单易用的方式来播放 wav 和 mp3 文件。
   
完整说明：

[https://www.instructables.com/Audio-Player-on-Circuit-Playground-Bluefruit/](https://www.instructables.com/Audio-Player-on-Circuit-Playground-Bluefruit/) 
