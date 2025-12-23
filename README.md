# Sungrow SG5.0RS Price-Aware Solar Curtailment for Home Assistant

Automatically curtail solar production when your battery is full and feed-in prices are zero or negative, preventing you from paying to export electricity to the grid.

## 🌟 Features

- **Price-Aware Curtailment**: Automatically shuts down solar when exporting at zero/negative prices
- **Battery Protection**: Only curtails when Powerwall/battery is ≥99% full
- **Smart Restart**: Automatically resumes when:
  - Battery needs charging
  - Home is consuming power
  - Feed-in prices become profitable
- **Manual Override**: Dashboard controls for manual start/stop
- **Configurable Thresholds**: Adjust price and battery thresholds via UI
- **Safety Features**: Always re-enables at sunset for next day
- **Notifications**: Alerts when curtailment occurs

## 📋 Requirements

### Hardware
- **Sungrow SG5.0RS** string inverter (or compatible SG-series model)
- **WiNet-S** WiFi/Ethernet dongle (or direct LAN connection)
- **Home Assistant** instance
- **Battery system** with HA integration (Tesla Powerwall, etc.)
- **Dynamic pricing sensor** (Amber Electric, etc.)

### Software
- Home Assistant 2024.1 or newer
- Modbus integration (built-in)
- **[Teslemetry Integration](https://www.home-assistant.io/integrations/teslemetry/)** - For Tesla Powerwall battery data
- **[Amber Electric Integration](https://www.home-assistant.io/integrations/amberelectric/)** - For dynamic pricing data (Australian users)

**Note:** You can use alternative battery and pricing integrations - see [docs/INTEGRATIONS.md](docs/INTEGRATIONS.md) for details.

## 🚀 Quick Start

### 1. Installation

```bash
# Clone this repository
git clone https://github.com/Artic0din/sungrow-sg5-price-curtailment.git
cd sungrow-sg5-price-curtailment

# Copy files to Home Assistant config
cp config/packages/sungrow_control.yaml /config/packages/
cp config/integrations/modbus_sungrow.yaml /config/integrations/
cp config/sungrow_customize.yaml /config/
```

### 2. Configure Secrets

Add to `/config/secrets.yaml`:

```yaml
sungrow_modbus_host_ip: 192.168.x.x  # Your inverter IP
sungrow_modbus_port: 502
sungrow_modbus_slave: 1
```

### 3. Update Configuration

Add to `/config/configuration.yaml`:

```yaml
homeassistant:
  packages:
    sungrow: !include packages/sungrow_control.yaml
  customize: !include sungrow_customize.yaml

modbus: !include integrations/modbus_sungrow.yaml
```

### 4. Customize for Your Setup

Edit `/config/packages/sungrow_control.yaml` and update these sensor names to match your system:

```yaml
# Line ~110: Update to your battery sensor
sensor.powerwall_percentage_charged

# Line ~115: Update to your feed-in price sensor
sensor.sandhurst_estate_feed_in_price
```

### 5. Restart Home Assistant

```bash
ha core restart
```

## ⚙️ Configuration

### Setting Thresholds

Configure via UI in **Settings → Helpers** or directly in your dashboard:

| Setting | Default | Description |
|---------|---------|-------------|
| **Curtail Below Price** | $0.00/kWh | Shutdown when feed-in price ≤ this value |
| **Restart Above Price** | $0.05/kWh | Resume when feed-in price > this value |
| **Battery Threshold** | 99% | Only curtail when battery ≥ this level |

### Price Thresholds Explained

**Curtail Threshold ($0.00/kWh):**
- Set to `0.00` to curtail during zero and negative pricing
- Set to `0.05` to curtail when price is below 5 cents
- Set to `-0.10` to only curtail when you'd pay more than 10c to export

**Restart Threshold ($0.05/kWh):**
- Should be HIGHER than curtail threshold to prevent flapping
- Set based on when exporting becomes profitable for you
- Consider your import tariff when setting this

### Example Scenarios

**Scenario 1: Negative Pricing Event**
```
Time: 2:00 PM
Feed-in: -$0.05/kWh (you pay to export!)
Battery: 100%
Export: 2000W

→ 2:03 PM: Solar shuts down automatically
→ Saves you ~$0.10/hour
→ 3:00 PM: Price returns to +$0.10/kWh
→ 3:02 PM: Solar restarts automatically
```

**Scenario 2: Zero Feed-In**
```
Time: 11:00 AM
Feed-in: $0.00/kWh (no payment)
Battery: 99%
Export: 1500W

→ 11:03 AM: Solar shuts down
→ Reduces inverter wear, extends equipment life
→ 11:45 AM: Battery drops to 94%
→ 11:47 AM: Solar restarts (battery needs charge)
```

**Scenario 3: High Peak Pricing**
```
Time: 5:00 PM
Feed-in: $0.25/kWh (peak rate)
Battery: 100%
Export: 3000W

→ Solar STAYS ON (above restart threshold)
→ Earns ~$0.75/hour from export
```

## 🎮 Usage

### Manual Control

**Dashboard Controls:**
- `input_select.set_sg_inverter_run_mode` - Manual start/stop
- `input_boolean.sungrow_curtailment_enabled` - Enable/disable automation

**Via UI:**
1. Go to your dashboard
2. Find "Sungrow Inverter Run Mode" dropdown
3. Select "Shutdown" or "Enabled"

### Monitoring

**Check Current Status:**
- **Developer Tools → States** - Search "inverter" to see all sensors
- **Settings → Automations → Traces** - View automation execution history
- **Notifications** - Receive alerts when curtailment occurs

**Key Sensors:**
- `sensor.inverter_total_dc_power` - Current solar production
- `sensor.inverter_run_mode` - Current inverter state
- `sensor.inverter_work_state` - Detailed inverter status
- `binary_sensor.pv_generating` - Solar generation indicator

### Automation Traces

View detailed execution logs:
1. Go to **Settings → Automations & Scenes**
2. Click on "Sungrow: Shutdown when Powerwall full..."
3. Click **⋮ → Traces**
4. See exact conditions and actions taken

## 📊 Dashboard

Add these to your Lovelace dashboard:

```yaml
# Manual Control Card
type: entities
title: Solar Control
entities:
  - entity: input_select.set_sg_inverter_run_mode
    name: Inverter Mode
  - entity: input_boolean.sungrow_curtailment_enabled
    name: Auto Curtailment
  - entity: sensor.inverter_run_mode
    name: Current State
  - entity: sensor.inverter_total_dc_power
    name: Solar Production

# Threshold Settings Card
type: entities
title: Curtailment Settings
entities:
  - entity: input_number.sungrow_curtailment_price_threshold_low
    name: Shutdown Price
  - entity: input_number.sungrow_restart_price_threshold_high
    name: Restart Price
  - entity: input_number.sungrow_curtailment_powerwall_threshold
    name: Battery Threshold

# Current Conditions Card
type: entities
title: Current Conditions
entities:
  - entity: sensor.powerwall_percentage_charged
    name: Battery Level
  - entity: sensor.sandhurst_estate_feed_in_price
    name: Feed-in Price
  - entity: sensor.inverter_meter_power
    name: Grid Power
  - entity: binary_sensor.pv_generating
    name: Generating
```

See [examples/dashboard.yaml](examples/dashboard.yaml) for a complete dashboard configuration.

## 🔧 Troubleshooting

### Automation Not Triggering

**Check conditions:**
1. `input_boolean.sungrow_curtailment_enabled` is ON
2. All sensors are available (not "unknown" or "unavailable")
3. Conditions sustained for full 3 minutes
4. Currently daytime (between sunrise/sunset)

**Debug:**
- Go to automation → **⋮ → Traces** to see why it didn't fire
- Check template conditions in **Developer Tools → Template**

### Inverter Not Responding to Commands

**Verify modbus connection:**
1. **Developer Tools → Services**
2. Test direct modbus write:
   ```yaml
   service: modbus.write_register
   data:
     hub: SungrowSHx
     slave: 1
     address: 5005
     value: 206
   ```
3. Check if `sensor.inverter_total_dc_power` drops to 0W

**Common issues:**
- WiNet-S IP address changed (check router DHCP)
- Modbus slave ID incorrect (should be 1)
- Firewall blocking port 502
- WiNet-S requires write-enable setting (check web interface)

### Template Sensors Unavailable

**Check sensor names match:**
- Open `/config/packages/sungrow_control.yaml`
- Find the `template:` section
- Verify all sensor names exist in **Developer Tools → States**

**Common mismatches:**
- `sensor.total_dc_power` vs `sensor.inverter_total_dc_power`
- `sensor.meter_power` vs `sensor.inverter_meter_power`

**Fix:**
- Edit template sensor names to match your actual entities
- Reload: **Developer Tools → YAML → Template Entities**

### Duplicate Sensors with "_2" Suffix

**Cause:** Multiple modbus configurations loaded

**Fix:**
1. Search `/config/` for other `modbus_sungrow*.yaml` files
2. Check **Settings → Devices & Services** for old Sungrow integration
3. Remove duplicates, restart Home Assistant
4. Delete unavailable entities: **Settings → Entities → Filter: unavailable**

## 🔐 Security Notes

- Modbus TCP has no authentication - ensure WiNet-S is on a trusted network
- Consider firewall rules to restrict modbus access
- Use Home Assistant's built-in authentication for dashboard access
- Keep Home Assistant updated for security patches

## 📝 How It Works

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Home Assistant                        │
│                                                          │
│  ┌──────────────┐    ┌───────────────┐                 │
│  │   Dashboard  │───▶│  Automations  │                 │
│  │   Controls   │    │   & Logic     │                 │
│  └──────────────┘    └───────┬───────┘                 │
│                              │                          │
│  ┌──────────────┐    ┌───────▼───────┐                 │
│  │   Sensors    │◀───│ Modbus TCP    │                 │
│  │  (battery,   │    │ Integration   │                 │
│  │   pricing)   │    └───────┬───────┘                 │
│  └──────────────┘            │                          │
└──────────────────────────────┼──────────────────────────┘
                               │ Port 502
                        ┌──────▼──────┐
                        │  WiNet-S    │
                        │   Dongle    │
                        └──────┬──────┘
                               │ RS485
                        ┌──────▼──────┐
                        │  Sungrow    │
                        │  SG5.0RS    │
                        │  Inverter   │
                        └─────────────┘
```

### Control Flow

1. **Monitoring Loop** (every 30 seconds):
   - Read modbus sensors from inverter
   - Check battery level
   - Check feed-in price
   - Evaluate curtailment conditions

2. **Curtailment Decision** (when conditions met for 3 minutes):
   - Battery ≥ threshold
   - Exporting > 100W
   - Price ≤ threshold
   - Set `input_select` to "Shutdown"

3. **Modbus Write** (triggered by input_select change):
   - Write value `206` to register `5005`
   - Inverter receives shutdown command
   - Solar production stops

4. **Restart Decision** (when any condition met for 1-2 minutes):
   - Battery < 95%, OR
   - Not exporting, OR
   - Importing from grid, OR
   - Price > restart threshold
   - Set `input_select` to "Enabled"

5. **Modbus Write** (re-enable):
   - Write value `207` to register `5005`
   - Inverter receives enable command
   - Solar production resumes

### Modbus Register Reference

| Register | Address | Function | Write Values |
|----------|---------|----------|--------------|
| 5006 | 5005 | Run Mode Control | 206=Shutdown, 207=Enabled |
| 5007 | 5006 | Power Limit Toggle | 85=Disabled, 170=Enabled |
| 5008 | 5007 | Power Limit % | 0-1000 (×0.1%) |

See [docs/MODBUS_REGISTERS.md](docs/MODBUS_REGISTERS.md) for complete register map.

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [mkaiser's Sungrow Integration](https://github.com/mkaiser/Sungrow-SHx-Inverter-Modbus-Home-Assistant) - Modbus register documentation
- [SunGather Project](https://github.com/bohdan-s/SunGather) - Register definitions
- Sungrow official documentation - Communication protocols
- Home Assistant community

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/Artic0din/sungrow-sg5-price-curtailment/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Artic0din/sungrow-sg5-price-curtailment/discussions)
- **Home Assistant Community**: [Forum Thread](https://community.home-assistant.io/)

## 🗺️ Roadmap

- [ ] Support for additional Sungrow models (SG3.0RS, SG6.0RS)
- [ ] Historical analytics dashboard
- [ ] Mobile app integration
- [ ] Multi-inverter support

## ⚠️ Disclaimer

This software controls your solar inverter. While tested and working, use at your own risk. Always monitor the system for the first few days after installation. Not affiliated with Sungrow.

---

**Made with ☀️ for the Home Assistant community**

**Star this repo if it saves you money! ⭐**
