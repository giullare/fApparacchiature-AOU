# 06 — Modulo 4: Gestione Pazienti in Radioterapia Metabolica con Lu177
### Riferimento normativo: D.Lgs. 101/20

---

## 6.1 Panoramica del modulo

Il Modulo 4 è un sistema specializzato per la gestione clinica, dosimetrica ed ematologica dei **pazienti sottoposti a radioterapia metabolica con Lutezio-177 (Lu177)**, un radiofarmaco terapeutico utilizzato prevalentemente nel trattamento del carcinoma prostatico avanzato (PSMA) e dei tumori neuroendocrini (DOTATATE).

Questo modulo integra:
- Anagrafica clinica del paziente
- Pianificazione e registrazione dei cicli di trattamento
- Dosimetria personalizzata per ciclo
- Monitoraggio ematologico nel tempo
- Gestione dei rifiuti correlati al trattamento
- Elaborazione statistica dei dati clinici

---

## 6.2 Struttura del modulo

```
Modulo 4 — Radioterapia Metabolica Lu177
├── 👥 Anagrafica Pazienti
├── 💉 Dati di Trattamento
│   ├── Cicli di terapia
│   └── Somministrazioni
├── 📡 Dati Dosimetrici ed Ematologici
│   ├── Dosimetria per ciclo
│   └── Follow-up ematologico
├── 📊 Elaborazione Statistica
│   └── Dati di trattamento aggregati
├── 🗑️ Rifiuti Correlati
├── 🏛️ Norme Interne
├── 🚨 Segnaletica
├── 📐 Norme Tecniche di Riferimento
└── 📊 Statistica Operativa
```

---

## 6.3 Anagrafica Pazienti

### Dati identificativi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `id_paziente` | UUID | ✓ | ID interno anonimizzato (per export statistici) |
| `codice_paziente` | String | ✓ | Codice alfanumerico interno (es. "LU2024-001") |
| `cognome` | String | ✓ | — |
| `nome` | String | ✓ | — |
| `data_nascita` | Date | ✓ | — |
| `sesso` | Enum | ✓ | `M` \| `F` |
| `codice_fiscale` | String | — | — |
| `numero_nosologico` | String | — | Numero cartella clinica ospedaliera |
| `medico_inviante` | String | ✓ | — |
| `reparto_inviante` | String | — | — |
| `diagnosi_principale` | Text | ✓ | — |
| `indicazione_trattamento` | Enum | ✓ | `PSMA_LU177` \| `DOTATATE_LU177` \| `ALTRO` |
| `data_prima_visita` | Date | — | — |
| `stato_paziente` | Enum | ✓ | `IN_TRATTAMENTO` \| `CONCLUSO` \| `DECEDUTO` \| `PERSO_FU` |

### Dati clinici preliminari

| Campo | Tipo | Note |
|-------|------|-------|
| `peso_kg` | Decimal | Aggiornato ad ogni ciclo |
| `altezza_cm` | Decimal | — |
| `superficie_corporea_m2` | Decimal | Calcolata automaticamente (BSA) |
| `funzionalita_renale_egfr` | Decimal | eGFR in mL/min/1.73m² |
| `funzionalita_epatica` | Text | Note cliniche |
| `scintigrafia_pre_terapia` | File | Immagine diagnostica di selezione |
| `controindicazioni` | Text | — |
| `consenso_informato` | File | PDF firmato |

---

## 6.4 Dati di Trattamento

### 6.4.1 Schema trattamento (pianificazione)

| Campo | Tipo | Note |
|-------|------|-------|
| `protocollo_terapeutico` | String | Es. "VISION", "NETTER-1", "Custom" |
| `numero_cicli_pianificati` | Integer | Tipicamente 4-6 |
| `attivita_per_ciclo_gbq` | Decimal | Attività pianificata per ogni somministrazione |
| `intervallo_settimane` | Integer | Tipicamente 6-8 settimane |
| `data_inizio_trattamento` | Date | — |

