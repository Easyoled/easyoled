### 4.11 Sensors & GPS

#### SENSOR_LIST

**Purpose:** List available sensors on the phone.

**Response:** `SENSOR_LIST:light,accel,gyro,mag,prox,step,pressure,battery,gps`

---

#### SENSOR_INFO

**Purpose:** Return sensor details line-by-line.

**Format:** `SENSOR_INFO:name,type=N,vendor=X,unit=Y,range=min..max`
**End:** `SENSOR_INFO:DONE`

---

#### SENSOR_START

**Purpose:** Start a sensor.

**Syntax:**
```
SENSOR_START,name,intervalMs
```

**Supported names:**
- `light` → 0-3000 lux → `SENSOR_DATA:light,<lux>`
- `accel` → -39.2..39.2 m/s² → `SENSOR_DATA:accel,<x>,<y>,<z>`
- `gyro` → -34.9..34.9 rad/s → `SENSOR_DATA:gyro,<x>,<y>,<z>`
- `mag` → -200..200 µT → `SENSOR_DATA:mag,<x>,<y>,<z>`
- `prox` → 0-5 cm → `SENSOR_DATA:prox,<cm>`
- `step` → step count → `SENSOR_DATA:step,<steps>`
- `pressure` → 300-1100 hPa → `SENSOR_DATA:pressure,<hPa>`
- `battery` → 0-100 % → `SENSOR_DATA:battery,<percent>`
- `gps` → lat,lon,alt,acc → `SENSOR_DATA:gps,<lat>,<lon>,<alt>,<acc>`

**Interval:** 50-60000 ms.
**Rule:** Only one sensor active at a time. New SENSOR_START stops previous.

**Response:** `ACK_OK:SENSOR_START:name:interval`
**Related:** `SENSOR_STOP`, `SENSOR_LIST`, `SENSOR_INFO`

---

#### SENSOR_STOP

**Purpose:** Stop sensor.

**Syntax:**
```
SENSOR_STOP,name     (or omit name for ALL)
```

**Response:** `ACK_OK:SENSOR_STOP:name` or `ACK_OK:SENSOR_STOP:ALL`

---

#### GPS_STATUS

**Purpose:** Query GPS state.

**Response:** `GPS_STATUS:on|off:streaming|idle`

**Note:** GPS streaming starts with `SENSOR_START,gps,<ms>`. No separate GPS_START command.

---

#### GPS_ENABLE

**Purpose:** Open system location settings.

**Response:** `ACK_OK:GPS_ENABLE`

---

### 4.12 VU Meter (Internal Visualizer)

#### VU_SOURCE

**Purpose:** Bind audio source to VU bars (internal Visualizer API).

**Syntax:**
```
VU_SOURCE,barRmsId,barPeakId
```

- `-1` = that bar unused
- Bar1: fast RMS (beats)
- Bar2: soft RMS (volume)

**Notes:** RECORD_AUDIO permission required. Bars must be `SEGMENTBAR`.

---

#### VU_START

**Purpose:** Manually start VU system.

---

#### VU_STOP

**Purpose:** Stop VU system.

---

#### VU_GAIN

**Purpose:** Set VU sensitivity.

**Syntax:**
```
VU_GAIN,value     (0.05-2.0)
```

- Low (0.2) → only high-volume moves
- Default (0.35) → balanced
- High (0.7) → moves even at low volume

---

#### VU_STATUS

**Purpose:** Query VU state.

**Response:** `VU_STATUS:RUNNING:PLAYER=MP3:RMS=5:PEAK=6`

---

### 4.13 Matrix

#### MATRIX_ROW

**Purpose:** Write one matrix row (bit mask).

**Syntax:**
```
MATRIX_ROW,id,rowIndex,data
```

**Data formats:**
- Binary string: `"11110000"`
- Hex: `"0xF0"`
- Decimal: `"240"`

**Example:**
```
MATRIX_ROW,1,0,1111000011110000
```

---

#### MATRIX_CLEAR

**Purpose:** Clear matrix.

**Syntax:**
```
MATRIX_CLEAR,id
```

---

### 4.14 Group

#### GROUP_MOVE

**Purpose:** Move group.

**Syntax:**
```
GROUP_MOVE,id,x,y
```

---

#### GROUP_BGCOLOR

**Purpose:** Change group background color.

**Syntax:**
```
GROUP_BGCOLOR,id,color,alpha
```

---

#### GROUP_ADD

**Purpose:** Add child object to group.

