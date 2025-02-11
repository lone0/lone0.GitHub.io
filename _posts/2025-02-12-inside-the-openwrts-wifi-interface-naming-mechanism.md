### **Inside the OpenWrt’s WiFi Interface Naming Mechanism**

#### **Introduction**

When using OpenWrt as a wireless router, attentive users might notice an notable behavior during system startup: the WiFi interface undergoes a renaming process. For instance, the system log might display:

```sh
phy0-ap0: renamed from wlan0
```

This suggests that when OpenWrt loads the WiFi driver, a default `wlan0` interface is created, but it is then renamed by the system. This renaming is not accidental; rather, it is a deliberate mechanism in OpenWrt for managing WiFi interfaces.

Why does OpenWrt rename these interfaces? How is this renaming implemented? What components are involved? This article will dive deep into OpenWrt’s WiFi interface naming mechanism and analyze its underlying implementation in detail.

---

#### **WiFi Interface Naming Rules**

By default, when the Linux kernel loads a WiFi driver, it automatically creates network interfaces such as `wlan0`, `wlan1`, etc. While this naming convention is sufficient for standard devices like laptops, it is inadequate for OpenWrt-based routers.

Modern routers typically feature multiple wireless bands (e.g., 2.4GHz, 5GHz, 6GHz). In OpenWrt, these bands are represented as **radios** (`radio0`, `radio1`, `radio2`). Additionally, each `radio` can host multiple SSIDs (wireless networks). The conventional `wlanX` naming scheme fails to convey this hierarchical structure effectively.

To address this, OpenWrt introduced a structured **phyX-apY** naming convention:

- `phyX` represents the physical WiFi device (`radio`), such as `phy0` or `phy1`, corresponding to different wireless chipsets or frequency bands.
- `apY` represents an access point (SSID) created on a specific `phyX`, such as `ap0` or `ap1`.

For example:

- `phy0-ap0`: The first access point (SSID) created on `phy0`.
- `phy1-ap1`: The second access point (SSID) created on `phy1`.

This structured naming convention allows OpenWrt to manage WiFi interfaces more clearly, especially in scenarios involving multiple SSIDs and frequency bands.

---

#### **SoftMAC vs. FullMAC WiFi Drivers**

To understand interface renaming in OpenWrt, we need to distinguish between **SoftMAC** and **FullMAC** WiFi drivers.

- **SoftMAC (Software MAC) Drivers**: The MAC layer is implemented in the kernel (mac80211 subsystem). The kernel is responsible for handling packet management, encryption, and interface creation. Common SoftMAC drivers include ath9k, ath10k, and mt76.
- **FullMAC (Hardware MAC) Drivers**: The MAC layer is handled by the firmware running inside the WiFi chip itself. The driver acts as a bridge between the firmware and the system. Common FullMAC drivers include Broadcom’s brcmfmac and Realtek’s rtl8821cu.

---

#### **Triggers for Interface Renaming or Creation**

At what stage during OpenWrt’s startup does WiFi interface renaming or creation occur?

First, it's important to understand that the **mac80211** subsystem in the Linux kernel **automatically creates interfaces** (e.g., `wlan0`) when initializing WiFi devices. However, OpenWrt **does not want the kernel to create interfaces automatically**; instead, it wants to determine when and how interfaces are created based on user configuration.

To achieve this, OpenWrt **patches mac80211**, preventing the automatic creation of interfaces at the kernel level for SoftMAC drivers. This ensures that OpenWrt has full control over interface creation. However, FullMAC drivers **still create default interfaces** automatically (e.g., `wlan0`), necessitating a renaming process to conform to OpenWrt’s `phyX-apY` naming scheme.

The renaming or creation process takes place during the execution of **netifd (Network Interface Daemon)**, and it is primarily handled by **ucode scripts** (a JavaScript-like scripting language introduced in OpenWrt 22.03).

The **WiFi interface renaming or creation workflow** is as follows:

1. **When netifd starts**, it loads WiFi configurations and executes the `mac80211.sh` script.
2. The `mac80211.sh` script invokes the function `drv_mac80211_setup()` to initialize the wireless interface.
3. This process makes a call to **ubus** (OpenWrt’s inter-process communication mechanism) to invoke the `hostapd.uc` ucode script, executing its method `config_set()`.
4. Inside `config_set()`, the process enters a state machine called `pending`, which subsequently calls `wdev_create()`, ultimately triggering a **netlink operation** to either create or rename the interface.
5. If a **new interface is being created**, `nl80211.request(NL80211_CMD_NEW_INTERFACE, ...)` is issued to create the interface.
6. If the interface is **being renamed**, `rtnl.request(rtnl.const.RTM_SETLINK, ...)` is issued to change the interface name.

At this stage, the `phyX-apY` formatted interface is successfully created, and `hostapd` (the user-space WiFi management service) begins execution, utilizing `phyX-apY` as the primary interface for wireless access points.

---

#### **The Role of ucode in WiFi Interface Management**

Prior to OpenWrt 22.03, many network management functions were implemented using Shell scripts, such as `mac80211.sh` for WiFi configuration. However, Shell scripts are not well-suited for handling complex logic (such as asynchronous events and inter-process communication). To address this, OpenWrt introduced **ucode** in version 22.03 and significantly expanded its capabilities in version 24.10/master.

##### **Key Features of ucode**

- **JavaScript-like syntax**, making it more readable and maintainable.
- **Higher execution efficiency**, with a 64K byte interpreter, suitable for resource-constrained embedded devices
- **Rich bindings**, including support for **ubus, netlink, and socket APIs**, allowing direct interaction with system resources.

##### **Applications of ucode in WiFi Management**

- Manages **WiFi interface creation and renaming**, replacing portions of Shell scripts.
- Uses **ubus** to communicate with `hostapd` and `wpa_supplicant`, managing WiFi connections.
- Directly interacts with the **nl80211** kernel subsystem via **netlink**, enabling interface creation, deletion, and renaming.

---

### **Final Thoughts**

OpenWrt’s structured approach to WiFi interface management is more than just a cosmetic renaming process—it reflects a deeper architectural choice. The shift from traditional Shell scripts to **ucode** demonstrates OpenWrt’s commitment to more maintainable and scalable network management. Meanwhile, the **phyX-apY** naming scheme offers clarity in complex multi-radio, multi-SSID scenarios.

Whether you're hacking OpenWrt for customization or just curious about its internals, understanding how WiFi interfaces are created and managed under the hood gives you a new appreciation for the design choices that power this open-source router platform.

