# random（随机数）

random 模块实现了伪随机数生成器（PRNG）。

**注**：

- `()`是开区间括号，不包括它们的端点。例如，(0, 1) 表示大于 0 且小于 1。在集合表示法中：(0, 1) = { x | 0< x < 1 }。 
- `[]`是闭式区间括号，包括端点。例如，[0, 1]表示大于或等于 0 且小于或等于 1。在集合表示法中：[0, 1] = { x | 0 <= x <= 1 }。
- `randrange()`、`randint()` 和 `choice()` 函数仅在 MICROPY_PY_RANDOM_EXTRA_FUNCS 配置选项已启用时才可使用。

## 随机整数

- random.`getrandbits`(n)

  返回一个 n 比特位的整数 (0 <= n <= 32)，相当于 random.randrange($2 ^ n$)。
<br><br>

- random.`randint`(a, b)

  返回 [a, b] 之间随机整数。
<br><br>

- random.`randrange`(stop)
- random.`randrange`(start, stop)
- random.`randrange`(start, stop [ , step ])

  如果不指定 start，返回 [0, stop] 之间随机数，否则返回 [start, stop] 之间随机数。 step 是可选的步距参数，如 `randrange(1, 10, 2)` 将返回 1 到 9 之间的奇数。
<br><br>

## 随机浮点数

- random.`random`()

  返回 [0.0, 1.0) 之间随机数。
<br><br>

- random.`uniform`(a, b)

  返回 [a, b) 之间随机数 N。在 $a \leq b$ 时，$a \leq N \leq b$；在 $b < a$ 时，$b \leq N \leq a$。

## 其它函数

- random.`seed`(n=None, / )

  用整数 n 作为种子初始化随机数生成器。当没有参数（或 None）时，它将（如果支持）使用真实的随机数（通常是硬件生成的随机数）初始化 PRNG。
<br><br>

- random.`choice`(sequence)

  从序列（元组、列表或任何支持下标的对象）中随机选取。
