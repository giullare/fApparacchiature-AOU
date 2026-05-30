# 03 — Modulo 1: Gestione Grandi Apparecchiature Radiologiche
### Riferimento normativo: D.Lgs. 101/20 — Titoli XI e XIII

---

## 3.1 Panoramica del modulo

Il Modulo 1 gestisce il ciclo di vita completo di tutte le **apparecchiature che producono radiazioni ionizzanti** nelle strutture sanitarie, coprendo sia gli adempimenti di Fisica Medica (esposizioni mediche — pazienti) sia quelli di Radioprotezione (ambienti e lavoratori).

### Tipologie di apparecchiature gestite

| Ambito | Tipologie |
|--------|-----------|
| **Radiologia diagnostica** | Mammografi, Endorali, MOC, Portatili, TAC, Pensili, Telecomandati, Ortopantomografi |
| **Radiologia interventistica** | Angiografi, Archi a C, CBCT, Litotritori, TAC interventistici |
| **Medicina Nucleare** | PET-CT, SPECT, SPECT-CT, D-SPECT, Gammacamera piccolo campo, Gammacamera grande campo |
| **Radioterapia** | Acceleratori lineari, Tomoterapia, CyberKnife, GammaPod, Yort |

---

## 3.2 Sezioni principali dell'interfaccia

```
Modulo 1 — Apparecchiature Radiologiche
├── 📋 Anagrafica Apparecchiature
│   ├── [Lista apparecchiature attive]
│   ├── [Nuova apparecchiatura]
│   └── [Scheda singola apparecchiatura]
├── 📁 Documentazione Correlata
│   ├── Esposizione Medica (D.Lgs. 101/20 Titolo XIII)
│   └── Esposizione dei Lavoratori (D.Lgs. 101/20 Titolo XI)
├── 🏛️ Norme Interne
│   ├── Radiologia
│   ├── Interventistica
│   ├── Radioterapia
│   └── Medicina Nucleare
├── 🚨 Segnaletica
│   ├── Zona Controllata
│   ├── Zona Sorvegliata
│   └── Altra segnaletica
├── 📊 Statistiche
│   ├── Statistiche Generali
│   └── Statistiche per Apparecchiatura
├── 🗄️ Apparecchiature Cessate
└── 🔧 Apparecchiature da Installare
```

---

## 3.3 Scheda Apparecchiatura — Campi e sezioni

### 3.3.1 Dati Anagrafici

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `codice` | String | ✓ | Codice univoco interno, autogenerato o manuale |
| `descrizione` | String | ✓ | Nome descrittivo dell'apparecchiatura |
| `ambito_intervento` | Enum | ✓ | `RADIOLOGIA` \| `RADIOLOGIA_INTERVENTISTICA` \| `MEDICINA_NUCLEARE` \| `RADIOTERAPIA` |
| `tipologia` | Enum (dipende da ambito) | ✓ | Vedi lista tipologie per ambito in §3.1 |
| `modello` | String | ✓ | — |
| `costruttore` | String | ✓ | — |
| `matricola` | String | ✓ | — |
| `serial_number` | String | — | — |
| `foto` | File[] | — | Immagini fotografiche dell'apparecchiatura |

### 3.3.2 Parametri Radiologici

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `corrente_max_ma` | Decimal | ✓ | Corrente massima in mA |
| `tensione_max_kvolt` | Decimal | ✓ | Tensione massima in KVolt |
| `energia_max_kev` | Decimal | — | Energia massima in KEV — determina alcuni adempimenti specifici |

> **Nota implementativa:** il campo `energia_max_kev` deve triggerare logica condizionale: sopra certi valori soglia si attivano adempimenti aggiuntivi da specificare con il team di fisica medica.

### 3.3.3 Collocazione Fisica

Struttura ad **albero multilivello**:

```
Livello 1:  Sito / Presidio    (es. "Complesso di Via Pansini")
Livello 2:  Immobile           (es. "Edificio 20")
Livello 3:  Piano              (es. "Piano 2")
Livello 4:  Locale             (es. "Sala TAC - Stanza 204")
```

