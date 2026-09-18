# Comandos e caminho de uma nova feature

Guia de bolso para usar o schema `sdd-superpowers` no dia a dia: o que digitar, o que cada comando faz, qual skill do Superpowers entra em cada ponto, e o passo a passo de uma feature do zero até o merge.

---

## 1. Setup (uma vez por máquina e por projeto)

### Por máquina

```bash
npm i -g @fission-ai/openspec@latest            # OpenSpec ≥ 1.13
claude plugin install superpowers@claude-plugins-official

# habilita TODOS os workflows /opsx (o perfil "core" só instala 4 e o schema precisa de new/continue/ff/verify)
openspec config set profile custom
openspec config set workflows '["propose","explore","new","continue","apply","ff","sync","archive","bulk-archive","verify","update"]'
```

### Por projeto

```bash
cd meu-projeto
openspec init                                    # cria openspec/ e os comandos /opsx no harness
mkdir -p openspec/schemas
cp -r ~/development/my/sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/
#   (ou symlink relativo enquanto está em desenvolvimento:
#    ln -s ../../../sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/sdd-superpowers)
echo "schema: sdd-superpowers" > openspec/config.yaml
openspec schema validate sdd-superpowers
openspec update --force                          # regenera os comandos /opsx com os workflows habilitados
cat openspec/schemas/sdd-superpowers/templates/adopters/CLAUDE.md.fragment.md >> CLAUDE.md
```

Confira: `openspec schemas` deve listar `sdd-superpowers (project)` e `ls .claude/commands/opsx/` deve mostrar `apply archive bulk-archive continue explore ff new propose sync update verify`.

---

## 2. Comandos `/opsx:*` (dentro do Claude Code)

| Comando | O que faz | Quando usar |
|---|---|---|
| `/opsx:new <nome>` | Cria `openspec/changes/<nome>/` com o schema padrão | Início de qualquer feature |
| `/opsx:continue` | Gera o **próximo** artefato pendente do grafo (brainstorm → proposal → design → specs → tasks → plan → … → retrospective → finalize) | Fluxo passo a passo, revisando cada artefato |
| `/opsx:ff <nome>` | Cria a change e gera **todos** os artefatos de planejamento de uma vez (brainstorm até plan) | Mudança pequena e bem entendida |
| `/opsx:propose <nome>` | Igual ao `ff`, mas com nome/descrição livre | Alternativa ao `ff` |
| `/opsx:explore` | Modo de investigação sem criar artefatos | Antes de decidir se vale abrir uma change |
| `/opsx:apply` | Executa o bloco `apply` do schema: pre-flight → worktree → subagent-driven TDD → `apply.md` | Depois do `plan.md` existir |
| `/opsx:verify` | Roda os 7 checks e escreve `verify.md` | Depois de cada `/opsx:apply` |
| `/opsx:sync` | Copia as delta specs da change para `openspec/specs/` sem arquivar | Raro; o `archive` já faz isso |
| `/opsx:archive` | Sincroniza specs + move a change para `openspec/changes/archive/` | Depois do `finalize.md` |
| `/opsx:bulk-archive` | Arquiva várias changes | Limpeza |
| `/opsx:update` | Atualiza uma change existente (specs/tasks) quando o escopo mudou | Mudança de rumo no meio |

Use sempre o slash command; o skill equivalente (`openspec-continue-change` etc.) é chamado por ele.

### CLI `openspec` (no terminal)

```bash
openspec status --change <nome>        # o que está pronto, o que está bloqueado, qual é o próximo
openspec list                          # changes abertas
openspec list --specs                  # capabilities já especificadas
openspec validate --all                # valida changes + specs (o verify roda isso)
openspec show <nome>                   # mostra uma change ou spec
openspec view                          # dashboard interativo
openspec instructions <artefato> --change <nome>   # vê a instrução que o agente vai receber
```

---

## 3. Skills do Superpowers e onde cada uma entra

| Skill | Quem invoca | Momento |
|---|---|---|
| `superpowers:brainstorming` | artefato `brainstorm` | 1º `/opsx:continue` — conversa guiada, uma pergunta por vez, 2-3 alternativas |
| `superpowers:writing-plans` | artefato `plan` | quebra `tasks.md` em micro-passos TDD com caminhos de arquivo e comandos de teste |
| `superpowers:using-git-worktrees` | `/opsx:apply` passo 1 | worktree isolada por change (no Claude Code usa a ferramenta nativa) |
| `superpowers:subagent-driven-development` | `/opsx:apply` passo 2 | um subagente novo por task; **ativa sozinha** `test-driven-development` e `requesting-code-review` |
| `superpowers:test-driven-development` | transitiva | RED → GREEN → REFACTOR em toda task; código sem teste falhando antes é apagado |
| `superpowers:requesting-code-review` | transitiva | revisão por subagente após cada task + revisão final do branch |
| `superpowers:finishing-a-development-branch` | artefato `finalize` | roda testes, detecta worktree, menu: merge local / PR / manter |
| `superpowers:systematic-debugging` | você, fora do schema | bug fix — não abre change |
| `superpowers:verification-before-completion` | sempre | antes de declarar qualquer coisa "pronta" |

Skills que o schema **não** usa: `executing-plans` (não ativa TDD/review transitivamente) e `dispatching-parallel-agents` (o SDD já paraleliza o que dá).

---

## 4. Caminho de uma nova feature (passo a passo)

Exemplo: "comunidade pode publicar produtos na loja".

### Fase 0 — vale abrir uma change?

Abre change se: nova capability, mudança de contrato/comportamento, breaking change, arquitetura, dependência externa nova, schema de banco, integração entre sistemas.
Não abre (branch + PR direto, skills do Superpowers continuam valendo): bug fix, typo, config, docs, bump de dependência sem breaking.

