# 08 — Adempimenti Fisica Medica (Titolo XIII D.Lgs. 101/20)

---

## 8.1 Panoramica

Questo documento descrive tutti i flussi operativi e le logiche applicative per gli adempimenti di **Fisica Medica** (esposizioni mediche — pazienti), come definiti dal **Titolo XIII del D.Lgs. 101/20** e dettagliati dall'Allegato XXVII.

I processi principali sono:
1. Manuale di Qualità dell'Apparecchiatura
2. Accettazione (collaudo iniziale)
3. Verifiche periodiche di corretto funzionamento
4. Verifiche dopo manutenzione rilevante
5. Verifiche LDR

---

## 8.2 Manuale di Qualità dell'Apparecchiatura

### Riferimento normativo
- D.Lgs. 101/20, Allegato XXVII

### Descrizione
Il Manuale è un documento **strutturato e parzialmente auto-generato** che accompagna ogni apparecchiatura per tutta la sua vita operativa.

### Struttura e modalità di generazione

```
PARTE STATICA
(inserita manualmente all'atto della creazione dell'apparecchiatura)
  ├── Dati anagrafici dell'apparecchiatura
  ├── Caratteristiche tecniche (parametri radiologici)
  ├── Figure di responsabilità (RIR, EFM)
  ├── Normativa applicabile
  ├── Planimetria e zone classificate
  └── Manuale d'uso (allegato)

PARTE DINAMICA
(generata automaticamente dalla piattaforma)
  ├── Sezione Accettazione → popolata dal record di accettazione (§8.3)
  ├── Sezione Verifiche Periodiche → popolata dai record CQ (§8.4)
  └── Sezione LDR → popolata dai report LDR (§8.6)
```

### Funzionalità richieste

| Funzione | Descrizione |
|----------|-------------|
| Visualizzazione in-app | Il manuale è consultabile direttamente nella scheda apparecchiatura |
| Export PDF | Generazione PDF completo (parti statiche + dinamiche aggiornate) |
| Stampa | Layout ottimizzato per stampa A4 |
| Versioning | Ogni export PDF è versionato e archiviato con data di generazione |

---

## 8.3 Accettazione — Collaudo Iniziale

### Riferimento normativo
- D.Lgs. 101/20, art. relativo all'entrata in uso (Titolo XIII)

### Descrizione
Prima dell'entrata in uso di un'apparecchiatura è obbligatorio eseguire un **collaudo con esito positivo**. L'esito positivo rappresenta la "data di nascita" dell'apparecchiatura nel sistema.

### Flusso operativo

```
1. Creazione record accettazione
       ↓
2. Selezione protocollo di accettazione (per tipo apparecchiatura)
       ↓
3. Caricamento file di lavoro (Excel) con parametri rilevati
       ↓
4. Caricamento report delle verifiche eseguite
       ↓
5a. Benestare qualità tecnica (SFM)        5b. Benestare clinico (RIR)
       ↓                                          ↓
                    6. Entrata in uso (ATTIVA)
                           ↓
              7. Data accettazione salvata → Manuale Qualità aggiornato
```

### Campi del record di accettazione

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `protocollo_id` | FK → ProtocolloVerifica | ✓ | Protocollo di accettazione applicabile |
| `data_inizio_verifica` | Date | ✓ | — |
| `data_fine_verifica` | Date | ✓ | — |
| `eseguito_da_sfm` | FK → Utente | ✓ | SFM che ha eseguito le prove |
| `elenco_tipologie_controllo` | String[] | ✓ | Lista prove eseguite |
| `file_lavoro` | File | ✓ | Foglio Excel dei dati rilevati |
| `report_verifiche` | File | ✓ | Report con risultati |
| `benestare_qualita_tecnica_data` | Date | ✓ | — |
| `benestare_qualita_tecnica_by` | FK → Utente (SFM) | ✓ | — |
| `benestare_clinico_data` | Date | ✓ | — |
| `benestare_clinico_by` | FK → Utente (RIR) | ✓ | — |
| `esito` | Enum | ✓ | `SUPERATO` \| `NON_SUPERATO` \| `CON_RISERVA` |
| `note` | Text | — | — |

### Protocolli di accettazione

Il sistema mantiene un catalogo di **protocolli di accettazione** per tipologia di apparecchiatura:

```typescript
interface ProtocolloAccettazione {
  id: string;
  codice: string;               // Es. "PA-TAC-001"
  descrizione: string;          // Es. "Protocollo accettazione TAC"
  ambito: AmbitorIntervento;
  tipologie_applicabilita: string[];
  revisione: string;
  data_entrata_vigore: Date;
  elenco_prove: string[];       // Lista delle prove previste
  documento: File;              // Documento del protocollo
}
```

---

## 8.4 Verifiche Periodiche di Corretto Funzionamento

### Descrizione
Le apparecchiature sono sottoposte a verifiche di corretto funzionamento a **intervalli regolari**, secondo protocolli stabiliti dall'SFM e dipendenti dalla tipologia.

### Flusso operativo

```
Sistema → Alert scadenza verifica imminente (30gg, 7gg)
       ↓
SFM crea nuovo record verifica periodica
       ↓
Selezione protocollo di verifica periodica
       ↓
Esecuzione prove → Caricamento file di lavoro
       ↓
Caricamento report
       ↓
Registrazione esito (SUPERATO / NON_SUPERATO / CON_RISERVA)
       ↓
Se NON_SUPERATO → Apertura segnalazione + notifica RIR
       ↓
Apparecchiatura aggiornata → Manuale Qualità aggiornato
```

