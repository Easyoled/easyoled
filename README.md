<div align="center">

# EasyOLED / EasyLCD Mobile

**Arduino ve ESP için modern, dokunmatik HMI kontrol paneli + Eski Android telefonları karakter LCD'ye dönüştüren mobil uygulama.**

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://github.com/Easyoled/easyoled/blob/main/LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android%206.0%2B-green.svg)](https://github.com/Easyoled/easyoled/wiki/Installation)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.x-purple.svg)](https://kotlinlang.org/)

📱 Tek bir APK. Tek bir HC-05. Sınırsız arayüz.
📟 Eski telefonlarınız artık karakter LCD.

[**EasyOLED İndir**](https://github.com/Easyoled/easyoled/releases/latest) ·
[**EasyLCD Mobile v2.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0) ·
[**Wiki**](https://github.com/Easyoled/easyoled/wiki) ·
[**Hızlı Başlangıç**](https://github.com/Easyoled/easyoled/wiki/Quick-Start)

</div>

---

## 📦 Bu Repo İki Proje Barındırır

| Proje | Amaç | Android |
|---|---|---|
| 🎨 **[EasyOLED](#-easyoled-nedir)** | Dokunmatik HMI kontrol paneli | 6.0+ |
| 📟 **[EasyLCD Mobile](#-easylcd-mobile-nedir)** | Eski telefonları karakter LCD'ye dönüştürür | v1.0: 2.3.6+ / v2.0: 6.0+ |

Aşağıda her iki projenin de tanıtımı ve kurulum linkleri bulunur.

---

## 📖 EasyOLED Nedir?

EasyOLED, Arduino'dan gelen **basit seri port komutlarıyla** Android
telefonunu tam donanımlı bir **dokunmatik operatör paneli** haline
getirir.

- **Bluetooth (HC-05)** veya **USB-OTG (FT232RL/CH340/CP2102)** üzerinden Arduino'ya bağlanır
- Gelen komutları ekrana çizer (`TEXT,1,Merhaba,50,100,32`)
- Kullanıcı etkileşimlerini Arduino'ya geri gönderir (`BTN_CLICK`, `SLIDER_CHANGE`)
- 30+ hazır nesne: Gauge, Slider, POT, Encoder, SegmentBar, Needle, Image, GIF, Button, ListBox, ComboBox, QRCode, MediaPlayer...
- **Mark Dirty Render** ile sadece değişen bölge yeniden çizilir → yüksek performans
- **SAVE_SCENE** ile tasarımı JSON'a kaydet, geri yükle
- **LINKS** komutuyla Arduino adına HTTPS isteği yap (ESP8266'nın yapamadığı)
- **Internal VU Metre** — müzik çalarken ses dalgasını yakalar
- **İnternet radyosu** — M3U playlist + QR paylaşım
- **Telefon sensörleri** — 9 tür (ışık, ivme, jiroskop, manyetik, yakınlık, adım, basınç, batarya, GPS)
- **MediaStore müzik** — cihazdaki tüm müzikleri dizin bağımsız listeler
- **Modüler müzik kontrolü** — `MP_PLAY`, `MP_STOP`, `MP_NEXT`, `MP_VOL_UP`... (Arduino'dan uzaktan)
- **SAF uyumu** — Android 10+ scoped storage ile uyumlu (Storage Access Framework)

### 🆕 v3.5'te Ne Yeni?

**Bağlantı**
- 🔌 **USB-OTG desteği** — Bluetooth'a alternatif olarak FT232RL/CH340/CP2102 ile kablolu bağlantı
- 📶 **Bluetooth kapalı uyarısı** — açmak ister misiniz dialog + sistem izni
- 🌐 **WiFi kontrolü** — `LINKS`/`WEATHER` öncesi otomatik ağ kontrolü

**Yeni Yetenekler**
- 🌡️ **Telefon sensörleri** — 9 tür; tek seferde bir sensör, `SENSOR_START`/`STOP`
- 🎵 **MediaPlayer modüler kontrol** — `MP_MODE`, `MP_PLAY`, `MP_PAUSE`, `MP_STOP`, `MP_NEXT`, `MP_PREV`, `MP_VOL_UP`, `MP_VOL_DOWN`, `MP_MUTE`, `MP_STATUS`
- 💾 **MediaStore müzik listeleme** — dizin taraması yok, Music/ klasörü zorunlu değil
- 🖼️ **Image yuvarlak modu** — `shape=1` parametresi (x,y=merkez, width=çap)
- ⚡ **GIF hız kontrolü** — `speed` parametresi (0.25x – 4.0x)

**Performans & Uyumluluk**
- 🚀 **Sayfa bazlı render optimizasyonu** — pasif sayfadaki nesneler çizilmez
- 📂 **Menüden sahne restore** — 0-9 slot listesi, tarih/boyut ile
- 🛡️ **SAF-aware SAVE_SCENE** — Android 10+ scoped storage uyumlu
- ✅ **ACK desteği** — `UPDATE`, `DEL`, `MP_*` gibi kritik komutlara onay

### 🚀 EasyOLED Hızlı Kurulum

**1. APK'yı indir**

[**⬇️ En Son Sürümü İndir**](https://github.com/Easyoled/easyoled/releases/latest)

- Dosya: `EasyOLED-v3.5.0.apk` (~19 MB)
- Gereksinim: Android 6.0 (Marshmallow) ve üzeri

**2. Arduino'ya bağla**

**Seçenek A — Bluetooth:**
HC-05 Bluetooth modülünü Arduino'ya bağla, telefonla eşleştir, EasyOLED menüsünden cihazı seç.

**Seçenek B — USB-OTG:**
FT232RL / CH340 / CP2102 modülünü OTG kablosuyla telefona bağla, EasyOLED menüsünden `USB Bağlantısı` seç.

**3. İlk komutunu gönder**

```cpp
#include <SoftwareSerial.h>
SoftwareSerial bt(10, 11);

void setup() {
  bt.begin(9600);
  delay(500);
  bt.println("TEXT,1,Merhaba EasyOLED,50,100,32,#FFFFFF,LEFT");
  bt.println("BTN,2,BaSS_la,50,200,150,50,#00AA00,#FFFFFF,20");

  // 🆕 v3.5: MP3 kontrolü
  bt.println("MEDIAPLAYER,201,300,300,120,120,#FF0000,#333333,0,0");
  bt.println("MP_MODE,201,0");     // MP3 modu
  bt.println("MP_PLAY,201");       // Çalmaya başla

  // 🆕 v3.5: Işık sensörü
  bt.println("SENSOR_START,light,500");   // 500ms'de bir ışık değeri
}
