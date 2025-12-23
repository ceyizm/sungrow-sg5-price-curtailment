# Sungrow SG5.0RS Modbus Register Reference

Complete register map for the Sungrow SG5.0RS string inverter via WiNet-S dongle.

## Important Notes

### Register Address Convention
- **Register number** (from Sungrow docs): e.g., 5006
- **Modbus address** (what you write in config): Register - 1
- **Example:** Register 5006 = Address 5005

### Data Types
- `uint16` - Unsigned 16-bit integer
- `uint32` - Unsigned 32-bit integer (requires `swap: word`)
- `int16` - Signed 16-bit integer
- `int32` - Signed 32-bit integer (requires `swap: word`)

### Input Types
- `input` - Read-only registers (sensors)
- `holding` - Read/write registers (controls)

## Control Registers (Holding - Read/Write)

### Core Control

| Register | Address | Name | Type | Function | Write Values |
|----------|---------|------|------|----------|--------------|
| 5006 | 5005 | Run Mode Control | uint16 | Start/Stop inverter | 206 = Shutdown<br>207 = Enabled |
| 5007 | 5006 | Power Limit Toggle | uint16 | Enable/disable limiting | 85 = Disabled<br>170 = Enabled |
| 5008 | 5007 | Power Limit % | uint16 | Output power limit | 0-1000 (×0.1%) |

### Control Examples

**Shutdown inverter:**
```yaml
service: modbus.write_register
data:
  hub: SungrowSHx
  slave: 1
  address: 5005
  value: 206
```

**Enable inverter:**
```yaml
service: modbus.write_register
data:
  hub: SungrowSHx
  slave: 1
  address: 5005
  value: 207
```

**Set 50% power limit:**
```yaml
service: modbus.write_register
data:
  hub: SungrowSHx
  slave: 1
  address: 5007
  value: 500  # 50.0%
```

## Monitoring Registers (Input - Read Only)

### Basic Information

| Register | Address | Name | Type | Scale | Unit | Scan Interval |
|----------|---------|------|------|-------|------|---------------|
| 5000 | 4999 | Device Type Code | uint16 | 1 | - | 1200s |
| 5001 | 5000 | Nominal Active Power | uint16 | 0.1 | kW | 1200s |
| 5038 | 5037 | Work State Code | uint16 | 1 | - | 60s |

**Work State Codes:**
- `0x0000` (0) = Run
- `0x8000` (32768) = Stop
- `0x1300` (4864) = Key Stop
- `0x1400` (5120) = Standby
- `0x1500` (5376) = Emergency Stop
- `0x1600` (5632) = Starting
- `0x5500` (21760) = Fault
- `0x8100` (33024) = Derating Run
- `0x8200` (33280) = Dispatch Run
- `0x9100` (37120) = Alarm Run

### Energy Production

| Register | Address | Name | Type | Scale | Unit | State Class |
|----------|---------|------|------|-------|------|-------------|
| 5003 | 5002 | Daily PV Generation | uint16 | 0.1 | kWh | total_increasing |
| 5004-5005 | 5003 | Total PV Generation | uint32 | 1 | kWh | total_increasing |
| 5006-5007 | 5005 | Total Running Time | uint32 | 1 | h | measurement |
| 5113 | 5112 | Daily Running Time | uint16 | 1 | min | measurement |

### DC Power (Solar Input)

| Register | Address | Name | Type | Scale | Unit | Scan Interval |
|----------|---------|------|------|-------|------|---------------|
| 5011 | 5010 | MPPT1 Voltage | uint16 | 0.1 | V | 15s |
| 5012 | 5011 | MPPT1 Current | uint16 | 0.1 | A | 15s |
| 5013 | 5012 | MPPT2 Voltage | uint16 | 0.1 | V | 15s |
| 5014 | 5013 | MPPT2 Current | uint16 | 0.1 | A | 15s |
| 5017-5018 | 5016 | Total DC Power | uint32 | 1 | W | 5s |

### AC Power (Grid Output)

| Register | Address | Name | Type | Scale | Unit | Scan Interval |
|----------|---------|------|------|-------|------|---------------|
| 5009-5010 | 5008 | Total Apparent Power | uint32 | 1 | VA | 60s |
| 5031-5032 | 5030 | Total Active Power | uint32 | 1 | W | 5s |
| 5033-5034 | 5032 | Total Reactive Power | int32 | 1 | var | 60s |
| 5035 | 5034 | Power Factor | int16 | 0.001 | % | 60s |
| 5148 | 5147 | Grid Frequency | uint16 | 0.01 | Hz | 60s |

