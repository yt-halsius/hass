# Effekttid – tariff, topp-3 & straffkostnad (Home Assistant)

Ett paket som:

* växlar **utility\_meter-tariff** med din `binary_sensor.effekttid` (säsong + vardag + TOD),
* mäter **timmedel** (kWh/h = kW) under effekttid,
* håller **Topp 3** timmedel,
* räknar **månadens medel** av topp-3 och **straffkostnad** = medel\*pris.

Formel: `((kw1 + kw2 + kw3) / 3) * pris = straffkostnad`

---

## Förutsättningar

* En binärsensor `binary_sensor.effekttid` som redan styr **när** effekttid gäller.
* En effektsensor i W från AMS **eller** en befintlig kWh-sensor.
* Home Assistant med **packages** aktiverat.

```yaml
# I configuration.yaml
homeassistant:
  packages: !include_dir_named packages
```

---

## Installation

1. Lägg filen `effekttid_tariff.yaml` i mappen `config/packages/`.
2. Öppna filen och BYT:

   * `sensor.ams_power` → din effektsensor i W (om du saknar kWh-sensor).
3. Kontrollera att `select.ams_energy_hourly_tm` finns (skapas av utility\_meter).
4. **Kontrollera konfigurationen** och starta om HA.

> Har du redan en kWh-sensor? Peka `utility_meter` på den och ta bort `sensor: platform: integration`.

---

## Vad paketet skapar

* **Utility Meter**: `sensor.ams_energy_hourly_tm_effekttid` / `…_ovrigt` (+ `select.ams_energy_hourly_tm`)
* **Hjälpare**:

  * `input_number.ams_top1_kw`, `ams_top2_kw`, `ams_top3_kw`
  * `input_datetime.ams_top1_time`, `…_top2_time`, `…_top3_time`
  * `input_number.effektavgift_kr_per_kw` (ex. 75 kr/kW)
* **Template-sensorer**:

  * `sensor.effekttopp_medel_kW_manad` (medel av topp-3)
  * `sensor.effektavgift_straffkostnad_kr_manad` (medel \* pris)
* **Automationer**:

  * Synkar tariff ⇄ `binary_sensor.effekttid` (använder `select.select_option`)
  * Uppdaterar **topp-3** varje hel timme från `last_period` (effekttid-tariffen)
  * (Valfri) Nollställer topp-3 vid månadsskifte

---

## Så funkar det (kort)

* Din `binary_sensor.effekttid` växlar **tariffen** till `effekttid/ovrigt`.
* `utility_meter` räknar kWh **per timme** och lägger timmen på rätt tariff.
* `last_period` för `…_effekttid` = **föregående timmes** kWh ⇒ timmedel i **kW**.
* Automation uppdaterar topp-3 och tiderna.
* `effektavgift_kr_per_kw` \* (medel av topp-3) = **straffkostnad**.

---

## Exempel (Lovelace)

```yaml
type: entities
title: Effekttid toppar
entities:
  - sensor.effekttopp_medel_kw_manad
  - sensor.effektavgift_straffkostnad_kr_man
  - input_number.ams_top1_kw
  - input_datetime.ams_top1_time
  - input_number.ams_top2_kw
  - input_datetime.ams_top2_time
  - input_number.ams_top3_kw
  - input_datetime.ams_top3_time
  - input_number.effektavgift_kr_per_kw
```

---

## Felsökning

* **”Integration error: packages”** → lägg `packages:` under `homeassistant:` (inte på toppnivå).
* **Tjänst saknas** → tariff-automation använder `select.select_option` (inte `utility_meter.select_tariff`).
* **Ingen data i `last_period`** → verifiera att `utility_meter` pekar på rätt kWh-sensor och att en timme passerat.
* **Fel entity-id** → kontrollera `select.ams_energy_hourly_tm` i Utvecklarverktyg → Tillstånd.

---

## Licens

MIT — använd, ändra och dela fritt.
