<div align="center">

# EasyOLED

**Arduino ve ESP için modern, dokunmatik HMI kontrol paneli.**

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://github.com/Easyoled/easyoled/blob/main/LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Android%206.0%2B-green.svg)](https://github.com/Easyoled/easyoled/wiki/Installation)
[![Version](https://img.shields.io/badge/Version-v3.0.0-blue.svg)](https://github.com/Easyoled/easyoled/releases/latest)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9.x-purple.svg)](https://kotlinlang.org/)

📱 Tek bir APK. Tek bir HC-05. Sınırsız arayüz.

[**İndir**](https://github.com/Easyoled/easyoled/releases/latest) ·
[**Wiki**](https://github.com/Easyoled/easyoled/wiki) ·
[**Hızlı Başlangıç**](https://github.com/Easyoled/easyoled/wiki/Quick-Start) ·
[**Örnekler**](https://github.com/Easyoled/easyoled/wiki/Examples)

</div>

---

## 📖 EasyOLED Nedir?

EasyOLED, Arduino'dan gelen **basit seri port komutlarıyla** Android
telefonunu tam donanımlı bir **dokunmatik operatör paneli** haline
getirir.

- **Bluetooth (HC-05)** üzerinden Arduino'ya bağlanır
- Gelen komutları ekrana çizer (`TEXT,1,Merhaba,50,100,32`)
- Kullanıcı etkileşimlerini Arduino'ya geri gönderir (`BTN_CLICK`, `SLIDER_CHANGE`)
- 30+ hazır nesne: Gauge, Slider, POT, Encoder, SegmentBar, Needle, Image, GIF, Button, ListBox, ComboBox, QRCode, MediaPlayer...
- **Mark Dirty Render** ile sadece değişen bölge yeniden çizilir → yüksek performans
- **SAVE_SCENE** ile tasarımı JSON'a kaydet, geri yükle
- **LINKS** komutuyla Arduino adına HTTPS isteği yap (ESP8266'nın yapamadığı)
- **Internal VU Metre** — müzik çalarken ses dalgasını yakalar
- **İnternet radyosu** — M3U playlist + QR paylaşım

---

## 🚀 Hızlı Kurulum

### 1. APK'yı indir

[**⬇️ En Son Sürümü İndir (v3.0.0)**](https://github.com/Easyoled/easyoled/releases/latest)

- Dosya: `EasyOLED-v3.0.0.apk` (~18 MB)
- Gereksinim: Android 6.0 (Marshmallow) ve üzeri

### 2. Arduino'ya bağla

HC-05 Bluetooth modülünü Arduino'ya bağla, telefonla eşleştir, EasyOLED menüsünden cihazı seç.

### 3. İlk komutunu gönder

```cpp
#include <SoftwareSerial.h>
SoftwareSerial bt(10, 11);

void setup() {
  bt.begin(9600);
  delay(500);
  bt.println("TEXT,1,Merhaba EasyOLED,50,100,32,#FFFFFF,LEFT");
  bt.println("BTN,2,BaSS_la,50,200,150,50,#00AA00,#FFFFFF,20");
}
```

**Sonuç:** Ekranda "Merhaba EasyOLED" yazısı + yeşil buton görünür.

📖 **Detaylı adımlar için:** [Quick Start Wiki](https://github.com/Easyoled/easyoled/wiki/Quick-Start)

---

## 📚 Dokümantasyon

Tam dokümantasyon **[Wiki](https://github.com/Easyoled/easyoled/wiki)** sayfasında:

| Sayfa | İçerik |
|---|---|
| [**Installation**](https://github.com/Easyoled/easyoled/wiki/Installation) | APK kurulumu, izinler, klasör yapısı |
| [**Quick Start**](https://github.com/Easyoled/easyoled/wiki/Quick-Start) | 5 dakikada ilk panel |
| [**Python Setup**](https://github.com/Easyoled/easyoled/wiki/Python-Setup) | Python araçları için ortam kurulumu |
| [**Objects**](https://github.com/Easyoled/easyoled/wiki/Objects) | 30+ nesnenin tüm parametreleri |
| [**Controls**](https://github.com/Easyoled/easyoled/wiki/Controls) | UPDATE, SETSTATE, LINK, TIMER, LINKS ve diğer komutlar |
| [**Events**](https://github.com/Easyoled/easyoled/wiki/Events) | Arduino'ya gönderilen tüm mesajlar |
| [**Special Characters**](https://github.com/Easyoled/easyoled/wiki/Special-Characters) | Türkçe, emoji, sembol kodlama |
| [**Advanced Features**](https://github.com/Easyoled/easyoled/wiki/Advanced-Features) | GROUP, ICON, LINK, TIMER detaylı |
| [**Examples**](https://github.com/Easyoled/easyoled/wiki/Examples) | 5 hazır senaryo (VU, radyo, dashboard) |
| [**Troubleshooting**](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) | Yaygın hatalar ve çözümleri |
| [**FAQ**](https://github.com/Easyoled/easyoled/wiki/FAQ) | Sık sorulan sorular |
| [**Changelog**](https://github.com/Easyoled/easyoled/wiki/Changelog) | Sürüm notları |

---

## 🆘 Sorun mu Yaşıyorsun?

Aşağıdaki adımları **sırayla** dene. Çoğu sorun bu aşamalarda çözülür.

### 1️⃣ Wiki'ye Bak

| Sorun | Çözüm Sayfası |
|---|---|
| Kurulum / izinler | [Installation](https://github.com/Easyoled/easyoled/wiki/Installation) |
| Klasör seçimi / dosya erişimi | [Installation](https://github.com/Easyoled/easyoled/wiki/Installation) |
| Komutlar çalışmıyor | [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) |
| Bluetooth bağlanmıyor | [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) |
| Resim / font görünmüyor | [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting) |
| Türkçe karakterler bozuk | [Special Characters](https://github.com/Easyoled/easyoled/wiki/Special-Characters) |
| Python kütüphanesi eksik | [Python Setup](https://github.com/Easyoled/easyoled/wiki/Python-Setup) |
| Genel sorular | [FAQ](https://github.com/Easyoled/easyoled/wiki/FAQ) |

### 2️⃣ Mevcut Issue'ları Ara

Belki başka biri aynı sorunu yaşamıştır:

🔍 [**GitHub Issues'da ara**](https://github.com/Easyoled/easyoled/issues)

### 3️⃣ Yeni Issue Aç

Sorununu bulamadıysan **yeni bir issue aç** — şu bilgileri ekle:

- **Android sürümü** ve cihaz modeli
- **EasyOLED sürümü** (☰ menü → Hakkında)
- **Arduino / HC-05 modeli**
- **Hata mesajının tam metni** (ekran görüntüsü ideal)
- **Yeniden üretme adımları** ("şunu yap, sonra bunu yap")

🐛 [**Yeni Issue Aç**](https://github.com/Easyoled/easyoled/issues/new)

### 4️⃣ Yine de Çözülmediyse — Bize Yaz

Wiki'de yok, Issue'da yok, çözemiyorsan doğrudan bize ulaş:

📧 **easyoledproject@gmail.com**

**E-posta'ya ekle:**
- Sorunun kısa açıklaması
- Ne denediğin (hangi wiki sayfası, hangi komut)
- Tam hata mesajı
- Cihaz bilgisi

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

EasyOLED açık kaynaklı bir projedir. Katkı sağlamak istersen:

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

**EasyOLED** turns your Android phone into a full-featured **touch
operator panel** for Arduino / ESP via simple serial commands.

### What It Does

- Connects to Arduino via **Bluetooth (HC-05)**
- Renders 30+ object types: Gauge, Slider, POT, Encoder, SegmentBar,
  Needle, Image, GIF, Button, ListBox, QRCode, MediaPlayer...
- **Mark Dirty Render** — only redraws changed regions (high performance)
- **SAVE_SCENE** — persist design to JSON
- **LINKS** command — HTTPS requests on behalf of Arduino
- **Internal VU Meter** — captures audio waveform for live visualization

### Quick Install

[**⬇️ Download Latest Release (v3.0.0)**](https://github.com/Easyoled/easyoled/releases/latest)

**Requirements:** Android 6.0+ · HC-05 / HC-06 Bluetooth module

### Documentation

Full docs on the [**Wiki**](https://github.com/Easyoled/easyoled/wiki):

- [Installation](https://github.com/Easyoled/easyoled/wiki/Installation)
- [Quick Start](https://github.com/Easyoled/easyoled/wiki/Quick-Start)
- [Python Setup](https://github.com/Easyoled/easyoled/wiki/Python-Setup)
- [Objects](https://github.com/Easyoled/easyoled/wiki/Objects)
- [Troubleshooting](https://github.com/Easyoled/easyoled/wiki/Troubleshooting)
- [FAQ](https://github.com/Easyoled/easyoled/wiki/FAQ)

### Need Help?

1. Check the [**Wiki**](https://github.com/Easyoled/easyoled/wiki)
2. Search [**existing Issues**](https://github.com/Easyoled/easyoled/issues)
3. [**Open a new Issue**](https://github.com/Easyoled/easyoled/issues/new)
4. Still stuck? Email us: 📧 **easyoledproject@gmail.com**

### License

**CC BY-NC 4.0** — Free for personal use, **not** for commercial use.

Commercial licensing: 📧 [easyoledproject@gmail.com](mailto:easyoledproject@gmail.com)

### Authors

- **Şafak Ağustoslu** — Electronics Technician
- **Tanalp Ağustoslu** — AI Engineer (MSc)
