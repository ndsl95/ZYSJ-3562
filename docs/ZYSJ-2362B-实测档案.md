# ZYSJ-2362B（RK3562）主板实测档案

> 整理日期：2026-09-27
> 资料来源：**实物测量 + 官方规格书 + 运行中系统实测（串口 + 设备树 + 分区读取）**
> 本档案所有"实测"结论均来自这块板子本身，不是网上抄的通用参数。

---

## 1. 板卡身份

| 项目 | 内容 | 来源 |
|---|---|---|
| 型号 | **ZYSJ-2362B-V0.2** | 正面/背面丝印 |
| 生产日期 | 2024.06.21 | 丝印 |
| 厂商 | **深圳市众云世纪科技有限公司**（ZYSJ） | 规格书 |
| 厂商地址 | 深圳市宝安区西乡碧桂园凤凰智谷 A 栋 1102 | 规格书 |
| 厂商电话 | 0755-23328965 | 规格书 |
| 厂商官网 | `www.zysj-sz.com`（产品页 `productshow.asp?id=872`） | 网上调研 |
| 主控 | **Rockchip RK3562** | 实物丝印 |
| 设备序列号 | **B3J4YDPE** | `ro.serialno` |
| eth0 MAC | **3a:28:40:45:3b:74** | 实测（本地管理地址，厂商未烧 MAC） |
| 同系列型号 | ZYSJ-2362K（同平台） | 规格书 |
| 转售型号 | BSDZYSJ2362B | 转售商页面 |

**定位**：立式自助终端主板（出厂预装菜鸟驿站 / 拼多多驿站 kiosk 应用）。

---

## 2. 核心规格（官方 + 实测核对）

| 项目 | 参数 |
|---|---|
| 操作系统 | Android 13（出厂）/ Linux |
| CPU | Rockchip RK3562，22nm，4×Cortex-A53 @2.0GHz |
| GPU | ARM Mali-G52 2EE（OpenGL ES 1.1/2.0/3.2、OpenCL 2.0、Vulkan 1.1） |
| NPU | 1.0 TOPS，支持 INT4/INT8/INT16/FP16 |
| 内存 | **LPDDR4 2GB**（实测 2012696 kB）✅ |
| 存储 | **eMMC 16GB**（实测 mmcblk2 = 15388672 KB）✅ |
| 扩展存储 | TF Card |
| 网络 | **1 路以太网 10/100M（百兆）** |
| 无线 | 2.4GHz WiFi 802.11b/g/n/ac；蓝牙选配（见 §9 实测） |
| 显示输出 | 1×LVDS（单/双路，6/8位，1080P60，支持 3.3V/5V/12V 屏供电）<br>1×MIPI（最高 2560×1600@60） |
| 背光 | 板载背光控制，**12V 背光供电** |
| 音频 | 1×喇叭输出（2×5W 8R）、1×耳麦、2×麦克风输入 |
| 摄像头 | 1×MIPI 输入（500W/1300W 选配） |
| RTC | 外置电池，支持定时开关机 |
| USB | **7×USB2.0 HOST + 1×USB2.0 OTG** |
| 红外 | 1×红外接收座 |
| LED | 1×电源灯（红）、1×系统灯（蓝，默认闪烁） |
| 按键 | 复位、电源、升级（AD 键） |
| 串口 | 2×RS232 + 2×UART TTL + 1×RS485（选配） |
| 扩展 | 4×IO、1×I2C、1×AD、1×POE（选配） |
| 补光灯 | 5V/12V 可选，最大 3A |
| 尺寸 | 100mm × 70mm × 15.5mm，板厚 1.6mm，TG170 8 层，安装孔 2.5mm×4 |
| 电源 | DC12V / 5.5mm 内芯 2.1mm，2~5A，浪涌<18V，纹波<100mV |
| 工作温度 | -10 ~ 70°C |

---

## 3. 系统信息（实测）

