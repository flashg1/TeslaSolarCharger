# TeslaSolarCharger
Home Assistant smart charger Blueprint to charge Tesla car using surplus solar electricity.

###############################################################################
# Disclaimer:
#
# Even though this automation has been created with care, the author cannot be responsible for any damages caused by this automation.  Use at your own risk.
#
###############################################################################
# Note:
# - Credit goes to Alphaemef for the original solar charger Blueprint.
#   https://gist.github.com/Alphaemef/70f203cc6a5acb75d987ae88d9be2edf
###############################################################################

![Screenshot_20230630-134732_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/fe86679c-5ca3-474c-80e9-d623a7e83c6e)

![Screenshot_20230630-135925_Home Assistant](https://github.com/flashg1/TeslaSolarCharger/assets/122323972/2f04b1e2-b56d-493c-977f-82d5dd04cbe5)


Installation
============

-	Set up "Grid Power Net" in HA config, eg.

/config/configuration.yaml

template:

    # For Enphase, grid_power_net is an integer in watts. Positive value means importing power from grid. Negative value means exporting power to grid.
    - sensor:
        name: Grid Power Net
        state_class: measurement
        icon: mdi:transmission-tower
        unit_of_measurement: W
        device_class: power
        state: >
            {{ states('sensor.envoy_current_power_consumption')|int - states('sensor.envoy_current_power_production')|int }}


-	Copy the 2 Blueprint files to,
\\HOMEASSISTANT\config\blueprints\automation\flashg\Tesla_solar_charger_automation.yaml
\\HOMEASSISTANT\config\blueprints\script\homeassistant\Tesla_solar_charger_script.yaml

-	Restart HA.

-	Create 2 helper booleans, eg.
Settings > Devices & Services > Helpers > Create Helper > Toggle
1.	Telsa Model3 charge from grid and solar
2.	Tesla Model3 stop charging

-	Config the Blueprint script specifying helper booleans created above, eg.
Settings > Automations & Scenes > Blueprints > Tesla solar charger script

-	Config the Blueprint automation specifying above script, eg.
Settings > Automations & Scenes > Blueprints > Tesla solar charger automation


My setup
========

-	Enphase Envoy Integration, https://github.com/posixx/home_assistant_custom_envoy
-	Tesla Custom Integration, https://github.com/alandtse/tesla
-	Tesla UMC charger, max 15A.
-	Tesla Model 3.


Function
========

-	Charge from excess solar adjusting Tesla car charging current according to feedback loop value "Grid Power Net".  The "Grid Power Net" sensor expresses negative power in Watts when exporting to grid, and positive power when consuming from grid.


How to use
==========

-	Set your charging limit in app or car.
-	Connect charger to car.
-	There are 2 options on how to charge the car (see below).
-	The script will stop automatically 1% before the charging limit is reached.
-	To abort charging, turn on "Tesla Model3 stop charging".  Will take about a minute to terminate the charging script if using default values.

2 options on how to charge the car:

Option 1
--------
To charge from excess solar, just plug in the charger.  The initial charge current is 6A.  After about 1 minute it will adjust the current according to amount of excess power exported to grid.

Option 2
--------
To charge from grid, set your desired charging current and turn on "Telsa Model3 charge from grid and solar".
