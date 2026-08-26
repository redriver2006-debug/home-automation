# Home Assistant configuration

Sanitised export of a Home Assistant Core (Docker) setup on a Raspberry Pi.

## Contents
- `homeassistant/` - configuration.yaml, automations, scripts, scenes
- `esphome/` - ESP32-C6 IR blaster configs (Daikin, Fujitsu) and shared packages

## Not included
Secrets, tokens, the recorder database, dashboards (`.storage/`), SSH keys and
`www/` media are all excluded. Names, coordinates, IP addresses, MAC and Zigbee
IEEE addresses have been replaced with placeholders.

Copy `secrets.yaml.example` to `secrets.yaml` and fill in your own values.
