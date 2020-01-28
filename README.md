
# New switchmodes for Tasmota
With Switchmode 11 and 12 you can control a dimmer with one switch.

## SwitchMode 11
Set push-button with dimmer mode 

Tasmota will send a TOGGLE command (switch2#state=2) when the button is pressed for a short time and is then released. When pressing the button (closing the circuit) for a long time (set in SetOption32) Tasmota sends repeated INC_DEC (increment or decrement the dimmer) commands (switch2#state=4) as long as the button is pressed. Releasing the button starts a internal timer, the time is set in SetOption32. When released for the time set in SetOption32 Tasmota sends a CLEAR command (switch2#state=6). If the button is pressed again before the timeout Tasmota sends a INV command (switch#state=5). The INV command is for the controlling sortware (home assistant) to switch between incrementing and decrementing the dimmer. 

## SwitchMode 12
Set inverted push-button with dimmer mode.
The same as Switchmode 11 with inverted Input


# My setup
I used a shelly_2 to control a shelly_RGBW 

A short press of the switch sends a message to toggle the dimmer.
A long press sends repeated messages to increment the dimmer.
If a second press of the switch follows the first press a message is sent to invert the function from increment to decrement and repeatet messages are sent to decrement the dimmer.
After releasing the switch a timeout message resets the automation.

## Tasmota Rules for shelly_2 Switch
### normal toggle function
on switch2#state=2 do publish stat/richelly/DVES_B95200/power2 toggle endon

### keeping the switch closed will increment (or decrement) the dimmer
on switch2#state=4 do publish stat/richelly/DVES_B95200/power2 inc_dec endon

### inversion of increment and decrement for the next inc_dec states
on switch2#state=5 do publish stat/richelly/DVES_B95200/power2 inv endon

### timeout will clear all functionalities
on switch2#state=6 do publish stat/richelly/DVES_B95200/power2 clear endon

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
      topic: "stat/richelly/DVES_B95200/power2"  # set the state of DVES_B95200 power2 to "clear"
      payload: "clear"

- alias: 'light inc'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'clear' 
    to: 'inc_dec'
  action:
  - data:
      entity_id: light.dves_b062b0_light_3
    service: light.turn_on  
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B95200/power2"  # set the state of DVES_B95200 power2 to "clear"
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
      topic: "stat/richelly/DVES_B95200/power2" # set the state of DVES_B95200 power2 to "dec"
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
      topic: "stat/richelly/DVES_B95200/power2"  # set the state of DVES_B95200 power2 to "clear"
      payload: "clear"

- alias: 'light dec'
  trigger:
  - entity_id: sensor.dves_b95200_switch_2
    platform: state
    from: 'dec'
    to: 'inc_dec'
  action:
  - data:
      entity_id: light.dves_b062b0_light_3
    service: light.turn_on    
  - service: mqtt.publish
    data:
      topic: "stat/richelly/DVES_B95200/power2"  # set the state of DVES_B062B0 power2 to "dec"
      payload: "dec"
  - service: mqtt.publish
    data:
      topic: "cmnd/richelly/DVES_B062B0/channel3"  # execute decrement light on DVES_B062B0 channel 3
      payload: "-"
```
