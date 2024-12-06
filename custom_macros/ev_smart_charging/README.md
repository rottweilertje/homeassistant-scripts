# Calculate Charging Speed

## The Problem
There are solutions to charging your car on the cheapest hours of the day when you have a dynamic energy contract.

* [Homeassistant](https://homeassistant.io)
* With Integration [EV Smart Charging](https://github.com/jonasbkarlsson/ev_smart_charging)

But although you can add two entities (cars) to the integration, when they share a 11kWh grid, if they both start charging, they will charge only at half the speed. Hence some discussion on the repo in issue [#331](https://github.com/jonasbkarlsson/ev_smart_charging/issues/331) over on the EV Smart Charging repo.

## A Solution
A macro that takes both cars into consideration.
- Checks when they are planned to be allowed to charge by EV Smart Charging. 
- Compare the charging hours in order to calculate how many hours overlap.
- Calculate a new value for `ev_smart_charging_charging_speed` for each car (in % per hour) proportionally to the "double booked" hours.

Combined with several automations and some helpers it is now possible to let Homeassistant look for the cheapest hours needed to charge both cars, while still keeping into account the scheduled completion time for each car separately.

### Helpers
| Entity                                      | Type         | Description                                      |
|---------------------------------------------|--------------|--------------------------------------------------|
| car_1_base_charge_percentage_per_hour       | input_number | Percentage of battery "car 1" charges in an hour |
| car_2_base_charge_percentage_per_hour       | input_number | Percentage of battery "car 2" charges in an hour |
| car_1_calculated_charge_percentage_per_hour | input_number | New calculated charge percentage for "car 1"     |
| car_2_calculated_charge_percentage_per_hour | input_number | New calculated charge percentage for "car 2"     |

### Automations

1. EV Smart Charging Calculate Charge Speed
An automation that is triggered every time the `ev_smart_charging_charging` for each of the cars is updated.
It calculates a new value for each car and stores it in the `car_x_calculated_charge_percentage_per_hour` helper.

2. EV Smart Charging Update Charging Speed
An automation that is triggered every time the `car_x_calculated_charge_percentage_per_hour` is updated.
It checks if the `ev_smart_charging_charging_speed` and `car_x_calculated_charge_percentage_per_hour` are the same, if stores the calculated value as the new `ev_smart_charging_charging_speed`.

3. Reset CarX Calculated Charge Percentage
Two automations, one for each car, that triggers when the car is Disconnected from the charging station.
It resets the `ev_smart_charging_charging_speed` to the corresponding `car_x_base_charge_percentage_per_hour`, ready for when the car connects again and we can start the whole anew.