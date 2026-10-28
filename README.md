# Remote Switch --- Wireless Video Trigger System

**Author:** Harshvardhan Gupta

## Overview

The Remote Switch is a custom-built, battery-powered wireless control
system developed for the inauguration ceremony held during the
university convocation of the graduating batch of 2025.

The system remotely triggers one of four predefined video sequences
associated with the inauguration of two laboratories and two Netaji
Subhas Chandra Bose statues at the university's East and West campuses.
An Arduino-based hardware unit sends commands over Bluetooth to a Python
application on a laptop, which starts the corresponding video.

The system was designed with a focus on deliberate activation, clear
operator feedback, reliable operation, and protection against
low-battery conditions. It was tested and rehearsed before being used at
the live event.

## Key Features

-   **Wireless control:** Arduino communicates with a laptop over an
    HC-05 Bluetooth module.
-   **Four selectable video sequences:** A large pushbutton cycles
    through four options.
-   **Deliberate activation:** An emergency-stop-style switch is used to
    control triggering; the system requires re-arming before another
    trigger can be accepted.
-   **Visual status indicators:** LEDs indicate power, selected video,
    switch status, and low-battery condition.
-   **Low-battery protection:** Battery voltage is monitored through an
    ADC voltage divider. At or below 6 V, the firmware disables video
    outputs and indicates low battery.
-   **Timer-based status blinking:** Timer1 in CTC mode and an interrupt
    service routine control the armed-state LED blinking.
-   **Enclosed hardware:** Components are mounted on zero PCB inside an
    IP65-rated plastic enclosure.

## System Architecture

``` text
[9 V Battery]
      |
      v
[Arduino + Switches + LEDs] <---- Bluetooth ----> [HC-05 Module]
                                                     |
                                                     v
                                           [Laptop / Python App]
                                                     |
                                                     v
                                           [Selected Video Playback]
```

The Arduino reads the controls, updates the selected-video indicators,
monitors battery voltage, and sends a serial value (`1`--`4`) over
Bluetooth when triggered. The Python application maps that value to the
corresponding video file and starts playback.

## Hardware Components

  -----------------------------------------------------------------------
  Component                           Purpose
  ----------------------------------- -----------------------------------
  Arduino Nano Every (as shown in the Main controller
  schematic)                          

  HC-05 Bluetooth module              Wireless serial communication

  Emergency stop switch               Deliberate trigger and re-arming
                                      control

  Large pushbutton                    Cycles through the four video
                                      options

  SPDT power switch                   Power control

  Indicator LEDs                      Power, selected video, switch
                                      status, and low-battery indication

  220 Ω resistors                     LED current limiting

  1 kΩ and 2 kΩ resistors             Voltage divider for the HC-05 RX
                                      input

  1 MΩ and 120 kΩ resistors           Battery-voltage sensing divider

  10 kΩ resistors                     Pull-up circuitry

  9 V alkaline battery and connector  Power supply

  Zero PCB and insulated wires        Circuit assembly and wiring

  90-degree female Berg strip         HC-05 mounting

  IP65-rated plastic enclosure        Component protection

  Hot glue, two-part epoxy, and       Mechanical mounting and finishing
  adhesive                            

  Custom 3D-printed decals            Control and status labeling
  -----------------------------------------------------------------------

Refer to the project report for the complete circuit schematic and
physical build photographs.

## Firmware and Software

### Arduino firmware

The Arduino firmware is responsible for:

-   Reading the video-selection pushbutton and emergency switch.
-   Cycling through four video selections.
-   Driving the corresponding video-selection LEDs/outputs.
-   Sending the selected video number over Bluetooth serial at **9600
    baud** when triggered.
-   Measuring battery voltage using the internal 1.12 V ADC reference
    and a resistive divider.
-   Disabling the video outputs when the measured battery voltage
    reaches the 6 V low-battery threshold.
-   Controlling the emergency-switch status LED using Timer1 in CTC
    mode.

The firmware uses the Arduino `SoftwareSerial`, AVR I/O, and AVR
interrupt libraries.

### Python application

The laptop application listens for Bluetooth serial data and maps the
received values to video files.

    Serial value Video sequence
  -------------- ------------------------
             `1` West Campus --- Part 1
             `2` East Campus --- Part 1
             `3` West Campus --- Part 2
             `4` East Campus --- Part 2

The original implementation uses Python's `pyserial` library for serial
communication and `os.startfile()` to launch videos on Windows.

