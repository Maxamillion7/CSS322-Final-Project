#include <DHT11.h>
DHT11 dht11(A0);

int pin[] = {8, 7, 2, 3, 4, 5, 6, 9, 10, 11};
bool lightState = false; // To keep track of the light state for blinking

void setup() {
  Serial.begin(9600);

  // Setup the buzzer
  pinMode(13, OUTPUT);
  digitalWrite(13, HIGH);

  // Setup the 7-segment display
  for (int i = 0; i <= 6; i++) {
    pinMode(pin[i], OUTPUT);
    digitalWrite(pin[i], LOW);
  }

  // Setup the LED light
  for (int i = 7; i <= 9; i++) {
    pinMode(pin[i], OUTPUT);
    digitalWrite(pin[i], LOW);
  }
}

void loop() {
  // Error setting
  bool Error = LOW;

  int temperature = 0;
  int humidity = 0;

  // Read temperature and humidity values
  int result = dht11.readTemperatureHumidity(temperature, humidity);

  if (result == 0) {
    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.print(" °C\tHumidity: ");
    Serial.print(humidity);
    Serial.println(" %");
  } else {
    Serial.println(DHT11::getErrorString(result));
    Error = HIGH; //set that there is an error
  }

  // Control buzzer based on temperature
  bool buzzerState = HIGH; // Default to off

  if (temperature < 22) {
    buzzerState = LOW;       // Buzzer ON
    digitalWrite(pin[7], HIGH);  // BLUE light ON
    digitalWrite(pin[8], LOW);
    digitalWrite(pin[9], LOW);
  } else if (temperature > 29) {
    buzzerState = LOW;       // Buzzer ON
    digitalWrite(pin[7], LOW);
    digitalWrite(pin[8], LOW);
    digitalWrite(pin[9], HIGH);  // RED light ON
  } else {
    digitalWrite(pin[7], LOW);
    digitalWrite(pin[8], HIGH);  // GREEN light ON
    digitalWrite(pin[9], LOW);
  }

  // Control 7-segment display based on humidity
  if (humidity < 40) {
    buzzerState = LOW; // Buzzer ON
    digitalWrite(pin[3], HIGH);  // 'L' on 7-segment
    digitalWrite(pin[4], HIGH);
    digitalWrite(pin[5], HIGH);
    digitalWrite(pin[2], LOW);
    digitalWrite(pin[6], LOW);
    digitalWrite(pin[1], LOW);
    digitalWrite(pin[0], LOW);
  } else if (humidity > 60) {
    buzzerState = LOW; // Buzzer ON
    digitalWrite(pin[6], HIGH);  // 'H' on 7-segment
    digitalWrite(pin[5], HIGH);
    digitalWrite(pin[4], HIGH);
    digitalWrite(pin[2], HIGH);
    digitalWrite(pin[1], HIGH);
    digitalWrite(pin[3], LOW);
    digitalWrite(pin[0], LOW);
  } else {
    digitalWrite(pin[6], HIGH);  // '-' on 7-segment
    digitalWrite(pin[2], LOW);
    digitalWrite(pin[3], LOW);
    digitalWrite(pin[4], LOW);
    digitalWrite(pin[5], LOW);
    digitalWrite(pin[1], LOW);
    digitalWrite(pin[0], LOW);
  }

  // Set buzzer state
  digitalWrite(13, buzzerState);

  if (Error == 1) {
    digitalWrite(pin[6], HIGH);  // 'E' on 7-segment
    digitalWrite(pin[2], LOW);
    digitalWrite(pin[3], HIGH);
    digitalWrite(pin[4], HIGH);
    digitalWrite(pin[5], HIGH);
    digitalWrite(pin[1], LOW);
    digitalWrite(pin[0], HIGH);

    blinkLights();  // Blink all specified lights
  }

  delay(1000);
}

void blinkLights() {
  int lightPins[] = {9, 10, 11};  // Pins to blink
  lightState = !lightState;  // Toggle the light state
  for (int i = 0; i < 3; i++) {
    digitalWrite(lightPins[i], lightState ? HIGH : LOW);
  }
  delay(250);
}