### 6.4.2 Record singolo ciclo di somministrazione

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `numero_ciclo` | Integer | ✓ | 1, 2, 3... |
| `data_somministrazione` | Date | ✓ | — |
| `ora_somministrazione` | Time | ✓ | Per calcoli dosimetrici |
| `attivita_somministrata_gbq` | Decimal | ✓ | Valore misurato al calibratore |
| `attivita_residua_siringa_gbq` | Decimal | ✓ | Misura post-somministrazione |
| `attivita_effettiva_gbq` | Decimal | ✓ | Calcolata: somministrata - residua |
| `lotto_radiofarmaco` | String | ✓ | — |
| `fornitore_radiofarmaco` | String | ✓ | — |
| `certificato_lotto` | File | ✓ | Certificato di qualità del lotto |
| `peso_paziente_ciclo_kg` | Decimal | ✓ | Peso al giorno della somministrazione |
| `medico_prescrivente` | FK → Utente | ✓ | — |
| `fisico_medico` | FK → Utente | ✓ | — |
| `infermiere_somministratore` | String | — | — |
| `note_cliniche` | Text | — | — |
| `effetti_avversi` | Text | — | Tossicità riscontrate |
| `esito_ciclo` | Enum | ✓ | `COMPLETATO` \| `RIDOTTO` \| `SOSPESO` \| `INTERROTTO` |
| `motivo_modifica` | Text | Cond. | Se esito ≠ COMPLETATO |

---

## 6.5 Dati Dosimetrici

La dosimetria personalizzata con Lu177 richiede acquisizioni scintigrafiche e calcolo della dose assorbita per organo.

### 6.5.1 Acquisizioni scintigrafiche post-somministrazione

Per ogni ciclo, serie di acquisizioni SPECT/CT a diversi time-point:

| Campo | Tipo | Note |
|-------|------|-------|
| `fk_ciclo` | FK → Ciclo | — |
| `time_point_ore` | Integer | Es. 2h, 24h, 48h, 96h post-somministrazione |
| `data_acquisizione` | DateTime | — |
| `tipo_acquisizione` | Enum | `PLANAR` \| `SPECT` \| `SPECT_CT` |
| `file_immagine` | File | Formato DICOM (riferimento/path) |
| `attivita_corpo_intero_mbq` | Decimal | — |

### 6.5.2 Dosimetria per organo a rischio (OAR)

| Campo | Tipo | Note |
|-------|------|-------|
| `fk_ciclo` | FK → Ciclo | — |
| `organo` | Enum | `RENI` \| `MIDOLLO_OSSEO` \| `MILZA` \| `FEGATO` \| `TUMORE` |
| `dose_assorbita_gy` | Decimal | Dose assorbita in Gray |
| `dose_cumulativa_gy` | Decimal | Somma cicli precedenti + corrente |
| `metodo_calcolo` | String | Metodo dosimetrico utilizzato |
| `software_utilizzato` | String | — |
| `report_dosimetrico` | File | — |

### 6.5.3 Parametri di decadimento e ritenzione

| Campo | Tipo | Note |
|-------|------|-------|
| `t_fisico_ore` | Decimal | Emivita fisica Lu177 = 161.5 ore |
| `t_biologico_reni_ore` | Decimal | Emivita biologica renale (misurata) |
| `t_effettivo_reni_ore` | Decimal | Calcolato automaticamente |
| `costante_decadimento` | Decimal | λ = ln(2)/t½ |

---

## 6.6 Dati Ematologici

