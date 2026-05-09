# curtain-1 – ESPHome Curtain Controller

An ESPHome configuration for a motorised curtain controller based on the ESP32-C3 and a 28BYJ-48 stepper motor driven by a ULN2003 board. Exposes a native Home Assistant `cover` entity with position control, homing, and max position calibration.

---

## Hardware

| Component | Details |
|---|---|
| Microcontroller | ESP32-C3 DevKitM-1 |
| Motor driver | ULN2003 |
| Stepper motor | 28BYJ-48 |
| Limit switch | Normally Open (NO) |

### Pinout

| GPIO | Function |
|---|---|
| GPIO0 | ULN2003 IN1 (pin_a) |
| GPIO1 | ULN2003 IN2 (pin_b) |
| GPIO2 | ULN2003 IN3 (pin_c) |
| GPIO3 | ULN2003 IN4 (pin_d) |
| GPIO4 | Limit switch (INPUT_PULLDOWN, NO) |

> **Note:** GPIO0–3 are strapping pins on the ESP32-C3. They work fine for stepper use once booted but you may see warnings during compilation. These can be safely ignored.

---

## How it works

### Home position

The limit switch on GPIO4 defines position 0 — the fully open position. When the switch closes, the motor stops immediately, the position counter is zeroed, and Home Assistant is updated.

### Travel length

The closed position is defined by **Curtain Travel (steps)** — the number of motor steps from home (open) to fully closed. This is set either manually in HA or automatically using the **Set Max Position** buttons.

### Motor side

Because the device can be mounted on either side of a window, **Motor Side** (Left/Right) reverses the direction of all motor movement so that open/close always behave correctly regardless of physical orientation.

---

## Home Assistant entities

### Cover
- **Curtain** — standard HA cover entity with position slider (0–100%). Supports open, close, stop, and set position.

### Buttons
| Button | Function |
|---|---|
| Run Homing Sequence | Drives motor to the limit switch and zeros the position |
| Jog Open 1000 steps | Moves 1000 steps in the open direction |
| Jog Close 1000 steps | Moves 1000 steps in the close direction |
| Set Max Position – Start | Drives motor toward the closed end until stopped |
| Set Max Position – Mark Here | Stops the motor and saves current position as the travel length |

### Select
| Entity | Options |
|---|---|
| Motor Side | Left / Right — reverses motor direction |

### Number
| Entity | Range | Description |
|---|---|---|
| Curtain Travel (steps) | 1000–300000 | Total steps from open to closed |

### Binary sensors
| Entity | Description |
|---|---|
| Curtain Homed | True when a successful homing has been performed |
| Curtain Home Switch | Raw state of the limit switch |

---

## Initial setup

1. Flash the firmware via ESPHome.
2. Set **Motor Side** to match the physical position of the device (Left or Right).
3. Press **Jog Open / Close** to verify the motor moves in the correct direction. If open and close are reversed, switch Motor Side.
4. Press **Run Homing Sequence**. The motor will back off the switch if already triggered, then seek it slowly. Once found, position is zeroed.
5. Press **Set Max Position – Start**. The motor drives toward the closed end.
6. When the curtain is fully closed, press **Set Max Position – Mark Here**. The step count is saved as the travel length.
7. The curtain is now calibrated. Use the cover entity in HA normally.

> Re-home after any power cycle if position accuracy is important, as the stepper has no encoder and position is tracked in software only.

---

## Substitutions

Tunable at the top of the YAML without editing the rest of the config:

| Key | Default | Description |
|---|---|---|
| `name` | `curtain-1` | ESPHome device name |
| `sweep_speed` | `600` | Motor speed in steps/s |
| `default_end_position` | `30000` | Initial travel length in steps |

---

## Secrets

The following entries are required in your `secrets.yaml`:

```yaml
curtain-1-api-encryption-key: "your-key-here"
curtain-1-ota-password: "your-password-here"
wifi_ssid: "your-ssid"
wifi_password: "your-wifi-password"
```

---

## Notes

- The 28BYJ-48 is a low-torque motor. Keep `sweep_speed` at or below 600 steps/s for reliable operation. Lower values give more torque.
- Step mode is set to `HALF_STEP` in the config for smoother movement and better positional resolution.
- Position is published to HA every 3 seconds while the motor is moving, and once more when it stops.
- All settings (travel length, motor side) survive reboots via ESPHome's flash storage (`restore_value: true`).
