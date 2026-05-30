# 09 — Adempimenti Radioprotezione (Titolo XI D.Lgs. 101/20)

---

## 9.1 Panoramica

Questo documento descrive i flussi operativi per gli adempimenti di **Radioprotezione** (esposizione dei lavoratori e sorveglianza ambientale), come definiti dal **Titolo XI del D.Lgs. 101/20** e gestiti dall'Esperto di Radioprotezione (EdR).

I processi principali sono:
1. Notifica di Pratica
2. Nulla Osta (NO)
3. Prima Verifica
4. Verifiche di Sorveglianza Periodica
5. Notifica di Cessazione
6. Gestione STRIMS
7. Gestione INAIL
8. Verbali di Sopralluogo
9. Dosimetria degli Operatori (in sviluppo)

---

## 9.2 Notifica di Pratica

### Riferimento normativo
- D.Lgs. 101/20, art. 46

### Descrizione
Comunicazione obbligatoria agli organi competenti che deve essere trasmessa via **PEC** prima dell'entrata in uso di un'apparecchiatura soggetta a notifica.

### Campi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `numero_protocollo` | String | ✓ | Numero protocollo della PEC inviata |
| `data_notifica` | Date | ✓ | Data invio |
| `ente_destinatario` | String | ✓ | Es. "ASL competente", "ISIN" |
| `documento_notifica` | File | ✓ | Documento di notifica (con valutazione del rischio allegata) |
| `ricevuta_pec` | File | ✓ | Ricevuta di consegna PEC |
| `documento_valutazione_rischio` | File | ✓ | Doc. art. 109 D.Lgs. 101/20 (va anche all'RSPP) |
| `inviato_rspp` | Boolean | ✓ | Conferma invio doc. valutazione rischio all'RSPP |
| `data_invio_rspp` | Date | Cond. | Se `inviato_rspp = true` |
| `fac_simile_disponibile` | Boolean | — | Sistema fornisce fac-simile per il tipo di apparecchiatura |

### Template fac-simile

Il sistema deve mantenere un archivio di **fac-simile di Notifica di Pratica** per tipo di apparecchiatura, precompilabili con i dati anagrafici dell'apparecchiatura selezionata.

---

## 9.3 Nulla Osta (NO)

### Tipi di Nulla Osta

| Tipo | Descrizione |
|------|-------------|
| **NO Tipo A** | Per pratiche con sorgenti sigillate o apparecchi generatori di radiazioni ionizzanti |
| **NO Tipo B** | Per pratiche con sorgenti non sigillate |

### Campi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `tipo_no` | Enum | ✓ | `TIPO_A` \| `TIPO_B` |
| `numero_no` | String | ✓ | — |
| `data_rilascio` | Date | ✓ | — |
| `ente_rilascio` | String | ✓ | — |
| `data_scadenza` | Date | — | Se NO temporaneo |
| `documento_no` | File | ✓ | — |
| `stato` | Enum | ✓ | `VALIDO` \| `IN_SCADENZA` \| `SCADUTO` \| `IN_RINNOVO` |

---

## 9.4 Benestare all'Utilizzo

Documento che formalizza il via libera dell'EdR all'utilizzo dell'apparecchiatura dopo la prima verifica.

| Campo | Tipo | Note |
|-------|------|-------|
| `apparecchiatura_id` | FK | — |
| `data_benestare` | Date | — |
| `firmato_da` | FK → Utente (EdR) | — |
| `documento` | File | — |
| `valido_fino` | Date | Se temporaneo / con condizioni |
| `condizioni` | Text | Eventuali prescrizioni |

---

## 9.5 Prima Verifica

### Descrizione
Analogo al collaudo di accettazione della Fisica Medica, ma eseguita dall'EdR secondo un **protocollo di prima verifica** specifico per tipologia di apparecchiatura.

### Campi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `protocollo_prima_verifica_id` | FK | ✓ | Protocollo EDR applicabile |
| `data_inizio` | Date | ✓ | — |
| `data_fine` | Date | ✓ | — |
| `eseguito_da_edr` | FK → Utente | ✓ | — |
| `elenco_tipologie_controllo` | String[] | ✓ | — |
| `dati_verifiche` | File | ✓ | Dati rilevati durante la verifica |
| `report_prima_verifica` | File | ✓ | — |
| `esito` | Enum | ✓ | `SUPERATO` \| `NON_SUPERATO` \| `CON_RISERVA` |
| `note` | Text | — | — |

---

## 9.6 Verifiche di Sorveglianza Periodica

### Descrizione
Verifiche periodiche condotte dall'EdR sull'apparecchiatura con frequenza stabilita dal protocollo.

### Campi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `protocollo_sorveglianza_id` | FK | ✓ | — |
| `data_verifica` | Date | ✓ | — |
| `eseguito_da_edr` | FK → Utente | ✓ | — |
| `dati_verifiche` | File | — | — |
| `report_sorveglianza` | File | ✓ | — |
| `esito` | Enum | ✓ | — |
| `dosimetria_ambientale_inclusa` | Boolean | — | Se la verifica include misure dosimetriche |
| `valori_dosimetrici` | JSON | Cond. | Valori misurati, se applicabile |
| `note` | Text | — | — |
| `data_prossima_verifica` | Date | ✓ | — |

---

## 9.7 Notifica di Cessazione

### Riferimento normativo
- D.Lgs. 101/20, art. 48

### Descrizione
Da inviare agli organi competenti via PEC quando un'apparecchiatura cessa definitivamente l'attività.

### Flusso

```
Operatore imposta stato = CESSATA
       ↓
Sistema richiede creazione record "Notifica di Cessazione"
       ↓
Compilazione documento
       ↓
Invio via PEC
       ↓
Caricamento ricevuta PEC
       ↓
Aggiornamento STRIMS (notifica cessazione)
       ↓
Apparecchiatura spostata in archivio "Cessate"
```

### Campi

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `apparecchiatura_id` | FK | ✓ | — |
| `data_cessazione` | Date | ✓ | — |
| `motivo_cessazione` | Enum | ✓ | `OBSOLESCENZA` \| `GUASTO_IRREPARABILE` \| `SOSTITUZIONE` \| `TRASFERIMENTO` \| `ALTRO` |
| `numero_protocollo_pec` | String | ✓ | — |
| `data_invio_pec` | Date | ✓ | — |
| `ente_destinatario` | String | ✓ | — |
| `documento_notifica` | File | ✓ | — |
| `ricevuta_pec` | File | ✓ | — |

---

## 9.8 Gestione STRIMS

### Descrizione
STRIMS è la piattaforma ISIN per la registrazione delle sorgenti e delle apparecchiature. Ogni apparecchiatura deve essere registrata, aggiornata a ogni cambio significativo e notificata alla cessazione.

### Operazioni supportate nel sistema

| Operazione | Descrizione | Documenti |
|------------|-------------|-----------|
| Registrazione iniziale | All'atto della notifica di pratica | Ricevuta STRIMS, numero identificativo |
| Aggiornamento dati | In caso di variazioni significative | — |
| Notifica di pratica | Caricamento documento su STRIMS | Conferma caricamento |
| Notifica di cessazione | All'atto della cessazione | Ricevuta |

### Campi nella scheda apparecchiatura

| Campo | Tipo | Note |
|-------|------|-------|
| `strims_id_apparecchiatura` | String | ID assegnato da STRIMS |
| `strims_stato` | Enum | `DA_REGISTRARE` \| `REGISTRATO` \| `CESSATO` |
| `strims_data_registrazione` | Date | — |
| `strims_ricevuta_registrazione` | File | — |
| `strims_np_caricata` | Boolean | Notifica di Pratica caricata su STRIMS |
| `strims_nc_caricata` | Boolean | Notifica di Cessazione caricata su STRIMS |

> **Nota implementativa (roadmap):** Futura integrazione API con STRIMS per automazione degli invii, quando disponibile.

---

## 9.9 Gestione INAIL

| Campo | Tipo | Note |
|-------|------|-------|
| `inail_stato` | Enum | `DA_REGISTRARE` \| `REGISTRATO` \| `CESSATO` |
| `inail_numero_pratica` | String | — |
| `inail_data_registrazione` | Date | — |
| `inail_ricevuta` | File | — |
| `inail_data_cessazione` | Date | — |
| `inail_ricevuta_cessazione` | File | — |

---

## 9.10 Verbali di Sopralluogo Periodici

I verbali documentano i **sopralluoghi periodici** dell'EdR presso i reparti con apparecchiature radiologiche.

### Struttura verbale

| Campo | Tipo | Obbligatorio | Note |
|-------|------|:---:|-------|
| `id` | UUID | ✓ | — |
| `apparecchiatura_id` | FK | — | Può essere generico per reparto |
| `reparto_id` | FK | ✓ | — |
| `data_sopralluogo` | Date | ✓ | — |
| `edr_id` | FK → Utente | ✓ | EdR che ha effettuato il sopralluogo |
| `partecipanti` | String | — | Altre persone presenti |
| `oggetto` | String | ✓ | Oggetto del sopralluogo |
| `rilievi` | Text | ✓ | Osservazioni e rilievi |
| `non_conformita` | Text | — | Eventuali non conformità |
| `azioni_correttive` | Text | Cond. | Se ci sono non conformità |
| `scadenza_azioni` | Date | Cond. | — |
| `verbale_firmato` | File | ✓ | PDF con firma EdR |
| `stato` | Enum | ✓ | `APERTO` \| `CHIUSO` \| `IN_ATTESA_AZIONI` |
| `data_chiusura` | Date | Cond. | — |

---

## 9.11 Gestione Dosimetria Operatori (in sviluppo)

> **Stato:** Funzionalità da sviluppare in fase successiva ("Gestione dosimetria operatori — Da sviluppare" nella documentazione originale).

### Scope pianificato

**Gestione dei dosimetri personali:**
- Anagrafica lavoratori esposti
- Assegnazione dosimetri personali (badge, anello, dosimetro di petto)
- Registrazione letture periodiche (mensili/trimestrali)
- Confronto con dosi limite e livelli di indagine
- Storico dosimetrico per lavoratore

**Cassetto virtuale per gli operatori:**
- Accesso del singolo lavoratore alle proprie dosi cumulate
- Confronto con i limiti normativi
- Download referto dosimetrico personale

### Struttura dati preliminare

```typescript
interface LavoratoreEsposto {
  id: string;
  tenant_id: string;
  nome: string;
  cognome: string;
  codice_fiscale: string;
  reparto_id: FK;
  mansione: string;
  categoria: 'A' | 'B';               // Categoria di rischio
  data_assunzione_reparto: Date;
  data_cessazione?: Date;
}

interface DosimetriaPersonale {
  id: string;
  lavoratore_id: FK;
  periodo: string;                     // Es. "Q1-2025", "Gen-2025"
  tipo_dosimetro: string;
  dose_hp10_msv: number;               // Corpo intero
  dose_hp007_msv?: number;             // Cute
  dose_hp3_msv?: number;               // Cristallino
  laboratorio_lettura: string;
  rapporto_lettura: File;
  esito: 'NELLA_NORMA' | 'LIVELLO_INDAGINE' | 'SUPERATO_LIMITE';
}
```

---

## 9.12 Riepilogo Adempimenti — Vista per Apparecchiatura (EDR)

```
📋 COMPLIANCE RADIOPROTEZIONE — [Nome Apparecchiatura]
┌────────────────────────────────────────────────────────────┐
│ ✅ Notifica di Pratica          Inviata: 05/03/2022        │
│ ✅ Nulla Osta (Tipo A)          Valido fino: 04/03/2027    │
│ ✅ Prima Verifica               Eseguita: 10/03/2022       │
│ ✅ Benestare utilizzo           Rilasciato: 15/03/2022     │
│ ✅ Registrazione STRIMS         Stato: REGISTRATO          │
│ ✅ Registrazione INAIL          Stato: REGISTRATO          │
│ ✅ Sorveglianza periodica 2024  Esito: SUPERATO            │
│ ⚠️  Sorveglianza periodica 2025 Scadenza: 10/03/2025       │
│ ✅ Verbale sopralluogo         Ultimo: 20/11/2024          │
└────────────────────────────────────────────────────────────┘
```