```
内核            : Linux 5.10.157 #8779 SMP PREEMPT Mon Sep 30 11:49:09 CST 2024 aarch64
设备树 model    : Rockchip RK3562 ZYSJ 2362K V01 Board
compatible      : rockchip,rk3562-ZYSJ-2362K-v01
                  rockchip,rk3562-evb2-ddr4-v10      ← 基于瑞芯微官方 EVB2 参考设计
                  rockchip,rk3562
系统版本        : Android 13
Build ID        : rk3562_t-userdebug 13 TQ2A.230305.008.F1 eng.server.20240930.115542 release-keys
ro.product.model: MX-V3
fingerprint     : google/sailfish/sailfish:9/PQ1A.190105.004/...   ← 厂商伪装成 Pixel 1 / Android 9
ro.debuggable   : 1        （adb 以 root 运行）
ro.secure       : 1
SELinux         : permissive （不拦截，只记录）
Bootloader      : 已解锁（androidboot.verifiedbootstate = orange）
SELinux 上下文  : u:r:su:s0（提权后）
```

**内核命令行（关键部分）**

```
androidboot.selinux=permissive
buildvariant=userdebug
androidboot.serialno=B3J4YDPE
console=ttyFIQ0        earlycon=uart8250,mmio32,0xff210000   ← 调试串口 = UART0
androidboot.dtbo_idx=0
androidboot.storage=emmc
```

**出厂预装的应用（从运行日志抓到）**

- `com.cainiao.station` —— 菜鸟驿站
- `com.xunmeng.station` —— 拼多多系（多多买菜 / 驿站端）
- `android.rockchip.update.service`
- AOSP 标准应用若干

---

## 4. 分区表（GPT，从 eMMC 原始读出）

eMMC 总容量 15388672 KB ≈ **15.4 GB**

| # | 名称 | 大小 | 起始偏移 | 说明 |
|---|---|---|---|---|
| — | 保留区 | 4 MB | 0x000000 | MBR + GPT + **Rockchip loader** |
| 1 | security | 4 MB | 0x400000 | |
| 2 | uboot | 4 MB | 0x800000 | **bootloader 主体在这** |
| 3 | trust | 4 MB | 0xc00000 | **实测全 0**（ATF/OPTEE 并进 uboot） |
| 4 | misc | 4 MB | 0x1000000 | |
| 5 | dtbo | 4 MB | 0x1400000 | 无面板配置 |
| 6 | vbmeta | 1 MB | 0x1800000 | AVB 校验数据 |
| 7 | boot | 64 MB | 0x1900000 | **内核 + 板级 DTB（面板配置在这里）** |
| 8 | recovery | 96 MB | 0x5900000 | |
| 9 | backup | 384 MB | 0xb900000 | RK OTA 备用区 |
| 10 | cache | 384 MB | 0x23900000 | |
| 11 | metadata | 16 MB | 0x3b900000 | |
| 12 | frp | 0.5 MB | 0x3c900000 | |
| 13 | baseparameter | 1 MB | 0x3c980000 | 显示基础参数 |
| 14 | **super** | 3112 MB | 0x3ca80000 | 动态分区：system/vendor/product |
| 15 | **userdata** | 10945.5 MB | 0xff280000 | /data，实测 ext4 |

**文件系统**：userdata 为 **ext4**（不是 f2fs），`userdata.img` 可直接 loop 挂载查看。

---

## 5. 接口与逐脚定义

### 5.1 电源与音频

| 座子 | 间距 | 脚序 |
|---|---|---|
| 12V IN JACK | 2.0mm | 1 12V / 2 12V / 3 GND / 4 GND |
| 屏背光 LCD BL JACK | 2.0mm | 1 GND / 2 GND / 3 **LCD-ADJ**（背光调节）/ 4 **LCD-BLON**（背光开关）/ 5 12V / 6 12V |
| MIC JACK | 2.0mm | 1 GND / 2 MIC+ |
| SPEAKER OUT JACK | 2.0mm | 1 LP / 2 LN / 3 RP / 4 RN ⚠️ 文档描述疑似笔误（P 通常=正） |
| 补光灯 BL JACK | 1.25mm | 1 GND / 2 VLED（12V 输出） |
| 电池 BAT JACK | 1.25mm | 1 GND / 2 BAT+ |

### 5.2 串口与扩展

