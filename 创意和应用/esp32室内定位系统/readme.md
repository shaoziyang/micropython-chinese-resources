# ESP32 室内定位系统

**摘要**：本文介绍了一种基于 RSSI 的室内定位系统（IPS），该系统完全基于 ESP32 微控制器硬件，使用 MicroPython 构建。三个 ESP32 接入点（TestNetwork1、TestNetwork2、TestNetwork3）在固定位置充当锚节点。移动目标 ESP32 扫描来自每个锚点的 Wi-Fi 信号强度，应用卡尔曼滤波器来抑制多径噪声，使用对数距离路径损耗模型将滤波后的 RSSI 值转换为距离估计，并使用闭合形式的三边测量来计算 2D 位置。通过使用最小二乘拟合在1、2、3和4米距离收集 50 个 RSSI 样本，对路径损耗模型进行了经验校准，得出a=-61.92 dBm，n=1.64。从三个锚节点记录的实际测量数据表明，卡尔曼滤波使锚1的RSSI标准偏差降低了57.1%，锚2降低了25.0%，锚3降低了4.1%。该系统成功地跟踪了从（1.99，0.84）米到（1.85，0.94）米的目标节点轨迹，在受约束的微控制器硬件上实现了稳定的实时定位。完整的实现是开源的，可以在 GitHub 上找到。

![](trajectory.webp)

## 要求

- ESP32
- 3个无线锚点/接入点
- MicroPython 或 Arduino IDE

## 系统架构

```
          Anchor Node 1
               *
              / \
             /   \
            /     \
           /   X   \
          / Target  \
         /           \
        *-------------*
Anchor Node 2     Anchor Node 3
```

## 数学模型

**RSSI 距离估算**

$ d = 10^{\frac{A - RSSI}{10n}} $

其中
- `d` = 估计距离
- `A` = 1米处 RSSI
- `n` = 路径损耗
- `RSSI` = 接收信号强度指示

**A值的计算**
A 的值是通过将 ESP32 设备放置在距离锚节点仅1米处并记录多个RSSI样本来计算的。

公式

$ A = \frac{\sum RSSI_i}{N} $

其中
- `RSSI_i` = 单独 RSSI 采样
- `N` = 样本总数

**路径损耗指数计算(N)**
路径损耗指数取决于室内环境,例如:
- 墙壁
- 障碍
- 反思
- 多路径干扰

典型值

| 环境 | N 值 |
|-|-|
| 自由空间 | 2.0 |
| 办公室 | 2.0 - 4.0 |
| 密集的室内环境 | 4.0 - 6.0 |

公式

$ N = \frac{A - RSSI}{10 \log_{10}(d)} $

其中

- `A` = 1米处RSSI
- `RSSI` = 距离 `d` 测量 RSSI
- `d` = 以米为单位的已知距离

## 卡尔曼滤波

$ \hat{x}_k = \hat{x}_{k-1} + K_k(z_k - \hat{x}_{k-1}) $

其中

- $ \hat{x}_k $ =  RSSI 滤波值
- `z_k` = RSSI 测量值
- `K_k` = 卡尔曼增益


## 三角计算公式

$ (x-x_i)^2 + (y-y_i)^2 = r_i^2 $


用于使用三个锚节点来估计目标节点位置。


## 相关链接

- [reddit](https://www.reddit.com/r/esp32projects/comments/1tl6ng0/built_indoor_positioning_system_on_esp32_using_3/)
- [linkedin](https://www.linkedin.com/posts/harshonweb_built-indoor-positioning-system-on-esp32-ugcPost-7462890248155611136-CUaB/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD3j6f0BKuGJORdwBDPuhNf2LW0j8wkgNMs)
- [论文](https://zenodo.org/records/20310320)
- [github 仓库](https://github.com/harshsaxena213/Indoor-Positioning-System-Application-On-ESP32/tree/main)
