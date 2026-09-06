# 🌦️ Dynamic Weather Window Icon for Home Assistant

This project is a **dynamic animated sensor** that generates an SVG icon in real-time, simulating a view from a window. The icon is fully procedural (the code generates the graphics itself), requires no external files, and adapts to the current weather conditions in your region.

## ✨ Key Features

*   **🕶️ UV-Adaptive Sun:** The color of the sun and its rays changes depending on the `uv_index` (from soft yellow to extreme red). At night, the sun automatically turns gray.
*   **💨 Smart Aerodynamics:** The direction of cloud movement and the tilt of raindrops depend on the `wind_bearing` attribute (wind direction).
*   **☁️ Procedural Clouds:** The number of clouds (from 1 to 12) is calculated based on `cloud_coverage`. Each cloud has a unique size, altitude, and speed, which scale according to wind strength.
*   **🌧️ Adaptive Precipitation:** The icon distinguishes between "Rain", "Pouring", and "Lightning", changing the number of drops and their falling intensity.
*   **🖼️ Window Effect:** All weather phenomena (sun, clouds, rain) are rendered inside a stylized frame and properly clipped at the edges of the "glass" using `clipPath`.
*   **❄️ Snowfall:** Added a snowflake rendering layer using a detailed vector path.
*   **🌧️+❄️ Mixed Precipitation:** Implemented `snowy-rainy` logic. Now, during sleet, raindrops and snowflakes fall simultaneously.
  
## 🚀 Installation

<details>
  <summary><b>Add this code to your `configuration.yaml` (under the `template:` section). </b></summary>

```yaml
template:
  - sensor:
      - name: "Forecast Live Icon"
        state: "{{ states('weather.forecast') }}"
        picture: >

```
 📝 <a href="https://raw.githubusercontent.com/S0ft0r/ha-blueprints/refs/heads/main/svg/forecast.yaml">Full code here</a>
</details>

## 🛠️ Technical Logic Description

### UV Index Color Indication
The sun's color is calculated based on the following scale:
- **Low (0-2):** Yellow `#fdd835`
- **Moderate (3-5):** Golden `#fbc02d`
- **High (6-7):** Orange `#fb8c00`
- **Very High (8-10):** Red-Orange `#d84315`
- **Extreme (11+):** Red `#b71c1c`

### Sun and Rays
Triggered by the UV index:
- **Low (<3):** 5 rays.
- **Moderate (3-7):** 7 rays.
- **High (>8):** 9 rays.
The ray animation is implemented to create a "flowing" light effect.

### Animation and Wind
- **Direction:** If the wind blows from the West (181°-360°), clouds move from left to right. If from the East (0°-180°) — from right to left.
- **Speed:** Cloud movement speed is directly tied to `wind_speed`. The higher the wind speed, the faster the clouds drift.
- **Tilt:** Raindrops tilt along the X-axis depending on wind direction, creating a slanted rain effect.

### Optimization
The icon uses `urlencode` to pass the SVG code into the `entity_picture` attribute. This allows it to be used via standard Home Assistant methods in any card (Entities, Glance, Mushroom, etc.).

## 📝 Requirements
*   An active weather integration (e.g., `weather.forecast`).
*   The standard `sun` component enabled.

## 📝 Animation Examples
| Variable <br> Cloudiness | Cloudy | Rain | Sunny <br> High UV | Sunny <br> Normal UV | Snowy-rain |
| :---: | :---: | :---: | :---: | :---: | :---: | 
| <img src="./1.svg" width="50"> | <img src="./2.svg" width="50"> | <img src="./3.svg" width="50"> | <img src="./4.svg" width="50"> | <img src="./5.svg" width="50"> | <img src="./6.svg" width="50"> | 
---

## 📝 Usage Examples
<details>
  <summary>**multiple-entity-row:**</summary>

Some UI elements cannot directly work with `entity_picture` — they require a string with an image name. Therefore, to use the animated SVG, a different approach is needed: rendering the icon using `card_mod`. Below is an example that uses the weather icon alongside [blinds](./blind.md) and [sun compass](./compass.md) icons.

```yaml
type: entities
state_color: true
entities:
  - entity: sensor.forecast_live_icon
    name: Blinds
    icon: mdi:blinds 
    type: custom:multiple-entity-row
    show_state: false
    state_color: true
    name: false
    entities:
      - entity: sensor.sun_compass_icon
        name: false
        unit: false
        state_color: true
        icon:  mdi:sun-compass
    entities:
      - entity: sensor.blinds_live_icon
        name: false
        unit: false
        state_color: true
        icon:  mdi:blinds
    card_mod:
      style: 
        hui-generic-entity-row $: |
          state-badge {
            color: transparent !important;
            --mdc-icon-size: 32px; 

            background-color: transparent !important;
            background-image: url("{{ state_attr('sensor.forecast_live_icon', 'entity_picture') }}") !important;
            background-size: 32px 32px !important;
            background-repeat: no-repeat !important;
            background-position: center !important;
            
            width: 32px !important;
            height: 32px !important;
            display: inline-block !important;
            border-radius: 0 !important;
            box-shadow: none !important;
          }
          .entities-row .entity:nth-child(1) state-badge {
            color: transparent !important;
            --mdc-icon-size: 32px; 
            
            background-color: transparent !important;
            background-image: url("{{ state_attr('sensor.sun_compass_icon', 'entity_picture') }}") !important;
            background-size: 32px 32px !important;
            background-repeat: no-repeat !important;
            background-position: center !important;
            
            width: 32px !important;
            height: 32px !important;
            display: inline-block !important;
            border-radius: 0 !important;
            box-shadow: none !important;
          }
          .entities-row .entity:nth-child(1) state-badge ha-state-icon {
            opacity: 0 !important;
          }

          .entities-row .entity:nth-child(2) state-badge {
            color: transparent !important;
            --mdc-icon-size: 32px; 
            
            background-color: transparent !important;
            background-image: url("{{ state_attr('sensor.blinds_live_icon', 'entity_picture') }}") !important;
            background-size: 32px 32px !important;
            background-repeat: no-repeat !important;
            background-position: center !important;
            
            width: 32px !important;
            height: 32px !important;
            display: inline-block !important;
            border-radius: 0 !important;
            box-shadow: none !important;
          }
          .entities-row .entity:nth-child(2) state-badge ha-state-icon {
            opacity: 0 !important;
          }

```
</details>


**Author:** [Softor]
**License:** MIT
```