| 座子 | 间距 | 脚序 |
|---|---|---|
| **RS232 JACK** | 1.25mm | 1 GND / 2 RX4 / 3 TX4 / 4 RX2 / 5 TX2 / 6 5V ⚠️ **真 RS232 电平**（经 SP3232EEN） |
| **UART JACK（TTL）** | 1.25mm | 1 GND / 2 RX8 / 3 TX8 / 4 RX7 / 5 TX7 / 6 3.3V ✅ 可直连 USB-TTL |
| I2C JACK | 1.25mm | 1 GND / 2 SDA / 3 SCL / 4 RST / 5 INT / 6 3.3V |
| GPIO JACK | 1.25mm | 1 GND / 2 IO4 / 3 IO3 / 4 IO2 / 5 IO1 / 6 3.3V（IO1/IO2 默认高，IO3/IO4 默认低） |
| RS485 JACK（选配） | 1.25mm | 1 GND / 2 RS485-B(ttyS6-B) / 3 RS485-A(ttyS6-A) / 4 5V |
| KEY JACK | 1.25mm | 1 **AD（升级键）** / 2 RST / 3 PWR / 4 GND |
| LED/IR IN JACK | 1.25mm | 1 3.3V / 2 IR / 3 GND / 4 ADC(1.8V) / 5 LEDR / 6 LEDG |
| FAN JACK | 1.25mm | 1 GND / 2 12V(或5V) / 3 NC / 4 PWM |
| POE JACK（选配） | 1.25mm | POE1_2 / POE3_6 / POE4_5 / POE7_8 |
| USB2.0-HOST JACK ×7 | 2.0mm | 5 个为 `GND / DP / DM / 5V`，**2 个为反序** `5V / DM / DP / GND` ⚠️ |

### 5.3 LVDS（2.0mm，30 pin）

```
1-3   POWER (3.3V/5V/12V，由跳帽选择)
4-6   GND
7  TA1-   8  TA1+   9  TB1-   10 TB1+
11 TC1-   12 TC1+   13 GND    14 GND
15 TCLK1- 16 TCLK1+ 17 TD1-   18 TD1+
19 TA2-   20 TA2+   21 TB2-   22 TB2+
23 TC2-   24 TC2+   25 GND    26 GND
27 TCLK2- 28 TCLK2+ 29 TD2-   30 TD2+
```

**LVDS 屏电压跳帽（2.0mm，6 pin）**
```
1 12V / 2 LCD-VDD-IN / 3 5V / 4 LCD-VDD-IN / 5 3.3V / 6 LCD-VDD-IN
```

### 5.4 FPC MIPI LED 接口（0.3mm，31 pin，底层）

```
1-3   LED+  (背光正极)      4     GND
5-8   LED-  (背光负极)      9-10  GND
11 MiPi 2+   12 MiPi 2-   13 GND
14 MiPi 1+   15 MiPi 1-   16 GND
17 MiPi LCK+ 18 MiPi LCK- 19 GND
20 MiPi 0+   21 MiPi 0-   22 GND
23 MiPi 3+   24 MiPi 3-   25 GND
26 NC        27 RESET      28 NC
29 VDDIO 1.8V   30 VDD 3.3V   31 VDD 3.3V
```

### 5.5 其它 FPC（底层）

- **FPC I2C（触摸）0.5mm，10 pin**：`GND GND 3.3V SDA SCL GND INT RST GND GND`
- **MIPI Camera 0.5mm，30 pin**：1 NC / 2 VDD28 / 3 VDD13 / 4 VDD18 / 5 NC / 6 GND / 7 VDD28 / 8 GND / 9 SDA / 10 SCL / 11 RST / 12 PWDN / 13 GND / 14 MLCK / 15 GND / 16 DP3 / 17 DN3 / 18 GND / 19 DP2 / 20 DN2 / 21 GND / 22 DP1 / 23 DN1 / 24 GND / 25 CLKP / 26 CLKN / 27 GND / 28 DP0 / 29 DN0 / 30 GND

---

## 6. 显示子系统（实测设备树）

### 6.1 屏幕定义

```
接口      : MIPI DSI，4 lane（dsi,lanes = 4）
格式      : RGB888（dsi,format = 0）
flags     : 0x0A03（VIDEO + VIDEO_BURST + CLOCK_NON_CONTINUOUS + NO_EOTP）
分辨率    : 800 × 1280  （竖屏）
像素时钟  : 71.000 MHz
刷新率    : 71,000,000 / (908 × 1317) ≈ 59.4 Hz
compatible: simple-panel-dsi
status    : okay
```

**完整时序**

| 参数 | 水平 (h) | 垂直 (v) |
|---|---|---|
| 有效像素 active | 800 | 1280 |
| 前肩 front-porch | 52 | 15 |
| 后肩 back-porch | 48 | 16 |
| 同步脉宽 sync-len | 8 | 6 |
| 极性 active | 0 | 0 |
| de-active | 0 | |
| pixelclk-active | 0 | |