### Grid Meter (Import/Export)

| Register | Address | Name | Type | Scale | Unit | Description |
|----------|---------|------|------|-------|------|-------------|
| 5083-5084 | 5082 | Meter Power | int32 | 1 | W | + = export, - = import |
| 5095-5096 | 5094 | Total Exported Energy | uint32 | 0.1 | kWh | Energy sold to grid |
| 5099-5100 | 5098 | Total Imported Energy | uint32 | 0.1 | kWh | Energy bought from grid |

### Environmental

| Register | Address | Name | Type | Scale | Unit | Scan Interval |
|----------|---------|------|------|-------|------|---------------|
| 5008 | 5007 | Inverter Temperature | int16 | 0.1 | °C | 600s |

## Configuration Examples

### Minimal Sensor Configuration

```yaml
modbus:
  - name: SungrowSHx
    type: tcp
    host: !secret sungrow_modbus_host_ip
    port: !secret sungrow_modbus_port

    sensors:
      # Control register (read state)
      - name: Run State Register
        device_address: !secret sungrow_modbus_slave
        address: 5005
        input_type: holding
        data_type: uint16
        scan_interval: 30

      # Solar production
      - name: Total DC Power
        device_address: !secret sungrow_modbus_slave
        address: 5016
        input_type: input
        data_type: uint32
        swap: word
        unit_of_measurement: W
        device_class: power
        state_class: measurement
        scan_interval: 5
```

### Full Monitoring Configuration

See `/config/integrations/modbus_sungrow.yaml` in this repository for complete configuration.

## Calculated Sensors

These are not direct modbus registers but calculated from multiple sensors:

### MPPT Power Calculation

```yaml
template:
  - sensor:
      - name: MPPT1 Power
        unit_of_measurement: W
        state: "{{ (states('sensor.mppt1_voltage') | float * states('sensor.mppt1_current') | float) | int }}"
```

**Formula:** Power (W) = Voltage (V) × Current (A)

### Load Power Calculation

```yaml
template:
  - sensor:
      - name: Load Power
        unit_of_measurement: W
        state: "{{ (states('sensor.total_active_power') | float + states('sensor.meter_power') | float) | int }}"
```

**Formula:** Load = Inverter Output + Grid Power

**Logic:**
- If exporting (+meter): Load = Output + Export
- If importing (-meter): Load = Output - Import

## Scan Interval Recommendations

| Data Type | Recommended Interval | Reason |
|-----------|---------------------|--------|
| Power (W) | 5s | Real-time monitoring |
| Voltage/Current | 15s | Balance detail vs. load |
| Energy (kWh) | 60s | Slowly changing |
| Temperature | 600s (10 min) | Very slowly changing |
| Static info | 1200s (20 min) | Almost never changes |

**Note:** Shorter intervals increase network traffic and system load. Find the balance for your needs.

## Troubleshooting

### Register Returns 0 or Unavailable

**Possible causes:**
1. Register not supported on SG5.0RS (check model compatibility)
2. Wrong data type (uint16 vs uint32)
3. Missing `swap: word` for 32-bit registers
4. Incorrect address (remember to subtract 1)
5. WiNet-S doesn't expose this register

**Debug:**
```yaml
# Try reading as raw uint16
- name: Debug Register
  address: 5005
  input_type: holding
  data_type: uint16
```

### Values Seem Wrong

**Check scaling:**
- Most power values: scale 1
- Voltage/current: scale 0.1
- Energy: daily = 0.1, total = 1
- Power factor: scale 0.001

**Example fix:**
```yaml
# If temperature shows 410 instead of 41.0:
scale: 0.1  # ← Add this
```

### Can't Write to Control Registers

**Requirements:**
1. Must use `input_type: holding`
2. Address must be exact (5005 for register 5006)
3. Value must be exact (206 or 207, not 205 or 208)
4. WiNet-S must support writes (not all do)

**Test:**
```bash
# Use modpoll tool to test:
modpoll -m tcp -a 1 -r 5006 -t 4 192.168.1.145 206
```

## References

- [Official Sungrow Modbus Protocol](https://github.com/bohdan-s/Sungrow-Inverter/blob/main/Modbus%20Information/)
- [SunGather Register Database](https://github.com/bohdan-s/SunGather)
- [mkaiser's Integration](https://github.com/mkaiser/Sungrow-SHx-Inverter-Modbus-Home-Assistant)

## Contributing

Found an error or have additional registers? Open a PR to update this document!
