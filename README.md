# Embedded UltraSonic Range Finder

This project implements an ultrasonic range finder using an ESP32, which measures the distance to an object and displays the results on a 16x2 I2C LCD. A passive buzzer is used to provide auditory feedback, where the tone changes depending on the distance to the detected object.

## Features
- **Distance Measurement**: Measures the distance to an object using an ultrasonic sensor and displays the result on an LCD.
- **Auditory Feedback**: The buzzer emits a tone that changes in frequency based on the proximity to the object (higher tone for closer objects).
- **Serial Output**: Outputs the distance data to the serial monitor for debugging and monitoring.

## Components
- **ESP32 (ESP32-S3-WROOM)**
- **HC-SR04 Ultrasonic Range Finder**
- **16x2 I2C LCD**
- **Passive Buzzer**
- **Wires & Breadboard**

## Pin Configuration

| Component                | GPIO Pin    | Function          |
|--------------------------|-------------|-------------------|
| Ultrasonic Sensor Trig    | GPIO 4      | Trigger signal    |
| Ultrasonic Sensor Echo    | GPIO 5      | Echo signal       |
| LCD SDA                   | GPIO 6      | I2C Data Line     |
| LCD SCL                   | GPIO 7      | I2C Clock Line    |
| Passive Buzzer            | GPIO 15     | Generates sound   |

## Circuit Diagram
- Connect the ultrasonic sensor’s Trig pin to GPIO 4 and Echo pin to GPIO 5.
- Connect the I2C LCD's SDA to GPIO 6 and SCL to GPIO 7.
- Connect the passive buzzer to GPIO 15.

## Code Explanation
The project uses the **LiquidCrystal_I2C** library to interface with the LCD and the **tone()** function to control the buzzer. The ultrasonic sensor calculates the distance by sending out a sound pulse and measuring the time it takes for the echo to return. Based on the distance, the buzzer's tone frequency is adjusted, providing auditory feedback.

```cpp
#include <Wire.h> 
#include <LiquidCrystal_I2C.h> 

// Define pins connected from the ultrasonic sensor to ESP32
const int trigPin = 4;
const int echoPin = 5;

// Initialize the LCD with I2C address 0x27, 16 columns, and 2 rows
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Define the pin for the passive buzzer
const int buzzerPin = 15;

long pingTime;
float distance;

void setup() {
  Serial.begin(115200);
  
  // Set up the ultrasonic sensor pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Initialize the I2C communication for the LCD
  Wire.begin(6, 7); 
  lcd.begin(16, 2);
  lcd.backlight();
  lcd.print("System Ready");
  
  delay(2000);  // Pause for 2 seconds
  lcd.clear();
}

void loop() {
  // Trigger the ultrasonic sensor
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  // Read the echo time and calculate the distance
  pingTime = pulseIn(echoPin, HIGH);
  distance = (float)pingTime * 340.0 / 2.0 / 10000.0;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Display the distance on the LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Distance: ");
  lcd.print(distance);
  lcd.setCursor(0, 1);
  lcd.print(" cm");
  
  // Adjust the buzzer tone based on distance
  if (distance <= 30) {
    int frequency = map(distance, 0, 30, 2000, 100);
    frequency = constrain(frequency, 100, 2000);
    tone(buzzerPin, frequency);
  } else {
    noTone(buzzerPin);
  }

  delay(300);  // Delay before the next measurement
}
```

## How to Use
1. **Hardware Setup**:
   - Assemble the circuit according to the pin configuration table.
2. **Software Setup**:
   - Install the necessary **LiquidCrystal_I2C** library: [LiquidCrystal_I2C](https://github.com/johnrickman/LiquidCrystal_I2C).
   - Upload the provided code to your ESP32 using the Arduino IDE.
3. **Run the Project**:
   - Open the serial monitor to observe the distance readings.
   - Watch the LCD display the distance to the object.
   - Listen for changes in the buzzer tone as the distance to the object changes.

## Future Improvements
- Add a visual graph or LED indicators to represent distance changes.
- Implement Wi-Fi functionality to send distance data to a web server.
- Improve the accuracy of distance measurements by applying noise reduction algorithms.

## License
This project is open-source under the MIT License. Feel free to modify and distribute as needed.