**合计**：h_total = 908，v_total = 1317

### 6.2 初始化序列（极简，无厂商私有码）

```
panel-init-sequence:
  05 C8 01 11   → type=05, delay=200ms, len=1, 0x11 = Sleep Out
  05 64 01 29   → type=05, delay=100ms, len=1, 0x29 = Display On

panel-exit-sequence:
  05 00 01 28   → Display Off
  05 00 01 10   → Sleep In
```

> **说明**：只需要标准的 Sleep Out / Display On，说明这块屏不依赖私有初始化码，**换同类通用屏容易**。

### 6.3 面板控制引脚

| 属性 | 值 | 解析 |
|---|---|---|
| `reset-gpios` | `<191 26 1>` | phandle 191 = **GPIO3**（0xffac0000），pin 26 = **GPIO3_D2**，flag 1 = **低电平复位** |
| `backlight` | phandle 204 | `pwm-backlight` 节点 |
| 各种延时 | prepare 100ms / init 100ms / enable 60ms / disable 60ms / unprepare 60ms / reset 60ms | |

### 6.4 显示路由（关键）

| 路由 | 状态 |
|---|---|
| `route-dsi` | **okay** ✅ 当前生效 |
| `route-lvds` | **disabled** ❌ |
| `route-rgb` | **disabled** ❌ |

> ⚠️ **这套固件只开了 MIPI DSI，LVDS 在设备树里是关闭的。**
> 要用 LVDS 屏必须改设备树（打开 `route-lvds`）重新编译。

### 6.5 GPIO 控制器 phandle 映射

| 控制器 | 地址 | phandle |
|---|---|---|
| GPIO0 | 0xff260000 | 60 |
| GPIO1 | 0xff620000 | 234 |
| GPIO2 | 0xff630000 | 237 |
| GPIO3 | 0xffac0000 | **191** |
| GPIO4 | 0xffad0000 | 186 |

---

## 7. 背光架构（实测结论）

### 7.1 结论

> ## 板子背光 = **12V 直供 + PWM 调光 + 开关**，**没有升压**

依据：
1. 规格书原文「板载背光控制，**12V 背光供电**」
2. `LCD BL JACK` 只提供 `12V + LCD-ADJ(PWM) + LCD-BLON(开关)` 三件套 —— 这是喂给**屏侧背光驱动**的标准接口
3. MIPI FPC 上 `LED+` 占 3 脚、`LED-` 占 4 脚 —— 是**给屏输送 12V 大电流**，不是驱动裸 LED 串

### 7.2 背光 PWM 参数（设备树实测）

| 属性 | 值 |
|---|---|
| 驱动 | `pwm-backlight` |
| status | okay |
| PWM 周期 | 25000 ns → **40 kHz** |
| PWM phandle | 0xEE（238），通道 0，极性 0（高=亮） |
| 亮度范围 | 0~255，默认 **200** |
| 亮度表 | 0,20,20,21,21,…,255 |
| `enable-gpios` / `power-supply` | **无**（PWM 直接驱动板上电路） |

### 7.3 测量记录与排除过程

在 MIPI LCD 座附近测得 **5V** 和 **11.66V** 两个电压：

```
12V ──┤SW├──[ 10µH 电感 ]──── 5V 输出
       │
     [SL 肖特基]  ← 续流二极管
       │
      GND
```

- 电感一端（SW 节点平均电压）= **11.66V**
- 电感另一端（输出）= **5V**

→ **该区域是 12V→5V 降压（Buck）电路，不是背光升压。整个区域没有 20~40V。**

元件：`100`（10µH 电感）、`SL`（肖特基续流二极管）、6 脚 IC（丝印 `620P03`/`020AP`，型号查不到）

### 7.4 对选屏的意义

> **要配的屏，背光必须是「12V 输入」类型。**

| ✅ 可用 | ❌ 不可用 |
|---|---|
| 屏自带背光驱动 IC（接受 12V + PWM + EN） | 裸 LED 长串，需要 ~30V |
| 或 LED 串仅 3 颗串联（Vf ≈ 9~10.5V），能直接吃 12V | 需要外部升压供电 |

### 7.5 背光转接板设计输入（若需自制）

**板子侧输入**

