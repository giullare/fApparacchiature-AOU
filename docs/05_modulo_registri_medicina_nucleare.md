# 05 — Modulo 3: Registri di Sorveglianza in Medicina Nucleare
### Riferimento normativo: D.Lgs. 101/20

---

## 5.1 Panoramica del modulo

Il Modulo 3 gestisce i **registri di sorveglianza ambientale e radiologica** specifici della Medicina Nucleare. Si tratta di un sistema di raccolta dati strutturato per dosimetria ambientale, contaminazione superficiale, gestione radiofarmaci e sorgenti, rifiuti radioattivi e controllo della ventilazione.

I registri devono essere:
- **Auto-compilabili** (inserimento guidato dall'interfaccia)
- **Stampabili** in versione formattata da allegare al Registro di Radioprotezione cartaceo
- **Archiviati** con storico completo per periodo

---

## 5.2 Struttura del modulo

```
Modulo 3 — Registri Sorveglianza Medicina Nucleare
├── 📄 Documentazione e Registri Chiave
│   ├── Relazione Tecnica + file di appoggio
│   ├── Nulla Osta (NO) autorizzativo
│   ├── Piantina strumentazione di monitoraggio
│   ├── Registro dinamico Apparecchiature MN
│   ├── Registro Sorgenti Sigillate e Non Sigillate
│   ├── Registro attività radiofarmaco (semestrale)
│   └── Registro strumentazione radioprotezione + calibrazioni
│
├── 🌡️ Sezione Dosimetria Ambientale
│   ├── Dosimetri
│   └── Sonde (area monitor)
│
├── ⚠️ Sezione Contaminazione Superficiale
│
├── 🗑️ Sezione Registro Rifiuti
│
├── 💊 Sezione Inventario Radiofarmaci e Sorgenti
│   └── Attività autorizzate
│
├── 🔬 Sezione Inventario Strumentazione
│
├── 💨 Sezione Controllo Ventilazione e Pressione
│
├── 📊 Statistica Operativa
│
└── 🖨️ Generazione Report e Verbale Periodico
```

---

## 5.3 Documentazione e Registri Chiave

### 5.3.1 Relazione Tecnica

| Campo | Tipo | Note |
|-------|------|-------|
| `anno_riferimento` | Year | Anno di competenza |
| `data_redazione` | Date | — |
| `redattore` | FK → Utente (EdR) | — |
| `documento_relazione` | File | PDF della relazione tecnica firmata |
| `file_supporto` | File[] | File Excel o altri file di appoggio allegati |

### 5.3.2 Nulla Osta Autorizzativo

| Campo | Tipo | Note |
|-------|------|-------|
| `numero_no` | String | Numero del Nulla Osta |
| `data_rilascio` | Date | — |
| `ente_rilascio` | String | — |
| `data_scadenza` | Date | Per NO temporanei |
| `tipo_no` | Enum | `TIPO_A` \| `TIPO_B` |
| `documento` | File | — |
| `stato` | Enum | `VALIDO` \| `SCADUTO` \| `IN_RINNOVO` |

### 5.3.3 Registro Dinamico Apparecchiature MN

Questo registro riflette in tempo reale lo stato delle apparecchiature di Medicina Nucleare (gestite anche nel Modulo 1), con focus su:
- Data ultima verifica
- Numero di utilizzi nel periodo
- Stato operativo corrente
- Eventuali anomalie segnalate

---

## 5.4 Sezione Dosimetria Ambientale

### 5.4.1 Dosimetri (passivi)

**Tipologie:** TLD (LiF), OSL, film badge, dosimetri a dito

| Campo | Tipo | Note |
|-------|------|-------|
| `id_dosimetro` | String | Codice identificativo |
| `tipo_dosimetro` | Enum | `TLD` \| `OSL` \| `FILM_BADGE` \| `ANELLO` |
| `posizione_installazione` | String | Descrizione posizione |
| `fk_piantina_punto` | Reference | Collegamento al punto sulla piantina |
| `data_installazione` | Date | — |
| `data_ritiro` | Date | — |
| `periodo_esposizione` | String | Es. "Q1-2024", "Semestre 1-2024" |
| `dose_misurata_msv` | Decimal | Dose misurata in mSv |
| `laboratorio_lettura` | String | Laboratorio che ha effettuato la lettura |
| `rapporto_lettura` | File | Rapporto del laboratorio |
| `esito` | Enum | `NELLA_NORMA` \| `SUPERATO_LIVELLO_INDAGINE` \| `SUPERATO_LIMITE` |

### 5.4.2 Sonde (area monitor — dosimetri attivi)

| Campo | Tipo | Note |
|-------|------|-------|
| `id_sonda` | String | Codice identificativo |
| `modello` | String | — |
| `matricola` | String | — |
| `posizione` | String | — |
| `tipo_rilevazione` | Enum | `GAMMA` \| `BETA` \| `ALFA` \| `NEUTRONI` |
| `data_ultima_calibrazione` | Date | — |
| `unita_misura` | Enum | `µSv/h` \| `mSv/h` \| `Sv/h` |
| `valore_fondo_riferimento` | Decimal | — |
| `soglia_allerta_1` | Decimal | Primo livello di allerta |
| `soglia_allerta_2` | Decimal | Secondo livello di allerta (livello di allarme) |

**Registro letture sonde:**

| Campo | Tipo | Note |
|-------|------|-------|
| `fk_sonda` | FK → Sonda | — |
| `data_lettura` | DateTime | — |
| `operatore` | FK → Utente | — |
| `valore_misurato` | Decimal | — |
| `esito` | Enum | `NELLA_NORMA` \| `LIVELLO_INDAGINE` \| `LIVELLO_ALLARME` |
| `azione_intrapresa` | Text | In caso di superamento soglie |

---

## 5.5 Sezione Contaminazione Superficiale

| Campo | Tipo | Note |
|-------|------|-------|
| `data_misura` | DateTime | — |
| `operatore` | FK → Utente | — |
| `strumento_utilizzato` | FK → Strumentazione | — |
| `area_misurata` | String | Descrizione area (es. "Bancone preparazione", "Pavimento zona calda") |
| `fk_piantina_punto` | Reference | — |
| `tipo_contaminante` | String | Radionuclide (es. Tc-99m, F-18, Lu-177) |
| `valore_misurato_bq_cm2` | Decimal | Attività superficiale in Bq/cm² |
| `limite_riferimento_bq_cm2` | Decimal | Limite normativo o interno |
| `esito` | Enum | `ASSENTE` \| `ENTRO_LIMITI` \| `SUPERATO_LIMITE` |
| `azione_decontaminazione` | Text | Azioni intraprese se superamento |
| `verifica_post_decontaminazione` | Decimal | Valore dopo decontaminazione |
| `periodo` | String | Riferimento periodo |

---

## 5.6 Sezione Registro Rifiuti

### Tipologie di rifiuti radioattivi gestiti

| Categoria | Esempi |
|-----------|--------|
| Solidi a bassa attività | Guanti, siringhe vuote, materiale monouso contaminato |
| Liquidi | Urine pazienti (primo giorno), soluzioni contaminate |
| Sorgenti esaurite | Sorgenti di calibrazione, generatori esauriti |

### Campi per ogni record di rifiuto

| Campo | Tipo | Note |
|-------|------|-------|
| `data_produzione` | Date | — |
| `tipo_rifiuto` | Enum | `SOLIDO_BASSA` \| `SOLIDO_MEDIA` \| `LIQUIDO` \| `SORGENTE_ESAURITA` |
| `radionuclide` | String | Es. "Tc-99m", "I-131", "Lu-177" |
| `attivita_iniziale_mbq` | Decimal | — |
| `data_smaltimento_previsto` | Date | Calcolata in base a periodo di dimezzamento |
| `data_smaltimento_effettivo` | Date | — |
| `modalita_smaltimento` | Enum | `DECADIMENTO_IN_SITO` \| `SMALTIMENTO_ESTERNO` |
| `smaltitore` | String | Azienda di smaltimento (se esterno) |
| `documento_smaltimento` | File | — |
| `stato` | Enum | `IN_DECADIMENTO` \| `SMALTITO` \| `TRASFERITO` |

---

## 5.7 Sezione Inventario Radiofarmaci e Sorgenti

### 5.7.1 Inventario Radiofarmaci

Aggiornato **semestralmente** o alla variazione significativa.

| Campo | Tipo | Note |
|-------|------|-------|
| `nome_radiofarmaco` | String | Es. "⁹⁹ᵐTc-MDP", "¹⁸F-FDG" |
| `radionuclide` | String | — |
| `fornitore` | String | — |
| `attivita_autorizzata_mbq` | Decimal | Dal Nulla Osta |
| `attivita_utilizzata_semestre_mbq` | Decimal | — |
| `data_rilevamento` | Date | — |
| `semestre` | String | — |

### 5.7.2 Inventario Sorgenti Sigillate e Non Sigillate

| Campo | Tipo | Note |
|-------|------|-------|
| `id_sorgente` | String | Codice univoco |
| `tipo_sorgente` | Enum | `SIGILLATA` \| `NON_SIGILLATA` |
| `radionuclide` | String | — |
| `attivita_certificata_mbq` | Decimal | — |
| `data_attivita_certificata` | Date | — |
| `attivita_attuale_calcolata` | Decimal | Calcolata automaticamente dal sistema (decadimento) |
| `produttore` | String | — |
| `certificato_sorgente` | File | — |
| `utilizzo` | String | Es. "Calibrazione", "Controllo qualità" |
| `ubicazione` | String | Dove è conservata |
| `stato` | Enum | `ATTIVA` \| `ESAURITA` \| `RESTITUITA` |

> **Funzionalità automatica:** Il sistema calcola e aggiorna l'attività attuale delle sorgenti applicando la legge del decadimento radioattivo: A(t) = A₀ × e^(-λt), dove λ = ln(2)/t½

---

## 5.8 Sezione Inventario Strumentazione

Strumentazione di radioprotezione utilizzata in MN:

| Campo | Tipo | Note |
|-------|------|-------|
| `nome` | String | Es. "Calibratore di dose", "Contatore Geiger" |
| `modello` | String | — |
| `matricola` | String | — |
| `tipo_rilevazione` | String | — |
| `data_ultima_calibrazione` | Date | — |
| `data_prossima_calibrazione` | Date | Alert automatico |
| `certificato_calibrazione` | File | — |
| `stato` | Enum | `IN_REGOLA` \| `SCADUTA` \| `IN_RIPARAZIONE` |

---

## 5.9 Sezione Controllo Ventilazione e Pressione

| Campo | Tipo | Note |
|-------|------|-------|
| `data_controllo` | DateTime | — |
| `operatore` | FK → Utente | — |
| `numero_ricambi_ora` | Decimal | Ricambi d'aria per ora |
| `limite_ricambi_ora` | Decimal | Limite normativo/interno |
| `pressione_differenziale_pa` | Decimal | In Pascal (negativa per zone calde) |
| `esito_ventilazione` | Enum | `ADEGUATA` \| `INSUFFICIENTE` |
| `esito_pressione` | Enum | `ADEGUATA` \| `NON_ADEGUATA` |
| `azione_correttiva` | Text | In caso di non conformità |
| `report` | File | — |

---

## 5.10 Verbale Periodico (auto-compilabile)

Il sistema genera automaticamente un **verbale periodico** aggregando i dati inseriti nelle varie sezioni per un determinato periodo.

### Struttura del verbale generato

```
VERBALE PERIODICO DI SORVEGLIANZA — MEDICINA NUCLEARE
Periodo: [Da] — [A]

1. DOSIMETRIA AMBIENTALE
   - Tabella dosimetri: posizione | periodo | dose misurata | esito
   - Tabella sonde: posizione | data | valore | esito

2. CONTAMINAZIONE SUPERFICIALE
   - Tabella misure: area | data | valore | esito

3. INVENTARIO RADIOFARMACI E SORGENTI
   - Riepilogo attività utilizzate vs autorizzate

4. RIFIUTI
   - Tabella rifiuti prodotti e smaltiti nel periodo

5. VENTILAZIONE E PRESSIONE
   - Tabella controlli: data | ricambi/h | pressione | esito

6. ANOMALIE E AZIONI CORRETTIVE
   - Lista non conformità e azioni intraprese

Firma EdR: ___________      Data: ___________
```

**Output:** Il verbale è scaricabile in PDF con firma digitale o spazio per firma autografa.

---

## 5.11 Statistiche Operative

| Indicatore | Frequenza |
|------------|-----------|
| Trend dosi ambientali per punto di misura | Mensile/Trimestrale |
| Contaminazioni rilevate vs limite | Semestrale |
| Attività radiofarmaci utilizzata vs autorizzata | Semestrale |
| Rifiuti prodotti per radionuclide | Annuale |
| Stato calibrazioni strumentazione | Continuo |
| Percentuale non conformità ventilazione | Semestrale |