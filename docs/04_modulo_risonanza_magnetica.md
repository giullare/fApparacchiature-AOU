# 04 — Modulo 2: Gestione Apparecchiature Risonanza Magnetica
### Riferimento normativo: DM 01/21 e s.m.i.

---

## 4.1 Panoramica del modulo

Il Modulo 2 gestisce specificamente le **apparecchiature di Risonanza Magnetica (RM)**, soggette a normativa dedicata (DM 01/21) con requisiti di sicurezza, documentazione e controllo diversi rispetto alle apparecchiature ionizzanti.

> **Differenza chiave rispetto al Modulo 1:** Le apparecchiature RM non producono radiazioni ionizzanti, ma sono soggette a rischi specifici legati all'intensità del campo magnetico. La normativa di riferimento è il DM 01/21 e non il D.Lgs. 101/20.

---

## 4.2 Struttura del modulo

```
Modulo 2 — Risonanza Magnetica
├── 📋 Anagrafica Apparecchiature RM
│   ├── [Lista RM attive]
│   ├── [Nuova apparecchiatura RM]
│   └── [Scheda singola RM]
├── 📁 Documentazione Normativa
│   ├── CAI (Comunicazione di Avvio Installazione)
│   ├── Benestare
│   └── Controlli Periodici
├── 🏛️ Norme Interne
│   └── Regolamento interno
├── 📐 Norme Tecniche di Riferimento
├── 🚨 Segnaletica (varia per RM)
├── 📊 Statistiche
│   ├── Statistiche Generali
│   └── Statistiche per Apparecchiatura
├── 🗄️ RM Cessate
└── 🔧 Apparecchiature RM da Installare
```

---

## 4.3 Scheda Apparecchiatura RM — Campi

### 4.3.1 Dati Anagrafici

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `codice` | String | ✓ | Codice univoco interno |
| `descrizione` | String | ✓ | Nome descrittivo |
| `modello` | String | ✓ | — |
| `costruttore` | String | ✓ | — |
| `matricola` | String | ✓ | — |
| `serial_number` | String | — | — |
| `foto` | File[] | — | Immagini dell'apparecchiatura e dell'ambiente |
| `ubicazione` | FK → Collocazione | ✓ | Struttura multilivello (come Modulo 1) |
| `reparto` | String | ✓ | — |
| `caposala` | String | — | Riferimento caposala del reparto |

### 4.3.2 Caratteristiche del Campo Magnetico

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `intensita_campo_tesla` | Decimal | ✓ | Intensità di campo magnetico principale in Tesla (es. 1.5T, 3T, 7T) |
| `tipo_magnete` | Enum | — | `SUPERCONDUTTORE` \| `PERMANENTE` \| `RESISTIVO` |
| `campo_max_fringe` | Decimal | — | Campo massimo nelle zone perimetrali (usato per calcolo zone) |

### 4.3.3 Zone di Sicurezza

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `piantina_zone` | File | ✓ | Planimetria con zona di rispetto e zona controllata magnetica |
| `descrizione_zona_rispetto` | Text | ✓ | Delimitazione e caratteristiche |
| `descrizione_zona_controllata` | Text | ✓ | — |

> **Nota:** La piantina deve evidenziare le linee di isocampo (tipicamente 0.5 mT per la zona di rispetto) secondo quanto richiesto dal DM 01/21.

### 4.3.4 Figure di Responsabilità e Sicurezza

| Ruolo | Sigla | Campi |
|-------|-------|-------|
| Esperto in Sicurezza RM | ES | nome, cognome, qualifica, email, telefono |
| Medico Responsabile RM | MR | nome, cognome, specializzazione, email, telefono |

### 4.3.5 Riferimenti Assistenza

| Campo | Tipo | Note |
|-------|------|-------|
| Numero assistenza tecnica | String | — |
| Global service | String | — |
| Email assistenza | Email | — |

### 4.3.6 Stato e Ciclo di Vita

| Campo | Tipo | Note |
|-------|------|-------|
| `stato` | Enum | `ATTIVA` \| `CESSATA` \| `IN_INSTALLAZIONE` \| `IN_MANUTENZIONE` |
| `data_attivazione` | Date | Data di messa in servizio dopo benestare |

---

## 4.4 Documentazione Normativa (DM 01/21)

### 4.4.1 CAI — Comunicazione di Avvio Installazione

