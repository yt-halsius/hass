# README – Effektsäsong & Effekttid (Home Assistant)

## Vad du får

* **`binary_sensor.effektavgift_period`** – *Säsongssensor* som är `on` mellan **1 nov 07:00** och **31 mar 19:00** (lokal tid). Har attribut som start/slut, nästa skifte och nedräkning.
* **`binary_sensor.effekttid_vardagar`** – *Villkorad sensor* som är `on` **endast** när:

  * säsongen är aktiv (`effektavgift_period` = `on`)
  * det är **arbetsdag** (Workday)
  * tiden är inom ditt **Time-of-Day**-fönster (TOD)

## Beroenden (externa entiteter)

För **Effekttid** behövs två externa sensorer (plattformar i HA):

1. **Workday** – svensk arbetsdag inkl. helgdagar
   Skapar `binary_sensor.workday_sensor`:

```yaml
binary_sensor:
  - platform: workday
    name: "Workday Sensor"
    country: SE
    workdays: [mon, tue, wed, thu, fri]
    excludes: [holiday]
```

2. **Time of Day (tod)** – tidfönster
   Skapar `binary_sensor.effekttid_fonster`:

```yaml
binary_sensor:
  - platform: tod
    name: "Effekttid fönster"
    after: "07:00"
    before: "19:00"
# Om fönstret passerar midnatt: after: "19:00", before: "07:00"
```

> Säsongssensorn `effektavgift_period` är en **template** och har inga externa beroenden.

## Installation (templates)

Aktivera mapp-inkludering:

```yaml
template: !include_dir_merge_list templates
```

Lägg följande filer i `config/templates/`:

### 1) Effektavgift säsong

`templates/effekttid.yaml`

```yaml
- binary_sensor:
    - name: "Effektavgift period"
      unique_id: effektavgift_period
      icon: mdi:flash-alert
      state: >-
        {# ... din befintliga state-mall för 1/11 07:00 – 31/3 19:00 ... #}
      attributes:
        {# ... start_date/start_time/end_date/end_time/next_change/seconds_until_change/time_until_change ... #}
```

### 2) Villkorad effektavgift (Säsong + Workday + TOD)

`templates/effekttid-guarded.yaml`

```yaml
- binary_sensor:
    - name: "Effekttid"
      unique_id: effekttid_vardagar
      icon: mdi:flash-alert-outline
      state: >-
        {{ is_state('binary_sensor.effektavgift_period','on')
           and is_state('binary_sensor.workday_sensor','on')
           and is_state('binary_sensor.effekttid_fonster','on') }}
      attributes:
        effektavgift_period: "{{ states('binary_sensor.effektavgift_period') }}"
        arbetsdag: "{{ states('binary_sensor.workday_sensor') }}"
        tod_state: "{{ states('binary_sensor.effekttid_fonster') }}"
```

## Användning

* Använd **`binary_sensor.effektavgift_period`** för att veta om du befinner dig i effektsäsong (med attribut för nästa skifte).
* Använd **`binary_sensor.effekttid_vardagar`** som villkor i automationer där **säsong + vardag + tidfönster** krävs.

> Efter ändringar: **Inställningar → System → YAML-konfiguration → Läs om mall-entiteter** (eller starta om).
> Om dina entitetsnamn skiljer sig, uppdatera dem i templaten ovan.
