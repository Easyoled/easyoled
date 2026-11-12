### 3.5 Text & Media Widgets

---

#### SCROLL_TEXT

**Purpose:** Scrolling text (horizontal / vertical / loop).

**Syntax:**
```
SCROLL_TEXT,id,text,x,y,width,height,direction,scrollAmount,textSize,color,bgColor,fontType,isBold
```

**Params:**
- `id` (int, 1)
- `text` (str, "Kayan metin...")
- `x`, `y` (float, 50)
- `width`, `height` (float, 200, 40)
- `direction` (choice, 0) — 0=horizontal, 1=vertical, 2=loop
- `scrollAmount` (float, 50)
- `textSize` (float, 24)
- `color` (color, `#FFFFFF`)
- `bgColor` (color, TRANSPARENT)
- `fontType` (int, 1)
- `isBold` (choice, 1)

**Example:**
```
SCROLL_TEXT,23,Hoşgeldiniz,50,50,200,40,2,80,24,#FFFFFF,TRANSPARENT,1,1
```

**UPDATE:** `text`
**SETSTATE:** `text`, `color`, `scrollAmount`, `direction`
**Timer:** 30 ms

---

#### WAVE

**Purpose:** Real-time line graph.

**Syntax:**
```
WAVE,id,x,y,width,height,minVal,maxVal,color,bgColor,dataCount,title,yLabel,xLabel,textColor,textSize,fontType,isBold
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50, 100)
- `width`, `height` (float, 300, 150)
- `minVal`, `maxVal` (float, 0, 100)
- `color` (color, `#00FF00`)
- `bgColor` (color, `#000000`)
- `dataCount` (int, 10)
- `title`, `yLabel`, `xLabel` (str, "")
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 14)
- `fontType` (int, 1)
- `isBold` (choice, 1)

**Example:**
```
WAVE,24,50,100,300,150,0,100,#00FF00,#000000,20,Sıcaklık,Derece,Dakika,#FFFFFF,14,1,1
```

**Special:** `WAVE_DATA,id,value` — add data point

---

#### MEDIAPLAYER

**Purpose:** MP3 or FM radio control. Can also be triggered via ActionButton.

**Syntax:**
```
MEDIAPLAYER,id,x,y,width,height,color,bgColor,playState,mode
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 120, 120)
- `color` (color, `#FF0000`)
- `bgColor` (color, `#333333`)
- `playState` (choice, 0) — 0=play icon, 1=pause icon
- `mode` (choice, 0) — 0=MP3, 1=RADIO

**Example:**
```
MEDIAPLAYER,26,100,100,120,120,#FF0000,#333333,0,0
```

**UI Event:** `MEDIA_PLAY` / `MEDIA_PAUSE` / `MEDIA_NEXT` / `MEDIA_PREV` / `MEDIA_VOL_UP` / `MEDIA_VOL_DOWN`
**Arduino control:** Use `MP_*` commands (see MP_* group)
**VU:** `VU_SOURCE` can be bound while playing

---

#### RADIO_LOAD

**Purpose:** Internet radio station list (M3U) + optional QR.

**Syntax:**
```
RADIO_LOAD,id,x,y,width,height,m3uFileName,selectedIndex,autoPlay,bgColor,textColor,selectColor,textSize,qrSize
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50)
- `width`, `height` (float, 200, 200)
- `m3uFileName` (str, "radyo.m3u")
- `selectedIndex` (int, -1)
- `autoPlay` (choice, 0)
- `bgColor` (color, `#000000`)
- `textColor` (color, `#FFFFFF`)
- `selectColor` (color, `#FF00FF`)
- `textSize` (float, 16)
- `qrSize` (float, 0) — 0=no QR, >0=QR at bottom-right

**Example:**
```
RADIO_LOAD,27,50,50,300,200,radyo.m3u,-1,0,#000000,#FFFFFF,#FF00FF,16,80
```

**UI Event:** `RADIO_CHANGE:id:index:name:url`
**Related:** `RADIO_PLAY`, `RADIO_PAUSE`, `RADIO_STOP`, `RADIO_MUTE`, `RADIO_STATUS`

---

### 3.6 Images

---

#### IMAGE

**Purpose:** Image window. FIT/FILL/STRETCH, circular mode, pivot, auto-rotate.

**Syntax:**
```
IMAGE,id,x,y,width,height,fileName,scale,alpha,bgColor,borderColor,cornerRadius,rotation,pivotPx,pivotPy,scaleType,bgColorAlpha,shape,autoRotate,rotateSpeed,rotateDirection
```

