# Erisin-ES8508P-FC
Technical documentation for Headunit to Opel/Vauxhall

Erisin / Vastking A007 Platform Notes

Reverse engineering notes for the Vastking A007 / Unisoc UMS512 Android head unit platform.

## Hardware

| Property | Value |
|----------|-------|
| Platform | UMS512 |
| Board | A007 |
| Device | KingPad_SA10_EEA |
| CPU | Unisoc |
| Android | 10 (reports as Android 13) |
| Build Type | userdebug |
| SELinux | Permissive |
| Baseband | FM_BASE_19C_W20.42.1_P4 |

---

## Interesting observations

- Build type is `userdebug`
- SELinux is `Permissive`
- OTA updates supported
- Wireless ADB can be enabled via hidden factory password
- Factory passwords are property-driven
- Platform uses extensive Android SystemProperties

# Hidden Factory Codes

| Code | Function |
|------|----------|
| 5555 | Enable ADB over TCP |
| 55550 | Disable ADB |
| 0129 | Show Debug Tool |
| 5555321 | MCU Reset ARM |
| 147258 | Enable Remote Debugging |
| 741852 | Disable Remote Debugging |
| 112233 | Open Car File Manager |
| 99990 | Logger |
| 99991 | GPS Helper |
| 99992 | Engineer Mode |
| 99993 | Switch Bluetooth Mode |
| 99994 | SPRD MMI Test |
| 99995 | SPRD MES Test |

# Interesting System Properties

persist.car.factory_psw
persist.car.custom_password
persist.car.setting_debug
persist.adb.tcp.port
service.adb.tcp.port
persist.car.memory_test

These properties are accessed via Android SystemProperties and override many default passwords and behaviour.

# Factory Password Logic

Unlike many Android head units, factory passwords are **not hardcoded**.

The factory password is loaded as follows:


Where
```java
PASSWORD_FACTORY =
    SystemProperties.get(
        PropDefine.PROP_FACTORY_PSW,
        PASSWORD_FACTORY);
```    
Default Password:
0000

can therefore be overridden through:

persist.car.factory_psw

This explains why different vendors ship different factory passwords while using identical APKs.

```
---# Wireless ADB```md# Wireless ADBFactory Code:
```

5555

```
Implementation:```javaPropDefine.setPropAndSync(    "service.adb.tcp.port",    "5555");PropDefine.setPropAndSync(    "persist.adb.tcp.port",    "5555");PropDefine.setPropAndSync(    "ctl.restart",    "adbd");
```

The factory app displays the current WiFi IP after restarting adbd.

When no WiFi is connected:

```
OK: 0.0.0.0
```

This is expected behaviour.

# Reverse Engineering Findings
The following packages have been inspected.- Car Settings- FactorySettingFragment- PlatformSupportHelper- Car Provider- Car Assistant- OTA ComponentsInteresting architecture:Car Launcher    │    ▼Car Settings    │    ▼FactorySettingFragment    │    ▼SettingManager    │    ▼MCU / CAN / System Services
