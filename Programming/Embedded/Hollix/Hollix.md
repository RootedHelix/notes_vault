## LCD Screen:
https://www.tslcd.com/3-0-inch-432xrgbx432-rgb-interface-hx8363-a-ic-tft-lcd-display-module-1000-nits-normally-black-50g-tft-display-screen_p712.html

- the size of the projection directly correlates with the size of the LCD ie. if the desired projection is 3” then a 3” dimension LCD should be used

[[TSD 3.0 inch 432×432 RGB TFT module with the HX8363-A driver Quote]]

#### 1000 Nits 480x480 High Brightness Display from "Display Module" : 

https://www.displaymodule.com/products/4-inch-ips-high-brightness-display-480x480-1000nits-with-rgb?variant=20413601579106&country=US&currency=USD&utm_source=chatgpt.com


## MCU:   
Current - 
STM32H750XBH6

Target Prototype: 

> Now Target: NUCLEO-H753ZI

original but discontinued: NUCLEO-H743ZI2

Amazon

Target -
STM32H7 (H743 / H750 class)

  

## Modern high-voltage backlight driver IC: 


Purchased from Digikey - TPS61176EVM-566

- Prototype: https://www.adafruit.com/product/6354?srsltid=AfmBOoo31U2-IeQuNC7xwyEOULczhZbMNIPAeokWE5LPb7IQpbjKIDMF&utm_source=chatgpt.com
- product grade:   MPS MP3310

## Battery Charging / Power Path Control 

MCP73871 Click
-  True **power-path / load-sharing** — run system while charging  
-  Proper **LiPo charge control** (CC/CV profile)  
- Automatic **USB vs battery switching**  
-  Charge current programming  
-  Status outputs for LEDs or MCU

The **MCP73871** manages **battery charging and power-path control**, but it **does not provide a regulated voltage suitable for the MCU**. You still need a **voltage regulator** between the charger and the MCU.

## Voltage Regulator

Purchased: SparkFun AP63203 3.3 --> NEEDS Capacitors

order # is: 000245170.



Buck regulator (3)

Buck Converter (more efficient)

Examples:

- TPS62162
    
- AP63203
    

Advantages:

- high efficiency
    
- longer battery life
    

Disadvantages:

- slightly more complex

## Power

LiPo 4000 mAh 1S pack


https://meshnology.com/products/3-7v-4000mah-lipo-battery-with-usb-charger-for-arduino-esp32-drone?variant=46431264080110&utm_source=chatgpt.com

#### Power Adapter: 

 FainWan 20 Pairs JST 1.25mm 2 Pin Micro Male Female Connector Plug 80mm Wires Cables

https://www.amazon.com/FainWan-1-25mm-Female-Connector-Cables/dp/B09PN4149L/ref=nav_signin?crid=15OSC5SFDIXBG&dib=eyJ2IjoiMSJ9.ovIv-HhYES_1TIPOqIWmr7qbrHvMfg6xAP16UJZGUa9qpzWi_HkQECq9eINAp_nRJAKxeuexkZ2J1WHOakQaS9mJXczOwTiLwSCH63fQQ6XL1nb-fecs0S25Hix3-MrOOwHWA2aIAO-Z0D66paXW5ZScMzmorkf7BFJdExQqpbx6SZ2U41XRzDRiY8_DJq7FOF7-hiKLa-b_O_XISnYOKm6ePrf9qu8rNmhyxLjw5d4.LBfLF3tqUU9klQywqyFKtCCfOWOVoZkjbjhzXYj-e8o&dib_tag=se&keywords=JST-1.25+2-pin+pigtail+cable.&qid=1773379802&sprefix=jst-1.25+2-pin+pigtail+cable.%2Caps%2C169&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1

### Your System Likely Draws:



- STM32H750 + logic → ~100–200 mA
    
- 1000-nit backlight → 400–800 mA
    
- Boost converter losses → ~10–15%
    

Total realistic average:  
**600–900 mA**

### 🛠️ Example Typical Application (Design Insight)

In a proper MCP73871 design:

- USB 5 V (from USB-C port) feeds the **MCP73871 IN**
    
- **OUT** goes to your system load (STM32 + display + boost converter)
    
- **VBAT** connects to your 4000 mAh LiPo pack
    
- MCP73871 handles:
    
    - charging the battery
        
    - powering your system directly when USB is present
        
    - switching to battery seamlessly when USB is removed
        
    - prioritizing system load over battery charge if input power is limite