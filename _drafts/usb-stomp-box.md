---
layout: post
title: "USB Stomp Box"
description: "Home made USB device for controlling PC guitar software"
category: Arduino
tags: ['arduino', 'teensy', 'usb', 'keyboard', 'mouse', 'making']
---

Christmas is right around the corner and choosing presents has been a nightmare this year. For my Dad I decided to make something. He's been using some Amp modelling software on the PC and one awkward bit is triggering the record function and then getting ready to play. It would be handy if he had a "guitar pedal" that could control the software.

Enter the [teensy](https://shop.pimoroni.com/products/teensy-lc). It's small and cheap and has super easy USB Keyboard emulation built in.

As well as the teensy I needed:

 * [A sturdy looking stomp box switch](http://www.ebay.co.uk/itm/DPDT-Momentary-On-On-Foot-Switch-Guitar-Effects-Pedal-Metal-Stomp-Box/262241729461)
 * [A sturdy case](http://www.ebay.co.uk/itm/1591ATCL-Hammond-Clear-Polycarbonate-Enclosure-Box-100-x-50-x-25mm/262321566055)

I picked up both from [Switch Electronics](http://stores.ebay.co.uk/ledessential/).

In terms of code this was fairly easy:

```
const int BUTTON_PIN = 23;
const int LED_PIN = 13;
const unsigned long DEBOUNCE_DELAY = 50;

int button_state;
int last_button_state;
unsigned long last_debounce_time = 0;

void setup() {
  pinMode(13, OUTPUT);
  pinMode(23, INPUT_PULLUP);

  for (int i=0; i<5; i++) {
    digitalWrite(LED_PIN, LOW);
    delay(80);
    digitalWrite(LED_PIN, HIGH);
    delay(80);
  }

  button_state = last_button_state = digitalRead(BUTTON_PIN);
}

void loop() {
 int reading = digitalRead(BUTTON_PIN);

  if (reading != last_button_state) {
    last_debounce_time = millis();
  }

  if ((millis() - last_debounce_time) > DEBOUNCE_DELAY && reading != button_state) {
    button_state = reading;

    if (button_state == HIGH) {
      Keyboard.println("MERRY XMAS FROM JOHN");
    }
  }

  digitalWrite(LED_PIN, !button_state);

  last_button_state = reading;
}
```

The LED flashes when you plug it in so you know its working. Some debouncing and state tracking code let me run the all important `Keyboard.println` when the button is pressed and then released. While the switch is held down the LED is on, again just for debugging.

I don't have access to the software to figure out the keyboard shortcuts so for now it prints a festive message - we can update the code when he tries it out.

If the software doesn't actually have any keyboard shortcuts at all (and you can't even press enter on a highlight widget) then I have a backup plan. He can definitely start the app with a mouse click, and the Teensy supports that too:

```
// simple mouse clock
Mouse.click()

// LEFT off, MIDDLE off, RIGHT on
Mouse.set_buttons(0, 0, 1);
```

If I had more time i'd look at the Bounce library and remove the state tracking code.