| 信号 | 来源 | 特性 | 待实测确认 |
|---|---|---|---|
| 12V | BL JACK 5/6 脚 | DC 12V | 带载能力 |
| LCD-ADJ | BL JACK 3 脚 | **PWM 40 kHz**，高=亮 | **电平 3.3V 还是 1.8V** |
| LCD-BLON | BL JACK 4 脚 | 开关 | 电平 / 极性 |
| GND | BL JACK 1/2 脚 | — | 必须共地 |

**注意事项**
- ⚠️ **40 kHz 超出很多 DIM 脚范围**（常见 20~30 kHz）→ 建议加 **RC 滤波**转模拟调光（1kΩ + 100nF，截止约 1.6 kHz）
- ⚠️ **电平匹配**：若 ADJ/BLON 为 1.8V 而驱动器门槛 2.0V，会时亮时不亮
- BLON 建议加 RC 延时（10k + 100nF ≈ 1ms），等 12V 稳定后再使能
- 背光冷启动电流可达稳态 1.5~2 倍，12V 走线和连接器留余量
- ⚠️ **EMI**：升压的开关节点和电感**必须远离 MIPI 排线**，否则花屏/闪屏

**驱动 IC 参考选型**（具体参数以数据手册为准）

| 型号 | 厂商 | 输出 | 封装 | 调光 |
|---|---|---|---|---|
| MP3302 | MPS | ~36V | SOT23-6 | PWM |
| SY7201 | Silergy | ~30V | SOT23-6 | PWM |
| AP3031 | Diodes | ~40V | SOT23-6 | PWM |
| TPS61165 | TI | ~38V | SOT23-6 | PWM |
| SGM3730 | 圣邦微 | ~38V | SOT23-6 | PWM |

---

## 8. 调试通道

### 8.1 调试串口（推荐首选）

| 项目 | 值 |
|---|---|
| 位置 | **板子背面三个裸露铜点：`TX` / `RX` / `GND`** |
| 控制器 | **UART0** @ `0xff210000`（ttyFIQ0，内核 cmdline 确认） |
| 波特率 | **1500000** 8N1 ✅（已实测） |
| 接线 | 模块 TX→板 RX，模块 RX→板 TX，**GND 必须共地** |
| 电平 | **3.3V TTL**；USB-TTL 模块跳线选 3.3V，**不要接 3.3V 供电脚** |

> ⚠️ 从 U-Boot 到内核全程可用，是**刷机失败后唯一的救命通道**，建议焊接固定。

### 8.2 进烧录 / MaskROM 模式

| 方式 | 说明 |
|---|---|
| **背面触点** | `GND / D0 / MASKROM` 三个裸铜点。断电后用镊子短接 `MASKROM`(或 `D0`) 到 `GND`，**保持短接的同时上电**，1~2 秒后松开 |
| **升级键（AD 键）** | `KEY JACK` 第 1 脚 |
| 判断 | 电脑出现 `Rockusb Device` 即成功 |

> `D0` 是 eMMC DAT0 数据线，短接它会让 SoC 读不到启动代码而回落到 MaskROM。优先用 `MASKROM` 触点。

### 8.3 网络数据通道（高速，实测可用）

**关键发现：板子入站 TCP 被 Android 防火墙挡住（连接超时），但出站完全可用。**

因此通道方向反过来：**PC 监听，板子主动连过来推数据**

```bash
# 板子端（root）
toybox nc <PC_IP> <PC_PORT> < /文件
```

实测速率 **~10.7 MB/s**（百兆以太网）。串口 1500000 只有约 150 KB/s 且会被内核日志打断。

**降低内核刷屏**（让串口输出干净）：
```bash
echo 1 > /proc/sys/kernel/printk
```

### 8.4 提权（实测可用）

```bash
su 0          # SUID 的 su（/system/xbin/su），无回显，直接进 root shell
              # 提示符从 console:/ $  变成  console:/ #
              # 验证： id  → uid=0(root) ... context=u:r:su:s0
```
> 注意：这个 su **不支持 `-c` 参数**（`su -c id` 会报 `invalid uid/gid '-c'`）。

### 8.5 ⚠️ RS232 座是高电平，不能接 TTL

板上有一颗 **`SIPEX SP3232EEN`**（RS-232 收发器，16 脚 SOIC），位于 MIPI LCD 座下方。

→ **`RS232 JACK` 的 `TX2/RX2/TX4/RX4` 是真正的 RS-232 电平（±5~10V）**，直接接 USB-TTL 会烧模块。

**分辨方法**：万用表量空闲电压 —— 约 3.3V = TTL（可接）；负压 = RS232（不能接）。

