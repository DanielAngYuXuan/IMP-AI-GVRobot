# Gesture and Voice Controlled Robot

A 4WD robot built on a Raspberry Pi 5. It is controlled by spoken commands and by body gestures seen by a Raspberry Pi AI Camera. The planned final feature is a follow-me mode.

## Hardware

- Raspberry Pi 5
- Raspberry Pi AI Camera (Sony IMX500)
- USB condenser microphone
- Bluetooth speaker (a USB speaker also works)
- 4WD chassis with 4 DC motors
- 2 x L298N motor drivers
- 12V adapter for the motors
- Breadboard
- 3D printed enclosure for the Pi and the two drivers

Not used: HC-SR04 ultrasonic sensors and the M.2 HAT+.

## What it does

| Input | How it works |
|---|---|
| Voice | USB microphone, speech recognition, matched against a list of commands |
| Gestures | The AI Camera runs a pose model (HigherHRNet) on its own chip and returns body keypoints. Simple rules turn the keypoints into gestures |

Voice and gestures both call one function, `robot_do()`, which drives the motors.

### Voice commands

| Say | Action |
|---|---|
| move front | Forward |
| move back | Backward |
| turn left | Turn left |
| turn right | Turn right |
| stop | Stop |
| status, hello | Spoken reply only |
| shut down | Quit the program |

Speech recognition uses Google's free service, so the Pi needs internet.

### Gestures

Left and right mean your own arms.

| Gesture | Action |
|---|---|
| Both arms out to the sides | Forward |
| Left arm out to the side | Turn left |
| Right arm out to the side | Turn right |
| Left hand above shoulder | Turn left |
| Right hand above shoulder | Turn right |
| Both hands above shoulders | Stop |

Forward and backward moves run for 1 second and turns for 0.4 seconds, then stop by themselves.

## Files

| File | Purpose |
|---|---|
| robot_main.py | Main script: camera gestures, voice commands and motor control |
| motor_test.py | Spins each wheel on its own, then all four |
| motor_test_rear.py | Low speed test of the two rear wheels |
| generate_enclosure.py | Generates the enclosure STL files |
| enclosure_base.stl | Enclosure base |
| enclosure_lid.stl | Enclosure lid |

## Software setup

Run these in a terminal on the Pi.

```
sudo apt update && sudo apt full-upgrade
sudo apt install imx500-all python3-pyaudio espeak-ng flac
pip install SpeechRecognition --break-system-packages
sudo reboot
```

Download the example files for the AI Camera.

```
cd ~
git clone https://github.com/raspberrypi/picamera2-examples
```

Put `robot_main.py` in `~/picamera2-examples/examples/imx500/` so it can find the labels file in the `assets` folder.

Find the GPIO chip number.

```
gpiodetect
```

Use the number shown next to `pinctrl-rp1`. In this build it is 15, set as `GPIO_CHIP` in `robot_main.py`.

## Wiring

### Pi pins to the L298N inputs

Driver 1 runs the left side and driver 2 runs the right side. Each wheel has its own channel.

| Pi pin | GPIO | Goes to | Wheel |
|---|---|---|---|
| 29 | GPIO5 | Driver 1 IN1 | Front left |
| 31 | GPIO6 | Driver 1 IN2 | Front left |
| 33 | GPIO13 | Driver 1 IN3 | Rear left |
| 35 | GPIO19 | Driver 1 IN4 | Rear left |
| 38 | GPIO20 | Driver 2 IN1 | Front right |
| 40 | GPIO21 | Driver 2 IN2 | Front right |
| 36 | GPIO16 | Driver 2 IN3 | Rear right |
| 37 | GPIO26 | Driver 2 IN4 | Rear right |
| 6 | GND | Ground junction | |

### Motors and power

- Driver 1: front-left motor on OUT1/OUT2, rear-left motor on OUT3/OUT4.
- Driver 2: front-right motor on OUT1/OUT2, rear-right motor on OUT3/OUT4.
- The 12V adapter positive goes to the 12V terminal on both drivers.
- The adapter negative goes to a ground junction, then to the GND terminal on both drivers.
- Pi pin 6 joins the same ground through the breadboard ground rail.
- Leave the ENA and ENB jumpers on.
- Do not connect the 5V terminal of either driver to the Pi.
- The Pi has its own USB-C power supply.
- The camera uses its ribbon cable. The microphone is USB and the speaker is Bluetooth.

## Running

Open `robot_main.py` in Thonny on the Pi desktop and press Run. The first start takes a few minutes while the camera loads its model.

To test without motors, set `USE_MOTORS = False` at the top of the script.

Settings at the top of the script include `SPEED`, `MOVE_SECONDS`, `TURN_SECONDS`, `GPIO_CHIP` and the gesture thresholds.

## Enclosure

- Inside size: 124 x 127 x 34 mm. Outside size: 128.8 x 131.8 x 40 mm, plus mounting ears.
- Standoffs with 2.5 mm holes for the Pi and both L298N boards. Four corner posts for the lid screws.
- Slots for the Pi ports, the camera ribbon and the motor wires. Vent slots in the lid.
- Measure the hole spacing of your L298N and edit `L298_HOLE_INSET` in `generate_enclosure.py` if it is not 37 mm.

## Troubleshooting

| Problem | Fix |
|---|---|
| Camera window closes after about 5 seconds | Use `rpicam-hello -t 0` |
| IMX500 camera dev-node not found | Shut down, check the ribbon cable, boot with the camera plugged in |
| gpiozero cannot open the GPIO chip | The script uses lgpio directly with `GPIO_CHIP` |
| ALSA and JACK warnings in Thonny | Harmless messages from the audio library |
| Speech library asks for flac | `sudo apt install flac` |
| Voice commands not heard | Check the microphone with `arecord -l` and that no old script is still running |
| Left or right turns are swapped | Swap that side's motor wires on the driver |

## Limitations

- There is no obstacle detection. Test with the wheels lifted, then at low speed in a clear area.
- Speech recognition needs internet.
- Gestures need your upper body in view, about 1.5 to 3 metres from the camera.

## Next step

Follow-me mode: steer from where the person is in the camera frame and use their size in the frame as a rough distance.
