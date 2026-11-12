# EasyOLED 3.6 — Command & Object Reference

> Version **3.6.0** · Command-line reference for the EasyOLED design/runtime system.
> Used as both user manual and specification for the Python simulator.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Common Conventions](#2-common-conventions)
3. [Objects](#3-objects)
   - 3.1 Basic Shapes
   - 3.2 Interactive Widgets
   - 3.3 Progress & Meter Widgets
   - 3.4 Clocks & Timers
   - 3.5 Text & Media Widgets
   - 3.6 Images
   - 3.7 Matrix
   - 3.8 Buttons
   - 3.9 Containers
   - 3.10 Special Objects
4. [Commands](#4-commands)
5. [Events (EasyOLED → Arduino)](#5-events-easyoled--arduino)
6. [Notes & Gotchas](#6-notes--gotchas)
7. [Simulator Implementation Hints](#7-simulator-implementation-hints)

---

## 1. Overview

EasyOLED receives comma-separated commands over a serial link (Bluetooth SPP or USB-OTG) and renders onto an Android Canvas.

Three command categories:

| Category | Example | Meaning |
|----------|---------|---------|
| **Create** | `TEXT,1,...` | Add a new object |
| **Modify** | `UPDATE,5,75`, `SETSTATE,5,color,#F00` | Update existing object |
| **Control** | `PAGE,2`, `PING`, `WAVE_DATA,7,12.5` | App/page control |

Objects are identified by `id` (1-999). **IDs are page-independent** — the same id can be referenced from any page.

---

## 2. Common Conventions

### 2.1 Command Line Format

```
COMMAND,param1,param2,...,paramN\n
```

- Command name is uppercase (parser is case-insensitive: `TEXT`, `text`, `Text` are equivalent)
- Parameters separated by commas
- A trailing `\n` is expected from Arduino
- **Note:** `UPDATE` reads only `parts[2]` → use `SETSTATE` for comma-containing text
- `TEXT`, `SETSTATE`, `MATRIX_TEXT`, `MATRIX_STRING` accept multi-comma (text = remaining parts)

### 2.2 UPDATE vs SETSTATE vs U

| Command | Runtime? | Persisted via SAVE_SCENE |
|---------|----------|--------------------------|
| `UPDATE,id,value` | Yes | ❌ not saved |
| `U,id,value[,id,value]...` | Yes (fast, parser-skip) | ❌ not saved |
| `SETSTATE,id,prop,value` | No (persistent) | ✅ saved |

### 2.3 Delta Syntax

```
UPDATE,5,$+10     → Slider 5 += 10
UPDATE,5,$-3      → Slider 5 -= 3
UPDATE,5,75       → Slider 5  = 75
```

`$` prefix only works in `UPDATE` (literal in `SETSTATE`).

### 2.4 Coordinate System

- Canvas origin (0,0) = top-left
- Unit: pixels
- Display: matches device resolution
- When `shape=1` (IMAGE, LIQUID): `x, y` is the **center** of the object

### 2.5 Color Parsing

| Format | Example | Meaning |
|--------|---------|---------|
| `#RRGGBB` | `#FF0000` | Standard hex RGB |
| `#AARRGGBB` (decimal A) | `#80FF0000` | A as percent 0-99 |
| `#AARRGGBB` (hex A) | `#FFFF0000` | Standard Android hex |
| `TRANSPARENT` | | Fully transparent |
| Named colors | `WHITE`, `BLACK`, `RED`, `GREEN`, `BLUE`, `YELLOW`, `CYAN`, `MAGENTA`, `GRAY` | |
| Raw integer | `16711680` | ARGB int (0 = transparent) |

### 2.6 Color Alpha Values

`bgAlpha`, `alpha`, `bgImageAlpha`, `bgColorAlpha` are **0.0–1.0 floats**.

### 2.7 Throttle

`SLIDER`, `POT`, `RGBPICKER` apply **500 ms throttle** during drag. On finger-up, the final value is sent without throttle.

### 2.8 Long Press

Holding for 2000 ms sends `OBJ_INFO:PAGE=x,ID=y,TYPE=z`. Movement threshold: 20 px.

### 2.9 Timer Behavior on Delete/Restore

Timer-capable objects (DCLOCK, CLOCK, ScrollText, Gif, Needle, Line, Liquid, Matrix):
- **DEL/CLEAR** → `onObjectRemoved` → `stopTimer()`
- **RESTORE_SCENE** → `reactivateTimersForAllPages` → restarted

---

## 3. Objects

Each object reference follows this structure:
- **Purpose**
- **Syntax**
- **Params**
- **Example**
- **UPDATE / SETSTATE**
- **UI Event** (if interactive)
- **Timer** (if applicable)

### 3.1 Basic Shapes

---

#### TEXT

**Purpose:** Draw text. Multi-line (`\n`), Turkish/Unicode, rotatable, background image/border supported.

**Syntax:**
```
TEXT,id,text,x,y,size,color,align,font,isBold,bgColor,bgAlpha,inputType,decimalPlaces,width,height,borderColor,borderWidth,cornerRadius,padding,bgImageFile,bgImageAlpha,direction,angle
```

**Params:**
- `id` (int, 1)
- `text` (str, "Merhaba") — `\n` for newline
- `x`, `y` (float, 100) — top-left (or box top-left)
- `size` (float, 30) — pixel font size
- `color` (color, `#FFFFFF`)
- `align` (choice, LEFT) — LEFT/CENTER/RIGHT
- `font` (int, 1) — 1=Sans, 2=Serif, 3=Mono, 4+=Custom
- `isBold` (choice, 1) — 1=Bold, 0=Normal
- `bgColor` (color, TRANSPARENT)
- `bgAlpha` (float, 1.0)
- `inputType` (choice, 0) — 0=text, 1=int, 2=float (keyboard on tap)
- `decimalPlaces` (int, 0)
- `width`, `height` (float, 0) — 0=auto
- `borderColor` (color, TRANSPARENT)
- `borderWidth` (float, 2)
- `cornerRadius` (float, 0)
- `padding` (float, 8)
- `bgImageFile` (str, "")
- `bgImageAlpha` (float, 1.0)
- `direction` (choice, 0) — 0=LR, 1=RL, 2=TB, 3=BT
- `angle` (float, 0) — -360..360

**Example:**
```
TEXT,1,Merhaba Dünya,100,100,30,#FFFFFF,CENTER,1,1,#000000,0.8,0,0,300,100,#FF0000,2,10,8,,1.0,0,0
```

**UPDATE / SETSTATE:**
- `UPDATE,1,Yeni metin` — runtime text
- `UPDATE,1,$+5` / `$-5` — numeric delta (if text is number)
- `SETSTATE,1,color,#FF0000` — persistent
- `SETSTATE,1,bgColor,#000000` · `textSize,40` · `isBold,0` · `cornerRadius,10` · `angle,45` · `direction,1` · `align,CENTER` · `fontType,2`
- `MOVE,1,x,y` — persistent position

**UI Event:** `inputType>0` → tap opens keyboard → `UPDATE,id,text` sent.

**Related:** `LINK,textId,targetId,currentValue` — binds a text to another object's value.

**Auto-sizing:** When `width=0` or `height=0`, box grows to fit text. Multi-line heights are computed from `lineCount × lineHeight + padding`.

---

#### RECT

**Purpose:** Rectangle (visual only, not a button).

**Syntax:**
```
RECT,id,x,y,w,h,isFilled,color,strokeWidth,cornerRadius
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `w`, `h` (float, 100, 0)
- `isFilled` (choice, 0) — 1=filled, 0=outline
- `color` (color, `#FFFFFF`)
- `strokeWidth` (float, 2)
- `cornerRadius` (float, 0)

**Example:**
```
RECT,2,50,50,200,100,1,#00FF00,2,10
```

**SETSTATE:** `color`, `isFilled`, `strokeWidth`, `cornerRadius` · **MOVE:** `x,y`

---

#### CIRC

**Purpose:** Circle or ellipse. Optionally a button.

**Syntax:**
```
CIRC,id,cx,cy,rx,ry,rotation,filled,color,strokeWidth,isButton,label,textColor,textSize,fontType,isBold
```

**Params:**
- `id` (int, 1)
- `cx`, `cy` (float, 200)
- `rx`, `ry` (float, 50)
- `rotation` (float, 0)
- `filled` (choice, 1)
- `color` (color, `#00FF00`)
- `strokeWidth` (float, 2)
- `isButton` (choice, 0)
- `label` (str, "")
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 0) — 0=rx/2
- `fontType` (int, 1)
- `isBold` (choice, 1)

**Example:**
```
CIRC,4,200,200,50,50,0,1,#00FF00,2,1,DUR,WHITE,24,1,1
```

**UPDATE:** `label` → `CIRCLE_LABEL:id:label`
**SETSTATE:** `label`, `color`, `textColor`, `rotation`, `cx`, `cy`, `rx`, `ry`
**UI Event:** `isButton=1` → `BTN_CLICK:page,id,label`

---

#### LINE

**Purpose:** Line segment. Pivot + auto-rotate.

**Syntax:**
```
LINE,id,startX,startY,angle,length,lineType,thickness,color,lineCount,pivotRatio,autoRotate,rotateSpeed,rotateDirection
```

**Params:**
- `id` (int, 1)
- `startX`, `startY` (float, 100)
- `angle` (float, 0) — 0=right, 90=down
- `length` (float, 100)
- `lineType` (choice, 0) — 0=solid, 1=dashed, 2=dotted, 3=dash-dot
- `thickness` (float, 2)
- `color` (color, `#FFFFFF`)
- `lineCount` (choice, 1) — 1=single, 2=double parallel
- `pivotRatio` (float, 0.0) — 0=start, 1=end, 0.5=middle
- `autoRotate` (choice, 0)
- `rotateSpeed` (float, 60)
- `rotateDirection` (choice, 1) — 1=CW, -1=CCW

**Example:**
```
LINE,5,100,100,45,150,0,2,#FFFFFF,1,0.5,1,90,1
```

**UPDATE:** `angle` (pivot preserved) → `LINE_ANGLE:id:angle`
**SETSTATE:** `color`, `thickness`, `angle`, `pivotRatio`, `autoRotate`, `rotateSpeed`, `rotateDirection`
**Timer:** 33 ms if `autoRotate=1`

---

#### TRIANGLE

**Purpose:** Isosceles or equilateral triangle. Optionally a button.

**Syntax:**
```
TRIANGLE,id,x,y,width,height,shapeType,rotation,isFilled,color,strokeWidth,isButton,label,textColor,textSize,fontType,isBold
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 100)
- `shapeType` (choice, 0) — 0=isosceles, 1=equilateral
- `rotation` (float, 0)
- `isFilled` (choice, 1)
- `color` (color, `#FFFFFF`)
- `strokeWidth` (float, 2)
- `isButton` (choice, 0)
- `label` (str, "")
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 0)
- `fontType` (int, 1)
- `isBold` (choice, 1)

**Example:**
```
TRIANGLE,6,100,100,80,80,0,0,1,#FFFFFF,2,1,UP,WHITE,24,1,1
```

**UPDATE:** `label` → `TRIANGLE_LABEL:id:label`
**SETSTATE:** `rotation`, `color`, `isFilled`, `shapeType`

---

#### PIXEL

**Purpose:** Tiny marker square (used as pivot reference for IMAGE).

**Syntax:**
```
PIXEL,id,x,y,color,size
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `color` (color, `#FFFFFF`)
- `size` (float, 3)

**Example:**
```
PIXEL,7,150,200,#FFFFFF,5
```

**UPDATE:** `MOVE,7,x,y` (persistent) · `SETSTATE,7,color,#F00` · `size,5`
**Related:** `IMAGE` with `pivot=FROM,7` uses this pixel as pivot.

---

#### RECTRND

**Purpose:** Random-number box. Tap → `RECTRND_CLICK:id:value`. Also triggerable from Arduino with `UPDATE,id,0`.

**Syntax:**
```
RECTRND,id,x,y,width,height,minVal,maxVal,isFilled,color,textColor,textSize,fontType,decimalPlaces
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 80, 60)
- `minVal`, `maxVal` (float, 0, 100)
- `isFilled` (choice, 1)
- `color` (color, `#FFFFFF`)
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 32)
- `fontType` (int, 7)
- `decimalPlaces` (int, 2) — 0-6, 0=integer

**Example:**
```
RECTRND,3,100,100,80,60,1,6,1,#FFFFFF,#FFFFFF,32,7,2
```

**UPDATE:** `UPDATE,3,0` → new random → `RECTRND_CLICK:id:new_value`
**SETSTATE:** `color`, `textColor`, `decimalPlaces`

---

### 3.2 Interactive Widgets

---

#### SLIDER

**Purpose:** Horizontal or vertical slider. 500 ms throttle; final value sent on finger-up.

**Syntax:**
```
SLIDER,id,x,y,width,height,minVal,maxVal,currentValue,trackColor,thumbColor,bgColor,isVertical
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 200, 40)
- `minVal`, `maxVal` (float, 0, 100)
- `currentValue` (float, 50)
- `trackColor` (color, `#0000FF`)
- `thumbColor` (color, `#FFFFFF`)
- `bgColor` (color, `#808080`)
- `isVertical` (choice, 0)

**Example:**
```
SLIDER,9,100,100,200,40,0,100,50,#0000FF,#FFFFFF,#808080,0
```

**UPDATE:** `UPDATE,9,75` · `$+10` / `$-10`
**SETSTATE:** `currentValue`, `trackColor`, `thumbColor`, `bgColor`, `minVal`, `maxVal`
**UI Event:** `SLIDER_CHANGE:id:value` (throttle 500 ms)
**Out of range:** `SLIDER_HATA:id:current_value`

---

#### POT (aliases: POTENTIOMETER, ROUNDSLIDER)

**Purpose:** Rotary potentiometer.

**Syntax:**
```
POT,id,cx,cy,radius,minVal,maxVal,currentValue,startAngle,sweepAngle,isClockwise,trackColor,thumbColor,arcThickness,textColor,textSize
```

**Params:**
- `id` (int, 1)
- `cx`, `cy`, `radius` (float, 200, 200, 50)
- `minVal`, `maxVal`, `currentValue` (float, 0, 100, 0)
- `startAngle` (float, 135)
- `sweepAngle` (float, 270)
- `isClockwise` (choice, 1)
- `trackColor` (color, `#00FF00`)
- `thumbColor` (color, `#FFFFFF`)
- `arcThickness` (float, 0) — 0=radius*0.15
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 24) — 0=hidden

**Example:**
```
POT,10,200,200,50,0,100,50,135,270,1,#00FF00,#FFFFFF,0,#FFFFFF,24
```

**UPDATE:** `currentValue`, `$+5`, `$-5`
**SETSTATE:** `currentValue`, `trackColor`, `thumbColor`, `arcThickness`, `startAngle`, `sweepAngle`, `isClockwise`
**UI Event:** `POT_CHANGE:id:value` (throttle 500 ms)

---

#### ENCODER (alias: ENC)

**Purpose:** Rotary knob with optional center button. Beeps each step, shows accumulated count.

**Syntax:**
```
ENCODER,id,cx,cy,radius,knobColor,indicatorColor,isButton,label,textColor,textSize,stepsPerTurn,borderColor,tickColor,centerDotColor
```

**Params:**
- `id` (int, 1)
- `cx`, `cy`, `radius` (float, 200, 200, 40)
- `knobColor` (color, `#333333`)
- `indicatorColor` (color, `#FFFFFF`)
- `isButton` (choice, 0)
- `label` (str, "")
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 0)
- `stepsPerTurn` (int, 12) — 1-360; angle per step = 360°/N
- `borderColor` (color, `#FFFFFF`) — TRANSPARENT=skip
- `tickColor` (color, `#FFFFFF`) — TRANSPARENT=skip
- `centerDotColor` (color, `#FFFFFF`) — TRANSPARENT=skip

**Example:**
```
ENCODER,11,200,200,40,#333333,#FFFFFF,1,VOLUME,WHITE,16,12,#FFFFFF,#FFFFFF,#FFFFFF
```

**SETSTATE:** `knobColor`, `indicatorColor`, `visualRotation`, `label`, `textColor`, `textSize`, `stepsPerTurn`
**UI Event:** rotation → `ENCODER:id:+N` / `ENCODER:id:-N`; button → `BTN_CLICK:page,id,label`
**Auto:** 20 ms ToneGenerator beep each step; count resets on release.

---

#### SWITCH

**Purpose:** Toggle. Color pill or image-based (`imgOn` / `imgOff`).

**Syntax:**
```
SWITCH,id,x,y,width,height,state,leftColor,rightColor,btnColor,isVertical,imgOn,imgOff
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 80, 40)
- `state` (choice, 0)
- `leftColor` (color, `#FF0000`)
- `rightColor` (color, `#00FF00`)
- `btnColor` (color, `#FFFFFF`)
- `isVertical` (choice, 0)
- `imgOn` (str, "") — image in `images/`
- `imgOff` (str, "")

**Example:**
```
SWITCH,8,100,100,80,40,0,#FF0000,#00FF00,#FFFFFF,0,on.png,off.png
```

**UPDATE:** `UPDATE,8,1`
**SETSTATE:** `state`, `leftColor`, `rightColor`, `btnColor`, `isVertical`
**UI Event:** `SWITCH_CHANGE:id:state` (anti-bounce 350 ms)

---

#### RADIO

**Purpose:** Radio button (round or square).

**Syntax:**
```
RADIO,id,x,y,width,height,style,state,borderColor,fillColor,bgColor
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 40, 40)
- `style` (choice, 0) — 0=circle, 1=square (✓)
- `state` (choice, 0)
- `borderColor` (color, `#FFFFFF`)
- `fillColor` (color, `#FF0000`)
- `bgColor` (color, TRANSPARENT)

**Example:**
```
RADIO,15,100,100,40,40,0,0,#FFFFFF,#FF0000,TRANSPARENT
```

**UPDATE:** `UPDATE,15,1`
**SETSTATE:** `state`, `borderColor`, `fillColor`
**UI Event:** `RADIO_CHANGE:id:state`

---

#### TRISTATE

**Purpose:** Three-position switch (left / center / right).

**Syntax:**
```
TRISTATE,id,x,y,width,height,state,color0,color1,color2,label0,label1,label2,bgColor,alpha
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 150, 40)
- `state` (choice, 0) — 0, 1, 2
- `color0`, `color1`, `color2` (color, RED/YELLOW/GREEN)
- `label0`, `label1`, `label2` (str, "Düşük"/"Orta"/"Yüksek")
- `bgColor` (color, TRANSPARENT)
- `alpha` (float, 1.0)

**Example:**
```
TRISTATE,16,100,100,150,40,0,#FF0000,#FFFF00,#00FF00,Düşük,Orta,Yüksek,TRANSPARENT,1.0
```

**UPDATE:** `UPDATE,16,2`
**SETSTATE:** `state`, `color0`, `color1`, `color2`
**UI Event:** `TRISTATE_CHANGE:id:state`

---

#### LISTBOX

**Purpose:** Scrollable list.

**Syntax:**
```
LISTBOX,id,x,y,width,height,items,selectedIndex,bgColor,textColor,selectColor,textSize,rowSpacing,showBorder,cornerRadius
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50)
- `width`, `height` (float, 150, 100)
- `items` (list, `Elma|Armut|Muz`) — `|` or `,` separator
- `selectedIndex` (int, 0)
- `bgColor` (color, `#000000`)
- `textColor` (color, `#FFFFFF`)
- `selectColor` (color, `#FF00FF`)
- `textSize` (float, 16)
- `rowSpacing` (float, 0)
- `showBorder` (choice, 1)
- `cornerRadius` (float, 5)

