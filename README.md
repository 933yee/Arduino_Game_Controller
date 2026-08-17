# Arduino Game Controller

A custom Arduino controller paired with a Unity game collection. An analog joystick, push buttons, a potentiometer, and an MPU6050 provide conventional controls and motion input; Unity reads the controller over a serial connection and maps it to several playable demos.

## Demo

### Menu

<img width="478" alt="Game selection menu" src="https://user-images.githubusercontent.com/92087014/212650674-b859029c-d3bf-4795-9322-0b28ad4d0d34.png">

### Gameplay

https://user-images.githubusercontent.com/92087014/212684518-9af759e0-4499-4d7a-a28d-84974c96564d.mp4

https://user-images.githubusercontent.com/92087014/212684528-163196d9-e0bd-435b-88cb-0c84869b57d8.mp4

https://user-images.githubusercontent.com/92087014/212684539-4fbbb68f-566e-4e53-a339-88a047a51bfd.mp4

## Features

- A Unity menu that launches four controller-driven experiences: Tetris, an obstacle runner, a racing game, and a 3D character scene.
- Joystick input for movement and camera control.
- MPU6050 accelerometer/gyroscope input, filtered with a Kalman filter, for tilt steering and orientation.
- Dedicated buttons for selection, exit, rotation, hard drop, jumping, and zooming.
- A potentiometer that controls in-game audio volume.
- A text-based serial protocol that lets each Unity scene select the input mode it needs.

## System overview

```text
joystick / buttons / potentiometer / MPU6050
                     |
                  Arduino
                     |
             USB serial @ 115200
                     |
                  Unity game
```

The Arduino sketch accepts a one-character mode command from Unity and returns one line per input event:

| Mode | Intended scene | Example messages |
| --- | --- | --- |
| `M` | Menu/Tetris-style controls | `Movement: x y`, `Clockwise: 1`, `Space: 1`, `Volume: value` |
| `R` | Racing | `Angle:x y z`, `Exit:1` |
| `H` | Character navigation | `Movement: x y`, `CameraMovement: x y`, `ZoomIn: 1` |
| `C` | Combined movement and tilt | movement/camera messages plus `Angle:x y z` |

Unity reads these messages on a background thread and applies them to rigid bodies, cameras, UI, and scene transitions.

## Repository structure

```text
gameController/gameController.ino   Arduino firmware and serial protocol
Assets/menu/                        Game-selection UI and scene scripts
Assets/tetris/                      Tetris implementation and controller mapping
Assets/cubeRun/                     Obstacle-runner gameplay
Assets/car/                         Tilt-controlled racing demo
Assets/3Dhuman/                     Character and camera demo
Assets/SerialComm/                  Unity serial communication helpers
Assets/PathCreator/                 Path tooling used by the racing scene
```

## Hardware

The sketch is written around the following parts and pin assignments:

| Component | Arduino connection |
| --- | --- |
| MPU6050 | I2C (`Wire`) |
| Joystick X/Y | `A0` / `A1` |
| Potentiometer | `A2` |
| Action buttons | digital pins `9`–`13` |
| Additional button | digital pin `7` with pull-up |

The firmware depends on `I2Cdev`, `MPU6050`, and `Kalman` Arduino libraries.

## Setup

1. Wire the controller according to the pin table and install the required Arduino libraries.
2. Open `gameController/gameController.ino`, select the correct board and serial port, then upload it.
3. Open the Unity project containing this `Assets` directory.
4. In each serial-enabled component, change the Inspector's `port` field from the original `COM4` setting to the controller's current port.
5. Keep the baud rate at `115200` to match the firmware, then open the menu scene and press Play.

## Notes

- This repository is an educational project snapshot and contains the Unity `Assets` tree rather than a fully self-contained Unity project with `ProjectSettings` and `Packages`.
- Several art packages and the Path Creator/SerialComm helpers are third-party assets. Their original licensing terms still apply.
- Serial port names are machine-specific, and only one scene/component should own the port at a time.