**Syntax:**
```
GROUP_ADD,groupId,childType,childParams...
```

**Example:**
```
GROUP_ADD,1,RECT,5,100,100,150,100,1,#FF0000
```

---

#### GROUP_REMOVE

**Purpose:** Remove child from group.

**Syntax:**
```
GROUP_REMOVE,groupId,childId
```

---

### 4.15 ListBox

#### LISTBOX_SET_ITEMS

**Purpose:** Update ListBox items.

**Syntax:**
```
LISTBOX_SET_ITEMS,id,item1|item2|item3
```

---

#### LISTBOX_SET_INDEX

**Purpose:** Update ListBox selected index.

**Syntax:**
```
LISTBOX_SET_INDEX,id,index
```

---

### 4.16 Fingerprint

#### FP_VERIFY

**Purpose:** Start fingerprint verification.

**Syntax:**
```
FP_VERIFY,id
```

**Response:** `FP_VERIFY_OK:id` or `FP_VERIFY_FAIL:id:reason`
**Reasons:** `USER_CANCEL`, `LOCKOUT`, `TIMEOUT`, `NO_HARDWARE`, `NO_ENROLLED`, `BUSY`

---

#### FP_STATUS

**Purpose:** Query fingerprint availability.

**Response:** `FP_STATUS:AVAILABLE:ENROLLED` or `FP_STATUS:UNAVAILABLE:reason`

---

#### FP_CANCEL

**Purpose:** Cancel active fingerprint request.

**Syntax:**
```
FP_CANCEL,id
```

**Response:** `FP_VERIFY_FAIL:id:FORCE_CANCEL`

---

### 4.17 Bluetooth

#### BT_STATUS

**Purpose:** Query Bluetooth adapter state.

**Response:** `BT_STATUS:ON` or `BT_STATUS:OFF`

---

#### BT_SEND

**Purpose:** One-shot send to a specific MAC address.

**Syntax:**
```
BT_SEND,mac,message
```

**Response:** `ACK_OK:BT_SEND:mac` or `ERR:BT_SEND:reason`

---

### 4.18 Bluetooth Gateway

**File:** `bt_devices.txt` (root folder).

**Format:**
```
# type,number,MAC,name,read_cmd,timeout_ms,display_id
classic,12,AA:BB:CC:DD:EE:01,Salon,READ,1000,103
```

Types: `classic` (v3.6 active) or `ble` (v3.6 parsed but inactive).

---

#### BL_READ_ALL

**Purpose:** Read all devices in bt_devices.txt.

**Response:** `BL_DATA,num1,val1,num2,val2,...` and `BL_DONE,ok,err`

---

#### BL_READ

**Purpose:** Read specific device.

**Syntax:**
```
BL_READ,n
```

**Response:** `BL_DATA,n,val` + `BL_DONE,1,0`

---

#### BL_SEND

**Purpose:** Send custom command to specific device.

**Syntax:**
```
BL_SEND,n,cmd
```

**Response:** `BL_SENT,n,ack`

---

#### BL_TIMER

**Purpose:** Periodic read timer.

**Syntax:**
```
BL_TIMER,sec      (0=stop, 1-3600)
```

**Response:** `BL_TIMER_ACK,sec`

---

#### BL_TIMER_STOP

**Purpose:** Stop timer.

**Response:** `BL_TIMER_ACK,STOPPED`

---

#### BL_LIST

**Purpose:** Send registered devices to Arduino.

**Response:** `BL_LIST_START,count` + `BL_LIST_ITEM,num,mac,name,displayId` × N + `BL_LIST_END`

---

#### BL_RELOAD

**Purpose:** Reload `bt_devices.txt`.

**Response:** `BL_RELOAD_ACK,count`

---

#### BL_STATUS

**Purpose:** Gateway status report.

**Response:** `BL_STATUS_OK,TIMER=sec,DEVICES=n,QUEUE=n,STATE=IDLE|BUSY`

---

### 4.19 Menu

#### HIDE_MENU

**Purpose:** Hide top-right ☰ menu.

**Syntax:**
```
HIDE_MENU,mode     (mode=2 default)
```

**Response:** `ACK_OK:HIDE_MENU:mode`
**Note:** Restore via double-tap on empty area or `SHOW_MENU`.

---

#### SHOW_MENU

**Purpose:** Re-show hidden ☰ menu.

**Response:** `ACK_OK:SHOW_MENU`

---

### 4.20 Alarms

#### ALARM_MESSAGE

