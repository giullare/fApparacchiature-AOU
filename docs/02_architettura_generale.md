# 02 — Architettura Generale e Stack Tecnologico

## 2.1 Panoramica architetturale

La piattaforma adotta un'architettura **web multi-tier** con separazione netta tra frontend, backend e layer di persistenza. L'approccio è **multi-tenant** (multi-azienda), con isolamento dei dati per ogni ente.

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│   Browser Web (PC/Tablet)        App Mobile (iOS / Android)      │
└────────────────────────┬─────────────────────────────────────────┘
                         │ HTTPS / REST / WebSocket
┌────────────────────────▼─────────────────────────────────────────┐
│                      API GATEWAY / LOAD BALANCER                 │
│   Autenticazione JWT · Rate limiting · SSL Termination           │
└────────────────────────┬─────────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────────┐
│                     APPLICATION LAYER                            │
│                                                                  │
│  ┌──────────────┐  ┌────────────────┐  ┌──────────────────────┐ │
│  │  Modulo 1    │  │   Modulo 2     │  │      Modulo 3        │ │
│  │ Apparecch.   │  │  Risonanza     │  │  Registri MN         │ │
│  │ Radiologiche │  │  Magnetica     │  │  Sorveglianza        │ │
│  └──────────────┘  └────────────────┘  └──────────────────────┘ │
│                                                                  │
│  ┌──────────────┐  ┌────────────────┐  ┌──────────────────────┐ │
│  │  Modulo 4    │  │  Documentale   │  │   Reportistica       │ │
│  │  RT Lu177    │  │  & Archivio    │  │   & Statistiche      │ │
│  └──────────────┘  └────────────────┘  └──────────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           SERVIZI TRASVERSALI                             │   │
│  │  Auth/RBAC · Notifiche · Audit Log · File Storage        │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────┬─────────────────────────────────────────┘
                         │
┌────────────────────────▼─────────────────────────────────────────┐
│                       DATA LAYER                                 │
│   Database Relazionale (PostgreSQL)                              │
│   File Storage (S3-compatible)                                   │
│   Cache (Redis)                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Moduli funzionali principali

| # | Modulo | Normativa | Descrizione |
|---|--------|-----------|-------------|
| 1 | **Gestione Grandi Apparecchiature Radiologiche** | D.Lgs. 101/20 Titoli XI e XIII | Radiologia, Interventistica, Radioterapia, Medicina Nucleare |
| 2 | **Gestione Apparecchiature Risonanza Magnetica** | DM 01/21 | Specifico per RM: sicurezza, CAI, verbali periodici |
| 3 | **Registri di Radioprotezione in Medicina Nucleare** | D.Lgs. 101/20 | Dosimetria ambientale, contaminazione, radiofarmaci, rifiuti |
| 4 | **Gestione Pazienti in Terapia Metabolica** | D.Lgs. 101/20 | Specifico per trattamenti con Lu177 |
| T | **Servizi Trasversali** | — | Utenti, documenti, statistiche, notifiche, audit |

---

## 2.3 Stack tecnologico consigliato

### Frontend
```
Framework:      React 18+ con TypeScript
State mgmt:     Zustand o Redux Toolkit
UI Components:  shadcn/ui + Tailwind CSS
Routing:        React Router v6
Charts:         Recharts o Chart.js
PDF viewer:     react-pdf
Forms:          React Hook Form + Zod (validazione)
Date handling:  date-fns
HTTP client:    Axios o Fetch API nativa
```

### Backend
```
Runtime:        Node.js 20 LTS (o alternativa: Python/FastAPI)
Framework:      Express.js o NestJS (consigliato per struttura)
ORM:            Prisma o TypeORM
Auth:           JWT + refresh token, bcrypt per password
File upload:    Multer + integrazione S3
API docs:       Swagger/OpenAPI 3.0
Validazione:    Zod o class-validator
Job scheduler:  node-cron (per notifiche scadenze)
```

### Database e storage
```
Database:       PostgreSQL 15+ (relazionale, ACID)
Caching:        Redis (sessioni, rate limiting)
File storage:   MinIO (self-hosted S3-compatible) o AWS S3
Backup:         Scheduled dumps + point-in-time recovery
```

