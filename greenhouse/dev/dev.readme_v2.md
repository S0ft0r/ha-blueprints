# 🌱 Автоматическое управление поливом теплиц

Blueprint для Home Assistant — полный цикл управления поливом двух теплиц
с баком-накопителем, прогревом воды и защитой от аварийных ситуаций.

---

## Возможности

- **Наполнение бака** с контролем уровня, таймаутом и обнаружением отсутствия роста
- **Прогрев воды** после каждого наполнения (настраиваемая длительность)
- **Полив двух зон** (огурцы и помидоры) по датчикам влажности почвы с гистерезисом
- **Одновременный полив** обеих зон с независимым завершением каждой
- **Три режима работы**: Авто / Обслуживание / Стоп
- **Вечернее окно полива** — полив только в заданный интервал времени
- **Заявки на полив** — формируются днём, выполняются вечером
- **Возобновление полива** после пустого бака → наполнения → прогрева
- **Проверка приводов** — повторная попытка при несрабатывании крана
- **Аварийная остановка** при критических ошибках
- **Восстановление после рестарта** Home Assistant и пропадания питания
- **Диагностика** — предупреждение, если полив идёт, но влажность не растёт
- **Гибкие настройки** — каждый параметр задаётся статически или через helper-сущность

---

## Архитектура

```
┌─────────────────────────────────────────────────┐
│                  Blueprint                       │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐  │
│  │ Триггеры │→ │ Условия  │→ │   Действия    │  │
│  │ (23 шт)  │  │          │  │ (YAML-якоря)  │  │
│  └──────────┘  └──────────┘  └───────────────┘  │
│        ↕              ↕              ↕           │
│  ┌───────────────────────────────────────────┐   │
│  │         Helper-сущности (внешние)         │   │
│  │  input_select · input_boolean · timer     │   │
│  │  input_datetime · input_number            │   │
│  └───────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

## Требования

- Home Assistant 2024.6 или новее (поддержка `selector: choose`)
- Датчик уровня воды в баке (0–100%)
- Два датчика влажности почвы
- Четыре управляемых крана/клапана (switch или valve):
  - магистральный клапан
  - кран набора воды в бак
  - кран полива огурцов
  - кран полива помидоров

---

## Установка

### 1. Создайте helper-сущности

Все сущности ниже **обязательны** — blueprint использует их
для хранения состояния системы в runtime.

Создайте их через **Настройки → Устройства и службы → Вспомогательные элементы**
или добавьте в `configuration.yaml`.

### 2. Импортируйте blueprint

[![Импорт Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/S0ft0r/ha-blueprints/main/greenhouse_v2/greenhouse_v2.yaml)

### 3. Создайте автоматизацию на основе blueprint

Настройки → Автоматизации → Создать автоматизацию → Использовать blueprint

---

## Helper-сущности

### Через UI (Настройки → Вспомогательные элементы)

Создайте каждую сущность вручную, используя таблицу ниже.

### Через `configuration.yaml`

Скопируйте полный блок конфигурации из раздела ниже.

---

### Сводная таблица

| Сущность | Тип | Назначение |
|---|---|---|
| `input_select.watering_system_mode` | input_select | Режим системы |
| `input_select.watering_tank_state` | input_select | Состояние бака |
| `input_select.watering_zone_cucumbers_state` | input_select | Состояние зоны огурцов |
| `input_select.watering_zone_tomatoes_state` | input_select | Состояние зоны помидоров |
| `input_boolean.watering_request_cucumbers` | input_boolean | Заявка на полив огурцов |
| `input_boolean.watering_request_tomatoes` | input_boolean | Заявка на полив помидоров |
| `input_boolean.watering_warmup_active` | input_boolean | Флаг активного прогрева |
| `input_boolean.watering_filled_in_maintenance` | input_boolean | Бак наполнялся в режиме обслуживания |
| `input_boolean.watering_error_system` | input_boolean | Общая ошибка системы |
| `input_boolean.watering_error_tank` | input_boolean | Ошибка бака |
| `input_boolean.watering_error_fill_timeout` | input_boolean | Таймаут наполнения |
| `input_boolean.watering_error_no_level_rise` | input_boolean | Нет роста уровня |
| `input_boolean.watering_error_zone_cucumbers` | input_boolean | Ошибка зоны огурцов |
| `input_boolean.watering_error_zone_tomatoes` | input_boolean | Ошибка зоны помидоров |
| `input_boolean.watering_error_cucumbers_timeout` | input_boolean | Таймаут полива огурцов |
| `input_boolean.watering_error_tomatoes_timeout` | input_boolean | Таймаут полива помидоров |
| `input_datetime.watering_warmup_end_time` | input_datetime | Время окончания прогрева |
| `timer.watering_warmup` | timer | Таймер прогрева |
| `timer.watering_fill_timeout` | timer | Таймер таймаута наполнения |
| `timer.watering_no_rise` | timer | Таймер отсутствия роста уровня |
| `timer.watering_cucumbers_timeout` | timer | Таймер таймаута полива огурцов |
| `timer.watering_tomatoes_timeout` | timer | Таймер таймаута полива помидоров |

---

### Код для `configuration.yaml`

```yaml
###############################################################################
# Полив теплиц — Helper-сущности
###############################################################################

