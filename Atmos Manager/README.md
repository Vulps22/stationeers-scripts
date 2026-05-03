# Atmos Manager

A two-chip system for centrally controlling atmospheric vents across multiple rooms. One manager chip reads logic switches for all rooms and broadcasts their states; each room has its own controller chip that receives and acts on that broadcast.

## How it works

The manager packs the pressure and filtration switch states for all rooms into a single integer and writes it to a transmitter's `Setting` property every tick. Each controller reads that integer, extracts the two bits assigned to its room, and drives its local vents accordingly.

### Bitpacking layout

Each room occupies 2 bits in the packed integer:

| Bit position | Meaning              |
|-------------|----------------------|
| 0 (LSB)     | Pressure vent enable |
| 1           | Filtration vent enable |

Rooms are assigned in pairs starting from bit 0:

| Room        | roomId | Bits |
|-------------|--------|------|
| Atrium      | 0      | 1:0  |
| Machine     | 1      | 3:2  |
| Filtration  | 2      | 5:4  |
| Hydroponics | 3      | 7:6  |
| Furnace     | 4      | 9:8  |

---

## manager.ic10

Reads named logic switches for each room and writes the packed result to a transmitter.

### Device connections

| Pin | Device      |
|-----|-------------|
| d0  | Transmitter |

### Switch naming convention

For each room, two switches must exist on the logic network with these exact names:

| Room        | Pressure switch | Filtration switch |
|-------------|-----------------|-------------------|
| Atrium      | `AtriumP`       | `AtriumF`         |
| Machine     | `MachineP`      | `MachineF`        |
| Filtration  | `FiltrationP`   | `FiltrationF`     |
| Hydroponics | `HydroponicsP`  | `HydroponicsF`    |
| Furnace     | `FurnaceP`      | `FurnaceF`        |

The manager uses batch-by-name reads (`lbn`) against device hash `1773255945`.

---

## Controller.ic10

Per-room chip. Reads the packed integer from the transmitter, extracts its two bits, and controls a pressure vent and a filtration vent.

### Device connections

| Pin | Device           |
|-----|------------------|
| d0  | Transmitter      |
| d1  | Pressure vent    |
| d2  | Filtration vent  |
| d3  | Pressure sensor  |

### Configuration

Two values must be set in the script before flashing:

| Define          | Description                                                  |
|-----------------|--------------------------------------------------------------|
| `roomId`        | Room index matching the table above (0 = Atrium, 4 = Furnace) |
| `targetPressure`| Target pressure in kPa (default: 75)                        |

### Behaviour

- **Filtration vent**: turned on when the room's filtration bit is set, off otherwise.
- **Pressure vent**: turned on when the room's pressure bit is set **and** current pressure is below `targetPressure`. Turned off as soon as target pressure is reached or the bit is cleared.

---

## Setup summary

1. Place one manager chip + transmitter accessible to the logic switch network.
2. Name your switches using the convention above.
3. For each room, place a controller chip and connect transmitter, pressure vent, filtration vent, and pressure sensor.
4. Set `roomId` in each controller chip to match its room.
5. Adjust `targetPressure` per room if needed.
