# ArduinoML implementation in Elixir

This directory is one quick implementation of ArduinoML in Elixir

## Requirements
- An Elixir installation on your system.
- *optional:* Compile the project with ```mix compile```.
- *optional:* Execute the tests with ```mix test```.
- Run it on a file with ```mix run FILE_PATH``` (example: ```mix run samples/switch.exs```).

## Project structure

- The *abstract syntax* is in the [lib/arduino_ml/model](./lib/arduino_ml/model) folder. It is made of structures, so there is no inheritance nor references.
- The *concrete syntax* is in the file [lib/arduino_ml.ex](./lib/arduino_ml.ex).
- The *validation* is in the file [lib/arduino_ml/model_validation/model_validator.ex](./lib/arduino_ml/model_validation/model_validator.ex).
- The *code generation* is in the folder [lib/arduino_ml/code_production](./lib/arduino_ml/code_production).

## Syntax example

This example can be found in [samples/switch.exs](./samples/switch.exs).

```elixir
import ArduinoML

application "Switch!"

sensor button: 9
actuator led: 12

state :on, on_entry: :led ~> :high
state :off, on_entry: :led ~> :low

initial :off

transition from: :on, to: :off, when: is_high?(:button)
transition from: :off, to: :on, when: is_high?(:button)

finished!
```

This is transpiled into... that creepy code:

```c
// generated by ArduinoML #Elixir.

// Bricks <~> Pins.
int BUTTON = 9;
int LED = 12;

// Setup the inputs and outputs
void setup() {
  pinMode(BUTTON, INPUT);

  pinMode(LED, OUTPUT);
}

// Static setup code.
int state = LOW;
int prev = HIGH;
long time = 0;
long debounce = 200;

// States declarations.
void state_off() {
  digitalWrite(LED, LOW);

  boolean guard = millis() - time > debounce;

  if (digitalRead(BUTTON) == HIGH && guard) {
    time = millis();
    state_on();
  } else {
    state_off();
  }
}

void state_on() {
  digitalWrite(LED, HIGH);

  boolean guard = millis() - time > debounce;

  if (digitalRead(BUTTON) == HIGH && guard) {
    time = millis();
    state_off();
  } else {
    state_on();
  }
}

// This function specify the first state.
void loop() {
  state_off();
}
```

## TODO List

- Implement more examples.
