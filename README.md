 # EXPERIMENT-03-INTERFACING-DIGITAL-SENSOR-DHT-11-Temperature-Sensor-and-Rain-Sensor-WITH-EDGE-DEVELOPMENT-BOARD-

---

### **NAME: OVIYA P**  
### **DEPARTMENT: B.E.CSE-IOT**  
### **ROLL NO:212223110033**  
### **DATE OF EXPERIMENT:13/05/2026**  

---

## **AIM:**  
To interface an **Temperature and humidity sensor (DHT 11) Rain Sensor (LM393)** with the **Raspberry Pi 4** and display the sensor readings using HiveMQ cloud.

---

## **APPARATUS REQUIRED:**  
1. Raspberry Pi 4  
2. Temperature and Humidity sensor (DHT-11) and Rain Sensor (LM393)  
3. Jumper Wires  
4. Breadboard  
5. USB Cable  
6. Computer with Thonny IDE  

---

## **THEORY:**  
<img width="1293" height="744" alt="image" src="https://github.com/user-attachments/assets/3c04afa6-1517-45d2-88f1-e671d9ed1ffb" />

 ### FIGURE-01 RASPI PI 4 PINOUT DIAGRAM: 

The Raspberry Pi 4 Model B is built around a Broadcom BCM2711 system-on-chip that integrates a quad-core ARM Cortex-A72 (64-bit) CPU, VideoCore VI GPU, memory controller, and peripheral interfaces, forming a compact yet complete computer architecture where the SoC connects internally to RAM, USB 3.0 controller, Gigabit Ethernet, HDMI display, and wireless modules. Its 40-pin GPIO header provides a flexible pin configuration consisting of power pins (5 V and 3.3 V), multiple ground pins, and general-purpose input/output pins that operate at 3.3 V logic and can be programmed for digital I/O or alternate functions. Key alternate functions include I²C (SDA, SCL) for sensor communication, SPI (MOSI, MISO, SCLK, CS) for high-speed peripheral interfacing, UART (TX, RX) for serial communication, and PWM for control applications.  For communication, I2C (SDA, SCL), SPI (MOSI, MISO, SCK), and UART (TX, RX) interfaces are mapped across different GPIO pins, allowing seamless connectivity with sensors and peripherals. All GPIO pins support PWM (Pulse Width Modulation), making it useful for motor control, LED brightness adjustment, and sound applications. The BOOTSEL button enables USB mass storage mode for firmware flashing, while the DEBUG pins (SWD interface) provide debugging capabilities. With its low power consumption, flexible GPIO options, and rich interface support, the Raspberry Pi Pico is widely used for IoT, embedded systems, robotics, and automation projects.This architecture and pin multiplexing allow the Raspberry Pi 4 to act as both a general-purpose computing platform and an embedded controller, supporting rapid prototyping, hardware interfacing, and IoT applications.
## Temperature and Humidity Sensor (DHT-11):
The DHT11 is a low-cost digital sensor used to measure ambient temperature and relative humidity in embedded and IoT applications. It integrates a thermistor for temperature sensing and a capacitive humidity sensor for detecting moisture levels in the air, along with an internal 8-bit microcontroller that processes the signals and provides calibrated digital output through a single-wire communication interface. The sensor operates typically at 3.3 V to 5 V, measures temperature in the range of 0 °C to 50 °C with ±2 °C accuracy, and humidity from 20% to 80% with ±5% accuracy. Due to its simple interface, low power consumption, and reliable performance, it is widely used in weather monitoring systems, home automation, agricultural monitoring, and basic environmental data acquisition projects.

<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/5c8d35b5-4381-434f-8617-db4b8fe19154" />

### FIGURE-02 Temperature and Humidity Sensor (DHT-11)

## Rain Sensor (LM393):
A rain sensor is one kind of switching device which is used to detect the rainfall. It works like a switch and the working principle of this sensor is, whenever there is rain, the switch will be normally closed. The rain sensor module/board is shown below. Basically, this board includes nickel coated lines and it works on the resistance principle. This sensor module permits to gauge moisture through analog output pins & it gives a digital output while moisture threshold surpasses. This module is similar to the LM393 IC because it includes the electronic module as well as a PCB. Here PCB is used to collect the raindrops. When the rain falls on the board, then it creates a parallel resistance path to calculate through the operational amplifier. This sensor is a resistive dipole, and based on the moisture only it shows the resistance. For example, it shows more resistance when it is dry and shows less resistance when it is wet..
<img width="525" height="240" alt="image" src="https://github.com/user-attachments/assets/fdf4df13-4659-46ed-bbab-44bbae5a360f" />

<img width="443" height="266" alt="image" src="https://github.com/user-attachments/assets/c9750628-c41f-4181-8dcd-e07873d227d9" />

<img width="393" height="120" alt="image" src="https://github.com/user-attachments/assets/997aa631-9647-42cc-94d6-754742f7bf18" />


 ### FIGURE-03 Rain Sensor (LM393) Rain Sensor Module & connection diagram