**Example:**
```
LISTBOX,17,50,50,150,100,Elma|Armut|Muz,0,#000000,#FFFFFF,#FF00FF,16,0,1,5
```

**Special:** `LISTBOX_SET_ITEMS,id,item1|item2` · `LISTBOX_SET_INDEX,id,index`
**UPDATE:** `SETSTATE,17,selectedIndex,2`
**UI Event:** `LISTBOX_CHANGE:id:index:text`

---

#### LISTBOXR

**Purpose:** Rich list with icons, subtitles, and per-row actions.

**Syntax:**
```
LISTBOXR,id,x,y,width,height,selectedIndex,bgColor,textColor,subtitleColor,selectColor,textSize,showBorder,cornerRadius,rowSpacing,padding,rowBgColor,item1,item2,...
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50)
- `width`, `height` (float, 250, 300)
- `selectedIndex` (int, 0)
- `bgColor` (color, `#000000`)
- `textColor` (color, `#FFFFFF`)
- `subtitleColor` (color, `#808080`)
- `selectColor` (color, `#FF00FF`)
- `textSize` (float, 18) — 0=auto
- `showBorder` (choice, 1)
- `cornerRadius` (float, 8)
- `rowSpacing` (float, 10)
- `padding` (float, 10)
- `rowBgColor` (color, `#333333`)
- `item1, item2, ...` (str) — each `icon|label|subtitle|action`

