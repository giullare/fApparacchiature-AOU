# 11 — Statistiche, Cruscotti e Reportistica

---

## 11.1 Panoramica

Il sistema offre tre livelli di analisi:

1. **Dashboard operativa** — stato in tempo reale (compliance, scadenze, alert)
2. **Statistiche per modulo** — KPI specifici per ogni area funzionale
3. **Reportistica avanzata** — report periodici, export, analisi trend

---

## 11.2 Dashboard Principale (Home Page)

All'accesso, ogni utente vede una dashboard personalizzata per il proprio ruolo con:

### Widget principali

```
┌─────────────────────────────────────────────────────────────────┐
│  DASHBOARD — [Nome Organizzazione]         [Oggi: 30/05/2026]   │
├──────────────┬──────────────┬──────────────┬────────────────────┤
│  🔴 Alert    │  ⚠️  In      │  ✅ In       │  📋 Da fare        │
│   Critici    │  Scadenza    │  Regola      │   oggi             │
│     3        │     12       │    87        │     5              │
├──────────────┴──────────────┴──────────────┴────────────────────┤
│  SCADENZE IMMINENTI (prossimi 30 giorni)                        │
│  • TAC Sala 3 — CQ Annuale — Scade il 15/06/2026      [Apri]  │
│  • PET-CT — Verifica EDR — Scade il 22/06/2026        [Apri]  │
│  • Calibratore dose — Calibrazione — Scade 01/07/2026  [Apri]  │
├─────────────────────────────────────────────────────────────────┤
│  COMPLIANCE OVERVIEW                                            │
│  Modulo 1 Radiologia     ████████████░░  87%                    │
│  Modulo 2 RM             ████████████████ 100%                  │
│  Modulo 3 Medicina Nuc.  ████████░░░░░░░  62%                   │
│  Modulo 4 Lu177          ██████████████░  93%                   │
├─────────────────────────────────────────────────────────────────┤
│  ULTIME ATTIVITÀ                                                │
│  • [Mario Rossi] ha caricato Report CQ TAC Sala 3 — 2h fa      │
│  • [Laura Bianchi] ha aggiunto verbale sopralluogo — 1 giorno fa│
└─────────────────────────────────────────────────────────────────┘
```

### Personalizzazione per ruolo

| Ruolo | Widget prioritari |
|-------|-------------------|
| EFM | CQ in scadenza, LDR da fare, nuove apparecchiature |
| EDR | Sorveglianze in scadenza, NO in scadenza, dosimetria operatori |
| RIR | Benestare clinico da dare, compliance overview |
| ADMIN_ORG | Overview completa, gestione utenti, spazio storage |

---

## 11.3 Statistiche Modulo 1 — Apparecchiature Radiologiche

### 11.3.1 Statistiche Generali

**Vista tabellare con filtri:**
- Per ambito (Radiologia / Interventistica / MN / Radioterapia)
- Per stato (Attiva / Cessata / In installazione)
- Per reparto
- Per tipologia

| KPI | Calcolo |
|-----|---------|
| Totale apparecchiature attive | Count dove stato = ATTIVA |
| Apparecchiature obsolete | Definire criterio (es. > 15 anni dalla data accettazione) |
| % in rete (LAN) | Count(lan=true) / Count(totale) |
| % con MedSquare | Count(medsquare=true) / Count(totale) |
| CQ effettuati nell'anno | Count per anno corrente |
| % CQ con esito positivo | Count(esito=SUPERATO) / Count(CQ anno) |
| Dose ambientale media per tipo | Media valori dosimetrici per tipologia apparecchiatura |

**Grafici:**
- Torta: distribuzione per ambito
- Barre: CQ per mese nell'anno corrente
- Timeline: acquisizioni e cessazioni nel tempo

### 11.3.2 Statistiche per Singola Apparecchiatura

```
Apparecchiatura: TAC — Sala 3 — Edificio 20
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CONTROLLI DI QUALITÀ (ultimi 5 anni)
Anno    | CQ Eseguiti | Superati | Con Riserva | Non Superati
--------|-------------|----------|-------------|-------------
2022    |      4      |    4     |      0      |      0
2023    |      4      |    3     |      1      |      0
2024    |      4      |    4     |      0      |      0
2025    |      2      |    2     |      0      |      -

DOSIMETRIA AMBIENTALE (ultimi 3 anni)
[Line chart: valori misurati per punto di misura nel tempo]

GUASTI
Totale guasti registrati: 3
- Hardware: 2 (66%)
- Software: 1 (33%)
Tempo medio risoluzione: 4.5 giorni
```

---

## 11.4 Statistiche Modulo 2 — Risonanza Magnetica

| KPI | Calcolo |
|-----|---------|
| Totale RM attive | — |
| RM con intensità > 1.5T | Count con campo > 1.5 |
| CQ semestrali effettuati | Per semestre corrente |
| % CQ con esito positivo | — |
| Guasti registrati nell'anno | — |

**Grafico specifico:** Trend esiti CQ per semestre (pass/fail per macchina)

---

## 11.5 Statistiche Modulo 3 — Medicina Nucleare

### Per sezione

