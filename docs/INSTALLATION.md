# Detailed Installation Guide

Step-by-step installation instructions for the Sungrow SG5.0RS Price-Aware Curtailment system.

## Prerequisites

Before starting, ensure you have:

✅ Home Assistant installed and running (2024.1+)
✅ Sungrow SG5.0RS inverter with WiNet-S dongle or LAN connection
✅ Inverter IP address (e.g., 192.168.1.145)
✅ Tesla Powerwall with Teslemetry integration configured
✅ Amber Electric account and integration configured
✅ File Editor or SSH access to Home Assistant

## Step 1: Find Your Inverter IP Address

### Option A: Check Router DHCP Table
1. Log into your router admin panel
2. Look for "DHCP Clients" or "Connected Devices"
3. Find "WiNet-S" or a device with Sungrow's MAC address
4. Note the IP address (e.g., 192.168.1.145)

### Option B: Check WiNet-S WiFi Interface
1. Connect to WiNet-S WiFi network (if available)
2. Open browser to `http://10.10.100.254` or `http://192.168.10.1`
3. Find network settings showing current IP

### Option C: Port Scan
```bash
nmap -p 502 192.168.1.0/24
```
Look for devices with port 502 open (Modbus).

## Step 2: Test Modbus Connection

Before installing, verify you can communicate with the inverter:

1. Go to **Settings → Devices & Services → Integrations**
2. Click **Add Integration** → Search "Modbus"
3. Configure test connection:
   ```yaml
   name: Test
   type: tcp
   host: 192.168.1.145  # Your inverter IP
   port: 502
   ```
4. Add a test sensor:
   ```yaml
   - name: Test DC Power
     address: 5016
     slave: 1
     data_type: uint32
     swap: word
   ```
5. Check if sensor shows valid data
6. Remove test integration once confirmed working

## Step 3: Clone Repository

### Option A: Via Git (Recommended)
```bash
cd /config
git clone https://github.com/Artic0din/sungrow-sg5-price-curtailment.git temp
cp -r temp/config/* /config/
rm -rf temp
```

### Option B: Manual Download
1. Download ZIP from GitHub
2. Extract files
3. Copy to Home Assistant via File Editor:
   - `config/packages/sungrow_control.yaml` → `/config/packages/`
   - `config/integrations/modbus_sungrow.yaml` → `/config/integrations/`
   - `config/sungrow_customize.yaml` → `/config/`

## Step 4: Create Directory Structure

```bash
# Create required directories if they don't exist
mkdir -p /config/packages
mkdir -p /config/integrations
```

## Step 5: Configure Secrets

Edit `/config/secrets.yaml` and add:

```yaml
# Sungrow Inverter Modbus Configuration
sungrow_modbus_host_ip: 192.168.1.145  # ← Change to YOUR inverter IP
sungrow_modbus_port: 502
sungrow_modbus_slave: 1
```

**Security Note:** Never commit `secrets.yaml` to version control!

## Step 6: Verify Integration Sensors

Before proceeding, confirm these sensors exist:

### Teslemetry Sensors
```bash
# Check battery sensor
ha states get sensor.powerwall_percentage_charged
```

If sensor name is different:
1. Go to **Developer Tools → States**
2. Search for "powerwall" or "battery"
3. Note the exact entity_id
4. Update in next step

### Amber Electric Sensors
```bash
# Check pricing sensor
ha states get sensor.sandhurst_estate_feed_in_price
```

If sensor name is different:
1. Search for "amber" or "feed" in States
2. Note your feed-in price sensor name
3. Update in next step

## Step 7: Customize Sensor Names

Edit `/config/packages/sungrow_control.yaml` to match YOUR sensor names:

**Find and replace these:**

1. **Battery sensor** (appears ~6 times):
   ```yaml
   # Find:
   sensor.powerwall_percentage_charged

   # Replace with your battery sensor:
   sensor.your_battery_name_here
   ```

2. **Feed-in price sensor** (appears ~4 times):
   ```yaml
   # Find:
   sensor.sandhurst_estate_feed_in_price

   # Replace with your pricing sensor:
   sensor.your_pricing_name_here
   ```

**Quick find/replace in File Editor:**
- Open `/config/packages/sungrow_control.yaml`
- Use Ctrl+F (or Cmd+F) to find
- Replace all instances

## Step 8: Update Configuration.yaml

Edit `/config/configuration.yaml` and add these sections:

```yaml
homeassistant:
  packages:
    sungrow: !include packages/sungrow_control.yaml
  customize: !include sungrow_customize.yaml

modbus: !include integrations/modbus_sungrow.yaml
```

**Important Notes:**

If you already have a `homeassistant:` section:
```yaml
homeassistant:
  # ... your existing settings ...
  packages:  # ← Add this
    sungrow: !include packages/sungrow_control.yaml
  customize: !include sungrow_customize.yaml  # ← Add this
```

