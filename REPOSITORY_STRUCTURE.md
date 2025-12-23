# Repository Structure

Complete file tree and description of the Sungrow SG5.0RS Price-Aware Curtailment repository.

## Directory Tree

```
sungrow-sg5-price-curtailment/
├── .github/                          # GitHub specific files (for Actions, templates, etc.)
├── config/                           # Home Assistant configuration files
│   ├── integrations/
│   │   └── modbus_sungrow.yaml      # Modbus sensor definitions
│   ├── packages/
│   │   └── sungrow_control.yaml     # Main package (automations, inputs, templates)
│   └── sungrow_customize.yaml       # Entity customizations (friendly names)
├── docs/                             # Documentation
│   ├── INSTALLATION.md              # Detailed installation guide
│   ├── INTEGRATIONS.md              # Required integrations (Teslemetry, Amber)
│   └── MODBUS_REGISTERS.md          # Complete register reference
├── examples/                         # Example configurations
│   ├── dashboard.yaml               # Lovelace dashboard configuration
│   └── secrets.yaml.example         # Example secrets file
├── .gitignore                        # Git ignore rules
├── CONTRIBUTING.md                   # Contribution guidelines
├── LICENSE                           # MIT License
└── README.md                         # Main documentation

```

## File Descriptions

### Root Files

| File | Purpose |
|------|---------|
| **README.md** | Main project documentation, quick start, features |
| **LICENSE** | MIT License - open source terms |
| **CONTRIBUTING.md** | Guidelines for contributing to the project |
| **.gitignore** | Prevents committing sensitive files (secrets, logs) |

### Configuration Files (`/config`)

#### `/config/integrations/modbus_sungrow.yaml`
**Purpose:** Modbus TCP configuration for Sungrow inverter communication

**Contains:**
- Connection settings (IP, port, slave ID)
- 20+ sensor definitions (power, energy, voltage, current, etc.)
- Control register definitions (run state, power limiting)
- Scan intervals optimized for performance

**Used by:** `modbus: !include integrations/modbus_sungrow.yaml` in configuration.yaml

**Key sections:**
- Control registers (holding) - read/write
- Production sensors (input) - read-only
- MPPT string monitoring
- Grid import/export tracking
- Temperature and status monitoring

#### `/config/packages/sungrow_control.yaml`
**Purpose:** Complete curtailment system in one package file

**Contains:**
- `input_select` - Manual inverter control (Enabled/Shutdown)
- `input_number` - Price and battery thresholds (3 sliders)
- `input_boolean` - Enable/disable curtailment automation
- `automation` - 4 automations:
  1. Write modbus commands when input_select changes
  2. Shutdown when Powerwall full + low price
  3. Restart when conditions favorable
  4. Safety restart at sunset
- `template` - Calculated sensors:
  - Binary sensors (pv_generating, exporting, importing)
  - MPPT power calculations
  - Load power calculation
  - State decoders (work_state, run_mode, device_type)

**Used by:** `packages: sungrow: !include packages/sungrow_control.yaml` in configuration.yaml

**Why packages?** Combines multiple component types (input_select, automation, template) in one file.

#### `/config/sungrow_customize.yaml`
**Purpose:** Entity customization for user-friendly names and icons

**Contains:**
- Friendly names for all sensors
- Icons for binary sensors
- Organized by category (Power, MPPT, Energy, State, Control)

**Used by:** `customize: !include sungrow_customize.yaml` in configuration.yaml

**Benefits:**
- Dashboard shows clean names
- Consistent iconography
- Easy to identify entities

### Documentation Files (`/docs`)

#### `/docs/INSTALLATION.md`
**Purpose:** Step-by-step installation guide

**Sections:**
1. Prerequisites verification
2. Finding inverter IP
3. Testing modbus connection
4. Cloning repository
5. Configuring secrets
6. Customizing sensor names
7. Updating configuration.yaml
8. Validation and restart
9. Verification checklist
10. Testing manual control
11. Setting thresholds
12. Monitoring first day

**Target audience:** New users setting up from scratch

#### `/docs/INTEGRATIONS.md`
**Purpose:** Details on required Home Assistant integrations

**Covers:**
1. **Teslemetry Integration:**
   - Installation steps
   - Required sensors
   - Alternative battery integrations
   - Sensor name mapping

2. **Amber Electric Integration:**
   - Setup instructions
   - Finding feed-in price sensor
   - Alternative pricing sources (Octopus, Tibber)
   - Manual pricing option

3. **Modbus Integration:**
   - Configuration details
   - Requirements

**Target audience:** Users needing to configure dependencies

#### `/docs/MODBUS_REGISTERS.md`
**Purpose:** Complete register reference for SG5.0RS

**Sections:**
1. Register addressing explanation
2. Control registers (holding - read/write)
3. Monitoring registers (input - read-only)
4. Register tables by category:
   - Basic information
   - Energy production
   - DC power (MPPT)
   - AC power (grid output)
   - Grid meter (import/export)
   - Environmental
5. Configuration examples
6. Calculated sensor formulas
7. Scan interval recommendations
8. Troubleshooting

