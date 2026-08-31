# Prompt de Handoff — Projeto agente-universo

> Use este texto no início de uma **nova sessão** com qualquer agente AI pra retomar o projeto sem perder contexto.
>
> **Data de criação:** 2026-08-31
> **Última atualização:** 2026-08-31 (Fase 0 concluída)

---

## 📋 VERSÃO LONGA (copie/cole inteira se for a 1ª msg da nova sessão)

```
Olá! Estou continuando um projeto chamado **agente-universo**.

## O que estou tentando criar

Um agente de IA universal, open-source (MIT), self-hosted (VPS ou PC, qualquer SO),
inspirado no Hermes Agent da Nous Research. Diferenciais:

1. Agrega 5.700+ skills de múltiplas fontes (Hermes 42 + OpenClaw 5.300+ + DSH ~300)
2. Roda em qualquer LLM que o usuário quiser (DeepSeek, Claude, GPT, Ollama local)
3. Self-hosted, zero custo, zero lock-in
4. Tem 12 MCPs BR já integrados (mercado_livre, fiscal, comexstat, etc)
5. Único agregador universal de skills no mercado (ninguém mais faz isso)

## Onde está tudo

- **Projeto local:**  ~/projetos/agente-universo/
- **Repo GitHub:**  https://github.com/multibrandsbrazil-prog/agente-universo (público)
- **Branch:**       main (2 commits: c2e3a13 inicial + 36aed6f cleanup)
- **Plano atual:**  ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md

## O que já foi feito

✅ Pesquisa completa do Hermes Agent (capacidades, 20 melhorias possíveis)
✅ Pesquisa de 30+ repositórios no GitHub (DSH, LiteLLM, mem0, Langfuse, DSPy, GEPA, Ruflo, etc)
✅ Auditoria empírica de 18 repos (3 farsas detectadas: BridgeWard, e2b fragments, ink-ui stale)
✅ Lacunas críticas resolvidas (Ruflo vs nossa ideia, compat SKILL.md, integrações DSH)
✅ Plano v1 + auditoria crítica + plano v2 corrigido (25KB, 7 fases, 23 mitigações de compat)
✅ Repositório git criado + 2 commits + push pro GitHub
✅ Auditoria de segurança (sem credenciais vazadas, paths sensíveis substituídos por $HOME)
✅ Fase 0 do plano concluída (verificações de ambiente OK)

## O que NÃO foi feito (ainda)

❌ Fase 1: Subir DSH core (git clone + pnpm install + dsh web em :3080)
❌ Fase 2: Skills Bridge + importar 42 skills Hermes locais
❌ Fase 3: OpenClaw skills via clawhub (50 top)
❌ Fase 4: MCP BR + Langfuse observability
❌ Fase 5: Sandbox Dormice + Memory mem0
❌ Fase 6: Self-evolution DSPy+GEPA
❌ Fase 7: Benchmark vs Hermes + publicar

## Regras de ouro (do Renan, suas preferências)

1. **NÃO instalar nada sem aprovação explícita** — toda fase tem comando + entregável + teste
2. **Validar empíricamente, não por hype/stars** — auditar código antes de usar plugin 3rd-party
3. **Pinar versões** — DSH é alpha 0.1.2, pinar tag exata
4. **Curar antes de agregar** — OpenClaw awesome-list filtrou 7.215 itens, usar SÓ curada
5. **Auditar plugins antes de ativar** — plugins com <100⭐ precisam code review
6. **Não mexer em ~/importaai/** sem permissão explícita
7. **Privacy:** não expor /home/openclaw em docs públicos (usar $HOME)
8. **Nunca usar `hermes mcp run` (não existe)** — MCP servers vão no config.yaml
9. **DSH plugins usam `clawhub install <slug>` (não `git clone`)**

## Pendências de decisão (me perguntar antes de Fase 1)

- Quem vai manter o repo? (seu GitHub pessoal / org compartilhada)
- Quer rodar Fase 1 agora? (eu rodo e mostro cada comando + saída)
- Quer ler o plano v2 inteiro antes? (25KB, ~30min de leitura)

## Comandos úteis

cd ~/projetos/agente-universo         # navegar pro projeto
cat docs/PLANO-FINAL-v2-CORRIGIDO.md  # plano atual
gh repo view agente-universo          # ver repo remoto
git log --oneline                     # histórico

## Estado atual do ambiente (verificado na Fase 0)

- Node v22.22.2, pnpm v10.33.0 (upgrade pra 11.7+ na Fase 1)
- Python 3.11.15, Docker 29.4.1, Ollama instalado
- GH CLI logado em github.com/multibrandsbrazil-prog
- 65GB disco livre, sem GPU NVIDIA (usar API pra LLM)
- OS: Ubuntu 24.04.4 LTS, x86_64
```

---

## ⚡ VERSÃO CURTA (use se a sessão já é continuidade e você só quer lembrar o agente)

```
Continuando projeto agente-universo.
- Repo: github.com/multibrandsbrazil-prog/agente-universo (público, MIT, 2 commits)
- Local: ~/projetos/agente-universo/
- Plano: ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md (7 fases)
- Inspiração: Hermes Agent (Nous Research)
- Base: DeepSeek Harness (DSH) — alpha 0.1.2
- Diferencial: agrega 5.700+ skills de Hermes+OpenClaw+DSH (nenhum agregador existe)
- Status: Fase 0 ✅ (verificações). Fase 1 ⏸️ (DSH core ainda não foi clonado/instalado)
- Regra de ouro: NÃO instalar nada sem aprovação. Validar antes de commitar.
- Pendência: quem mantém o repo? seu GitHub ou org compartilhada?
```

---

## 📌 Versão "TL;DR" (1 linha pra contexto rápido)

```
Projeto agente-universo (github.com/multibrandsbrazil-prog/agente-universo): agente MIT self-hosted
inspirado no Hermes, base DSH, agregando 5.700+ skills. Fase 0 concluída, Fase 1 (subir DSH core) não
iniciada ainda. Plano: ~/projetos/agente-universo/docs/PLANO-FINAL-v2-CORRIGIDO.md
```

---

## 🔄 Quando atualizar este prompt

- Quando **concluir uma fase** do plano (atualizar "O que foi feito" e "O que falta")
- Quando **mudar o ambiente** (Node version, GPU, etc)
- Quando **trocar de conta/repo** GitHub
- Quando **descobrir uma regra nova** do Renan

Salvar como `~/projetos/agente-universo/PROMPT-HANDOFF.md` e versionar no git.