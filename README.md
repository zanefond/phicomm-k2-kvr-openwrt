# 斐讯 K2 (PSG1218A/B) ImmortalWrt 21.02 KVR 纯 AP 组网固件编译仓库

本项目提供基于 **ImmortalWrt 21.02.7** (`ramips/mt7620`) 的云端自动化 ImageBuilder 打包工作流，专门为“**入户光纤 - 光猫 - 多台斐讯 K2 旁路 AP**”拓扑结构量身定制。

通过精简内核与冗余驱动（移除 PPPoE、IPv6 防火墙等），腾出宝贵的 8MB Flash 空间，集成了中文 LuCI 界面以及完整的 **802.11k / 802.11v / 802.11r (KVR)** 无缝漫游协议支持。

---

## 📥 验证过的 ImageBuilder 下载地址

- **ImmortalWrt Release**: `21.02.7`
- **Target Architecture**: `ramips / mt7620`
- **Profile Name**: `phicomm_psg1218a`
- **ImageBuilder URL**:
  `https://downloads.immortalwrt.org/releases/21.02.7/targets/ramips/mt7620/immortalwrt-imagebuilder-21.02.7-ramips-mt7620.Linux-x86_64.tar.xz`

---

## 🛠️ 固件优化与软件包策略

为了在 **8MB Flash** 的斐讯 K2 上完美运行并支持快速漫游，固件做了以下精简与增强：

1. **移除冗余包（节省空间）**：
   `-ppp -ppp-mod-pppoe -ip6tables -odhcp6c -kmod-ipv6 -odhcpd-ipv6only -wpad-basic -wpad-basic-wolfssl -wpad-mini`
2. **添加核心功能包**：
   `wpad-openssl luci luci-i18n-base-zh-cn`（`wpad-openssl` 原生完美支持 802.11r 快速漫游及 WPA2/3 加密）。
3. **智能漫游支持**：
   同时尝试集成 `usteer` 智能引导守护进程（若固件超出 8MB 则自动回退至核心精简版，确保 100% 编译成功并能正常刷入）。

---

## 🌐 拓扑结构与组网教程

### 拓扑设计
```text
[入户光纤] -> [光猫 (主路由/开启DHCP, 192.168.1.1)]
                    │
                    ├─(LAN口)─ [K2 主 AP (固件默认 IP: 192.168.1.253, DHCP已关)]
                    │          SSID: Home-WiFi (2.4G & 5G KVR 开启)
                    │
                    └─(LAN口)─ [K2 从 AP (修改 IP: 192.168.1.252, DHCP已关)]
                               SSID: Home-WiFi (相同的 BSSID 漫游域 4f57)
```

### 刷机与配置步骤
1. **初次刷机**：
   - 将电脑网线连接到 K2 的 **LAN 口**（不要插 WAN 口）。
   - 电脑设置静态 IP `192.168.1.100`。
   - 通过 Breed Web 恢复控制台或不死 U-Boot 刷入编译好的 `immortalwrt-ramips-mt7620-phicomm_psg1218a-squashfs-sysupgrade.bin` 固件。
2. **登录后台**：
   - 刷机完成后，K2 的默认管理 IP 为 **`192.168.1.253`**。
   - 浏览器访问 `http://192.168.1.253`，用户名 `root`，默认密码 `12345678`。
3. **无线漫游 (KVR) 参数说明**：
   - 首次启动脚本已自动在 `/etc/config/wireless` 中配置好以下参数：
     - `ieee80211r='1'` (802.11r 快速 BSS 转换)
     - `mobility_domain='4f57'` (漫游域ID，多台 K2 必须保持一致)
     - `ieee80211k='1'` (邻居报告)
     - `ieee80211v='1'` (BSS 漫游过渡管理)
   - 您可在【网络】->【无线】中随时调整 SSID 名称、密码或信道。多台 AP 请保持 **SSID、加密方式、密码以及 Mobility Domain (漫游域)** 完全一致，以实现手机等终端在房间移动时的无缝极速切换。
