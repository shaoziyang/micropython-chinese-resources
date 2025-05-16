# math（数学函数）

math模块提供常用的数学计算函数，包括指数、对数、三角函数等。在PyBoard中（STM32系列），默认使用了32位精度的浮点数。

## 函数

- math.`acos`(x)

  计算反余弦。
<br><br>

- math.`acosh`(x)

  计算反双曲余弦。
<br><br>

- math.`asin`(x)

  计算反正弦。
<br><br>

- math.`asinh`(x)

  计算反双曲正弦。
<br><br>

- math.`atan`(x)

  计算反正切。
<br><br>

- math.`atan2`(y, x)

  计算 y/x 反正切。
<br><br>

- math.`atanh`(x)

  计算反双曲正切。
<br><br>

- math.`ceil`(x)

  向上计算整数部分。
<br><br>

- math.`copysign`(x, y)

  返回 x，并带有 y 的符号位。比如：
  
  ```
  >>> math.copysign(12,2)
  12.0
  >>> math.copysign(12,-2)
  -12.0
  >>> math.copysign(-12,2)
  12.0
  ```
<br>

- math.`cos`(x)

  计算余弦。
<br><br>

- math.`cosh`(x)

  计算双曲余弦。
<br><br>

- math.`degrees`(x)

  弧度转为角度。
<br><br>

- math.`erf`(x)

  返回误差函数。
<br><br>

- math.`erfc`(x)

  返回余误差函数。
<br><br>

- math.`exp`(x)

  计算指数。
<br><br>

- math.`expm1`(x)

  返回 exp(x) - 1。
<br><br>

- math.`fabs`(x)

  计算绝对值。
<br><br>

- math.`floor`(x)

  向下计算整数部分。
<br><br>

- math.`fmod`(x, y)

  计算浮点余数。
<br><br>

- math.`frexp`(x)

  分解浮点数为尾数和指数。返回结果是元祖格式 (m, e)，对应关系是 $ x =  m * 2 ^ e $。 如果 x == 0 就返回 (0.0, 0), 否则 $ 0.5 \leq abs(m) < 1 $。
<br><br>

- math.`gamma`(x)

  计算伽马函数。
<br><br>

- math.`isfinite`(x)

  如果是有限数返回 True。
<br><br>

- math.`isinf`(x)

  如果是无穷大返回 True。
<br><br>

- math.`isnan`(x)

  如果不是数字返回 True。
<br><br>

- math.`ldexp`(x, exp)

  返回 $ x * 2 ^ {exp} $。
<br><br>

- math.`lgamma`(x)

  返回伽马函数的自然对数。
<br><br>

- math.`log`(x)
- math.`log`(x, base)

  计算自然对数（e为底数）。如果有两个参数，返回以给定 base 为底的对数。
<br><br>

- math.`log10`(x)

  计算常用对数（10为底）。
<br><br>

- math.`log2`(x)

  计算2为底的对数。
<br><br>

- math.`modf`(x)

  浮点数分解为小数和整数的元组，小数在前。
<br><br>

- math.`pow`(x, y)

  计算指数，也可以用 \*\* 进行计算。
<br><br>

- math.`radians`(x)

  角度转换为弧度。
<br><br>

- math.`sin`(x)

  计算正弦。
<br><br>

- math.`sinh`(x)

  计算双曲正弦。
<br><br>

- math.`sqrt`(x)

  计算开平方。
<br><br>

- math.`tan`(x)

  计算正切。
<br><br>

- math.`tanh`(x)

  计算双曲正切。
<br><br>

- math.`trunc`(x)

  取整数部分。


## 常量

- math.`e`

  自然对数的底数（2.718282）。
<br><br>

- math.`pi`

  圆周率（3.141593）。