**Dosimetria Ambientale:**
| KPI | Descrizione |
|-----|-------------|
| Misure nel periodo | Count per punto di misura |
| Dosi medie per punto | Media valori misurati |
| Superamenti livello indagine | Count per punto e periodo |
| Superamenti limite | Count assoluto |

**Contaminazione:**
| KPI | Descrizione |
|-----|-------------|
| Test eseguiti | Per mese |
| % aree pulite | Entro limiti |
| Radionuclidi contaminanti | Distribuzione |

**Rifiuti:**
| KPI | Descrizione |
|-----|-------------|
| Produzione per radionuclide | MBq/mese |
| Smaltimenti effettuati | Per modalità |

**Ventilazione:**
| KPI | Descrizione |
|-----|-------------|
| Controlli effettuati | Count semestrale |
| Non conformità ventilazione | Count |

---

## 11.6 Statistiche Modulo 4 — Lu177

### Statistiche cliniche aggregate (anonimizzate)

```
RIEPILOGO PAZIENTI Lu177
━━━━━━━━━━━━━━━━━━━━━━━

Totale pazienti trattati (storico): 47
  ├── PSMA Lu177: 32 (68%)
  └── DOTATATE Lu177: 15 (32%)

Pazienti in trattamento attivo: 8
Pazienti conclusi: 34
Persi al follow-up: 5

CICLI DI TERAPIA
Cicli totali somministrati: 156
Media cicli per paziente: 3.3
Attività media per ciclo: 7.4 GBq (range: 5.5 - 8.0)

TOSSICITÀ EMATOLOGICA (CTCAE)
G0: 42%  G1: 31%  G2: 18%  G3: 8%  G4: 1%

DOSIMETRIA RENALE (cumulativa)
Media dose renale cumulativa: 18.4 Gy
Pazienti con dose > 23 Gy (soglia attenzione): 7 (15%)
```

**Grafici disponibili:**
- Box plot: distribuzione dosi renali
- Scatter: correlazione dose renale / variazione eGFR
- Line chart: trend PSA/cromogranina A nel follow-up
- Kaplan-Meier: sopravvivenza libera da progressione (se dati disponibili)

---

## 11.7 Report Generabili

Il sistema permette di generare report in formato **PDF** e/o **Excel** per:

| Report | Periodicità | Destinatari | Contenuto |
|--------|-------------|-------------|-----------|
| Report Annuale CQ | Annuale | SFM, RIR, Datore di Lavoro | Riepilogo tutti i CQ dell'anno, esiti, trend |
| Report Sorveglianza EDR | Annuale/Semestrale | EDR, Datore di Lavoro | Verbali, verifiche, dosimetria |
| Report Compliance | Mensile | ADMIN_ORG, RQ | Stato adempimenti per modulo |
| Report Scadenze | Settimanale | EFM, EDR | Scadenze prossime 30-60 giorni |
| Verbale Periodico MN | Semestrale | EDR | Aggregato dati Modulo 3 (auto-compilato) |
| Report Dosimetrico Operatori | Trimestrale | EDR | Dosi per lavoratore |
| Report Statistico Lu177 | Semestrale/Annuale | EFM | Dati clinici anonimizzati |
| Inventario Apparecchiature | Su richiesta | Tutti | Lista completa con stato compliance |

### Formato PDF generato

```
Header: Logo organizzazione + data generazione + periodo riferimento
Body:   Contenuto strutturato con tabelle e grafici inline
Footer: "Generato da Piattaforma Fisica Sanitaria e Radioprotezione 
        il [data] dall'utente [nome] — Riservato"
Firma:  Spazio per firma elettronica o autografa
```

---

## 11.8 Export Dati

| Formato | Casi d'uso |
|---------|-----------|
| **CSV** | Export grezza per analisi esterne (Excel, R, Python) |
| **Excel (.xlsx)** | Report strutturati multi-foglio |
| **PDF** | Report ufficiali, verbali, manuale di qualità |
| **JSON** | Integrazione con sistemi esterni / API |

### Regole export

- L'export include solo dati del tenant corrente
- I dati dei pazienti (Modulo 4) sono pseudonimizzati negli export statistici
- Ogni export è loggato nell'audit log
- L'export può essere filtrato per periodo, reparto, tipologia

---

## 11.9 Sistema di Notifiche e Alert

### Canali di notifica

| Canale | Configurazione |
|--------|----------------|
| **Email** | Obbligatorio (SMTP configurabile) |
| **Notifica in-app** | Campana nel header, contatore non lette |
| **Push notification** | Futuro (app mobile) |

### Configurazione alert per utente

Ogni utente può configurare:
- Quali eventi gli generano una notifica
- Con quanto anticipo (7 / 14 / 30 / 60 giorni prima della scadenza)
- Frequenza digest email (immediata / giornaliera / settimanale)

### Tipi di alert

| Priorità | Tipo | Esempio |
|----------|------|---------|
| 🔴 Critica | Scadenza superata | CQ non eseguito, NO scaduto |
| 🟠 Alta | Scadenza imminente < 7 gg | — |
| 🟡 Media | Scadenza imminente < 30 gg | — |
| 🔵 Info | Evento completato | CQ superato, nuovo verbale |
| ⚪ Sistema | Azione richiesta | Benestare clinico da dare |