### Campi del record verifica periodica

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `protocollo_id` | FK | ✓ | — |
| `data_verifica` | Date | ✓ | — |
| `eseguito_da` | FK → Utente | ✓ | — |
| `file_lavoro` | File | — | — |
| `report` | File | ✓ | — |
| `esito` | Enum | ✓ | — |
| `info_guasto` | Text | Cond. | Se esito ≠ SUPERATO |
| `tipo_guasto` | Enum | — | `HARDWARE` \| `SOFTWARE` \| `CALIBRAZIONE` \| `ALTRO` |
| `note` | Text | — | — |
| `anno` | Integer | ✓ | Anno di riferimento |
| `prossima_verifica_data` | Date | ✓ | Calcolata in base periodicità protocollo |

### Periodicità indicative

| Tipologia | Periodicità |
|-----------|-------------|
| Mammografi | Semestrale / Annuale (secondo linee guida EUREF) |
| TAC | Semestrale / Annuale |
| Acceleratori lineari | Giornaliera (CQ rapido) + Mensile + Annuale |
| PET-CT | Giornaliera (CQ rapido) + Trimestrale + Annuale |
| RM | Semestrale (cfr. Modulo 2) |

> Le periodicità esatte sono definite nel protocollo e configurabili per ogni apparecchiatura.

---

## 8.5 Verifiche dopo Manutenzione Rilevante

### Descrizione
Ogni volta che un'apparecchiatura subisce un **intervento di manutenzione rilevante** (sostituzione componente principale, guasto significativo, aggiornamento software critico), deve essere eseguita una verifica analoga all'accettazione prima di rimettere l'apparecchiatura in servizio.

### Gestione nel sistema

1. L'operatore registra l'intervento di manutenzione rilevante
2. Lo stato dell'apparecchiatura passa automaticamente in `IN_MANUTENZIONE`
3. Viene creato un record di verifica di tipo `POST_MANUTENZIONE`
4. Fino a esito positivo della verifica, l'apparecchiatura rimane `IN_MANUTENZIONE`

### Campi aggiuntivi per manutenzione

| Campo | Tipo | Note |
|-------|------|-------|
| `tipo_intervento_manutenzione` | String | Descrizione dell'intervento |
| `eseguito_da_tecnico` | String | Nome tecnico manutentore |
| `societa_manutenzione` | String | — |
| `data_intervento` | Date | — |
| `componenti_sostituiti` | Text | Elenco componenti |
| `rapporto_tecnico_manutenzione` | File | Report del tecnico manutentore |

---

## 8.6 Verifiche LDR

### Riferimento normativo
- D.Lgs. 101/20 (frequenza annuale o biennale)

### Descrizione
Le verifiche LDR (Livelli Diagnostici di Riferimento) valutano se le dosi erogate ai pazienti sono in linea con i livelli diagnostici di riferimento nazionali/europei.

Si ottengono mediante:
- Dati raccolti manualmente (campionamento)
- Dati derivanti da software installati sull'apparecchiatura (es. MedSquare)

### Gestione nel sistema

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `anno` | Integer | ✓ | — |
| `tipo_verifica` | Enum | ✓ | `ANNUALE` \| `BIENNALE` |
| `metodo_raccolta` | Enum | ✓ | `MANUALE` \| `MEDSQUARE` \| `ALTRO_SOFTWARE` |
| `report_ldr` | File | ✓ | Documento PDF con risultati |
| `esito` | Enum | ✓ | `ENTRO_LDR` \| `SUPERATO_LDR` \| `ANALISI_IN_CORSO` |
| `note` | Text | — | — |
| `data_prossima_verifica` | Date | ✓ | — |

> **Se `medsquare_installato = true`:** il sistema segnala la disponibilità del software per estrarre automaticamente i dati LDR.

---

## 8.7 Sezione Norme di Riferimento (Fisica Medica)

Il sistema archivia e rende consultabili le norme interne per:

```
Norme di Radiologia
  └── Per tipo apparecchiatura (TAC, Mammografi, ecc.)
      └── Per singola apparecchiatura
          
Norme di Radiologia Interventistica
Norme di Radioterapia
Norme di Medicina Nucleare
```

Ogni norma interna è:
- Associata all'apparecchiatura (o al tipo, o all'ambito)
- Firmata da EDR e Datore di Lavoro
- Versionata
- Scaricabile in PDF

---

## 8.8 Riepilogo Adempimenti — Vista per Apparecchiatura

La scheda apparecchiatura deve mostrare un **pannello di compliance** con lo stato di tutti gli adempimenti:

```
📋 COMPLIANCE FISICA MEDICA — [Nome Apparecchiatura]
┌────────────────────────────────────────────────────────┐
│ ✅ Manuale di Qualità             Generato: 15/01/2025 │
│ ✅ Accettazione                   Data: 10/03/2022     │
│ ✅ Verifica CQ Annuale 2024       Esito: SUPERATO      │
│ ⚠️  Verifica CQ Annuale 2025      Scadenza: 15/03/2025 │
│ ✅ LDR 2024                       Esito: ENTRO LDR     │
│ 🔴 LDR 2023                       Mancante             │
└────────────────────────────────────────────────────────┘
```

**Legenda:**
- ✅ Verde: in regola
- ⚠️ Giallo: scadenza imminente (< 30 giorni) o con riserva
- 🔴 Rosso: scaduto o mancante