# Rapt Pill - Info / Teardown

I'm interested in replacing the default RAPT pill firmware with something a little more local as to not rely on cloud servers and business continuity. This document will be updated with my current findings as long as I remain interested.

### Default firmware- web endpoints:

The following have been found through examination of the local web server on the Pill:

POST factory_reset.json

POST set_interval.json 

Post details:
"X-Custom-interval": Math.round(gel("interval").value) * 60
},
body: { timestamp: Date.now() },
});


POST set_beta.json

GET static_info.json

GET get_interval.json

GET sensors.json - *interesting as returns current sensor values*

GET get_dev_buttons.json

GET ap_time.json

POST poke_ap_timeout.json

POST set_dev_opts.json

GET ots_status.json

GET get_calibration_state.json

GET sensors_cal.json 

POST set_calibration_state.json

POST cal_1p.json
POST cal_2p.json

GET get_log.json


As the current firmware enters a deep sleep and only awakes to fire off an update to azure- this is of limited use. 

Browsing to the diag page and enabling test api / beta stream can seems to enable auto- turn off. These options can be exposed on the page itself by entering : ```devOptions(true)``` into the browser console.

### Quick Hardware analysis

The platform is built upon a single PCB with an ESP32-S0WD 

* attached 32MB of flash memory (IS25WP032) https://www.issi.com/WW/pdf/IS25WP032-064-128.pdf

On a part of the board etched out are two sensors: 
* LIS3DH (A C3H gyro) - some info found here: https://www.adafruit.com/product/2809 
* 1AE temp sensor
https://www.alldatasheet.net/view_marking.jsp?Searchword=1AE&sField=4&seekcls=
https://www.ti.com/lit/ds/symlink/tmp1075.pdf?ts=1638081231479&ref_url=https%253A%252F%252Fwww.google.com%252F (WSON package)
https://github.com/PatrickBaus/Arduino-TMP1075

Both of these sensors are I2C sensors. 



U5 (as marked on PCB) is labelled PHN1 - Some form of switching regulator
U6 (as marked on PCb) is labelled AAL / 135 / D6 - MCP73832 https://www.microchip.com/en-us/product/MCP73832

The board also has unsoldered headers for serial comms- see sample serial output below.  

Serial Outputs indicates the following:

I (626) gpio: GPIO[18]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0

I (636) gpio: GPIO[23]| InputEn: 1| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0

I (646) gpio: GPIO[5]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0

I (656) gpio: GPIO[34]| InputEn: 1| OutputEn: 0| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0

From this it looks like the following:
GPIO18 / GPIO5 - both defined as outputs - fairly likely that one is the ESP LED. The other could be the charging LED unless that is directly driven by the charging circuitry

GPIO23 / GPIO34 - defined as inputs: 34 is an ADC so could be used to detect battery charge level. GPIO32 could be used for the I2- unusual as the standard i2c pins are 21 and 22.



