# 通用显示和事件驱动框架 MPDisplay

MPDisplay 是 Python、MicroPython、CircuitPython的通用显示和事件驱动框架，适用于多种 Python，包括MicroPython、CircuitPython和 CPython（big Python）。

![](mpdisplay.gif)

它可以按原样用于创建应用程序的图形前端，也可以用作LVGL、MicroPython touch等GUI库的基础，甚至可以用作您一直在考虑开发的GUI框架。它的主要目的是为MicroPython提供显示和触摸驱动程序，但对于可能从未接触过MicroPython的开发人员来说，它同样有用。

需要注意的是，MPDisplay是GUI库的基础，而不是GUI库本身。它不提供小部件，如按钮、复选框或滑块，也不提供定时机制。如果需要，您将需要一个GUI库来提供这些功能，尽管许多应用程序不需要它们。

https://github.com/bdbarnett/mpdisplay