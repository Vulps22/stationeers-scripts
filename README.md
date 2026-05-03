# Stationeers Scripts

A collection of ic10 scripts for [Stationeers](https://store.steampowered.com/app/544550/Stationeers/).

## Contents

### [Atmos Manager](./Atmos%20Manager/)

A two-chip system for centralised atmospheric control across multiple rooms. A single manager chip reads named logic switches for up to five rooms and broadcasts packed pressure/filtration states to all controller chips simultaneously. Each controller chip manages its room's pressure vent and filtration vent independently.

- `manager.ic10` — reads switches, packs state, broadcasts via transmitter
- `Controller.ic10` — per-room chip; unpacks state and controls local vents

### [gasManagement](./gasManagement/)

Two chips for monitoring and temperature-controlling five gas lines (Oxygen, Nitrogen, CO2, Volatiles, Waste).

- `GasManager.ic10` — reads gas analyzers and drives LEDs and gauges; also exposes gas name hashes in memory for other chips to consume
- `gasTemperatureManager.ic10` — reads hashes from GasManager and controls per-gas valves and pumps to maintain a target temperature via a direct heat exchanger

### CentrifugeManager.ic10

Manages centrifuges by monitoring their reagent counts and automatically opening or closing each one based on fill level. Designed to be scalable — it ships configured for two centrifuges but supports any number.

- Opens a centrifuge (sets `Open 1`) when reagents reach 300 or more, stopping it and making contents accessible for unloading.
- Closes a centrifuge (sets `Open 0`) when reagents drop to 1 or below, returning it to the operating/processing state.
- Centrifuges must be named `Centrifuge 1`, `Centrifuge 2`, etc. on the logic network.

**To add more centrifuges:** add a `put db N HASH("Centrifuge N")` line at the top of the script (following the existing pattern), and increase the count on line 11 (`bgtal index 2 reset`) to match the new total.

## License

Do whatever you want with it all.
