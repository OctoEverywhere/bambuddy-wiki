---
title: Energy Tracking
description: Monitor power consumption and calculate electricity costs
---

# Energy Tracking

Track power consumption from your 3D printers and calculate electricity costs per print or cumulatively.

---

## :material-lightning-bolt: Overview

Energy tracking helps you understand:

- **Power consumption** per print
- **Electricity costs** over time
- **Most efficient** printers and settings
- **Total energy** usage

---

## :material-power-plug: Requirements

Energy tracking requires a smart plug with power monitoring:

### Supported Smart Plugs

| Type | Power Monitoring | Notes |
|------|:----------------:|-------|
| **Tasmota** | :material-check: | Direct HTTP API |
| **Home Assistant** | :material-check: | Via HA sensors |
| **REST / Webhook** | :material-check: | Via JSON path extraction from status endpoint |
| **MQTT** | :material-check: | Via MQTT topic subscription |

!!! info "Home Assistant Energy Sensors"
    Many HA plugs expose energy data as separate sensor entities. Configure these in the plug settings under **Energy Monitoring (Optional)**. REST/Webhook and MQTT plugs extract energy data from JSON responses/messages using configurable paths. See [Smart Plugs](smart-plugs.md) for setup.

---

## :material-cog: Configuration

### Setting Up Energy Tracking

1. Configure a smart plug for your printer (Tasmota, Home Assistant, REST/Webhook, or MQTT)
2. Go to **Settings** > **Smart Plugs**
3. Assign the plug to a printer
4. For HA plugs: Configure energy sensor entities if needed
5. Energy tracking starts automatically

### Tracking Mode

Choose how energy is tracked:

| Mode | Description |
|------|-------------|
| **Per Print** | Track energy for each individual print |
| **Total** | Track cumulative energy usage |

Configure in **Settings** > **General**.

---

## :material-meter-electric: Energy Data

### Per-Print Tracking

When enabled, each print records:

| Data | Description |
|------|-------------|
| **Energy (Wh)** | Watt-hours consumed |
| **Peak Power (W)** | Maximum power draw |
| **Avg Power (W)** | Average during print |
| **Cost** | Calculated electricity cost |

### Archive Display

Energy data appears on archive cards:

```
⚡ 245 Wh • $0.04
```

---

## :material-chart-line: Energy Statistics

### Dashboard Widget

The Energy widget shows:

- **Total consumption** for period
- **Cost breakdown** by printer
- **Trend over time**
- **Comparison** between printers

Per-print energy also feeds the [Statistics page](statistics.md): the **Filament Trends** widget gains an **Energy Over Time** chart when measured data exists in the selected range, and the **Most Expensive** record ranks prints by filament + energy cost.

### Energy by Printer

| Printer | Total Energy | Avg per Print | Cost |
|---------|:------------:|:-------------:|:----:|
| Workshop X1C | 15.2 kWh | 380 Wh | $2.28 |
| Office P1S | 8.5 kWh | 425 Wh | $1.28 |

---

## :material-cash: Cost Calculation

### Setting Electricity Rate

