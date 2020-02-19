---
title: Porting FreeRTOS to STM32F429Discovery
published: true
---

## 工具

- stlink：用來燒錄和 debug 的工具(a chip)
- openocd：用來執行 GDB server 與啟用 ARM semihosting
- gcc-arm-none-eabi: ARM GCC
- FreeRTOS source code
- Board: STM32F429 Discovery

## OS X 上安裝 gcc-arm-none-eabi、stlink、openocd

- stlink: brew install stlink (http://macappstore.org/stlink/)
- OpenOCD: brew install open-ocd (http://brewformulas.org/OpenOcd)
- gcc-arm-none-eabi:
  **方法一**

  ```
  $ brew tap PX4/homebrew-px4
  $ brew update
  $ sudo brew install gcc-arm-none-eabi    (might not the latest version)
  ```

  **方法二**

  1. Download "Mac installation tarball"
  2. Decompress tar
  3. Set PATH of gcc-arm-none-eabi

## Porting

Porting FreeRTOS 到 STM32F429-Discovery 主要是有幾個重點：

- Utility: 來自 STM32F429I-Discovery_FW_V1.0.1(官方 driver)，用途是提供一些操作硬體周邊的函式庫(API)，例如 LCD
- FreeRTOS：FreeRTOS 的 source code，在下載回來的包中 FreeRTOSV8.2.1/FreeRTOS/Source
- CORTEX_M4F_SK: 在下載回來的包中 FreeRTOSV8.2.1/FreeRTOS/Demo，用途是提供平台 CORTEX_M4F_SK 上的驅動函式庫，是屬於 FreeRTOS 軟體方開發的接口

## 應用程式的開發

以 Lab39 的 freertos-basic 中 FreeRTOS 應用程式原始碼 src/ 與 include/ 為例，我們整合成 app/，並把 src/ 中的 main.c 單獨拉出來，延續 myFreeRTOS (branch: game) 的檔案架構下，應用程式 game/ 和 main.c 是放在這裡 CORTEX_M4F_SK 中，所以專案結構如下：

```
.
|___CORTEX_M4F_SK
|               |__ app/
|               |__ main.c
|               |__ others C files
|___ FreeRTOS
|___ Utility
```

Hint：main.c 有樣板，在 STM32F429I-Discovery_FW_V1.0.1/Projects/Template

參考

- http://wiki.csie.ncku.edu.tw/embedded/freertos
- http://www.st.com/web/en/resource/technical/document/reference_manual/DM00031020.pdf