**Purpose:** Show alarm message (finds page with sensor object, opens it with red background).

**Syntax:**
```
ALARM_MESSAGE,id,sensorId,title,text,button1,button2,color,bgColor,timeout
```

**Params:**
- `id` (int, 1) — message object id
- `sensorId` (int, 1) — target sensor object
- `title` (str, "ALARM")
- `text` (str, "Acil durum!")
- `button1` (str, "Durdur")
- `button2` (str, "") — can be empty
- `color` (color, `#FF0000`)
- `bgColor` (color, `#000000`)
- `timeout` (int, 0) — 0=no timeout, >0=ms

**Response:** `MSG_CLICK:id:button` or `MSG_TIMEOUT:id`

---

## 5. Events (EasyOLED → Arduino)

Full event list sent from the app to Arduino (each terminated with `\n`).

### Touch / Interaction

| Message | Description |
|---------|-------------|
| `BTN_CLICK:page,id,label` | Button pressed |
| `OBJ_INFO:PAGE=x,ID=y,TYPE=z` | Long press (2 s) |
| `POT_CHANGE:id:value` | Potentiometer turned |
| `ENCODER:id:+N` / `ENCODER:id:-N` | Encoder rotated |
| `SLIDER_CHANGE:id:value` | Slider dragged |
| `SWITCH_CHANGE:id:state` | Switch toggled |
| `RADIO_CHANGE:id:state` | Radio button toggled |
| `RADIO_CHANGE:id:index:name:url` | Internet radio station selected |
| `TRISTATE_CHANGE:id:state` | Tristate switched |
| `LISTBOX_CHANGE:id:index:text` | ListBox item selected |
| `LISTBOXR_SEL:id:index:label` | ListBoxR row selected |
| `COMBOBOX_CHANGE:id:index:text` | ComboBox item selected |
| `GROUP_CLICK:id` | Group tapped |
| `RECTRND_CLICK:id:value` | RectRnd tapped (new random) |
| `CLOCK_CLICK:HH:MM` | Analog clock tapped |
| `CIRCLE_LABEL:id:newLabel` | Circle label updated |
| `TRIANGLE_LABEL:id:newLabel` | Triangle label updated |
| `IMAGE_ROTATE:id:angle` | Image rotation updated |
| `GAUGE_CHANGE:id:value` | Gauge value changed |
| `RULER_CHANGE:id:value` | Ruler value changed |
| `THERMOMETER_CHANGE:id:value` | Thermometer value changed |
| `BATTERY_CHANGE:id:level` | Battery level changed |
| `PROGRESS_CHANGE:id:value` | Progress value changed |
| `SEGMENTBAR_CHANGE:id:value` | SegmentedBar value changed |
| `HIT:imageMapId:label` | ImageMap hit region tapped |
| `R30G50B150W0` | RGBPicker color selection |
| `SLIDER_HATA:id:current` | Slider out-of-range UPDATE rejected |
| `PROGRESS_HATA:id:current` | Progress out-of-range UPDATE rejected |
| `MSG_CLICK:id:button` | Message button tapped |
| `MSG_TIMEOUT:id` | Message timed out |

### Media / Radio

| Message | Description |
|---------|-------------|
| `MEDIA_PLAY` / `MEDIA_PAUSE` | Media play/pause |
| `MEDIA_NEXT` / `MEDIA_PREV` | Next/prev song |
| `MEDIA_VOL_UP` / `MEDIA_VOL_DOWN` | Volume up/down |
| `QR_DATA:text` | QR code scanned |
| `WEATHER_DATA:...` | Weather data |
| `RADIO_STATE:PLAYING:name` | Radio playing |
| `RADIO_STATE:ENDED:name` | Radio ended |
| `RADIO_PLAY` / `RADIO_PAUSE` | Radio-mode MediaPlayer control |
| `RADIO_NEXT` / `RADIO_PREV` | Radio-mode MediaPlayer control |

### Info / Status

| Message | Description |
|---------|-------------|
| `FONT_LIST:...` | Font list |
| `ID_INFO:PAGE=x,MAX=y` | Page ID info |
| `PONG` | PING reply |
| `ACK_OK:command` | Operation succeeded |
| `ERR:command:error` | Operation failed |
| `TIME:yyyy-MM-dd HH:mm:ss` | GET_TIME reply |
| `BT_STATUS:ON|OFF` | Bluetooth adapter state |

### LINKS (HTTP)

