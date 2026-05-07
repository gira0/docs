# Home Assistant

Notes, automations, and integrations related to Home Assistant.

## Getting started

- Setup tips
- Useful integrations
- Example automation snippets

## Example automation (YAML)

```yaml
alias: Turn on porch light at sunset
trigger:
  - platform: sun
    event: sunset
action:
  - service: light.turn_on
    target:
      entity_id: light.porch
```

## Topics to add

- Integrations (Zigbee, Z-Wave, MQTT)
- Sensor templating and scripting
- Backup and update strategies

Add your Home Assistant notes and examples here.