If you already have `modbus:` configured:
- Merge the Sungrow config into your existing modbus setup
- Or use `!include_dir_merge_list integrations/`

## Step 9: Validate Configuration

Before restarting, check for errors:

```bash
ha core check
```

**Expected output:** "Configuration valid!"

**If errors occur:**
- Check YAML indentation (must use spaces, not tabs)
- Verify all sensor names exist
- Ensure files are in correct directories
- Check secrets.yaml has correct IP

## Step 10: Restart Home Assistant

```bash
ha core restart
```

Wait ~1 minute for restart to complete.

## Step 11: Verify Installation

### Check Entities Created

Go to **Developer Tools → States** and verify:

**Modbus Sensors (should exist):**
- `sensor.inverter_total_dc_power`
- `sensor.inverter_meter_power`
- `sensor.inverter_total_active_power`
- `sensor.inverter_work_state_code`
- `sensor.inverter_run_state_register`

**Template Sensors (should exist):**
- `binary_sensor.pv_generating`
- `binary_sensor.exporting_power`
- `binary_sensor.importing_power`
- `sensor.inverter_mppt1_power`
- `sensor.inverter_mppt2_power`
- `sensor.inverter_load_power`
- `sensor.inverter_work_state`
- `sensor.inverter_run_mode`

**Control Inputs (should exist):**
- `input_select.set_sg_inverter_run_mode`
- `input_boolean.sungrow_curtailment_enabled`
- `input_number.sungrow_curtailment_price_threshold_low`
- `input_number.sungrow_restart_price_threshold_high`
- `input_number.sungrow_curtailment_powerwall_threshold`

**Automations (should exist):**
Go to **Settings → Automations & Scenes**:
- "Sungrow: Write run mode to modbus"
- "Sungrow: Shutdown when Powerwall full and low/negative feed-in price"
- "Sungrow: Restart when conditions favorable"
- "Sungrow: Restart at sunset (safety)"

### Check for Errors

Go to **Settings → System → Logs** and search for:
- "sungrow"
- "modbus"

No critical errors should appear.

## Step 12: Test Manual Control

**Critical test - do this before enabling automation:**

1. Go to `input_select.set_sg_inverter_run_mode`
2. Note current `sensor.inverter_total_dc_power` value (e.g., 1500W)
3. Select **"Shutdown"**
4. Wait 10 seconds
5. Check `sensor.inverter_total_dc_power` → should be **0W**
6. Check `sensor.inverter_run_state_register` → should be **206**
7. Select **"Enabled"**
8. Wait 10 seconds
9. Check power returns to normal (~1500W)
10. Check register → should be **207**

**If test fails:**
- Verify IP address in secrets.yaml is correct
- Check firewall not blocking port 502
- Try rebooting WiNet-S dongle
- See [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

## Step 13: Configure Thresholds

Set your desired curtailment thresholds:

1. Go to **Settings → Devices & Services → Helpers**
2. Find these inputs and set:

| Input | Recommended | Description |
|-------|-------------|-------------|
| Shutdown Price Threshold | $0.00 | Curtail when ≤ this |
| Restart Price Threshold | $0.05 | Resume when > this |
| Powerwall Threshold | 99% | Only curtail when ≥ this |

## Step 14: Enable Automation

1. Find `input_boolean.sungrow_curtailment_enabled`
2. Toggle to **ON**
3. System is now active!

## Step 15: Monitor First Day

**Watch for:**
- Automation triggering correctly during low prices
- Notifications arriving
- Inverter responding to commands
- No unexpected shutdowns

**Check traces:**
- **Settings → Automations → Traces**
- View execution history
- Verify conditions evaluated correctly

## Step 16: (Optional) Add Dashboard

Copy dashboard YAML from [examples/dashboard.yaml](../examples/dashboard.yaml) to your Lovelace configuration.

Or create cards manually using the entities created.

## Rollback Procedure

If something goes wrong and you need to undo:

1. **Disable automation:**
   ```
   input_boolean.sungrow_curtailment_enabled → OFF
   ```

2. **Remove from configuration.yaml:**
   ```yaml
   # Comment out or remove:
   # homeassistant:
   #   packages:
   #     sungrow: !include packages/sungrow_control.yaml
   # modbus: !include integrations/modbus_sungrow.yaml
   ```

3. **Restart:**
   ```bash
   ha core restart
   ```

4. **Delete files (if needed):**
   ```bash
   rm /config/packages/sungrow_control.yaml
   rm /config/integrations/modbus_sungrow.yaml
   rm /config/sungrow_customize.yaml
   ```

## Next Steps

- Read [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues
- Check [INTEGRATIONS.md](INTEGRATIONS.md) for alternative setups
- Join discussions for support

## Support

Having issues? Open an issue on GitHub with:
- Home Assistant version
- Error logs (Settings → System → Logs)
- Your configuration (sensor names changed)
- Steps to reproduce
