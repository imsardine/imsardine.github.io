---
title: lsusb 指令的用法
date: 2016-11-06 20:41:27
tags: [linux, command]
---

`lsusb` 可以列出連接的 USB 裝置。

    lsusb [-v]

例如：

```
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 002: ID 80ee:0021 VirtualBox USB Tablet
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```

<!-- more -->

> <i class="fa fa-lightbulb-o fa-3x"></i>
> OS X 沒有 `lsusb` 指令，可以用 Homebrew 安裝 [jlhonora/lsusb][] 進行擴充：
>
>     brew update && brew tap jlhonora/lsusb && brew install lsusb
>
> 用法大致上跟 Linux 的版本相同 (背後利用 OS X 內建的 `system_profiler SPUSBDataType`)，細節可以參考 `man lsusb`

 [jlhonora/lsusb]: https://github.com/jlhonora/lsusb

## 解讀資訊

每一行都代表一個 USB 裝置，包含 4 項資訊：

        <1>        <2>        <3>       <4>
         |          |          |         |
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

 1. 裝置所在的 USB bus 編號；一個 USB bus 可以連接多個裝置，每個裝置有不同的編號。(裝置編號從 002 開始，001 代表 USB bus 自己)
 2. 裝置在該 USB bus 下的編號 (device number)。
 3. 冒號前後分別是裝置的 vendor ID (製造商) 與 product ID (產品)。
 4. Vendor ID 與 product ID 對應的名稱。例如這裡的 [`1d6b`][1d6b] 代表的是 Linux Foundation，而 [`0002`][1d6b:0002] 則代表 2.0 root hub。

> <i class="fa fa-lightbulb-o fa-3x"></i>
> Vendor ID 及 product ID 所代表的名稱，可以到 [The USB ID Repository][id-repo] 查詢。

 [1d6b]: https://usb-ids.gowdy.us/read/UD/1d6b
 [1d6b:0002]: https://usb-ids.gowdy.us/read/UD/1d6b/0002
 [id-repo]: http://www.linux-usb.org/usb-ids.html

在 Linux，每一個 USB bus 都用一個虛擬裝置 root hub 來表示，它的 device number 固定是 001，vendor ID 固定是 `1d6b` (Linux Foundation)，且 product ID 本身就大概能看出 USB bus 的版本：

 * `0001` - 1.1 root hub 即 USB 1.1, Full Speed (12 Mbit/s)
 * `0002` - 2.0 root hub 即 USB 2.0, High Speed (480 Mbit/s)
 * `0003` - 3.0 root hub 即 USB 3.0, SuperSpeed (5 Gbit/s)

> <i class="fa fa-lightbulb-o fa-3x"></i>
> 由於 root hub 是虛擬裝置，搭配下面提到的 `-t` 比較有用，所以平常可以搭配 `grep` 將它們濾除。例如：
>
> ```
> $ lsusb | grep -v 'root hub'
> Bus 002 Device 002: ID 80ee:0021 VirtualBox USB Tablet
> ```

## 裝置詳細資訊

加 `-v` (`--verbose`) 可以顯示裝置的細部資訊，通常會搭配 `-s` 或 `-d` 指定裝置：

    lsusb -v -s <BUS_NUMBER>:<DEVICE_NUMBER>
    lsusb -v -d <VENDOR_ID>:<DEVICE_ID>

以上面的 VirtualBox USB Tablet 為例，可以用 `-s 2:2` 或 `-d 80ee:0021` 指定，效果是一樣的：

```
$ lsusb -v -d 80ee:0021

Bus 002 Device 002: ID 80ee:0021 VirtualBox USB Tablet
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               1.10
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0
  bDeviceProtocol         0
  bMaxPacketSize0         8
  idVendor           0x80ee VirtualBox
  idProduct          0x0021 USB Tablet
  bcdDevice            1.00
  iManufacturer           1
  iProduct                3
  iSerial                 0
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2 
...
```

## 階層關係

若想要看出 USB bus 跟裝置間的階層關係 (hierarchy)，可以搭配 `-t` 使用：

    lsusb -t

例如：

```
$ lsusb -t
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ohci-pci/12p, 12M
    |__ Port 1: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 12M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/12p, 480M
```

雖然看不出裝置名稱，但還是有些資訊可供識別：

 * `Class` 表示[裝置類型 (device class)][device-class]，例如 `root_hub`、`Hub` (外接 hub)、`Mass Storage` (外接儲存裝置)、`Imaging` (PTP/MTP) 等。
 * 一個 device 可以有多個 interface (`If <NUM>`)，例如手機通常提供多個 interface，每個都有不同的 device class。
 * 每一行最後面的數字 (例如上面的 12M、480M)，代表傳輸速度，USB 1.1、2.0、3.0 分別是 12M、480M 跟 5000M。

至於裝置名稱，就只能透過 `-s` 反查：

    lsusb [-v] -s <BUS_NUMBER>:<DEVICE_NUMBER>

例如：

```
$ lsusb -s 2:2
Bus 002 Device 002: ID 80ee:0021 VirtualBox USB Tablet
```

 [device-class]: https://en.wikipedia.org/wiki/USB#Device_classes

更多關於 Linux 的學習心得，請參考 [Linux 學習筆記](https://jeremykao.gitbooks.io/learning-linux/)。