**Item format:** `iconFile|label|subtitle|action` — empty field = `NULL`
Action can contain commas (e.g. `PAGE,3`) — parser groups them.

**Example:**
```
LISTBOXR,18,50,50,250,300,0,#000000,#FFFFFF,#808080,#FF00FF,18,1,8,10,10,#333333,music.png|Radyo|Favoriler|PAGE,3,settings.png|Ayarlar|Sistem|PAGE,5
```

**SetState:** `selectedIndex`, `rowSpacing`, `cornerRadius`, `bgColor`
**UI Event:** `LISTBOXR_SEL:id:index:label` + row's `action` executed

---

#### COMBOBOX

**Purpose:** Dropdown list. Tap opens a temporary ListBox.

**Syntax:**
```
COMBOBOX,id,x,y,width,height,items,selectedIndex,textColor,bgColor,textSize
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 50)
- `width`, `height` (float, 150, 40)
- `items` (list, `Seçenek1|Seçenek2|Seçenek3`)
- `selectedIndex` (int, 0)
- `textColor` (color, `#FFFFFF`)
- `bgColor` (color, `#333333`)
- `textSize` (float, 16)

**Example:**
```
COMBOBOX,19,50,50,150,40,Seçenek1|Seçenek2,0,#FFFFFF,#333333,16
```