**UI:** selector a cascata (selezione Livello 1 popola opzioni Livello 2, ecc.)

Campi aggiuntivi di collocazione:
- `planimetria` — allegato (immagine/PDF) con zona classificate, schermature, luci segnaletiche
- `zone_classificate` — text area descrittiva

### 3.3.4 Riferimenti Normativi

| Campo | Tipo | Note |
|-------|------|-------|
| `norma_riferimento` | FK → Norme | Collegamento alla sezione norme del sistema |
| `reparto_assegnazione` | FK → Reparti | — |

### 3.3.5 Responsabilità dell'Apparecchiatura

**Figure per Fisica Medica:**

| Ruolo | Sigla | Campi |
|-------|-------|-------|
| Responsabile Impianto Radiologico | RIR | nome, cognome, email, telefono |
| Esperto in Fisica Medica | EFM | nome, cognome, email, telefono |

**Figure per Radioprotezione:**

| Ruolo | Sigla | Campi |
|-------|------|-------|
| Esperto di Radioprotezione | EdR | nome, cognome, grado abilitazione, email, telefono |
| Medico Autorizzato | MA | nome, cognome, email, telefono |

### 3.3.6 Riferimenti Gestionali

| Campo | Tipo | Note |
|-------|------|-------|
| Società di manutenzione | String | Nome azienda |
| Tecnico di riferimento | String | Nome e cognome |
| Numero assistenza tecnica | String | — |
| Global service | String | — |
| Email assistenza | Email | — |
| Riferimento reparto / caposala | String | — |

### 3.3.7 Adempimenti Registrativi

**Adempimento INAIL:**

| Campo | Tipo | Note |
|-------|------|-------|
| `inail_stato` | Enum | `DA_REGISTRARE` \| `REGISTRATO` \| `NON_APPLICABILE` |
| `inail_data_registrazione` | Date | — |
| `inail_ricevuta` | File | Upload ricevuta registrazione |

**Adempimento STRIMS:**

| Campo | Tipo | Note |
|-------|------|-------|
| `strims_stato` | Enum | `DA_REGISTRARE` \| `REGISTRATO` \| `NON_APPLICABILE` |
| `strims_data_registrazione` | Date | — |
| `strims_ricevuta` | File | Upload ricevuta registrazione |
| `strims_notifica_pratica_inviata` | Boolean | — |
| `strims_notifica_cessazione_inviata` | Boolean | — |

### 3.3.8 Parametri Tecnici

| Campo | Tipo | Note |
|-------|------|-------|
| `lan_collegata` | Boolean | L'apparecchiatura è collegata in rete LAN |
| `medsquare_installato` | Boolean | Installato software MedSquare per determinazione LDR |
| `data_accettazione` | Date | Data del collaudo di accettazione con esito positivo |
| `stato` | Enum | `ATTIVA` \| `CESSATA` \| `IN_INSTALLAZIONE` \| `IN_MANUTENZIONE` |

### 3.3.9 Dati Identificativi Bene Aziendale

| Campo | Tipo | Note |
|-------|------|-------|
| `sap_id` | String | Identificativo SAP (opzionale) |
| `siap_descrizione` | String | Descrizione SIAP (opzionale) |
| `allegato_cespite` | File | Documento amministrativo cespite |

---

## 3.4 Documentazione Correlata — Esposizione Medica (Titolo XIII)

Per ogni apparecchiatura devono essere gestiti i seguenti documenti:

| # | Documento | Tipo | Periodicità | Note |
|---|-----------|------|-------------|------|
| 1 | Manuale di Qualità | Documento strutturato | — | Auto-compilato dinamicamente (vedi §3.5) |
| 2 | Accettazione (Collaudo) | Record + allegati | Una tantum | Alla messa in servizio |
| 3 | Controlli Periodici — Report ed esiti | Record + allegati | Annuale / secondo protocollo | Include info guasti |
| 4 | Inventario della strumentazione e stato calibrazioni | Lista + allegati | Aggiornamento continuo | — |
| 4 | LDR periodici | Documento PDF | Annuale / Biennale | — |
| 5 | Storage documentazione | Archivio | — | Tutti i documenti allegati |

