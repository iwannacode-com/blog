# I Wanna Code Particle Photon: Shouting rainbow.

![Photon shouting rainbow](https://raw.githubusercontent.com/iwannacode-com/blog/master/post0001/pic.jpg)

In this post we're going to learn how to cycle through the rainbow colors
using the RGB LED embedded on the [Particle
Photon](https://store.particle.io/?product=particle-photon). This LED is used
by the Photon to indicate its current
[state](https://docs.particle.io/guide/getting-started/modes/photon/) and
there's no public method on the API to set its color via coding. However, the
`particle` command line tool has one option called `nyan` that makes your
Photon cycle through the rainbow colors. If you want to try it, execute the
following command:

`particle cloud nyan`

This indicates that although there's no public method for it, we can still set
the LED color via coding. By checking the `system_cloud_internal.cpp` file from
the firmware code, we can find the method `LED_Signaling_Override`, which is
called when upon the execution of the nyan command.

The following code shows how we can use it on our own Photon applications. The
application reads a button state. When the button is pressed, it starts the
rainbow sequence. You can download the code from our Github
[repository](https://github.com/iwannacode-com/blog/tree/master/post0001).


```
int buttonPin = D2;
int buttonState;

unsigned long ledSignalingCounter;
unsigned long now;

boolean nyan;

// Values for setting the rainbow colors on the RGB LED.
// VIBGYOR = Violet Indigo Blue Green Yellow Orange Red
const uint32_t VIBGYOR_Colors[] = {
  0xEE82EE, 0x4B0082, 0x0000FF, 0x00FF00, 0xFFFF00, 0xFFA500, 0xFF0000
};
const int VIBGYOR_Size = sizeof (VIBGYOR_Colors) / sizeof (uint32_t);
int VIBGYOR_Index;

void setup() {
  pinMode(buttonPin, INPUT);
}

// Copied from system/src/system_cloud_internal.cpp (photon firmware code)
// The file can be seen at:
// https://github.com/spark/firmware/blob/d2b55ee9a151c4cbc63ede84ec2a89bbedf3c69e/system/src/system_cloud_internal.cpp
/* This function MUST NOT BlOCK!
 * It will be executed every 1ms if LED_Signaling_Start() is called
 * and stopped as soon as LED_Signaling_Stop() is called */
void LED_Signaling_Override(void) {
  LED_SetSignalingColor(VIBGYOR_Colors[VIBGYOR_Index]);
  LED_On(LED_RGB);
  ++VIBGYOR_Index;
  if (VIBGYOR_Index >= VIBGYOR_Size) {
    VIBGYOR_Index = 0;
  }
}

void setNyanMode(int _nyan) {
  // Return if nothing has changed.
  if(nyan == _nyan) {
    return;
  }

  nyan = _nyan;

  if(nyan == true) {
    // Allows the user to manually set the RGB LED color.
    LED_Signaling_Start();
    ledSignalingCounter = now;
  } else {
    // Returns to the default REB LED state.
    LED_Signaling_Stop();
  }
}

void loop() {
  now = millis();

  buttonState = digitalRead(buttonPin);

  // If the button is pressed, start shouting rainbow.
  if(buttonState == HIGH) {
    setNyanMode(true);
  } else {
    setNyanMode(false);
  }

  if(nyan) {
    // If shouting rainbow, update the RGB LED color every 100ms.
    if(now - ledSignalingCounter >= 100) {
      LED_Signaling_Override();
      ledSignalingCounter = now;
    }
  }
}
```
