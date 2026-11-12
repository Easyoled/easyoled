
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
Shared `Paint` object can leak `textAlign` from a draw() that used `CENTER`. This bug historically caused ListBox/ListBoxR text to render partially clipped when a ScrollText/DCLOCK ran before them. **Fixed by explicitly setting `paint.textAlign = Paint.Align.LEFT` at start of ListBox/ListBoxR draw**, and defensively resetting after each `obj.draw()` in `redrawActivePage` / `redrawRegion`.

### DCLOCK dirty rect
DCLOCK's `draw()` centers text; when text is wider than the box, partial render only touches center. **Fixed by `computeDigitalClockDirtyRect(obj)`** which measures worst-case `"88:88:88"` × `"88/88/8888"` + 15% safety margin.

---

## 7. Simulator Implementation Hints

### 7.1 Python Simulator Structure

The simulator (see `easyoled_python_show.py`) processes a CSV of command groups. Each SPACE advances one step.

**CSV format** (`;` delimiter):

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

---

*End of document. Version 3.6.0 · Last updated: 2026*