**Target audience:** Advanced users, developers, troubleshooters

### Example Files (`/examples`)

#### `/examples/dashboard.yaml`
**Purpose:** Complete Lovelace dashboard configuration

**Includes 4 views:**
1. **Control** - Manual controls, settings, current conditions
2. **Monitoring** - Power flow, MPPT strings, energy, status
3. **Pricing** - Price history, curtailment logic, economic impact
4. **Automations** - Status, traces, quick actions

**Card types used:**
- Entities cards
- Gauge cards
- Markdown cards
- History graphs
- Button cards

**Customization:** Fully customizable, copy sections as needed

#### `/examples/secrets.yaml.example`
**Purpose:** Template for creating secrets.yaml

**Contains:**
- Commented example values
- Instructions for finding inverter IP
- Examples for different network setups
- Security warnings

**Usage:** Copy to `/config/secrets.yaml` and edit with your values

## Installation Files Mapping

When installing, files map to Home Assistant as follows:

| Repository File | Home Assistant Location |
|----------------|------------------------|
| `config/integrations/modbus_sungrow.yaml` | `/config/integrations/modbus_sungrow.yaml` |
| `config/packages/sungrow_control.yaml` | `/config/packages/sungrow_control.yaml` |
| `config/sungrow_customize.yaml` | `/config/sungrow_customize.yaml` |
| `examples/secrets.yaml.example` | `/config/secrets.yaml` (edit first!) |
| `examples/dashboard.yaml` | (Copy to Lovelace dashboard) |

## Configuration.yaml Integration

Add these lines to `/config/configuration.yaml`:

```yaml
homeassistant:
  packages:
    sungrow: !include packages/sungrow_control.yaml
  customize: !include sungrow_customize.yaml

modbus: !include integrations/modbus_sungrow.yaml
```

## What Gets Created in Home Assistant

### Entities Created

**Modbus Sensors (~20):**
- `sensor.inverter_*` - All inverter data
- Control registers, production, MPPT, grid, etc.

**Template Sensors (8):**
- `binary_sensor.pv_generating`
- `binary_sensor.exporting_power`
- `binary_sensor.importing_power`
- `sensor.inverter_mppt1_power`
- `sensor.inverter_mppt2_power`
- `sensor.inverter_load_power`
- `sensor.inverter_work_state`
- `sensor.inverter_run_mode`
- `sensor.inverter_device_type`
- `sensor.inverter_power_limit_enabled`

**Control Inputs (5):**
- `input_select.set_sg_inverter_run_mode`
- `input_boolean.sungrow_curtailment_enabled`
- `input_number.sungrow_curtailment_price_threshold_low`
- `input_number.sungrow_restart_price_threshold_high`
- `input_number.sungrow_curtailment_powerwall_threshold`

**Automations (4):**
- "Sungrow: Write run mode to modbus"
- "Sungrow: Shutdown when Powerwall full and low/negative feed-in price"
- "Sungrow: Restart when conditions favorable"
- "Sungrow: Restart at sunset (safety)"

**Total:** ~37 entities created

## File Sizes

Approximate file sizes:

| File | Size | Description |
|------|------|-------------|
| modbus_sungrow.yaml | ~8 KB | Sensor definitions |
| sungrow_control.yaml | ~12 KB | Automations + templates |
| sungrow_customize.yaml | ~3 KB | Friendly names |
| dashboard.yaml | ~6 KB | Dashboard config |
| README.md | ~15 KB | Main docs |
| INSTALLATION.md | ~10 KB | Setup guide |
| INTEGRATIONS.md | ~8 KB | Integration guide |
| MODBUS_REGISTERS.md | ~12 KB | Register reference |

**Total repository:** ~75 KB (excluding .git)

## Dependencies

**Required integrations:**
- Modbus (built-in)
- Teslemetry
- Amber Electric (or alternative)

**Optional integrations:**
- EMHASS (for advanced optimization)
- Notify (for push notifications)

## Version Control

**What to commit:**
- All config files
- All documentation
- All examples
- .gitignore
- LICENSE

**What NOT to commit:**
- secrets.yaml (sensitive data)
- .db files (database)
- .log files (logs)
- Test files

**GitHub Actions ready:** Directory `.github/` prepared for CI/CD workflows

## Maintenance

**Regular updates:**
- README.md - Features, compatibility
- INSTALLATION.md - New steps, troubleshooting
- MODBUS_REGISTERS.md - New registers discovered
- dashboard.yaml - UI improvements

**Version tracking:**
- Use GitHub releases for versions
- Tag format: v1.0.0
- Update CHANGELOG.md for each release

## Support Files

**Where to get help:**
- README.md - Quick reference
- INSTALLATION.md - Setup issues
- INTEGRATIONS.md - Integration problems
- MODBUS_REGISTERS.md - Register questions
- CONTRIBUTING.md - How to contribute
- GitHub Issues - Bug reports
- GitHub Discussions - Questions

---

**Repository maintained by the Home Assistant community**
**Star the repo if it helps you! ⭐**
