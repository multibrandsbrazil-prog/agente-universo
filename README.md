# Agente Universo

> Agente universal self-hosted inspirado no Hermes Agent (Nous Research).
> Status: **planejamento** — zero instalação executada.

---

## 📏 TL;DR

Construir um agente open-source que agregue **5.700+ skills** de múltiplas fontes (Hermes, OpenClaw, DSH), suporte **multi-LLM** (DeepSeek, Claude, GPT, Ollama), rode em **VPS ou PC, qualquer SO**, sob **MIT/grátis/zero lock-in**. Inspiração arquitetural vem do Hermes (session log durável, subagentes, skills procedurais, self-evolution).

**Diferencial único:** nenhum agregador de skills existe no mercado. Hermes tem 42, OpenClaw 5.300+, DSH ~300. Nosso seria o primeiro catálogo unificado.

---

## 📂 Estrutura do projeto

```
agente-universo/
├── README.md              ← você está aqui
├── .gitignore             ← arquivos ignorados (criado quando git iniciar)
├── docs/                  ← toda pesquisa + plano
│   ├── PESQUISA-HERMES-CAPACIDADES.md       (16K) — O que Hermes faz + 20 melhorias
│   ├── PESQUISA-REPOSITORIOS-AGENTE-NOVO.md (20K) — 30+ repos validados
│   ├── AUDITORIA-EMPIRICA-REPOS.md          (14K) — 18 repos auditados (3 farsas)
│   ├── LACUNAS-FECHADAS.md                  (14K) — Ruflo + SKILL.md + integrações
│   ├── PLANO-FINAL-CONSOLIDADO.md           (21K) — v1 (com 14 problemas)
│   └── PLANO-FINAL-v2-CORRIGIDO.md          (25K) — v2 ✅ PLANO ATUAL
└── scripts/               ← (vazio — scripts vão aqui quando começar Fase 1)
```

**Arquivo executivo:** [`docs/PLANO-FINAL-v2-CORRIGIDO.md`](docs/PLANO-FINAL-v2-CORRIGIDO.md) — leia este pra entender o que vai ser feito.

**Arquitetura de referência (Hermes, 19 partes):** [`docs/ARQUITETURA-HERMES.md`](docs/ARQUITETURA-HERMES.md) — inventário exaustivo da arquitetura real do Hermes Agent (não README de marketing — código real), pra usar como base de comparação durante a implementação.

**🗂️ Documento único consolidado:** [`docs/DOCUMENTO-CONSOLIDADO.md`](docs/DOCUMENTO-CONSOLIDADO.md) — todos os 11 documentos do projeto concatenados em ordem lógica (196KB, ~4.300 linhas). Pra quem quer ler tudo num arquivo só.

---

## 🎯 Stack final (validado empiricamente)

| Camada | Componente | Validação |
|---|---|---|
| 🎯 Core | DeepSeek Harness (`dsh` v0.1.2-alpha.3) | ✅ 206k⭐, MIT |
| 🧠 Multi-LLM | DSH providers nativos | ✅ Built-in |
| 💾 Memory | `volcengine/OpenViking` + plugin DSH oficial | ✅ 34.690⭐, AGPL-3.0, plugin DSH mantido pelo próprio time |
| 🛡️ Sandbox | `BitMiracle-AI/Dormice` | ✅ Self-hosted |
| 🛡️ Prompt defense | In-house (~200 linhas Python) | Ruflo aidefence como ref |
| 📊 Observability | `Yuntwo/dsh-langfuse-plugin` | ⚠️ Auditar antes |
| 📊 OTel | `traceloop/openllemetry` | ✅ 7.4k⭐ |
| 🧬 Self-evolution | DSPy + GEPA | ✅ Stanford + SOTA |
| 🔌 MCP BR | Suas12 MCPs locais | ✅ Funcionam via DSH bridge |
| 📚 Skills | Hermes42 + OpenClaw5.300+ + DSH~300 | Adaptador ~100 linhas |

---

## 🗺️ Roadmap (7 fases, todas requerem aprovação antes de executar)

| Fase | Tempo | Status | O que faz |
|---|---|---|---|
| 0 | 30min | ⏸️ | Verificar pré-requisitos (node, pnpm, gh) |
| 1 | 1-2h | ⏸️ | Subir DSH core + validar UI |
| 2 | 3-4h | ⏸️ | Skills Bridge + 42 skills Hermes |
| 3 | 1-2 dias | ⏸️ | OpenClaw skills via clawhub |
| 4 | 3-4h | ⏸️ | MCP BR + Langfuse observability |
| 5 | 4-6h | ⏸️ | Sandbox Dormice + Memory OpenViking (server Python self-hosted porta 1933) |
| 6 | 2-3 dias | ⏸️ | Self-evolution DSPy+GEPA |
| 7 | 1 dia | ⏸️ | Benchmark vs Hermes + publicar |

**Total MVP:** ~7-10 dias úteis.

---

## 📚 Fontes consultadas (sem clones, só APIs)

- `gh api` — metadata, files count, contributors, releases, issues, CI workflows, LICENSE
- `npm registry` — versão publicada + repo link
- `raw.githubusercontent.com` — README + SKILL.md + package.json + AGENTS.md
- `awesome-list` curadas (VoltAgent, 0xNyk, awesome-dsh-plugin)
- Repos oficiais lidos: `~/.hermes/SOUL.md`, `~/.hermes/hermes-agent/AGENTS.md`, `~/.hermes/config.yaml`

---

## ⚠️ Princípios do projeto

1. **Zero instalação sem aprovação explícita** — toda fase tem comando + entregável + teste antes
2. **Validar empíricamente, não por stars** — 3 repos foram rejeitados (BridgeWard, e2b fragments, ink-ui stale) mesmo com stars razoáveis
3. **Curar antes de agregar** — OpenClaw awesome-list filtrou 7.215 itens, usar SÓ a curada
4. **Pinar versões** — DSH é alpha, pinar tag exata
5. **Auditar plugins 3rd-party** — plugins DSH da comunidade têm ⭐0-25, ler código antes. Exceção: plugins oficiais mantidos pelo próprio time do projeto (ex: `@openviking/openclaw-plugin`) já passam por curadoria upstream.

---

## 📞 Comandos úteis

```bash
# Navegar pro projeto
cd ~/projetos/agente-universo

# Ver plano atual
cat docs/PLANO-FINAL-v2-CORRIGIDO.md

# Quando quiser começar Fase 0:
# (nada — só verificar ambiente, sem instalar)
```

---

## 🚦 Próximo passo

**Plano finalizado.** Aguardando:
1. Aprovação do plano v2
2. Definição de onde publicar (seu GitHub / org compartilhado)
3. Decisão: você roda ou eu rodo cada fase

**Nada será instalado até você dar OK explícito na Fase 0.**