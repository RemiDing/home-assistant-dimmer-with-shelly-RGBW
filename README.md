# home-assistant-dimmer-with-shelly-RGBW

Rules for Switch

on switch2#state=2 do publish stat/richelly/DVES_B062B0/power2 toggle endon
#normal toggle functiona

on switch2#state=3 do publish stat/richelly/DVES_B062B0/power2 hold endon
#hold will increment or decrement the dimmer

on switch2#state=4 do publish stat/richelly/DVES_B062B0/power2 inv endon
#inversion of increment and decrement for the next hold states

on switch2#state=5 do publish stat/richelly/DVES_B062B0/power2 clear endon
#timeout will clear all functionalities
