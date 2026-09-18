# sdd-superpowers

Schema customizado do [OpenSpec](https://github.com/Fission-AI/OpenSpec) que integra o pipeline de artefatos (spec-driven) com as skills de execução do [Superpowers](https://github.com/obra/superpowers) (brainstorming, writing-plans, worktrees, subagent-driven TDD, code review, finishing-a-development-branch).

É um híbrido de dois projetos MIT, pegando o melhor de cada:

| Vem de | O quê |
|---|---|
| [danielhanold/superspec](https://github.com/danielhanold/superspec) v4 | Mecânica do workflow: commit pre-flight dos artefatos antes do worktree, `apply.md` como recibo (verify só depois de apply existir), `design.md` opcional, loop de convergência apply → verify com contador de iteração |
| [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas) `superpowers-bridge` v1 | PRECHECK de skills antes de invocar, `brainstorm.md` como captura bruta e `design.md` como reestruturação, artefato `retrospective` (evidência primeiro, candidatos a memória), guia "quando NÃO abrir uma change", fragmento para `CLAUDE.md` |
| Este repo | `finalize` invoca a skill `superpowers:finishing-a-development-branch` em vez de embutir 350 linhas de bash; base branch resolvida sem depender de `origin/main`; templates 100% em inglês; alinhado ao OpenSpec 1.13 (`skip_specs`, regra de checkbox `[x]`, `## Purpose` em capability nova) |

Validado com `openspec schema validate` no OpenSpec **1.13.1** e escrito contra Superpowers **6.3.0**.

## Fluxo

```text
PLANEJAMENTO
  brainstorm.md ──┬─→ proposal.md ──→ specs/**/*.md ──→ tasks.md ──→ plan.md
                  └─→ design.md (opcional; "Not needed: <motivo>" para pular)

APLICAÇÃO  (/opsx:apply — requires: plan, tracks: tasks.md)
  0. pre-flight: skills, ferramentas, commit de openspec/changes/<name>/
  1. superpowers:using-git-worktrees
  2. superpowers:subagent-driven-development (+ TDD + code-review transitivos)
  3. apply.md (recibo, iteração N)
  4. /opsx:verify → verify.md ──FAIL──→ volta ao passo 2

FECHAMENTO
  retrospective.md → finalize.md (superpowers:finishing-a-development-branch) → /opsx:archive
```

Cada artefato só fica disponível quando o anterior existe; `verify` exige `apply.md`, então não dá para verificar antes de implementar.

## Instalação

Pré-requisitos: OpenSpec ≥ 1.13 e o plugin Superpowers instalado no seu harness (Claude Code: `claude plugin install superpowers@claude-plugins-official`).

```bash
# na raiz do projeto que vai usar o schema
openspec init                       # se ainda não tem openspec/
mkdir -p openspec/schemas
cp -r /caminho/para/sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/
echo "schema: sdd-superpowers" > openspec/config.yaml   # ou use --schema por change

openspec schema validate sdd-superpowers
openspec schemas                    # sdd-superpowers deve aparecer
```

Opcional, mas recomendado: cole `openspec/schemas/sdd-superpowers/templates/adopters/CLAUDE.md.fragment.md` no `CLAUDE.md` do projeto. É o que ensina o agente a decidir *quando* abrir uma change (feature, contrato, arquitetura) e quando fazer PR direto (bug fix, typo, config).

## Uso

Dentro do harness (Claude Code etc.), com os comandos `/opsx:*` que o OpenSpec instala:

```text
/opsx:new minha-feature   # cria a change
/opsx:continue            # → brainstorm (conversa guiada pela skill)
/opsx:continue            # → proposal
/opsx:continue            # → design (ou "Not needed: ...")
/opsx:continue            # → specs
/opsx:continue            # → tasks
/opsx:continue            # → plan (writing-plans)
/opsx:apply               # worktree + subagent-driven TDD → apply.md
/opsx:verify              # 7 checks → verify.md
/opsx:continue            # → retrospective
/opsx:continue            # → finalize (menu: merge local / PR / manter)
/opsx:archive             # sincroniza delta specs e arquiva a change
```

Para mudanças pequenas e bem entendidas, `/opsx:ff minha-feature` gera todos os artefatos de planejamento de uma vez; depois `/opsx:apply` em diante é igual.

Para pular o schema numa change específica: `/opsx:new fix-rapido --schema spec-driven`.

## Decisões de design

- **`design.md` opcional, mas sempre "completo"**: quando não é necessário, o agente escreve uma linha (`Not needed: <motivo>`). Assim o grafo não fica com um artefato eternamente pendente e o `/opsx:continue` avança linear.
- **`verify` requer `apply`, não `plan`**: o bridge declara `requires: [plan]` e tenta corrigir com prosa; aqui o recibo `apply.md` resolve estruturalmente.
- **Commit pre-flight antes do worktree**: corrige o bug [#6 do bridge](https://github.com/JiangWay/openspec-schemas/issues/6) (artefatos untracked não existem dentro do worktree).
- **Base branch sem `origin/main`**: corrige o [#14 do bridge](https://github.com/JiangWay/openspec-schemas/issues/14); resolve via `origin/HEAD`, depois `main`/`master`/`develop` locais, depois o root commit.
- **`finalize` delega à skill**: `finishing-a-development-branch` já faz verify tests → detectar worktree → menu → merge/PR → cleanup. Reimplementar isso em YAML (como o superspec v4) é frágil e assume GitHub + `gh` + topologia fixa de branches.
- **Retrospective antes do finalize**: fica no mesmo PR que o código, escrita com o contexto "quente".
- **Sem fallback para `executing-plans`**: ela não ativa TDD nem code review transitivamente; sem subagentes, use o `spec-driven` nativo.

## Aviso: ledger do subagent-driven-development (Superpowers ≥ 6.x)

A skill grava o progresso em `.superpowers/sdd/<basename-do-plano>/progress.md` na raiz da worktree, com o caminho completo do plano na primeira linha. Como todo plano deste schema se chama `plan.md`, o diretório é sempre `.superpowers/sdd/plan/`; a skill distingue changes pelo caminho gravado no ledger, mas não apaga o ledger alheio. Rode `/opsx:apply` sempre numa worktree por change (o passo 1 do apply instrui a limpar um ledger de outra change, se houver).

## Licença

MIT. Derivado de [danielhanold/superspec](https://github.com/danielhanold/superspec) (MIT © 2026 Daniel Hanold) e [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas) (MIT). Instruções de `specs`/`tasks`/`design` adaptadas do schema `spec-driven` do OpenSpec (MIT).
