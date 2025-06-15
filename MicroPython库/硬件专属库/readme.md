# 硬件专属库

在某些情况下，以下特定于硬件/开发板的库具有与 `machine` 库中相似的函数或类。当这种情况发生时，特定于硬件专属库中的功能会提供该平台独有的硬件功能。

若要编写可移植代码，请使用 `machine` 模块中的函数和类。若要访问特定于平台的硬件，请使用相应的库，例如 Pyboard 的 `pyb` 库。

- [pyboard](pyboard/readme.md)
  - [stm](pyboard/stm/readme.md)