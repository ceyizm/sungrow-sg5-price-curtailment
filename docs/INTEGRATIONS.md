# Required Integrations

This project requires the following Home Assistant integrations to function properly:

## Core Integrations

### 1. Teslemetry Integration (Battery Data)

**Purpose:** Provides Tesla Powerwall battery level data for curtailment decisions.

**Installation:**
1. Go to **Settings → Devices & Services → Add Integration**
2. Search for "Teslemetry"
3. Follow the setup wizard
4. You'll need:
   - Tesla account credentials
   - Teslemetry API access token

**Required Sensor:**
- `sensor.powerwall_percentage_charged` - Battery state of charge (0-100%)

**Official Documentation:** https://www.home-assistant.io/integrations/teslemetry/

**Alternative Battery Integrations:**
If you don't have a Tesla Powerwall, you can use any battery integration that provides a percentage sensor:
- Sonnen battery
- LG Chem RESU
- BYD Battery-Box
- Any solar battery with HA integration

Just update the sensor name in `/config/packages/sungrow_control.yaml`:
```yaml
# Replace this line (~110):
sensor.powerwall_percentage_charged
# With your battery sensor:
sensor.your_battery_percentage
```

---

### 2. Amber Electric Integration (Dynamic Pricing)

**Purpose:** Provides real-time electricity feed-in tariff data for price-based curtailment.

**Installation:**
1. Have an Amber Electric account (Australian customers)
2. Go to **Settings → Devices & Services → Add Integration**
3. Search for "Amber Electric"
4. Enter your Amber API key (get from Amber app/website)

**Required Sensor:**
- `sensor.sandhurst_estate_feed_in_price` - Current feed-in tariff ($/kWh)
  - Note: "sandhurst_estate" is the NMI site ID - yours may differ

**Official Documentation:** https://www.home-assistant.io/integrations/amberelectric/

**Finding Your Sensor Name:**
After installing Amber integration:
1. Go to **Developer Tools → States**
2. Search for "amber" or "feed_in"
3. Find your feed-in price sensor (usually `sensor.<site_name>_feed_in_price`)
4. Update in `/config/packages/sungrow_control.yaml`

**Alternative Pricing Integrations:**
If you're not with Amber Electric or not in Australia:
- **Octopus Energy** (UK) - `octopus_energy` integration
- **Tibber** (Europe) - `tibber` integration
- **Generic REST sensor** for your retailer's API
- **Manual pricing** - create input_number instead

Example for manual pricing:
```yaml
input_number:
  manual_feed_in_price:
    name: "Current Feed-in Price"
    min: -0.50
    max: 1.00
    step: 0.01
    unit_of_measurement: "$/kWh"
```

Then update automations to use `input_number.manual_feed_in_price` instead.

---

### 3. Modbus Integration (Inverter Control)

**Purpose:** Communicates with Sungrow inverter for reading sensors and sending control commands.

**Installation:**
Built into Home Assistant - configured via YAML (included in this repo)

**Configuration:**
Handled by `/config/integrations/modbus_sungrow.yaml`

**Required:**
- Sungrow inverter IP address (WiNet-S or direct LAN)
- Port 502 accessible
- Modbus slave ID (usually 1)

**No additional integration needed** - modbus is core to Home Assistant.

---

## Sensor Mapping Reference

Edit `/config/packages/sungrow_control.yaml` to match your sensor names:

| Feature | Default Sensor | Line # | Your Replacement |
|---------|---------------|--------|------------------|
| Battery Level | `sensor.powerwall_percentage_charged` | ~110, 119, 127, 144, 151, 164 | `sensor.your_battery_soc` |
| Feed-in Price | `sensor.sandhurst_estate_feed_in_price` | ~115, 129, 155, 147 | `sensor.your_feedin_price` |
| Grid Power | `sensor.inverter_meter_power` | ~114, 143, 149, 158 | (Auto-created by modbus) |

## Testing Integrations

**Verify Teslemetry:**
```bash
# Check battery sensor exists
ha states get sensor.powerwall_percentage_charged

# Should return: state: "85" (or similar)
```

**Verify Amber:**
```bash
# Check pricing sensor exists
ha states get sensor.sandhurst_estate_feed_in_price

# Should return: state: "0.15" (or similar)
```

**Verify Modbus:**
```bash
# Check inverter sensor exists
ha states get sensor.inverter_total_dc_power

# Should return: state: "1500" (or similar)
```

## Troubleshooting

**Teslemetry not working:**
- Ensure Tesla account has Powerwall access
- Check Teslemetry API token is valid
- Verify Powerwall is online in Tesla app

**Amber not showing prices:**
- Confirm Amber account is active
- API key must have read permissions
- NMI must match your site

**Modbus not connecting:**
- Ping inverter IP address
- Check port 502 is not firewalled
- Verify WiNet-S is on same network as HA
- Try rebooting WiNet-S dongle

## Optional Enhancements

### EMHASS Integration
For advanced battery optimization:
- Install EMHASS add-on
- Integrates with curtailment for optimal scheduling
- See [EMHASS documentation](https://emhass.readthedocs.io/)

### Energy Dashboard
Add sensors to Energy Dashboard:
1. **Settings → Dashboards → Energy**
2. **Solar Production:** `sensor.inverter_daily_power_yield_raw`
3. **Grid Import:** `sensor.inverter_total_imported_energy`
4. **Grid Export:** `sensor.inverter_total_exported_energy`
5. **Battery:** (from Teslemetry integration)
