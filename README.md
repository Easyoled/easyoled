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
```

**Sonuç:** Ekranda "Merhaba EasyOLED" yazısı + yeşil buton + müzik player + ışık sensörü verisi görünür.

📖 **Detaylı adımlar:** [Quick Start Wiki](https://github.com/Easyoled/easyoled/wiki/Quick-Start)

---

## 📟 EasyLCD Mobile Nedir?

**EasyLCD Mobile**, çekmecedeki eski Android telefonları **HD44780 uyumlu karakter LCD ekran** olarak kullanmanızı sağlar. Donanım gerekmez — telefon ekranı 4x20 (veya 2x20 büyük mod) karakter LCD gibi çalışır.

Arduino seri porttan (Bluetooth HC-05 veya USB-OTG FT232RL) basit komutlar gönderir, uygulama bunları HD44780 uyumlu şekilde ekrana çizer.

> 💡 **Neden EasyLCD Mobile?** Elektronik kartlar ve LCD'ler pahalı hale geldi. Çoğumuzun çekmecesinde eski ama çalışan Android telefonlar var. EasyLCD Mobile, bu telefonları **sıfır maliyetle** birer karakter LCD'ye dönüştürür.

### ✨ Özellikler

- 📶 **Çift bağlantı:** Bluetooth (HC-05) **+ USB-OTG** (FT232RL / CH340 / CP2102)
- 🔤 **Türkçeleştirme + Unicode + emoji** (tek hücre)
- 🎨 **24 slot EPROM** karakter hafızası (JSON)
- 🔊 **TTS** (sesli okuma) — TR/EN, satır bazlı
- 📷 **QR / Barkod** okuyucu (ML Kit + CameraX)
- 🌐 **ESP / WiFi** — URL passthrough (ThingSpeak dahil)
- 🌡️ **Termal / idle uyarı** — pil ısısı + boşta kalma bildirimi
- 🖥️ **4x20 / 2x20 büyük mod** + font seçimi (satır bazlı)
- 💾 **Kalıcı ayarlar** (`settings.json`) + **hata loglama** (`error.log`)
- 🚫 **Donanım gerekmez** — telefon ekranı LCD olur

### 📱 Hangi Sürüm Size Uygun?

EasyLCD Mobile **iki sürüm** halinde sunulur:

#### 📟 EasyLCD Mobile v1.0 — Eski Android için

> 🎯 **Hedef:** Android **2.3.6 – 5.1** aralığındaki çok eski telefonlar
> 🛠️ **Teknoloji:** MIT App Inventor (orijinal sürüm)

| Özellik | Durum |
|---|---|
| Android 2.3.6+ | ✅ |
| Bluetooth (HC-05) | ✅ |
| Temel LCD komutları | ✅ |
| QR / Barkod | ✅ |
| EPROM (sınırlı) | ✅ |
| USB-OTG, TTS, ESP/WiFi, termal uyarı | ❌ |

📥 **[EasyLCD Mobile v1.0 İndir](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v1.0.0)**  
📖 **[v1.0 Wiki Sayfası](https://github.com/Easyoled/easyoled/wiki/EasyLCD-Mobile)**

#### 📟 EasyLCD Mobile v2.0 — Modern Android için ⭐

> 🎯 **Hedef:** Android **6.0 (Marshmallow) ve üzeri**
> 🛠️ **Teknoloji:** Kotlin + Android Studio (sıfırdan yeniden yazıldı)

**v1.0'a göre yeni:**
- 🔌 **USB-OTG** (FT232RL / CH340 / CP2102)
- 🌐 **ESP / WiFi URL passthrough** (ThingSpeak, HTTP API)
- 🔊 **TTS** (sesli okuma) — TR/EN
- 🌡️ **Termal / idle uyarı** + renkli Toast
- 🎨 **Büyük mod (2x20)** + satır bazlı font
- 💾 **JSON tabanlı** EPROM + ayarlar
- 📋 **Hata log dosyaları** (`error.log` / `debug.log`)
- 🖼️ **Unicode + emoji** tek hücre desteği
- 🎯 **HD44780 uyumlu** cursor davranışı

> ✅ **Komut yapısı v1.0 ve EasyLCD Modülü PCB ile aynıdır** (`LCDWR`, `LCDDD`, `LCDBL`, `LCDSC`...). Sadece bazı komutlar farklıdır (`ESPGO/ESPSD` vs `ESPON/ESPOF`, `LCDBM`, `LCDF1-4`).

📥 **[EasyLCD Mobile v2.0.0 İndir](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0)**  
📖 **[v2.0 Wiki Sayfası](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0)**

#### 🤔 Hangisini İndirmeliyim?

| Telefonun Android Sürümü | Önerilen Sürüm |
|---|---|
| **2.3.6 – 4.4** (çok eski) | [**v1.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v1.0.0) (tek seçenek) |
| **5.0 – 5.1** (Lollipop) | [**v1.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v1.0.0) (v2.0 test edilmedi) |
| **6.0 – 9.0** (Marshmallow – Pie) | [**v2.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0) ⭐ |
| **10.0 ve üzeri** | [**v2.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0) ⭐ |

**Nasıl öğrenirim?** → Telefon **Ayarlar → Telefon Hakkında → Android Sürümü**

### 🚀 EasyLCD Mobile Hızlı Kurulum (v2.0)

**1. APK'yı indir**

[**⬇️ EasyLCD Mobile v2.0.0 İndir**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0)

**2. Arduino'ya bağla**

HC-05 Bluetooth modülünü Arduino'ya bağla (veya USB-OTG + FT232RL kullan).

**3. İlk komutunu gönder**

```cpp
#include <SoftwareSerial.h>
SoftwareSerial bt(10, 11);

