# Floorplan for Home Assistant


![floorplan](https://user-images.githubusercontent.com/2073827/27056482-911f2e14-500b-11e7-90f0-44a344c39f85.png)

## Background

The Home Assistant [front end](https://home-assistant.io/docs/frontend/) provides a great way of viewing and interacting with your entities. This project builds on top of that, allowing you to extend the front end by adding your own visuals, such as floorplans, images of your devices, etc.

With Floorplan for Home Assistant, you can:

- Display your floorplan image as a state card or custom panel
- Include any number of entities (i.e. binary sensors, lights, cameras, etc.) on your floorplan
- Style entity states using CSS
- Gradually transition between states using color gradients
- Display the last triggered binary sensor using CSS
- Display hover-over text for each entity

## Usage

To get started, copy the following files from this repo to your Home Assistant directory:

```
www/custom_ui/floorplan/ha-floorplan.html
www/custom_ui/floorplan/floorplan.svg
www/custom_ui/floorplan/floorplan.css
```

Although a sample floorplan SVG file is included in this repo, you will want to create your own. See the appendix for more information.

You then have two options for how you want to the floorplan to appear in Home Assistant:

- custom state card
- custom panel

Of course, you can choose to have it displayed in both places.

### Option 1: Floorplan custom state card

![image](https://user-images.githubusercontent.com/2073827/27063035-97aa2a6e-5032-11e7-8e8e-79935a19aebf.png)

To display the floorplan as a custom state card, copy the following file from this repo to your Home Assistant directory:

```
www/custom_ui/state-card-floorplan.html
```

Since Home Assistant requires a single entity to be used as the target for a state card, we create a virtual entity to represent the overall floorplan. You can choose any type of entity for this, such as the MQTT binary sensor. Add the following to your Home Assistant configuration:

```
binary_sensor:
  - platform: mqtt
    state_topic: dummy/floorplan/sensor
    name: Floorplan
```

Then, add the following to get Home Assistant to display this new virtual entity using the floorplan custom state card:

```
homeassistant:
  customize:    
    binary_sensor.floorplan:
      custom_ui_state_card: floorplan
      config: !include floorplan.yaml
```

To actually display the floorplan custom state card in the front end, add the virtual entity to one of your groups:

```
group:
  zones:
    name: Zones
    entities:
      - binary_sensor.floorplan
```

You can also add a 'last motion' entity to keep track of which binary sensor was triggered last. See the appendix for more information.

### Option 2: Floorplan custom panel

![image](https://user-images.githubusercontent.com/2073827/27063110-08d3fd82-5033-11e7-85b6-671722608394.png)

To display the floorplan as a custom panel, copy the following file from this repo to your Home Assistant directory:

```
panels/floorplan.html
```

Then, add the following to your Home Assistant configuration:

```
  - name: floorplan
    sidebar_title: Floorplan
    sidebar_icon: mdi:home
    url_path: floorplan
    config: !include floorplan.yaml
```

### Configure the floorplan

Whether your floorplan is being displayed as a custom state card or as a custom panel, the same configuration file `floorplan.yaml` is used. This is where you tell Home Assistant which entities you want to display on your floorplan.

The example `floorplan.yaml` included in this repo shows how to add various entities to your floorplan and style their appearance based on their states.

At the top of the file, you provide a name for the floorplan, as well as the location of the SVG and CSS files:

```
      name: Demo Floorplan
      image: /local/custom_ui/floorplan/floorplan.svg
      stylesheet: /local/custom_ui/floorplan/floorplan.css
```

If you want to display a 'lost motion' entity, you can include that in the next section of the file. You specify the name of the entity, as well as the CSS class used to style its appearance:

```
      last_motion_entity: sensor.template_last_motion
      last_motion_class: last-motion
```

The remainder of the file is where you add your groups.

```
      groups:
```

You need to place each of your entities into a group, since configuration is performed at a group level. The groups can be given any name, and have no purpose other than to allow for configuration of multiple items in one place.

Below are some examples of groups, showing how to configure different types of entites in the floorplan.

#### Sensors

Below is an example of a 'Sensors' group, showing how to add a temperature sensor (as text) to your floorplan. in the screenshot above, this can be seen at text (i.e. '9.0°').

The sensor's state is displayed using a text template. Whenever a text template is used, the floorplan SVG needs to contain an SVG text element whose id matches the Home Assistant entity its representing.

The sensor's CSS class is determined dynamically using a class template. In the example below, the CSS class is determined based on the actual temperature value. Both text and class templates allow you to inject your own expressions and code using JavaScript string literals.

```
        - name: Sensors
          entities:
             - sensor.melbourne_now
          text_template: '${entity.state ? entity.state : "unknown"}'
          class_template: '
            var temp = parseFloat(entity.state.replace("°", ""));
            if (temp < 10)
              return "temp-low";
            else if (temp < 30)
              return "temp-medium";
            else
              return "temp-high";
```

#### Switches

Below is an example of a 'Switches' group, showing how to add switches to your floorplan. The appearance of each switch is styled using the appropriate CSS class, based on its current state. The `action` is optional, and allows you to specify which service should be called when this entiy is clicked.

```
        - name: Switches
          entities:
             - switch.doorbell
          states:
            - state: 'on'
              class: 'doorbell-on'
            - state: 'off'
              class: 'doorbell-off'
          action:
            service: toggle
```

#### Lights

Below is an example of a 'Lights' group, showing how to add lights to your floorplan. The appearance of each light is styled using the appropriate CSS class, based on its current state.

```
        - name: Lights
          entities:
             - light.hallway
             - light.living_room
             - light.patio
          states:
            - state: 'on'
              class: 'light-on'
            - state: 'off'
              class: 'light-off'
```

#### Alarm Panel

Below is an example of an 'Alarm Panel' group, showing how to add an alarm panel (as text) to your floorplan. The appearance of the alarm panel is styled using the appropriate CSS class, based on its current state. In the screenshot above, this can be seen as text (i.e. 'disarmed').

```
       - name: Alarm Panel
          entities:
             - alarm_control_panel.alarm
          states:
            - state: 'armed_away'
              class: 'alarm-armed'
            - state: 'armed_home'
              class: 'alarm-armed'
            - state: 'disarmed'
              class: 'alarm-disarmed'
```

#### Binary Sensors

Below is an example of a 'Binary sensors' group, showing how to add binary sensors to your floorplan. The appearance of each binary sesor is styled using the appropriate CSS class, based on its current state. In the screenshot above, these can be seen as zones (i.e. rooms).

The `state_transitions` section is optional, and allows your binary sensors to visually transition from one state to another, using the fill colors defined in the CSS classes associated with each state. You can specify the duration (in seconds) for the transition from one color to the other.

```
        - name: Binary Sensors
          entities:
            - binary_sensor.front_hallway
            - binary_sensor.kitchen
            - binary_sensor.master_bedroom
            - binary_sensor.theatre_room
          states:
            - state: 'off'
              class: 'info-background'
            - state: 'on'
              class: 'warning-background'
          state_transitions:
            - name: On to off
              from_state: 'on'
              to_state: 'off'
              duration: 10
```

#### Cameras

Below is an example of a 'Cameras' group, showing how to add cameras to your floorplan. The appearance of each camera is styled using the appropriate CSS class, based on its current state. In the screenshot above, these can be seen as camera icons.

        - name: Cameras
          entities:
            - camera.hallway
            - camera.driveway
            - camera.front_door
            - camera.backyard
          states:
            - state: 'idle'
              class: 'camera-idle'

#### Media Players

Below is an example of a 'Media Players' group, showing how to add media players to your floorplan. The appearance of each media player is styled using the appropriate CSS class, based on its current state. In the screenshot above, these can be seen as Logitech Squeezebox icons.

        - name: Media Players
          entities:
            - media_player.alfresco
            - media_player.ensuite
            - media_player.salon
          states:
            - state: 'off'
              class: 'squeezebox-off'
            - state: 'idle'
              class: 'squeezebox-off'
            - state: 'paused'
              class: 'squeezebox-off'
            - state: 'playing'
              class: 'squeezebox-on'

## Appendix

### Creating a floorplan SVG file

[Inkscape](https://inkscape.org/en/develop/about-svg/) is a free application that lets you create vector images. You can make your floorplan as simple or as detailed as you want. The only requirement is that you create a shape (i.e. `rectangle`, `path`, etc.) for each entity ( i.e. binary sensor, switch, or light) you want to display on your floorplan. Each of these shapes needs to have its `id` set to the entity name in Home Assistant.

For example, below is what the shape looks like for a Front Hallway binary sensor. The `id` of the shape is set to the entity name `binary_sensor.front_hallway`. This allows the shape to automatically get hooked up to the right entity when the state card is displayed.

```html
<path id="binary_sensor.front_hallway" d="M650 396 c0 -30 4 -34 31 -40 17 -3 107 -6 200 -6 l169 0 0 40 0 40
-200 0 -200 0 0 -34z"/>
```
Once you've created your SVG file, save it a directory within your Home Assistant installation. For example:
```
<path_to_home_assistant>/www/custom_ui/floorplan/floorplan.svg
```

### Adding a last motion entity to your floorplan

As an optional step, you can create a 'last motion' entity to keep track of which binary sensor was triggered last. To do so, add the following:

```
sensor:
  - platform: template
    sensors:
      template_last_motion:
        friendly_name: 'Last Motion'
        value_template: >
          {%- set sensors = [states.binary_sensor.theatre_room, states.binary_sensor.back_hallway, states.binary_sensor.front_hallway, states.binary_sensor.kitchen] %}
          {% for sensor in sensors %}
            {% if as_timestamp(sensor.last_changed) == as_timestamp(sensors | map(attribute='last_changed') | max) %}
              {{ sensor.name }}
            {% endif %}
          {% endfor %}
```

To actually display the 'last motion' entity', add it to one of your groups:

```
group:
  zones:
    name: Zones
    entities:
      - sensor.template_last_motion
      - binary_sensor.floorplan
```
