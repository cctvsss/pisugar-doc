---
sidebar_position: 9
---

# Custom Battery Capacity

If you find that the current capacity of the PiSugar battery doesn't meet your requirements, you can replace it with a larger single lithium battery on your own.

:::danger

**DO NOT attempt to connect multiple batteries in parallel.** The PiSugar charging circuit is NOT designed for parallel battery configurations and doing so can be dangerous.

:::

:::warning

Modifications to the circuit may void the product warranty.

:::

## Battery Specifications

When selecting a replacement battery, please ensure it meets the following criteria:

- **Type**: Single-cell lithium battery (3.7V)
- **Connector**: JST PH 2.0mm connector (plus version) or direct soldering to the board (zero version)
- **Polarity**: Please check the polarity markings on the PiSugar board

## Custom Battery Curve (For PiSugar 2/3)

If you are using a battery with a different capacity than the original, you may need to adjust the battery curve in the PiSugar software.

To measure the battery curve, you can use the following steps:

1. Fully charge the battery using the PiSugar charging circuit.
2. Connect to the Raspberry Pi and run our provided battery logging [script](https://github.com/PiSugar/pisugar-power-manager-rs/blob/master/scripts/record-level.sh) under light system load conditions.
3. Let it discharge continuously until the battery is depleted and the system shuts down.
4. Extract data points from the recorded time and voltage measurements.
5. Calculate the estimated battery percentage corresponding to each voltage level.
6. Write these values into the JSON configuration `/etc/pisugar-server/config.json`. (add the `battery_curve` key if it doesn't exist)

```json
{
  "battery_curve": [
    [4.2, 100] 
    [4.0, 90], 
    [3.5, 60],
    [3.3, 10],
  ]
}
```

Note: The above values are examples. Your actual battery curve should be based on measurements from your specific battery.

# PiSugar Battery Curve Calibration Guide

> **Why is this important?**  
> The battery curve determines how the PiSugar service calculates battery percentage. Calibrating it ensures accurate battery readings.

---

## 1. Download the Script and Save Locally

Create a directory and download the official script:

```bash
mkdir -p ~/scripts
wget -q https://raw.githubusercontent.com/PiSugar/pisugar-power-manager-rs/master/scripts/pisugar_battery_curve.py -O ~/scripts/pisugar_battery_curve.py
```

Check if the file exists:

```bash
ls -l ~/scripts/pisugar_battery_curve.py
```

---

## 2. Run the Script Using `screen`

The full charge/discharge process takes **about 6 hours**.  
Running it directly via SSH is risky because if the session disconnects, the process will stop.  
We use `screen` to keep the session alive.

Start a screen session:

```bash
screen -S battery_curve
```

Run the script:

```bash
python3 ~/scripts/pisugar_battery_curve.py
```

Detach from the screen (keep it running in the background):

**Press**:  
```
Ctrl + A  then D
```

To reattach later and check progress:

```bash
screen -r battery_curve
```

---

## 3. Wait for Completion and Check the Result

Once the script completes, it will generate a file named:

```
battery_curve.json
```

in the same directory.

Check the file:

```bash
cat ~/scripts/battery_curve.json
```

Example content:

```json
"battery_curve": [
  [4.19, 100],
  [4.07, 94],
  [4.02, 87],
  [3.93, 81],
  [3.86, 75],
  [3.80, 69],
  [3.74, 62],
  [3.67, 56],
  [3.63, 50],
  [3.59, 44],
  [3.57, 38],
  [3.53, 31],
  [3.52, 25],
  [3.48, 19],
  [3.43, 13],
  [3.39, 6],
  [3.10, 0]
]
```

---

## 4. Add the Curve to PiSugar Config

Edit the configuration file:

```bash
sudo nano /etc/pisugar-server/config.json
```

Find or add `battery_curve` (if it exists, **replace it**):

```json
"battery_curve": [
  [4.19, 100],
  [4.07, 94],
  [4.02, 87],
  [3.93, 81],
  [3.86, 75],
  [3.80, 69],
  [3.74, 62],
  [3.67, 56],
  [3.63, 50],
  [3.59, 44],
  [3.57, 38],
  [3.53, 31],
  [3.52, 25],
  [3.48, 19],
  [3.43, 13],
  [3.39, 6],
  [3.10, 0]
]
```

Save and exit:

```
Ctrl + O  (Save)
Ctrl + X  (Exit)
```

---

## 5. Restart PiSugar Service

```bash
sudo systemctl restart pisugar-server
```

After restart, PiSugar will calculate the battery percentage using the new curve.

---

## Important Notes

> **Tip:** Keep your Pi powered during calibration.  
> **Warning:** Do not disconnect SSH while the script is running.  
> **Safety:** Improper handling of lithium batteries can pose safety risks. Always follow proper safety procedures.

- Do **not disconnect power** during the calibration process.  
- The script automatically **restores charging** at the end (even if an error occurs, thanks to `finally` block).  
- If SSH disconnects, use:
  ```bash
  screen -r battery_curve
  ```
- After updating the config, **always restart the PiSugar service**.
- Using batteries larger than the recommended size may affect the physical fit in your project.
- The PiSugar's charging circuit is designed for specific battery capacities; extremely large batteries may take longer to charge.
- Always use quality batteries from reputable manufacturers.
- Never use damaged or swollen batteries.


