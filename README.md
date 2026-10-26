# L298N Motor Not Spinning or Humming — Test Code & Wiring Checklist

Test sketches for Arduino Uno and ESP32 (arduino-esp32 core 3.x), shared ground wiring guide, and jumper configuration for fixing L298N motor driver issues.

---

### 1. 7-Point Jumper & Wiring Checklist
* **Shared Ground (Most Common):** The battery negative (`-`), the L298N `GND` terminal, and your microcontroller's `GND` **must** all be connected together.
* **ENA / ENB Jumpers:** Jumper **ON** = motor runs at 100% speed (your code's PWM does nothing). Jumper **OFF** = ENA must be wired to a PWM pin.
* **5V Regulator Jumper:** 
  * Power supply $\le 12\text{ V}$: Keep jumper **ON** (the L298N 5V terminal becomes an *output* to power low-current circuits, but do *not* connect it to your PC-powered board's 5V pin).
  * Power supply $> 12\text{ V}$: Remove jumper and feed an external regulated 5V into the 5V terminal.
* **Voltage Drop:** The L298N drops about $2\text{ V}$ to $3\text{ V}$. A $5\text{ V}$ supply only gives your motors about $2\text{ V}$ – $3\text{ V}$. Use a $7\text{ V}$ – $12\text{ V}$ power source (like a 2S Li-ion pack or AA battery holder).

---

### 2. Test Code for Arduino Uno
```cpp
const int ENA = 9;   // PWM pin (~)
const int IN1 = 8;
const int IN2 = 7;

void setup() {
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
}

void loop() {
  digitalWrite(IN1, HIGH);  // forward
  digitalWrite(IN2, LOW);
  analogWrite(ENA, 255);    // test at full speed first
  delay(2000);

  digitalWrite(IN1, LOW);   // IN1 == IN2 -> stop (brake)
  digitalWrite(IN2, LOW);
  delay(1000);

  digitalWrite(IN1, LOW);   // reverse
  digitalWrite(IN2, HIGH);
  analogWrite(ENA, 180);    // lower it to find your motor's minimum
  delay(2000);
}
```

---

### 3. Test Code for ESP32 (`arduino-esp32` Core 3.x)
On core 3.x, `analogWrite()` works directly, or you can use the built-in LEDC functions:

```cpp
const int ENA = 25;  // PWM-capable pin
const int IN1 = 26;
const int IN2 = 27;

void setup() {
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  
  // Core 3.x LEDC setup for ENA
  ledcAttach(ENA, 1000, 8);   // pin, frequency (Hz), resolution (bits)
}

void loop() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  ledcWrite(ENA, 255);        // full speed forward
  delay(2000);

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);     // stop
  delay(1000);

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  ledcWrite(ENA, 180);        // reverse
  delay(2000);
}
```

---

### 4. Quick Wiring Reference (ESP32)
* **Motor Power + ($7\text{ V} - 12\text{ V}$):** Connect to L298N `12V` terminal.
* **Motor Power Ground ($-$) & Microcontroller GND:** Connect together to L298N `GND` terminal (**Shared Ground**).
* **Motors:** Connect to `OUT1` and `OUT2`.
* **Control Pins (Jumper removed from ENA):** `ENA` $\rightarrow$ `GPIO 25`, `IN1` $\rightarrow$ `GPIO 26`, `IN2` $\rightarrow$ `GPIO 27`.
