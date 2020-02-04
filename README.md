# Dell-Inspiron-7590-With-SHP14C7-Mojave
Clover EFI for Dell Inspiron 7590 with Sharp SHP14C7.      
**注意 | 本 EFI 仅供参考，系统目前已经可以满足日常使用的需要，但无线网卡尚未测试！相关的完善将在近期进行。**

# 先行说明：建立本 Repo 的原因
本机型的 4K 版本有两种屏幕，分别为友达的`AUO41EB`与夏普的`SHP14C7`。但经尝试发现后者并不像前者般容易驱动。使用常规的 `WhateverGreen` + 注入参数至 `config.plist` 的 `Devices` -> `Properties` 子项的方式并不能成功驱动后者，同时会出现 `division-by-zero` 的 Kernel Panic，并立即重启，因此无法正常进入系统。       
`division-by-zero` 这一问题理论上在新版的 `WhateverGreen` 已经可以由其本身解决，但在装有 SHP14C7 屏幕的机器上似乎并不奏效。       
经尝试后发现，该屏幕在 `10.14.2` 版本下可以使用二进制破解 `AppleIntelCFLGraphicsFramebuffer.kext` 的方法规避这一 Panic 正常进入系统。具体参数已注入本 repo 的 `config.plist`。（具体注入内容见文末）

关于 KextstoPatch 的参考文章：     
《[Coffee Lake Intel UHD Graphics 630 on macOS Mojave: A compromise solution to the kernel panic due to division by zero in the framebuffer driver](https://www.firewolf.science/2018/10/coffee-lake-intel-uhd-graphics-630-on-macos-mojave-a-compromise-solution-to-the-kernel-panic-due-to-division-by-zero-in-the-framebuffer-driver)》      
《[[FIX] Coffee Lake Intel UHD Graphics 630 on macOS Mojave: Kernel panic due to divide-by-zero](https://www.tonymacx86.com/threads/fix-coffee-lake-intel-uhd-graphics-630-on-macos-mojave-kernel-panic-due-to-divide-by-zero.261687/)》       
《[10.14-10.14.5 macOS Mojave 各平台核显DVMT Framebuffer二进制补丁](http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1795107&highlight=macOS%2BMojave%2B10.14.1)》      

关于其他硬件驱动的细节将在日后更新。

# 在 10.14.2 下驱动 SHP14C7 的可行办法
先使用 `config-install.plist` （即仿冒 IGPU ID：0x12345678）安装系统并完成初始设置，然后在终端中重建 Kext 缓存：`sudo kextcache -i /`，然后重启。      
之后再使用 `config.plist` 启动系统，如无意外应该可以正常驱动 UHD630 及本屏幕。

# 硬件配置

## 已驱动 / 已知可驱动
* Machine：Dell Inspiron 7590
* CPU：Intel Core i7-9750H
* IGPU：Intel Graphics UHD 630
* RAM：16 GB * 2 = 32 GB RAM
* Display：4K Sharp Display - Sharp SHP14C7
* SSD：WD PC SN520 NVMe WDC 512GB SSD
* Audio：Realtek ALC295（戴尔定制型号：ALC3254）
* 【计划 / 即将更换】_WLAN + Bluetooth：Broadcom DW1820A_

## 未驱动
* Intel Wireless-AC 9560（无解）
* Goodix fingerpint reader（无解）
* Nvidia Geforce GTX 1650（无解）

# 10.14.2 对应的 KextstoPatch 内容
```
<key>KextsToPatch</key>
<array>
	<dict>
		<key>Comment</key>
		<string>To implement correct brightness levels, change F%uT%04x to F%uTxxxx in AppleBacklightInjector.kext (RehabMan)</string>
		<key>Disabled</key>
		<false/>
		<key>Find</key>
		<data>
		RiV1VCUwNHgA
		</data>
		<key>InfoPlistPatch</key>
		<false/>
		<key>Name</key>
		<string>com.apple.driver.AppleBacklight</string>
		<key>Replace</key>
		<data>
		RiV1VHh4eHgA
		</data>
	</dict>
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