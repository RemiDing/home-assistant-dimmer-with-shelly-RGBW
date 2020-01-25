
# New switchmodes for Tasmota
With Switchmode 11 and 12 you can control a dimmer with one switch.

I used a shelly_2 to control a shelly_RGBW 

A short press of the switch sends a message to toggle the dimmer.
A long press sends repeated messages to increment the dimmer.
If a second press of the switch follows the first press a message is sent to invert the function from increment to decrement and repeatet messages are sent to decrement the dimmer.
After releasing the switch a timeout message resets the automation.

## Tasmota Rules for shelly_2 Switch
### normal toggle function
on switch2#state=2 do publish stat/richelly/DVES_B062B0/power2 toggle endon

### hold will increment or decrement the dimmer
on switch2#state=3 do publish stat/richelly/DVES_B062B0/power2 hold endon

### inversion of increment and decrement for the next hold states
on switch2#state=4 do publish stat/richelly/DVES_B062B0/power2 inv endon

### timeout will clear all functionalities
on switch2#state=5 do publish stat/richelly/DVES_B062B0/power2 clear endon

# Example for Automation in Home Assist
```
Of the shelly_2 only one switch (sensor.dves_b95200_switch_2) is used. 
Only channel 3 of the shelly_RGBW (cmnd/richelly/DVES_B062B0/channel3) is used and works as a dimmer.
cmnd/richelly/DVES_B062B0/channel3  with payload "+"  increments the dimmer
cmnd/richelly/DVES_B062B0/channel3  with payload "-"  decrements the dimmer
```
```
- alias: 'light toggle'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    to: 'toggle'
  action:
  - data:
      entity_id: light.dves_b062b0_light_3
    service: light.toggle
  - service: mqtt.publish
    data: 
      topic: "stat/richelly/DVES_B062B0/power2"  # set the state of DVES_B062B0 power2 to "clear"
      payload: "clear"

- alias: 'light inc'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'clear' 
    to: 'hold'
  action:
  - data:
      entity_id: light.dves_b062b0_light_3
    service: light.turn_on  
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B062B0/power2"  # set the state of DVES_B062B0 power2 to "clear"
      payload: "clear"
  - service: mqtt.publish
    data:
      topic: "cmnd/richelly/DVES_B062B0/channel3"  # execute increment light on DVES_B062B0 channel 3
      payload: "+"

- alias: 'switch inc dec'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'clear'
    to: 'inv'
  action: 
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B062B0/power2" # set the state of DVES_B062B0 power2 to "dec"
      payload: "dec"   


- alias: 'switch dec inc'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'dec'
    to: 'inv'
  action:
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B062B0/power2"  # set the state of DVES_B062B0 power2 to "clear"
      payload: "clear"

- alias: 'light dec'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'dec'
    to: 'hold'
  action:
  - data:
      entity_id: light.dves_b062b0_light_3
    service: light.turn_on    
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B062B0/power2"  # set the state of DVES_B062B0 power2 to "dec"
      payload: "dec"
  - service: mqtt.publish
    data:
      topic: "cmnd/richelly/DVES_B062B0/channel3"  # execute decrement light on DVES_B062B0 channel 3
      payload: "-"
```
