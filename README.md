# Custom Alarm Panel Card

This custom alarm panel card was forked in September 2020 from [Kevin Cooper's repo](https://github.com/JumpMaster/custom-lovelace) (no relation). 

I have also included my complete working [manual alarm control panel](https://www.home-assistant.io/integrations/manual/) configuration, automations, script, lovelace dashboard, etc in the [ExampleConfig](https://github.com/jcooper-korg/AlarmPanel/tree/master/ExampleConfig) folder.

## Card Modification Goals 

* improve the appearance of the arm/disarm and keypad buttons (which are too small on current Home Assistant versions)
* high visibility and ergonomically sized buttons on web, iOS companion app, and wall mounted tablet
* hide the key pad when disarmed (if `alarm_control_panel.code_arm_required` is off)
* add a config option to show Ready / Not Ready when disarmed, monitoring a given list of entities
* add a config option to show a countdown timer when arming or pending

I changed the buttons from mwc-button to regular buttons. I couldn't find a reliable way to make the mwc-buttons larger (even using tools like [Thomas Loven's card mod](https://github.com/thomasloven/lovelace-card-mod)), and they were just much too small on a wall-mounted tablet.

I moved the Disarm button from the top button row to the keypad, to the right of the 0 button. This allows the overall keypad area and button size to be increased, which is helpful on a phone screen or wall mounted tablet.

## Screenshots

<img src="https://github.com/jcooper-korg/AlarmPanel/blob/master/Screenshots/Disarmed-ready.png?raw=true" width="400">
<img src="https://github.com/jcooper-korg/AlarmPanel/blob/master/Screenshots/Disarmed-not-ready.png?raw=true" width="400">
<img src="https://github.com/jcooper-korg/AlarmPanel/blob/master/Screenshots/Armed-home.png?raw=true" width="400">

## Installation

To use this card in Home Assistant:

<img src="https://github.com/jcooper-korg/AlarmPanel/blob/master/Screenshots/Installation-Resource.png?raw=true" width="400">


* copy the `alarm_control_panel-card.js` into the www folder in your config folder (create the www folder if it's missing, and restart Home Assistant)
* install it as a custom Lovelace resource in Configuration > Lovelace Dashboards > Resources. 
	* Turn on Advanced Mode in your user profile if you can't see the Resources tab. 
	* The Url of the file will be `/local/alarm_control_panel-card.js`, and the type is "JavaScript Module".  
	* If you are making local modifications to the file, you can add a version number to the end of the Url, like `/local/alarm_control_panel-card.js?v=3` and increment the number each time you make a change, to force it to use the new version instead of your browser cached version.
* add the alarm panel to your lovelace view using a Manual card, with type set as `type: 'custom:alarm_control_panel-card'` (see my [example config](https://github.com/jcooper-korg/AlarmPanel/blob/master/ExampleConfig/AlarmLovelaceDashboard.yaml))


## Card configuration options

See the [ExampleConfig](https://github.com/jcooper-korg/AlarmPanel/tree/master/ExampleConfig) folder for my configuration files. See details on my setup in the Example Configuration section below.

The card options are:

* `entity`: (required string) the name of the manual `alarm_control_panel` entity
* `timer_entity`: (optional string) the name of the global timer entity to be used during arming/pending
* `scale`: (optional string). default is 14px. increase/decrease the size of the buttons/text/etc by changing this number
* `title`: (optional string) if provided will show this title at the top of the card, and the alarm state will be below it. if not provided, will show the alarm state as the title (which saves some vertical space, if you are space constrained, like on a wall tablet)
* `states`: (optional list). list of arming states to support. Default is `armed_away` and `armed_home`. If you use more than two, you may need to adjust the `.actions button` widths 
* `confirm_entities`: (optional list) a list of sensors which will be continuously monitored when disarmed so it can show Ready/Not ready text in the car header
* `labels`: (optional list) list of text replacements, allowing you to customize the text that is shown for `ui.card.alarm_control_panel.arm_away`, `ui.card.alarm_control_panel.arm_home`, `ui.card.alarm_control_panel.clear_code`, `ready` and `not_ready` 
* `display_letters`: (optional boolean) shows letters on number pad buttons, like a telephone keypad
* `style`: (optional string) this text will be appended to the card css style, allowing you to override colors, etc. Also see [Thomas Loven's card mod](https://github.com/thomasloven/lovelace-card-mod)

Other options from [Kevin Cooper's original card](https://github.com/JumpMaster/custom-lovelace/tree/master/alarm_control_panel-card) which may  not work well when `alarm_control_panel.code_arm_required` is off:

* `auto_hide`: (optional boolean) hides the keypad and action buttons. click on the badge to show/hide them.  
* `auto_enter`: (optional object) specify code_length and arm_action to automatically arm when entering the code.  

## My Setup

My alarm setup consists of:

* Home Assistant running on Hass.io, on a [Raspberry Pi 3b+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/)
* a wall-mounted 7" Amazon Fire Tablet, running [Fully Kiosk](https://www.fully-kiosk.com)
* the iOS [Home Assistant Companion](https://apps.apple.com/us/app/home-assistant/id1099568401)
* an alarm panel kit from [Konnected.io](https://konnected.io/collections/shop-now) with various door/window/motion/smoke sensors, and a siren, beeper, and red/green LEDs

## Example Configuration
My config files are in the [ExampleConfig](https://github.com/jcooper-korg/AlarmPanel/tree/master/ExampleConfig) folder. This was last tested with HA 0.114.3.

* My Alarm lovelace dashboard has two cards- the new custom alarm panel card, and a [Conditional Card](https://www.home-assistant.io/lovelace/conditional) which shows when any of the door/window sensors is opened, while disarmed.
* I have configured the custom card with:
	* label replacements to use shorter all-caps words for the AWAY, HOME, etc.
	* confirm_entities list of sensors, so that it shows "Ready" if they're all off, or "Not ready" if any are on
* I have set up automations to handle:
	* turning on/off the green/red LEDs, beeper, and siren based on sensor entity states and the manual `alarm_control_panel` armed/disarmed/triggered state
	* notifying our iphone's when armed / disarmed or when triggered
	* triggering the alarm on smoke sensors, regardless of arming state
* In order to include the name of the entity that triggered the alarm in the trigger notifications, I'm using an input_text entity in my config, which is set when the alarm trigger automation runs, and is then referenced by the notification
* In order to be able to trigger the alarm immediately for some sensors, while other sensors (e.g. entry doors) are delayed, I have a script called `trigger_alarm_immediately` which first disarms the alarm, and then triggers. Requires that the the `delay_time` is set to 0 for the disarmed state in the `alarm_control_panel` configuration.
* To show a countdown timer when arming/pending, I use a timer entity (added in configuration.yaml). Using a global timer allows it to show the same time remaining on different instances of the card (e.g. phone, wall tablet, desktop), regardless of whether the card was showing at the time of the alarm state change.
* I created a separate user named Alarm Panel that I use to log in from my wall mounted tablet. I'm using [Custom Header](https://maykar.github.io/custom-header/) to hide the sidebar and title bar on the wall mounted tablet for that user.