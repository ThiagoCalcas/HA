Setting up the Fortrezz MimoLite to automate garage door

Remove the jumper and connect the wires as per manual.
With the jumper on the Mimo will have a open/close key scenario, which we don't want. Having it staying "open" for a garage door means keep pressing the button and keeping the circuit closed.
By removing the jumper we set it to close the circuit as a momentary action, like pressing a button and releasing it.

![image](https://user-images.githubusercontent.com/15160444/177218522-517cf5fd-6430-4f7e-9861-3bd60fb06cc5.png)

I added a generic magnetic door sensor using thin electric wires in a red/black pair. 

![image](https://user-images.githubusercontent.com/15160444/177218595-d32c08c1-a1da-4d34-b9df-bbcdc4255964.png)

Add the MimoLite to HA using whichever interface you like. I used Z-Wave JS.
This version on Fortrezz Mimo (the lite one) doesn't have a "on change" update for the garage door sensor, so we can't have real time information. It is still good enough for what I want to do.

Now that we have it on Ha, we can rename it and have something like this:

![image](https://user-images.githubusercontent.com/15160444/177218837-0ebf14b0-591c-4707-9872-3117d5dec124.png)

The General Purpose sensor will show 0-1 when door is closed, and 2600+ when door is opened. This is not nice, so we need to add a custom sensor in `sensors.yaml`
```
##### Garage Door ##################################
- platform: template
  sensors:
    garage_door_status:
      friendly_name: Garage Door Status
      value_template: >-
        {% set doorstate = states('sensor.garage_door_relay_general_purpose') | int %}
        {% if doorstate < 10 %}
        closed
        {% else %}
        open
        {% endif %}
      icon_template: >-
        {% set doorstate = states('sensor.garage_door_relay_general_purpose') | int %}
        {% if doorstate < 10 %}
        mdi:garage
        {% else %}
        mdi:garage-open
        {% endif %}

```

Now we can use the `sensor.garage_door_status` to check the status, so we can now add to `scripts.yaml` some actions to know when we are opening or closing the door.

```
open_garage_door:
  sequence:
    - alias: "Check if gatage door is closed"
      condition: state
      entity_id: sensor.garage_door_status
      state: "closed"
    - alias: "Open garage door"
      service: switch.turn_on
      data:
        entity_id: switch.garage_door_relay

close_garage_door:
  sequence:
    - alias: "Check if gatage door is open"
      condition: state
      entity_id: sensor.garage_door_status
      state: "open"
    - alias: "Cloase garage door"
      service: switch.turn_on
      data:
        entity_id: switch.garage_door_relay
```

I then added a dashboard card to see the garage door status
```
type: custom:mushroom-entity-card
entity: sensor.garage_door_status
name: Status
```
and some custom buttons to either open or close the door, based on the status
```
type: conditional
conditions:
  - entity: sensor.garage_door_status
    state: closed
card:
  show_name: true
  show_icon: true
  type: button
  tap_action:
    action: toggle
  entity: script.open_garage_door
  icon: mdi:garage-open
  name: Open Garage Door

```
```
type: conditional
conditions:
  - entity: sensor.garage_door_status
    state: open
card:
  show_name: true
  show_icon: true
  type: button
  tap_action:
    action: toggle
  entity: script.close_garage_door
  icon: mdi:garage
  name: Close Garage Door
```
![image](https://user-images.githubusercontent.com/15160444/177219440-7e93ddd0-517d-4e48-b37c-e8a0f3ea5fa8.png)

