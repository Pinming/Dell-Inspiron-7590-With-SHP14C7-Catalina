# Dell-Inspiron-7590-With-SHP14C7-Mojave
Clover EFI for Dell Inspiron 7590 with Sharp SHP14C7.      
**注意 | 本 EFI 仅供参考，系统目前已经可以满足日常使用的需要，但无线网卡尚未测试！相关的完善将在近期进行。**

# 先行说明：建立本 Repo 的原因
本机型的 4K 版本有两种屏幕，分别为友达的`AUO41EB`与夏普的`SHP14C7`。但经尝试发现后者并不像前者般容易驱动。使用常规的 `WhateverGreen` + 注入参数至 `config.plist` 的 `Devices` -> `Properties` 子项的方式并不能成功驱动后者，同时会出现 `division-by-zero` 的 Kernel Panic，并立即重启，因此无法正常进入系统。   

`Division-by-zero` 这一问题理论上在新版的 `WhateverGreen` 已经可以由其本身解决，但在装有夏普 SHP14C7 屏幕的机器上似乎并不奏效。   
    
经尝试后发现，该屏幕在 `10.14.2` 和 `10.14.3` 版本下可以使用二进制破解 `AppleIntelCFLGraphicsFramebuffer.kext` 的方法规避这一 Panic 正常进入系统。具体参数已注入本 repo 的 `config.plist`。（具体注入内容见文末）

目前个人认为，要解决在 SHP14C7 上的 `division-by-zero`，在 `WhateverGreen` 并不能发挥其预期作用的情况下，根本思路还是在于计算相关值，二进制破解 `AppleIntelCFLGraphicsFramebuffer.kext` 从而使修改的量传递至 kext 的正确位置。但该过程涉及反编 kext，个人目前能力有限，难以实现这一目标，希望有 julao 可以出手相助！ 

同时，关于其他硬件驱动的细节将在日后更新。

# 关于二进制破解 Kext 的参考文章
《[Coffee Lake Intel UHD Graphics 630 on macOS Mojave: A compromise solution to the kernel panic due to division by zero in the framebuffer driver](https://www.firewolf.science/2018/10/coffee-lake-intel-uhd-graphics-630-on-macos-mojave-a-compromise-solution-to-the-kernel-panic-due-to-division-by-zero-in-the-framebuffer-driver)》      
《[[FIX] Coffee Lake Intel UHD Graphics 630 on macOS Mojave: Kernel panic due to divide-by-zero](https://www.tonymacx86.com/threads/fix-coffee-lake-intel-uhd-graphics-630-on-macos-mojave-kernel-panic-due-to-divide-by-zero.261687/)》       
《[10.14-10.14.5 macOS Mojave 各平台核显DVMT Framebuffer二进制补丁](http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1795107&highlight=macOS%2BMojave%2B10.14.1)》

# 使用本 EFI 在 10.14.3 下驱动 SHP14C7 的可行办法
先使用 `config.plist` 并修改启动用的集显 Platform-ID：`0x12345678`，然后安装系统并完成初始设置，首次进入系统后，在 Terminal 中重建 Kext 缓存：`sudo kextcache -i /`，然后重启。      
之后再使用 `config.plist`（内置的 Platform ID：`0x3E9B0009`）启动系统，如无意外应该可以正常驱动 UHD630 及本屏幕。 

# 目前存在的 Bug
* macOS 版本**不能升级**（本 repo 中的二进制破解仅适用于 `10.14.2` && `10.14.3` ！各版本对应的 `KextstoPatch` 并不相同
* 无线网卡尚未测试
* 雷电接口无法识别，预计无法实现雷电功能；只可作为普通 USB-C 接口使用
* 内置麦克风无法使用【无解】
* `F6` && `F7` 调节亮度映射错误，对应的按键是 `Fn + S` && `Fn + B`（你怎么骂人呢！）【近期会对键盘映射做修复】
* ~~在 Windows 系统下热重启至 Mac 会导致声卡不能正常工作~~ 通过强制加载 `AppleHDA` 及使用 `SSDT-ALC295.aml` 基本可以解决该问题，当前测试下还未发现失效情况
* ~~HDMI 连接会导致 Kernel Panic~~ 通过在 `Devices` -> `Properties` 中注入接口数据使得 HDMI 连接时不崩溃，但似乎仍不输出画面【还需要进行测试及改进】
* 直接启动 FaceTime 无法正常启用摄像头，需要先启动 PhotoBooth
* 电池的容量 (Capacity) 识别错误，应为 97Wh，但实时电量显示基本准确

# 硬件配置

## 已驱动 / 已知可驱动
**Dell Inspiron 7590** with Sharp SHP14C7 4K Display
* CPU：Intel Core i7-9750H @ 2.60 Ghz (Boost to 4.50 Ghz)
* IGPU：Intel Graphics UHD 630
* RAM：Hynix DDR4 2666Mhz / 16 GB * 2 = 32 GB RAM
* Display：Sharp SHP14C7 @ 15.5' / 4K（通过 `WhateverGreen` 配合 `SSDT-PNLF.aml` 以及 `KextstoPatch` 确保正确驱动及正确且可调的背光亮度）
* SSD：WD PC SN520 NVMe WDC 512GB SSD
* Audio：Realtek ALC295（戴尔定制型号：ALC3254）（内置麦克风不能驱动）（Layout-ID = 77，选用 28 可能导致 kernel_task 占用过高而导致 CPU 高频不下）
* 【计划 / 即将更换】_WLAN + Bluetooth：Broadcom DW1820A_

## 已知不可驱动
* Nvidia Geforce GTX 1650（无解）
* Realtek Memory Card Reader（无解）
* Intel Wireless-AC 9560（WiFi 无解 / 仅蓝牙可有限度使用）
* Goodix fingerpint reader（无解）

# 10.14.2 && 10.14.3 对应的 KextstoPatch 内容
```
<key>KextsToPatch</key>
<array>
	<dict>
		<key>Comment</key>
		<string>Disable MinStolenSize</string>
		<key>Disabled</key>
		<false/>
		<key>Find</key>
		<data>
		dkZI/wUwagg=
		</data>
		<key>InfoPlistPatch</key>
		<false/>
		<key>Name</key>
		<string>AppleIntelCFLGraphicsFramebuffer</string>
		<key>Replace</key>
		<data>
		60ZI/wUwagg=
		</data>
	</dict>
	<dict>
		<key>Comment</key>
		<string>Set the number of active lanes to 4 (for laptop with 4K display) (by FireWolf)</string>
		<key>Disabled</key>
		<false/>
		<key>Find</key>
		<data>
		i5bAJQAAio6VIwAAD7aG
		</data>
		<key>InfoPlistPatch</key>
		<false/>
		<key>Name</key>
		<string>AppleIntelCFLGraphicsFramebuffer</string>
		<key>Replace</key>
		<data>
		uAQAAACJhrwlAAAxwF3D
		</data>
	</dict>
</array>
```