void setup() {
  bt.begin(9600);
  delay(500);

  bt.println("LCDDD");                              // Ekranı temizle
  bt.println("LCDWR 1,1-Merhaba EasyLCD Mobile");
  bt.println("LCDWR 2,1-Sicaklik: 25.5 C");
  bt.println("LCDWR 3,1-Nem: 60%");
  bt.println("LCDWR 4,1-Durum: OK");
  bt.println("LCDBL 7");                            // Arka ışık seviye 7
}
```

**Sonuç:** Ekranda 4 satır yazı + ayarlanmış arka ışık görünür.

📖 **Detaylı komut seti ve örnekler:** [v2.0 Wiki](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0)

---

## 📚 Dokümantasyon

Tam dokümantasyon **[Wiki](https://github.com/Easyoled/easyoled/wiki)** sayfasında.

### EasyOLED Wiki Sayfaları

| Sayfa | İçerik |
|---|---|
| [**Installation**](https://github.com/Easyoled/easyoled/wiki/Installation) | APK kurulumu, izinler, klasör yapısı |
| [**Quick Start**](https://github.com/Easyoled/easyoled/wiki/Quick-Start) | 5 dakikada ilk panel |
| [**Python Setup**](https://github.com/Easyoled/easyoled/wiki/Python-Setup) | Python araçları için ortam kurulumu |
| [**Objects**](https://github.com/Easyoled/easyoled/wiki/Objects) | 30+ nesnenin tüm parametreleri |
| [**Controls**](https://github.com/Easyoled/easyoled/wiki/Controls) | UPDATE, SETSTATE, LINK, TIMER, LINKS, MP_*, SENSOR_* |
| [**Events**](https://github.com/Easyoled/easyoled/wiki/Events) | Arduino'ya gönderilen tüm mesajlar |
| [**Special Characters**](https://github.com/Easyoled/easyoled/wiki/Special-Characters) | Türkçe, emoji, sembol kodlama |
| [**Advanced Features**](https://github.com/Easyoled/easyoled/wiki/Advanced-Features) | GROUP, ICON, LINK, TIMER detaylı |
| [**Examples**](https://github.com/Easyoled/easyoled/wiki/Examples) | 5 hazır senaryo |
| [**Troubleshooting**](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) | Yaygın hatalar ve çözümleri |
| [**FAQ**](https://github.com/Easyoled/easyoled/wiki/FAQ) | Sık sorulan sorular |

### EasyLCD Mobile Wiki Sayfaları

| Sayfa | İçerik |
|---|---|
| [**EasyLCD Mobile v1.0**](https://github.com/Easyoled/easyoled/wiki/EasyLCD-Mobile) | Android 2.3.6 – 5.1 için eski sürüm |
| [**EasyLCD Mobile v2.0**](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0) | **Modern sürüm** — Kotlin, USB-OTG, ESP/WiFi, TTS |

---

## 🆘 Sorun mu Yaşıyorsun?

Aşağıdaki adımları **sırayla** dene. Çoğu sorun bu aşamalarda çözülür.

### 1️⃣ Wiki'ye Bak

**EasyOLED için:**

| Sorun | Çözüm Sayfası |
|---|---|
| Kurulum / izinler | [Installation](https://github.com/Easyoled/easyoled/wiki/Installation) |
| Komutlar çalışmıyor | [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) |
| Bluetooth / USB bağlanmıyor | [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) |
| Türkçe karakterler bozuk | [Special Characters](https://github.com/Easyoled/easyoled/wiki/Special-Characters) |
| Müzik listesi boş görünüyor | [FAQ — MediaStore](https://github.com/Easyoled/easyoled/wiki/FAQ) |
| Sensör verisi gelmiyor | [FAQ — Sensörler](https://github.com/Easyoled/easyoled/wiki/FAQ) |
| Python kütüphanesi eksik | [Python Setup](https://github.com/Easyoled/easyoled/wiki/Python-Setup) |
| Genel sorular | [FAQ](https://github.com/Easyoled/easyoled/wiki/FAQ) |

**EasyLCD Mobile için:**

| Sorun | Çözüm Sayfası |
|---|---|
| v1.0 kullanıyorum, sorun var | [v1.0 Wiki](https://github.com/Easyoled/easyoled/wiki/EasyLCD-Mobile) |
| v2.0 kullanıyorum, sorun var | [v2.0 Wiki](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0) |
| USB-OTG / şarj sorunu | [v2.0 Wiki — USB-OTG](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0) |

### 2️⃣ Mevcut Issue'ları Ara

🔍 [**GitHub Issues'da ara**](https://github.com/Easyoled/easyoled/issues)

### 3️⃣ Yeni Issue Aç

**Hangi proje için:** EasyOLED / EasyLCD Mobile v1.0 / EasyLCD Mobile v2.0

🐛 [**Yeni Issue Aç**](https://github.com/Easyoled/easyoled/issues/new)

### 4️⃣ Yine de Çözülmediyse — Bize Yaz

📧 **easyoledproject@gmail.com**

---

## 🔗 Bağlantılar

- 🏠 **Ana Sayfa:** https://github.com/Easyoled/easyoled
- 📖 **Wiki:** https://github.com/Easyoled/easyoled/wiki
- 🚀 **Releases:** https://github.com/Easyoled/easyoled/releases
- 🐛 **Issues:** https://github.com/Easyoled/easyoled/issues
- 💬 **Discussions:** https://github.com/Easyoled/easyoled/discussions

---

## 📄 Lisans

**CC BY-NC 4.0** — Creative Commons Attribution-NonCommercial 4.0

- ✅ Kişisel kullanım serbest
- ✅ Değiştirip kullanabilirsin
- ✅ Kaynak göstermek şartıyla paylaşabilirsin
- ❌ **Ticari kullanım yasak**

**Ticari kullanım** için: 📧 [easyoledproject@gmail.com](mailto:easyoledproject@gmail.com)

Tam metin → [LICENSE](https://github.com/Easyoled/easyoled/blob/main/LICENSE)

---

## 👥 Yazarlar

| | |
|---|---|
| **Şafak Ağustoslu** | Elektronik Teknikeri |
| **Tanalp Ağustoslu** | Yapay Zeka Mühendisi (Yüksek Lisans) |

---

## 🙏 Katkı

EasyOLED / EasyLCD Mobile açık kaynaklı projelerdir. Katkı sağlamak istersen:

- 🐛 **Hata bildir** → [Issues](https://github.com/Easyoled/easyoled/issues)
- 💡 **Özellik öner** → [Discussions](https://github.com/Easyoled/easyoled/discussions)
- 📝 **Dokümantasyon düzelt** → [Wiki](https://github.com/Easyoled/easyoled/wiki)
- 💻 **Kod katkısı** → [Contributing](https://github.com/Easyoled/easyoled/wiki/Contributing)

---

<div align="center">

**⭐ Faydalı bulduysan yıldız vermeyi unutma!**

© 2026 Şafak Ağustoslu & Tanalp Ağustoslu

</div>

---

## 🇬🇧 English

### Two Projects in One Repo

| Project | Purpose | Android |
|---|---|---|
| 🎨 **[EasyOLED](#-what-is-easyoled)** | Touch HMI control panel | 6.0+ |
| 📟 **[EasyLCD Mobile](#-what-is-easylcd-mobile)** | Turns old phones into character LCDs | v1.0: 2.3.6+ / v2.0: 6.0+ |

---

### 📖 What is EasyOLED?

**EasyOLED** turns your Android phone into a full-featured **touch
operator panel** for Arduino / ESP via simple serial commands.

- Connects via **Bluetooth (HC-05)** or **USB-OTG (FT232RL/CH340/CP2102)**
- Renders 30+ object types: Gauge, Slider, POT, Encoder, SegmentBar,
  Needle, Image, GIF, Button, ListBox, QRCode, MediaPlayer...
- **Mark Dirty Render** — only redraws changed regions
- **SAVE_SCENE** — persist design to JSON
- **LINKS** command — HTTPS requests on behalf of Arduino
- **Internal VU Meter** — captures audio waveform
- **Phone sensors** — 9 types (light, accel, gyro, mag, prox, step, pressure, battery, GPS)
- **MediaStore music** — lists all music device-wide, folder-independent
- **Modular music control** — `MP_PLAY`, `MP_STOP`, `MP_NEXT`, `MP_VOL_UP`...
- **SAF-aware** — Android 10+ scoped storage compatible

### 🆕 What's New in v3.5?

**Connection**
- 🔌 **USB-OTG support** — wired alternative to Bluetooth
- 📶 **Bluetooth off warning** — enable dialog + system permission
- 🌐 **WiFi check** — automatic network validation before `LINKS`/`WEATHER`

**New Capabilities**
- 🌡️ **Phone sensors** — 9 types; one at a time, `SENSOR_START`/`STOP`
- 🎵 **Modular MediaPlayer** — `MP_MODE`, `MP_PLAY`, `MP_PAUSE`, `MP_STOP`, `MP_NEXT`, `MP_PREV`, `MP_VOL_UP`, `MP_VOL_DOWN`, `MP_MUTE`, `MP_STATUS`
- 💾 **MediaStore music listing** — no directory scan, no Music/ requirement
- 🖼️ **Image circular mode** — `shape=1` (x,y=center, width=diameter)
- ⚡ **GIF speed control** — `speed` param (0.25x – 4.0x)

**Performance & Compatibility**
- 🚀 **Page-based render optimization** — inactive pages not drawn
- 📂 **Restore scene from menu** — 0-9 slot list with date/size
- 🛡️ **SAF-aware SAVE_SCENE** — Android 10+ scoped storage
- ✅ **ACK support** — for `UPDATE`, `DEL`, `MP_*` critical commands

**Download:** [**⬇️ Latest Release**](https://github.com/Easyoled/easyoled/releases/latest)

**Requirements:** Android 6.0+ · HC-05 / HC-06 Bluetooth module OR USB-OTG cable + FT232RL/CH340

---

### 📟 What is EasyLCD Mobile?

**EasyLCD Mobile** turns old Android phones into **HD44780-compatible character LCDs** (4x20 or 2x20 big mode). No hardware needed — the phone screen works like a character LCD.

Arduino sends simple serial commands (via Bluetooth HC-05 or USB-OTG FT232RL), the app draws them like an HD44780 LCD.

#### Which Version?

| Your Android Version | Recommended |
|---|---|
| **2.3.6 – 5.1** (very old) | [**v1.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v1.0.0) |
| **6.0+** (modern) | [**v2.0** ⭐](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0) |

**EasyLCD Mobile v2.0 highlights:**
- 📶 Dual connection: **Bluetooth + USB-OTG** (FT232RL / CH340)
- 🌐 **ESP / WiFi URL passthrough** (ThingSpeak, any HTTP API)
- 🔊 **TTS** (TR/EN text-to-speech)
- 🌡️ **Thermal / idle warnings**
- 🎨 **Big mode (2x20)** + per-line font selection
- 📋 Error logging (`error.log` / `debug.log`)
- 🔤 Unicode + emoji single-cell rendering

> ✅ Command structure is **identical** to v1.0 and EasyLCD Module PCB (`LCDWR`, `LCDDD`, `LCDBL`, `LCDSC`...).

**Download v2.0:** [**⬇️ EasyLCD Mobile v2.0.0**](https://github.com/Easyoled/easyoled/releases/tag/easylcd_mobile_v2.0.0)  
**Wiki v2.0:** [EasyLCD_Mobile_v2.0](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0)

---

### 📚 Documentation

Full docs on the [**Wiki**](https://github.com/Easyoled/easyoled/wiki).

**EasyOLED pages:**
[Installation](https://github.com/Easyoled/easyoled/wiki/Installation) ·
[Quick Start](https://github.com/Easyoled/easyoled/wiki/Quick-Start) ·
[Objects](https://github.com/Easyoled/easyoled/wiki/Objects) ·
[Controls](https://github.com/Easyoled/easyoled/wiki/Controls) ·
[Events](https://github.com/Easyoled/easyoled/wiki/Events) ·
[Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) ·
[FAQ](https://github.com/Easyoled/easyoled/wiki/FAQ)

**EasyLCD Mobile pages:**
[**v1.0 Wiki**](https://github.com/Easyoled/easyoled/wiki/EasyLCD-Mobile) ·
[**v2.0 Wiki**](https://github.com/Easyoled/easyoled/wiki/EasyLCD_Mobile_v2.0)

---

### 🆘 Need Help?

1. Check the [**Wiki**](https://github.com/Easyoled/easyoled/wiki)
2. Search [**existing Issues**](https://github.com/Easyoled/easyoled/issues)
3. [**Open a new Issue**](https://github.com/Easyoled/easyoled/issues/new) — mention which project (EasyOLED / EasyLCD v1 / EasyLCD v2)
4. Still stuck? Email us: 📧 **easyoledproject@gmail.com**

---

### 📄 License

**CC BY-NC 4.0** — Free for personal use, **not** for commercial use.

Commercial licensing: 📧 [easyoledproject@gmail.com](mailto:easyoledproject@gmail.com)

---

### 👥 Authors

- **Şafak Ağustoslu** — Electronics Technician
- **Tanalp Ağustoslu** — AI Engineer (MSc)