**UPDATE:** `UPDATE,19,index`
**SETSTATE:** `selectedIndex`, `textColor`, `bgColor`
**UI Event:** `COMBOBOX_CHANGE:id:index:text`
**Note:** Temp ListBox created with id=9999; removed on tap elsewhere.

---

#### ICON

**Purpose:** Multi-state icon. Shows `imageNames[currentState]`.

**Syntax:**
```
ICON,id,x,y,size,image1.png,image2.png,...,currentState
```

**Parse heuristic:** Last param is `currentState` if it's a plain integer (no `.`); otherwise it's a filename.

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `size` (float, 50)
- `imageNames` (str, `icon1.png,icon2.png`) — from `icons/`
- `currentState` (int, 0) — optional, last

**Example:**
```
ICON,35,100,100,50,moon.png,sun.png,cloud.png,0
```

**UPDATE:** `UPDATE,35,1` → `ICON_STATE:id:state`
**SETSTATE:** `currentState`, `size`

---

#### RGBPICKER (aliases: COLORPICKER, PICKER)

**Purpose:** RGBW color picker — hue ring + saturation center + W slider. 500 ms throttle.

**Syntax:**
```
RGBPICKER,id,cx,cy,outerRadius,ringThickness,hue,saturation,currentR,currentG,currentB,currentW,showW,sliderHeight
```