**Pivot alternative (Format 2):**
```
IMAGE,id,x,y,w,h,fileName,100,1.0,TRANSPARENT,GRAY,0,0,FROM,5,-1,0,1.0,0,0,60,1
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50) — top-left; **center** if `shape=1`
- `width`, `height` (float, 200, 150) — **width=diameter** if `shape=1`
- `fileName` (str, "resim.png") — from `images/`
- `scale` (int, 100) — 1-100
- `alpha` (float, 1.0)
- `bgColor` (color, TRANSPARENT)
- `borderColor` (color, `#808080`)
- `cornerRadius` (float, 0)
- `rotation` (float, 0)
- `pivotPx`, `pivotPy` (float, -1) — -1=center
- `scaleType` (choice, 0) — 0=FIT, 1=FILL, 2=STRETCH
- `bgColorAlpha` (float, 1.0)
- `shape` (choice, 0) — 0=rectangle, 1=circular
- `autoRotate` (choice, 0)
- `rotateSpeed` (float, 60)
- `rotateDirection` (choice, 1)

**Example (circular):**
```
IMAGE,20,200,200,150,150,kadran.png,100,1.0,TRANSPARENT,GRAY,0,0,-1,-1,-1,0,1.0,1,0,60,1
```

**UPDATE:** `rotation` (`$+10`) → `IMAGE_ROTATE:id:angle`
**SETSTATE:** `rotation`, `scaleType`, `scale`, `alpha`, `borderColor`, `cornerRadius`, `bgColorAlpha`, `autoRotate`, `rotateSpeed`
**Error:** `ERR:IMAGE:id:FILE_NOT_FOUND`

---

#### IMAGEMAP

**Purpose:** Image with invisible hit regions. Tap region → `HIT:id:label`.

**Syntax:**
```
IMAGEMAP,id,x,y,w,h,fileName,alpha,borderColor,borderWidth,hitCount,hit1_label,hit1_x,hit1_y,hit1_w,hit1_h,hit2_label,...
```

**Params:**
- `id` (int, 1)
- `x`, `y`, `w`, `h` (float, 50, 50, 300, 200)
- `fileName` (str, "araba.png")
- `alpha` (float, 1.0)
- `borderColor` (color, TRANSPARENT)
- `borderWidth` (float, 0)
- `hitCount` (int, 0) — 0-10
- `hitN_label` (str, "H1")
- `hitN_x`, `hitN_y` (float, 100)
- `hitN_w`, `hitN_h` (float, 40)

**Example (2 hits):**
```
IMAGEMAP,21,50,50,300,200,araba.png,1.0,#FF0000,2,2,kapi,100,120,60,80,teker,180,160,50,50
```

**UI Event:** `HIT:id:label`

---

#### GIF_LOAD

**Purpose:** Animated GIF. Speed control 0.25x–4.0x.

**Syntax:**
```
GIF_LOAD,id,x,y,width,height,fileName,autoPlay,bgColor,cornerRadius,borderColor,borderWidth,rotation,speed
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 100)
- `fileName` (str, "animasyon.gif")
- `autoPlay` (choice, 1)
- `bgColor` (color, TRANSPARENT)
- `cornerRadius` (float, 0)
- `borderColor` (color, TRANSPARENT)
- `borderWidth` (float, 2)
- `rotation` (float, 0)
- `speed` (float, 1.0) — 0.25–4.0

**Example:**
```
GIF_LOAD,22,100,100,200,200,anim.gif,1,TRANSPARENT,10,TRANSPARENT,2,0,2.0
```

**UPDATE:** `play` / `stop` / `pause` / `resume`
**Timer:** `50ms / speed` (min 15ms)

---

#### QRCODE

**Purpose:** QR code reader button. Opens camera; `QR_DATA:text` on scan.

**Syntax:**
```
QRCODE,id,x,y,width,height,label,textColor,bgColor
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 80)
- `label` (str, "QR Oku")
- `textColor` (color, `#FFFFFF`)
- `bgColor` (color, TRANSPARENT)

**Example:**
```
QRCODE,25,100,100,80,80,QR Oku,#FFFFFF,TRANSPARENT
```

**UI Event:** `QR_DATA:text`

---

### 3.7 Matrix

---

#### MATRIX_SETUP

**Purpose:** LED dot-matrix panel (raw row mode).

