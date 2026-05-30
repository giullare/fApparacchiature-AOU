# 07 — Schema Dati Completo: Entità Apparecchiatura

---

## 7.1 Premessa

Questo documento descrive in modo esaustivo la struttura dati dell'entità centrale del sistema: l'**Apparecchiatura**. È il riferimento definitivo per il team di sviluppo backend per la definizione delle tabelle, delle API e della logica di validazione.

---

## 7.2 Anagrafica Base (comune a tutti i moduli)

```typescript
interface Apparecchiatura {
  // Identificazione
  id: string;                          // UUID v4
  tenant_id: string;                   // UUID — quale azienda sanitaria
  codice: string;                      // Codice univoco interno (tenant-scoped)
  descrizione: string;                 // Nome leggibile

  // Classificazione
  modulo: 'RADIOLOGICA' | 'RM';        // A quale modulo appartiene
  ambito_intervento?: AmbitorIntervento; // Solo per modulo RADIOLOGICA
  tipologia: string;                   // Dipende da ambito (vedi lookup)

  // Dati tecnici identificativi
  modello: string;
  costruttore: string;
  matricola: string;
  serial_number?: string;

  // Collocazione fisica
  sito_id: string;                     // FK → Sito
  immobile_id: string;                 // FK → Immobile
  piano_id: string;                    // FK → Piano
  locale_id: string;                   // FK → Locale
  reparto_id: string;                  // FK → Reparto
  caposala?: string;

  // Ciclo di vita
  stato: StatoApparecchiatura;
  data_accettazione?: Date;
  data_cessazione?: Date;

  // Flags tecnici
  lan_collegata: boolean;
  medsquare_installato: boolean;

  // Collegamento bene aziendale
  sap_id?: string;
  siap_descrizione?: string;

  // Timestamps
  created_at: Date;
  updated_at: Date;
  created_by: string;                  // FK → Utente
}

type AmbitorIntervento =
  | 'RADIOLOGIA'
  | 'RADIOLOGIA_INTERVENTISTICA'
  | 'MEDICINA_NUCLEARE'
  | 'RADIOTERAPIA';

type StatoApparecchiatura =
  | 'IN_INSTALLAZIONE'
  | 'ATTIVA'
  | 'IN_MANUTENZIONE'
  | 'CESSATA';
```

---

## 7.3 Parametri Radiologici (solo modulo RADIOLOGICA)

```typescript
interface ParametriRadiologici {
  apparecchiatura_id: string;
  corrente_max_ma: number;
  tensione_max_kvolt: number;
  energia_max_kev?: number;            // Trigger adempimenti aggiuntivi se > soglia
  intensita_campo_tesla?: number;       // Solo RM
  tipo_magnete?: 'SUPERCONDUTTORE' | 'PERMANENTE' | 'RESISTIVO'; // Solo RM
}
```

---

## 7.4 Figure Responsabili

```typescript
interface FiguraResponsabile {
  id: string;
  apparecchiatura_id: string;
  ruolo: RuoloResponsabile;
  nome: string;
  cognome: string;
  email: string;
  telefono?: string;
  grado_abilitazione?: '1' | '2' | '3'; // Solo per EdR
  valido_dal: Date;
  valido_al?: Date;                     // Storico sostituzioni
}

type RuoloResponsabile =
  | 'RIR'    // Responsabile Impianto Radiologico
  | 'EFM'    // Esperto in Fisica Medica
  | 'EdR'    // Esperto di Radioprotezione
  | 'MA'     // Medico Autorizzato
  | 'ES'     // Esperto Sicurezza (RM)
  | 'MR';    // Medico Responsabile (RM)
```

---

## 7.5 Riferimenti Gestionali (Assistenza)

```typescript
interface RiferimentoGestionale {
  id: string;
  apparecchiatura_id: string;
  tipo: 'MANUTENZIONE' | 'ASSISTENZA' | 'REPARTO' | 'ALTRO';
  nome_azienda?: string;
  nome_referente?: string;
  telefono?: string;
  email?: string;
  note?: string;
}
```