---

## 3.5 Manuale di Qualità dell'Apparecchiatura

Il Manuale è generato **dinamicamente** dalla piattaforma, seguendo l'Allegato XXVII del D.Lgs. 101/20.

### Struttura

```
Manuale di Qualità — [Nome Apparecchiatura]
├── PARTE STATICA (compilata una volta)
│   ├── Dati anagrafici
│   ├── Caratteristiche tecniche
│   ├── Normativa di riferimento
│   ├── Figure di responsabilità
│   └── Planimetria e zone classificate
│
└── PARTE DINAMICA (aggiornata automaticamente)
    ├── Risultati prove di accettazione iniziali
    ├── Risultati prove verifiche periodiche e su evento
    └── Risultati verifiche LDR
```

**UI:** Il manuale deve essere visualizzabile in-app e stampabile/esportabile in PDF.

---

## 3.6 Documentazione Correlata — Esposizione Lavoratori (Titolo XI)

| # | Documento | Azione richiesta |
|---|-----------|-----------------|
| 1 | Notifica di Pratica (art. 46 D.Lgs. 101/20) | Upload + gestione protocollo PEC, data notifica |
| 2 | Nulla Osta (NO) — Tipo A o Tipo B | Upload + stato |
| 2 | Benestare all'utilizzo | Upload |
| 2 | Notifica di Cessazione (art. 48) o di NO | Upload + data |
| 3 | STRIM — registrazione e cessazione | Upload ricevuta + stato |
| 4 | INAIL — registrazione e cessazione | Upload ricevuta + stato |
| 5 | Verbali sopralluoghi periodici | Upload + data |

---

## 3.7 Statistiche

### Statistiche Generali (vista aggregata)

| Indicatore | Descrizione |
|------------|-------------|
| Conteggio totale apparecchiature | Per ambito, per tipologia, per reparto |
| Apparecchiature obsolete | Conteggio e lista (criterio di obsolescenza da definire) |
| CQ effettuati / anno e relativi esiti | Percentuale superamento, trend |
| Dose ambientale per tipologia | Aggregato per tipo di apparecchiatura |
| Apparecchiature in rete | Conteggio con LAN = true |
| Apparecchiature con SW monitoraggio dose | Conteggio con MedSquare = true |

### Statistiche per Singola Apparecchiatura

| Indicatore | Descrizione |
|------------|-------------|
| Esito CQ per anno | Grafico temporale pass/fail |
| Esito dosimetria ambientale per anno | Trend valori misurati |
| Info numero e tipo di guasto | Lista guasti con categorizzazione |

---

## 3.8 Ciclo di Vita dell'Apparecchiatura

```
[IN_INSTALLAZIONE] ──→ [ACCETTAZIONE] ──→ [ATTIVA]
                                             │
                                    ┌────────┴────────┐
                                    ▼                 ▼
                           [IN_MANUTENZIONE]    [CESSATA]
                                    │
                                    ▼
                                [ATTIVA]
```

**Sezione Apparecchiature Cessate:** archivio storico consultabile con tutta la documentazione.

**Sezione Apparecchiature da Installare:** planning delle future installazioni con documentazione preparatoria.

---

## 3.9 Notifiche e Alert

Il sistema deve inviare notifiche automatiche (email + notifica in-app) per:

| Evento | Destinatari | Anticipo |
|--------|-------------|---------|
| Scadenza CQ imminente | EFM, RIR | 30 giorni, 7 giorni |
| CQ scaduto | EFM, RIR | Immediato |
| Scadenza LDR | EFM | 60 giorni |
| Scadenza registrazione STRIMS / INAIL | EDR | 30 giorni |
| Guasto registrato | EFM, RIR | Immediato |
| Apparecchiatura in manutenzione > X giorni | EFM | Configurabile |