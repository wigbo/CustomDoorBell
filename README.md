# 🏠 Smart Access & Doorbell Terminal (AMOLED + PoE)

Dieses Projekt beschreibt ein hochintegriertes, unterputz-verbautes Zutritts- und Informations-Terminal. Es kombiniert biometrische Sicherheit (Fingerabdruck), RFID/NFC-Technologie und ein hochauflösendes Status-Display in einem professionellen Gehäuse.

Das System ist für maximale Stabilität und nahtlose Integration in **Home Assistant** (via ESPHome) konzipiert.

---

## 🏗 System-Architektur (Master/Slave-Prinzip)

Um maximale Zuverlässigkeit zu gewährleisten, ist das System strikt in zwei spezialisierte Einheiten getrennt:

1. **Haupt-ESP (Master / Backend):** Ein **Waveshare ESP32-S3-PoE**. Er übernimmt die kritische Logik, die Netzwerkverbindung via LAN, die Stromversorgung (PoE) sowie die Anbindung der Sensoren (Fingerprint & NFC). Er dient als Kommunikations-Bridge zu Home Assistant.
2. **Display-ESP (Slave / Frontend):** Ein **Waveshare ESP32-S3-AMOLED-1.91**. Er dient ausschließlich der Visualisierung und UI-Interaktion. Er erhält Befehle via UART vom Master und führt keine eigene Zutrittslogik aus.

---

## 🚀 Features & Funktionalität

### 1. Sicherheit & Zutritt
* **Biometrie:** Grow R503 Fingerabdruck-Scanner mit steuerbarem RGB-LED-Ring.
* **NFC/RFID:** PN532 Modul für schlüssellosen Zugang via Tags oder Smartphones.
* **Thin-Client Logik:** Die Authentifizierung erfolgt lokal, die Berechtigungsprüfung (Zeitpläne, User-Rechte) jedoch zentral in Home Assistant.

### 2. Dynamische UI-Modi (Display-Cases)
Das Display reagiert blitzschnell auf Events via Home Assistant API:
* **Willkommen:** Personalisierter Screen mit Hintergrundbild (`hintergrund.png`) und Familienname.
* **Klingel-Event:** Cyan-farbene Glocke mit "Breathing"-Animation (Pulsieren).
* **Zutritts-Feedback:** Visuelle Bestätigung für "OFFEN" (Grün) oder "STOPP" (Rot) mit High-Speed Animationen.
* **Paket- & Info-Modus:** Speziell definierte Screens für Lieferdienste oder Abfall-Erinnerungen.

### 3. Hardware-Optimierung
* **AMOLED-Schutz:** Echter "Hard-Power-Cut" der Panel-Stromversorgung (GPIO38) im Idle-Mode zur Vermeidung von "OLED-Glow" und Einbrennen.
* **Performance:** Optimiert für 8MB Octal-PSRAM zur ruckelfreien Verarbeitung großer Fonts (Montserrat Bold 700) und Full-Screen Animationen bei 60 FPS.

---

## 🛠 Hardware-Komponenten

| Komponente | Modell | Funktion |
| :--- | :--- | :--- |
| **Backend MCU** | Waveshare ESP32-S3-PoE | LAN, PoE-Wandlung, MQTT, Sensor-Management. |
| **Frontend MCU** | Waveshare ESP32-S3-AMOLED 1.91" | Grafik-Engine & UI (LVGL). |
| **Biometrie** | Grow R503 | Fingerabdruck-Sensor. |
| **NFC Reader** | PN532 (I2C Mode) | RFID/NFC Identifikation. |
| **Service-Port** | Magnet-Pogo-Pin (4-pol) | Externer USB-Zugang für Diagnose/Update. |
| **Puffer** | Elko 1000µF / 10V | Stabilisierung der AMOLED-Spannungsversorgung. |

---

## 🔌 Verkabelung & Pinout

### Kommunikation (Inter-ESP & Sensoren)
| Verbindung | Master (Backend) Pin | Slave (Frontend) Pin |
| :--- | :--- | :--- |
| **UART Bridge** | GPIO 19 (TX) | GPIO 1 (RX) |
| **UART Bridge** | GPIO 18 (RX) | GPIO 0 (TX) |
| **I2C (NFC)** | GPIO 4 (SDA), GPIO 5 (SCL) | - |
| **UART (R503)** | GPIO 16 (RX), GPIO 17 (TX) | - |

### Stromversorgung (Power Distribution)
Das gesamte Terminal wird über PoE gespeist. Der Master verteilt 5V an alle Komponenten.
* **VBUS (5V) Loop:** PoE-Board ➔ Elko ➔ Display (VSYS) ➔ NFC (VCC) ➔ R503 (VCC).
* **Gemeinsames Ground (GND):** Alle Komponenten müssen eine gemeinsame Masse nutzen.

---

## 🧰 Wartung (Service-Konzept)

Da das Gerät fest verbaut ist, wurde ein **magnetischer Pogo-Pin-Anschluss** an der Unterseite integriert. Dieser führt den nativen USB-Port des Backend-ESP nach außen.

* **Rettungs-Szenario:** Über eine "Serial Bridge Firmware" auf dem Master kann der Frontend-ESP (Display) getunnelt werden, um Grafik-Updates ohne Ausbau durchzuführen.
* **OTA:** Beide ESPs unterstützen Over-the-Air Updates via Home Assistant/ESPHome.

---

## 🔌 API Services (Home Assistant)

| Dienst | Parameter | Wirkung |
| :--- | :--- | :--- |
| `set_idle` | - | Kappt physikalisch die Panel-Power (GPIO38). |
| `trigger_doorbell` | - | Startet Puls-Animation (15s Timer). |
| `show_access_granted`| - | Zeigt "OFFEN" Icon & Text (5s). |
| `show_custom` | `message`, `icon` | Dynamischer Text mit Icons (herz, muell, info, etc.). |

---

## 📦 Installation & Build-Notes

1.  **Dateien:** `hintergrund.png` (536x240px) in `/config/esphome/` ablegen.
2.  **Kompilierung:** Aufgrund der Octal-PSRAM Nutzung ist ein `Clean Build` zwingend erforderlich.
3.  **Code-Stabilität:** Zur Vermeidung von `LoadProhibited` Abstürzen werden Labels zur Laufzeit via C++ (`lv_label_set_text`) statt statischer YAML-Bindungen gesetzt.

---
*Dokumentation Version 1.0 - Zusammenführung von Hardware v12 und Firmware v18.*