### Infrastruttura
```
Containerizzazione:   Docker + Docker Compose
Orchestrazione:       Kubernetes (produzione) o Docker Compose (sviluppo)
Reverse proxy:        Nginx o Traefik
CI/CD:                GitHub Actions o GitLab CI
Monitoring:           Prometheus + Grafana
Logging:              ELK Stack o Loki + Grafana
```

---

## 2.4 Pattern multi-tenant

Ogni **azienda sanitaria** (tenant) è isolata con:

```
Strategia consigliata: Row-Level Security (RLS) su PostgreSQL

Ogni tabella principale include:
  - tenant_id: UUID NOT NULL → FK a tabella "organizations"
  - Tutte le query includono WHERE tenant_id = :currentTenantId
  - Policies RLS di PostgreSQL come secondo livello di sicurezza

Vantaggi:
  ✓ Un unico schema di database
  ✓ Manutenzione semplificata
  ✓ Isolamento garantito a livello DB
  ✓ Backup per tenant possibile con pg_dump --table filtrato
```

---

## 2.5 Struttura del progetto (monorepo consigliato)

```
radioprotezione-platform/
├── apps/
│   ├── web/                    # Frontend React
│   │   ├── src/
│   │   │   ├── modules/
│   │   │   │   ├── apparecchiature/
│   │   │   │   ├── risonanza-magnetica/
│   │   │   │   ├── medicina-nucleare/
│   │   │   │   ├── radioterapia-lu177/
│   │   │   │   ├── documentale/
│   │   │   │   ├── reportistica/
│   │   │   │   └── admin/
│   │   │   ├── shared/
│   │   │   │   ├── components/
│   │   │   │   ├── hooks/
│   │   │   │   ├── utils/
│   │   │   │   └── types/
│   │   │   └── App.tsx
│   │   └── package.json
│   │
│   └── api/                    # Backend Node.js/NestJS
│       ├── src/
│       │   ├── modules/
│       │   │   ├── apparecchiature/
│       │   │   ├── risonanza-magnetica/
│       │   │   ├── medicina-nucleare/
│       │   │   ├── radioterapia/
│       │   │   ├── documentale/
│       │   │   ├── reportistica/
│       │   │   └── auth/
│       │   ├── common/
│       │   │   ├── guards/
│       │   │   ├── interceptors/
│       │   │   ├── decorators/
│       │   │   └── dto/
│       │   └── main.ts
│       └── package.json
│
├── packages/
│   ├── shared-types/           # Tipi TypeScript condivisi
│   ├── ui-components/          # Design system condiviso
│   └── validation-schemas/     # Schemi Zod condivisi
│
├── database/
│   ├── migrations/             # Migrazioni Prisma/Flyway
│   ├── seeds/                  # Dati iniziali (norme, tipologie, ecc.)
│   └── schema.prisma
│
├── infrastructure/
│   ├── docker-compose.yml
│   ├── docker-compose.prod.yml
│   └── nginx.conf
│
└── docs/                       # ← Questa cartella di specifiche
    ├── 00_README.md
    ├── 01_vision_e_obiettivi.md
    └── ...
```

---

## 2.6 Requisiti di sincronizzazione real-time

Per garantire la sincronizzazione in tempo reale tra sessioni diverse:

```
Tecnologia:     WebSocket (Socket.io o ws nativo)
Scenari:
  - Aggiornamento stato controllo di qualità
  - Notifica scadenza imminente (CQ, LDR, verbali)
  - Aggiornamento inventario apparecchiature
  - Completamento upload documento

Fallback:       Long polling per ambienti con firewall restrittivi
```

---

## 2.7 Integrazione MS Office

Il sistema deve supportare import e archiviazione dei seguenti formati:

| Formato | Utilizzo |
|---------|---------|
| `.xlsx`, `.xls` | File di lavoro CQ, fogli dosimetria, registri |
| `.docx`, `.doc` | Protocolli, notifiche, verbali |
| `.pdf` | Documenti autorizzativi, report firmati, manuali |
| `.jpg`, `.png`, `.tiff` | Fotografie apparecchiature, planimetrie |
| `.zip` | Pacchetti documentali |

**Funzionalità richieste:**
- Upload con anteprima per PDF e immagini
- Collegamento documento a specifica apparecchiatura / verifica / verbale
- Ricerca full-text nei metadati dei documenti
- Versioning: sostituzione documento con mantenimento storico versioni precedenti
- Download originale e/o visualizzazione in-app