# ===========================================================================
# Режимы и состояния (input_select)
# ===========================================================================

input_select:

  watering_system_mode:
    name: "Полив: режим системы"
    options:
      - auto
      - maintenance
      - stop
    initial: stop
    icon: mdi:cog

  watering_tank_state:
    name: "Полив: состояние бака"
    options:
      - error
      - empty
      - filling
      - warmup
      - ready
    initial: error
    icon: mdi:water-boiler

  watering_zone_cucumbers_state:
    name: "Полив: состояние зоны огурцов"
    options:
      - no_need
      - need_watering
      - waiting_conditions
      - watering
      - waiting_resume
      - zone_error
    initial: no_need
    icon: mdi:sprout

  watering_zone_tomatoes_state:
    name: "Полив: состояние зоны помидоров"
    options:
      - no_need
      - need_watering
      - waiting_conditions
      - watering
      - waiting_resume
      - zone_error
    initial: no_need
    icon: mdi:fruit-cherries

# ===========================================================================
# Заявки и флаги (input_boolean)
# ===========================================================================

input_boolean:

  # --- Заявки на полив ---

  watering_request_cucumbers:
    name: "Полив: заявка на полив огурцов"
    initial: false
    icon: mdi:water-plus

  watering_request_tomatoes:
    name: "Полив: заявка на полив помидоров"
    initial: false
    icon: mdi:water-plus

  # --- Прогрев и обслуживание ---

  watering_warmup_active:
    name: "Полив: прогрев бака активен"
    initial: false
    icon: mdi:thermometer

  watering_filled_in_maintenance:
    name: "Полив: бак наполнялся в режиме обслуживания"
    initial: false
    icon: mdi:water-alert

  # --- Флаги ошибок ---

  watering_error_system:
    name: "Полив: общая ошибка системы"
    initial: false
    icon: mdi:alert-circle

  watering_error_tank:
    name: "Полив: ошибка бака"
    initial: false
    icon: mdi:alert

  watering_error_fill_timeout:
    name: "Полив: таймаут наполнения бака"
    initial: false
    icon: mdi:timer-alert

  watering_error_no_level_rise:
    name: "Полив: нет роста уровня при наполнении"
    initial: false
    icon: mdi:trending-neutral

  watering_error_zone_cucumbers:
    name: "Полив: ошибка зоны огурцов"
    initial: false
    icon: mdi:alert

  watering_error_zone_tomatoes:
    name: "Полив: ошибка зоны помидоров"
    initial: false
    icon: mdi:alert

  watering_error_cucumbers_timeout:
    name: "Полив: таймаут полива огурцов"
    initial: false
    icon: mdi:timer-alert

  watering_error_tomatoes_timeout:
    name: "Полив: таймаут полива помидоров"
    initial: false
    icon: mdi:timer-alert

# ===========================================================================
# Время окончания прогрева (input_datetime)
# ===========================================================================

input_datetime:

  watering_warmup_end_time:
    name: "Полив: время окончания прогрева"
    has_date: true
    has_time: true
    icon: mdi:clock-end

# ===========================================================================
# Таймеры (timer)
# ===========================================================================

timer:

  watering_warmup:
    name: "Полив: таймер прогрева бака"
    restore: true
    icon: mdi:thermometer-lines

  watering_fill_timeout:
    name: "Полив: таймер таймаута наполнения"
    restore: true
    icon: mdi:timer-alert-outline

  watering_no_rise:
    name: "Полив: таймер отсутствия роста уровня"
    restore: true
    icon: mdi:trending-neutral

  watering_cucumbers_timeout:
    name: "Полив: таймер таймаута полива огурцов"
    restore: true
    icon: mdi:timer-alert-outline

  watering_tomatoes_timeout:
    name: "Полив: таймер таймаута полива помидоров"
    restore: true
    icon: mdi:timer-alert-outline