**Params:**
- `id` (int, 1)
- `cx`, `cy`, `outerRadius` (float, 200, 200, 100)
- `ringThickness` (float, 25)
- `hue` (float, 0) — 0-360
- `saturation` (float, 100) — 0-100
- `currentR`, `currentG`, `currentB` (int, 255, 0, 0)
- `currentW` (int, 0) — 0-255
- `showW` (choice, 1)
- `sliderHeight` (float, 20) — 8-60

**Example:**
```
RGBPICKER,14,200,200,100,25,0,100,255,0,0,0,1,20
```

**Output:** `R30G50B150W0` (throttle 500 ms; final on release)
**SetState:** `currentR`, `currentG`, `currentB`, `currentW`, `showW`
**Behavior:** Hue drag → W=0; W>0 → RGB=0 (clean white); W=0 → last RGB restored from cache.

---

### 3.3 Progress & Meter Widgets

---

#### PROGRESS

**Purpose:** Progress bar. Warning + critical + gradient support.

**Syntax:**
```
PROGRESS,id,x,y,w,h,minVal,maxVal,currentValue,color,bgColor,isVertical,textSize,textColor,fontType,isBold,warningVal,warningColor,criticalVal,criticalColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params:**
- `id` (int, 1)
- `x`, `y`, `w`, `h` (float, 100, 100, 200, 30)
- `minVal`, `maxVal`, `currentValue` (float, 0, 100, 50)
- `color`, `bgColor` (color, GREEN, DKGRAY)
- `isVertical` (choice, 0)
- `textSize` (float, -1) — -1=height*0.4
- `textColor` (color, `#FFFFFF`)
- `fontType` (int, 1)
- `isBold` (choice, 1)
- `warningVal` (float, -1) — -1=disabled
- `warningColor` (color, `#FFFF00`)
- `criticalVal` (float, -1)
- `criticalColor` (color, `#FF0000`)
- `useGradient` (choice, 0)
- `gradientStartColor`, `gradientMidColor`, `gradientEndColor` (color, GREEN/YELLOW/RED)