**Syntax:**
```
MATRIX_SETUP,id,x,y,rows,cols,pixelSize,scrollSpeed,color,bgColor,gridColor,invertY
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `rows`, `cols` (int, 8, 32) — 1-64
- `pixelSize` (float, 4)
- `scrollSpeed` (float, 0) — 0=static
- `color` (color, `#FF0000`)
- `bgColor` (color, `#000000`)
- `gridColor` (color, `#333333`)
- `invertY` (choice, 0)

**Example:**
```
MATRIX_SETUP,41,50,100,8,32,4,0,#FF0000,#000000,#333333,0
```

**Related:** `MATRIX_ROW`, `MATRIX_CLEAR`, `MATRIX_STRING`

---

#### MATRIX_TEXT

**Purpose:** LED dot-matrix panel (text mode). Uses `matrix_font.txt`.

**Syntax:**
```
MATRIX_TEXT,id,x,y,rows,cols,pixelSize,scrollSpeed,color,bgColor,gridColor,text
```

**Example:**
```
MATRIX_TEXT,42,50,100,8,32,4,0.05,#FF0000,#000000,#333333,MERHABA
```

**Text features:** `\n` multi-line; `[C]` centers line; `[199]` = Ç, `[214]` = Ö, `[350]` = Ş, `[304]` = İ.

**Related:** `MATRIX_STRING` (update text)

---

#### MATRIX_STRING

**Purpose:** Update existing Matrix text.

**Syntax:**
```
MATRIX_STRING,id,text
```

**Example:**
```
MATRIX_STRING,41,YENI METIN
```

**Note:** `scrollOffset` reset to 0 (text starts from beginning).

---

### 3.8 Buttons

---

#### BTN

**Purpose:** Standard button. Text or image.

**Syntax:**
```
BTN,id,label,x,y,w,h,color,imageFile,textSize,textColor,fontType,isBold
```

**Params:**
- `id` (int, 1)
- `label` (str, "Buton")
- `x`, `y` (float, 100)
- `w`, `h` (float, 100, 40)
- `color` (color, `#FFFFFF`) — TRANSPARENT=only red border
- `imageFile` (str, "")
- `textSize` (float, 20)
- `textColor` (color, `#FF0000`)
- `fontType` (int, 1)
- `isBold` (choice, 1)

**Example:**
```
BTN,36,Gönder,100,100,120,50,#00AAFF,btn.png,20,#FFFFFF,1,1
```

**UPDATE:** `label` → `BTN_LABEL:id:new_label`
**SETSTATE:** `label`, `color`, `textColor`
**UI Event:** `BTN_CLICK:page,id,label`

---

#### ACTION_BTN

**Purpose:** Action button. Tap executes its internal command.

**Syntax:**
```
ACTION_BTN,id,x,y,w,h,label,textColor,bgColor,imageFile,textSize,fontType,isBold,action
```

**Params:**
- `id` (int, 1)
- `x`, `y`, `w`, `h` (float, 100, 100, 120, 50)
- `label` (str, "Aksiyon")
- `textColor` (color, `#FFFFFF`)
- `bgColor` (color, `#0000FF`)
- `imageFile` (str, "")
- `textSize` (float, 0) — 0=auto
- `fontType` (int, 1)
- `isBold` (choice, 1)
- `action` (str, "PAGE,+1") — comma-capable

**Example:**
```
ACTION_BTN,37,Sayfa+,100,100,120,50,#FFFFFF,#0000FF,,0,1,1,PAGE,+1
```

**Common Actions:**

| Action | Effect |
|--------|--------|
| `PAGE,+1` | Next page |
| `MP_TOGGLE,26` | Toggle MediaPlayer |
| `TIMER_START,1,1000,0,UPDATE,5,$+1` | Start timer |
| `WEATHER,Istanbul,TR,30` | Start weather |
| `VU_GAIN,0.5` | Change VU gain |
| `BL_READ_ALL` | Gateway read all |

---

### 3.9 Containers

---

#### GROUP

**Purpose:** Container for child objects. Moves with children.

