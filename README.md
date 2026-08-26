# Goe_ha
Panel for Wallbox Goe witch PV
Instalation PV 2x450W and HMS-800W and 2x340W and HM-800 evective get energy, about 1.5KW/h and 2 

## Panel
![Panel Home Assistant](docs/images/panel.png)

# Energy Measurement and EV Charging

The system consists of two independent Tuya-based measurement installations located in the house.

One Tuya plug measures the energy exported to the grid. Because the plug measures power in only one direction, exported energy is reported as a positive +kW value. It cannot distinguish the actual direction of current flow.

The second measurement point monitors the energy consumption of the individual household loads.

By combining both measurements and applying a correction factor, the system can calculate the actual PV production and determine the current energy balance:

- Household consumption
- Energy exported to the grid
- Actual PV production
- Available solar surplus

The calculated solar surplus is then used to automatically control the EV charging installation.

---

## Automatic EV Charging

The system uses different charging thresholds depending on the current electricity tariff.

### Expensive Electricity

When electricity is expensive:

- Start charging: solar surplus ≥ 800 W
- Stop charging: solar surplus < 700 W

The 100 W hysteresis prevents the charger from repeatedly switching on and off when PV production fluctuates around the threshold.

### Cheap Electricity

During the low-tariff period:

- Start charging: solar surplus ≥ 200 W
- Stop charging: solar surplus < 150 W

The lower threshold allows the system to use cheap grid electricity while still prioritizing available PV production.

### Electricity Tariff

The system supports a dual-tariff electricity plan:

| Period | Approx. price | Charging threshold |
| --- | ---: | --- |
| 13:00–15:00 | 0.60 PLN/kWh | Start ≥ 200 W / Stop < 150 W |
| 22:00–06:00 | 0.60 PLN/kWh | Start ≥ 200 W / Stop < 150 W |
| Weekends & public holidays | 0.60 PLN/kWh | Start ≥ 200 W / Stop < 150 W |
| Other hours | 1.30 PLN/kWh | Start ≥ 800 W / Stop < 700 W |

Charging can also be manually forced, overriding the automatic charging conditions.

---

## Charging Logic
                    Solar surplus
                         │
                         ▼
                 Determine tariff
                         │
              ┌──────────┴──────────┐
              │                     │
         Expensive tariff       Cheap tariff
              │                     │
        Start ≥ 800 W          Start ≥ 200 W
        Stop  < 700 W          Stop  < 150 W
              │                     │
              └──────────┬──────────┘
                         ▼
                   EV charging
---

# TODO

## EV Cable Lock

- [ ] Add support for controlling the EV charging cable lock.
- [ ] Detect whether the cable is correctly locked.
- [ ] Prevent charging when the required cable-lock state is not detected.
- [ ] Detect cable unlock/disconnection and safely stop charging.

## Charging Current Control

- [ ] Improve charging-current regulation.
- [ ] Current regulation currently works in one direction only.
- [ ] Add better handling of charger state and current limits.
- [ ] Automatically adjust charging current according to available PV surplus.

## Initial Charging State

- [ ] Define the initial charging state when a vehicle is connected.
- [ ] Support automatic charging immediately after connection.
- [ ] Support waiting for configured charging conditions.
- [ ] Support waiting for RFID authorization before starting charging.
- [ ] Restore the previous charging state when appropriate.

## RFID Authorization

- [ ] Add RFID reader support.
- [ ] Detect whether the connected vehicle/user is authorized.
- [ ] Allow charging only after successful RFID authorization.
- [ ] Support different charging rules for different users.
- [ ] Associate an RFID identifier with a vehicle/user.

This is particularly useful when the charger is made available to other users through PlugHome or another paid charging service.

The RFID workflow could be:

Vehicle connected
       |
       v
Detect cable / vehicle state
       |
       v
Wait for RFID
       |
       v
Identify user / vehicle
       |
   +---+---+
   |       |
Authorized Unknown
   |       |
   v       v
Charging  Reject

---

## Charging Session Accounting

- [ ] Record every charging session.
- [ ] Record start and end time.
- [ ] Record total energy consumed.
- [ ] Record charging power/current.
- [ ] Separate energy consumed during cheap and expensive tariff periods.
- [ ] Calculate the actual electricity cost of each session.
- [ ] Associate each session with an RFID user/vehicle.

Example:

Charging session
+-- 4.2 kWh x 0.60 PLN = 2.52 PLN
+-- 1.8 kWh x 1.30 PLN = 2.34 PLN
+-- Electricity cost = 4.86 PLN
---

## Paid Charging / PlugHome

- [ ] Add support for paid charging.
- [ ] Calculate the actual electricity cost based on the tariff active during the charging session.
- [ ] Add a configurable service fee / commission.
- [ ] Calculate the final amount charged to the user.
- [ ] Associate payments with the RFID user/vehicle.
- [ ] Store charging history for billing purposes.
- [ ] Provide a charging-session summary.

The final price should be based on:

Energy consumed
       |
       v
Determine tariff for each period
       |
       v
Calculate actual electricity cost
       |
       v
Add service fee / commission
       |
       v
Final amount charged to user

Example:

Electricity cost    4.86 PLN
Service fee         2.00 PLN
---------------------------
Final price         6.86 PLN

This approach allows the system to calculate the actual cost of each charging session, rather than using a fixed price per kWh.

---

## Future Improvements

- [ ] Improve PV surplus calculation.
- [ ] Add configurable start/stop thresholds.
- [ ] Make electricity prices configurable.
- [ ] Configure tariff schedules and public holidays.
- [ ] Add charging statistics.
- [ ] Add daily/monthly energy reports.
- [ ] Add PV production vs. household consumption statistics.
- [ ] Add total savings generated by solar charging.
- [ ] Add total savings generated by low-tariff charging.
- [ ] Add a complete charging history.
- [ ] Add user/vehicle statistics for shared charging.

---

## Project Goal

The goal of the project is to create an automated EV charging system integrated with Home Assistant, capable of combining household energy consumption, PV production and electricity tariffs.

The system should automatically decide when charging should start, when it should stop and how much current should be used, while allowing manual control when required.

For shared charging, RFID authorization and tariff-aware billing can additionally provide a foundation for operating the charger as a paid private charging point.
