Overview:
CapeCast Retro takes the classic, beloved Nokia 5110 LCD and gives it a new life as a dedicated desktop weather display. Powered by the Wemos D1 Mini (ESP8266), this project ditches unreliable local sensors (like the DHT11) in favor of high-accuracy, live meteorological data pulled directly from the Open-Meteo API over Wi-Fi.

Key Features:

Retro Aesthetic, Modern Data: Utilizes the iconic 84x48 monochrome Nokia LCD, complete with adjustable backlight, to display hyper-local Cape Town weather.

Custom Vector Graphics: No bulky image arrays here. The ESP8266 uses the Adafruit GFX library to draw dynamic, scalable weather icons (sun, moon, clouds, rain, snow, and lightning) on the fly, adapting to the current weather code.

Dual-Page UI: A single tactile button toggles between two distinct pages:

Weather Dashboard: Large temperature readout, weather condition, "Feels Like" temp, and wind speed.
NTP Clock: A giant, timezone-adjusted digital clock (SAST) with a formatted date.
Auto-Sync & Refresh: Automatically synchronizes time via NTP on boot and silently fetches fresh weather data every 10 minutes in the background.

Compact & Efficient: Uses Software SPI for the display, keeping the wiring minimal and leaving plenty of GPIO pins free for future expansions.

Hardware Used:

Wemos D1 Mini (ESP8266)

Nokia 5110 LCD Display (PCD8544 controller)

Momentary Push Button

330Ω Resistor (for the backlight)

Wiring:

LCD RST → D2

LCD CE/CS → D1

LCD DC → D6

LCD DIN/MOSI → D7

LCD CLK/SCK → D5

LCD VCC → 3.3V

LCD GND → GND

LCD BL → 3.3V (via 330Ω resistor)

Button → D3 (GPIO0) & GND

Software & Libraries:

Arduino IDE

ESP8266 Board Package

Adafruit PCD8544 & Adafruit GFX (for the display and custom graphics)

ArduinoJson (for parsing the Open-Meteo API)

Open-Meteo API (Free, no API key required)

How it works:
On boot, the ESP8266 connects to Wi-Fi and synchronizes its internal clock via NTP (using setenv to handle the SAST timezone). It then queries the Open-Meteo API for current weather conditions. The JSON response is parsed, and the Adafruit GFX library is used to draw scaled-down icons and text onto the Nokia display. The code is structured to be non-blocking, ensuring the Wi-Fi connection remains stable while the button and screen remain responsive. (Note: Because these LCDs are often mounted upside down, display.setRotation(2); is used in the setup to ensure the screen is right-side up!)

Physical Screen: 84 pixels wide × 48 pixels tall
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Bank 0  ████████████████████████████████████████████████  │ ← Y=0 (pixels 0-7)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
├─────────────────────────────────────────────────────────────┤
│  Bank 1  ████████████████████████████████████████████████  │ ← Y=1 (pixels 8-15)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
├─────────────────────────────────────────────────────────────┤
│  Bank 2  ████████████████████████████████████████████████  │ ← Y=2 (pixels 16-23)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
├─────────────────────────────────────────────────────────────┤
│  Bank 3  ████████████████████████████████████████████████  │ ← Y=3 (pixels 24-31)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
├─────────────────────────────────────────────────────────────┤
│  Bank 4  ████████████████████████████████████████████████  │ ← Y=4 (pixels 32-39)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
├─────────────────────────────────────────────────────────────┤
│  Bank 5  ████████████████████████████████████████████████  │ ← Y=5 (pixels 40-47)
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
│          ████████████████████████████████████████████████  │
└─────────────────────────────────────────────────────────────┘
  X=0                                                       X=83
Key points illustrated:
X axis (0–83): moves horizontally, one pixel at a time.

Y axis (0–5): selects one of 6 banks, each bank is 8 pixels tall.

When you call setCursor(x, y), the controller jumps to column x and the top of bank y.

Writing a byte fills 8 vertical pixels in that column (one bit per row within the bank).

What happens when you write a character:

setCursor(0, 1)  →  jumps here:
                    ┌───┐
                    │ ▼ │  ← X=0, top of Bank 1
┌───────────────────┼───┼─────────────────────────────────────┐
│  Bank 0           │   │                                     │
├───────────────────┼───┼─────────────────────────────────────┤
│  Bank 1           │ █ │                                     │ ← your byte fills
│                   │ █ │                                     │   8 pixels vertically
│                   │ █ │                                     │   in this column
│                   │ █ │                                     │
│                   │ █ │                                     │
│                   │ █ │                                     │
│                   │ █ │                                     │
│                   │ █ │                                     │
├───────────────────┼───┼─────────────────────────────────────┤
│  Bank 2           │   │                                     │
└───────────────────┴───┴─────────────────────────────────────┘
                   X=0
Common confusion:
You might expect	Reality
setCursor(0, 8) = next line down	  ❌ Y max is 5 (only 6 banks)
Y moves one pixel	                  ❌ Y moves 8 pixels (one whole bank)
Cursor = pixel coordinate	          ❌ Cursor = (column, bank) address
So to place text on consecutive lines, you increment Y by 1, not by 8 — even though the text visually is 8 pixels tall. That's the "grid" trick: the hardware groups pixels into banks of 8, and the cursor only addresses banks, not individual rows.
