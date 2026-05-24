# Solar Flow Widget for ioBroker VIS

A visualization of live energy flows between solar production, battery, grid, house consumption and a charging car. Built as HTML/SVG widgets for use in ioBroker VIS.

![solFlowWidget](https://www.photovoltaikforum.com/core/attachment/244127-solar-widget-gif/)

---

## Widgets Overview

The project contains two independent widgets, each placed as a separate element on a VIS view.

### 1. `solFlowWidget.html` — Main Flow Widget
Shows the full energy flow diagram: solar → inverter → battery / grid / house / car, with animated flow lines between all nodes.

### 2. `solFlowWidgetCarDetails/` — Car Detail Widget
A focused horizontal view showing the wallbox (Wattpilot) connected to the active car, with a cable animation, car battery level and charging power. Split into two files:
- `solFlowCarDetails.html` — SVG/HTML structure only
- `carDetails_Script.html` — VIS script widget that drives the SVG

---

## Multi-Car Support

Both widgets support switching between two cars based on a single ioBroker state.

### How it works

An **ioBroker JavaScript script** (separate from these widgets) monitors which car is plugged into the wallbox and writes a car key — `'enyaq'` or `'id3'` — into the state `javascript.0.car.pluggedCar`.

The widgets read this state on every update cycle and automatically switch the car image and the displayed SoC to match the active car.

### Car configuration — Main Widget (`solFlowWidget.html`)

Cars are defined at the top of the script section:

```js
const carConfig = {
    enyaq: {
        name:  'Enyaq',
        image: '/vis.0/Strom/img/enyaq.png',
        socId: 'vw-connect.0.TMBJC7NY9NF038592.status.charging.status.battery.stateOfChargeInPercent',
        color: '#78828C'
    },
    id3: {
        name:  'ID3',
        image: '/vis.0/Strom/img/id3.png',
        socId: 'vw-connect.1.WVWZZZE1ZPP028522.status.batteryStatus.currentSOC_pct',
        color: '#78828C'
    }
};
```

To add a third car: add a new entry. The key must exactly match what the ioBroker script writes to `javascript.0.car.pluggedCar` (case-sensitive).

### Car configuration — Car Detail Widget (`carDetails_Script.html`)

Because the detail widget is a VIS script widget, object literals `{…}` cannot be used — VIS treats every `{…}` in the script text as a state binding to subscribe to. Car images are therefore configured as plain ternaries:

```js
let carImage = (activeCar === 'id3') ? '/vis.0/Strom/img/id3.png' : '/vis.0/Strom/img/enyaq.png';
```

Both car SoC states are always bound so VIS re-triggers the script when either car updates:

```js
let soc_enyaq = {vw-connect.0.TMBJC7NY9NF038592...};
let soc_id3   = {vw-connect.1.WVWZZZE1ZPP028522...};
let carCarChSt = (activeCar === 'id3') ? soc_id3 : soc_enyaq;
```

---

## Main Widget — How It Works

### State IDs

All ioBroker state IDs are defined in one named object at the top of the script. No index-based array lookups:

```js
const stateIds = {
    prod:      'javascript.0.solar.current_production',
    bat:       'javascript.0.solar.toBattery',
    grid:      'modbus.0.holdingRegisters.200.40087_W',
    house:     'javascript.0.solar.curConsumptionHouse',
    carPower:  'fronius-wattpilot.0.power',
    batSoc:    'modbus.0.holdingRegisters.1.40351_ChaState',
    batStatus: 'modbus.0.holdingRegisters.1.40354_ChaSt',
    activeCar: 'javascript.0.car.pluggedCar'
};
```

### Data refresh

`getServerValues()` fetches all states — including both car SoC IDs — in a single `servConn.getStates()` call every 2 seconds. The active car is resolved from the `activeCar` state; the correct SoC is picked from the result. If `activeCar` is not set, the first car in `carConfig` is used as fallback.

### Startup sequence

When the widget loads, these functions must run in order:

1. `initSVG()` — positions and unhides all SVG groups and text elements
2. `addAnimCircles(3)` — clones animation circle SVGs directly into the DOM (required because `<use>` creates shadow DOM copies whose `animateMotion` paths cannot be updated via `getElementById`)
3. `getServerValues()` — first data fetch
4. `setInterval(getServerValues, 2000)` — refresh every 2 seconds

### SVG structure

Each node (solar, grid, battery, house, car, inverter) is defined as a `<g>` symbol in `<defs>` and placed on the canvas via `<use>`. The car node uses `id="carImage"` on its `<image>` element so `updateCarDisplay()` can swap the image href dynamically. The car circle uses `fill="none"` so the image is visible through it as a background.

---

## Car Detail Widget — How It Works

### Two-widget pattern

VIS allows placing two widgets in the same container. The SVG widget (`solFlowCarDetails.html`) holds only visual structure. The script widget (`carDetails_Script.html`) holds all logic and manipulates the SVG DOM directly via `getElementById`.

### VIS script widget constraints

VIS evaluates `{state.id}` bindings in script widget text before executing the JavaScript. This means:
- Any `{…}` on a single line — including JS object literals and single-line if/else bodies — is scanned and potentially replaced
- **Never use single-line `{…}` blocks** in VIS script widgets. Use ternary operators or multi-line blocks instead
- Object literals `{ key: value }` must be avoided entirely — use plain variables

### Connection state

The cable between wallbox and car switches between two SVG symbols (`#carConnectionConnected` / `#carConnectionBroken`) based on the `fronius-wattpilot.0.carConnected` state.

---

## Required Images

Images are stored in `/vis.0/Strom/img/` on the ioBroker file system:

| File | Used for |
|---|---|
| `enyaq.png` | Enyaq car icon |
| `id3.png` | ID3 car icon |
| `solar.png` | Solar production node |
| `haus.png` | House consumption node |
| `grid.png` | Grid node |
| `wechselrichter.png` | Inverter node |
| `wattpilot.png` | Wallbox node (car detail widget) |
