# Gas Management

Two chips that work together: `GasManager.ic10` monitors gas pressures and temperatures and drives display devices, and `gasTemperatureManager.ic10` uses that data to control a cooling circuit for each gas line.

---

## GasManager.ic10

Monitors five gas lines and updates LEDs and gauges for each one every tick.

### Gases monitored

Devices are addressed by batch name using the `*` wildcard suffix:

| Slot | Gas name    |
|------|-------------|
| 0    | `Oxygen*`   |
| 1    | `Nitrogen*` |
| 2    | `CO2*`      |
| 3    | `Volatiles*`|
| 4    | `Waste*`    |

Any analyzer, LED, or gauge whose name starts with one of these strings will be matched.

### Device types used (batch by name)

| Type     | Hash        | Addressed by          |
|----------|-------------|-----------------------|
| Analyzer | `435685051` | Gas name (e.g. `Oxygen*`) |
| LED      | `-2041948072` | Gas name            |
| Gauge    | `-1252835556` | Gas name            |

### What it writes

For each gas:

- **LED** — Mode set to `4`; `Setting` set to the gas temperature in **°C** (converted from Kelvin). If pressure is zero, temperature is forced to 0 to avoid misleading readings.
- **Gauge** — Color set to `6`; `Setting` set to the pressure ratio (`pressure / 50000 kPa`), where 50 000 kPa (50 MPa) is treated as full scale.

### Memory layout

The chip stores gas name hashes in its own memory for use by other chips:

| Slot | Value               |
|------|---------------------|
| 0    | `HASH("Oxygen*")`   |
| 1    | `HASH("Nitrogen*")` |
| 2    | `HASH("CO2*")`      |
| 3    | `HASH("Volatiles*")`|
| 4    | `HASH("Waste*")`    |

`gasTemperatureManager.ic10` reads these slots directly so both chips always agree on gas names.

---

## gasTemperatureManager.ic10

Reads gas hashes from a GasManager chip and controls a valve and pump per gas line to maintain a target temperature.

### Device connections

| Pin | Device                        |
|-----|-------------------------------|
| d0  | GasManager chip (for gas hashes) |

Valves and pumps are addressed by batch name matching the same gas name hashes.

### Device types used (batch by name)

| Type    | Hash          |
|---------|---------------|
| Analyzer| `435685051`   |
| Valve   | `-1280984102` |
| Pump    | `-321403609`  |

### Configuration

| Define        | Description                          | Default |
|---------------|--------------------------------------|---------|
| `targetTemp`  | Target temperature in Kelvin         | 303 K (≈ 30 °C) |

### Behaviour per gas

The chip reads the current temperature from the gas analyzer and compares it to `targetTemp`:

| Condition             | Valve | Pump |
|-----------------------|-------|------|
| Temperature > target  | Open  | Off  |
| Temperature ≤ target  | Closed| On   |

**Valve open** — connects the gas line to a direct heat exchanger, cooling the pipe network.

**Pump on** — draws gas away from the cold side of the valve to prevent freezing when the valve is closed.

### Setup notes

- `gasTemperatureManager` must be able to see the GasManager chip on `d0`.
- Valve and pump devices must be named to match the gas name patterns (e.g. a valve named `Oxygen Valve` will match `Oxygen*`).
- The number of gases iterated is hardcoded to `5` — change the `blt index 5 gasLoop` line if you add or remove gas lines.