## Requirements

### Hardware

-   Arduino Nano Every or the exact compatible board used by the
    firmware
-   HC-05 Bluetooth module
-   Switches, LEDs, resistors, battery, PCB, and enclosure as listed
    above

### Software

-   Arduino IDE or a compatible Arduino build environment
-   Python 3
-   `pyserial`
-   Windows, because the supplied Python implementation uses
    `os.startfile()`
-   Four local video files

Install the Python dependency with:

``` bash
python -m pip install pyserial
```

## Setup

1.  **Assemble the hardware.** Use the circuit schematic in the project
    report to wire the controller, switches, LEDs, battery-sensing
    divider, and HC-05 module.
2.  **Upload the firmware.** Open the Arduino firmware in the Arduino
    IDE, select the correct board and port, and upload it.
3.  **Pair Bluetooth.** Pair the laptop with the HC-05 module using the
    appropriate Bluetooth settings for the module.
4.  **Identify the serial port.** Find the COM port assigned to the
    Bluetooth serial connection in Windows Device Manager.
5.  **Configure the Python script.** Set `COM_PORT` to the assigned port
    and set the four video path variables to the actual locations of
    your video files.
6.  **Install dependencies.** Run `python -m pip install pyserial`.
7.  **Run the application.** Start the Python script and verify that it
    reports incoming serial values and launches the intended videos.
8.  **Test before use.** Verify switch behavior, video selection, LED
    indications, Bluetooth range, battery protection, and video mapping
    before relying on the system.

### Configuration example

Update these values in the Python script to match your computer:

``` python
COM_PORT = "COM13"
BAUD_RATE = 9600

VIDEO_A = r"C:\path\to\West_Campus_Part_1.mp4"
VIDEO_B = r"C:\path\to\East_Campus_Part_1.mp4"
VIDEO_C = r"C:\path\to\West_Campus_Part_2.mp4"
VIDEO_D = r"C:\path\to\East_Campus_Part_2.mp4"
```

`COM13` is the port used in the original report; your computer may
assign a different port. Replace the example paths with the real paths
to your video files.

## Operating Logic

1.  Power on the unit.
2.  Check the power and status indicators.
3.  Use the large pushbutton to cycle through the four video options.
    The corresponding indicator shows the selected option.
4.  Use the emergency switch according to the intended arming and
    trigger procedure.
5.  When a valid trigger is received, the Arduino transmits the selected
    option as a number from `1` to `4`.
6.  The laptop application receives the number and starts the associated
    video.
7.  If the battery voltage falls to the configured low-battery
    threshold, the firmware turns off the video outputs and indicates
    the low-battery state.

**Important:** Test the full arming, triggering, and re-arming sequence
on the assembled hardware before any live use.

## Testing and Deployment

According to the project report, validation included:

-   Wireless range testing
-   Endurance testing
-   Electrical stress testing
-   Multiple rehearsals before the event

The system was used during the live inauguration ceremony and, as
documented in the report, performed reliably during the event. The
report also notes transient current spikes of approximately 200 mA
during Bluetooth transmission.

## Repository Structure

A suggested repository layout is:

``` text
remote-switch/
├── README.md
├── firmware/
│   └── remote_switch.ino
├── software/
│   ├── remote_switch.py
│   └── videos/                 # Optional; do not commit large/private videos
├── hardware/
│   ├── schematic.pdf
│   └── photos/
└── docs/
    └── Remote_Switch_Report.pdf
```

Adjust the filenames and folders to match the files actually included in
your repository.

## Safety and Limitations

-   Verify all wiring, voltage-divider values, and polarity before
    powering the circuit.
-   The HC-05 RX input uses a resistor divider in the documented design;
    preserve appropriate logic-level interfacing.
-   Confirm the battery-voltage measurement against a multimeter during
    testing.
-   The low-battery cutoff and switch behavior should be tested on the
    actual hardware.
-   The supplied Python approach uses Windows-specific `os.startfile()`.
-   The Python script must be configured with the correct serial port
    and local video paths.
-   Do not commit confidential event media or other large video files
    unless you have permission and a suitable distribution plan.

## Acknowledgements

Developed by **Harshvardhan Gupta** for the university convocation
inauguration of the graduating batch of 2025.

## License

No license is specified in the project report. Add a `LICENSE` file if
you intend to make the hardware design and source code reusable by
others. Until a license is added, others should not assume they have
permission to redistribute or modify the project.