**Syntax:**
```
GROUP,id,x,y,bgColor,alpha,action
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `bgColor` (color, TRANSPARENT)
- `alpha` (float, 1.0)
- `action` (str, "") — optional tap action

**Example:**
```
GROUP,38,50,50,TRANSPARENT,1.0,PAGE,2
```

**Related:** `GROUP_MOVE`, `GROUP_BGCOLOR`, `GROUP_ADD`, `GROUP_REMOVE`
**UI Event:** `GROUP_CLICK:id`

---

### 3.10 Special Objects

---

#### NEEDLE

**Purpose:** Rotating needle over a dial (often paired with IMAGE dial).

**Syntax:**
```
NEEDLE,id,cx,cy,length,thickness,needleColor,centerColor,centerFilled,centerRadius,angle,smooth,smoothSpeed,autoRotate,rotateSpeed,rotateDirection
```

**Params:**
- `id` (int, 1)
- `cx`, `cy` (float, 200)
- `length`, `thickness` (float, 100, 6)
- `needleColor` (color, `#FF0000`)
- `centerColor` (color, `#FFFFFF`)
- `centerFilled` (choice, 1)
- `centerRadius` (float, 12)
- `angle` (float, 0)
- `smooth` (choice, 0)
- `smoothSpeed` (float, 0.15)
- `autoRotate` (choice, 0)
- `rotateSpeed` (float, 60)
- `rotateDirection` (choice, 1)

**Example:**
```
NEEDLE,33,200,200,100,6,#FF0000,#FFFFFF,1,12,0,1,0.15,0,60,1
```

**UPDATE:** `angle`, `$+N` → `NEEDLE_CHANGE:id:angle`
**SETSTATE:** `needleColor`, `centerColor`, `centerFilled`, `length`, `smooth`, `smoothSpeed`, `autoRotate`
**Timer:** 16 ms (smooth), 33 ms (autoRotate)

---

#### LIQUID

**Purpose:** Animated liquid level indicator.

**Syntax:**
```
LIQUID,id,x,y,w,h,minVal,maxVal,currentValue,fillColor,bgColor,waveAmp,waveSpeed,borderColor,borderWidth,cornerRadius,shape
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100) — center if `shape=1` or `shape=2`
- `width`, `height` (float, 100, 200) — width=diameter if `shape=1`
- `minVal`, `maxVal`, `currentValue` (float, 0, 100, 50)
- `fillColor` (color, `#00FFFF`)
- `bgColor` (color, `#333333`)
- `waveAmp` (float, 6) — 0-100
- `waveSpeed` (float, 0.08) — 0.0-1.0
- `borderColor` (color, `#FFFFFF`)
- `borderWidth` (float, 2)
- `cornerRadius` (float, 10)
- `shape` (choice, 0) — 0=rect, 1=circle, 2=flask

**Example (rectangle):**
```
LIQUID,39,100,100,100,200,0,100,50,#00FFFF,#333333,6,0.08,#FFFFFF,2,10,0
```

**UPDATE:** `currentValue`, `$+N`, `$-N` → `LIQUID_CHANGE:id:value`
**SETSTATE:** `currentValue`, `fillColor`, `waveAmp`, `waveSpeed`
**Timer:** 33 ms (wave animation)

---

## 4. Commands

### 4.1 Value Update

---

#### UPDATE

**Purpose:** Update an object's primary value at runtime (not saved to SAVE_SCENE).

**Syntax:**
```
UPDATE,id,new_value
UPDATE,id,$+5       (delta)
UPDATE,id,$-3
```

**Targets:**
- TEXT: text (supports `$+N` numeric delta)
- Slider/Progress/Gauge/Thermometer/Ruler/SegmentBar: currentValue
- Switch/Radio/TriState: state
- MultiStateIcon: currentState
- Image: rotation
- Circle/Triangle/Button: label
- RectRnd: (trigger)
- Battery: currentLevel
- MediaPlayer: playState
- Needle: angle
- Liquid: currentValue
- Line: angle

---

#### U

**Purpose:** Fast numeric update (parser-skip). Ideal for VU meter and continuous updates.

**Syntax:**
```
U,id,value
U,id1,val1,id2,val2,...
```

**Supported targets:**
- Text (text)
- SegmentedBar (currentValue)
- Slider (currentValue)
- ProgressBar (currentValue)
- Gauge (currentValue)
- Thermometer (currentValue)
- Ruler (currentValue)
- RoundSlider/POT (currentValue)
- Battery (currentLevel)

**Note:** No `$` delta. Not persisted.

---

#### SETSTATE

**Purpose:** Update a persistent field. Change saved to SAVE_SCENE.

**Syntax:**
```
SETSTATE,id,property,value
```

**Example:**
```
SETSTATE,5,bgColor,#FF0000
SETSTATE,5,textSize,32
SETSTATE,5,rotation,45
```

**Supported properties:** See each object's params table.

---

#### WAVE_DATA

**Purpose:** Add data point to a WAVE graph.

**Syntax:**
```
WAVE_DATA,id,value
```

