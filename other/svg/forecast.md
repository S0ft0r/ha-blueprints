
# 🌦️ Dynamic Weather Window Icon for Home Assistant

Этот проект представляет собой **динамический анимированный сенсор**, который генерирует SVG-иконку в реальном времени, имитирующую вид из окна. Иконка полностью процедурная (код генерирует графику сам), не требует внешних файлов и адаптируется под текущие погодные условия вашего региона.

## ✨ Основные особенности

*   **🕶️ UV-адаптивное солнце:** Цвет солнца и его лучей меняется в зависимости от `uv_index` (от мягкого желтого до экстремального красного). Ночью солнце автоматически становится серым.
*   **💨 Умная аэродинамика:** Направление движения облаков и наклон капель дождя зависят от атрибута `wind_bearing` (направление ветра).
*   **☁️ Процедурные облака:** Количество облаков (от 1 до 12) рассчитывается на основе `cloud_coverage`. Каждое облако имеет уникальный размер, высоту и скорость, которые масштабируются в зависимости от силы ветра.
*   **🌧️ Адаптивные осадки:** Иконка различает «Дождь», «Ливень» и «Грозу», меняя количество капель и интенсивность их падения.
*   **🖼️ Эффект окна:** Все погодные явления (солнце, облака, дождь) отрисованы внутри стилизованной рамы и корректно обрезаются по краям «стекла» с помощью `clipPath`.
*   **❄️ Снегопад:** Добавлен слой отрисовки снежинок с использованием детализированного векторного пути.
*   **🌧️+❄️ Смешанные осадки:** Реализована логика `snowy-rainy`. Теперь при мокром снеге капли дождя и снежинки падают одновременно.
  
## 🚀 Установка

<details>
  <summary><b>Добавьте этот код в ваш `configuration.yaml` (раздел `template:`). </b></summary>

```yaml
template:
  - sensor:
      - name: "Forecast Live Icon"
        state: "{{ states('weather.forecast') }}"
        picture: >

```
 📝 <a href="https://raw.githubusercontent.com/S0ft0r/ha-blueprints/refs/heads/main/other/svg/forecast.yaml">Полный код здесь</a>
</details>

## 🛠️ Техническое описание логики

### Цветовая индикация UV-индекса
Цвет солнца рассчитывается по следующей шкале:
- **Низкий (0-2):** Желтый `#fdd835`
- **Умеренный (3-5):** Золотистый `#fbc02d`
- **Высокий (6-7):** Оранжевый `#fb8c00`
- **Очень высокий (8-10):** Красно-оранжевый `#d84315`
- **Экстремальный (11+):** Красный `#b71c1c`
- 
### Солнце и Лучи
Используется триггер по UV-индексу:
- **Low (<3):** 5 лучей.
- **Moderate (3-7):** 7 лучей.
- **High (>8):** 9 лучей.
Анимация лучей реализована так, что создает эффект «стекания» света.

### Анимация и Ветер
- **Направление:** Если ветер дует с Запада (181°-360°), облака плывут слева направо. Если с Востока (0°-180°) — справа налево.
- **Скорость:** Скорость движения облаков напрямую связана с `wind_speed`. Чем выше скорость ветра, тем быстрее проплывают облака.
- **Наклон:** Капли дождя отклоняются по оси X в зависимости от направления ветра, создавая эффект косого дождя.

### Оптимизация
Иконка использует `urlencode` для передачи SVG-кода в атрибут `entity_picture`. Это позволяет использовать её стандартными средствами Home Assistant в любых картах (Entities, Glance, Mushroom и др.).

## 📝 Требования
*   Активная интеграция погоды (например, `weather.forecast`).
*   Включенный стандартный компонент `sun`.

## 📝 Примеры анимаций
| Переменная <br> облачность | Облачно | Дождь | Солнечно <br> высокий УФИ | Солнечно <br> УФИ в норме | Снег с дождем |
| :---: | :---: | :---: | :---: | :---: | :---: | 
| <img src="./1.svg" width="50"> | <img src="./2.svg" width="50"> | <img src="./3.svg" width="50"> | <img src="./4.svg" width="50"> | <img src="./5.svg" width="50"> | <img src="./6.svg" width="50"> | 
---

## 📝 Примеры использования
<details>
  <summary>**multiple-entity-row:**</summary>

```yaml
type: entities
state_color: true
entities:
  - entity: sensor.forecast_live_icon
    name: Жалюзи
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


**Автор:** [Softor]
**Лицензия:** MIT
