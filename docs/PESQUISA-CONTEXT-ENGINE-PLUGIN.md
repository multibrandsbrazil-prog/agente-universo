# Pesquisa: Plugin de Context Engine pro agente-universo

**Data:** 31/08/2026
**Objetivo:** Achar plugin de `contextEngine` (slot do OpenClaw) compatível com o nosso core, que faça **busca semântica/relevância no histórico de conversa** antes de montar o request pro LLM.

---

## 🎯 O que precisamos

| Camada | Comportamento desejado |
|---|---|
| **System prompt** | Igual ao Hermes — MEMORY + USER + tools + skills, sempre inteiro, nunca mexe |
| **Histórico de conversa** | **Filtrado por relevância** antes de ir pro LLM (não manda tudo) |
| **Como filtra** | Busca semântica (BM25 + embedding) com scoring de saliência |
| **Fallback** | Se filtro falhar → últimos N turnos literais (igual Hermes) |

---

## ✅ Plugin escolhido: LUCID Context Engine

**Repo:** https://github.com/Spaztazim/lucid-context-engine
**npm:** `git clone https://github.com/Spaztazim/lucid-context-engine.git ~/.openclaw/extensions/lucid-context-engine`
**Licença:** MIT
**Compatibilidade:** OpenClaw v3.7+ (usa slot `contextEngine` nativo)

### Como funciona (do README oficial)

A cada turno:

1. **Filtra trivial** — pula "ok", "obrigado", heartbeats
2. **Busca híbrida** — BM25 + semântica via QMD
3. **Score por saliência** — `salience = qmd_score × recency × type × collection`
4. **Respeita budget** — só injeta o que cabe no context window restante
5. **Injeta no system prompt** — LLM vê o contexto recalled automaticamente

### Pesos de saliência

| Fator | Peso |
|---|---|
| Recency ≤7d | 1.5× |
| Recency ≤30d | 1.2× |
| Recency ≤90d | 1.0× |
| Recency >90d | 0.8× |
| Type LESSONS.md | 2.0× |
| Type decision | 1.5× |
| Type memory/* | 1.0× |
| Type log | 0.7× |
| Collection memory | 1.5× |
| Collection codex | 1.2× |
| Default | 1.0× |

### Configuração

```json
{
  "plugins": {
    "slots": {
      "contextEngine": "lucid"
    },
    "entries": {
      "lucid-context-engine": {
        "enabled": true,
        "topK": 5,
        "threshold": 0.0
      }
    }
  }
}
```

---

## ❌ Alternativa descartada: lossless-claw

**Repo:** https://github.com/Martian-Engineering/lossless-claw
**Por que não:** Usa DAG de resumos (Lossless Context Management). Preserva TUDO mas resumiu. Não é busca semântica — é compaction inteligente. Mais complexo, mais storage, overkill pro nosso caso.

**Quando usar:** se precisar garantia absoluta de zero perda (auditoria, compliance).

---

## 📌 Decisão arquitetural pro agente-universo

Adotar o **slot `contextEngine` do OpenClaw** como contrato padrão:

1. **Core do agente-universo** implementa os 4 hooks:
   - `ingest()` — entrada de msg nova
   - `assemble()` — escolhe quais msgs mandar (delega pro plugin)
   - `compact()` — fallback se plugin não tiver
   - `after_turn()` — salva estado

2. **Engine padrão** = compaction do Hermes (threshold 0.5, target_ratio 0.2, protect_last_n 20, protect_first_n 3)

3. **Plugin oficial** = LUCID Context Engine (busca semântica + saliência)

4. **System prompt** fica intocado — sempre MEMORY + USER + tools + skills

---

## 🔗 Fontes consultadas

| # | Fonte | URL |
|---|---|---|
| 1 | OpenClaw ContextEngine docs | https://docs.openclaw.ai/concepts/context-engine |
| 2 | OpenClaw Plugins docs | https://docs.openclaw.ai/tools/plugin |
| 3 | LUCID Context Engine README | https://github.com/Spaztazim/lucid-context-engine |
| 4 | lossless-claw README | https://github.com/Martian-Engineering/lossless-claw |
| 5 | LCM paper (Voltropy) | https://papers.voltropy.com/LCM |
| 6 | OpenClaw Plugin Architecture | https://docs.openclaw.ai/plugins/architecture-internals |

---

## 📋 Próximos passos

1. [ ] Patchear `PLANO-FINAL-v2-CORRIGIDO.md` substituindo Fase 4 (Mem0 ⭐1) por:
   - Slot `contextEngine` plugável (4 hooks)
   - Engine padrão = compaction Hermes
   - Plugin LUCID como opção de relevance filter
2. [ ] Atualizar README com arquitetura de memória
3. [ ] Commit + push