```` 
rst:0x5 (DEEPSLEEP_RESET),boot:0x33 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0030,len:7112
load:0x40078000,len:14660
load:0x40080400,len:3472
entry 0x40080640
I (27) boot: ESP-IDF v4.3-dirty 2nd stage bootloader
I (27) boot: compile time 07:23:18
I (27) boot: chip revision: 1
I (30) boot.esp32: SPI Speed      : 40MHz
I (35) boot.esp32: SPI Mode       : DIO
I (40) boot.esp32: SPI Flash Size : 4MB
I (44) boot: Enabling RNG early entropy source...
I (50) esp_image: segment 0: paddr=00190020 vaddr=3f400020 size=31564h (202084) map
I (58) esp_image: segment 1: paddr=001c158c vaddr=3ffb0000 size=049e4h ( 18916) load
I (69) esp_image: segment 2: paddr=001c5f78 vaddr=40080000 size=0a0a0h ( 41120) load
I (81) esp_image: segment 3: paddr=001d0020 vaddr=400d0020 size=9b9c0h (637376) map
I (83) esp_image: segment 4: paddr=0026b9e8 vaddr=4008a0a0 size=0b8b4h ( 47284) load
I (99) esp_image: segment 5: paddr=002772a4 vaddr=400c0000 size=00290h (   656)
I (100) esp_image: segment 6: paddr=0027753c vaddr=50000000 size=008b8h (  2232)
I (108) boot: Fast booting app from partition at offset 0x190000
I (114) boot: Disabling RNG early entropy source...
I (131) cpu_start: Pro cpu up.
I (131) cpu_start: Single core mode
I (142) cpu_start: Pro cpu start user code
I (142) cpu_start: cpu freq: 160000000
I (142) cpu_start: Application information:
I (146) cpu_start: Project name:     rapt-hydrometer
I (152) cpu_start: App version:      20211113_000924_76d5b95
I (158) cpu_start: Compile time:     Nov 13 2021 00:09:38
I (164) cpu_start: ELF file SHA256:  4c85c4f0d75614c2...
I (170) cpu_start: ESP-IDF:          v4.3-dirty
I (176) heap_init: Initializing. RAM available for dynamic allocation:
I (183) heap_init: At 3FF80290 len 00001D50 (7 KiB): RTCRAM
I (189) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (195) heap_init: At 3FFBC1C0 len 00023E40 (143 KiB): DRAM
I (201) heap_init: At 3FFE0440 len 0001FBC0 (126 KiB): D/IRAM
I (208) heap_init: At 40078000 len 00008000 (32 KiB): IRAM
I (214) heap_init: At 40095954 len 0000A6AC (41 KiB): IRAM
I (221) spi_flash: detected chip: issi
I (225) spi_flash: flash io: dio
I (229) sleep: Configure to isolate all GPIO pins in sleep state
I (235) sleep: Enable automatic switching of GPIO sleep configuration
I (242) esp_core_dump_uart: Init core dump to UART
I (248) cpu_start: Starting scheduler on PRO CPU.
I (254) main: Hello world!
I (254) gpio: GPIO[18]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (264) gpio: GPIO[23]| InputEn: 1| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0
I (274) gpio: GPIO[5]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (284) gpio: GPIO[34]| InputEn: 1| OutputEn: 0| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (404) system_api: Base MAC address is not set
I (404) system_api: read default base MAC address from EFUSE
I (404) ota: OTA boot offset 0x00190000, running offset 0x00190000
I (414) ota: Current firmware version: 20211113_000924_76d5b95
I (434) wifi:wifi driver task: 3ffc623c, prio:23, stack:6656, core=0
I (434) wifi:wifi firmware version: c7d0450
I (434) wifi:wifi certification version: v7.0
I (434) wifi:config NVS flash: enabled
I (434) wifi:config nano formating: disabled
I (444) wifi:Init data frame dynamic rx buffer num: 32
I (444) wifi:Init management frame dynamic rx buffer num: 32
I (454) wifi:Init management short buffer num: 32
I (454) wifi:Init dynamic tx buffer num: 32
I (464) wifi:Init static rx buffer size: 1600
I (464) wifi:Init static rx buffer num: 10
I (464) wifi:Init dynamic rx buffer num: 32
I (474) wifi_init: rx ba win: 6
I (474) wifi_init: tcpip mbox: 32
I (484) wifi_init: udp mbox: 6
I (484) wifi_init: tcp mbox: 6
I (484) wifi_init: tcp tx win: 5744
I (494) wifi_init: tcp rx win: 5744
I (494) wifi_init: tcp mss: 1440
I (504) wifi_init: WiFi IRAM OP enabled
I (504) wifi_init: WiFi RX IRAM OP enabled
I (514) wifi_init: rx ba win: 6
I (514) wifi_init: tcpip mbox: 32
I (514) wifi_init: udp mbox: 6
I (524) wifi_init: tcp mbox: 6
I (524) wifi_init: tcp tx win: 5744
I (534) wifi_init: tcp rx win: 5744
I (534) wifi_init: tcp mss: 1440
I (534) wifi_init: WiFi IRAM OP enabled
I (544) wifi_init: WiFi RX IRAM OP enabled
I (554) auth: Auth task started
I (574) accel.c: LIS3DH detected
I (574) accel.c: Configuring LIS3DH
I (664) accel.c: LIS3DH configured
I (664) tmp1075: Attempting to detect TMP1075
I (684) tmp1075: TMP1075 detected and configured
I (684) main: Minimum free heap size: 192116 bytes
I (684) main: Reporting task started
I (694) main: wifi about to be started
I (704) wifi_manager: AP password: kegland1
I (704) wifi_manager: Set STA IP String to: 0.0.0.0
I (1224) phy_init: phy_version 4670,719f9f6,Feb 18 2021,17:07:07
I (1234) wifi_manager:
I (1244) http_server: Registering URI handlers
I (1244) wifi_manager: MESSAGE: ORDER_LOAD_AND_RESTORE_STA
I (1244) wifi_manager: wifi_manager_fetch_wifi_sta_config: ssid:SSID password:***
I (1254) wifi_manager: Saved wifi found on startup. Will attempt to connect.
I (1254) wifi_manager: MESSAGE: ORDER_CONNECT_STA
I (1844) wifi_manager: WIFI_EVENT_STA_CONNECTED
I (3924) esp_netif_handlers: sta ip: 192.168.1.188, mask: 255.255.255.0, gw: 192.168.1.1
I (3924) wifi_manager: IP_EVENT_STA_GOT_IP
I (3924) wifi_manager: WM_EVENT_STA_GOT_IP
I (3924) wifi_manager: Set STA IP String to: 192.168.1.188
I (4334) main: request data: macAddress=xx-xx-xx-xx-xx-xx&temperature=20.875&gravity=904.837&x=415.257&y=-8.34286&z=933.543&battery=0&rssi=-53&deviceType=PillG1&version=20211113_000924_76d5b95
I (6534) main: telemetry status code: 200
I (7134) wifi_manager: wifi_manager_event_loop_mutex released
W (7164) wifi_manager: wifi_manager_event_handler got an event when event group has been freed
W (7164) wifi_manager: wifi_manager_event_handler got an event when event group has been freed
I (7214) wifi_manager: got to end of wifi_manager_stop()
I (7214) power: Going back to deep sleep...
````