**Example:**
```
PROGRESS,12,100,100,200,30,0,100,50,#00FF00,#333333,0,-1,#FFFFFF,1,1,70,#FFFF00,90,#FF0000,0
```

**UPDATE:** `UPDATE,12,75` · `$+5` / `$-5` → `PROGRESS_CHANGE:id:value`
**Out of range:** `PROGRESS_HATA:id:current_value`

---

#### SEGMENTBAR (aliases: SEGBAR, VUMETER)

**Purpose:** Segmented bar (VU meter). Color decision based on **segment value** (not position), so thresholds auto-map correctly.

**Syntax:**
```
SEGMENTBAR,id,x,y,w,h,segmentCount,isVertical,minVal,maxVal,currentValue,segmentColor,warningColor,criticalColor,warningVal,criticalVal,gap,bgColor,borderColor
```

**Params:**
- `id` (int, 1)
- `x`, `y`, `w`, `h` (float, 50, 50, 40, 300)
- `segmentCount` (int, 10) — 1-50
- `isVertical` (choice, 1)
- `minVal`, `maxVal`, `currentValue` (float, 0, 100, 0)
- `segmentColor` (color, `#00FF00`)
- `warningColor` (color, `#FFAA00`)
- `criticalColor` (color, `#FF0000`)
- `warningVal` (float, 70) — -1=disabled
- `criticalVal` (float, 90)
- `gap` (float, 3)
- `bgColor` (color, `#333333`)
- `borderColor` (color, `#FFFFFF`)