```

> **Примечание:** если вы уже используете `input_select:`, `input_boolean:`,
> `input_datetime:` или `timer:` в `configuration.yaml`, не дублируйте
> корневые ключи — добавьте сущности внутрь существующих блоков.

---

### Опциональные helper-сущности

Следующие сущности **не обязательны** — они нужны, только если вы хотите
менять параметры системы без редактирования автоматизации
(через выбор «сенсор» вместо «значение» в настройках blueprint).

```yaml
# ===========================================================================
# Опциональные числовые параметры (input_number)
# ===========================================================================

input_number:

  # --- Пороги бака ---

  watering_tank_level_empty:
    name: "Полив: порог пустого бака"
    min: 0
    max: 50
    step: 1
    initial: 10
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:gauge-empty

  watering_tank_level_min_working:
    name: "Полив: минимальный рабочий уровень"
    min: 5
    max: 50
    step: 1
    initial: 20
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:gauge-low

  watering_tank_level_auto_target:
    name: "Полив: целевой автоуровень"
    min: 30
    max: 100
    step: 1
    initial: 80
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:gauge

  watering_tank_level_full:
    name: "Полив: порог полного бака"
    min: 50
    max: 100
    step: 1
    initial: 95
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:gauge-full

  # --- Пороги влажности огурцов ---

  watering_cucumbers_moisture_low:
    name: "Полив: нижний порог влажности огурцов"
    min: 5
    max: 90
    step: 1
    initial: 40
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:water-minus

  watering_cucumbers_moisture_high:
    name: "Полив: верхний порог влажности огурцов"
    min: 10
    max: 95
    step: 1
    initial: 70
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:water-plus

  # --- Пороги влажности помидоров ---

  watering_tomatoes_moisture_low:
    name: "Полив: нижний порог влажности помидоров"
    min: 5
    max: 90
    step: 1
    initial: 35
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:water-minus

  watering_tomatoes_moisture_high:
    name: "Полив: верхний порог влажности помидоров"
    min: 10
    max: 95
    step: 1
    initial: 65
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:water-plus

  # --- Таймауты ---

  watering_warmup_duration:
    name: "Полив: длительность прогрева"
    min: 10
    max: 300
    step: 10
    initial: 120
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:thermometer-lines

  watering_fill_timeout:
    name: "Полив: таймаут наполнения"
    min: 5
    max: 180
    step: 5
    initial: 60
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:timer-alert-outline

  watering_no_rise_timeout:
    name: "Полив: таймаут отсутствия роста уровня"
    min: 2
    max: 30
    step: 1
    initial: 10
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:trending-neutral

  watering_zone_timeout:
    name: "Полив: таймаут полива зоны"
    min: 10
    max: 240
    step: 10
    initial: 90
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:timer-alert-outline

  watering_valve_action_timeout:
    name: "Полив: таймаут привода крана"
    min: 5
    max: 120
    step: 5
    initial: 30
    unit_of_measurement: "сек"
    mode: slider
    icon: mdi:valve

  watering_sensor_stale_timeout:
    name: "Полив: допустимое время без обновления датчика"
    min: 5
    max: 180
    step: 5
    initial: 60
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:clock-alert-outline

  # --- Дебаунс ---

  watering_tank_debounce:
    name: "Полив: подтверждение уровня бака"
    min: 5
    max: 120
    step: 5
    initial: 30
    unit_of_measurement: "сек"
    mode: slider
    icon: mdi:filter-outline

  watering_moisture_debounce:
    name: "Полив: подтверждение влажности"
    min: 1
    max: 30
    step: 1
    initial: 5
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:filter-outline

  # --- Диагностика ---

  watering_diag_check_after:
    name: "Полив: проверка после начала полива"
    min: 10
    max: 60
    step: 5
    initial: 30
    unit_of_measurement: "мин"
    mode: slider
    icon: mdi:stethoscope

  watering_diag_min_delta:
    name: "Полив: минимальный рост влажности"
    min: 1
    max: 20
    step: 1
    initial: 3
    unit_of_measurement: "%"
    mode: slider
    icon: mdi:delta

  # --- Рестарт ---

  watering_startup_delay:
    name: "Полив: задержка после старта HA"
    min: 10
    max: 300
    step: 10
    initial: 60
    unit_of_measurement: "сек"
    mode: slider
    icon: mdi:restart

# ===========================================================================
# Опциональное время вечернего окна (input_datetime)
# ===========================================================================

