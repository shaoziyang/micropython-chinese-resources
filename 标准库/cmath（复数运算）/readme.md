# cmath（复数运算）

cmath 库提供了复数运算功能，可以方便的进行复数运算。但它需要硬件支持浮点运算功能（FPU），目前只有STM32和ESP32上支持cmath，而在ESP8266和WiPy（CC3200）上不支持cmath模块。

## 函数

* cmath.`cos`(z)

  计算余弦。
<br><br>

* cmath.`exp`(z)

  指数计算。
<br><br>

* cmath.`log`(z)

  计算自然对数。
<br><br>

* cmath.`log10`(z)

  计算常用对数（底数是10）。
<br><br>

* cmath.`phase`(z)

  计算相位，范围是(-pi, +pi），以弧度表示。
<br><br>

* cmath.`polar`(z)

  返回复数对应的极坐标。
<br><br>

* cmath.`rect`(r, phi)

  返回极坐标对应的复数。
<br><br>

* cmath.`sin`(z)

  计算正弦。
<br><br>

* cmath.`sqrt`(z)

  计算开平方。

 

## 常量

* cmath.`e`

  自然对数的底数（2.718282）。
<br><br>

* cmath.`pi`

  圆周率（3.141593）。

  使用上面的函数，就可以非常容易的进行各种复数计算。

  ```py
  >>> import cmath
  >>> z=1+2j
  >>> z*z
  (-3+4j)
  >>> z ** 5
  (41.00001-38j)
  >>> cmath.exp(z)
  (-1.131204+2.471727j)
  >>> cmath.phase(z)
  1.107149
  >>> cmath.sqrt(z)
  (1.27202+0.7861515j)
  ```