---

### 4.2 Visibility & Lifecycle

#### VIS

**Purpose:** Show/hide an object.

**Syntax:**
```
VIS,id,0/1     (0=hidden, 1=visible)
```

---

#### DEL

**Purpose:** Delete an object.

**Syntax:**
```
DEL,id
```

**Note:** Background resources (radio, music, timer) stopped. UNDO can restore.

---

#### UNDO

**Purpose:** Undo last delete/add.

**Syntax:**
```
UNDO
```

Single-step undo — multiple calls walk back further.

---

#### CLEAR

**Purpose:** Clear the active page (all objects removed).

**Syntax:**
```
CLEAR
```

Only active page timers stopped; other pages' timers preserved. UNDO can restore in reverse order.

---

#### MOVE

**Purpose:** Move object to new coordinates. Persistent.

**Syntax:**
```
MOVE,id,x,y
```

All GraphicObjects supported.

---

### 4.3 Page & Scene

#### PAGE

**Purpose:** Change active page.

**Syntax:**
```
PAGE,index       (0-10)
PAGE,+1          (next page, cycles 0-9, skips page 10)
```

---

#### SAVE_SCENE

**Purpose:** Save all pages' persistent design to JSON file.

**Syntax:**
```
SAVE_SCENE,slot     (1-10)
```

**Saved:**
- All objects and their parameters
- Page background colors + images
- SETSTATE changes

**Not saved:**
- UPDATE changes
- U fast updates
- Timer/radio/mediaplayer runtime state

---

#### RESTORE_SCENE

**Purpose:** Restore a saved scene.

**Syntax:**
```
RESTORE_SCENE,slot     (1-10)
```

**Note:** Timer-capable objects restarted via `reactivateTimersForAllPages`.

---

### 4.4 Timers

#### TIMER_START

**Purpose:** Start a timer.

**Syntax:**
```
TIMER_START,id,ms,repeat,command
```

**Params:**
- `id` (int) — unique timer id
- `ms` (int) — interval in ms
- `repeat` (int) — 0=infinite, >0=count
- `command` (str) — full command per tick (comma-capable)

**Example:**
```
TIMER_START,1,1000,0,UPDATE,5,$+1
```
→ Every 1 s, add +1 to Slider 5.

**Related timer-capable objects:**

| Object | Period |
|--------|--------|
| DCLOCK | 1000 ms |
| CLOCK | 1000 ms |
| SCROLL_TEXT | 30 ms |
| GIF_LOAD | 50 ms / speed |
| NEEDLE (smooth) | 16 ms |
| NEEDLE (autoRotate) | 33 ms |
| LINE (autoRotate) | 33 ms |
| LIQUID (wave) | 33 ms |
| MATRIX (scroll) | scrollSpeed × 1000 ms |

---

#### TIMER_STOP

**Purpose:** Stop a timer.

**Syntax:**
```
TIMER_STOP,id     (omit id = stop all)
```

---

### 4.5 Linking

#### LINK

**Purpose:** Bind a Text object to another object's value.

**Syntax:**
```
LINK,textId,targetId,property
```

**Properties:** `currentValue`, `currentLevel`, `state`, `text`, `label`

**Example:**
```
LINK,10,5,currentValue      → Text 10 shows Slider 5's value
```

---

#### LINKS

**Purpose:** HTTP proxy. Arduino sends URL → EasyOLED does HTTP GET → replies to Arduino.

**Syntax:**
```
LINKS,id,url
LINKS,id,url,KEYWORD
LINKS,id,url,KEYWORD,LENGTH
```

**Arduino receives:**
- `LREQ,id,STARTED` — request started
- `LRESP,id,statusCode,body` — success
- `LRESP,id,-1,NET_ERR` — DNS/connect error
- `LRESP,id,0,TIMEOUT` — 15 s timeout
- `LRESP,id,-4,SSL_ERR` — SSL error
- `LRESP,id,4xx` or `5xx` — HTTP error

---

### 4.6 Background

#### BG_COLOR

**Purpose:** Set page background color.

**Syntax:**
```
BG_COLOR,#RRGGBB,alpha
```

---

#### BG_IMAGE

**Purpose:** Set page background image.

**Syntax:**
```
BG_IMAGE,file.png,alpha
```

---

#### BG_REMOVE

**Purpose:** Remove page background image.

**Syntax:**
```
BG_REMOVE
```

---

### 4.7 Fonts & Info

#### FONTLIST

**Purpose:** List loaded fonts.