---

## 7.6 Adempimenti INAIL e STRIMS

```typescript
interface AdempimentoINAIL {
  apparecchiatura_id: string;
  stato: 'DA_REGISTRARE' | 'REGISTRATO' | 'NON_APPLICABILE';
  data_registrazione?: Date;
  ricevuta_file_id?: string;           // FK → FileAllegato
}

interface AdempimentoSTRIMS {
  apparecchiatura_id: string;
  stato: 'DA_REGISTRARE' | 'REGISTRATO' | 'NON_APPLICABILE';
  data_registrazione?: Date;
  ricevuta_file_id?: string;
  notifica_pratica_inviata: boolean;
  data_notifica_pratica?: Date;
  notifica_cessazione_inviata: boolean;
  data_notifica_cessazione?: Date;
}
```

---

## 7.7 Documenti Allegati

```typescript
interface FileAllegato {
  id: string;
  tenant_id: string;
  nome_originale: string;
  nome_storage: string;                // Nome nel bucket S3
  url_signed?: string;                 // URL presignato temporaneo
  mime_type: string;
  dimensione_bytes: number;
  categoria: CategoriaFile;
  apparecchiatura_id?: string;
  verifica_id?: string;
  verbale_id?: string;
  paziente_id?: string;
  descrizione?: string;
  versione: number;                    // 1, 2, 3... (versioning)
  sostituisce_file_id?: string;        // FK → versione precedente
  uploaded_at: Date;
  uploaded_by: string;                 // FK → Utente
}

type CategoriaFile =
  | 'FOTO_APPARECCHIATURA'
  | 'PLANIMETRIA'
  | 'MANUALE_USO'
  | 'MANUALE_QUALITA'
  | 'PROTOCOLLO_CQ'
  | 'REPORT_CQ'
  | 'FILE_LAVORO_CQ'
  | 'NOTIFICA_PRATICA'
  | 'NULLA_OSTA'
  | 'RICEVUTA_INAIL'
  | 'RICEVUTA_STRIMS'
  | 'VERBALE_SOPRALLUOGO'
  | 'LDR'
  | 'CERTIFICATO_CALIBRAZIONE'
  | 'RELAZIONE_TECNICA'
  | 'CONSENSO_INFORMATO'
  | 'REPORT_DOSIMETRICO'
  | 'REFERTO_EMATOLOGICO'
  | 'DOCUMENTO_SMALTIMENTO'
  | 'ALTRO';
```

---

## 7.8 Protocolli di Verifica

```typescript
interface ProtocolloVerifica {
  id: string;
  tenant_id: string;
  codice: string;
  descrizione: string;
  tipo: TipoProtocollo;
  ambito_applicabilita: AmbitorIntervento[];
  tipologie_applicabilita: string[];   // Tipologie di apparecchiatura
  revisione: string;                   // Es. "Rev. 3"
  data_entrata_vigore: Date;
  documento_protocollo_id: string;     // FK → FileAllegato
  periodicita_mesi?: number;           // Per verifiche periodiche
  created_by: string;
  created_at: Date;
}

type TipoProtocollo =
  | 'ACCETTAZIONE'
  | 'PERIODICO'
  | 'POST_MANUTENZIONE'
  | 'LDR'
  | 'PRIMA_VERIFICA_EDR'
  | 'SORVEGLIANZA_PERIODICA_EDR';
```

---

## 7.9 Record di Verifica (Controllo di Qualità)