| Message | Description |
|---------|-------------|
| `LREQ,id,STARTED` | Request started |
| `LRESP,id,statusCode,body` | Success with body |
| `LRESP,id,statusCode` | Success without body |
| `LRESP,id,statusCode,NOTFOUND` | KEYWORD not found |
| `LRESP,id,-1,NET_ERR` | DNS/connect error |
| `LRESP,id,0,TIMEOUT` | 15 s timeout |
| `LRESP,id,-4,SSL_ERR` | SSL error |
| `LRESP,id,4xx` / `5xx` | HTTP error |

### Object Updates

| Message | Description |
|---------|-------------|
| `NEEDLE_CHANGE:id:angle` | Needle UPDATE done |
| `ICON_STATE:id:state` | Icon UPDATE done |
| `BTN_LABEL:id:newLabel` | Button UPDATE done |
| `LINE_ANGLE:id:angle` | Line UPDATE done |
| `LIQUID_CHANGE:id:value` | Liquid UPDATE done |
| `MATRIX_SPEED:id:speed` | Matrix scrollSpeed changed |
| `TIMER_DONE:id` | Timer repeat completed |

### MediaPlayer status

| Message | Description |
|---------|-------------|
| `MP_STATUS:id:mode:state:title:elapsed:duration:remaining:storage` | MP_STATUS reply |
| `ACK_OK:MP_MODE:id:mode` | MP_MODE confirm |
| `ACK_OK:MP_PLAY:id` | MP_PLAY confirm |
| `ACK_OK:MP_PAUSE:id` | MP_PAUSE confirm |
| `ACK_OK:MP_STOP:id` | MP_STOP confirm |
| `ACK_OK:MP_NEXT:id` | MP_NEXT confirm |
| `ACK_OK:MP_PREV:id` | MP_PREV confirm |
| `ACK_OK:MP_VOL_UP:id` | MP_VOL_UP confirm |
| `ACK_OK:MP_VOL_DOWN:id` | MP_VOL_DOWN confirm |
| `ACK_OK:MP_MUTE:id:isMuted` | MP_MUTE confirm |

### Sensors

| Message | Description |
|---------|-------------|
| `SENSOR_LIST:name1,name2,...` | Available sensors |
| `SENSOR_DATA:name,values` | Periodic sensor data |
| `SENSOR_INFO:name,type=N,vendor=X,unit=Y,range=min..max` | Sensor detail |
| `SENSOR_INFO:DONE` | Sensor info list end |
| `ACK_OK:SENSOR_START:name:intervalMs` | Sensor start confirm |
| `ACK_OK:SENSOR_STOP:name|ALL` | Sensor stop confirm |
| `GPS_STATUS:on|off:streaming|idle` | GPS status |
| `ACK_OK:GPS_ENABLE` | GPS_ENABLE confirm |

### Fingerprint / Time / BT

| Message | Description |
|---------|-------------|
| `FP_VERIFY_OK:id` | Fingerprint verified |
| `FP_VERIFY_FAIL:id:reason` | Fingerprint failed |
| `FP_STATUS:AVAILABLE|UNAVAILABLE:reason` | Fingerprint status |
| `ACK_OK:BT_SEND:mac` | BT_SEND confirm |

### Bluetooth Gateway

| Message | Description |
|---------|-------------|
| `BL_DATA,num1,val1,num2,val2,...` | Batch read result |
| `BL_DONE,ok,err` | Read done |
| `BL_ERR,num,reason` | Device error |
| `BL_SENT,num,ack` | Command sent |
| `BL_LIST_START,count` | Gateway list start |
| `BL_LIST_ITEM,num,mac,name,displayId` | Gateway device row |
| `BL_LIST_END` | Gateway list end |
| `BL_RELOAD_ACK,count` | Reload confirm |
| `BL_STATUS_OK,TIMER=n,DEVICES=n,QUEUE=n,STATE=IDLE|BUSY` | Status report |
| `BL_TIMER_ACK,sec` | Timer confirm |

---

## 6. Notes & Gotchas

### Throttle
`SLIDER`, `POT`, `RGBPICKER` use **500 ms** throttle during drag; final value is sent without throttle.

### Long Press
`LONG_PRESS_MS = 2000`. Movement threshold 20 px. Emits `OBJ_INFO:PAGE=x,ID=y,TYPE=z`.

### Save vs Update
`UPDATE` → runtime, not saved. `SETSTATE` → persistent, saved to SAVE_SCENE. `U` also runtime-only.

### Target SDK
Android 6.0 (SDK 23) and above.