Follow-up ematologico obbligatorio nei pazienti in trattamento con Lu177 (rischio mielotossicità):

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `fk_paziente` | FK → Paziente | ✓ | — |
| `data_prelievo` | Date | ✓ | — |
| `timing` | Enum | ✓ | `PRE_CICLO_1` \| `POST_CICLO_1` \| `PRE_CICLO_2` \| ... \| `FOLLOW_UP_3M` \| `FOLLOW_UP_6M` |
| `wbc_x10_9_l` | Decimal | — | Globuli bianchi |
| `hgb_g_dl` | Decimal | — | Emoglobina |
| `plt_x10_9_l` | Decimal | — | Piastrine |
| `neutrofili_x10_9_l` | Decimal | — | — |
| `creatinina_mg_dl` | Decimal | — | Funzione renale |
| `egfr_ml_min` | Decimal | — | — |
| `psa_ng_ml` | Decimal | — | Per PSMA (risposta al trattamento) |
| `cromogranina_a` | Decimal | — | Per NET (risposta al trattamento) |
| `tossicita_ematologica` | Enum | — | Grading CTCAE: `G0` \| `G1` \| `G2` \| `G3` \| `G4` |
| `referto_allegato` | File | — | Referto ematochimico |
| `note` | Text | — | — |

---

## 6.7 Elaborazione Statistica dei Dati di Trattamento

Il sistema genera automaticamente statistiche aggregate (anonimizzate) per:

### Statistiche di popolazione

| Indicatore | Descrizione |
|------------|-------------|
| Numero pazienti trattati (per anno, cumulativo) | — |
| Distribuzione per indicazione (PSMA vs DOTATATE vs altro) | — |
| Numero medio di cicli completati | — |
| Attività media somministrata per ciclo | — |
| Distribuzione tossicità ematologica (grading CTCAE) | — |

### Statistiche dosimetriche

| Indicatore | Descrizione |
|------------|-------------|
| Dose media organi a rischio (reni, midollo) per ciclo e cumulativa | — |
| Distribuzione dosi tumorali | — |
| Correlazione dose renale / eGFR | — |

### Output

- Grafici interattivi (line chart, box plot, scatter plot)
- Export in Excel / CSV per analisi esterne
- Generazione report PDF per pubblicazioni / audit interni

---

## 6.8 Gestione Rifiuti Correlati al Trattamento Lu177

Specifico per Lu177 (t½ = 6.73 giorni), i rifiuti includono:

| Tipo rifiuto | Esempi | Gestione |
|--------------|--------|---------|
| Materiale monouso contaminato | Siringhe (residuo), guanti, garze | Decadimento in sito (zona calda), poi smaltimento come rifiuto speciale non radioattivo |
| Rifiuti corporei | Urine prime 24-48h (se paziente ricoverato) | Raccolta in contenitori schermati, decadimento |
| Flaconi vuoti radiofarmaco | — | Restituzione al fornitore o decadimento |

Per ogni record rifiuto:

| Campo | Tipo | Note |
|-------|------|-------|
| `fk_paziente` | FK → Paziente | — |
| `fk_ciclo` | FK → Ciclo | — |
| `tipo_rifiuto` | Enum | `MONOUSO_CONTAMINATO` \| `RIFIUTI_CORPOREI` \| `FLACONI_VUOTI` |
| `peso_kg` | Decimal | — |
| `attivita_stimata_mbq` | Decimal | — |
| `data_produzione` | Date | — |
| `data_smaltimento_previsto` | Date | Calcolata automaticamente (almeno 10 t½) |
| `data_smaltimento_effettivo` | Date | — |
| `modalita_smaltimento` | Enum | `DECADIMENTO_IN_SITO` \| `SMALTIMENTO_ESTERNO` |
| `stato` | Enum | `IN_DECADIMENTO` \| `SMALTITO` |

---

## 6.9 Norme, Segnaletica e Procedure

### Norme interne
- Procedure di somministrazione Lu177
- Gestione del paziente post-somministrazione (isolamento, precauzioni di contatto)
- Procedura di emergenza (spandimento)
- Istruzioni per il paziente (da fornire prima della dimissione)

### Segnaletica
- Segnaletica camera isolamento (se paziente ricoverato)
- Segnaletica bagno dedicato
- Avvisi per personale e visitatori

### Norme tecniche di riferimento
- Linee guida EANM per dosimetria Lu177
- Linee guida SNMMI
- Circolari ISIN applicabili