# 🦾 DataRaptor - Globale Medienanalyse Pipeline

[![Python](https://img.shields.io/badge/python-3.11%2B-blue.svg)](https://python.org)
[![MongoDB](https://img.shields.io/badge/mongodb-6.0%2B-green.svg)](https://mongodb.com)
[![Docker](https://img.shields.io/badge/docker-compose-blue.svg)](https://docker.com)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

> **Modulare Data Lakehouse-Plattform zur automatisierten globalen Medienanalyse mit Deutschland-Bezug**

DataRaptor sammelt, analysiert und übersetzt automatisch internationale Nachrichtenartikel aus 1500+ RSS-Feeds und extrahiert deutschlandrelevante Inhalte für umfassende Medienbeobachtung.

## 🚀 Quick Start

```bash
# Repository klonen
git clone https://github.com/Watchkido/DataRaptor.git
cd DataRaptor

# Pipeline starten
docker-compose up -d --build

# Feeds sammeln
python src/dataraptor/DatenSammelnContainer/FEEDSCRAPPER/main.py

# Web-GUI öffnen
streamlit run src/dataraptor/DatenSammelnContainer/FEEDSCRAPPER/rss_suchen_gui.py
```

## 📋 Inhaltsverzeichnis

- [🎯 Features](#-features)
- [🏗️ Architektur](#️-architektur)
- [🛠️ Installation](#️-installation)
- [📊 Module & Workflows](#-module--workflows)
- [🗄️ Datenbank-Design](#️-datenbank-design)
- [⚡ Performance & Skalierung](#-performance--skalierung)
- [🔧 Konfiguration](#-konfiguration)
- [📈 Monitoring & Logs](#-monitoring--logs)
- [🧪 Tests](#-tests)
- [🛡️ Sicherheit](#️-sicherheit)
- [🤝 Contributing](#-contributing)

## 🎯 Features

### 🔄 Datenerfassung & -verarbeitung

- **Multi-Source Ingestion**: 1500+ internationale RSS-Feeds
- **Intelligente Deduplizierung**: Mehrstufig via Fingerprint, GUID, Link
- **Content Extraction**: Fallback-Strategien (Readability → Trafilatura → Newspaper3k → Selenium)
- **Anti-Bot Protection**: User-Agent Rotation, zufällige Delays, Cookie-Handling

### 🌍 Geographische Klassifikation

- **TLD-Analyse**: Länderzuordnung über Domain-Endungen
- **IP-Geolocation**: API-Integration für präzise Standortbestimmung
- **Sprachbasierte Heuristiken**: Mehrsprachige Ländererkennung
- **Kombiniertes Scoring**: Multi-Level-Validierung

### 🧠 NLP & Sentiment-Analyse

- **Deutschland-Bezug Erkennung**: Automatische Relevanz-Klassifikation
- **Sentiment-Analyse**: Stimmungsbewertung pro Artikel
- **Themen-Kategorisierung**: ML-basierte Inhaltsklassifikation
- **Multi-Language Support**: Übersetzung in DE/EN

### 📊 Analytics & Reporting

- **Real-time Dashboard**: Streamlit-basierte Visualisierung
- **Automated Reports**: Wöchentliche Trend-Analysen
- **API-Endpoints**: FastAPI-Server für Datenabfragen
- **Performance Metrics**: Umfassende Erfolgs-KPIs

## 🏗️ Architektur

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Datenquellen  │    │   Verarbeitung  │    │     Serving     │
├─────────────────┤    ├─────────────────┤    ├─────────────────┤
│ • RSS Feeds     │───▶│ • Deduplizierung│───▶│ • FastAPI       │
│ • News APIs     │    │ • NLP/ML        │    │ • Streamlit     │
│ • Web Scraping  │    │ • Übersetzung   │    │ • Reports       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Data Storage Layer                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  MongoDB    │  │ PostgreSQL  │  │ File System │             │
│  │ (Articles)  │  │ (Metadata)  │  │ (Logs/Cache)│             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

### Datenfluss

1. **Collector** → sammelt Artikel aus RSS-Feeds
2. **Processor** → bereinigt, dedupliziert, analysiert
3. **Enricher** → fügt Metadaten, Übersetzungen hinzu
4. **Analyzer** → führt NLP/ML-Analysen durch
5. **Serving** → stellt Daten via API/Dashboard bereit

## 🛠️ Installation

### Voraussetzungen

- Python 3.11+
- Docker & Docker Compose
- MongoDB 6.0+
- 16GB RAM (empfohlen)
- 100GB Speicher

### Setup

1. **Repository klonen**

```bash
git clone https://github.com/Watchkido/DataRaptor.git
cd DataRaptor
```

2. **Umgebung konfigurieren**

```bash
# Python Virtual Environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# oder
venv\Scripts\activate     # Windows

# Abhängigkeiten installieren
pip install -r requirements.txt
```

3. **Konfiguration erstellen**

```bash
# Umgebungsvariablen kopieren
cp .env.example .env

# MongoDB-Verbindung konfigurieren
echo "MONGODB_URI=mongodb://localhost:27017/dataraptor" >> .env
```

4. **Services starten**

```bash
# Infrastruktur mit Docker
docker-compose up -d

# Oder manuell MongoDB starten
mongod --dbpath ./data/mongodb
```

## 📊 Module & Workflows

### 🔍 Core Module

| Modul       | Beschreibung       | Eingabe           | Ausgabe          |
| ----------- | ------------------ | ----------------- | ---------------- |
| `mod_000_*` | Feed-Erfassung     | RSS-URLs          | Rohartikel       |
| `mod_050_*` | Geo-Klassifikation | Artikel-URLs      | Ländercodes      |
| `mod_070_*` | Deutschland-Filter | Artikel-Text      | Relevanz-Score   |
| `mod_080_*` | Content-Extraction | HTML-Seiten       | Bereinigter Text |
| `mod_090_*` | Sentiment-Analyse  | Artikel-Text      | Bewertungen      |
| `mod_100_*` | Reporting          | Analysierte Daten | CSV/JSON Reports |

### 🚦 Workflow-Orchestrierung

**Hauptorchestrator** (`main1.py`):

```python
# Scheduler-gesteuerter Batch-Prozess
def main_pipeline():
    feeds_abrufen()           # 00:00 - Feed-Sammlung
    land_eintragen()          # 01:00 - Geo-Klassifikation
    deutschland_filter()       # 02:00 - Relevanz-Filter
    artikel_abholen()         # 03:00 - Content-Extraction
    sentiment_analyse()       # 04:00 - NLP-Analyse
    reports_generieren()      # 05:00 - Reporting
```

**API-Server** (`main2.py`):

```python
# FastAPI-Endpoints für Datenabfragen
@app.get("/articles")         # Artikel-Suche
@app.get("/analytics")        # Sentiment-Trends
@app.get("/health")          # System-Status
```

### 🔄 Batch-Verarbeitung

```python
# Beispiel: Bulk-Updates für Performance
bulk_ops = [
    UpdateOne(
        {'_id': article['_id']},
        {'$set': {
            'analysis.sentiment': sentiment_score,
            'analysis.keywords': keywords,
            'updated_at': datetime.utcnow()
        }}
    )
    for article in articles_batch
]
collection.bulk_write(bulk_ops, ordered=False)
```

## 🗄️ Datenbank-Design

### MongoDB Collections

#### Articles Collection

```javascript
{
  "_id": ObjectId("..."),
  "guid": "unique-article-id",
  "fingerprint": "content-hash-sha256",
  "title": "Artikel-Titel",
  "content": "Volltext-Inhalt...",
  "link": "https://example.com/article",
  "publication_date": ISODate("2024-01-01T12:00:00Z"),
  "feed_id": ObjectId("..."),
  "feed_title": "Quelle Name",
  "author": "Autor Name",
  "categories": ["Politik", "Wirtschaft"],

  // Geo-Klassifikation
  "country_code": "DE",
  "country_source": "domain_tld",
  "ip_location": {...},

  // Deutschland-Relevanz
  "thema_germany": 1,
  "germany_keywords": ["Berlin", "Bundestag"],

  // NLP-Analyse
  "analysis": {
    "sentiment": 0.65,
    "confidence": 0.89,
    "keywords": ["Politik", "Regierung"],
    "language": "de",
    "topics": ["Politik"]
  },

  // Übersetzungen
  "translations": {
    "de": {
      "title": "Deutsche Übersetzung",
      "content": "Deutscher Inhalt...",
      "summary": "Kurzzusammenfassung"
    },
    "en": {
      "title": "English Translation",
      "content": "English content...",
      "summary": "Brief summary"
    }
  },

  // Metadaten
  "created_at": ISODate("2024-01-01T10:00:00Z"),
  "updated_at": ISODate("2024-01-01T15:30:00Z"),
  "version": 3
}
```

#### Feeds Collection

```javascript
{
  "_id": ObjectId("..."),
  "url": "https://example.com/rss.xml",
  "title": "Feed-Titel",
  "description": "Feed-Beschreibung",
  "language": "de",
  "country": "DE",
  "category": "news",
  "last_fetched": ISODate("2024-01-01T12:00:00Z"),
  "fetch_interval": 3600,
  "active": true,
  "error_count": 0,
  "success_rate": 0.95
}
```

### Indizierung für Performance

```javascript
// Essentielle Indizes
db.articles.createIndex({ guid: 1 }, { unique: true });
db.articles.createIndex({ fingerprint: 1 });
db.articles.createIndex({ link: 1 });
db.articles.createIndex({ publication_date: -1 });
db.articles.createIndex({ feed_id: 1, publication_date: -1 });
db.articles.createIndex({ country_code: 1 });
db.articles.createIndex({ thema_germany: 1 });

// Volltext-Suche
db.articles.createIndex({
  title: "text",
  content: "text",
  "translations.de.content": "text",
  "translations.en.content": "text",
  "analysis.keywords": "text",
});

// Compound Indizes für Analytics
db.articles.createIndex({
  publication_date: -1,
  country_code: 1,
  "analysis.sentiment": 1,
});
```

## ⚡ Performance & Skalierung

### Deduplizierungs-Strategie

**Mehrstufige Duplikatserkennung**:

```python
def create_fingerprint(title, content):
    """Eindeutiger Hash für Duplikatserkennung"""
    text = f"{title} {content}".lower()
    text = re.sub(r'[^\w\s]', '', text)
    text = re.sub(r'\s+', ' ', text).strip()
    return hashlib.sha256(text.encode('utf-8')).hexdigest()

# Upsert mit Multi-Level Matching
result = collection.bulk_write([
    UpdateOne(
        {
            '$or': [
                {'guid': doc['guid']},
                {'fingerprint': doc['fingerprint']},
                {'link': doc['link']}
            ]
        },
        {
            '$setOnInsert': {k: v for k, v in doc.items() if k != 'version'},
            '$set': {'updated_at': datetime.utcnow()},
            '$inc': {'version': 1}
        },
        upsert=True
    )
    for doc in articles_batch
], ordered=False)
```

### Skalierungsstrategien

**Horizontal Scaling**:

```javascript
// Sharding für große Datenmengen
sh.enableSharding("dataraptor");
sh.shardCollection("dataraptor.articles", { publication_date: 1 });

// Oder feed-basiert
sh.shardCollection("dataraptor.articles", { feed_id: 1 });
```

**Memory Optimization**:

```yaml
# mongod.conf
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 8
      journalCompressor: snappy
    collectionConfig:
      blockCompressor: snappy
```

### Performance Monitoring

```python
# Query-Performance analysieren
db.articles.find({...}).explain("executionStats")

# Index-Nutzung überwachen
db.articles.aggregate([{ $indexStats: {} }])

# Langsame Queries identifizieren
db.setProfilingLevel(2, { slowms: 100 })
db.system.profile.find().limit(5).sort({ ts: -1 })
```

## 🔧 Konfiguration

### Umgebungsvariablen (.env)

```env
# MongoDB-Verbindung
MONGODB_URI=mongodb://localhost:27017/dataraptor
MONGODB_DATABASE=dataraptor

# API-Konfiguration
API_HOST=0.0.0.0
API_PORT=8000
API_WORKERS=4

# Scraping-Limits
MAX_CONCURRENT_REQUESTS=10
REQUEST_DELAY=1.0
USER_AGENT="DataRaptor/1.0 (+https://example.com/bot)"

# NLP-Services
OPENAI_API_KEY=your_key_here
DEEPL_API_KEY=your_key_here

# Logging
LOG_LEVEL=INFO
LOG_PATH=/mnt/external/news_logs/
```

### Feed-Konfiguration (sources.txt)

```ini
[BBC]
url = http://feeds.bbci.co.uk/news/rss.xml
country = GB
language = en
category = news
interval = 3600

[CNN]
url = http://rss.cnn.com/rss/edition.rss
country = US
language = en
category = news
interval = 1800
```

### Docker-Konfiguration

```yaml
# docker-compose.yml
version: "3.8"
services:
  mongodb:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
      - ./config/mongod.conf:/etc/mongod.conf

  dataraptor:
    build: .
    depends_on:
      - mongodb
    volumes:
      - ./src:/app/src
      - /mnt/external/news_logs:/app/logs
      - /mnt/external/news_data:/app/data
    environment:
      - MONGODB_URI=mongodb://mongodb:27017/dataraptor
```

## 📈 Monitoring & Logs

### Log-Struktur

```
/mnt/external/news_logs/
├── application.log          # Haupt-Anwendungslog
├── feeds/
│   ├── rss_feeds.log       # Feed-Abruf Logs
│   └── scraping.log        # Scraping-Aktivitäten
├── analysis/
│   ├── sentiment.log       # NLP-Analyse Logs
│   └── translation.log     # Übersetzungs-Logs
└── errors/
    ├── failed_requests.log # Fehlgeschlagene Requests
    └── data_quality.log    # Datenqualitätsprobleme
```

### Health-Check Endpoints

```python
@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow(),
        "version": "1.0.0",
        "database": await check_mongodb_connection(),
        "feeds_active": await count_active_feeds(),
        "articles_today": await count_todays_articles()
    }

@app.get("/metrics")
async def metrics():
    return {
        "articles_total": await articles_collection.count_documents({}),
        "feeds_total": await feeds_collection.count_documents({}),
        "processing_rate": await calculate_processing_rate(),
        "error_rate": await calculate_error_rate(),
        "storage_usage": await get_storage_usage()
    }
```

### Performance Dashboard

```python
# Streamlit Dashboard
import streamlit as st
import plotly.express as px

def show_dashboard():
    st.title("📊 DataRaptor Analytics")

    # KPI-Metriken
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("Artikel heute", articles_today, delta=articles_delta)
    with col2:
        st.metric("Aktive Feeds", active_feeds, delta=feeds_delta)
    with col3:
        st.metric("Durchschnittliches Sentiment", avg_sentiment, delta=sentiment_delta)
    with col4:
        st.metric("Verarbeitungsrate", processing_rate, delta=rate_delta)

    # Trend-Diagramme
    fig = px.line(sentiment_trends, x='date', y='sentiment', color='country')
    st.plotly_chart(fig)
```

## 🧪 Tests

### Test-Suite Struktur

```
tests/
├── test_01_unit.py           # Unit Tests
├── test_02_integration.py    # Integration Tests
├── test_03_system.py         # System Tests
├── test_04_fuzz.py          # Fuzz Testing
├── test_05_security.py      # Security Tests
├── test_07_performance.py   # Performance Tests
├── test_08_Affe.py          # Chaos Engineering
├── test_09_wiederherstellbarkeit.py  # Recovery Tests
└── test_10_Umwelt.py        # Environment Tests
```

### Tests ausführen

```bash
# Alle Tests
pytest tests/ -v

# Spezifische Test-Kategorien
pytest tests/test_01_unit.py -v
pytest tests/test_02_integration.py -v

# Performance Tests
pytest tests/test_07_performance.py --benchmark-only

# Coverage Report
pytest tests/ --cov=src/dataraptor --cov-report=html
```

### Test-Konfiguration

```python
# conftest.py
@pytest.fixture
def test_mongodb():
    """Test-MongoDB-Instanz"""
    client = MongoClient("mongodb://localhost:27017")
    db = client.test_dataraptor
    yield db
    client.drop_database("test_dataraptor")

@pytest.fixture
def sample_articles():
    """Sample-Artikel für Tests"""
    return [
        {
            "title": "Test Article",
            "content": "Test content...",
            "link": "https://test.com/article",
            "publication_date": datetime.utcnow()
        }
    ]
```

## 🛡️ Sicherheit

### Anti-Bot Maßnahmen

**Verbindliche Scraping-Konventionen**:

```python
from selenium.webdriver.chrome.options import Options

def setup_browser():
    """Anti-Bot Browser-Konfiguration"""
    chrome_options = Options()
    chrome_options.add_argument("--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0.0.0")
    chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
    chrome_options.add_experimental_option('useAutomationExtension', False)
    chrome_options.add_argument("--window-size=1200,900")
    # Kein Headless-Modus für bessere Tarnung

    driver = webdriver.Chrome(options=chrome_options)

    # Realistische Delays
    time.sleep(random.uniform(2.0, 5.0))

    return driver
```

### Datenschutz & DSGVO

**Noch zu implementieren**:

- [ ] Datenschutzerklärung
- [ ] Betroffenenrechte-Prozesse
- [ ] Verzeichnis der Verarbeitungstätigkeiten
- [ ] PII-Erkennung und -Schutz
- [ ] Datenaufbewahrungslimits

### MongoDB-Sicherheit

```yaml
# mongod.conf (TO-DO)
security:
  authorization: enabled
  ssl:
    mode: requireSSL
    CAFile: /path/to/ca.pem

net:
  bindIp: 127.0.0.1

storage:
  wiredTiger:
    engineConfig:
      journalCompressor: snappy
```

## 📦 Deployment

### Produktions-Setup

```bash
# Build für Produktion
docker build -t dataraptor:latest .

# Mit Docker Compose
docker-compose -f docker-compose.prod.yml up -d

# Umgebung prüfen
docker-compose ps
docker-compose logs -f dataraptor
```

### Kapazitätsplanung

**Geschätzte Ressourcen bei 1000 Feeds**:

- ~300.000 Artikel/Monat
- ~3.6 Millionen Artikel/Jahr
- Speicher: ~500GB/Jahr (mit Analysen)
- RAM: 16GB (empfohlen)
- CPU: 4 Cores (minimum)

## 🚨 Troubleshooting

### Häufige Probleme

**MongoDB-Verbindung**:

```bash
# Verbindung testen
mongo --eval "db.adminCommand('ismaster')"

# Logs prüfen
docker-compose logs mongodb
```

**Feed-Abruf Fehler**:

```bash
# Scraping-Logs prüfen
tail -f /mnt/external/news_logs/feeds/rss_feeds.log

# Test-Run einzelner Feed
python -c "from src.dataraptor.mod_000_* import test_feed; test_feed('http://example.com/rss')"
```

**Performance-Probleme**:

```javascript
// Langsame Queries identifizieren
db.articles.find({}).explain("executionStats");

// Index-Verwendung prüfen
db.articles.getIndexes();
```

## 🤝 Contributing

### Development Setup

```bash
# Development Environment
git clone https://github.com/Watchkido/DataRaptor.git
cd DataRaptor

# Virtual Environment
python -m venv venv
source venv/bin/activate

# Development Dependencies
pip install -r requirements-dev.txt

# Pre-commit Hooks
pre-commit install
```

### Code-Standards

```python
# Formatierung mit Black
black src/ tests/

# Linting mit flake8
flake8 src/ tests/

# Type Checking mit mypy
mypy src/
```

### Pull Request Prozess

1. Fork des Repositories
2. Feature-Branch erstellen (`git checkout -b feature/amazing-feature`)
3. Änderungen committen (`git commit -m 'Add amazing feature'`)
4. Tests ausführen (`pytest tests/`)
5. Push zum Branch (`git push origin feature/amazing-feature`)
6. Pull Request erstellen

## 📚 Dokumentation

### API-Dokumentation

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

### Zusätzliche Docs

- [Architektur-Überblick](docs/Architektur.md)
- [Data Lakehouse Konzept](docs/Data_Lakehouse.md)
- [Smart Translator Guide](docs/doc_090_smart_translator.md)
- [Docker Installation](docs/aaa_Docker_installation_libretranslate.md)

## 📈 Roadmap & Nächste Schritte

### Version 2.0 (Q2 2024)

- [ ] **Machine Learning**: Topic Modeling mit BERT
- [ ] **Real-time Processing**: Kafka-Integration
- [ ] **Multi-Language**: Erweiterte Sprachunterstützung
- [ ] **Predictive Analytics**: Trend-Vorhersagen

### Version 2.1 (Q3 2024)

- [ ] **Auto-Scaling**: Kubernetes-Deployment
- [ ] **Advanced NLP**: Named Entity Recognition
- [ ] **Data Lineage**: Vollständige Datenherkunft
- [ ] **Compliance**: DSGVO-Vollständigkeit

### Langfristig

- [ ] **Graph Database**: Neo4j für Beziehungsanalysen
- [ ] **Stream Processing**: Apache Spark Integration
- [ ] **ML Ops**: Automated Model Training
- [ ] **Federation**: Multi-Tenant Architecture

## 📞 Support & Kontakt

### Community

- **GitHub Issues**: [Bug Reports & Feature Requests](https://github.com/Watchkido/DataRaptor/issues)
- **Discussions**: [Fragen & Feedback](https://github.com/Watchkido/DataRaptor/discussions)

### Kommerzieller Support

Für Enterprise-Support und Custom-Entwicklung:

- **E-Mail**: support.dataraptor@watchkido.de
- **Website**: https://[dataraptor](https://stadtwettkampf.de/)

### Entwickler-Kontakt

- **GitHub**: [@Watchkido](https://github.com/Watchkido)
- **LinkedIn**: [DataRaptor Project](https://linkedin.com/in/dataraptor)

---

## 📄 Lizenz

Dieses Projekt ist unter der [MIT License](LICENSE) lizenziert - siehe die LICENSE-Datei für Details.

## 🙏 Danksagungen

- **MongoDB Team** für die excellente Dokumentation
- **BeautifulSoup/Scrapy Community** für Web-Scraping Tools
- **Streamlit Team** für das großartige Dashboard-Framework
- **Open Source Contributors** die dieses Projekt möglich machen

---

<div align="center">

**DataRaptor** - _Powered by Data, Driven by Insights_ 🦾

[![GitHub stars](https://img.shields.io/github/stars/Watchkido/DataRaptor.svg?style=social&label=Star)](https://github.com/Watchkido/DataRaptor)
[![GitHub forks](https://img.shields.io/github/forks/Watchkido/DataRaptor.svg?style=social&label=Fork)](https://github.com/Watchkido/DataRaptor/fork)
[![GitHub watchers](https://img.shields.io/github/watchers/Watchkido/DataRaptor.svg?style=social&label=Watch)](https://github.com/Watchkido/DataRaptor)

</div>
