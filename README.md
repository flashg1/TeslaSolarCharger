# Support for TeslaSolarCharger is ending ...
Please use [evSolarCharger](https://github.com/flashg1/evSolarCharger) instead.  Please read [this](https://github.com/flashg1/evSolarCharger/wiki/Installation#how-to-migrate-from-teslasolarcharger-to-evsolarcharger) if you need to migrate from TeslaSolarCharger.


# Tesla Solar Charger
Home Assistant Blueprint to charge Tesla car using surplus solar electricity and weather forecast.

###############################################################################
# Disclaimer:
#
# Even though this automation has been created with care, the author cannot be responsible for any damage caused by this automation.  Use at your own risk.
#
###############################################################################

![Screenshot_20230702-094232_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/58d1df89-905b-422c-8542-0081b9fa342f)

![Screenshot_20230630-135925_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/2f04b1e2-b56d-493c-977f-82d5dd04cbe5)


Features
========

-   Charge from excess solar adjusting Tesla car charging current according to feedback loop value "Grid Power Net".  The "Grid Power Net" sensor expresses negative power in Watts when exporting to grid, and positive power when consuming from grid.
-   Support multi-day solar charging using sun elevation triggers to start and stop. (Sun elevation triggers should be more favourable for countries in the northern hemisphere.)
-   Compatible with off-peak night time charging.
-   Configurable daily car charge limit for 7 days.  Default is to use the Tesla app charge limit.
-   Automatically adjust to the highest charge limit set within a rainy forecast period.  The highest charge limit is selected from the 7 days charge limit settings that are within the forecast period taking into account the charge limit on bad weather setting.  The objective is to charge more before a rainy period.  Default disabled.
-   Might be possible to prolong car battery life by setting daily charge limit to 70%, and only charge more before a rainy period by enabling option to adjust daily car charge limit based on weather.
-   Allow top up from grid if there is not enough solar electricity.  Need to toggle on charge from grid and set power offset to draw power from grid.
-   Support charging multiple Tesla cars at the same time based on power allocation weighting for each car.


My setup
========

-	Home Assistant, https://www.home-assistant.io/
-	Enphase Envoy Integration configured for 30 seconds update interval, https://www.home-assistant.io/integrations/enphase_envoy
-	Tesla Custom Integration v3.20.4, https://github.com/alandtse/tesla
-	Tesla UMC charger, 230V, max 15A.
-	Tesla Model 3.


Installation
============

-	Set up "Grid Power Net" sensor in Home Assistant (HA) config, eg.

/config/configuration.yaml
```
template:

    # For Enphase, grid_power_net is an integer in watts. Positive value means importing power from grid. Negative value means exporting power to grid.
    # For other inverter brands, adjust the formula to conform with above requirement according to your setup.
    - sensor:
        name: Grid Power Net
        state_class: measurement
        icon: mdi:transmission-tower
        unit_of_measurement: W
        device_class: power
        state: >
            {{ states('sensor.envoy_[YourEnvoyId]_current_power_consumption')|int - states('sensor.envoy_[YourEnvoyId]_current_power_production')|int }}
```

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fflashg1%2FTeslaSolarCharger%2Fblob%2Fmain%2FTesla_solar_charger_automation.yaml)

-	Import the Blueprint automatically by clicking above, or manually copy the Blueprint file to following location and reload HA config,
\\HOMEASSISTANT\config\blueprints\automation\flashg1\Tesla_solar_charger_automation.yaml

-	Create 3 helper booleans, eg.
Settings > Devices & Services > Helpers > Create Helper > Toggle
1.	Tesla Model3 set daily car charge limit
2.	Telsa Model3 charge from grid
3.	Tesla Model3 stop charging

-	Create 1 helper number or template sensor for power offset (required when charging from grid), eg.
Settings > Devices & Services > Helpers > Create Helper > Number or Template a sensor
1.	Tesla Model3 power offset

-	Config the Blueprint automation specifying charger voltage, maximum current and helper entities created above, ie.
Settings > Automations & Scenes > Blueprints > Tesla solar charger automation


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
To charge from excess solar, just plug in the charger.  The initial charge current is 5A.  After about 1 minute it will adjust the current according to amount of excess power exported to grid.

Option 2
--------
To charge from grid and solar, toggle on charge from grid and set power offset to draw power from grid.


Notes
=====

Please also check out the [wiki](https://github.com/flashg1/TeslaSolarCharger/wiki) pages.

Automation cannot be triggered
------------------------------
The Tesla triggers and conditions are slow to update unless car is polled often.  Polling too often can drain the car battery.  So might have to wait a minute or two for the conditions to refresh and the triggers to work.  Please see below for possible work-arounds.

Work-arounds:
1. Run the automation manually by selecting the automation and then select "Run Actions".
2. Turn polling off, then on.
3. Press the "Force data update" button before and after plugging in the charger.

Daily car charge limit settings
-------------------------------
- If set daily car charge limit is toggled off, charge limit will be set according to the Tesla app.
- If set daily car charge limit is toggled on and charge car based on weather is disabled, charge limit will be set according to the limit configured for the day.
- If set daily car charge limit is toggled on and charge car based on weather is enabled, charge limit will be adjusted to the highest limit set within the rainy forecast period taking into account the car charge limit on bad weather setting.
- If charge car based on weather is enabled, daily car charge limit and weather provider settings must be configured.

Special note for 3-phase chargers
---------------------------------
Please see [discussion](https://github.com/flashg1/TeslaSolarCharger/issues/18) on voltage to set for charger with 3-phase power.

Charge mutiple Tesla cars at the same time based on power allocation weighting for each car
-------------------------------------------------------------------------------------------
Note: This is theoretical only since I don't have 2 Tesla cars to test this, but happy for any feedback.  To ensure power is allocated according to weighting, the "Grid power net" update cycle should be the same as the script looping cycle, ie. 1 minute.

- Create power allocation weighting for each car.  For example, to create for car1,
```
Settings > Devices & services > Helpers > Create helper > Number >
Name: Car1 power allocation weight
Minimum value: 1
Maximum value: 10
```
- Create power allocation sensor for each car.  For example, to create power allocation sensor for car1 assuming we have car1 and car2,
```
Settings > Devices & services > Helpers > Create helper > Template > Template a sensor >
Name: Car1 power allocation
State template: {{ states('sensor.grid_power_net')|int * states('input_number.car1_power_allocation_weight')|int / (state_attr('automation.car1_solar_charger_automation', 'current') * states('input_number.car1_power_allocation_weight')|int + state_attr('automation.car2_solar_charger_automation', 'current') * states('input_number.car2_power_allocation_weight')|int) }}
Unit of measurement: W
Device class: Power
State class: Measurement
```
- Use the power allocation sensors defined above as input to "Grid power net" in each car's Blueprint.


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
# Watch this for car charge limit change
house_extra_entity: number.charge_limit
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
  - entity: input_boolean.[YourTeslaName]_set_daily_car_charge_limit
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