input_datetime:

  watering_window_start:
    name: "Полив: начало вечернего окна"
    has_date: false
    has_time: true
    initial: "18:00"
    icon: mdi:weather-sunset-up

  watering_window_end:
    name: "Полив: конец вечернего окна"
    has_date: false
    has_time: true
    initial: "22:00"
    icon: mdi:weather-sunset-down

# ===========================================================================
# Опциональные переключатели (input_boolean)
# ===========================================================================

input_boolean:

  watering_logging_enable:
    name: "Полив: логирование"
    initial: true
    icon: mdi:math-log

  watering_diag_enable:
    name: "Полив: диагностика эффективности полива"
    initial: true
    icon: mdi:stethoscope
```

---

## Режимы работы

### 🟢 Авто

Система самостоятельно:
- отслеживает влажность почвы и формирует заявки
- контролирует уровень бака
- наполняет бак и отрабатывает прогрев
- запускает и останавливает полив в вечернем окне
- возобновляет полив после повторного наполнения и прогрева

### 🟡 Обслуживание

Автоматика заблокирована. Пользователь вручную может:
- открывать и закрывать любые краны
- наполнять бак
- вносить удобрения

> **Важно:** после ручного наполнения и возврата в «Авто»
> бак автоматически переводится в прогрев.

### 🔴 Стоп

Все автоматические действия запрещены.
При переключении в этот режим все краны принудительно закрываются.

---

## Логические состояния

### Состояния бака

| Состояние | Описание | Полив разрешён |
|---|---|---|
| `error` | Датчик уровня недоступен или некорректен | ❌ |
| `empty` | Уровень ниже порога «пустой» | ❌ |
| `filling` | Идёт наполнение | ❌ |
| `warmup` | Прогрев после наполнения | ❌ |
| `ready` | Бак готов к поливу | ✅ |

### Состояния зон полива

| Состояние | Описание |
|---|---|
| `no_need` | Влажность выше верхнего порога |
| `need_watering` | Влажность ниже нижнего порога, заявка создана |
| `waiting_conditions` | Заявка есть, но условия не выполнены |
| `watering` | Кран открыт, полив идёт |
| `waiting_resume` | Полив прерван (бак пустой), ожидание возобновления |
| `zone_error` | Ошибка датчика или таймаут полива |

---

## Основные сценарии

### Штатный цикл полива

```
Влажность ↓ → Заявка → Проверка бака → Наполнение →
→ Прогрев (2ч) → Вечернее окно → Полив →
→ Влажность ↑ → Заявка снята → Кран закрыт
```

### Бак опустел во время полива

```
Уровень ↓ → Краны полива закрыты → Заявки сохранены →
→ Наполнение → Прогрев → Возобновление полива
(если вечернее окно ещё не закрылось)
```

### Обслуживание с удобрением

```
Режим → Обслуживание → Ручное наполнение →
→ Добавление удобрений → Ручной полив →
→ Режим → Авто → Прогрев (2ч) →
→ Штатная работа
```

### Перезагрузка HA

```
Старт HA → Задержка (ожидание Zigbee) →
→ Принудительное закрытие всех кранов →
→ Пересчёт состояния (уровень, прогрев, заявки) →
→ Штатная работа
```

---

## Защита от ошибок

| Ошибка | Реакция |
|---|---|
| Датчик уровня бака недоступен | Аварийная остановка, все краны закрыты |
| Датчик влажности зоны недоступен | Полив этой зоны остановлен, другая зона работает |
| Кран не сработал | Повторная попытка, затем ошибка и остановка |
| Таймаут наполнения бака | Краны набора закрыты, ошибка |
| Нет роста уровня при наполнении | Краны набора закрыты, ошибка |
| Таймаут полива зоны | Кран зоны закрыт, ошибка |
| Пропадание питания | Безопасный рестарт (см. выше) |

---

## Диагностика

При включённой опции «Диагностика эффективности полива» система проверяет,
растёт ли влажность почвы после начала полива. Если за заданное время
влажность не выросла на ожидаемое значение — формируется предупреждение.

Возможные причины:
- засор капельной линии
- вода не доходит до зоны датчика
- неисправность датчика
- проблема с самотёком или давлением

---

## Настройка параметров

Каждый числовой параметр (пороги, таймауты, задержки) можно задать двумя способами:

| Способ | Описание |
|---|---|
| **Значение** | Задаётся прямо в blueprint, изменяется только через редактирование автоматизации |
| **Сенсор** | Указывается helper-сущность (`input_number`, `input_datetime`, `input_boolean`), значение можно менять через UI или другие автоматизации |

---

## Лицензия

[Отказ от ответственности](https://github.com/S0ft0r/ha-blueprints/blob/main/as-is.md)
```

