# esp32_ota_ble
esp32 OTA through BLE  with Siliconlabs EFR Connect App

# 基于esp32c3的蓝牙无线升级功能
 esp32的idf已经提供了ota参考代码，参考代码基于http，esp32c3除了支持wifi 还支持ble，因此也有不少需要通过ble进行ota升级的需求
 
 Siliconlabs的efr connect app支持ota功能，因此通过抓包Siliconlabs efr connect app的ota过程后按其实现相应service 以及char 后就可以直接用efr connect来进行ble ota升级，省去了自己实现 ble ota client的工作。
 
 ## 开发前提
  * esp32c3，该项目在esp32-c3-devkitm-1评估板上实测
  * esp32的sdk idf v4.4
  * 手机app Siliconlabs efr connect（efr connect ota只支持传输.gbl的固件形式，因此待升级的esp32的bin文件直接将.bin后缀改为.gbl后就可以使用）
  
 ### 实现方法
   esp32 idf已经提供了很多参考代码实现，因此只要在其参考代码上进行修改即可，不得不说esp32 idf的参考代码实在丰富，相比其他家芯片来说大部分想要实现的功能都有参考实现，对开发者非常友好
   * 既然要使用siliconlabs的efr connect来做ota client，首先需要实现siliconlabs 自定义的service以及char，该部分定义可以从siliconlab的开发工具获得相关信息，如下
      siliconlabs的ota 相关的service以及char定义如下：
      
      1、
      Service
      Name: Silicon Labs OTA
      Type: com.silabs.service.ota
      UUID: 1D14D6EE-FD63-4FA1-BFA4-8F47B42119F0
      Source: Silicon Labs
      Abstract:
      The Silicon Labs OTA Service enables over-the-air firmware update of the device.

      2、
      Characteristics of the service:
        Silicon Labs OTA Control
          Characteristic
          Name: Silicon Labs OTA Control
          Type: com.silabs.characteristic.ota_control
          UUID: F7BF3564-FB6D-4E53-88A4-5E37E0326063
          Source: Silicon Labs
          Abstract:
      Silicon Labs OTA Control.

          Property requirements: 

            Reliable write - Excluded
            Indicate - Excluded
            Write Without Response - Excluded
            Read - Excluded
            Write - Mandatory
            Notify - Excluded
      3、
        Silicon Labs OTA Data
          Characteristic
          Name: Silicon Labs OTA Data
          Type: com.silabs.characteristic.ota_data
          UUID: 984227F3-34FC-4045-A5D0-2C581F81A153
          Source: Silicon Labs
          Abstract:
      Silicon Labs OTA Data.

          Property requirements: 

            Reliable write - Excluded
            Indicate - Excluded
            Write Without Response - Mandatory
            Read - Excluded
            Write - Mandatory
            Notify - Excluded
      
  * 了解siliconlabs efr connect app如何通过如上char进行ota流程，该部分通过抓包可以分析出来
        1、efr connect 在开始ota的时候会对ota control写0，以提示设备准备开始ota， 设备可以在此阶段进行erase flash
        2、通过ota data char 分段传输升级的固件，直至固件阐述完成
        3、固件传输完成后 efr connect 会通过对ota control写3，以提示ota固件传输完成，此时设备可以准备开始启动新固件
  * 按efr connect app的ota流程进行代码修改实现
  * 编译烧录固件
  * 编译待升级的测试固件，并将.Bin后缀名改为.gbl后缀名
  * 使用efr connect 进行升级验证
    efr32 connect 对esp32进行蓝牙升级的过程
          ![图片](https://user-images.githubusercontent.com/30143031/132782483-cf12eb56-f63d-42b5-a9f1-b7cea81b0d34.png)
          
### ota全过程log
         Executing action: monitor
      Serial port COM5
      Connecting....
      Detecting chip type... ESP32-C3
      Running idf_monitor in directory e:\projectcode\esp32c3\gatt_server_service_table
     
      [0;33m--- WARNING: GDB cannot open serial ports accessed as COMx[0m
      [0;33m--- Using \\.\COM5 instead...[0m
      --- idf_monitor on \\.\COM5 115200 ---
      --- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
      ESP-ROM:esp32c3-api1-20210207
      Build:Feb  7 2021
      rst:0x1 (POWERON),boot:0xc (SPI_FAST_FLASH_BOOT)
      SPIWP:0xee
      mode:DIO, clock div:1
      load:0x3fcd6100,len:0x1774
      load:0x403ce000,len:0x8d4
      load:0x403d0000,len:0x293c
      entry 0x403ce000
      I (30) boot: ESP-IDF v4.3 2nd stage bootloader
      I (30) boot: compile time 10:36:47
      I (30) boot: chip revision: 3
      I (32) boot.esp32c3: SPI Speed      : 80MHz
      I (36) boot.esp32c3: SPI Mode       : DIO
      I (41) boot.esp32c3: SPI Flash Size : 4MB
      I (46) boot: Enabling RNG early entropy source...
      I (51) boot: Partition Table:
      I (55) boot: ## Label            Usage          Type ST Offset   Length
      I (62) boot:  0 nvs              WiFi data        01 02 00009000 00004000
      I (70) boot:  1 otadata          OTA data         01 00 0000d000 00002000
      I (77) boot:  2 phy_init         RF data          01 01 0000f000 00001000
      I (85) boot:  3 ota_0            OTA app          00 10 00010000 00180000
      I (92) boot:  4 ota_1            OTA app          00 11 00190000 00180000
      I (99) boot: End of partition table
      I (104) esp_image: segment 0: paddr=00010020 vaddr=3c090020 size=1f290h (127632) map
      I (132) esp_image: segment 1: paddr=0002f2b8 vaddr=3fc8d600 size=00d60h (  3424) load
      I (133) esp_image: segment 2: paddr=00030020 vaddr=42000020 size=8cba4h (576420) map
      I (228) esp_image: segment 3: paddr=000bcbcc vaddr=3fc8e360 size=01d98h (  7576) load
      I (230) esp_image: segment 4: paddr=000be96c vaddr=40380000 size=0d4a0h ( 54432) load
      I (243) esp_image: segment 5: paddr=000cbe14 vaddr=50000000 size=00010h (    16) load
      I (248) boot: Loaded app from partition at offset 0x10000
      I (248) boot: Disabling RNG early entropy source...
      I (265) cpu_start: Pro cpu up.
      I (277) cpu_start: Pro cpu start user code
      I (278) cpu_start: cpu freq: 160000000
      I (278) cpu_start: Application information:
      I (280) cpu_start: Project name:     gatt_server_service_table_demo
      I (287) cpu_start: App version:      1
      I (292) cpu_start: Compile time:     Sep  9 2021 10:36:33
      I (298) cpu_start: ELF file SHA256:  7a46f6d09e802032...
      I (304) cpu_start: ESP-IDF:          v4.3
      I (308) heap_init: Initializing. RAM available for dynamic allocation:
      I (316) heap_init: At 3FC945F0 len 0002BA10 (174 KiB): DRAM
      I (322) heap_init: At 3FCC0000 len 0001F060 (124 KiB): STACK/DRAM
      I (329) heap_init: At 50000010 len 00001FF0 (7 KiB): RTCRAM
      I (335) spi_flash: detected chip: generic
      I (340) spi_flash: flash io: dio
      I (344) sleep: Configure to isolate all GPIO pins in sleep state
      I (350) sleep: Enable automatic switching of GPIO sleep configuration
      I (358) cpu_start: Starting scheduler.
      W (368) BTDM_INIT: esp_bt_controller_mem_release not implemented, return OK
      I (368) BTDM_INIT: BT controller compile version [501d88d]
      I (378) coexist: coexist rom version 9387209
      I (378) phy_init: phy_version 500,985899c,Apr 19 2021,16:05:08
      I (498) system_api: Base MAC address is not set
      I (498) system_api: read default base MAC address from EFUSE
      I (498) BTDM_INIT: Bluetooth MAC: 7c:df:a1:61:e8:39

      W (518) BT_BTM: BTM_BleWriteAdvData, Partial data write into ADV
      W (518) BT_BTM: BTM_BleWriteScanRsp, Partial data write into ADV
      I (528) GATTS_TABLE_DEMO: create attribute table successfully, the number handle = 5

      I (538) GATTS_TABLE_DEMO: SERVICE_START_EVT, status 0, service_handle 40
      I (538) GATTS_TABLE_DEMO: advertising start successfully
      I (36148) GATTS_TABLE_DEMO: ESP_GATTS_CONNECT_EVT, conn_id = 0
      I (36148) GATTS_TABLE_DEMO: 5d fd 5a 60 ac 8a
      I (36748) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 16, max_int = 32,conn_int = 6,latency = 0, timeout = 500
      I (36988) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 32,latency = 0, timeout = 1000
      I (37388) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 6,latency = 0, timeout = 500
      I (37588) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 32,latency = 0, timeout = 1000
      I (46108) GATTS_TABLE_DEMO: ESP_GATTS_DISCONNECT_EVT, reason = 0x13
      I (46108) GATTS_TABLE_DEMO: advertising start successfully
      I (57048) GATTS_TABLE_DEMO: ESP_GATTS_CONNECT_EVT, conn_id = 0
      I (57048) GATTS_TABLE_DEMO: 5d fd 5a 60 ac 8a
      I (57648) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 16, max_int = 32,conn_int = 6,latency = 0, timeout = 500
      I (57908) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 32,latency = 0, timeout = 1000
      I (58308) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 6,latency = 0, timeout = 500
      I (58508) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 32,latency = 0, timeout = 1000
      I (97988) GATTS_TABLE_DEMO: ESP_GATTS_MTU_EVT, MTU 250
      I (98068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 42, value len = 1, value :
      I (98068) GATTS_TABLE_DEMO: ota-control = 0
      I (98068) GATTS_TABLE_DEMO: ======beginota======
      I (98078) GATTS_TABLE_DEMO: Writing to partition subtype 17 at offset 0x190000
      I (98428) GATTS_TABLE_DEMO: update connection params status = 0, min_int = 0, max_int = 0,conn_int = 12,latency = 0, timeout = 500
      I (98618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98618) GATTS_TABLE_DEMO: ota-data = 244
      I (98678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98678) GATTS_TABLE_DEMO: ota-data = 244
      I (98678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98688) GATTS_TABLE_DEMO: ota-data = 244
      I (98708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98708) GATTS_TABLE_DEMO: ota-data = 244
      I (98738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98738) GATTS_TABLE_DEMO: ota-data = 244
      I (98768) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98768) GATTS_TABLE_DEMO: ota-data = 244
      I (98798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98798) GATTS_TABLE_DEMO: ota-data = 244
      I (98828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98828) GATTS_TABLE_DEMO: ota-data = 244
      I (98858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98858) GATTS_TABLE_DEMO: ota-data = 244
      I (98888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98888) GATTS_TABLE_DEMO: ota-data = 244
      I (98918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98918) GATTS_TABLE_DEMO: ota-data = 244
      I (98948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98948) GATTS_TABLE_DEMO: ota-data = 244
      I (98978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (98978) GATTS_TABLE_DEMO: ota-data = 244
      I (99008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99008) GATTS_TABLE_DEMO: ota-data = 244
      I (99038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99038) GATTS_TABLE_DEMO: ota-data = 244
      I (99068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99068) GATTS_TABLE_DEMO: ota-data = 244
      I (99098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99098) GATTS_TABLE_DEMO: ota-data = 244
      I (99148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99148) GATTS_TABLE_DEMO: ota-data = 244
      I (99158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99158) GATTS_TABLE_DEMO: ota-data = 244
      I (99188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99188) GATTS_TABLE_DEMO: ota-data = 244
      I (99218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99218) GATTS_TABLE_DEMO: ota-data = 244
      I (99248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99248) GATTS_TABLE_DEMO: ota-data = 244
      I (99278) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99278) GATTS_TABLE_DEMO: ota-data = 244
      I (99308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99308) GATTS_TABLE_DEMO: ota-data = 244
      I (99338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99338) GATTS_TABLE_DEMO: ota-data = 244
      I (99368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99368) GATTS_TABLE_DEMO: ota-data = 244
      I (99398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99398) GATTS_TABLE_DEMO: ota-data = 244
      I (99428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99428) GATTS_TABLE_DEMO: ota-data = 244
      I (99458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99458) GATTS_TABLE_DEMO: ota-data = 244
      I (99488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99488) GATTS_TABLE_DEMO: ota-data = 244
      I (99518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99518) GATTS_TABLE_DEMO: ota-data = 244
      I (99548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99548) GATTS_TABLE_DEMO: ota-data = 244
      I (99578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99578) GATTS_TABLE_DEMO: ota-data = 244
      I (99608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99608) GATTS_TABLE_DEMO: ota-data = 244
      I (99658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99658) GATTS_TABLE_DEMO: ota-data = 244
      I (99668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99668) GATTS_TABLE_DEMO: ota-data = 244
      I (99698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99698) GATTS_TABLE_DEMO: ota-data = 244
      I (99728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99728) GATTS_TABLE_DEMO: ota-data = 244
      I (99758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99758) GATTS_TABLE_DEMO: ota-data = 244
      I (99788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99788) GATTS_TABLE_DEMO: ota-data = 244
      I (99818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99818) GATTS_TABLE_DEMO: ota-data = 244
      I (99848) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99848) GATTS_TABLE_DEMO: ota-data = 244
      I (99878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99878) GATTS_TABLE_DEMO: ota-data = 244
      I (99908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99908) GATTS_TABLE_DEMO: ota-data = 244
      I (99938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99938) GATTS_TABLE_DEMO: ota-data = 244
      I (99968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99968) GATTS_TABLE_DEMO: ota-data = 244
      I (99998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (99998) GATTS_TABLE_DEMO: ota-data = 244
      I (100028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100028) GATTS_TABLE_DEMO: ota-data = 244
      I (100058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100058) GATTS_TABLE_DEMO: ota-data = 244
      I (100088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100088) GATTS_TABLE_DEMO: ota-data = 244
      I (100118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100118) GATTS_TABLE_DEMO: ota-data = 244
      I (100168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100168) GATTS_TABLE_DEMO: ota-data = 244
      I (100178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100178) GATTS_TABLE_DEMO: ota-data = 244
      I (100208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100208) GATTS_TABLE_DEMO: ota-data = 244
      I (100238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100238) GATTS_TABLE_DEMO: ota-data = 244
      I (100268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100268) GATTS_TABLE_DEMO: ota-data = 244
      I (100298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100298) GATTS_TABLE_DEMO: ota-data = 244
      I (100328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100328) GATTS_TABLE_DEMO: ota-data = 244
      I (100358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100358) GATTS_TABLE_DEMO: ota-data = 244
      I (100388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100388) GATTS_TABLE_DEMO: ota-data = 244
      I (100418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100418) GATTS_TABLE_DEMO: ota-data = 244
      I (100448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100448) GATTS_TABLE_DEMO: ota-data = 244
      I (100478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100478) GATTS_TABLE_DEMO: ota-data = 244
      I (100508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100508) GATTS_TABLE_DEMO: ota-data = 244
      I (100538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100538) GATTS_TABLE_DEMO: ota-data = 244
      I (100568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100568) GATTS_TABLE_DEMO: ota-data = 244
      I (100598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100598) GATTS_TABLE_DEMO: ota-data = 244
      I (100628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100628) GATTS_TABLE_DEMO: ota-data = 244
      I (100678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100678) GATTS_TABLE_DEMO: ota-data = 244
      I (100688) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100688) GATTS_TABLE_DEMO: ota-data = 244
      I (100718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100718) GATTS_TABLE_DEMO: ota-data = 244
      I (100748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100748) GATTS_TABLE_DEMO: ota-data = 244
      I (100778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100778) GATTS_TABLE_DEMO: ota-data = 244
      I (100808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100808) GATTS_TABLE_DEMO: ota-data = 244
      I (100838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100838) GATTS_TABLE_DEMO: ota-data = 244
      I (100868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100868) GATTS_TABLE_DEMO: ota-data = 244
      I (100898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100898) GATTS_TABLE_DEMO: ota-data = 244
      I (100928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100928) GATTS_TABLE_DEMO: ota-data = 244
      I (100958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100958) GATTS_TABLE_DEMO: ota-data = 244
      I (100988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (100988) GATTS_TABLE_DEMO: ota-data = 244
      I (101018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101018) GATTS_TABLE_DEMO: ota-data = 244
      I (101048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101048) GATTS_TABLE_DEMO: ota-data = 244
      I (101078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101078) GATTS_TABLE_DEMO: ota-data = 244
      I (101108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101108) GATTS_TABLE_DEMO: ota-data = 244
      I (101158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101158) GATTS_TABLE_DEMO: ota-data = 244
      I (101168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101168) GATTS_TABLE_DEMO: ota-data = 244
      I (101198) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101198) GATTS_TABLE_DEMO: ota-data = 244
      I (101228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101228) GATTS_TABLE_DEMO: ota-data = 244
      I (101258) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101258) GATTS_TABLE_DEMO: ota-data = 244
      I (101288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101288) GATTS_TABLE_DEMO: ota-data = 244
      I (101318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101318) GATTS_TABLE_DEMO: ota-data = 244
      I (101348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101348) GATTS_TABLE_DEMO: ota-data = 244
      I (101378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101378) GATTS_TABLE_DEMO: ota-data = 244
      I (101408) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101408) GATTS_TABLE_DEMO: ota-data = 244
      I (101438) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101438) GATTS_TABLE_DEMO: ota-data = 244
      I (101468) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101468) GATTS_TABLE_DEMO: ota-data = 244
      I (101498) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101498) GATTS_TABLE_DEMO: ota-data = 244
      I (101528) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101528) GATTS_TABLE_DEMO: ota-data = 244
      I (101558) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101558) GATTS_TABLE_DEMO: ota-data = 244
      I (101588) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101588) GATTS_TABLE_DEMO: ota-data = 244
      I (101618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101618) GATTS_TABLE_DEMO: ota-data = 244
      I (101668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101668) GATTS_TABLE_DEMO: ota-data = 244
      I (101678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101678) GATTS_TABLE_DEMO: ota-data = 244
      I (101708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101708) GATTS_TABLE_DEMO: ota-data = 244
      I (101738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101738) GATTS_TABLE_DEMO: ota-data = 244
      I (101768) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101768) GATTS_TABLE_DEMO: ota-data = 244
      I (101798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101798) GATTS_TABLE_DEMO: ota-data = 244
      I (101828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101828) GATTS_TABLE_DEMO: ota-data = 244
      I (101858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101858) GATTS_TABLE_DEMO: ota-data = 244
      I (101888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101888) GATTS_TABLE_DEMO: ota-data = 244
      I (101918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101918) GATTS_TABLE_DEMO: ota-data = 244
      I (101948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101948) GATTS_TABLE_DEMO: ota-data = 244
      I (101978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (101978) GATTS_TABLE_DEMO: ota-data = 244
      I (102008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102008) GATTS_TABLE_DEMO: ota-data = 244
      I (102038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102038) GATTS_TABLE_DEMO: ota-data = 244
      I (102068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102068) GATTS_TABLE_DEMO: ota-data = 244
      I (102098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102098) GATTS_TABLE_DEMO: ota-data = 244
      I (102128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102128) GATTS_TABLE_DEMO: ota-data = 244
      I (102178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102178) GATTS_TABLE_DEMO: ota-data = 244
      I (102188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102188) GATTS_TABLE_DEMO: ota-data = 244
      I (102218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102218) GATTS_TABLE_DEMO: ota-data = 244
      I (102268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102268) GATTS_TABLE_DEMO: ota-data = 244
      I (102298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102298) GATTS_TABLE_DEMO: ota-data = 244
      I (102328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102328) GATTS_TABLE_DEMO: ota-data = 244
      I (102358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102358) GATTS_TABLE_DEMO: ota-data = 244
      I (102388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102388) GATTS_TABLE_DEMO: ota-data = 244
      I (102418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102418) GATTS_TABLE_DEMO: ota-data = 244
      I (102448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102448) GATTS_TABLE_DEMO: ota-data = 244
      I (102478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102478) GATTS_TABLE_DEMO: ota-data = 244
      I (102508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102508) GATTS_TABLE_DEMO: ota-data = 244
      I (102538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102538) GATTS_TABLE_DEMO: ota-data = 244
      I (102568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102568) GATTS_TABLE_DEMO: ota-data = 244
      I (102598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102598) GATTS_TABLE_DEMO: ota-data = 244
      I (102628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102628) GATTS_TABLE_DEMO: ota-data = 244
      I (102658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102658) GATTS_TABLE_DEMO: ota-data = 244
      I (102708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102708) GATTS_TABLE_DEMO: ota-data = 244
      I (102718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102718) GATTS_TABLE_DEMO: ota-data = 244
      I (102748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102748) GATTS_TABLE_DEMO: ota-data = 244
      I (102778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102778) GATTS_TABLE_DEMO: ota-data = 244
      I (102808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102808) GATTS_TABLE_DEMO: ota-data = 244
      I (102838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102838) GATTS_TABLE_DEMO: ota-data = 244
      I (102868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102868) GATTS_TABLE_DEMO: ota-data = 244
      I (102898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102898) GATTS_TABLE_DEMO: ota-data = 244
      I (102928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102928) GATTS_TABLE_DEMO: ota-data = 244
      I (102958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102958) GATTS_TABLE_DEMO: ota-data = 244
      I (102988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (102988) GATTS_TABLE_DEMO: ota-data = 244
      I (103018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103018) GATTS_TABLE_DEMO: ota-data = 244
      I (103048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103048) GATTS_TABLE_DEMO: ota-data = 244
      I (103078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103078) GATTS_TABLE_DEMO: ota-data = 244
      I (103108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103108) GATTS_TABLE_DEMO: ota-data = 244
      I (103138) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103138) GATTS_TABLE_DEMO: ota-data = 244
      I (103168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103168) GATTS_TABLE_DEMO: ota-data = 244
      I (103218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103218) GATTS_TABLE_DEMO: ota-data = 244
      I (103228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103228) GATTS_TABLE_DEMO: ota-data = 244
      I (103258) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103258) GATTS_TABLE_DEMO: ota-data = 244
      I (103288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103288) GATTS_TABLE_DEMO: ota-data = 244
      I (103318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103318) GATTS_TABLE_DEMO: ota-data = 244
      I (103348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103348) GATTS_TABLE_DEMO: ota-data = 244
      I (103378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103378) GATTS_TABLE_DEMO: ota-data = 244
      I (103408) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103408) GATTS_TABLE_DEMO: ota-data = 244
      I (103438) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103438) GATTS_TABLE_DEMO: ota-data = 244
      I (103468) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103468) GATTS_TABLE_DEMO: ota-data = 244
      I (103498) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103498) GATTS_TABLE_DEMO: ota-data = 244
      I (103528) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103528) GATTS_TABLE_DEMO: ota-data = 244
      I (103558) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103558) GATTS_TABLE_DEMO: ota-data = 244
      I (103588) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103588) GATTS_TABLE_DEMO: ota-data = 244
      I (103618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103618) GATTS_TABLE_DEMO: ota-data = 244
      I (103648) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103648) GATTS_TABLE_DEMO: ota-data = 244
      I (103698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103698) GATTS_TABLE_DEMO: ota-data = 244
      I (103708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103708) GATTS_TABLE_DEMO: ota-data = 244
      I (103738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103738) GATTS_TABLE_DEMO: ota-data = 244
      I (103768) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103768) GATTS_TABLE_DEMO: ota-data = 244
      I (103798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103798) GATTS_TABLE_DEMO: ota-data = 244
      I (103828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103828) GATTS_TABLE_DEMO: ota-data = 244
      I (103858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103858) GATTS_TABLE_DEMO: ota-data = 244
      I (103888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103888) GATTS_TABLE_DEMO: ota-data = 244
      I (103918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103918) GATTS_TABLE_DEMO: ota-data = 244
      I (103948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103948) GATTS_TABLE_DEMO: ota-data = 244
      I (103978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (103978) GATTS_TABLE_DEMO: ota-data = 244
      I (104008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104008) GATTS_TABLE_DEMO: ota-data = 244
      I (104038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104038) GATTS_TABLE_DEMO: ota-data = 244
      I (104068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104068) GATTS_TABLE_DEMO: ota-data = 244
      I (104098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104098) GATTS_TABLE_DEMO: ota-data = 244
      I (104128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104128) GATTS_TABLE_DEMO: ota-data = 244
      I (104158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104158) GATTS_TABLE_DEMO: ota-data = 244
      I (104208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104208) GATTS_TABLE_DEMO: ota-data = 244
      I (104218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104218) GATTS_TABLE_DEMO: ota-data = 244
      I (104248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104248) GATTS_TABLE_DEMO: ota-data = 244
      I (104278) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104278) GATTS_TABLE_DEMO: ota-data = 244
      I (104308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104308) GATTS_TABLE_DEMO: ota-data = 244
      I (104338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104338) GATTS_TABLE_DEMO: ota-data = 244
      I (104368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104368) GATTS_TABLE_DEMO: ota-data = 244
      I (104398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104398) GATTS_TABLE_DEMO: ota-data = 244
      I (104428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104428) GATTS_TABLE_DEMO: ota-data = 244
      I (104458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104458) GATTS_TABLE_DEMO: ota-data = 244
      I (104488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104488) GATTS_TABLE_DEMO: ota-data = 244
      I (104518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104518) GATTS_TABLE_DEMO: ota-data = 244
      I (104548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104548) GATTS_TABLE_DEMO: ota-data = 244
      I (104578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104578) GATTS_TABLE_DEMO: ota-data = 244
      I (104608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104608) GATTS_TABLE_DEMO: ota-data = 244
      I (104638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104638) GATTS_TABLE_DEMO: ota-data = 244
      I (104668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104668) GATTS_TABLE_DEMO: ota-data = 244
      I (104718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104718) GATTS_TABLE_DEMO: ota-data = 244
      I (104728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104728) GATTS_TABLE_DEMO: ota-data = 244
      I (104758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104758) GATTS_TABLE_DEMO: ota-data = 244
      I (104788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104788) GATTS_TABLE_DEMO: ota-data = 244
      I (104818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104818) GATTS_TABLE_DEMO: ota-data = 244
      I (104848) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104848) GATTS_TABLE_DEMO: ota-data = 244
      I (104878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104878) GATTS_TABLE_DEMO: ota-data = 244
      I (104908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104908) GATTS_TABLE_DEMO: ota-data = 244
      I (104938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104938) GATTS_TABLE_DEMO: ota-data = 244
      I (104968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104968) GATTS_TABLE_DEMO: ota-data = 244
      I (104998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (104998) GATTS_TABLE_DEMO: ota-data = 244
      I (105028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105028) GATTS_TABLE_DEMO: ota-data = 244
      I (105058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105058) GATTS_TABLE_DEMO: ota-data = 244
      I (105088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105088) GATTS_TABLE_DEMO: ota-data = 244
      I (105118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105118) GATTS_TABLE_DEMO: ota-data = 244
      I (105148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105148) GATTS_TABLE_DEMO: ota-data = 244
      I (105178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105178) GATTS_TABLE_DEMO: ota-data = 244
      I (105228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105228) GATTS_TABLE_DEMO: ota-data = 244
      I (105238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105238) GATTS_TABLE_DEMO: ota-data = 244
      I (105268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105268) GATTS_TABLE_DEMO: ota-data = 244
      I (105298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105298) GATTS_TABLE_DEMO: ota-data = 244
      I (105328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105328) GATTS_TABLE_DEMO: ota-data = 244
      I (105358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105358) GATTS_TABLE_DEMO: ota-data = 244
      I (105388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105388) GATTS_TABLE_DEMO: ota-data = 244
      I (105418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105418) GATTS_TABLE_DEMO: ota-data = 244
      I (105448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105448) GATTS_TABLE_DEMO: ota-data = 244
      I (105478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105478) GATTS_TABLE_DEMO: ota-data = 244
      I (105508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105508) GATTS_TABLE_DEMO: ota-data = 244
      I (105538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105538) GATTS_TABLE_DEMO: ota-data = 244
      I (105568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105568) GATTS_TABLE_DEMO: ota-data = 244
      I (105598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105598) GATTS_TABLE_DEMO: ota-data = 244
      I (105628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105628) GATTS_TABLE_DEMO: ota-data = 244
      I (105658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105658) GATTS_TABLE_DEMO: ota-data = 244
      I (105688) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105688) GATTS_TABLE_DEMO: ota-data = 244
      I (105738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105738) GATTS_TABLE_DEMO: ota-data = 244
      I (105748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105748) GATTS_TABLE_DEMO: ota-data = 244
      I (105778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105778) GATTS_TABLE_DEMO: ota-data = 244
      I (105808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105808) GATTS_TABLE_DEMO: ota-data = 244
      I (105838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105838) GATTS_TABLE_DEMO: ota-data = 244
      I (105868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105868) GATTS_TABLE_DEMO: ota-data = 244
      I (105898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105898) GATTS_TABLE_DEMO: ota-data = 244
      I (105928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105928) GATTS_TABLE_DEMO: ota-data = 244
      I (105958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105958) GATTS_TABLE_DEMO: ota-data = 244
      I (105988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (105988) GATTS_TABLE_DEMO: ota-data = 244
      I (106018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106018) GATTS_TABLE_DEMO: ota-data = 244
      I (106048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106048) GATTS_TABLE_DEMO: ota-data = 244
      I (106078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106078) GATTS_TABLE_DEMO: ota-data = 244
      I (106108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106108) GATTS_TABLE_DEMO: ota-data = 244
      I (106138) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106138) GATTS_TABLE_DEMO: ota-data = 244
      I (106168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106168) GATTS_TABLE_DEMO: ota-data = 244
      I (106218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106218) GATTS_TABLE_DEMO: ota-data = 244
      I (106228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106228) GATTS_TABLE_DEMO: ota-data = 244
      I (106258) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106258) GATTS_TABLE_DEMO: ota-data = 244
      I (106288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106288) GATTS_TABLE_DEMO: ota-data = 244
      I (106318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106318) GATTS_TABLE_DEMO: ota-data = 244
      I (106348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106348) GATTS_TABLE_DEMO: ota-data = 244
      I (106378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106378) GATTS_TABLE_DEMO: ota-data = 244
      I (106408) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106408) GATTS_TABLE_DEMO: ota-data = 244
      I (106438) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106438) GATTS_TABLE_DEMO: ota-data = 244
      I (106468) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106468) GATTS_TABLE_DEMO: ota-data = 244
      I (106498) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106498) GATTS_TABLE_DEMO: ota-data = 244
      I (106528) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106528) GATTS_TABLE_DEMO: ota-data = 244
      I (106558) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106558) GATTS_TABLE_DEMO: ota-data = 244
      I (106588) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106588) GATTS_TABLE_DEMO: ota-data = 244
      I (106618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106618) GATTS_TABLE_DEMO: ota-data = 244
      I (106648) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106648) GATTS_TABLE_DEMO: ota-data = 244
      I (106678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106678) GATTS_TABLE_DEMO: ota-data = 244
      I (106728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106728) GATTS_TABLE_DEMO: ota-data = 244
      I (106738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106738) GATTS_TABLE_DEMO: ota-data = 244
      I (106768) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106768) GATTS_TABLE_DEMO: ota-data = 244
      I (106798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106798) GATTS_TABLE_DEMO: ota-data = 244
      I (106828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106828) GATTS_TABLE_DEMO: ota-data = 244
      I (106858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106858) GATTS_TABLE_DEMO: ota-data = 244
      I (106888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106888) GATTS_TABLE_DEMO: ota-data = 244
      I (106918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106918) GATTS_TABLE_DEMO: ota-data = 244
      I (106948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106948) GATTS_TABLE_DEMO: ota-data = 244
      I (106978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (106978) GATTS_TABLE_DEMO: ota-data = 244
      I (107008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107008) GATTS_TABLE_DEMO: ota-data = 244
      I (107038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107038) GATTS_TABLE_DEMO: ota-data = 244
      I (107068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107068) GATTS_TABLE_DEMO: ota-data = 244
      I (107098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107098) GATTS_TABLE_DEMO: ota-data = 244
      I (107128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107128) GATTS_TABLE_DEMO: ota-data = 244
      I (107158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107158) GATTS_TABLE_DEMO: ota-data = 244
      I (107188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107188) GATTS_TABLE_DEMO: ota-data = 244
      I (107238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107238) GATTS_TABLE_DEMO: ota-data = 244
      I (107248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107248) GATTS_TABLE_DEMO: ota-data = 244
      I (107278) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107278) GATTS_TABLE_DEMO: ota-data = 244
      I (107308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107308) GATTS_TABLE_DEMO: ota-data = 244
      I (107338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107338) GATTS_TABLE_DEMO: ota-data = 244
      I (107368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107368) GATTS_TABLE_DEMO: ota-data = 244
      I (107398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107398) GATTS_TABLE_DEMO: ota-data = 244
      I (107428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107428) GATTS_TABLE_DEMO: ota-data = 244
      I (107458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107458) GATTS_TABLE_DEMO: ota-data = 244
      I (107488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107488) GATTS_TABLE_DEMO: ota-data = 244
      I (107518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107518) GATTS_TABLE_DEMO: ota-data = 244
      I (107548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107548) GATTS_TABLE_DEMO: ota-data = 244
      I (107578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107578) GATTS_TABLE_DEMO: ota-data = 244
      I (107608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107608) GATTS_TABLE_DEMO: ota-data = 244
      I (107638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107638) GATTS_TABLE_DEMO: ota-data = 244
      I (107668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107668) GATTS_TABLE_DEMO: ota-data = 244
      I (107698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107698) GATTS_TABLE_DEMO: ota-data = 244
      I (107748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107748) GATTS_TABLE_DEMO: ota-data = 244
      I (107758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107758) GATTS_TABLE_DEMO: ota-data = 244
      I (107788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107788) GATTS_TABLE_DEMO: ota-data = 244
      I (107818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107818) GATTS_TABLE_DEMO: ota-data = 244
      I (107858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107858) GATTS_TABLE_DEMO: ota-data = 244
      I (107888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107888) GATTS_TABLE_DEMO: ota-data = 244
      I (107918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107918) GATTS_TABLE_DEMO: ota-data = 244
      I (107948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107948) GATTS_TABLE_DEMO: ota-data = 244
      I (107978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (107978) GATTS_TABLE_DEMO: ota-data = 244
      I (108008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108008) GATTS_TABLE_DEMO: ota-data = 244
      I (108038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108038) GATTS_TABLE_DEMO: ota-data = 244
      I (108068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108068) GATTS_TABLE_DEMO: ota-data = 244
      I (108098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108098) GATTS_TABLE_DEMO: ota-data = 244
      I (108128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108128) GATTS_TABLE_DEMO: ota-data = 244
      I (108158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108158) GATTS_TABLE_DEMO: ota-data = 244
      I (108188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108188) GATTS_TABLE_DEMO: ota-data = 244
      I (108238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108238) GATTS_TABLE_DEMO: ota-data = 244
      I (108248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108248) GATTS_TABLE_DEMO: ota-data = 244
      I (108278) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108278) GATTS_TABLE_DEMO: ota-data = 244
      I (108308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108308) GATTS_TABLE_DEMO: ota-data = 244
      I (108338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108338) GATTS_TABLE_DEMO: ota-data = 244
      I (108368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108368) GATTS_TABLE_DEMO: ota-data = 244
      I (108398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108398) GATTS_TABLE_DEMO: ota-data = 244
      I (108428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108428) GATTS_TABLE_DEMO: ota-data = 244
      I (108458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108458) GATTS_TABLE_DEMO: ota-data = 244
      I (108488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108488) GATTS_TABLE_DEMO: ota-data = 244
      I (108518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108518) GATTS_TABLE_DEMO: ota-data = 244
      I (108548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108548) GATTS_TABLE_DEMO: ota-data = 244
      I (108578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108578) GATTS_TABLE_DEMO: ota-data = 244
      I (108608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108608) GATTS_TABLE_DEMO: ota-data = 244
      I (108638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108638) GATTS_TABLE_DEMO: ota-data = 244
      I (108668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108668) GATTS_TABLE_DEMO: ota-data = 244
      I (108698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108698) GATTS_TABLE_DEMO: ota-data = 244
      I (108748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108748) GATTS_TABLE_DEMO: ota-data = 244
      I (108778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108778) GATTS_TABLE_DEMO: ota-data = 244
      I (108808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108808) GATTS_TABLE_DEMO: ota-data = 244
      I (108838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108838) GATTS_TABLE_DEMO: ota-data = 244
      I (108868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108868) GATTS_TABLE_DEMO: ota-data = 244
      I (108898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108898) GATTS_TABLE_DEMO: ota-data = 244
      I (108928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108928) GATTS_TABLE_DEMO: ota-data = 244
      I (108958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108958) GATTS_TABLE_DEMO: ota-data = 244
      I (108988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (108988) GATTS_TABLE_DEMO: ota-data = 244
      I (109018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109018) GATTS_TABLE_DEMO: ota-data = 244
      I (109048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109048) GATTS_TABLE_DEMO: ota-data = 244
      I (109078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109078) GATTS_TABLE_DEMO: ota-data = 244
      I (109108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109108) GATTS_TABLE_DEMO: ota-data = 244
      I (109138) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109138) GATTS_TABLE_DEMO: ota-data = 244
      I (109168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109168) GATTS_TABLE_DEMO: ota-data = 244
      I (109198) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109198) GATTS_TABLE_DEMO: ota-data = 244
      I (109228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109228) GATTS_TABLE_DEMO: ota-data = 244
      I (109278) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109278) GATTS_TABLE_DEMO: ota-data = 244
      I (109288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109288) GATTS_TABLE_DEMO: ota-data = 244
      I (109318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109318) GATTS_TABLE_DEMO: ota-data = 244
      I (109348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109348) GATTS_TABLE_DEMO: ota-data = 244
      I (109378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109378) GATTS_TABLE_DEMO: ota-data = 244
      I (109408) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109408) GATTS_TABLE_DEMO: ota-data = 244
      I (109438) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109438) GATTS_TABLE_DEMO: ota-data = 244
      I (109468) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109468) GATTS_TABLE_DEMO: ota-data = 244
      I (109498) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109498) GATTS_TABLE_DEMO: ota-data = 244
      I (109528) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109528) GATTS_TABLE_DEMO: ota-data = 244
      I (109558) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109558) GATTS_TABLE_DEMO: ota-data = 244
      I (109588) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109588) GATTS_TABLE_DEMO: ota-data = 244
      I (109618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109618) GATTS_TABLE_DEMO: ota-data = 244
      I (109648) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109648) GATTS_TABLE_DEMO: ota-data = 244
      I (109678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109678) GATTS_TABLE_DEMO: ota-data = 244
      I (109708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109708) GATTS_TABLE_DEMO: ota-data = 244
      I (109738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109738) GATTS_TABLE_DEMO: ota-data = 244
      I (109788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109788) GATTS_TABLE_DEMO: ota-data = 244
      I (109798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109798) GATTS_TABLE_DEMO: ota-data = 244
      I (109828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109828) GATTS_TABLE_DEMO: ota-data = 244
      I (109858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109858) GATTS_TABLE_DEMO: ota-data = 244
      I (109888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109888) GATTS_TABLE_DEMO: ota-data = 244
      I (109918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109918) GATTS_TABLE_DEMO: ota-data = 244
      I (109948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109948) GATTS_TABLE_DEMO: ota-data = 244
      I (109978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (109978) GATTS_TABLE_DEMO: ota-data = 244
      I (110008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110008) GATTS_TABLE_DEMO: ota-data = 244
      I (110038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110038) GATTS_TABLE_DEMO: ota-data = 244
      I (110068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110068) GATTS_TABLE_DEMO: ota-data = 244
      I (110098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110098) GATTS_TABLE_DEMO: ota-data = 244
      I (110128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110128) GATTS_TABLE_DEMO: ota-data = 244
      I (110158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110158) GATTS_TABLE_DEMO: ota-data = 244
      I (110188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110188) GATTS_TABLE_DEMO: ota-data = 244
      I (110218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110218) GATTS_TABLE_DEMO: ota-data = 244
      I (110248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110248) GATTS_TABLE_DEMO: ota-data = 244
      I (110298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110298) GATTS_TABLE_DEMO: ota-data = 244
      I (110308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110308) GATTS_TABLE_DEMO: ota-data = 244
      I (110338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110338) GATTS_TABLE_DEMO: ota-data = 244
      I (110368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110368) GATTS_TABLE_DEMO: ota-data = 244
      I (110398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110398) GATTS_TABLE_DEMO: ota-data = 244
      I (110428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110428) GATTS_TABLE_DEMO: ota-data = 244
      I (110458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110458) GATTS_TABLE_DEMO: ota-data = 244
      I (110488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110488) GATTS_TABLE_DEMO: ota-data = 244
      I (110518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110518) GATTS_TABLE_DEMO: ota-data = 244
      I (110548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110548) GATTS_TABLE_DEMO: ota-data = 244
      I (110578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110578) GATTS_TABLE_DEMO: ota-data = 244
      I (110608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110608) GATTS_TABLE_DEMO: ota-data = 244
      I (110638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110638) GATTS_TABLE_DEMO: ota-data = 244
      I (110668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110668) GATTS_TABLE_DEMO: ota-data = 244
      I (110698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110698) GATTS_TABLE_DEMO: ota-data = 244
      I (110728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110728) GATTS_TABLE_DEMO: ota-data = 244
      I (110778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110778) GATTS_TABLE_DEMO: ota-data = 244
      I (110788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110788) GATTS_TABLE_DEMO: ota-data = 244
      I (110818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110818) GATTS_TABLE_DEMO: ota-data = 244
      I (110848) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110848) GATTS_TABLE_DEMO: ota-data = 244
      I (110878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110878) GATTS_TABLE_DEMO: ota-data = 244
      I (110908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110908) GATTS_TABLE_DEMO: ota-data = 244
      I (110938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110938) GATTS_TABLE_DEMO: ota-data = 244
      I (110968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110968) GATTS_TABLE_DEMO: ota-data = 244
      I (110998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (110998) GATTS_TABLE_DEMO: ota-data = 244
      I (111028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111028) GATTS_TABLE_DEMO: ota-data = 244
      I (111058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111058) GATTS_TABLE_DEMO: ota-data = 244
      I (111088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111088) GATTS_TABLE_DEMO: ota-data = 244
      I (111118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111118) GATTS_TABLE_DEMO: ota-data = 244
      I (111148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111148) GATTS_TABLE_DEMO: ota-data = 244
      I (111178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111178) GATTS_TABLE_DEMO: ota-data = 244
      I (111208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111208) GATTS_TABLE_DEMO: ota-data = 244
      I (111238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111238) GATTS_TABLE_DEMO: ota-data = 244
      I (111288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111288) GATTS_TABLE_DEMO: ota-data = 244
      I (111298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111298) GATTS_TABLE_DEMO: ota-data = 244
      I (111328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111328) GATTS_TABLE_DEMO: ota-data = 244
      I (111358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111358) GATTS_TABLE_DEMO: ota-data = 244
      I (111388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111388) GATTS_TABLE_DEMO: ota-data = 244
      I (111418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111418) GATTS_TABLE_DEMO: ota-data = 244
      I (111448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111448) GATTS_TABLE_DEMO: ota-data = 244
      I (111478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111478) GATTS_TABLE_DEMO: ota-data = 244
      I (111508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111508) GATTS_TABLE_DEMO: ota-data = 244
      I (111538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111538) GATTS_TABLE_DEMO: ota-data = 244
      I (111568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111568) GATTS_TABLE_DEMO: ota-data = 244
      I (111598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111598) GATTS_TABLE_DEMO: ota-data = 244
      I (111628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111628) GATTS_TABLE_DEMO: ota-data = 244
      I (111658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111658) GATTS_TABLE_DEMO: ota-data = 244
      I (111688) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111688) GATTS_TABLE_DEMO: ota-data = 244
      I (111718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111718) GATTS_TABLE_DEMO: ota-data = 244
      I (111748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111748) GATTS_TABLE_DEMO: ota-data = 244
      I (111798) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111798) GATTS_TABLE_DEMO: ota-data = 244
      I (111808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111808) GATTS_TABLE_DEMO: ota-data = 244
      I (111838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111838) GATTS_TABLE_DEMO: ota-data = 244
      I (111868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111868) GATTS_TABLE_DEMO: ota-data = 244
      I (111898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111898) GATTS_TABLE_DEMO: ota-data = 244
      I (111928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111928) GATTS_TABLE_DEMO: ota-data = 244
      I (111958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111958) GATTS_TABLE_DEMO: ota-data = 244
      I (111988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (111988) GATTS_TABLE_DEMO: ota-data = 244
      I (112018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112018) GATTS_TABLE_DEMO: ota-data = 244
      I (112048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112048) GATTS_TABLE_DEMO: ota-data = 244
      I (112078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112078) GATTS_TABLE_DEMO: ota-data = 244
      I (112108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112108) GATTS_TABLE_DEMO: ota-data = 244
      I (112138) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112138) GATTS_TABLE_DEMO: ota-data = 244
      I (112168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112168) GATTS_TABLE_DEMO: ota-data = 244
      I (112198) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112198) GATTS_TABLE_DEMO: ota-data = 244
      I (112228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112228) GATTS_TABLE_DEMO: ota-data = 244
      I (112258) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112258) GATTS_TABLE_DEMO: ota-data = 244
      I (112308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112308) GATTS_TABLE_DEMO: ota-data = 244
      I (112318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112318) GATTS_TABLE_DEMO: ota-data = 244
      I (112348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112348) GATTS_TABLE_DEMO: ota-data = 244
      I (112378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112378) GATTS_TABLE_DEMO: ota-data = 244
      I (112408) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112408) GATTS_TABLE_DEMO: ota-data = 244
      I (112438) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112438) GATTS_TABLE_DEMO: ota-data = 244
      I (112468) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112468) GATTS_TABLE_DEMO: ota-data = 244
      I (112498) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112498) GATTS_TABLE_DEMO: ota-data = 244
      I (112528) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112528) GATTS_TABLE_DEMO: ota-data = 244
      I (112558) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112558) GATTS_TABLE_DEMO: ota-data = 244
      I (112588) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112588) GATTS_TABLE_DEMO: ota-data = 244
      I (112618) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112618) GATTS_TABLE_DEMO: ota-data = 244
      I (112648) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112648) GATTS_TABLE_DEMO: ota-data = 244
      I (112678) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112678) GATTS_TABLE_DEMO: ota-data = 244
      I (112708) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112708) GATTS_TABLE_DEMO: ota-data = 244
      I (112738) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112738) GATTS_TABLE_DEMO: ota-data = 244
      I (112768) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112768) GATTS_TABLE_DEMO: ota-data = 244
      I (112818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112818) GATTS_TABLE_DEMO: ota-data = 244
      I (112828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112828) GATTS_TABLE_DEMO: ota-data = 244
      I (112858) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112858) GATTS_TABLE_DEMO: ota-data = 244
      I (112888) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112888) GATTS_TABLE_DEMO: ota-data = 244
      I (112918) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112918) GATTS_TABLE_DEMO: ota-data = 244
      I (112948) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112948) GATTS_TABLE_DEMO: ota-data = 244
      I (112978) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (112978) GATTS_TABLE_DEMO: ota-data = 244
      I (113008) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113008) GATTS_TABLE_DEMO: ota-data = 244
      I (113038) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113038) GATTS_TABLE_DEMO: ota-data = 244
      I (113068) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113068) GATTS_TABLE_DEMO: ota-data = 244
      I (113098) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113098) GATTS_TABLE_DEMO: ota-data = 244
      I (113128) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113128) GATTS_TABLE_DEMO: ota-data = 244
      I (113158) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113158) GATTS_TABLE_DEMO: ota-data = 244
      I (113188) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113188) GATTS_TABLE_DEMO: ota-data = 244
      I (113218) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113218) GATTS_TABLE_DEMO: ota-data = 244
      I (113248) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113248) GATTS_TABLE_DEMO: ota-data = 244
      I (113298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113298) GATTS_TABLE_DEMO: ota-data = 244
      I (113308) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113308) GATTS_TABLE_DEMO: ota-data = 244
      I (113338) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113338) GATTS_TABLE_DEMO: ota-data = 244
      I (113368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113368) GATTS_TABLE_DEMO: ota-data = 244
      I (113398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113398) GATTS_TABLE_DEMO: ota-data = 244
      I (113428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113428) GATTS_TABLE_DEMO: ota-data = 244
      I (113458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113458) GATTS_TABLE_DEMO: ota-data = 244
      I (113488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113488) GATTS_TABLE_DEMO: ota-data = 244
      I (113518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113518) GATTS_TABLE_DEMO: ota-data = 244
      I (113548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113548) GATTS_TABLE_DEMO: ota-data = 244
      I (113578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113578) GATTS_TABLE_DEMO: ota-data = 244
      I (113608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113608) GATTS_TABLE_DEMO: ota-data = 244
      I (113638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113638) GATTS_TABLE_DEMO: ota-data = 244
      I (113668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113668) GATTS_TABLE_DEMO: ota-data = 244
      I (113698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113698) GATTS_TABLE_DEMO: ota-data = 244
      I (113728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113728) GATTS_TABLE_DEMO: ota-data = 244
      I (113758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113758) GATTS_TABLE_DEMO: ota-data = 244
      I (113808) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113808) GATTS_TABLE_DEMO: ota-data = 244
      I (113818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113818) GATTS_TABLE_DEMO: ota-data = 244
      I (113848) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113848) GATTS_TABLE_DEMO: ota-data = 244
      I (113878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113878) GATTS_TABLE_DEMO: ota-data = 244
      I (113908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113908) GATTS_TABLE_DEMO: ota-data = 244
      I (113938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113938) GATTS_TABLE_DEMO: ota-data = 244
      I (113968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113968) GATTS_TABLE_DEMO: ota-data = 244
      I (113998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (113998) GATTS_TABLE_DEMO: ota-data = 244
      I (114028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114028) GATTS_TABLE_DEMO: ota-data = 244
      I (114058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114058) GATTS_TABLE_DEMO: ota-data = 244
      I (114088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114088) GATTS_TABLE_DEMO: ota-data = 244
      I (114118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114118) GATTS_TABLE_DEMO: ota-data = 244
      I (114148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114148) GATTS_TABLE_DEMO: ota-data = 244
      I (114178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114178) GATTS_TABLE_DEMO: ota-data = 244
      I (114208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114208) GATTS_TABLE_DEMO: ota-data = 244
      I (114238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114238) GATTS_TABLE_DEMO: ota-data = 244
      I (114268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114268) GATTS_TABLE_DEMO: ota-data = 244
      I (114318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114318) GATTS_TABLE_DEMO: ota-data = 244
      I (114328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114328) GATTS_TABLE_DEMO: ota-data = 244
      I (114358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114358) GATTS_TABLE_DEMO: ota-data = 244
      I (114388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114388) GATTS_TABLE_DEMO: ota-data = 244
      I (114418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114418) GATTS_TABLE_DEMO: ota-data = 244
      I (114448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114448) GATTS_TABLE_DEMO: ota-data = 244
      I (114478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114478) GATTS_TABLE_DEMO: ota-data = 244
      I (114508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114508) GATTS_TABLE_DEMO: ota-data = 244
      I (114538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114538) GATTS_TABLE_DEMO: ota-data = 244
      I (114568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114568) GATTS_TABLE_DEMO: ota-data = 244
      I (114598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114598) GATTS_TABLE_DEMO: ota-data = 244
      I (114628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114628) GATTS_TABLE_DEMO: ota-data = 244
      I (114658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114658) GATTS_TABLE_DEMO: ota-data = 244
      I (114688) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114688) GATTS_TABLE_DEMO: ota-data = 244
      I (114718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114718) GATTS_TABLE_DEMO: ota-data = 244
      I (114748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114748) GATTS_TABLE_DEMO: ota-data = 244
      I (114778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114778) GATTS_TABLE_DEMO: ota-data = 244
      I (114828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114828) GATTS_TABLE_DEMO: ota-data = 244
      I (114848) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114848) GATTS_TABLE_DEMO: ota-data = 244
      I (114878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114878) GATTS_TABLE_DEMO: ota-data = 244
      I (114908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114908) GATTS_TABLE_DEMO: ota-data = 244
      I (114938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114938) GATTS_TABLE_DEMO: ota-data = 244
      I (114968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114968) GATTS_TABLE_DEMO: ota-data = 244
      I (114998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (114998) GATTS_TABLE_DEMO: ota-data = 244
      I (115028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115028) GATTS_TABLE_DEMO: ota-data = 244
      I (115058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115058) GATTS_TABLE_DEMO: ota-data = 244
      I (115088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115088) GATTS_TABLE_DEMO: ota-data = 244
      I (115118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115118) GATTS_TABLE_DEMO: ota-data = 244
      I (115148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115148) GATTS_TABLE_DEMO: ota-data = 244
      I (115178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115178) GATTS_TABLE_DEMO: ota-data = 244
      I (115208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115208) GATTS_TABLE_DEMO: ota-data = 244
      I (115238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115238) GATTS_TABLE_DEMO: ota-data = 244
      I (115268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115268) GATTS_TABLE_DEMO: ota-data = 244
      I (115318) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115318) GATTS_TABLE_DEMO: ota-data = 244
      I (115328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115328) GATTS_TABLE_DEMO: ota-data = 244
      I (115358) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115358) GATTS_TABLE_DEMO: ota-data = 244
      I (115388) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115388) GATTS_TABLE_DEMO: ota-data = 244
      I (115418) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115418) GATTS_TABLE_DEMO: ota-data = 244
      I (115448) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115448) GATTS_TABLE_DEMO: ota-data = 244
      I (115478) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115478) GATTS_TABLE_DEMO: ota-data = 244
      I (115508) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115508) GATTS_TABLE_DEMO: ota-data = 244
      I (115538) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115538) GATTS_TABLE_DEMO: ota-data = 244
      I (115568) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115568) GATTS_TABLE_DEMO: ota-data = 244
      I (115598) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115598) GATTS_TABLE_DEMO: ota-data = 244
      I (115628) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115628) GATTS_TABLE_DEMO: ota-data = 244
      I (115658) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115658) GATTS_TABLE_DEMO: ota-data = 244
      I (115688) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115688) GATTS_TABLE_DEMO: ota-data = 244
      I (115718) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115718) GATTS_TABLE_DEMO: ota-data = 244
      I (115748) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115748) GATTS_TABLE_DEMO: ota-data = 244
      I (115778) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115778) GATTS_TABLE_DEMO: ota-data = 244
      I (115828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115828) GATTS_TABLE_DEMO: ota-data = 244
      I (115838) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115838) GATTS_TABLE_DEMO: ota-data = 244
      I (115868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115868) GATTS_TABLE_DEMO: ota-data = 244
      I (115898) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115898) GATTS_TABLE_DEMO: ota-data = 244
      I (115928) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115928) GATTS_TABLE_DEMO: ota-data = 244
      I (115958) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115958) GATTS_TABLE_DEMO: ota-data = 244
      I (115988) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (115988) GATTS_TABLE_DEMO: ota-data = 244
      I (116018) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116018) GATTS_TABLE_DEMO: ota-data = 244
      I (116048) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116048) GATTS_TABLE_DEMO: ota-data = 244
      I (116078) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116078) GATTS_TABLE_DEMO: ota-data = 244
      I (116108) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116108) GATTS_TABLE_DEMO: ota-data = 244
      I (116138) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116138) GATTS_TABLE_DEMO: ota-data = 244
      I (116168) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116168) GATTS_TABLE_DEMO: ota-data = 244
      I (116198) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116198) GATTS_TABLE_DEMO: ota-data = 244
      I (116228) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116228) GATTS_TABLE_DEMO: ota-data = 244
      I (116258) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116258) GATTS_TABLE_DEMO: ota-data = 244
      I (116288) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116288) GATTS_TABLE_DEMO: ota-data = 244
      I (116348) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116348) GATTS_TABLE_DEMO: ota-data = 244
      I (116368) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116368) GATTS_TABLE_DEMO: ota-data = 244
      I (116398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116398) GATTS_TABLE_DEMO: ota-data = 244
      I (116428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116428) GATTS_TABLE_DEMO: ota-data = 244
      I (116458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116458) GATTS_TABLE_DEMO: ota-data = 244
      I (116488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116488) GATTS_TABLE_DEMO: ota-data = 244
      I (116518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116518) GATTS_TABLE_DEMO: ota-data = 244
      I (116548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116548) GATTS_TABLE_DEMO: ota-data = 244
      I (116578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116578) GATTS_TABLE_DEMO: ota-data = 244
      I (116608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116608) GATTS_TABLE_DEMO: ota-data = 244
      I (116638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116638) GATTS_TABLE_DEMO: ota-data = 244
      I (116668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116668) GATTS_TABLE_DEMO: ota-data = 244
      I (116698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116698) GATTS_TABLE_DEMO: ota-data = 244
      I (116728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116728) GATTS_TABLE_DEMO: ota-data = 244
      I (116758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116758) GATTS_TABLE_DEMO: ota-data = 244
      I (116788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116788) GATTS_TABLE_DEMO: ota-data = 244
      I (116818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116818) GATTS_TABLE_DEMO: ota-data = 244
      I (116868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116868) GATTS_TABLE_DEMO: ota-data = 244
      I (116878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116878) GATTS_TABLE_DEMO: ota-data = 244
      I (116908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116908) GATTS_TABLE_DEMO: ota-data = 244
      I (116938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116938) GATTS_TABLE_DEMO: ota-data = 244
      I (116968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116968) GATTS_TABLE_DEMO: ota-data = 244
      I (116998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (116998) GATTS_TABLE_DEMO: ota-data = 244
      I (117028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117028) GATTS_TABLE_DEMO: ota-data = 244
      I (117058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117058) GATTS_TABLE_DEMO: ota-data = 244
      I (117088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117088) GATTS_TABLE_DEMO: ota-data = 244
      I (117118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117118) GATTS_TABLE_DEMO: ota-data = 244
      I (117148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117148) GATTS_TABLE_DEMO: ota-data = 244
      I (117178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117178) GATTS_TABLE_DEMO: ota-data = 244
      I (117208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117208) GATTS_TABLE_DEMO: ota-data = 244
      I (117238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117238) GATTS_TABLE_DEMO: ota-data = 244
      I (117268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117268) GATTS_TABLE_DEMO: ota-data = 244
      I (117298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117298) GATTS_TABLE_DEMO: ota-data = 244
      I (117328) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117328) GATTS_TABLE_DEMO: ota-data = 244
      I (117378) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117378) GATTS_TABLE_DEMO: ota-data = 244
      I (117398) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117398) GATTS_TABLE_DEMO: ota-data = 244
      I (117428) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117428) GATTS_TABLE_DEMO: ota-data = 244
      I (117458) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117458) GATTS_TABLE_DEMO: ota-data = 244
      I (117488) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117488) GATTS_TABLE_DEMO: ota-data = 244
      I (117518) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117518) GATTS_TABLE_DEMO: ota-data = 244
      I (117548) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117548) GATTS_TABLE_DEMO: ota-data = 244
      I (117578) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117578) GATTS_TABLE_DEMO: ota-data = 244
      I (117608) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117608) GATTS_TABLE_DEMO: ota-data = 244
      I (117638) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117638) GATTS_TABLE_DEMO: ota-data = 244
      I (117668) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117668) GATTS_TABLE_DEMO: ota-data = 244
      I (117698) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117698) GATTS_TABLE_DEMO: ota-data = 244
      I (117728) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117728) GATTS_TABLE_DEMO: ota-data = 244
      I (117758) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117758) GATTS_TABLE_DEMO: ota-data = 244
      I (117788) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117788) GATTS_TABLE_DEMO: ota-data = 244
      I (117818) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117818) GATTS_TABLE_DEMO: ota-data = 244
      I (117868) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117868) GATTS_TABLE_DEMO: ota-data = 244
      I (117878) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117878) GATTS_TABLE_DEMO: ota-data = 244
      I (117908) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117908) GATTS_TABLE_DEMO: ota-data = 244
      I (117938) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117938) GATTS_TABLE_DEMO: ota-data = 244
      I (117968) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117968) GATTS_TABLE_DEMO: ota-data = 244
      I (117998) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (117998) GATTS_TABLE_DEMO: ota-data = 244
      I (118028) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118028) GATTS_TABLE_DEMO: ota-data = 244
      I (118058) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118058) GATTS_TABLE_DEMO: ota-data = 244
      I (118088) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118088) GATTS_TABLE_DEMO: ota-data = 244
      I (118118) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118118) GATTS_TABLE_DEMO: ota-data = 244
      I (118148) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118148) GATTS_TABLE_DEMO: ota-data = 244
      I (118178) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118178) GATTS_TABLE_DEMO: ota-data = 244
      I (118208) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118208) GATTS_TABLE_DEMO: ota-data = 244
      I (118238) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118238) GATTS_TABLE_DEMO: ota-data = 244
      I (118268) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 244, value :
      I (118268) GATTS_TABLE_DEMO: ota-data = 244
      I (118298) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 44, value len = 188, value :
      I (118298) GATTS_TABLE_DEMO: ota-data = 188
      I (118828) GATTS_TABLE_DEMO: GATT_WRITE_EVT, handle = 42, value len = 1, value :
      I (118828) GATTS_TABLE_DEMO: ota-control = 3
      I (118828) GATTS_TABLE_DEMO: ======endota======
      I (118828) esp_image: segment 0: paddr=00190020 vaddr=3c020020 size=05fa8h ( 24488) map
      I (118848) esp_image: segment 1: paddr=00195fd0 vaddr=3fc89a00 size=01a90h (  6800)
      I (118848) esp_image: segment 2: paddr=00197a68 vaddr=40380000 size=085b0h ( 34224)
      I (118858) esp_image: segment 3: paddr=001a0020 vaddr=42000020 size=15b1ch ( 88860) map
      I (118878) esp_image: segment 4: paddr=001b5b44 vaddr=403885b0 size=013a0h (  5024)
      I (118878) esp_image: segment 5: paddr=001b6eec vaddr=50000000 size=00010h (    16)
      I (118888) esp_image: segment 0: paddr=00190020 vaddr=3c020020 size=05fa8h ( 24488) map
      I (118898) esp_image: segment 1: paddr=00195fd0 vaddr=3fc89a00 size=01a90h (  6800)
      I (118898) esp_image: segment 2: paddr=00197a68 vaddr=40380000 size=085b0h ( 34224)
      I (118918) esp_image: segment 3: paddr=001a0020 vaddr=42000020 size=15b1ch ( 88860) map
      I (118928) esp_image: segment 4: paddr=001b5b44 vaddr=403885b0 size=013a0h (  5024)
      I (118928) esp_image: segment 5: paddr=001b6eec vaddr=50000000 size=00010h (    16)
      I (118988) GATTS_TABLE_DEMO: Prepare to restart system!
      x��tESP-ROM:esp32c3-api1-20210207
      Build:Feb  7 2021
      rst:0xc (RTC_SW_CPU_RST),boot:0xc (SPI_FAST_FLASH_BOOT)
      Saved PC:0x40383922
      0x40383922: esp_restart_noos atIDF/components/esp32c3/system_api_esp32c3.c:137 (discriminator 1)
      SPIWP:0xee
      mode:DIO, clock div:1
      load:0x3fcd6100,len:0x1774
      load:0x403ce000,len:0x8d4
      load:0x403d0000,len:0x293c
      entry 0x403ce000
      I (35) boot: ESP-IDF v4.3 2nd stage bootloader
      I (35) boot: compile time 10:36:47
      I (35) boot: chip revision: 3
      I (37) boot.esp32c3: SPI Speed      : 80MHz
      I (41) boot.esp32c3: SPI Mode       : DIO
      I (46) boot.esp32c3: SPI Flash Size : 4MB
      I (51) boot: Enabling RNG early entropy source...
      I (56) boot: Partition Table:
      I (60) boot: ## Label            Usage          Type ST Offset   Length
      I (67) boot:  0 nvs              WiFi data        01 02 00009000 00004000
      I (75) boot:  1 otadata          OTA data         01 00 0000d000 00002000
      I (82) boot:  2 phy_init         RF data          01 01 0000f000 00001000
      I (89) boot:  3 ota_0            OTA app          00 10 00010000 00180000
      I (97) boot:  4 ota_1            OTA app          00 11 00190000 00180000
      I (104) boot: End of partition table
      I (109) esp_image: segment 0: paddr=00190020 vaddr=3c020020 size=05fa8h ( 24488) map
      I (121) esp_image: segment 1: paddr=00195fd0 vaddr=3fc89a00 size=01a90h (  6800) load
      I (127) esp_image: segment 2: paddr=00197a68 vaddr=40380000 size=085b0h ( 34224) load
      I (140) esp_image: segment 3: paddr=001a0020 vaddr=42000020 size=15b1ch ( 88860) map
      I (156) esp_image: segment 4: paddr=001b5b44 vaddr=403885b0 size=013a0h (  5024) load
      I (158) esp_image: segment 5: paddr=001b6eec vaddr=50000000 size=00010h (    16) load
      I (165) boot: Loaded app from partition at offset 0x190000
      I (168) boot: Disabling RNG early entropy source...
      I (185) cpu_start: Pro cpu up.
      I (197) cpu_start: Pro cpu start user code
      I (197) cpu_start: cpu freq: 160000000
      I (197) cpu_start: Application information:
      I (200) cpu_start: Project name:     blink
      I (205) cpu_start: App version:      1
      I (209) cpu_start: Compile time:     Sep  9 2021 15:24:34
      I (215) cpu_start: ELF file SHA256:  09b3ffeaba99067a...
      I (221) cpu_start: ESP-IDF:          v4.3
      I (226) heap_init: Initializing. RAM available for dynamic allocation:
      I (233) heap_init: At 3FC8C2D0 len 00033D30 (207 KiB): DRAM
      I (240) heap_init: At 3FCC0000 len 0001F060 (124 KiB): STACK/DRAM
      I (246) heap_init: At 50000010 len 00001FF0 (7 KiB): RTCRAM
      I (253) spi_flash: detected chip: generic
      I (257) spi_flash: flash io: dio
      I (262) sleep: Configure to isolate all GPIO pins in sleep state
      I (268) sleep: Enable automatic switching of GPIO sleep configuration
      I (275) cpu_start: Starting scheduler.
      I (280) gpio: GPIO[5]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0
      Turning off the LED
      Turning on the LED
      Turning off the LED
      Turning on the LED
      Turning off the LED
      Turning on the LED
      Turning off the LED
