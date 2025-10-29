# Balkonpflanzen Automation

Dieses Projekt automatisiert die Überwachung und Pflege von Balkonpflanzen mithilfe von Sensoren, Datenanalyse und Dashboard-Funktionen.

## Projektstruktur

- **app/**: Hauptanwendung mit Modulen für Sensoren, Datenkollektor, Server, Analyse, Dashboard und Automatisierung.
- **balkongarten/**: Zusätzliche Ressourcen und Daten.
- **data/**: Rohdaten, verarbeitete Daten und Ergebnisse.
- **docs/**: Dokumentation und Anleitungen.
- **notebooks/**: Jupyter Notebooks für Datenanalyse und Feature Engineering.
- **scripts/**: Hilfsskripte für Datenverarbeitung und Systemmanagement.
- **tests/**: Testfälle zur Sicherstellung der Funktionalität.

## Hauptfunktionen

- Sensorintegration zur Erfassung von Umweltdaten
- Automatisierte Datenerfassung und Speicherung
- Datenanalyse und Feature Engineering
- Dashboard zur Visualisierung und Steuerung
- Unterstützung für Docker und InfluxDB/MariaDB

## Installation

1. Abhängigkeiten installieren:
   ```cmd
   pip install -r requirements.txt
   ```
2. Konfiguration anpassen (z.B. `config.py`, `.env`)
3. Anwendung starten:
   ```cmd
   python app/main.py
   ```

## Tests ausführen

```cmd
python -m pytest tests/
```

## Weitere Informationen

- Siehe `docs/` für detaillierte Anleitungen
- Siehe `CHANGELOG.md` für Änderungen
- Kontakt: Projektteam