**Example:**
```
SEGMENTBAR,13,50,50,40,300,10,1,0,100,45,#00FF00,#FFAA00,#FF0000,70,90,3,#333333,#FFFFFF
```

**UPDATE:** `UPDATE,13,75` · `$+10` / `$-10` · **Fast:** `U,13,120`
**SetState:** `currentValue`, `segmentCount`, `segmentColor`, `warningVal`, `criticalVal`
**UI Event:** `SEGMENTBAR_CHANGE:id:value`
**Related:** `VU_SOURCE,barRmsId,barPeakId` (see VU commands) or ESP32-side `U,id,val`

---

#### GAUGE

**Purpose:** Needle gauge. Warning + critical + gradient + auto-tick.

**Syntax:**
```
GAUGE,id,cx,cy,radius,minVal,maxVal,warningVal,currentValue,color,warningColor,textColor,textSize,startAngle,sweepAngle,isClockwise,arcThickness,stepSize,criticalVal,criticalColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params:**
- `id` (int, 1)
- `cx`, `cy`, `radius` (float, 200, 200, 80)
- `minVal`, `maxVal`, `warningVal` (float, 0, 100, 80)
- `currentValue` (float, 50)
- `color` (color, `#00FF00`)
- `warningColor` (color, `#FF0000`)
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 32)
- `startAngle` (float, 135)
- `sweepAngle` (float, 270)
- `isClockwise` (choice, 1)
- `arcThickness` (float, 0) — 0=radius*0.18
- `stepSize` (float, 10) — 0=auto "nice number"
- `criticalVal` (float, -1) — -1=disabled
- `criticalColor` (color, `#FF0000`)
- `useGradient` (choice, 0)
- `gradientStartColor`, `gradientMidColor`, `gradientEndColor` (color, GREEN/RED/RED)

**Example:**
```
GAUGE,32,200,200,80,0,100,80,45,#00FF00,#FF0000,#FFFFFF,32,135,270,1,0,10,-1,#FF0000,0
```

**UPDATE:** `UPDATE,32,75` → `GAUGE_CHANGE:id:value`
**SetState:** `currentValue`, `startAngle`, `sweepAngle`, `arcThickness`, `stepSize`, `criticalVal`, `useGradient`

---

#### RULER

**Purpose:** Ruler with tick labels. Warning + critical + gradient.

