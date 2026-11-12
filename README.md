# 🪵 Holzwaage

Eine selbstgebaute Waage unter dem Holzkorb neben dem Ofen. Sie erkennt **automatisch**, wann Holz nachgelegt oder der Korb neu befüllt wird, und rechnet den Verbrauch in **kg, kWh und Raummeter** um. Alles landet nativ in Home Assistant, ganz ohne Cloud.

Kein Knopfdruck beim Nachlegen, kein Strichliste führen. Einmal kalibrieren, dann läuft es.

## ✨ Mehrwert

- **Vollautomatische Erfassung**: Holz aus dem Korb nehmen = Verbrauch, Korb abnehmen und neu befüllen = Nachfüllen.
- **Nachlegen als Vorgang**: Mehrere Entnahmen innerhalb weniger Minuten (z. B. beim Anzünden) zählen als ein Nachlegen.
- **Verbrauch in kg, kWh und Raummeter**, passend zur Holzart (Buche, Fichte, ...).
- **Energie-Dashboard-tauglich**: Die Sensoren sind `total_increasing`, gut für Tages-/Wochen-/Jahresverbrauch mit `utility_meter`.
- **Wenig Aufwand**: Kalibrierung in zwei Klicks, Werte bleiben nach Neustart und OTA-Updates erhalten.
- **Lokal und offen**: ESPHome, ein ESP32, ein paar Euro Hardware.

## 🧰 Benötigte Teile

| Teil | Hinweis |
|---|---|
| ESP32 Entwicklungsboard | Config nutzt `esp32dev` |
| Wägezelle (Load Cell) | Je nach Korb und Holzmenge, z. B. 50 kg Balkenzelle |
| HX711 Verstärkermodul | 24-Bit-ADC für Wägezellen |
| Mechanik | Zwei Platten (z. B. Holz/Alu) mit Abstandshaltern, die Zelle dazwischen, Korb obendrauf |
| Kabel, Netzteil (5 V USB) | |

### Verdrahtung

| HX711 | ESP32 |
|---|---|
| DT | GPIO14 |
| SCK | GPIO12 |
| VCC | 3,3 V |
| GND | GND |

Die Pins stehen als `substitutions` oben in der Datei und sind leicht änderbar. GPIO12 ist ein Strapping-Pin. Hängt der ESP beim Booten, SCK kurz abziehen.

## 🚀 Installation

1. [ESPHome](https://esphome.io) installieren.
2. `secrets.yaml.example` nach `secrets.yaml` kopieren und WLAN-Daten eintragen.
3. Flashen:
   ```
   esphome run holzwaage.yaml
   ```
4. Das Gerät wird in Home Assistant automatisch erkannt.

## ⚖️ Kalibrierung

1. Leeren Korb auf die Waage stellen und **Kalibrierung 1: Korb leer (Tara)** drücken.
2. Referenzgewicht in den Korb legen, den Wert in **Referenzgewicht** eintragen und **Kalibrierung 2: Referenzgewicht** drücken.
3. Gewicht wieder entfernen. Der Status wechselt nach rund 10 s auf **Bereit**.

Die Kalibrierung bleibt bei normalen OTA- und USB-Updates erhalten. Nur ein komplettes Löschen des Flashs oder eine Änderung der Globals setzt sie zurück.

## 🔁 So funktioniert es

| Status | Bedeutung |
|---|---|
| Bereit | Gewicht ruhig, letzter Wert wird gemerkt |
| Nachlegen | Gewicht weicht um mehr als *Mindestaenderung* ab, Abschluss nach *Nachlegen Abschlusszeit* (Standard 5 min) Ruhe |
| Korb abgenommen | Gewicht liegt unter der Schwelle, es wird nichts gebucht |

Wird der Korb wieder aufgestellt und ist 8 s ruhig, gilt *neues Gewicht minus letztes Gewicht* als Befüllung.

## 📊 Entitäten in Home Assistant

**Sensoren**: Holz im Korb, Holz verbrannt gesamt (kg), Brennwert verbrannt gesamt (kWh), Holz verbrannt Raummeter (Rm), Holz nachgefuellt gesamt, Letztes Nachlegen, Letzte Befuellung, Status.

**Einstellungen**: Referenzgewicht, Heizwert Holz, Gewicht Holz pro Raummeter, Schwelle Korb abgenommen, Mindestaenderung, Nachlegen Abschlusszeit.

**Buttons**: Kalibrierung 1 und 2, Zaehler zuruecksetzen.

### Holzart wechseln

Heizwert und Raummetergewicht gelten für das Holz, das gerade nachgelegt wird (festgehalten zu Beginn des Nachlegens). Stelle beide Werte um, bevor du die Holzart wechselst. Alte Summen bleiben unverändert.

Richtwerte für lufttrockene Scheite (ca. 33 cm), grobe Schätzungen:

| Holzart | kg/Rm | kWh/kg | kWh/Rm |
|---|---|---|---|
| Buche / Eiche | 400 bis 450 | ca. 4,0 | ca. 1.600 bis 1.800 |
| Esche | 380 bis 420 | ca. 4,0 | ca. 1.500 bis 1.700 |
| Birke | 330 bis 360 | ca. 4,1 | ca. 1.350 bis 1.450 |
| Fichte / Kiefer | 290 bis 330 | ca. 4,1 bis 4,3 | ca. 1.200 bis 1.400 |

Die kWh sind der Heizwert des Holzes. Die tatsächliche Wärmeabgabe ist durch den Wirkungsgrad des Ofens geringer.

## ⚠️ Hinweise

- Die Schwelle *Korb abgenommen* muss kleiner sein als das negative Korbgewicht.
- Die Werte dieser README sind Richtwerte. Genauigkeit hängt von Holzfeuchte und Schichtung ab.
- Getestet wurde bisher nur der Build der Konfiguration.