**Response:** `FONT_LIST:1,SystemSans;2,SystemSerif;3,SystemMono;4,CustomFont...`

---

#### FONT_LOAD

**Purpose:** Load a custom font.

**Syntax:**
```
FONT_LOAD,fontType,fileName
```

**Note:** Fonts also auto-load from `fonts/font.txt` at startup.

---

#### GET_ID_INFO

**Purpose:** Return the max object ID on the active page.

**Response:** `ID_INFO:PAGE=x,MAX=y`

---

#### PING

**Purpose:** Connection test.

**Response:** `PONG`

---

#### GET_TIME

**Purpose:** Return system time.

**Response:** `TIME:yyyy-MM-dd HH:mm:ss`

---

### 4.8 Weather

#### WEATHER

**Purpose:** Fetch weather data.

**Syntax:**
```
WEATHER,city,countryCode,intervalMin
```

**Response:** `WEATHER_DATA:temp,feels,min,max,humidity,pressure,windSpeed,windDeg,main,desc,icon,sunrise,sunset`

**Related:** `SET_WEATHER_API`, `WEATHER_STOP`

---

#### SET_WEATHER_API

**Purpose:** Set OpenWeatherMap API key.

**Syntax:**
```
SET_WEATHER_API,apiKey
```

**Note:** Call once before `WEATHER`. Otherwise → `ERR:WEATHER:API_KEY_NOT_SET`.

---

#### WEATHER_STOP

**Purpose:** Stop running weather timer.

**Syntax:**
```
WEATHER_STOP
```

---

### 4.9 Radio (Internet)

#### RADIO_OPEN

**Purpose:** Open FM radio app.

**Syntax:**
```
RADIO_OPEN                     (uses radio.txt)
RADIO_OPEN,packageName         (override)
RADIO_OPEN,packageName,activityName
```

---

#### RADIO_PAUSE

**Purpose:** Pause internet radio. (alias: `RPAUSE`)

---

#### RADIO_PLAY

**Purpose:** Resume paused radio. (aliases: `RADIO_RESUME`, `RPLAY`)

---

#### RADIO_MUTE

**Purpose:** Toggle radio mute. (alias: `RMUTE`)

---

#### RADIO_STOP

**Purpose:** Stop radio. (alias: `RSTOP`)

---

#### RADIO_STATUS

**Purpose:** Query radio state. (alias: `RSTATUS`)

**Response:** `RADIO_STATUS:PLAYING:UNMUTED:name:url`

---

### 4.10 MediaPlayer (MP3 / FM)

#### MP_PLAY

**Purpose:** Start MediaPlayer. ID is global — works even if target on inactive page.

**Syntax:**
```
MP_PLAY,id
```

**Behavior (MP3):** first play → MediaStore scanned, resumes last index; STOP-then-PLAY → same song from start; PAUSE-then-PLAY → continues; already playing → no-op.
**Response:** `ACK_OK:MP_PLAY:id`

---

#### MP_PAUSE

**Purpose:** Pause. Song stays at current second.

**Response:** `ACK_OK:MP_PAUSE:id`

---

#### MP_STOP

**Purpose:** Stop and release player. **Same song will restart from beginning on next MP_PLAY** (unlike PAUSE).

**Response:** `ACK_OK:MP_STOP:id`

---

#### MP_TOGGLE

**Purpose:** Play/pause toggle.

**Response:** `ACK_OK:MP_PLAY:id` OR `ACK_OK:MP_PAUSE:id`

---

#### MP_NEXT

**Purpose:** Next song (MP3 only).

---

#### MP_PREV

**Purpose:** Previous song (MP3 only, loops).

---

#### MP_VOL_UP / MP_VOL_DOWN

**Purpose:** System music stream volume up/down.

---

#### MP_MUTE

**Purpose:** Toggle mute.

**⚠️ v3.5 limitation:** Only toggles RADIO. No-op in MP3 mode.

**Response:** `ACK_OK:MP_MUTE:id:0|1`

---

#### MP_MODE

**Purpose:** Change mode.

**Syntax:**
```
MP_MODE,id,0/1     (0=MP3, 1=RADIO)
```

**Response:** `ACK_OK:MP_MODE:id:mode`

---

#### MP_STATUS

**Purpose:** Query player status.

**Response:** `MP_STATUS:id:mode:state:title:elapsed:duration:remaining:storage`

Example: `MP_STATUS:201:MP3:PLAYING:VIDA LOCA:35:236:201:SD`

---