### Encoder stepsPerTurn
`360 / stepsPerTurn` = angle per step. Default 12 = 30° steps.

### Encoder click sound
ToneGenerator beep on each step.

### Radio.txt
FM radio config in root folder. Format: `package.activity` or `package,activity`.

### VU meter
Two methods:
1. **INTERNAL**: MP3/Radio playing → Visualizer API (VU_SOURCE)
2. **EXTERNAL**: ESP32 or other source via `U,id,value`

### VU permission
`RECORD_AUDIO` required for internal VU.

### SAF storage
Files use direct access on Android 6-9; SAF on Android 10+. User selects `oledresim` folder once.

### Subfolders
- `images/` — images, GIFs, bg images
- `icons/` — ICON object
- `fonts/` — font.txt + font files

### ImageMap design
In design program, set pixel size = hit size. Pixel's top-left = hit's top-left.

### RGBPicker W slider
When W rises, RGB zeroes (clean white). When W drops to 0, last RGB restored from cache.

### LINKS HTTP
Arduino sends URL → EasyOLED performs HTTPS/HTTP GET. URL cannot contain commas (%2C for comma). KEYWORD slices response after that word. LENGTH limits response size.

### MP_* family
MP_* commands are for Arduino. UI-triggered actions (Button/ActionButton) send MEDIA_* events without ACK. MP_* commands reply with their own ACK (`ACK_OK:MP_*:id`).

### MP_PLAY behavior
First play → MediaStore scan + last saved index; STOP-then-PLAY → same song from start; PAUSE-then-PLAY → continues from where paused; already playing → no-op.

### MP_MUTE limitation
v3.5: only toggles RADIO. No-op in MP3 mode.

### Sensor single
Only ONE sensor active at a time. New SENSOR_START auto-stops previous.

### MediaStore
Music listed via MediaStore API. Filter: `IS_MUSIC != 0 AND DURATION > 10000` + extension check.

### SAVE_SCENE v3.5
SAF-aware: compatible with Android 10+ scoped storage.

### Image circular
`IMAGE shape=1`: x,y = center, width = diameter. Height ignored.

### GIF speed
0.25x–4.0x. Motor-level (`GifDrawable.setSpeed`) + timer interval = `50ms / speed`, min 15 ms.

### Needle smooth
`smooth=1` → UPDATE values animate (ease-out). `smoothSpeed` smaller = slower approach.

### Line pivot
`pivotRatio=0` → fast path (legacy). `>0` → pivot preserved during rotation; startX/startY auto-shift.

### Liquid shape
`0=rect` (x,y=top-left), `1=circle` (x,y=center, width=diameter), `2=flask` (x,y=top-left).

### Matrix font
Text mode uses `matrix_font.txt`. `[code]` syntax for special chars (`[199]`=Ç). `[C]` centers line.

### Fingerprint
`FP_VERIFY` uses BiometricPrompt. Permission: `USE_BIOMETRIC`.

### Gateway bt_devices.txt
Format: `type,number,MAC,name,read_cmd,timeout_ms,display_id`. v3.6 only `classic` active; `ble` parsed but inactive.

### Text align pollution
Shared `Paint` object can leak `textAlign` from a `draw()` that used `CENTER`. This bug historically caused ListBox/ListBoxR text to render partially clipped when a ScrollText/DCLOCK ran before them. **Fixed by explicitly setting `paint.textAlign = Paint.Align.LEFT` at start of ListBox/ListBoxR draw**, and defensively resetting after each `obj.draw()` in `redrawActivePage` / `redrawRegion`.

### DCLOCK dirty rect
DCLOCK's `draw()` centers text; when text is wider than the box, partial render only touches center. **Fixed by `computeDigitalClockDirtyRect(obj)`** which measures worst-case `"88:88:88"` × `"88/88/8888"` + 15% safety margin.

### THERM bulb render
`bulbRadius = width × 0.9`, so bulb is wider than pipe. Dirty rect must use `maxOf(width/2, bulbRadius) + 8.5f` (stroke + AA + safety). Applied in 5 places: `objectIntersectsRect`, UPDATE `$` branch, UPDATE non-`$` branch, FastUpdate, markGatewayDisplayDirty.

### LISTBOXR icon slot
Icon slot uses `min(textSize × 2, maxAllowedIcon, width × 0.30)`. **All rows reserve the same slot width** — text start X is stable across rows regardless of whether an icon is present. Icon is centered within the slot (not left-aligned) so narrow icons look balanced.