Na dúvida: `/opsx:explore` para investigar antes.

### Fase 1 — Planejamento (na branch principal ou numa feature branch)

```text
git checkout -b feat/loja-comunidade          # recomendado; o worktree do apply nasce daqui

/opsx:new loja-comunidade
  → cria openspec/changes/loja-comunidade/

/opsx:continue                                 # → brainstorm.md
  skill: superpowers:brainstorming
  o agente explora o código, faz perguntas (uma por vez), propõe 2-3 abordagens,
  você escolhe; ele grava a captura bruta em brainstorm.md e PARA
  (não pula para writing-plans — isso é instrução do schema)

/opsx:continue                                 # → proposal.md
  Why / What Changes / Capabilities (novas + modificadas) / Impact — 1-2 páginas
  ⚠ a lista de Capabilities é o contrato com as specs; revise aqui

/opsx:continue                                 # → design.md
  só se: cross-cutting, dependência nova, migração, segurança/performance
  senão o agente escreve "Not needed: <motivo>" e segue

/opsx:continue                                 # → specs/<capability>/spec.md
  uma por capability; ADDED / MODIFIED / REMOVED; Requirement + Scenario (WHEN/THEN)
  ⚠ cada Scenario vira um teste; revise com cuidado — é o que o TDD vai implementar

/opsx:continue                                 # → tasks.md
  checklist grosso "- [ ] 1.1 ... e verificar que <teste> passa"

/opsx:continue                                 # → plan.md
  skill: superpowers:writing-plans
  micro-passos de 2-5 min por task, com arquivo, snippet e comando de teste;
  cada ## Task cita os ids X.Y do tasks.md

git add openspec/changes/loja-comunidade && git commit -m "docs(openspec): scaffold loja-comunidade"
  (o apply faz isso sozinho se você esquecer — passo 0c)
```

Atalho para mudança pequena: `/opsx:ff loja-comunidade` faz todas as etapas acima sem parar; revise o resultado antes do apply.

### Fase 2 — Implementação

```text
/opsx:apply
  0. pre-flight: confere skills, ferramentas (npm/pytest/docker…), commita a pasta da change,
     registra a branch atual
  1. superpowers:using-git-worktrees → worktree + branch "loja-comunidade"
  2. superpowers:subagent-driven-development
       para cada task do plan.md: subagente implementa com TDD → subagente revisa (spec + qualidade)
       → fix rounds se precisar → marca "- [x]" no tasks.md → commit
       revisão final do branch inteiro
  3. escreve apply.md (iteração 1, range de commits, X de Y tasks)

/opsx:verify                                    # → verify.md
  1 validate --all   2 tasks todas [x]   3 specs sincronizadas?   4 design ↔ specs
  5 tudo commitado   6 vazou algo em docs/superpowers/?   7 checks manuais têm teste equivalente?
  PASS → segue      FAIL → /opsx:apply de novo (iteração 2), depois /opsx:verify
  mais de 5 iterações → para e conversa
```

### Fase 3 — Fechamento

```text
/opsx:continue                                  # → retrospective.md
  evidências (commits, diff, tasks, iterações) → wins / misses / desvios do plano /
  skills usadas ou puladas (com justificativa) / surpresas / candidatos a memória
  mudança trivial: uma linha "Skipped: <motivo>"

/opsx:continue                                  # → finalize.md
  skill: superpowers:finishing-a-development-branch
  roda a suíte de testes → menu:
    1. merge local na branch base
    2. push + Pull Request
    3. manter a branch como está
  grava o resultado em finalize.md

/opsx:archive
  sincroniza specs/<capability>/spec.md da change para openspec/specs/
  move a pasta para openspec/changes/archive/YYYY-MM-DD-loja-comunidade/
  - escolheu merge local → roda na branch base
  - escolheu PR → roda na feature branch, push, e o merge do PR leva código + archive juntos
```

### Resumo em uma tela

```text
/opsx:new X → /opsx:continue ×6 (brainstorm, proposal, design, specs, tasks, plan)
→ /opsx:apply → /opsx:verify (repete até PASS)
→ /opsx:continue ×2 (retrospective, finalize) → /opsx:archive
```

---

## 5. Caminhos alternativos

| Situação | Caminho |
|---|---|
| Bug fix | `git checkout -b fix/…` → `superpowers:systematic-debugging` → TDD → PR. Sem change. |
| Refactor sem mudança de comportamento, mas grande | `/opsx:new` normal e `skip_specs: true` no `.openspec.yaml` da change (sem delta specs) |
| Escopo mudou no meio do apply | `/opsx:update` para ajustar specs/tasks → commit na worktree → `/opsx:apply` (iteração N+1) |
| Quer o fluxo OpenSpec puro numa change | `/opsx:new fix-rapido --schema spec-driven` |
| Perdeu o contexto (compaction) | `openspec status --change X` diz onde parou; o ledger em `.superpowers/sdd/plan/progress.md` da worktree diz quais tasks já foram |
| Testar o schema sem sujar nada | `openspec new change smoke && openspec status --change smoke && rm -rf openspec/changes/smoke` |

---

## 6. Sinais de que algo saiu do trilho

- Apareceu arquivo em `docs/superpowers/specs/` ou `docs/superpowers/plans/` → a skill escreveu fora da change (verify check 6 avisa). Mova o conteúdo para a change e apague.
- `/opsx:continue` ofereceu `verify` antes de você rodar `apply` → não deveria; `verify` exige `apply.md`. Reporte como bug do schema.
- O agente começou a implementar sem worktree ou na `main` → pare; `/opsx:apply` passo 1 é obrigatório.
- Brainstorm pulou direto para plano → o schema manda parar depois do `brainstorm.md`; peça `/opsx:continue` para o proposal.