```typescript
interface RecordVerifica {
  id: string;
  apparecchiatura_id: string;
  protocollo_id: string;               // FK → ProtocolloVerifica
  tipo: TipoProtocollo;
  data_inizio: Date;
  data_fine?: Date;
  eseguito_da: string;                 // FK → Utente (SFM o EdR)
  esito: EsitoVerifica;
  note?: string;

  // Documenti allegati al record
  file_lavoro_id?: string;             // Foglio Excel di lavoro
  report_id?: string;                  // Report finale

  // Per accettazione: benestare
  benestare_qualita_tecnica_data?: Date;
  benestare_qualita_tecnica_by?: string;
  benestare_clinico_data?: Date;
  benestare_clinico_by?: string;

  // Per verifiche periodiche
  semestre?: string;                   // Es. "1S-2024" (solo RM)
  anno?: number;

  // Guasti correlati
  info_guasto?: string;
  tipo_guasto?: string;

  created_at: Date;
  created_by: string;
}

type EsitoVerifica =
  | 'SUPERATO'
  | 'NON_SUPERATO'
  | 'CON_RISERVA'
  | 'IN_CORSO';
```

---

## 7.10 Lookup Tables (dati di sistema)

### Tipologie apparecchiatura per ambito

```
RADIOLOGIA:
  - MAMMOGRAFO
  - ENDORALE
  - MOC
  - PORTATILE
  - TAC
  - PENSILE
  - TELECOMANDATO
  - ORTOPANTOMOGRAFO
  - ALTRO

RADIOLOGIA_INTERVENTISTICA:
  - ANGIOGRAFO
  - ARCO_A_C
  - CBCT
  - LITOTRITORE
  - TAC_INTERVENTISTICA
  - ALTRO

MEDICINA_NUCLEARE:
  - PET_CT
  - SPECT
  - SPECT_CT
  - D_SPECT
  - GAMMACAMERA_PICCOLO_CAMPO
  - GAMMACAMERA_GRANDE_CAMPO
  - ALTRO

RADIOTERAPIA:
  - ACCELERATORE_LINEARE
  - TOMOTERAPIA
  - CYBERKNIFE
  - GAMMAPOD
  - YORT
  - ALTRO

RM (modulo 2):
  - RM_15T
  - RM_3T
  - RM_7T
  - RM_APERTO
  - ALTRO
```

---

## 7.11 Collocazione Fisica — Struttura Gerarchica

```typescript
interface Sito {
  id: string;
  tenant_id: string;
  nome: string;                        // Es. "Complesso Via Pansini"
  indirizzo: string;
}

interface Immobile {
  id: string;
  sito_id: string;
  nome: string;                        // Es. "Edificio 20"
}

interface Piano {
  id: string;
  immobile_id: string;
  nome: string;                        // Es. "Piano 2", "Seminterrato"
  numero: number;
}

interface Locale {
  id: string;
  piano_id: string;
  nome: string;                        // Es. "Sala TAC - 204"
  codice?: string;
}

interface Reparto {
  id: string;
  tenant_id: string;
  nome: string;
  responsabile?: string;
  email?: string;
}
```

---

## 7.12 Regole di Business e Validazioni

| Regola | Descrizione |
|--------|-------------|
| `BIZ-001` | Il `codice` apparecchiatura deve essere univoco per `tenant_id` |
| `BIZ-002` | La `tipologia` deve essere coerente con `ambito_intervento` (validazione lookup) |
| `BIZ-003` | Un'apparecchiatura `CESSATA` non può passare in `ATTIVA` senza nuovo processo di accettazione |
| `BIZ-004` | Se `energia_max_kev > [soglia_da_definire]` devono essere obbligatori adempimenti aggiuntivi |
| `BIZ-005` | La `data_cessazione` deve essere successiva alla `data_accettazione` |
| `BIZ-006` | Ogni apparecchiatura ATTIVA deve avere almeno un `RIR` e un `EFM` assegnati |
| `BIZ-007` | I file caricati non possono essere cancellati, solo sostituiti con nuova versione (audit trail immutabile) |
| `BIZ-008` | Ogni record di verifica con esito `NON_SUPERATO` deve essere gestito (azione correttiva richiesta) |
| `BIZ-009` | Il `tenant_id` non è mai modificabile dopo la creazione di un record |
| `BIZ-010` | Gli utenti vedono solo apparecchiature del proprio `tenant_id` (Row Level Security) |