1. Go to **Settings** > **General** > **Cost Tracking**
2. Choose the **Electricity price source**:
    - **Fixed price**: enter your rate per kWh under **Electricity cost per kWh**
    - **Home Assistant sensor**: pick the sensor that holds your current price, see [Dynamic Electricity Rates](#dynamic-electricity-rates)
3. Choose currency

### Cost Formula

```
Print Cost = (Energy in Wh / 1000) × Rate per kWh
```

**Example:**
```
245 Wh at $0.15/kWh = 0.245 × 0.15 = $0.037
```

### Energy Is Costed at the Price of the Hour It Was Used

Bambuddy records the electricity price with each hourly energy reading, and costs the energy used in each hour at that hour's price:

- **A print** that runs through a price change is costed hour by hour: its cheap hours at the cheap price and its expensive hours at the expensive one, not all of it at the price when it ends.
- **The Statistics page** in **Total** mode does the same for any date range and for the all-time figure.
- **Changing the price** applies to the energy used from then on. Energy already used keeps the price it was used at, so a new contract doesn't re-cost last year.

Energy used since the last hourly reading, and energy from plugs without a lifetime counter (MQTT plugs), is costed at the current price. Energy recorded before this feature existed keeps the price that was set when you upgraded.

---

## :material-clock-start: When Energy is Tracked

Energy tracking records:

| Phase | Tracked? |
|-------|:--------:|
| Preheating | :material-check: |
| Printing | :material-check: |
| Cooling | :material-check: |
| Idle (printer on, not printing) | Depends on mode |

### Per-Print Mode

Only tracks during active print:

- Starts when print begins
- Ends when print completes
- Includes preheating and cooling
- **Restart-resilient**: the starting plug counter is persisted on the archive row, so if the Bambuddy backend restarts during a print the per-print energy delta is still computed correctly at completion.

!!! info "More than one plug on a printer"
    You can link several plugs to the same printer — a dry box, a filter fan, a chamber light. Only one of them supplies the energy figure: Bambuddy prefers a plug marked **Powers the printer**, and of those the first that actually reports a meter reading. Accessory plugs without energy monitoring are skipped automatically, so no configuration is needed for the usual case. If a printer's energy stays empty, check that its own plug still has **Powers the printer** enabled.

### Total Mode

Tracks all energy when the plug is on — including idle, preheat, printing, cooldown, and standby. The value comes from the smart plug's lifetime counter (delivered via Tasmota/HA/REST), not from per-print deltas.

- Cumulative total reported as the plug's live lifetime counter
- Includes idle / standby / chamber heating
- Date-range views (Today / This Week / This Month / …) are supported via **hourly snapshots** of each plug's lifetime counter. For a date range, Bambuddy computes `lastSnapshot − baseline` per plug where *baseline* is the last snapshot at or before the range start.

!!! warning "Warming-up period"
    The hourly snapshot loop starts shortly after backend startup. On a fresh install — or immediately after upgrading to a version that ships snapshot support — there is not yet a baseline snapshot *before* every possible date range. In that case the Statistics page shows a yellow warning icon next to **Energy Used** / **Energy Cost**; hover over it to read the explanation. As soon as at least one snapshot exists before the selected range the warning disappears and values become accurate. No configuration or manual action is required — the system is simply collecting data.

---

## :material-file-export: Energy Export

Export energy data:

1. Go to **Statistics**
2. Click **Export**
3. Energy data included in CSV/Excel

Export includes:

- Per-print energy consumption
- Cost calculations
- Printer comparison data

---

## :material-chart-bar: Energy Analysis

### Efficient Settings

Compare energy usage across prints to find efficient settings:

| Setting | Energy Impact |
|---------|---------------|
| Lower temps | Less energy |
| Faster prints | Less energy (shorter runtime) |
| PLA vs ABS | ABS uses more (higher temps) |
| Enclosed chamber | Uses more (chamber heating) |

### Printer Efficiency

Compare printers:

- Some are more efficient than others
- Consider energy when choosing which printer to use
- Factor into total cost of ownership

---

## :material-poll: Total Cost of Printing

Combine energy costs with filament costs:

```
Total Print Cost = Filament Cost + Electricity Cost
```

**Example:**
```
Filament: 45g at $25/kg = $1.13
Electricity: 245 Wh at $0.15/kWh = $0.04
───────────────────────────────────
Total: $1.17
```

---

## :material-lightbulb: Tips

!!! tip "Accurate Rates"
    Use your actual electricity rate from your utility bill, including all fees and taxes.

!!! tip "Compare Printers"
    Track energy to understand which printer is most cost-effective for different jobs.

!!! tip "Factor in Failures"
    Failed prints still consume energy - another reason to optimize for reliability.

!!! tip "Peak Hours"
    If your electricity has time-of-use rates, schedule prints during off-peak hours.

!!! tip "Standby Power"
    Consider auto power-off to eliminate standby energy consumption.

---

## :material-home-automation: Dynamic Electricity Rates

If you have a dynamic electricity tariff (e.g., Amber, Tibber, Octopus Energy, Nord Pool), Bambuddy can take the price from Home Assistant. Each hour of energy is then costed at the price of that hour, see [Energy Is Costed at the Price of the Hour It Was Used](#energy-is-costed-at-the-price-of-the-hour-it-was-used).

### Read the Price Sensor (Recommended)

Bambuddy reads the price straight from a Home Assistant sensor, using the [Home Assistant connection](smart-plugs.md) you already set up for smart plugs:

1. Make sure Home Assistant is connected under **Settings** > **Network** > **Home Assistant**
2. Go to **Settings** > **General** > **Cost Tracking**
3. Set **Electricity price source** to **Home Assistant sensor**
4. Under **Price sensor**, pick or type the sensor holding your current price, e.g. `sensor.amber_general_price`

Bambuddy reads the sensor when you save, every hour, and at the start and end of every print. The price field then shows the last price read and can't be edited. If Home Assistant can't be reached, Bambuddy keeps using the last price it read.

!!! info "Sensor units"
    The sensor must give the price per kWh in your Bambuddy currency, e.g. `AUD/kWh` or `EUR/kWh`. Prices per MWh (common for Nord Pool and ENTSO-E day-ahead sensors) and per Wh are converted automatically. A sensor in cents (`c/kWh`, `ct/kWh`, `p/kWh`) needs a [template sensor](https://www.home-assistant.io/integrations/template/) that divides by 100. A negative price is costed as free.

!!! tip "Time-of-use tariffs without an API"
    If your provider has fixed peak and off-peak hours but no integration, create a template sensor in Home Assistant that returns the right price for the time of day, and pick that.

### Push the Price from Home Assistant

The older alternative, still supported: Home Assistant pushes the price to Bambuddy's API whenever it changes. Leave **Electricity price source** on **Fixed price** for this; the pushed price applies from the moment it arrives.

#### 1. Create an API Key

1. Go to **Settings** > **API Keys**
2. Create a key and tick the **Update electricity price** permission (this is the only settings field writable via API key — see [API Keys](api-keys.md#available-permissions) for why)
3. Copy the key

#### 2. Add REST Command to Home Assistant

Add to your `configuration.yaml`:

```yaml
rest_command:
  bambuddy_electricity_price:
    url: "http://YOUR_BAMBUDDY_IP:8000/api/v1/settings/electricity-price"
    method: POST
    headers:
      X-API-Key: "YOUR_API_KEY"
    content_type: "application/json"
    payload: '{"energy_cost_per_kwh": {{ states("sensor.electricity_price") }}}'
```

Replace:

- `YOUR_BAMBUDDY_IP` with your Bambuddy server address
- `YOUR_API_KEY` with the API key from step 1
- `sensor.electricity_price` with your energy provider's price sensor

!!! warning "Old `PATCH /settings` example no longer works"
    Bambuddy versions before v0.2.6 documented this integration using
    `PATCH /api/v1/settings`, but that route was always denied for API
    keys (it can rewrite SMTP/LDAP credentials) and the call would 403.
    The dedicated `POST /api/v1/settings/electricity-price` endpoint
    introduced in [#1356](https://github.com/maziggy/bambuddy/issues/1356)
    is the correct path — if your automation predates this fix, update
    both the URL and the method.

#### 3. Create Automation

```yaml
automation:
  id: 'bambuddy_push_electricity_price'
  alias: "Update Bambuddy Electricity Price"
  mode: restart
  trigger:
    - platform: state
      entity_id: sensor.electricity_price
      for: "00:00:05"
  condition:
    - condition: template
      value_template: >
        {{ states('sensor.electricity_price')|float(none) is not none }}
  action:
    - delay: "00:00:01"
    - service: rest_command.bambuddy_electricity_price
```

This updates the price in Bambuddy whenever your electricity rate changes.

### Common Energy Provider Sensors

| Provider | Typical Sensor |
|----------|----------------|
| Tibber | `sensor.tibber_prices` |
| Octopus Energy | `sensor.octopus_energy_electricity_current_rate` |
| Nordpool | `sensor.nordpool_kwh_*` |

!!! tip "Check Your Sensor"
    Verify your sensor returns a numeric value (e.g., `0.25`) not a string with currency symbol.
