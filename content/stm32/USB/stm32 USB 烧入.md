stm32大部分使用stlink/DAPlink等烧录器烧录程序，但是我发现买的一些开发板支持USB烧录，于是这次我打算试试画一个能用USB烧入代码的板子。

用的芯片是：stm32f072c8t6
开发软件：cubeMX、clion、STM32CubeProgrammer
根据手册设计了USB接口
![[blog/stm32/USB/stm32_USB_烧入_img/image-1.png]]

![[blog/stm32/USB/stm32_USB_烧入_img/image.png]]
![[image-2.png]]
虽然它说里面内置了DP的上拉电阻，但是我还是给它加了一个，以防外一
同时我也加了SW烧录接口，以防外一（幸好加了）

---

CubeMX 将USB开启，生成代码。烧录前先要进入Bootloader（BOOT0 拉高），然后就可以烧录了。
但是要使用dfu-util来实现DFU设备的烧入，但是stm32DFU和开源包dfu-util有冲突，于是用Zadig把驱动换成了WinUSB。

下载的dfu-util用Zadig更新后，将文件下dfu-util.exe和libusb-1.0.dll复制到项目目录下
然后在项目CMakeLists.txt底下加入下面的代码，重新更新一下，就能生成CMake应用程序

```CMakeLists.txt
  
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD  
        COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${CMAKE_PROJECT_NAME}.bin)  
  
add_custom_target(download  
        # 使用项目根目录下的 dfu-util.exe 进行烧录，并带上 :leave 参数使其烧录后自动运行  
        COMMAND ${CMAKE_SOURCE_DIR}/dfu-util.exe -a 0 -d 0483:df11 -s 0x08000000:leave -D ${CMAKE_PROJECT_NAME}.bin  
        DEPENDS ${CMAKE_PROJECT_NAME}.bin  
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}  
        COMMENT "Downloading firmware via dfu-util..."  
)
```

使用新生成的CMake应用程序，点击编译，就能完成编译下载了。

---

当然，这么简单就不至于写帖子了，还是遇到了很抽象的麻烦。

我使用STM32CubeProgrammer用USB连接时，发现设备被保护了
![[image-4.png]]
![[image-3.png]]

按弹窗说的勾选`读出解除保护 (MCU)`再连接也不行
但这时候USB已经可以通信了，只是被保护了，于是在cmd中用下面的代码强行清空MCU里面的文件
```
C:\Users\ThinkBook>"C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin\STM32_Programmer_CLI.exe" -c port=USB1 -rdu
      -------------------------------------------------------------------
                       STM32CubeProgrammer v2.23.0
      -------------------------------------------------------------------



USB speed   : Full Speed (12MBit/s)
Manuf. ID   : STMicroelectronics
Product ID  : STM32  BOOTLOADER
SN          : FFFFFFFEFFFF
DFU protocol: 1.1
Board       : --
Device ID   : unknown

Disabling memory Read Protection...

Memory Read Protection disabled successfully
```
得多试几次，就成功了

然后就按照上一节的配置clion，就可以用USB烧录了

---
[STM32CubeMX学习笔记（50）——USB接口使用（DFU固件升级）_usb dfu-CSDN博客](https://blog.csdn.net/qq_36347513/article/details/128499197)
在这里走了弯路，这篇文章要做的事情和我不一样

https://share.gemini.google/4IwTgtQ2jZTE
最终和gemini解决了
