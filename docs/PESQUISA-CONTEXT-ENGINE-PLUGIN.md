# Pesquisa: Plugin de Context Engine pro agente-universo

**Data:** 31/08/2026
**Objetivo:** Achar plugin de `contextEngine` (slot do OpenClaw) compatível com o nosso core DSH, que faça **busca semântica/relevância no histórico de conversa** antes de montar o request pro LLM.

**Decisão final:** **OpenViking** (`volcengine/OpenViking` + `@openviking/openclaw-plugin` oficial) — ver tabela comparativa no final.

---

## 🎯 O que precisamos

| Camada | Comportamento desejado |
|---|---|
| **System prompt** | Igual ao Hermes — MEMORY + USER + tools + skills, sempre inteiro, nunca mexe |
| **Histórico de conversa** | **Filtrado por relevância** antes de ir pro LLM (não manda tudo) |
| **Como filtra** | Busca semântica (BM25 + embedding) com scoring de saliência |
| **Fallback** | Se filtro falhar → últimos N turnos literais (igual Hermes) |

---

## ✅ Plugin escolhido: OpenViking

**Repo:** https://github.com/volcengine/OpenViking
**Plugin DSH:** https://github.com/volcengine/OpenViking/tree/main/examples/dsh-memory-plugin
**npm plugin:** `@openviking/openclaw-plugin@2026.8.31` (publicado HOJE)
**License:** AGPL-3.0
**Stars:** 34.688 (vs LUCID: 5, Mem0 plugin: ~100)
**Commits últimos 90d:** push diário (vs LUCID: 5 total)
**Autores:** 30 contributors (vs LUCID: 1)
**CI workflows:** 26 (vs LUCID: 0)

### Como funciona (do README oficial)

A cada turno:

1. **`afterTurn`** — mensagens novas vão pra sessão OpenViking
2. **`memory_store`** — fatos importantes persistidos imediatamente
3. **`/compact`** — mensagens viram memória de longo prazo
4. **`assemble`** — **antes de cada resposta**: memórias relevantes auto-injetadas no context

### Tools que o agente ganha

`memory_recall` · `memory_store` · `memory_forget` · `ov_search` · `ov_read` · `ov_recall_trace` (debug!) + 10 outros.

---

## ❌ Alternativas descartadas

### LUCID Context Engine (Spaztazim/lucid-context-engine)

5 commits, 1 autor, 0 issues, `ingest()` é no-op, depende de QMD externo. README parecia bom, código contou outra história. Análise profunda em `ANALISE-PROFUNDA-LUCID.md`.

### Mem0 (`runfali/dsh-mem0-plugins`)

~100⭐, baixa adoção, sem release estável. Foi o plano original mas auditoria empírica (curadoria multi-sinal) mostrou que OpenViking é estritamente superior em todos os critérios.

---

## 📌 Decisão arquitetural pro agente-universo

Adotar o **slot `contextEngine` do OpenClaw** com plugin oficial OpenViking:

1. **Core do agente-universo** implementa os 4 hooks do slot (herdados do OpenClaw)
2. **Engine** = `@openviking/openclaw-plugin` (oficial, mantido pelo time OpenViking)
3. **System prompt** fica intocado — sempre MEMORY + USER + tools + skills
4. **OpenViking server** roda self-hosted na VPS (Python ≥3.10, porta 1933)

---

## 📊 Comparativo final (auditoria empírica via `gh api`)

| Critério | LUCID | Mem0 plugin | OpenViking |
|---|---|---|---|
| Stars | 5 | ~100 | **34.688** |
| Commits últimos 90d | 5 total | irregular | **push diário** |
| Autores | 1 | 1-2 | **30 contributors** |
| Releases | 0 | variável | **30 releases, latest HOJE** |
| CI workflows | 0 | variável | **26 workflows** |
| Topic DSH oficial | ❌ | ❌ | ✅ `dsh-plugin` |
| Plugin DSH oficial | ❌ | ⚠️ clone manual | ✅ `clawhub install` |
| Persiste cross-session | ❌ (`ingest()` no-op) | ✅ | ✅ |
| Testes | 0 | parcial | ✅ |

---

## 🔗 Fontes consultadas

| # | Fonte | URL |
|---|---|---|
| 1 | OpenViking repo | https://github.com/volcengine/OpenViking |
| 2 | OpenViking plugin DSH | https://github.com/volcengine/OpenViking/tree/main/examples/dsh-memory-plugin |
| 3 | OpenClaw ContextEngine docs | https://docs.openclaw.ai/concepts/context-engine |
| 4 | OpenClaw OpenViking plugin | https://github.com/volcengine/OpenViking/tree/main/examples/openclaw-plugin |
| 5 | npm `@openviking/openclaw-plugin` | https://registry.npmjs.org/@openviking/openclaw-plugin |
| 6 | Análise profunda LUCID (rejeitado) | `docs/ANALISE-PROFUNDA-LUCID.md` |
| 7 | Pesquisa inicial | `docs/PESQUISA-CONTEXT-ENGINE-PLUGIN.md` (versão anterior) |

---

## 📋 Próximos passos

1. [x] Patchear `PLANO-FINAL-v2-CORRIGIDO.md` substituindo Fase 4 (Mem0 ⭐1) por OpenViking
2. [x] Atualizar README com arquitetura de memória
3. [x] Commit + push
4. [ ] Fase 4 instalação real (quando aprovado)
