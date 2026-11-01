# Team 2 · U-Wert der Gebäudehülle

BFH Minor BXMid03 «Data Management und BIM», HS 2026/27.

## Problem und Zielgruppe
*Wer* braucht *welche Entscheidung* und *wann*? (eine Rolle, eine Aufgabe):

**Rolle:** Informationsmanagement (BIM-Koordinator AN).
**Aufgabe:** Innert 2 Arbeitstagen nach dem IFC-Upload auf die CDE
entscheiden, ob die Lieferung des Architekten für den Energienachweis
taugt: **freigeben** oder **mit Befund zurückweisen**.
**Wann:** Meilenstein Ende Phase 32, vor der Baueingabe.

Heute (IST, siehe `workflow/`) kommt das IFC **ohne U-Werte**, die
Wandaufbauten separat als PDF. Die Bauphysik fragt per Telefon oder Mail
nach, rechnet in Excel und prüft die Grenzwerte von Hand. Am Schluss ist der
Nachweis eingereicht, das Modell aber weiterhin ohne U-Werte.

Das MVP prüft automatisch und wiederholbar, ob jede Aussenwand im IFC einen
U-Wert trägt (S1) und ob er den Grenzwert einhält (S2). Die Bauphysik nutzt
das Ergebnis von S2 direkt weiter; der Architekt erhält die Fehlerliste mit
GlobalId.

## Input
- IFC-Modell: Dateiname, Version, Quelle (siehe `models/README.md`)

**IFC-Modelle** (alle IFC4, Herkunft und Änderungen in `models/README.md`):

| Datei | Zweck | U-Wert |
|---|---|---|
| `models/Team2_Wand_U_ok.ifc` | positiver Testfall | 0.15 W/(m²K) |
| `models/Team2_Wand_U_zu_hoch.ifc` | negativer Testfall | 0.28 W/(m²K) |
| `models/Team2_Wand_ohne_U.ifc` | Zusatzfall IST-Zustand | fehlt |

Synthetische Testdaten: abgeleitet vom Kursmodell `Demo_Holzrahmen_korrigiert.ifc`
(Kursrepository `LTplus-AG/bxmid03-hs26`, Ordner `modelle/`), ergänzt um
`Pset_WallCommon`. Die U-Werte sind gesetzte Testwerte, nicht berechnet. Alle
drei Dateien enthalten dieselbe Wand, GlobalId `2ucZRLBGP4uxZW$9i1VAZ8`.

- Regel/Anforderung: Pfad in `requirements/`

**Regel/Anforderung:**

* `requirements/Team2_U-Wert.ids` (IDS 1.0, zwei Spezifikationen S1 und S2)
* `requirements/anforderungsmatrix.md` (Anforderung, Datenpfad, Einheit,
  Quelle Grenzwert)

## Verarbeitung
Welche Objekte (Scope), welcher Datenpfad (Pset.Property, Einheit), welche Regel?

| | |
|---|---|
| **Scope** | alle `IfcWall` mit `Pset_WallCommon.IsExternal = TRUE` |
| **Datenpfad** | `Pset_WallCommon.ThermalTransmittance` (IfcPropertySingleValue an der Wandinstanz) |
| **Einheit** | W/(m²K); IFC-Typ `IfcThermalTransmittanceMeasure`, SI, solange das Modell keine eigene Einheit zuweist |
| **Regel S1** «U-Wert geliefert» | vorhanden und > 0 |
| **Regel S2** «Grenzwert» | > 0 und ≤ 0.17 W/(m²K) (Einzelbauteilnachweis Neubau, Wand gegen aussen, SIA 380/1:2016) |

Ablauf im Skript `checks/ids_check.py`:

1. IDS und IFC einlesen; die Regeln kommen aus der IDS, nicht aus dem Code.
2. Wände im Scope **zählen**. Bei 0 Wänden bricht der Lauf mit Exitcode 2 ab,
   denn ein leerer Lauf belegt nichts.
3. Pro Wand S1 und S2 prüfen. Status: `bestanden`, `durchgefallen` oder
   `fehlend`. Ein fehlender U-Wert wird **nicht** als 0 gewertet.
4. Ergebnis als CSV und Protokoll nach `results/` schreiben.


## Output
Wo liegt das Ergebnis (`results/`)? Jede Zeile: GlobalId, Objekt, Ist-Wert, Soll-Wert, Ergebnis, Grund.

Ergebnisse liegen in `results/`:

| Datei | Inhalt |
|---|---|
| `positiv_U_ok.csv` | Ergebnis positiver Testfall |
| `negativ_U_zu_hoch.csv` | Ergebnis negativer Testfall |
| `fehlend_ohne_U.csv` | Ergebnis Zusatzfall IST |
| `lauf.log` | Datum, Befehl, Python-Version, Hash von Modell und IDS, Zählung, Exitcode |
| `zusammenfassung.md` | Ergebnis lesbar zusammengefasst |

Jede CSV-Zeile = eine Wand × eine Spezifikation (Trennzeichen `;`, öffnet
direkt in Excel):

| Spalte | Beispiel (negativer Fall, S2) |
|---|---|
| Spezifikation | `S2 U-Wert Aussenwand Grenzwert SIA 380/1 Neubau` |
| GlobalId | `2ucZRLBGP4uxZW$9i1VAZ8` |
| Objekt | `IFCWALL 'Wall #1'` |
| Regel | `Pset_WallCommon.ThermalTransmittance > 0 und <= 0.17 und Typ double` |
| Ist-Wert | `0.28` |
| Soll-Wert | `> 0 und <= 0.17` |
| Einheit | `W/(m²K)` |
| Ergebnis | `durchgefallen` |
| Grund | `Wert muss höchstens 0.17 sein` |
| Regelquelle | `Team2_U-Wert.ids@a303c90c` (Datei und Hash der IDS = Regelversion) |

## Setup
Was muss installiert sein? (Version angeben, z. B. IFC-Lite 0.41.0, Python 3.12)

* **Python 3.10 oder neuer**, getestet mit Python 3.11, 3.12 und 3.13.18.
* **Keine Zusatzpakete**, nur die Python-Standardbibliothek.
* Prüfskript `checks/ids_check.py`, Version 0.1.0.

## Ausführen
```
# exakter Befehl oder Klickpfad, Schritt für Schritt
```

## Testfälle
| Fall | Datei | Erwartet | Tatsächlich |
|---|---|---|---|
| positiv | | bestanden | |
| negativ | | durchgefallen | |

## Grenzen und offene Punkte
- 

## Team
Wer hat was beigetragen? (Commits zeigen es, hier kurz zusammenfassen)
