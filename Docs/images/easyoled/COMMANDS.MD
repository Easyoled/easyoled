# EasyOLED 3.6 — Command & Object Reference

> Version **3.6.0** · Command-line reference for the EasyOLED design/runtime system.
> Used both as a user manual and as the specification for the Python simulator.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Common Conventions](#2-common-conventions)
3. [Objects](#3-objects)
   - [Basic Shapes](#31-basic-shapes)
   - [Interactive Widgets](#32-interactive-widgets)
   - [Progress / Meter Widgets](#33-progress--meter-widgets)
   - [Clocks & Timers](#34-clocks--timers)
   - [Text & Media Widgets](#35-text--media-widgets)
   - [Images](#36-images)
   - [Matrix](#37-matrix)
   - [Containers](#38-containers)
4. [Commands](#4-commands)
   - [Value Update](#41-value-update)
   - [Visibility & Lifecycle](#42-visibility--lifecycle)
   - [Page & Scene](#43-page--scene)
   - [Timers](#44-timers)
   - [Linking](#45-linking)
   - [Fonts & Info](#46-fonts--info)
   - [Weather](#47-weather)
   - [Radio (Internet)](#48-radio-internet)
   - [MediaPlayer (MP3 / FM)](#49-mediaplayer-mp3--fm)
   - [Sensors & GPS](#410-sensors--gps)
   - [VU Meter (Internal Visualizer)](#411-vu-meter-internal-visualizer)
   - [Matrix Row/Clear](#412-matrix-rowclear)
   - [Group Operations](#413-group-operations)
   - [ListBox Operations](#414-listbox-operations)
   - [Fingerprint](#415-fingerprint)
   - [Time / Bluetooth](#416-time--bluetooth)
   - [Bluetooth Gateway](#417-bluetooth-gateway)
   - [Menu & Diagnostics](#418-menu--diagnostics)
5. [Events (EasyOLED → Arduino)](#5-events-easyoled--arduino)
6. [Notes & Gotchas](#6-notes--gotchas)
7. [Simulator Implementation Hints](#7-simulator-implementation-hints)

---

## 1. Overview

EasyOLED receives comma-separated commands over a serial link (Bluetooth SPP or USB-OTG) and renders on an Android Canvas. Three kinds of commands exist:

| Kategori | Örnek | Anlam |
|----------|-------|-------|
| **Create** | `TEXT,1,...`, `SLIDER,5,...` | Yeni nesne ekle |
| **Modify** | `UPDATE,5,75`, `SETSTATE,5,color,#F00` | Varolan nesneyi güncelle |
| **Control** | `PAGE,2`, `PING`, `WAVE_DATA,7,12.5` | Uygulama/sayfa kontrolü |

Nesneler `id` (1-999) ile adlandırılır; `id` **sayfa bağımsızdır**. Aynı id'ye tüm sayfalarda erişilebilir.

---

## 2. Common Conventions

### 2.1 Command Line Format

```
KOMUT,param1,param2,...,paramN\n
```

- Komut adı büyük harf (parser case-insensitive: `TEXT`, `text`, `Text` eşdeğer).
- Parametreler virgülle ayrılır.
- Her satırın sonuna Arduino tarafı `\n` ekler.
- **Önemli:** `UPDATE` sadece `parts[2]`'yi alır → virgüllü metin için `SETSTATE` kullanın.
- `TEXT`, `SETSTATE`, `MATRIX_TEXT`, `MATRIX_STRING` çoklu virgül destekler (text = kalan tüm parçalar).

### 2.2 UPDATE vs SETSTATE vs U

| Komut | Runtime mı? | SAVESCENE |
|-------|-------------|-----------|
| `UPDATE,id,değer` | Evet | ❌ kaydedilmez |
| `U,id,değer[,id,değer]...` | Evet (hızlı, parser'sız) | ❌ kaydedilmez |
| `SETSTATE,id,prop,değer` | Hayır (kalıcı) | ✅ kaydedilir |

### 2.3 Delta Syntax

```
UPDATE,5,$+10       → Slider 5'i +10
UPDATE,5,$-3        → Slider 5'i -3
UPDATE,5,75         → Slider 5'i = 75
```

`$` işareti sadece `UPDATE`'te çalışır (`SETSTATE`'te literal `$` olarak yorumlanır).

### 2.4 Coordinate System

- Canvas origin (0,0) = sol üst köşe
- Birim: piksel
- Android sürümü: ekran çözünürlüğü (DisplayMetrics)
- `shape=1` olan nesnelerde `x,y` = **merkez** (IMAGE, LIQUID)

### 2.5 Color Parsing

`parseColor` aşağıdaki formatları kabul eder:

| Format | Örnek | Anlam |
|--------|-------|-------|
| `#RRGGBB` | `#FF0000` | Standart hex RGB |
| `#AARRGGBB` | `#80FF0000` | **Ondalık** A → **yüzde** 0-99 (0xFF=255 full) |
| `#AARRGGBB` | `#FFFF0000` | **Harfli** A → standart Android |
| `TRANSPARENT` | | Saydam |
| `WHITE`, `BLACK`, `RED`, `GREEN`, `BLUE`, `YELLOW`, `CYAN`, `MAGENTA`, `GRAY` | | Sabit renkler |
| Tamsayı | `16711680` | Raw ARGB int (0 = transparent) |

### 2.6 Color Alpha Values

`bgAlpha`, `alpha`, `bgImageAlpha`, `bgColorAlpha` → **0.0-1.0** float.

### 2.7 Throttle

`SLIDER`, `POT`, `RGBPICKER` drag sırasında **500ms** throttle uygular. Parmak kalkınca son değer kesin gönderilir (throttle bypass).

### 2.8 Long Press

2000ms basılı tutma → `OBJ_INFO:PAGE=x,ID=y,TYPE=z` yollanır. Hareket eşiği 20px.

### 2.9 Timer Behavior on Delete/Restore

Nesne iç timer'ı olan her sınıf (DCLOCK, CLOCK, ScrollText, Gif, Needle, Line, Liquid, Matrix):
- **DEL/CLEAR** → `onObjectRemoved` → `stopTimer()`
- **RESTORE_SCENE** → `reactivateTimersForAllPages` → yeniden başlar

---

## 3. Objects

Her nesnenin belgelenmesi şu yapıyı izler:
- **Ne yapar** — amaç
- **Syntax** — komut ve parametre sırası
- **Params** — tablo (sıra, ad, tip, varsayılan)
- **Example** — çalışan örnek
- **Update / SetState** — hangi komut neyi değiştirir
- **UI Event** — dokunma sonucu Arduino'ya giden mesaj
- **Timer** — varsa periyot

### 3.1 Basic Shapes

#### TEXT

**Ne yapar:** Ekrana metin yazdırır. Çok satırlı (`\n`), Türkçe/Unicode, döndürülebilir, arka plan rengi/resmi/border destekli.

**Syntax**
```
TEXT,id,text,x,y,size,color,align,font,isBold,bgColor,bgAlpha,inputType,decimalPlaces,width,height,borderColor,borderWidth,cornerRadius,padding,bgImageFile,bgImageAlpha,direction,angle
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | 1-999 |
| 2 | text | str | `Merhaba` | `\n` = satır atla |
| 3 | x | float | 100 | Kutu varsa sol üst, yoksa metin başlangıç |
| 4 | y | float | 100 | |
| 5 | size | float | 30 | Piksel cinsinden yazı boyutu |
| 6 | color | color | `#FFFFFF` | Metin rengi |
| 7 | align | choice | `LEFT` | LEFT / CENTER / RIGHT |
| 8 | font | int | 1 | 1=Sans, 2=Serif, 3=Mono, 4+=Custom |
| 9 | isBold | choice | 1 | 1=Kalın, 0=Normal |
| 10 | bgColor | color | `TRANSPARENT` | Arka plan rengi |
| 11 | bgAlpha | float | 1.0 | 0.0-1.0 |
| 12 | inputType | choice | 0 | 0=text, 1=sayı, 2=float (klavye aç) |
| 13 | decimalPlaces | int | 0 | 0-6 (inputType=2 için) |
| 14 | width | float | 0 | 0=otomatik |
| 15 | height | float | 0 | 0=otomatik |
| 16 | borderColor | color | `TRANSPARENT` | |
| 17 | borderWidth | float | 2 | |
| 18 | cornerRadius | float | 0 | |
| 19 | padding | float | 8 | Metin-kenar boşluğu |
| 20 | bgImageFile | str | `` | `images/` klasöründen |
| 21 | bgImageAlpha | float | 1.0 | |
| 22 | direction | choice | 0 | 0=LR, 1=RL (aynalı), 2=TB (90°), 3=BT (270°) |
| 23 | angle | float | 0 | -360..360 serbest açı |

**Example**
```
TEXT,1,Merhaba Dünya,100,100,30,#FFFFFF,CENTER,1,1,#000000,0.8,0,0,300,100,#FF0000,2,10,8,,1.0,0,0
```

**Update / SetState**
- `UPDATE,1,Yeni metin` → runtime metin
- `UPDATE,1,$+5` / `$-5` → sayısal delta (metin sayı ise)
- `SETSTATE,1,color,#FF0000` — kalıcı
- `SETSTATE,1,bgColor,#000000` · `textSize,40` · `isBold,0` · `cornerRadius,10` · `angle,45` · `direction,1` · `align,CENTER` · `fontType,2`
- `MOVE,1,x,y` — kalıcı konum

**UI Event:** `inputType>0` ise dokununca klavye açılır, girilen değer `UPDATE,id,text` olarak yollanır.

**Related:** `LINK,textId,targetId,currentValue` ile başka nesnenin değeri TEXT'e otomatik yazılabilir.

---

#### RECT

**Ne yapar:** Dikdörtgen. Sadece görsel (buton değil).

**Syntax**
```
RECT,id,x,y,w,h,isFilled,color,strokeWidth,cornerRadius
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2 | x | float | 100 | Sol üst |
| 3 | y | float | 100 | |
| 4 | w | float | 100 | Genişlik |
| 5 | h | float | 0 | Yükseklik |
| 6 | isFilled | choice | 0 | 1=Dolu, 0=Çerçeve |
| 7 | color | color | `#FFFFFF` | |
| 8 | strokeWidth | float | 2 | (isFilled=0 için) |
| 9 | cornerRadius | float | 0 | |

**Example**
```
RECT,2,50,50,200,100,1,#00FF00,2,10
```

**SetState:** `color`, `isFilled`, `strokeWidth`, `cornerRadius` · **MOVE:** `x,y` kalıcı

---

#### CIRC

**Ne yapar:** Daire veya elips. Buton olabilir.

**Syntax**
```
CIRC,id,cx,cy,rx,ry,rotation,filled,color,strokeWidth,isButton,label,textColor,textSize,fontType,isBold
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2 | cx | float | 200 | Merkez X |
| 3 | cy | float | 200 | Merkez Y |
| 4 | rx | float | 50 | X yarıçapı |
| 5 | ry | float | 50 | Y yarıçapı (atlanırsa rx) |
| 6 | rotation | float | 0 | Derece |
| 7 | filled | choice | 1 | 1=Dolu, 0=Çerçeve |
| 8 | color | color | `#00FF00` | |
| 9 | strokeWidth | float | 2 | |
| 10 | isButton | choice | 0 | 1=BTN_CLICK gönderir |
| 11 | label | str | `` | Buton etiketi |
| 12 | textColor | color | `#FFFFFF` | |
| 13 | textSize | float | 0 | 0=rx/2 otomatik |
| 14 | fontType | int | 1 | |
| 15 | isBold | choice | 1 | |

**Example**
```
CIRC,4,200,200,50,50,0,1,#00FF00,2,1,DUR,WHITE,24,1,1
```

**Update:** `UPDATE,4,yeniEtiket` → `CIRCLE_LABEL:id:label` döner
**SetState:** `label`, `color`, `textColor`, `rotation`, `cx`, `cy`, `rx`, `ry`
**UI Event:** `isButton=1` ise tıklamada `BTN_CLICK:sayfa,id,label`

---

#### LINE

**Ne yapar:** Çizgi. Pivot + auto-rotate destekler.

**Syntax**
```
LINE,id,startX,startY,angle,length,lineType,thickness,color,lineCount,pivotRatio,autoRotate,rotateSpeed,rotateDirection
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2 | startX | float | 100 | |
| 3 | startY | float | 100 | |
| 4 | angle | float | 0 | 0=sağa, 90=aşağı |
| 5 | length | float | 100 | |
| 6 | lineType | choice | 0 | 0=Düz, 1=Kesikli, 2=Noktalı, 3=Tireli |
| 7 | thickness | float | 2 | |
| 8 | color | color | `#FFFFFF` | |
| 9 | lineCount | choice | 1 | 1=Tek, 2=Çift paralel |
| 10 | pivotRatio | float | 0.0 | 0=start, 1=end, 0.5=orta |
| 11 | autoRotate | choice | 0 | 1=Sürekli döner |
| 12 | rotateSpeed | float | 60 | Derece/saniye (1-3600) |
| 13 | rotateDirection | choice | 1 | 1=CW, -1=CCW |

**Example**
```
LINE,5,100,100,45,150,0,2,#FFFFFF,1,0.5,1,90,1
```

**Update:** `UPDATE,5,açı` → pivot korunur, `LINE_ANGLE:id:açı` döner
**SetState:** `color`, `thickness`, `angle`, `pivotRatio`, `autoRotate`, `rotateSpeed`, `rotateDirection`

**Timer:** `autoRotate=1` ise 33ms tick (30 FPS)

---

#### TRIANGLE

**Ne yapar:** Üçgen (ikizkenar / eşkenar). Buton olabilir.

**Syntax**
```
TRIANGLE,id,x,y,width,height,shapeType,rotation,isFilled,color,strokeWidth,isButton,label,textColor,textSize,fontType,isBold
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2 | x | float | 100 | |
| 3 | y | float | 100 | |
| 4 | width | float | 100 | Taban genişliği |
| 5 | height | float | 100 | shapeType=1'de görmezden gelinir |
| 6 | shapeType | choice | 0 | 0=İkizkenar, 1=Eşkenar (h = w × √3/2) |
| 7 | rotation | float | 0 | |
| 8 | isFilled | choice | 1 | |
| 9 | color | color | `#FFFFFF` | |
| 10 | strokeWidth | float | 2 | |
| 11 | isButton | choice | 0 | |
| 12 | label | str | `` | |
| 13 | textColor | color | `#FFFFFF` | rotation'dan bağımsız okunur |
| 14 | textSize | float | 0 | 0=h × 0.25 |
| 15 | fontType | int | 1 | |
| 16 | isBold | choice | 1 | |

**Example**
```
TRIANGLE,6,100,100,80,80,0,0,1,#FFFFFF,2,1,UP,WHITE,24,1,1
```

**Update:** `UPDATE,6,yeniEtiket` → `TRIANGLE_LABEL:id:label` döner
**SetState:** `rotation`, `color`, `isFilled`, `shapeType`

---

#### PIXEL

**Ne yapar:** Konum işaretleyici. IMAGE pivot referansı olarak kullanılır.

**Syntax**
```
PIXEL,id,x,y,color,size
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2 | x | float | 100 | Sol üst |
| 3 | y | float | 100 | |
| 4 | color | color | `#FFFFFF` | |
| 5 | size | float | 3 | Kare boyutu |

**Example**
```
PIXEL,7,150,200,#FFFFFF,5
```

**Update:** `MOVE,7,x,y` (kalıcı konum) · `SETSTATE,7,color,#F00` · `SETSTATE,7,size,5`
**Related:** `IMAGE` içinde `pivot=FROM,7` ile piksel konumu döndürme pivotu olur.

---

#### RECTRND

**Ne yapar:** Rastgele sayı üreten kutu. Dokununca `RECTRND_CLICK:id:değer` gönderir. Arduino `UPDATE,id,0` ile de tetiklenebilir.

**Syntax**
```
RECTRND,id,x,y,width,height,minVal,maxVal,isFilled,color,textColor,textSize,fontType,decimalPlaces
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2-3 | x,y | float | 100 | |
| 4-5 | width,height | float | 80,60 | |
| 6-7 | minVal,maxVal | float | 0,100 | |
| 8 | isFilled | choice | 1 | |
| 9 | color | color | `#FFFFFF` | |
| 10 | textColor | color | `#FFFFFF` | |
| 11 | textSize | float | 32 | |
| 12 | fontType | int | 7 | |
| 13 | decimalPlaces | int | 2 | 0-6, 0=tam sayı |

**Example**
```
RECTRND,3,100,100,80,60,1,6,1,#FFFFFF,#FFFFFF,32,7,2
```

**Update:** `UPDATE,3,0` → yeni rastgele tetikle → `RECTRND_CLICK:id:yeni_deger` yollanır
**SetState:** `color`, `textColor`, `decimalPlaces`

---

### 3.2 Interactive Widgets

#### SLIDER

**Ne yapar:** Kaydırıcı. Drag sırasında 500ms throttle; parmak kalkınca son değer kesin.

**Syntax**
```
SLIDER,id,x,y,width,height,minVal,maxVal,currentValue,trackColor,thumbColor,bgColor,isVertical
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float,float | 1,100,100 | |
| 4-5 | width,height | float | 200,40 | |
| 6-7 | minVal,maxVal | float | 0,100 | |
| 8 | currentValue | float | 50 | |
| 9 | trackColor | color | `#0000FF` | Dolu kısım |
| 10 | thumbColor | color | `#FFFFFF` | Sürgü |
| 11 | bgColor | color | `#808080` | Boş kısım |
| 12 | isVertical | choice | 0 | |

**Example**
```
SLIDER,9,100,100,200,40,0,100,50,#0000FF,#FFFFFF,#808080,0
```

**Update:** `UPDATE,9,75` · `UPDATE,9,$+10` · `UPDATE,9,$-10`
**SetState:** `currentValue`, `trackColor`, `thumbColor`, `bgColor`, `minVal`, `maxVal`
**UI Event:** `SLIDER_CHANGE:id:değer` (drag'de throttle, bırakınca kesin)
**Out of range:** `SLIDER_HATA:id:mevcutDeğer`

---

#### POT (aliases: POTENTIOMETER, ROUNDSLIDER)

**Ne yapar:** Döner potansiyometre (round slider).

**Syntax**
```
POT,id,cx,cy,radius,minVal,maxVal,currentValue,startAngle,sweepAngle,isClockwise,trackColor,thumbColor,arcThickness,textColor,textSize
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-4 | id,cx,cy,radius | int,float³ | 1,200,200,50 | |
| 5-7 | minVal,maxVal,currentValue | float³ | 0,100,0 | |
| 8 | startAngle | float | 135 | Klasik |
| 9 | sweepAngle | float | 270 | Klasik |
| 10 | isClockwise | choice | 1 | |
| 11 | trackColor | color | `#00FF00` | |
| 12 | thumbColor | color | `#FFFFFF` | |
| 13 | arcThickness | float | 0 | 0=radius*0.15 |
| 14 | textColor | color | `#FFFFFF` | |
| 15 | textSize | float | 24 | 0=gizli |

**Example**
```
POT,10,200,200,50,0,100,50,135,270,1,#00FF00,#FFFFFF,0,#FFFFFF,24
```

**Update:** `UPDATE,10,değer` · `$+5` · `$-5`
**SetState:** `currentValue`, `trackColor`, `thumbColor`, `arcThickness`, `startAngle`, `sweepAngle`, `isClockwise`
**UI Event:** `POT_CHANGE:id:değer` (throttle 500ms)

---

#### ENCODER (alias: ENC)

**Ne yapar:** Döner knob + merkez buton. Her adımda tık sesi, ekranda birikmiş adım sayacı.

**Syntax**
```
ENCODER,id,cx,cy,radius,knobColor,indicatorColor,isButton,label,textColor,textSize,stepsPerTurn,borderColor,tickColor,centerDotColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-4 | id,cx,cy,radius | int,float³ | 1,200,200,40 | |
| 5 | knobColor | color | `#333333` | Gövde |
| 6 | indicatorColor | color | `#FFFFFF` | Gösterge çizgisi |
| 7 | isButton | choice | 0 | |
| 8 | label | str | `` | |
| 9 | textColor | color | `#FFFFFF` | |
| 10 | textSize | float | 0 | 0=radius*0.45 |
| 11 | stepsPerTurn | int | 12 | 1-360, adım açısı = 360° / N |
| 12 | borderColor | color | `#FFFFFF` | TRANSPARENT=çizilmez |
| 13 | tickColor | color | `#FFFFFF` | TRANSPARENT=çizilmez |
| 14 | centerDotColor | color | `#FFFFFF` | TRANSPARENT=çizilmez |

**Example**
```
ENCODER,11,200,200,40,#333333,#FFFFFF,1,VOLUME,WHITE,16,12,#FFFFFF,#FFFFFF,#FFFFFF
```

**SetState:** `knobColor`, `indicatorColor`, `visualRotation`, `label`, `textColor`, `textSize`, `stepsPerTurn`
**UI Event:** dönerken `ENCODER:id:+N` / `ENCODER:id:-N` · butona basınca `BTN_CLICK:sayfa,id,label`
**Auto:** her adımda 20ms ToneGenerator beep · sayaç bırakılınca 0'a döner

---

#### SWITCH

**Ne yapar:** Aç/Kapa anahtarı. Renkli pill veya görselli (imgOn/imgOff).

**Syntax**
```
SWITCH,id,x,y,width,height,state,leftColor,rightColor,btnColor,isVertical,imgOn,imgOff
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float² | 1,100,100 | |
| 4-5 | width,height | float | 80,40 | |
| 6 | state | choice | 0 | 0=Kapalı, 1=Açık |
| 7 | leftColor | color | `#FF0000` | |
| 8 | rightColor | color | `#00FF00` | |
| 9 | btnColor | color | `#FFFFFF` | |
| 10 | isVertical | choice | 0 | |
| 11 | imgOn | str | `` | `images/` resim |
| 12 | imgOff | str | `` | `images/` resim |

**Example**
```
SWITCH,8,100,100,80,40,0,#FF0000,#00FF00,#FFFFFF,0,on.png,off.png
```

**Update:** `UPDATE,8,1` · **SetState:** `state`, `leftColor`, `rightColor`, `btnColor`, `isVertical`
**UI Event:** `SWITCH_CHANGE:id:state`
**Anti-bounce:** 350ms (rapid double-tap toggle önlenir)

---

#### RADIO

**Ne yapar:** Radyo butonu (yuvarlak veya kare).

**Syntax**
```
RADIO,id,x,y,width,height,style,state,borderColor,fillColor,bgColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float² | 1,100,100 | |
| 4-5 | width,height | float | 40,40 | |
| 6 | style | choice | 0 | 0=Yuvarlak, 1=Kare (✓) |
| 7 | state | choice | 0 | |
| 8 | borderColor | color | `#FFFFFF` | |
| 9 | fillColor | color | `#FF0000` | |
| 10 | bgColor | color | `TRANSPARENT` | |

**Example**
```
RADIO,15,100,100,40,40,0,0,#FFFFFF,#FF0000,TRANSPARENT
```

**Update:** `UPDATE,15,1` · **SetState:** `state`, `borderColor`, `fillColor`
**UI Event:** `RADIO_CHANGE:id:state`

---

#### TRISTATE

**Ne yapar:** Üç konumlu anahtar (sol/orta/sağ).

**Syntax**
```
TRISTATE,id,x,y,width,height,state,color0,color1,color2,label0,label1,label2,bgColor,alpha
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float² | 1,100,100 | |
| 4-5 | width,height | float | 150,40 | |
| 6 | state | choice | 0 | 0,1,2 |
| 7-9 | color0,color1,color2 | color³ | RED,YELLOW,GREEN | |
| 10-12 | label0,label1,label2 | str³ | Düşük,Orta,Yüksek | |
| 13 | bgColor | color | `TRANSPARENT` | |
| 14 | alpha | float | 1.0 | |

**Example**
```
TRISTATE,16,100,100,150,40,0,#FF0000,#FFFF00,#00FF00,Düşük,Orta,Yüksek,TRANSPARENT,1.0
```

**Update:** `UPDATE,16,2` · **SetState:** `state`, `color0`, `color1`, `color2`
**UI Event:** `TRISTATE_CHANGE:id:state`
**Not:** Her dokunuşta yeni state hesaplanır (dokunulan bölgeye göre) ve gönderilir.

---

#### LISTBOX

**Ne yapar:** Kaydırılabilir liste.

**Syntax**
```
LISTBOX,id,x,y,width,height,items,selectedIndex,bgColor,textColor,selectColor,textSize,rowSpacing,showBorder,cornerRadius
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float² | 1,50,50 | |
| 4-5 | width,height | float | 150,100 | |
| 6 | items | list | `Elma|Armut|Muz` | `|` veya `,` ayırıcı |
| 7 | selectedIndex | int | 0 | |
| 8 | bgColor | color | `#000000` | |
| 9 | textColor | color | `#FFFFFF` | |
| 10 | selectColor | color | `#FF00FF` | |
| 11 | textSize | float | 16 | |
| 12 | rowSpacing | float | 0 | Ek satır boşluğu |
| 13 | showBorder | choice | 1 | |
| 14 | cornerRadius | float | 5 | |

**Example**
```
LISTBOX,17,50,50,150,100,Elma|Armut|Muz,0,#000000,#FFFFFF,#FF00FF,16,0,1,5
```

**Special:** `LISTBOX_SET_ITEMS,id,item1|item2` · `LISTBOX_SET_INDEX,id,index`
**Update:** `SETSTATE,17,selectedIndex,2`
**UI Event:** `LISTBOX_CHANGE:id:index:metin`

---

#### LISTBOXR *(v3.6)*

**Ne yapar:** İkonlu + alt başlıklı + aksiyonlu zengin liste.

**Syntax** (items ayrı satırlar halinde, her item `icon|label|subtitle|action`)
```
LISTBOXR,id,x,y,width,height,selectedIndex,bgColor,textColor,subtitleColor,selectColor,textSize,showBorder,cornerRadius,rowSpacing,padding,rowBgColor,item1,item2,...
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2-3 | x,y | float | 50,50 | |
| 4-5 | width,height | float | 250,300 | |
| 6 | selectedIndex | int | 0 | |
| 7 | bgColor | color | `#000000` | |
| 8 | textColor | color | `#FFFFFF` | |
| 9 | subtitleColor | color | `#808080` | |
| 10 | selectColor | color | `#FF00FF` | |
| 11 | textSize | float | 18 | 0=otomatik |
| 12 | showBorder | choice | 1 | |
| 13 | cornerRadius | float | 8 | |
| 14 | rowSpacing | float | 10 | |
| 15 | padding | float | 10 | |
| 16 | rowBgColor | color | `#333333` | Seçili olmayan satır |
| 17+ | item1,item2,... | str | | `icon|label|subtitle|action` |

**Item formatı:** `iconFile|label|subtitle|action` — boş alan için `NULL`
Action, içinde virgül taşıyabilir (ör. `PAGE,3`) → parser bunu otomatik gruplar.

**Example**
```
LISTBOXR,18,50,50,250,300,0,#000000,#FFFFFF,#808080,#FF00FF,18,1,8,10,10,#333333,music.png|Radyo|Favoriler|PAGE,3,settings.png|Ayarlar|Sistem|PAGE,5
```

**SetState:** `selectedIndex`, `rowSpacing`, `cornerRadius`, `bgColor`
**UI Event:** `LISTBOXR_SEL:id:index:label` + satırın `action`'ı çalışır

---

#### COMBOBOX

**Ne yapar:** Açılır liste. Dokununca geçici bir ListBox açar.

**Syntax**
```
COMBOBOX,id,x,y,width,height,items,selectedIndex,textColor,bgColor,textSize
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-3 | id,x,y | int,float² | 1,50,50 | |
| 4-5 | width,height | float | 150,40 | |
| 6 | items | list | `Seçenek1|Seçenek2|Seçenek3` | |
| 7 | selectedIndex | int | 0 | |
| 8 | textColor | color | `#FFFFFF` | |
| 9 | bgColor | color | `#333333` | |
| 10 | textSize | float | 16 | |

**Example**
```
COMBOBOX,19,50,50,150,40,Seçenek1|Seçenek2,0,#FFFFFF,#333333,16
```

**Update:** `UPDATE,19,index` · **SetState:** `selectedIndex`, `textColor`, `bgColor`
**UI Event:** `COMBOBOX_CHANGE:id:index:metin`
**Not:** Geçici liste id=9999 ile eklenir; başka bir yere dokununca kaldırılır.

---

#### ICON

**Ne yapar:** Çok durumlu ikon. `imageNames` listesinden `currentState`. indeksli resmi gösterir.

**Syntax**
```
ICON,id,x,y,size,image1.png,image2.png,...,currentState
```

**Parse heuristic:** Son parametre tam sayı içeriyorsa ve `.` içermiyorsa `currentState`'dir; aksi halde resim dosyası adıdır.

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1 | id | int | 1 | |
| 2-3 | x,y | float | 100,100 | |
| 4 | size | float | 50 | Kare boyutu |
| 5+ | imageNames | str | `icon1.png,icon2.png` | `icons/` klasöründen |
| son | currentState | int | 0 | (opsiyonel) |

**Example**
```
ICON,35,100,100,50,moon.png,sun.png,cloud.png,0
```

**Update:** `UPDATE,35,1` → `ICON_STATE:id:state` döner
**SetState:** `currentState`, `size`

---

#### RGBPICKER (aliases: COLORPICKER, PICKER)

**Ne yapar:** RGBW renk seçici — hue halka + saturation merkez + W slider. Drag sırasında 500ms throttle.

**Syntax**
```
RGBPICKER,id,cx,cy,outerRadius,ringThickness,hue,saturation,currentR,currentG,currentB,currentW,showW,sliderHeight
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-4 | id,cx,cy,outerRadius | int,float³ | 1,200,200,100 | |
| 5 | ringThickness | float | 25 | 2..outerR*0.6 |
| 6 | hue | float | 0 | 0-360 |
| 7 | saturation | float | 100 | 0-100 |
| 8-10 | currentR,G,B | int³ | 255,0,0 | 0-255 |
| 11 | currentW | int | 0 | 0-255 |
| 12 | showW | choice | 1 | |
| 13 | sliderHeight | float | 20 | 8-60 |

**Example**
```
RGBPICKER,14,200,200,100,25,0,100,255,0,0,0,1,20
```

**Output:** `R30G50B150W0` (throttle 500ms; bırakınca kesin)
**SetState:** `currentR`, `currentG`, `currentB`, `currentW`, `showW`
**Behavior:** Hue drag → W=0 · W>0 → RGB=0 (temiz beyaz) · W=0 → son RGB cache'ten geri gelir

---

### 3.3 Progress / Meter Widgets

#### PROGRESS

**Ne yapar:** İlerleme çubuğu. Uyarı + kritik + gradyan.

**Syntax**
```
PROGRESS,id,x,y,w,h,minVal,maxVal,currentValue,color,bgColor,isVertical,textSize,textColor,fontType,isBold,warningVal,warningColor,criticalVal,criticalColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-5 | id,x,y,w,h | int,float⁴ | 1,100,100,200,30 | |
| 6-8 | min,max,cur | float³ | 0,100,50 | |
| 9-10 | color,bgColor | color² | GREEN,DKGRAY | |
| 11 | isVertical | choice | 0 | |
| 12 | textSize | float | -1 | -1=height*0.4 |
| 13 | textColor | color | `#FFFFFF` | |
| 14 | fontType | int | 1 | |
| 15 | isBold | choice | 1 | |
| 16 | warningVal | float | -1 | -1=kapalı |
| 17 | warningColor | color | `#FFFF00` | |
| 18 | criticalVal | float | -1 | |
| 19 | criticalColor | color | `#FF0000` | |
| 20 | useGradient | choice | 0 | |
| 21-23 | gradientStart,Mid,End | color³ | GREEN,YELLOW,RED | |

**Example**
```
PROGRESS,12,100,100,200,30,0,100,50,#00FF00,#333333,0,-1,#FFFFFF,1,1,70,#FFFF00,90,#FF0000,0
```

**Update:** `UPDATE,12,75` · `$+5` / `$-5` → `PROGRESS_CHANGE:id:değer`
**Out of range:** `PROGRESS_HATA:id:mevcutDeğer`

---

#### SEGMENTBAR (aliases: SEGBAR, VUMETER)

**Ne yapar:** Segmentli bar / VU metre. Renk kararı segmentin **değerine** göre verilir (LED konumuna değil); segment sayısı değişse bile eşikler otomatik doğru yere düşer.

**Syntax**
```
SEGMENTBAR,id,x,y,w,h,segmentCount,isVertical,minVal,maxVal,currentValue,segmentColor,warningColor,criticalColor,warningVal,criticalVal,gap,bgColor,borderColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-5 | id,x,y,w,h | int,float⁴ | 1,50,50,40,300 | |
| 6 | segmentCount | int | 10 | 1-50 |
| 7 | isVertical | choice | 1 | |
| 8-10 | min,max,cur | float³ | 0,100,0 | |
| 11 | segmentColor | color | `#00FF00` | |
| 12 | warningColor | color | `#FFAA00` | |
| 13 | criticalColor | color | `#FF0000` | |
| 14 | warningVal | float | 70 | -1=kapalı |
| 15 | criticalVal | float | 90 | -1=kapalı |
| 16 | gap | float | 3 | Segment aralığı |
| 17 | bgColor | color | `#333333` | Boş segment |
| 18 | borderColor | color | `#FFFFFF` | |

**Example**
```
SEGMENTBAR,13,50,50,40,300,10,1,0,100,45,#00FF00,#FFAA00,#FF0000,70,90,3,#333333,#FFFFFF
```

**Update:** `UPDATE,13,75` · `$+10` / `$-10` · **Fast:** `U,13,120`
**SetState:** `currentValue`, `segmentCount`, `segmentColor`, `warningVal`, `criticalVal`
**UI Event:** `SEGMENTBAR_CHANGE:id:değer`
**Related:** `VU_SOURCE,barRmsId,barPeakId` (bkz. VU group) veya ESP32 tarafı `U,id,val`

---

#### GAUGE

**Ne yapar:** İbreli gösterge. Uyarı + kritik + gradyan + akıllı tick.

**Syntax**
```
GAUGE,id,cx,cy,radius,minVal,maxVal,warningVal,currentValue,color,warningColor,textColor,textSize,startAngle,sweepAngle,isClockwise,arcThickness,stepSize,criticalVal,criticalColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-4 | id,cx,cy,radius | int,float³ | 1,200,200,80 | |
| 5-7 | min,max,warn | float³ | 0,100,80 | |
| 8 | currentValue | float | 50 | |
| 9 | color | color | `#00FF00` | |
| 10 | warningColor | color | `#FF0000` | |
| 11 | textColor | color | `#FFFFFF` | |
| 12 | textSize | float | 32 | |
| 13 | startAngle | float | 135 | |
| 14 | sweepAngle | float | 270 | |
| 15 | isClockwise | choice | 1 | |
| 16 | arcThickness | float | 0 | 0=radius*0.18 |
| 17 | stepSize | float | 10 | 0=otomatik "nice number" |
| 18 | criticalVal | float | -1 | -1=kapalı |
| 19 | criticalColor | color | `#FF0000` | |
| 20 | useGradient | choice | 0 | |
| 21-23 | gradientStart,Mid,End | color³ | GREEN,RED,RED | |

**Example**
```
GAUGE,32,200,200,80,0,100,80,45,#00FF00,#FF0000,#FFFFFF,32,135,270,1,0,10,-1,#FF0000,0
```

**Update:** `UPDATE,32,75` → `GAUGE_CHANGE:id:değer`
**SetState:** `currentValue`, `startAngle`, `sweepAngle`, `arcThickness`, `stepSize`, `criticalVal`, `useGradient`

---

#### RULER

**Ne yapar:** Bölmeli cetvel. Uyarı + kritik + gradyan.

**Syntax**
```
RULER,id,x,y,width,height,minVal,maxVal,criticalVal,currentValue,isHighCritical,isVertical,color,criticalColor,textColor,textSize,bgColor,stepSize,warningVal,warningColor,useGradient,gradientStartColor,gradientMidColor,gradientEndColor
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-5 | id,x,y,w,h | int,float⁴ | 1,50,50,30,150 | |
| 6-9 | min,max,crit,cur | float⁴ | 0,100,20,0 | |
| 10 | isHighCritical | choice | 1 | 1=üstü kritik, 0=altı kritik |
| 11 | isVertical | choice | 1 | |
| 12 | color | color | `#00FF00` | |
| 13 | criticalColor | color | `#FF0000` | |
| 14 | textColor | color | `#FFFFFF` | |
| 15 | textSize | float | 18 | |
| 16 | bgColor | color | `TRANSPARENT` | |
| 17 | stepSize | float | 0 | 0=otomatik range/10 |
| 18 | warningVal | float | -1 | |
| 19 | warningColor | color | `#FFFF00` | |
| 20 | useGradient | choice | 0 | |
| 21-23 | gradientStart,Mid,End | color³ | GREEN,YELLOW,RED | |

**Update:** `UPDATE,34,currentValue` → `RULER_CHANGE:id:değer`

---

#### THERM

**Ne yapar:** Termometre göstergesi.

**Syntax**
```
THERM,id,cx,topY,width,height,minVal,maxVal,currentValue,color,criticalVal,textColor,textSize,isBold
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-5 | id,cx,topY,w,h | int,float⁴ | 1,100,50,40,150 | |
| 6-8 | min,max,cur | float³ | 0,100,50 | |
| 9 | color | color | `#FF0000` | |
| 10 | criticalVal | float | 0 | 0=kapalı |
| 11 | textColor | color | `#FFFFFF` | |
| 12 | textSize | float | -1 | -1=otomatik bulb*0.6 |
| 13 | isBold | choice | 1 | |

**Update:** `UPDATE,30,75` · `$+1` / `$-1` → `THERMOMETER_CHANGE:id:değer`
**SetState:** `currentValue`, `criticalVal`, `color`
**Render:** Kritik üstüne çıkınca otomatik kırmızı

---

#### BATT

**Ne yapar:** Segmentli pil göstergesi.

**Syntax**
```
BATT,id,x,y,width,height,segments,currentLevel,color
```

**Params**

| # | Ad | Tip | Default | Notlar |
|---|-----|-----|---------|--------|
| 1-5 | id,x,y
