# PX4 Autonomous Takeoff-Hover-Land Mission in GPS Denied Environment

This project customizes the PX4 `commander` module to include a new command called `mission_start`, enabling fully autonomous drone behavior in GPS Denied Environments: **arm → takeoff → hover → monitor → land**.

---

## Important Notes

- This firmware is designed for **Gazebo-Classic (ROS1)** and **Gazebo-Harmonic (ROS2)** simulations.
- **Ignore inbuilt PX4 failsafe messages** that may appear in the PX4 console during simulation.
- Clone the repo using:

  ```bash
  git clone https://github.com/avianbob/PX4-Autonomous-Mission.git
  ```

---

## What This Command Does

When you run:

```bash
commander mission_start
```

The drone performs the following:

1. **Arms the drone**
2. **Switches to AUTO mode**
3. **Takes off to a predefined altitude (default 5m)**
4. **Hovers in place for a set duration (default 40s)**
5. **Logs sensor and flight data every few seconds (default 3s)**
6. **Monitors for battery low or RC signal loss**
7. **Lands and disarms when mission ends or safety triggers occur**

---

## Code Location

1. The logic can be found at:

```
Codes/Commander.cpp
```
2. The logic is to be implemented at:

```
PX4-Autopilot/src/modules/commander/Commander.cpp
```
---

## Mission Flow Breakdown

### 1. Takeoff Preparation

Sets parameters for:
- `MIS_TAKEOFF_ALT` = 5.0 meters
- `hover_time` = 40.0 seconds
- `print_interval` = 3.0 seconds

### 2. Arm and Wait

Sends arming command and blocks until drone is armed:
```cpp
send_vehicle_command(VEHICLE_CMD_COMPONENT_ARM_DISARM, ARMING_ACTION_ARM);
```

### 3. Switch to AUTO Mode

```cpp
send_vehicle_command(VEHICLE_CMD_DO_SET_MODE, 1, 2);
```

> Mode 2 = AUTO

### 4. Initiate Takeoff

```cpp
send_vehicle_command(VEHICLE_CMD_NAV_TAKEOFF);
```

---

### 5. Hover and Monitor

While hovering:
- Logs position, velocity, battery, and angular rate every few seconds
- Records home position

---

### 6. Monitors for Conditions to Land

It will trigger landing if any of these happen:

| Condition | Trigger |
|-----------|---------|
| Battery below `BAT_LOW_THR` | Lands |
| RC Signal Lost | Lands |
| Time Elapsed > Hover Duration | Lands |


---

### 7. Detect Landing and Disarm

Monitors:

```cpp
vehicle_land_detected_s.landed == true
```

Then safely disarms:

```cpp
send_vehicle_command(VEHICLE_CMD_COMPONENT_ARM_DISARM, ARMING_ACTION_DISARM);
```

---

## Logged Data

Every 3 seconds, prints:

- Position: `X`, `Y`, `Height`
- Velocity: `Vertical`, `Horizontal`
- Battery: `% Remaining`, `Current`
- Angular Velocity: `Roll`, `Pitch`, `Yaw`