**Syntax:**
```
RULER,id,x,y,width,height,minVal,maxVal,criticalVal,currentValue,isHighCritical,isVertical,color,criticalColor,textColor,textSize,bgColor,stepSize,warningVal,warningColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params:**
- `id` (int, 1)
- `x`, `y`, `w`, `h` (float, 50, 50, 30, 150)
- `minVal`, `maxVal`, `criticalVal`, `currentValue` (float, 0, 100, 20, 0)
- `isHighCritical` (choice, 1) — 1=high side is critical, 0=low side
- `isVertical` (choice, 1)
- `color` (color, `#00FF00`)
- `criticalColor` (color, `#FF0000`)
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, 18)
- `bgColor` (color, TRANSPARENT)
- `stepSize` (float, 0) — 0=auto range/10
- `warningVal` (float, -1)
- `warningColor` (color, `#FFFF00`)
- `useGradient` (choice, 0)
- `gradientStartColor`, `gradientMidColor`, `gradientEndColor` (color, GREEN/YELLOW/RED)

**UPDATE:** `UPDATE,34,currentValue` → `RULER_CHANGE:id:value`

---

#### THERM

**Purpose:** Thermometer gauge.

**Syntax:**
```
THERM,id,cx,topY,width,height,minVal,maxVal,currentValue,color,criticalVal,textColor,textSize,isBold
```

**Params:**
- `id` (int, 1)
- `cx`, `topY`, `w`, `h` (float, 100, 50, 40, 150)
- `minVal`, `maxVal`, `currentValue` (float, 0, 100, 50)
- `color` (color, `#FF0000`)
- `criticalVal` (float, 0) — 0=disabled
- `textColor` (color, `#FFFFFF`)
- `textSize` (float, -1) — -1=auto bulb*0.6
- `isBold` (choice, 1)

**UPDATE:** `UPDATE,30,75` · `$+1` / `$-1` → `THERMOMETER_CHANGE:id:value`
**SetState:** `currentValue`, `criticalVal`, `color`
**Render:** Auto red when value exceeds `criticalVal`. Bulb radius = `width × 0.9` (wider than pipe).

---

#### BATT

**Purpose:** Segmented battery indicator.

**Syntax:**
```
BATT,id,x,y,width,height,segments,currentLevel,color
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 100, 40)
- `segments` (int, 5)
- `currentLevel` (int, 3)
- `color` (color, `#00FF00`)

**Example:**
```
BATT,31,100,100,100,40,5,3,#00FF00
```

**UPDATE:** `UPDATE,31,currentLevel` → `BATTERY_CHANGE:id:level`

---

### 3.4 Clocks & Timers

---

#### DCLOCK

**Purpose:** Digital clock/date display. Self-timer (1 s).

**Syntax:**
```
DCLOCK,id,x,y,width,height,format,textSize,color,bgColor,fontType,isBold,dateFormat,timeFormat
```

**Params:**
- `id` (int, 1)
- `x`, `y` (float, 100)
- `width`, `height` (float, 200, 60)
- `format` (choice, 2) — 0=time only, 1=date only, 2=stacked, 3=side-by-side
- `textSize` (float, 32)
- `color` (color, `#FFFFFF`)
- `bgColor` (color, TRANSPARENT)
- `fontType` (int, 1)
- `isBold` (choice, 1)
- `dateFormat` (choice, 0) — 0=DD/MM/YYYY, 1=MM/DD/YYYY, 2=YYYY-MM-DD
- `timeFormat` (choice, 0) — 0=24h, 1=12h

**Example:**
```
DCLOCK,19,100,100,500,80,3,50,#FFFFFF,TRANSPARENT,1,1,0,0
```

**SETSTATE:** `format`, `textSize`, `color`, `timeFormat`
**Timer:** 1000 ms; dirty rect auto-expanded to cover worst-case text ("88:88:88 88/88/8888").

---

#### CLOCK

**Purpose:** Analog clock. Self-timer (1 s), blinking center pixel.

**Syntax:**
```
CLOCK,id,cx,cy,diameter,hour,minute,color,bgImageFile,bgImageAlpha
```

**Params:**
- `id` (int, 1)
- `cx`, `cy` (float, 200)
- `diameter` (float, 150)
- `hour`, `minute` (int, 0) — reserved (system time used)
- `color` (color, `#FFFFFF`)
- `bgImageFile` (str, "") — clipped to circle
- `bgImageAlpha` (float, 1.0)

**Example:**
```
CLOCK,17,300,500,400,12,0,#99FFFF00,w8.png,0.7
```

**SETSTATE:** `color`, `diameter`, `bgImageAlpha`
**Timer:** 1000 ms (blink)

---
