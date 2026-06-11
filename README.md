# Retail-Sales-Analytics-End-to-End-BI-Projekt


![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Oracle](https://img.shields.io/badge/Oracle-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)

## Projektübersicht

Dieses Projekt zeigt eine vollständige Business Intelligence Pipeline – von rohen CSV-Daten bis zum fertigen Power BI Dashboard. Als Datenbasis dient ein echter Walmart Retail Datensatz von Kaggle mit über 420.000 Transaktionen aus 45 Filialen über 3 Jahre (2010–2012).

## Architektur

```
Kaggle Dataset (CSV)
        │
        ▼
┌───────────────────┐
│  Staging Tabellen │  ← Rohdaten 1:1 importiert
│  STG_STORES       │
│  STG_SALES        │
│  STG_FEATURES     │
└────────┬──────────┘
         │ ETL (SQL)
         ▼
┌─────────────────────────────────────┐
│         Oracle Data Warehouse       │
│                                     │
│         ┌─────────────┐             │
│    ┌────►  FACT_SALES ◄────┐        │
│    │    └──────┬────▲──┘   │        │
│    │           │    │      │        │
│ DIM_STORE  DIM_DATE │ DIM_DEPT      │
│                     │               │
│    DIM_FEATURES─────┘               │
└─────────────────────────────────────┘
         │ SQL Views / KPIs
         ▼
┌───────────────────┐
│   Power BI        │
│   4 Seiten        │
│   DAX Measures    │
└───────────────────┘
```

## Datensatz

**Quelle:** [Walmart Retail Dataset – Kaggle](https://www.kaggle.com/)

| Tabelle | Beschreibung | Zeilen |
|---------|-------------|--------|
| Stores | 45 Filialen mit Typ (A/B/C) und Größe | 45 |
| Sales | Wöchentlicher Umsatz pro Filiale und Abteilung | 421.570 |
| Features | Externe Faktoren: Temperatur, Benzinpreis, Markdowns, CPI, Arbeitslosigkeit | 8.190 |

<img width="1522" height="732" alt="image" src="https://github.com/user-attachments/assets/3f294dc3-573e-4631-aeaa-f487ff17526e" />


## Star Schema

Das Datenmodell folgt dem **Star Schema** – dem Standard für Data Warehouses.

| Tabelle | Typ | Beschreibung |
|---------|-----|-------------|
| `FACT_SALES` | Fact | Zentrale Tabelle mit Umsätzen und Holiday-Gewichtung |
| `DIM_STORE` | Dimension | 45 Filialen mit Name, Typ, Größe |
| `DIM_DEPARTMENT` | Dimension | 81 Abteilungen mit Namen |
| `DIM_DATE` | Dimension | 143 Wochen mit Jahr, Monat, Quartal, Holiday-Name |
| `DIM_FEATURES` | Dimension | Externe Faktoren inkl. Markdown 1–5 |

<img width="1533" height="787" alt="image" src="https://github.com/user-attachments/assets/ee81fae1-ce95-464e-bb91-d4f0bb7cc98a" />


**Warum Star Schema?**
- Bessere Performance bei großen Datenmengen
- Keine Redundanz – Filialname wird nur einmal gespeichert
- Standard in professionellen Data Warehouses
- Optimal für Power BI Beziehungen

## ETL Prozess

Der ETL-Prozess wurde komplett in Oracle SQL durchgeführt:

| Problem | Lösung |
|---------|--------|
| Datumsformat `DD/MM/YYYY` | `TO_DATE(date, 'DD/MM/YYYY')` |
| `NA` Werte aus Excel | `CASE WHEN x = 'NA' THEN NULL` |
| `TRUE/FALSE` als Text | `CASE WHEN x = 'TRUE' THEN 1 ELSE 0` |
| Dezimalzeichen-Konflikt | `TO_NUMBER(x, '999999.99', 'NLS_NUMERIC_CHARACTERS=''.,''')` |
| Holiday-Gewichtung | `CASE IsHoliday WHEN 1 THEN 5.0 ELSE 1.0 END` |

## Power BI Dashboard

**4 Seiten:**

### Seite 1 – Executive Summary
- KPI Cards: Gesamt-Umsatz, bester Monat, Holiday-Anteil, YoY-Wachstum
- Liniendiagramm: Umsatz 2010 vs 2011 vs 2012
- Balkendiagramm: Umsatz nach Quartal
<img width="1077" height="573" alt="image" src="https://github.com/user-attachments/assets/271abf6b-4ec5-4960-abf1-30cae54f7da2" />


### Seite 2 – Store Analyse
- Top 10 Filialen nach Umsatz
- Vergleich Filialtyp A vs B vs C (Treemap)
- Holiday vs Regular Sales pro Filiale (gestapelter Balken)
<img width="1018" height="572" alt="image" src="https://github.com/user-attachments/assets/a79a779c-b54a-410f-85a8-2bf02300d67d" />


### Seite 3 – Department Performance
- Top 15 Abteilungen (horizontaler Balken)
- Heatmap: Store vs Abteilung
<img width="1021" height="572" alt="image" src="https://github.com/user-attachments/assets/17fe1309-d9e6-4a0f-8a6b-4580a3bf932f" />


### Seite 4 – Faktoren Impact
- Scatterplot: Markdown-Betrag vs Umsatz
- Vergleich: Super Bowl vs Thanksgiving vs Labor Day vs Christmas
<img width="1018" height="573" alt="image" src="https://github.com/user-attachments/assets/37824f0a-e189-4b3d-a805-0ed51dcdf661" />


**DAX Measures:**
- `Gesamt Umsatz`, `Holiday Umsatz`, `Regular Umsatz`
- `Gewichteter Umsatz` (mit Holiday_Weight 5x)
- `vs last Year %`, `vs last Month %`
- `Store Ranking`, `Abteilung Ranking`
- `Holiday Lift %`, `Total Markdown`
<img width="742" height="613" alt="image" src="https://github.com/user-attachments/assets/f2ff2617-396c-40f1-a315-fd8f908ddfd9" />

## Die Entscheidung
### 🏷️ Markdown = Preisvorteil

- **Markdown** → Rabatt  
- **Rabatt** → mehr Verkäufe  
- **Mehr Verkäufe** → mehr Umsatz  
- **Mehr Umsatz** → mehr Gewinn✅

---
💡 Markdown = unsere Aktionen & Preisnachlässe.

## Verwendete Technologien

| Tool | Verwendung |
|------|-----------|
| Oracle Database 26ai | Data Warehouse, ETL, SQL Views |
| Oracle SQL Developer | Datenbankentwicklung und Import |
| Power BI Desktop | Dashboard und DAX |
| Kaggle | Öffentlicher Datensatz |


## Lessons Learned

- **ETL ist nie einfach** – Datumsformate, Dezimalzeichen und NULL-Werte sind typische Praxis-Probleme
- **Oracle NLS-Einstellungen** beeinflussen Zahlenkonvertierungen
- **Star Schema** macht Power BI Beziehungen einfacher und schneller
- **Holiday-Gewichtung** zeigt den echten Business Impact von Feiertagen

## Autor

Moussa – Data & BI Portfolio Projekt