### TEXT multi-line box
`TEXT` box dimensions account for `\n`: `boxWidth = maxLineWidth + 2×padding`, `boxHeight = (lineCount − 1) × lineHeight + singleLineHeight + 2×padding`. When `width=0` or `height=0` (auto mode), box grows to fit multi-line content. If `width>0` or `height>0` (fixed), user is responsible for sizing.

### Canvas state reset
`redrawActivePage` and `redrawRegion` both reset `canvas.saveCount` to 1 at start (defensive). This protects against any leaked `save()` calls from objects. Also `paint.textAlign = Paint.Align.LEFT` is restored after each `obj.draw()` in the render loop.

---

## 7. Simulator Implementation Hints

### 7.1 Python Simulator Structure

The simulator (see `easyoled_python_show.py`) processes a CSV of command groups. Each SPACE advances one step.

**CSV format** (`;` delimiter):
```
;nesne;komut;AÇIKLAMA
group_id;object_name;full_command;description
```

**Group suffix semantics:**
- `N` (immediate) → send commands immediately on SPACE
- `Nda` (delayed) → send after 3 s (or custom delay via special id "45")
- `Nd` (loop) → send in a loop every 3 s

**Numeric root rule:** `5`, `5da`, `5d` all belong to group `5`. Same-step siblings share the root.

### 7.2 Multi-Page Demo Pattern

For a full video walkthrough:

1. **Cycle 1 (pages 0-9)** — no `CLEAR`. Each step: `PAGE,N` + objects. All 10 pages stay populated for manual swipe tour.
2. **Cycle 2+ (pages 0-9)** — each step: `PAGE,N` + `CLEAR` + objects. Each page cleaned before next object.

Python should **NOT** auto-send `CLEAR` — it must come from CSV.

### 7.3 Recommended Test Order

1. `PAGE,N` first (choose page)
2. `CLEAR` (if needed)
3. Object create commands
4. `SETSTATE`/`UPDATE`/`LINK` as applicable
5. `d`-suffix for cyclic demos (e.g. `UPDATE` loop for Gauge, Wave data)

### 7.4 Coordinate Assumptions

For 1080 × 1920 portrait:
- 2-column grid: `x=100` and `x=580`
- Row spacing: ≥100 px between object groups
- Text size ≤ 70 px to fit most boxes
- Bottom safe area (last 300 px) often used by system UI — place important elements above y=1600

### 7.5 Object Interdependencies

- `LINK,textId,targetId,prop` requires both objects on the same active page at LINK time
- `VU_SOURCE` requires SEGMENTBAR objects to exist first
- `MP_TOGGLE` from ActionButton requires a MEDIAPLAYER object with matching id
- `RADIO_LOAD` requires valid `.m3u` file
- `GROUP_ADD` requires GROUP to exist first

### 7.6 Event Capture

Simulator should display app → Arduino events in a scrolling log (last 30 lines visible is a good default). Events can arrive interleaved with `ACK_OK` messages.

### 7.7 Common Pitfalls

**Text align pollution:** If your simulator renders multiple objects in the same canvas context, ensure `paint.textAlign` is reset after each object. Otherwise ListBox text can render clipped.

**Timer storms:** Timer-capable objects (ScrollText, DCLOCK) generate high-frequency render events. Make sure page-change logic stops timers on the old page before starting new ones.

**Multi-page state:** Clearing a page should not reset timers on other pages. Each page's `pageObjects` list is independent.

**Partial render regions:** When implementing partial rendering, ensure the dirty rect accounts for:
- Text overflow (DCLOCK, Text with fixed box, Button labels)
- Bulb radius (THERM)
- Rotation (Image, Needle, Triangle, Line)
- Stroke width (Rect, Circle)

### 7.8 Testing Checklist

Before shipping a scene:

- [ ] All objects visible at expected positions
- [ ] Text renders fully (no clipping)
- [ ] Update flows work (UPDATE, SETSTATE, U)
- [ ] Timers tick correctly (DCLOCK, ScrollText, GIF, Needle, Line, Liquid, Matrix)
- [ ] Page changes don't leak timer state
- [ ] Scenes save/restore without data loss
- [ ] MediaPlayer control works (MP3 + Radio)
- [ ] VU meter responds to audio (if used)
- [ ] Sensor data flows (if sensors used)
- [ ] Long-press OBJ_INFO works
- [ ] CLEAR + REDRAW doesn't leave artifacts

---

*End of document. Version 3.6.0 · Last updated: 2026*
