---
title: LG webOS Smart TV
description: Instructions on how to integrate a LG webOS Smart TV within Home Assistant.
ha_category:
  - Media player
  - Notifications
ha_iot_class: Local Push
ha_release: 0.18
ha_codeowners:
  - '@thecode'
ha_domain: webostv
ha_config_flow: true
ha_ssdp: true
ha_platforms:
  - diagnostics
  - media_player
  - notify
ha_integration_type: integration
---

The `webostv` platform allows you to control a [LG](https://www.lg.com/) webOS Smart TV.

There is currently support for the following device types within Home Assistant:

- [Media player](/integrations/media_player/)
- [Notifications](/integrations/notify/)

To begin with enable *LG Connect Apps* feature in *Network* settings of the TV.

{% include integrations/config_flow.md %}

## Turn on action

If you want to use an automation to turn on an LG webOS Smart TV, install an {% term integration %} such as the [HDMI-CEC](/integrations/hdmi_cec/) or [WakeOnLan](/integrations/wake_on_lan/). They provide an action that can be used for that.


Common for webOS 3.0 and higher would be to use WakeOnLan feature. To use this feature your TV should be connected to your network via Ethernet rather than Wireless and you should enable the *LG Connect Apps* feature in *Network* settings of the TV (or *Mobile App* in *General* settings for older models) (*may vary by version).

On newer models (2017+), WakeOnLan may need to be enabled in the TV settings by going to Settings > General > Mobile TV On > Turn On Via WiFi [instructions](https://support.quanticapps.com/hc/en-us/articles/115005985729-How-to-turn-on-my-LG-Smart-TV-using-the-App-WebOS-).

{% important %}
This usually only works if the TV is connected to the same network. Routing the WakeOnLan packet to a different subnet requires special configuration on your router or may not be possible.
{% endimportant %}

You can create an automation from the user interface, from the device create a new automation and select the  **Device is requested to turn on** automation.
Automations can also be created using an automation action:

```yaml
# Example configuration.yaml entry
wake_on_lan: # enables `wake_on_lan` integration

automation:
  - alias: "Turn On Living Room TV with WakeOnLan"
    triggers:
      - trigger: webostv.turn_on
        entity_id: media_player.lg_webos_smart_tv
    actions:
      - action: wake_on_lan.send_magic_packet
        data:
          mac: aa:bb:cc:dd:ee:ff
```

Any other [actions](/docs/automation/action/) to power on the device can be configured.

## Sources

It is possible to select which sources will be available to the media player. When the TV is powered on press the **CONFIGURE** button in the {% term integration %} card and select the sources to enable. If you don't select any source the media player will offer all of the sources of the TV.

### Switching source with automation

Imagine you want your LG TV to automatically switch to a specific source when it turns on. Below is a simple automation example that launches `YouTube` after the TV is switched on.
It leverages `select_source` action from the [Media player](/integrations/media_player/) integration to launch a specific app installed on your LG TV.

To find available sources for your TV

1. Go to {% my developer_states title="**Developer Tools** > **States**" %}.
2. Find your TV's media_player entity.
3. Look for the `source_list` attribute which contains all available sources.
   
{% tip %}
Source list example: `source_list: ARD Mediathek, Apps, HDMI 1, Home Dashboard, JBL Bar 1300, Media Player, Netflix, Prime Video, Public Value, Spotify - Music and Podcasts, Timer, Web Browser, YouTube, ZDFmediathek`
{% endtip %} 

The automation can be created entirely through the Home Assistant UI. When setting it up, you'll only need to manually enter the source name (for example, "YouTube") in the action configuration. Below is the YAML code generated as a result:

```yml
alias: Switch TV source to YouTube by Default
description: 'Regardless if started from TV remote or via wake-on-lan, the TV will switch to YouTube right after it is on'
triggers:
  - device_id: <TV DEVICE ID>
    domain: media_player
    entity_id: <TV MEDIA PLAYER ENTITY ID>
    type: turned_on
    trigger: device
conditions: []
actions:
  - action: media_player.select_source
    metadata: {}
    data:
      source: YouTube
    target:
      device_id: <TV DEVICE ID>
mode: single
```

## Change channel through play_media action

The `play_media` action can be used in a script to switch to the specified TV channel. It selects the best matching channel according to the `media_content_id` parameter:

 1. Channel number *(i.e., '1' or '6')*
 2. Exact channel name *(i.e., 'France 2' or 'CNN')*
 3. Substring in channel name *(i.e., 'BFM' in 'BFM TV')*

```yaml
# Example action entry in script to switch to channel number 1
action: media_player.play_media
target:
  entity_id: media_player.lg_webos_smart_tv
data:
  media_content_id: 1
  media_content_type: "channel"

# Example action entry in script to switch to channel including 'TF1' in its name
action: media_player.play_media
target:
  entity_id: media_player.lg_webos_smart_tv
data:
  media_content_id: "TF1"
  media_content_type: "channel"
```

## Next/Previous buttons

The behavior of the next and previous buttons is different depending on the active source:

- if the source is 'LiveTV' (television): next/previous buttons act as channel up/down
- otherwise: next/previous buttons act as next/previous track

### Sound output

The current sound output of the TV can be found under the state attributes.
To change the sound output, the following action is available:

#### Action `webostv.select_sound_output`

| Data attribute | Optional | Description                             |
| ---------------------- | -------- | --------------------------------------- |
| `entity_id`            | no       | Target a specific webostv media player. |
| `sound_output`         | no       | Name of the sound output to switch to.  |

### Generic commands and buttons

Available actions: `button`, `command`

### Action `webostv.button`

| Data attribute | Optional | Description                                                                                                                                                                                                                                                                            |
| ---------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `entity_id`            | no       | Target a specific webostv media player.                                                                                                                                                                                                                                                |
| `button`               | no       | Name of the button. Known possible values are `LEFT`, `RIGHT`, `DOWN`, `UP`, `HOME`, `MENU`, `BACK`, `ENTER`, `DASH`, `INFO`, `ASTERISK`, `CC`, `EXIT`, `MUTE`, `RED`, `GREEN`, `BLUE`, `YELLOW`, `VOLUMEUP`, `VOLUMEDOWN`, `CHANNELUP`, `CHANNELDOWN`, `PLAY`, `PAUSE`, `NETFLIX`, `GUIDE`, `AMAZON`, `0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9` |

### Action `webostv.command`

| Data attribute | Optional | Description                                                                                                                                                                          |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `entity_id`            | no       | Target a specific webostv media player.                                                                                                                                              |
| `command`              | no       | Endpoint for the command, e.g.,  `system.launcher/open`.  The full list of known endpoints is available at <https://github.com/bendavid/aiopylgtv/blob/master/aiopylgtv/endpoints.py> |
| `payload`             | yes      | An optional payload to provide to the endpoint in the format of key value pair(s). |

### Example

```yaml
script:
  home_button:
    sequence:
      - action: webostv.button
        target:
          entity_id:  media_player.lg_webos_smart_tv
        data:
          button: "HOME"

  open_google_command:
    sequence:
      - action: webostv.command
        target:
          entity_id:  media_player.lg_webos_smart_tv
        data:
          command: "system.launcher/open"
          payload:
            target: https://www.google.com
```

## Notifications

The `webostv` notify platform allows you to send notifications to a LG webOS Smart TV.

The icon can be overridden for individual notifications by providing a path to an alternative icon image to use:

```yaml
automation:
  - alias: "Front door motion"
    triggers:
      - trigger: state
        entity_id: binary_sensor.front_door_motion
        to: "on"
    actions:
      - action: notify.livingroom_tv
        data:
          message: "Movement detected: Front Door"
          data:
            icon: "/home/homeassistant/images/doorbell.png"
```

## Notes

If Home Assistant and your TV are not on the same network, you need to create a firewall rule, which allows a connection on ports 3000 & 3001 with the TCP protocol from Home Assistant to your TV.
