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



U5 (as marked on PCB) is labelled PHN1 - TBC
U6 (as marked on PCb) is labelled AA1 / 135 / D6 - also TBC

The board also has unsoldered headers for serial comms- Output from these is TBC. 








