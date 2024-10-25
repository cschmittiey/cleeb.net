+++
title = 'Saintcon 2024 Ham Radio Minibadge Flashing Instructions'
date = 2024-10-25T11:20:30-06:00
draft = false
+++

At Saintcon 2024 we had a ham radio badge that needs to be flashed with custom firmware. Here's how to do it! 

Huge thanks to Ray-Man and redactd for their help in creating this awesome badge for our community!

You'll need some kind of way to do UPDI programming. We used an [Adafruit UPDI Friend](https://www.adafruit.com/product/5879) set to 3V. Unless you're using a breadboard to flash the badge, you will also likely want [this cable.](https://www.adafruit.com/product/5765)

## Using the Arduino IDE

1. Download the Arduino IDE
2. Add the megaTinyCore library to the Arduino IDE. Detailed instructions [here](https://github.com/SpenceKonde/megaTinyCore/blob/master/Installation.md) but the short version is:
    - Open the Arduino IDE
    - Go to File -> Preferences
    - In the "Additional Boards Manager URLs" field, add `http://drazzy.com/package_drazzy.com_index.json`
    - Go to Tools -> Board -> Boards Manager
    - Search for "megaTinyCore" and install it
![a screenshot of the arduino ide when adding the megatinycore library](/images/posts/saintcon2024/1.png)
3. Open the firmware.ino file, and replace the text that says "YOUR CALL SIGN HERE" with your call sign. For example, mine would read `const char* message = "KZ0P";`
4. Go to Tools -> Board and select the option that includes the "ATtiny202" that isn't the Optiboot option.
5. Then, set Tools -> Programmer to Serial UPDI
6. Connect your UPDI programmer to the badge. The UPDI pin can easily be located on the badge by looking for the trace with the jumper labeled J1 coming from the microcontroller. Ground will be the inner top right pin when looking at the back of the badge, and power will be the inner top left pin.
7. Compile and Upload the firmware to the badge. If you get an error, send me an email or discord message and I'll do my best to help but I am by no means an engineer.

## Using Platformio

While trying to do this myself at the conference, I ran into an issue where the extra repo for megatinycore had expired SSL certificates, and so I used platformio instead to flash the badge. You can find my platformio conversion of the project here. To flash the board, you'll need to install platformio core and the VS Code extension and run the following two commands to flash the badge after editing the .ino file to include your call sign:

```bash
source ~/.platformio/penv/bin/activate
pio run -e Upload_UPDI -t upload
```

I don't know how you'd run these commands on windows, so if you know what you're doing with windows and platformio reach out and let me know!