---

## 9. 无线（实测）

| 项目 | 实测结果 |
|---|---|
| SDIO 设备 | `mmc1:0001:1` |
| SDIO_ID | `024C:F179` |
| 驱动 | **`rtl8189fs`** |
| 芯片 | **Realtek RTL8189FS** |
| 频段 | **仅 2.4 GHz**，支持 802.11b/g/n |
| 5 GHz | ❌ **不支持** |
| 蓝牙 | ❌ 片上无蓝牙；`init.svc.vendor.bluetooth-1-0` 服务在跑且 `persist.bluetooth.rtkcoex=true`，但 `/sys/class/bluetooth` 为空 → **芯片多半未贴片**（规格书也标注蓝牙为"选配"） |
| 天线 | 板上有 IPEX 天线座 |

> ⚠️ 规格书写的「支持 802.11ac / WiFi5/6（选配）」与实际芯片不符，**以实测为准：只有 2.4G**。

---

## 10. 系统备份

### 10.1 备份清单（16 个文件 / 13.96 GB）

位置：`Q:\ZYSJ 3562\backup\`

| 文件 | 大小 |
|---|---|
| emmc_head32m.img | 32 MB（GPT 分区表 + Rockchip loader 区） |
| security.img | 4 MB |
| uboot.img | 4 MB |
| trust.img | 4 MB（全 0） |
| misc.img | 4 MB |
| dtbo.img | 4 MB |
| vbmeta.img | 1 MB |
| baseparameter.img | 1 MB |
| frp.img | 0.5 MB |
| metadata.img | 16 MB |
| boot.img | 64 MB（**含面板 DTB**） |
| recovery.img | 96 MB |
| super.img | 3112 MB |
| mmcblk2boot0.img | 4 MB（全 0） |
| mmcblk2boot1.img | 4 MB（全 0） |
| **userdata.img** | **10945.5 MB** |

附带：
- `MANIFEST.txt` —— 全部文件的 SHA256
- `gpt_partitions.json` —— 解析出的分区表

### 10.2 提取方式

在运行中的系统内用 root 权限 `dd` 读出，经以太网传回 PC，**全程未使用 USB OTG 口**。

### 10.3 面板配置在哪个分区

面板 DTB 在 **`boot.img`** 里（实测在 boot.img 中命中 `rk3562-ZYSJ`、`simple-panel-dsi`、`dsi,lanes`、`panel-init-sequence` 字符串）。

- `dtbo.img` 中**没有**面板配置
- `uboot.img` 中也有一份简化 DTB（供开机 logo 使用）

### 10.4 恢复思路

| 目标 | 做法 |
|---|---|
| 完全还原出厂 | 按分区逐个写回 |
| 只恢复 MIPI 屏配置 | 回刷 `boot.img`（会连带回退内核） |
| 保留新内核只换面板配置 | 从两个 boot.img 抽 DTB 互换后重新打包 |

**恢复通道**：走同一根网线反向推数据即可，**不依赖 USB OTG**。

---

## 11. 已知问题与坑

| 问题 | 说明 |
|---|---|
| **USB OTG 枚举失败（Code 43）** | 插电脑报"设备描述符请求失败"。怀疑插到了 7 个 HOST 口中的某一个（全板只有 1 个 OTG），且经过了外置 HUB |
| **2.0mm USB2.0-HOST 座脚序不一致** | 7 个座里 **2 个是 `5V/DM/DP/GND`** 反序，其余 5 个是 `GND/DP/DM/5V`，接线极易接反 |
| **喇叭 `LP/LN/RP/RN` 描述疑似笔误** | 文档把 LP 写成"左声道负极"（P 通常=Positive） |
| **`trust` / `mmcblk2boot0` / `mmcblk2boot1` 全 0** | 正常现象，bootloader 全在 `uboot` 分区 |
| **`vendor.ril-daemon` 反复崩溃** | `android.hardware.radio@1.5` 找不到 —— 本板无 4G，属无效服务，可忽略 |
| **满屏 `avc: denied`** | SELinux permissive，只记录不拦截 |
| **下载的官方固件是 LVDS 版本** | `update-rk3562(13.0.1_#8032-2362k-v1.0)-lvds1920_1080-20260304.zip`（948M）**是 LVDS 1920×1080 版本**，与板子当前的 MIPI 800×1280 不是同一套显示配置。刷完屏不亮 → 需回刷 `boot.img` |
| **面板配置与固件变体** | 2362B / 2362K 共用固件（设备树里写的是 2362K） |

---

## 12. 待办 / 未解决

- [ ] **背光 ADJ / BLON 的逻辑电平**（1.8V 还是 3.3V）—— 转接板设计前必须实测
- [ ] **目标屏的背光参数**：Vf / If / 串并数 —— 需向卖家索取
- [ ] **是否原配屏能直接对上** —— 建议问众云世纪原配型号
- [ ] USB OTG 口定位（背面对应丝印）后打通 adb
- [ ] `pwm-backlight` 的 PWM 具体挂在哪一路（phandle 238 未解析出）
- [ ] 设备树 dump 未覆盖 `pinctrl` 之外的若干顶层子树（已单独补 `pinctrl`）

---

## 13. 文件位置索引

```
Q:\ZYSJ 3562\
├── docs\
│   ├── RK3562小板ZYSJ2362B规格书20250317.pdf   ← 厂商官方规格书（原始文件，16 页）
│   └── ZYSJ-2362B-实测档案.md                   ← 本档案（唯一整理文档）
├── backup\                                      ← 出厂固件完整备份（16 文件 / 13.96 GB）
│   ├── *.img
│   ├── MANIFEST.txt
│   └── gpt_partitions.json
└── dump\                                        ← 设备树原始数据（从运行系统实时抓取）
    ├── devicetree.json                          ← 整棵设备树（1472 个属性）
    ├── devicetree_pinctrl.json                  ← pinctrl 子树（1203 个属性）
    ├── devicetree_dump.txt                      ← 原始转储
    └── panel_dt.json
```

### 板端遗留脚本（`/data/local/tmp/`，刷机会丢失）

| 文件 | 用途 |
|---|---|
| `w.sh` | 设备树递归遍历器（输出 `F:路径` + base64 内容） |
| `s.sh` / `s2.sh` / `su.sh` | 通过 `nc` 把文件推回 PC 的发送脚本 |

重新部署（刷机后）：
```bash
echo 'w(){ for f in "$1"/*; do' > /data/local/tmp/w.sh
echo '[ -d "$f" ] && { w "$f"; continue; }' >> /data/local/tmp/w.sh
echo 'echo "F:$f"; base64 "$f"' >> /data/local/tmp/w.sh
echo 'done; }' >> /data/local/tmp/w.sh
echo 'w /proc/device-tree' >> /data/local/tmp/w.sh
```

---

## 14. 资料获取与逆向方法

### 14.1 各类资料的可得性（结论）

| 想要的资料 | 网上可得性 | 最终结果 |
|---|---|---|
| 板卡规格书 / 接口图 | ⚠️ 只有厂家宣传页 + 转售商页面 | ✅ **通过厂商网盘拿到官方规格书 PDF**（16 页，含完整逐脚定义） |
| 板卡原理图 | ❌ 无公开（OEM 板只走客户渠道） | ❌ 仍未获得 |
| **31-Pin FPC 逐脚定义** | ❌ 无公开标准 | ✅ **规格书里已给出完整 31 脚定义**（见 §5.4） |
| **屏幕定义（分辨率/时序/lane）** | — | ✅ **直接从运行系统的设备树提取**（见 §6） |
| SoC（RK3562）资料 | ✅ 公开 | 芯片手册、`RK_DRM_Panel_Porting` 移植指南（RK3566/RK3562 通用） |

> **经验总结**：这块板子最终是靠「**串口进系统 → `su 0` 提权 → 读 `/proc/device-tree`**」拿全的。
> 比解包固件镜像更快、更准 —— **设备树给出的是实际生效的配置**，而不是一堆候选配置。

### 14.2 转售商页面参数对照（⚠️ 错误很多，不可信）

转售商型号 **BSDZYSJ2362B**（pcbalcd.com / Bestar，含 10.1~27 寸屏套件）：

| 项目 | 转售商页面 | 实测 / 官方 | 判定 |
|---|---|---|---|
| CPU | Cortex-A55 @1.8GHz | **Cortex-A53 @2.0GHz** | ❌ 错 |
| Android 版本 | Android 11 | **Android 13** | ❌ 错 |
| eDP 输出 | 最高 2560×1600@60 | 板子上**没有 eDP** | ❌ 错 |
| HDMI | 1 路 1080P@120 / 4K2K@30 | 板子上**没有 HDMI** | ❌ 错 |
| 尺寸 | 126.5mm × 70mm | **100mm × 70mm** | ❌ 错 |
| 串口 | 6 路 TTL | 2×RS232 + 2×UART TTL | ⚠️ 不准确 |
| USB | 1×USB3.0/2.0 + 2 个板载座 | **7×USB2.0 HOST + 1×OTG** | ⚠️ 不完整 |
| 网络 | 百兆自适应 | **百兆** | ✅ 对 |
| 音频 | 板载 8R/5W 功放 | **2×5W / 8R** | ✅ 对 |
| MIPI 与 LVDS 共用一组信号、二选一 | 是 | 设备树 `route-dsi`=okay、`route-lvds`=disabled | ✅ 思路对 |

> **结论：转售商页面参数一律不可信，以官方规格书 + 实测为准。**

### 14.3 备用路线：从固件镜像提取（本次未用，方法留存）

如果以后只有 `update.img` 而手上没有能跑起来的板子：

```
update.img
  └─ afptool / imgRePacker 解包
       ├─ boot.img      → 内核 + 板级 DTB（本板的面板配置就在这）
       ├─ resource.img  → rockchip resource 格式（.dtb + logo）
       ├─ parameter.txt → 分区表 & lcd_type 等 uboot 参数
       └─ uboot.img / trust.img / misc.img（uboot env 里常存 lcd_type）
  └─ dtc -I dtb -O dts  →  定位 panel / dsi / lvds / backlight 节点
```

**本机可复用工具**（位于 `Q:\RK3566\`）：

| 脚本 / 文件 | 用途 |
|---|---|
| `dt_decode.py` | DTB 解码 |
| `resolve_phandles.py` | 解析 `*_gpios` 的 phandle → 真实 GPIO |
| `parse_panel_seq.py` | 把 `panel-init-sequence` 解成可读的 MIPI 命令序列 |
| `collect-board*.ps1` / `dt-dump.sh` / `dt-phmap.sh` | 板端设备树采集 |
| `mipi/pinout-identification-kit.md` | FPC 脚位 8 步实测流程 |
| `vendor-inquiry.md` | 向厂家索要资料的问话模板 |
| `pdf/RK_DRM_Panel_Porting_CN.pdf` | 瑞芯微官方屏移植指南（RK3562 适用） |

> 本机**未安装** 7-Zip / binwalk / rkdeveloptool；Python 3.12 可用（含 lz4/lzma/zlib）。
> Rockchip 镜像格式简单，必要时可纯 Python 解包。

### 14.4 一条已被推翻的早期判断

早期调研认为：「**设备树拿不到 FPC 座的物理脚序**，只能找厂家要或万用表实测」。

**这条已被推翻** —— 官方规格书直接给出了 31-Pin FPC 的完整逐脚定义（§5.4），
加上设备树确认的 lane 数（4 lane）、供电轨（1.8V/3.3V），屏侧接入所需的信息已经齐了。

---

## 15. 外部来源链接

| 资料 | 链接 |
|---|---|
| 官方规格书（本地） | `docs/RK3562小板ZYSJ2362B规格书20250317.pdf` |
| 厂家官网产品页（ZYSJ-2362K） | http://www.zysj-sz.com/productshow.asp?id=872 |
| 行业媒体稿（ZYSJ2362B，众云世纪供稿） | http://www.pos580.com/news/show.php?itemid=50597 |
| 转售商规格页（BSDZYSJ2362B） | https://www.pcbalcd.com/rockchip-rk3562-android-board-bsdzysj2362b.html |
| 转售商 B2B 店铺 | https://zysj3288.b2b168.com/ |
| 瑞芯微 RK3562 芯片介绍 | http://www.rock-chips.com/a/cn/news/rockchip/2025/1125/2115.html |
| 瑞芯微 DRM Panel 移植指南（本地） | `Q:\RK3566\pdf\RK_DRM_Panel_Porting_CN.pdf` |
| 同类第三方 RK3562 硬件手册 | 创龙 SBC-TL3562 规格书、Forlinx OK3562 规格书、MYIR MYD-YR3562 |

---

*本档案由实测数据整理，未标注来源的均为实机测量结果。厂商规格书原始 PDF 见同目录。*
*原 `ZYSJ2362B-资料调研.md`（早期网上调研）已合并进本档案，原文件已删除。*
