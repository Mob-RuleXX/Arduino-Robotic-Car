// Define the pins for the joystick
int joyXPin = A0; // Joystick X-axis
int joyYPin = A1; // Joystick Y-axis
int joySWPin = 2; // Joystick switch (SW)

// Define the pins for the motors
int motor1Pin1 = 3; // Motor 1 direction pin 1
int motor1Pin2 = 4; // Motor 1 direction pin 2
int motor1SpeedPin = 5; // Motor 1 speed (PWM) pin

int motor2Pin1 = 6; // Motor 2 direction pin 1
int motor2Pin2 = 7; // Motor 2 direction pin 2
int motor2SpeedPin = 9; // Motor 2 speed (PWM) pin

// Define the pins for the ultrasonic sensor
int trigPin = 10; // Ultrasonic sensor trigger pin
int echoPin = 11; // Ultrasonic sensor echo pin

// Define the pin for the pushbutton
int buttonPin = 12; // Pushbutton pin

// Variable to store the joystick mode state
int joystickMode = 0; // 0 for disabled, 1 for enabled

// Variable to store the current mode (0 = joystick, 1 = ultrasonic)
int currentMode = 0;

// Variable to store the previous state of the joystick switch
int lastSWState = HIGH;

// Variable to store the previous state of the pushbutton
int lastButtonState = HIGH;

void setup() {
  // Initialize motor pins as outputs
  pinMode(motor1Pin1, OUTPUT);
  pinMode(motor1Pin2, OUTPUT);
  pinMode(motor1SpeedPin, OUTPUT);

  pinMode(motor2Pin1, OUTPUT);
  pinMode(motor2Pin2, OUTPUT);
  pinMode(motor2SpeedPin, OUTPUT);

  // Initialize joystick pins as inputs
  pinMode(joyXPin, INPUT);
  pinMode(joyYPin, INPUT);
  pinMode(joySWPin, INPUT_PULLUP); // Enable internal pull-up resistor for the switch

  // Initialize ultrasonic sensor pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Initialize pushbutton pin
  pinMode(buttonPin, INPUT_PULLUP); // Enable internal pull-up resistor for the pushbutton

  Serial.begin(9600); // Initialize serial communication for debugging
}

void loop() {
  // Read the current state of the joystick switch
  int currentSWState = digitalRead(joySWPin);

  // Read the current state of the pushbutton
  int currentButtonState = digitalRead(buttonPin);

  // Toggle the joystick mode if the switch is pressed and released
  if (currentSWState == LOW) {
    if (lastSWState == HIGH) {
      joystickMode = 1 - joystickMode; // Toggle between 0 and 1
      delay(200); // Debounce delay
    }
  }

  // Switch to ultrasonic mode if the pushbutton is pressed and released
  if (currentButtonState == LOW) {
    if (lastButtonState == HIGH) {
      currentMode = 1; // Switch to ultrasonic mode
      joystickMode = 0; // Disable joystick mode
      delay(200); // Debounce delay
    }
  }

  // Update the last switch state
  lastSWState = currentSWState;

  // Update the last button state
  lastButtonState = currentButtonState;

  if (currentMode == 0) { // Joystick mode
    if (joystickMode == 1) {
      // Read the joystick values
      int joyX = analogRead(joyXPin);
      int joyY = analogRead(joyYPin);

      // Map the joystick values to motor speed (-255 to 255)
      int motor1Speed = map(joyX, 0, 1023, -255, 255);
      int motor2Speed = map(joyY, 0, 1023, -255, 255);

      // Control motor 1
      if (motor1Speed > 0) {
        digitalWrite(motor1Pin1, HIGH);
        digitalWrite(motor1Pin2, LOW);
        analogWrite(motor1SpeedPin, motor1Speed);
      } else if (motor1Speed < 0) {
        digitalWrite(motor1Pin1, LOW);
        digitalWrite(motor1Pin2, HIGH);
        analogWrite(motor1SpeedPin, -motor1Speed);
      } else {
        digitalWrite(motor1Pin1, LOW);
        digitalWrite(motor1Pin2, LOW);
        analogWrite(motor1SpeedPin, 0);
      }

      // Control motor 2
      if (motor2Speed > 0) {
        digitalWrite(motor2Pin1, HIGH);
        digitalWrite(motor2Pin2, LOW);
        analogWrite(motor2SpeedPin, motor2Speed);
      } else if (motor2Speed < 0) {
        digitalWrite(motor2Pin1, LOW);
        digitalWrite(motor2Pin2, HIGH);
        analogWrite(motor2SpeedPin, -motor2Speed);
      } else {
        digitalWrite(motor2Pin1, LOW);
        digitalWrite(motor2Pin2, LOW);
        analogWrite(motor2SpeedPin, 0);
      }
    } else {
      // If joystick mode is disabled, stop the motors
      digitalWrite(motor1Pin1, LOW);
      digitalWrite(motor1Pin2, LOW);
      analogWrite(motor1SpeedPin, 0);

      digitalWrite(motor2Pin1, LOW);
      digitalWrite(motor2Pin2, LOW);
      analogWrite(motor2SpeedPin, 0);
    }
  } else { // Ultrasonic mode
    long duration, distance;

    // Send a pulse to the ultrasonic sensor
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    // Read the echo pin and calculate the distance
    duration = pulseIn(echoPin, HIGH);
    distance = duration * 0.034 / 2;

    Serial.print("Distance: ");
    Serial.println(distance);

    // Control the motors based on distance
    if (distance < 20) { // Obstacle detected within 20 cm
      // Reverse and turn to avoid obstacle
      digitalWrite(motor1Pin1, LOW);
      digitalWrite(motor1Pin2, HIGH);
      analogWrite(motor1SpeedPin, 150); // Adjust speed as needed

      digitalWrite(motor2Pin1, LOW);
      digitalWrite(motor2Pin2, HIGH);
      analogWrite(motor2SpeedPin, 150); // Adjust speed as needed

      delay(500); // Move backward for a while

      // Turn right
      digitalWrite(motor1Pin1, HIGH);
      digitalWrite(motor1Pin2, LOW);
      analogWrite(motor1SpeedPin, 150);

      digitalWrite(motor2Pin1, LOW);
      digitalWrite(motor2Pin2, LOW);
      analogWrite(motor2SpeedPin, 0);

      delay(500); // Turn right for a while
    } else {
      // Move forward
      digitalWrite(motor1Pin1, HIGH);
      digitalWrite(motor1Pin2, LOW);
      analogWrite(motor1SpeedPin, 150); // Adjust speed as needed

      digitalWrite(motor2Pin1, HIGH);
      digitalWrite(motor2Pin2, LOW);
      analogWrite(motor2SpeedPin, 150); // Adjust speed as needed
    }
  }
  // Delay for a short period to avoid excessive readings
  delay(10);
}
