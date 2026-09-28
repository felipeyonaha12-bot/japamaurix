---
title: "Diagrama de Migração AI_CORE — AS-IS → TARGET"
type: overview
domain: ai_core
date: 2026-09-28
status: proposed
source:
  - "[[AI_CORE_LLM_WIKI_MESTRE]]"
---

# Diagrama de Migração AI_CORE: AS-IS → TARGET

Legenda: 🟩 mantém · 🟨 move/renomeia · 🟥 sai do núcleo ou é absorvido · 🟦 novo

```mermaid
flowchart LR
  subgraph ASIS["AS-IS (hoje)"]
    direction TB
    A1["01_KNOWLEDGE/OBSIDIAN_VAULT/"]
    A1a["00_HOME · 01_HUMAN_CORE · 02_LLM_WIKI"]
    A1b["OBSIDIAN_VAULT/03_RAW"]
    A1c["09_SYSTEMS · 98_ATTACHMENTS"]
    A2["02_MEMORIA_E_GRAFOS/OMEGA (5 categorias)"]
    A3["03_RAW/docs"]
    A3u[".user_uploaded/"]
    A4["03_B3_INVESTIMENTOS"]
    A5["04_IMOBILIARIO (tudo no mesmo nível)"]
    A6["05_OUTPUT"]
    A7["05_SISTEMAS_E_ROTINAS"]
    A8["06_LAB (inclui roadmap)"]
    A9["graphify-out/ (raiz)"]
    A10["Raiz: Selic.md + diagramas .html"]
  end

  subgraph TGT["TARGET (proposto)"]
    direction TB
    T0["🟦 00_CONTROL (ARCHITECTURE, GOVERNANCE, ROUTING, SCHEMAS, MIGRATIONS)"]
    T1["🟨 01_KNOWLEDGE/ (sem nível OBSIDIAN_VAULT)"]
    T1a["🟩 00_HOME · 01_HUMAN_CORE · 02_LLM_WIKI (+ index.md, log.md)"]
    T1b["🟨 03_RAW_CANONICAL (imutável)"]
    T1c["🟨 09_SYSTEMS_KNOWLEDGE · 🟩 98_ATTACHMENTS"]
    T2["🟨 02_MEMORY/OMEGA (7 categorias, EN)"]
    T3["🟨 03_DOMAINS/B3"]
    T4["🟨 03_DOMAINS/IMOBILIARIO (00_CONTROL, 01_TEMPLATES, 02_IMOVEIS, 03_PUBLICACAO, 04_AUTOMACOES)"]
    T5["🟨 04_SYSTEMS"]
    T6["🟨 05_GRAPH/graphify-out"]
    T7["🟩 06_LAB (só experimentos)"]
    T8["🟦 90_INBOX/RAW (transitório)"]
    T9["🟦 99_ARCHIVE"]
    OUT["🟥 AI_OUTPUT/ (FORA do AI_CORE)"]
  end

  A1 --> T1
  A1a --> T1a
  A1b --> T1b
  A1c --> T1c
  A3 --> T8
  A3u --> T8
  T8 -->|ingest| T1b
  A2 --> T2
  A4 --> T3
  A5 --> T4
  A6 --> OUT
  A7 --> T5
  A8 --> T7
  A8 -->|roadmap| T0
  A9 --> T6
  A10 -->|Selic.md| T1a
  A10 -->|diagramas| T0
```

## Resumo do que muda

| Hoje | Vira | Tipo |
|---|---|---|
| Dois RAW + `.user_uploaded` | `90_INBOX/RAW` → `03_RAW_CANONICAL` | 🟥 consolidação |
| `05_OUTPUT/` | `AI_OUTPUT/` fora do núcleo | 🟥 sai |
| `02_MEMORIA_E_GRAFOS/OMEGA` | `02_MEMORY/OMEGA` (+ preferences, sessions) | 🟨 |
| `03_B3_INVESTIMENTOS`, `04_IMOBILIARIO` | `03_DOMAINS/B3`, `03_DOMAINS/IMOBILIARIO` | 🟨 |
| `05_SISTEMAS_E_ROTINAS` | `04_SYSTEMS` | 🟨 |
| `graphify-out/` | `05_GRAPH/graphify-out` | 🟨 |
| — | `00_CONTROL`, `90_INBOX`, `99_ARCHIVE` | 🟦 |
| Selic.md e diagramas na raiz | wiki / `00_CONTROL` | 🟨 |