| Campo | Tipo | Note |
|-------|------|-------|
| `cai_data_invio` | Date | — |
| `cai_protocollo` | String | Numero di protocollo |
| `cai_documento` | File | Upload del documento |
| `cai_ricevuta` | File | Ricevuta/conferma ricezione |
| `cai_stato` | Enum | `DA_INVIARE` \| `INVIATA` \| `APPROVATA` |

### 4.4.2 Benestare

| Campo | Tipo | Note |
|-------|------|-------|
| `benestare_data` | Date | Data rilascio benestare |
| `benestare_documento` | File | Upload documento benestare |
| `benestare_ente_rilascio` | String | Ente che ha rilasciato il benestare |
| `benestare_stato` | Enum | `IN_ATTESA` \| `OTTENUTO` \| `NON_APPLICABILE` |

### 4.4.3 Controlli Periodici

I controlli periodici RM sono **semestrali** e includono verifiche specifiche:

| Tipo di Controllo | Periodicità | Esito |
|-------------------|-------------|-------|
| Ventilazione | Semestrale | Pass/Fail + valore misurato |
| Livello di ossigeno | Semestrale | Pass/Fail + valore misurato (% O₂) |
| Tenuta gabbia di Faraday | Semestrale | Pass/Fail + dB misurati |
| Qualità immagine (phantom test) | Semestrale + su evento | Pass/Fail + valori parametri |
| Homogeneità del campo | Semestrale | Pass/Fail + ppm misurati |
| Sicurezza elettrica | Semestrale | Pass/Fail |

**Struttura record Controllo Periodico:**

| Campo | Tipo | Note |
|-------|------|-------|
| `data_verifica` | Date | — |
| `esecutore` | FK → Utente | Chi ha eseguito la verifica |
| `tipologie_controllo` | Enum[] | Checklist dei controlli eseguiti |
| `esito_generale` | Enum | `SUPERATO` \| `NON_SUPERATO` \| `CON_RISERVA` |
| `note` | Text | — |
| `report_allegato` | File | Report dettagliato |
| `info_guasto` | Text | Eventuali guasti riscontrati |
| `semestre` | String | Es. "1S-2024", "2S-2024" |

### 4.4.4 Verbali Periodici

| Campo | Tipo | Note |
|-------|------|-------|
| `data_sopralluogo` | Date | — |
| `redattore` | FK → Utente | — |
| `partecipanti` | String | — |
| `oggetto` | Text | — |
| `contenuto` | Text | — |
| `documento` | File | Verbale firmato in PDF |

### 4.4.5 Inventario Strumentazione e Calibrazioni

Gestione dello strumentario di radioprotezione/sicurezza RM:

| Campo | Tipo | Note |
|-------|------|-------|
| `nome_strumento` | String | — |
| `modello` | String | — |
| `matricola` | String | — |
| `data_ultima_calibrazione` | Date | — |
| `data_prossima_calibrazione` | Date | Alert automatico |
| `certificato_calibrazione` | File | — |
| `stato_calibrazione` | Enum | `IN_REGOLA` \| `SCADUTA` \| `IN_SCADENZA` |

---

## 4.5 Statistiche RM

### Statistiche Generali

| Indicatore | Descrizione |
|------------|-------------|
| Conteggio totale RM | Per intensità di campo, per reparto |
| Apparecchiature obsolete | Criterio configurabile |
| CQ effettuati / anno e esiti | Trend semestrali |

### Statistiche per Apparecchiatura

| Indicatore | Descrizione |
|------------|-------------|
| Esito CQ per anno/semestre | Grafico temporale |
| Info numero e tipo di guasto | Categorizzazione e trend |

---

## 4.6 Segnaletica

La sezione segnaletica RM gestisce documentazione e template per:
- Segnali di zona di rispetto magnetica
- Segnali di zona controllata magnetica
- Avvisi per portatori di pacemaker/impianti metallici
- Procedure di accesso (screening pazienti e personale)

Ogni template è scaricabile e associabile all'apparecchiatura.

---

## 4.7 Notifiche e Alert RM

| Evento | Destinatari | Anticipo |
|--------|-------------|---------|
| Scadenza controllo periodico semestrale | ES, MR | 30 giorni, 7 giorni |
| Controllo scaduto | ES, MR | Immediato |
| Scadenza calibrazione strumento | ES | 30 giorni |
| Benestare in scadenza (se temporaneo) | ES | 60 giorni |