## Working Principle:
Experiment 3A
The Temperature and Humidity Sensor (DHT-11) OUT is connected to one of the GPIO pins of the Raspberry Pi 4.
The Python script sets the measure the real time temperature and Humidity output and shown in HiveMQcloud with current status and Console.
CIRCUIT DIAGRAM
Connect the Vcc of the Temperature and Humidity Sensor (DHT-11) is connected to +5V in Raspberrry Pi4.
Connect the Gnd of the Temperature and Humidity Sensor (DHT-11) is connected to Gnd in Raspberrry Pi4.
Connect the OUT to any one GPIO.


Experiment 3B
The Rain Sensor (LM393) D0 is connected one of the GPIO pins in Raspberry Pi 4.
The Python script sets the Rain Sensor (LM393) value based on the variation in the rain water (Dry or Wet) and shown in HiveMQ Cloud and console.
CIRCUIT DIAGRAM
Connect the Rain Sensor (LM393) Vcc to any +5V.
Connect the Rain Sensor (LM393) GND to any GND.
Connect the Rain Sensor (LM393) D0 to any one GPIO. 


Experiment 3A
## PROGRAM (Python)
```
import time
import board
import digitalio
import busio
import json
import ssl

from adafruit_mcp3xxx.mcp3008 import MCP3008
from adafruit_mcp3xxx.analog_in import AnalogIn
import adafruit_mcp3xxx.mcp3008 as MCP

from PIL import Image, ImageDraw, ImageFont
import adafruit_ssd1306

import paho.mqtt.client as mqtt

# ---------------- MQTT CONFIG ----------------
BROKER = "eb095bd9c95842ba8d846b05458b5d0f.s1.eu.hivemq.cloud"
PORT = 8883
USERNAME = "hivemq.webclient.1777442120440"
PASSWORD = "2d@<hUTYgx51Lu#8>eGZ"
TOPIC = "raspi/temperature"

# MQTT Setup
client = mqtt.Client()
client.username_pw_set(USERNAME, PASSWORD)
client.tls_set(tls_version=ssl.PROTOCOL_TLS)

client.connect(BROKER, PORT)
client.loop_start()

# ---------------- OLED SETUP ----------------
RESET_PIN = digitalio.DigitalInOut(board.D4)
i2c = board.I2C()
oled = adafruit_ssd1306.SSD1306_I2C(128, 64, i2c, addr=0x3C, reset=RESET_PIN)

# ---------------- SPI + MCP3008 ----------------
spi = busio.SPI(clock=board.SCK, MISO=board.MISO, MOSI=board.MOSI)
cs = digitalio.DigitalInOut(board.D8)
mcp = MCP.MCP3008(spi, cs)

chan3 = AnalogIn(mcp, MCP.P3)

# ---------------- FUNCTION ----------------
def LM35():
    try:
        voltage = chan3.voltage
        temp = (voltage * 330 / 1024) * 100

        print("Temperature:", temp)

        # OLED Display
        oled.fill(0)
        image = Image.new("1", (oled.width, oled.height))
        draw = ImageDraw.Draw(image)

        font = ImageFont.truetype(
            "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf", 16
        )

        draw.text((0, 10), "LM35 Sensor", font=font, fill=255)
        draw.text((0, 30), f"Temp: {temp:.2f} C", font=font, fill=255)

        oled.image(image)
        oled.show()

        # MQTT Publish
        payload = json.dumps({"temperature": temp})
        client.publish(TOPIC, payload)

        print("Published to HiveMQ:", payload)

    except Exception as e:
        print("Error:", e)

# ---------------- LOOP ----------------
while True:
    LM35()
    time.sleep(20)

````

### OUPUT  


# FIGURE -04 
<img width="1599" height="899" alt="WhatsApp Image 2026-05-12 at 2 19 21 PM" src="https://github.com/user-attachments/assets/e6fa9768-73ae-498c-b5b6-a8ac0ef2b488" />

#  FIGURE -05 
<img width="1600" height="765" alt="WhatsApp Image 2026-05-12 at 2 18 31 PM" src="https://github.com/user-attachments/assets/30b50c21-8020-405b-8fcc-8355acbd0798" />

# FIGURE -06  
<img width="643" height="391" alt="WhatsApp Image 2026-05-12 at 2 18 18 PM" src="https://github.com/user-attachments/assets/1434a1d9-df2d-4476-a1d8-e83bcdea491d" />

Experiment 3B
## PROGRAM (Python)
```


 



 
````

### OUPUT  

# FIGURE -07 ADD TITILE HERE 

#  FIGURE -08 ADD TITILE HERE 

# FIGURE -09 ADD TITLE HERE 




## **RESULT:**  
The **Temperature and humidity sensor (DHT 11) Rain Sensor (LM393)** was successfully interfaced with the **Raspberry Pi 4**, and real-time **Temperature, Humidity and Rain status** were read and displayed in Console and HiveMq Cloud. 

---

