# Tesla Solar Charger
Home Assistant smart charger Blueprint to charge Tesla car using surplus solar electricity.

###############################################################################
# Disclaimer:
#
# Even though this automation has been created with care, the author cannot be responsible for any damage caused by this automation.  Use at your own risk.
#
###############################################################################
# Note:
# - Credit goes to Alphaemef for the original solar charger Blueprint.
#   https://github.com/Alphaemef/SolarBalance-Suite
###############################################################################

![Screenshot_20230702-094232_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/58d1df89-905b-422c-8542-0081b9fa342f)

![Screenshot_20230630-135925_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/2f04b1e2-b56d-493c-977f-82d5dd04cbe5)


Installation
============

-	Set up "Grid Power Net" sensor in Home Assistant (HA) config, eg.

/config/configuration.yaml
```
template:

    # For Enphase, grid_power_net is an integer in watts. Positive value means importing power from grid. Negative value means exporting power to grid.
    - sensor:
        name: Grid Power Net
        state_class: measurement
        icon: mdi:transmission-tower
        unit_of_measurement: W
        device_class: power
        state: >
            {{ states('sensor.envoy_[YourEnvoyId]_current_power_consumption')|int - states('sensor.envoy_[YourEnvoyId]_current_power_production')|int }}
```

-	Copy the Blueprint file to,
\\HOMEASSISTANT\config\blueprints\automation\flashg\Tesla_solar_charger_automation.yaml

-	Restart HA.

-	Create 2 helper booleans, eg.
Settings > Devices & Services > Helpers > Create Helper > Toggle
1.	Telsa Model3 charge from grid
2.	Tesla Model3 stop charging

-	Config the Blueprint automation specifying charger voltage, maximum current and helper booleans created above, ie.
Settings > Automations & Scenes > Blueprints > Tesla solar charger automation


My setup
========

-	Home Assistant, https://www.home-assistant.io/
-	Enphase Envoy Integration configured for 30 seconds update interval, https://github.com/briancmpbll/home_assistant_custom_envoy
-	Tesla Custom Integration v3.20.4, https://github.com/alandtse/tesla
-	Tesla UMC charger, 230V, max 15A.
-	Tesla Model 3.


Features
========

-	Charge from excess solar adjusting Tesla car charging current according to feedback loop value "Grid Power Net".  The "Grid Power Net" sensor expresses negative power in Watts when exporting to grid, and positive power when consuming from grid.
-   Support multi-day solar charging using sunrise trigger to start and sunset trigger to stop.
-   Compatible with off-peak night time charging.


How to use
==========

-	Set your charging limit in app or car.
-	Connect charger to car.  Normal charging at constant current should begin immediately if schedule charging is disabled.  After a little while, the script will take over and manage the charging current during daylight hours.  Please see work-arounds below if automation cannot be triggered.
-	There are 2 options on how to charge the car (see below).
-	The script will stop if charger is turned off manually or automatically by car when reaching charge limit.
-	To abort charging, turn on "Tesla Model3 stop charging".  The script will take about a minute to terminate if using default values.

2 options on how to charge the car:

Option 1
--------
To charge from excess solar, just plug in the charger.  The initial charge current is 6A.  After about 1 minute it will adjust the current according to amount of excess power exported to grid.

Option 2
--------
To charge from grid, set your desired charging current and turn on "Telsa Model3 charge from grid".


Notes
=====

Automation cannot be triggered
------------------------------
The Tesla triggers and conditions are slow to update unless car is polled often.  Polling too often can drain the car battery.  So might have to wait a minute or two for the conditions to refresh and the triggers to work.  Please see below for possible work-arounds.

Work-arounds:
1. Run the automation manually by selecting the automation and then select "Run Actions".
2. Turn polling off, then on.
3. Press the "Force data update" button before and after plugging in the charger.


GUI display examples
====================

Dashboard Tesla power card
--------------------------
https://github.com/reptilex/tesla-style-solar-power-card

```
type: custom:tesla-style-solar-power-card
name: Power Usage
show_w_not_kw: 1

# 3 flows between bubbles
grid_to_house_entity: sensor.grid_power_import
generation_to_grid_entity: sensor.grid_power_export
generation_to_house_entity: sensor.solar_power_consumption

# optional appliances with consumption and extra values
appliance1_consumption_entity: sensor.charger_power
appliance1_extra_entity: sensor.battery

# optional 3 main bubble icons for clickable entities
grid_entity: sensor.grid_power_net
house_entity: sensor.envoy_[YourEnvoyId]_current_power_consumption
generation_entity: sensor.solar_power_production

```

Dashboard Tesla solar charger control
-------------------------------------
```
type: entities
entities:
  - entity: automation.[YourTeslaName]_solar_charger_automation
  - type: attribute
    entity: automation.[YourTeslaName]_solar_charger_automation
    attribute: current
    name: Running instance count
  - type: attribute
    entity: automation.[YourTeslaName]_solar_charger_automation
    attribute: last_triggered
    name: Last triggered
  - entity: input_boolean.[YourTeslaName]_charge_from_grid
  - entity: input_boolean.[YourTeslaName]_stop_charging
  - entity: button.wake_up
  - entity: button.force_data_update
  - entity: device_tracker.location_tracker
  - entity: binary_sensor.charger
  - entity: binary_sensor.charging
  - entity: number.charging_amps
  - entity: sensor.range
  - entity: sensor.battery
  - entity: number.charge_limit
  - entity: sensor.time_charge_complete
  - entity: lock.charge_port_latch
```
