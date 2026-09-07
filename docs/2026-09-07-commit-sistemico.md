# /commit sistêmico — arquitetura, decisão e migração (2026-09-07)

> Este documento vive no repositório-perfil (`alealla/alealla`) de propósito: é
> **supra-repos** — a fonte da verdade de como qualquer agente (Claude, Codex,
> Grok, Kimi) deve commitar/publicar em **qualquer repositório do usuário**,
> não só num projeto específico. `AGENTS.md` deste repositório aponta pra cá.

## 1. O problema que motivou a mudança

Até 2026-09-07, cada repositório tinha seu próprio `.claude/commands/commit.md`
(às vezes também `ship.md`), copiado manualmente entre projetos. Uma auditoria
nos 5 principais (`aesthetic-ehr-final`, `luma-ultra`, `site-talita`,
`inventario`, skill global) achou:

- Cópias já divergentes — uma tinha atribuição de modelo desatualizada
  (`Claude Sonnet 4.6`) porque ninguém sincronizava as 5.
- `aesthetic-ehr-final` tinha a arquitetura mais madura (núcleo `/ship` +
  camadas `/commit-fast`/`/commit`/`/commit-all`), mas só valia ali.
- Contradições entre repos nunca resolvidas conscientemente: `git add -A` em 3
  repos vs. proibição explícita em 1 (nenhum dos 3 tinha *decidido* usar `-A`,
  só nunca formalizou o oposto).
- Nenhum repo tinha revisão sênior + teste de responsividade como padrão
  sistemático — cada um cobria uma fatia diferente.

## 2. Decisão

**`/commit` passa a ser sempre sistêmico.** Vive em
`~/.claude/skills/commit/SKILL.md` (Claude Code) e é replicado como instrução
de leitura direta em `AGENTS.md` para os demais agentes (Codex, Grok, Kimi) —
ver §5. **Nenhum repositório deve ter `.claude/commands/commit.md` próprio.**

### 2.1 Três níveis

| Comando | Gates de stack | `responsive-check` | Suíte completa | `pr-senior-review` | Handoff |
|---|---|---|---|---|---|
| `/commit-fast` | typecheck + lint | não | não | não | `mode=fast` |
| `/commit` (default) | + build | **sim** | não | **sim** | `mode=basic` |
| `/commit-full` | = básico | sim | **sim, serial** | sim | `mode=full` |

Sempre, nos três níveis: trava de arquivo sensível (padrões configuráveis) →
**stage seletivo** (nunca `git add -A`/`.` — decisão consciente, não omissão)
→ commit com atribuição por agente real → handoff.

### 2.2 O que é sistêmico vs. o que é local

- **Sistêmico** (`~/.claude/skills/commit/SKILL.md`, igual em todo repo):
  contexto git, invariante de branch protegida, trava de sensíveis, gates de
  stack detectados via `package.json`, `responsive-check` (rotina genérica,
  ver §3), mecânica de `pr-senior-review` (revisão de coesão/impacto/testes),
  push/PR/merge, limpeza pós-merge (branch/worktree/stash só da tarefa).
- **Local** (`.claude/commands/ship.md`, opcional, por repositório): tudo que
  exige conhecer domínio, deploy ou regra de negócio específica — trava LGPD
  de dado clínico, versionamento com regra de negócio, migration safety,
  staging gate + promoção de produção, `verify:*` de SEO/arquitetura, etc.
  Recebe `{mode, commitHash, branch, gatesRun, flags}` do sistêmico e roda
  **depois** de já existir um commit — se um gate de domínio falhar antes do
  push, `git reset --soft HEAD~1` desfaz e devolve o erro (commit é barato de
  desfazer, push não).
- **Parâmetros sem prosa** (`.claude/ship.config.json`, opcional): branch
  protegida, branch base, prefixos de branch, `stageMode`, padrões sensíveis
  extras (aditivos aos defaults, nunca substituem), comando de teste, config
  de `responsive-check`, estratégia de merge.
- **Checklist de revisão de domínio** (`.claude/review-checklist.md`,
  opcional): lido automaticamente pela rotina `pr-senior-review` — multi-
  tenancy, RBAC, formato de telefone, o que nunca pode aparecer num PR daquele
  domínio. Não duplicar como prosa solta no `ship.md`.

### 2.3 Atribuição de commit — regra permanente

```
Co-Authored-By: <Agente> <noreply@...>
Claude-Session: <URL da sessão>        (só quando o agente é Claude Code)
```

`<Agente>` é o nome do agente que **de fato executou o trabalho** — Claude,
Codex, Grok, Kimi etc. — **nunca uma versão específica** ("Sonnet 5", "Opus
4.7"): fica obsoleta em semanas e ninguém sincroniza a correção depois.
Decisão do usuário, 2026-09-07: "é importante mostrar quem fez o trabalho,
isso em todos os repos" — vale mesmo em repositórios que antes diziam
explicitamente "sem coautoria automática" (caso do `site-talita`, corrigido).

## 3. `responsive-check` — rotina genérica de guardrail visual

Antes só existia no `site-talita` (`verify-mobile.ts`, puppeteer, viewport
390×844). Generalizada em
`~/.claude/skills/commit/scripts/responsive-check.mjs`:

1. `config.responsive.check` do repo, se definido.
2. Script `verify:mobile`/`mobile:validate`/`test:responsive`/`test:visual` no
   `package.json`, se existir.
3. Frontend detectável + Playwright/Puppeteer instalado → roda o built-in.
4. Sem frontend detectável, ou sem lib de browser → **pula com motivo**
   (nunca falha por ausência).

Roda sempre em **subagente Sonnet**, nunca inline em modelo caro (screenshot é
a maior categoria de gasto medido em contexto de modelo caro). Corrige um bug
achado durante o smoke test real: o script antigo subia um dev server em
background quando precisava e nunca o matava — agora só mata o processo que
ele mesmo subiu (nunca um dev server que já estava de pé antes, que pode ser a
sessão interativa do usuário).

## 4. Migração aplicada (2026-09-07) — resultado real, verificado

Executada por Sonnet (proposta de arquitetura desenhada com Fable, a pedido
explícito do usuário — decisões de design que justificam modelo caro). Smoke
test real (não simulado): commit de verdade em cada repo, seguido de
verificação (`git ls-remote`, `gh pr view --json state`).

**Estado final (pós 5 rodadas de auditoria adversarial, ver §4.1): os 4 repos
estão migrados, mergeados e verificados** — nenhum ficou bloqueado ou
pendente. O bloqueio de `site-talita` (governança editorial do Fascículo 11)
e o não-push de `aesthetic-ehr-final` (branch com trabalho paralelo em
andamento) descritos na primeira versão deste documento foram ambos
resolvidos em sessões posteriores; ver PRs abaixo.

| Repositório | PRs mergeados (migração + correções) | Observações |
|---|---|---|
| `aesthetic-ehr-final` | #7237, #7239, #7240, #7242, #7245, #7246 | `ship.md` absorveu os EXTRAs de `commit.md`/`commit-fast.md`/`commit-all.md` (3 arquivos apagados); checklist de domínio foi para `review-checklist.md`; ratchets/`any`=0 (Política Zero) e camada 1 de revisão pré-commit restauradas depois de terem ficado de fora da migração inicial. |
| `luma-ultra` | #34, #35, #36, #37, #38, #39 | Ganhou `ship.md` que não existia antes: trava LGPD, colisão/segurança de migration Postgres, versionamento clínico 0.1/0.2/1.0, poll de Cloud Build. |
| `site-talita` | #506, #507, #508, #509 | Pendência de governança editorial resolvida numa sessão à parte antes da migração poder completar. |
| `inventario` | #1, #2, #3, #4, #5, #6 | Corrigido script `typecheck` ausente no `package.json` e version bump que ia pra um commit separado (quebrava revert atômico) — ver §4.1. |

**Garantia verificada em todos os 4**: nenhum trabalho alheio foi perdido,
misturado ou commitado por engano — cada commit tocou exatamente os arquivos
da migração (`git diff --stat`/`--cached --stat` conferido antes de cada
commit); nunca `git add -A`/`.`; trabalho de outras sessões (código, relatórios
sensíveis, arquivos de tracking) permaneceu intocado nas árvores de trabalho.
Todo PR acima foi confirmado `MERGED` via `gh pr view --json state,mergedAt`
antes de ser dado como concluído — nunca por relato de sessão anterior.

### 4.1 Cinco rodadas de auditoria adversarial

A migração inicial (rodada 1, fork) e a primeira releitura pessoal (rodada 2)
acharam 15 gaps de conteúdo perdido na tradução prosa→prosa. Isso levantou uma
categoria de bug mais séria — **regressão de comportamento/ordem de execução**,
não só texto faltando — que só apareceu comparando literalmente arquivo
original vs. novo, linha a linha, sem inferência nem resumo:

- **Rodada 3** (minha releitura completa): typecheck+lint rodando em paralelo
  (violava regra de memória de um SIGABRT real); camada 1 de revisão sênior
  pré-commit inteira ausente (só a camada 2, pós-push, tinha sobrevivido);
  prova de merge usando `git branch -d` em vez de `gh pr view` (sempre falso
  em squash merge); QA Sênior só a partir de `basic`; `--no-staging` perdido;
  Política Zero (`check:ratchets`, `quality:check`) ausente; task movendo pra
  `done` antes do PR existir.
- **Rodada 4** (Fable, independente, em paralelo): 9 gaps adicionais em
  `aesthetic-ehr-final` (detecção de task recém-criada, `Refs: tasks/`,
  fallback de suíte por módulo, etc.) + 3 em `luma-ultra` + 3 em `site-talita`
  (tier gating errado: build/runtime/deploy/docs presos a `full` quando o
  default real do sistêmico é `basic`) + 2 em `inventario` (version bump em
  commit separado, atribuição ausente).
- **Rodada 5** (minha + Fable, cada um em 2 repos, sem sobreposição): 2 gaps
  (PR com `--fill` genérico em `luma-ultra`; regra "sem commit artificial" em
  `site-talita`) + 12 em `aesthetic-ehr-final` (a maioria detalhe: mensagens
  exatas, comandos de fallback) + 5 em `inventario` (título do PR ausente
  travava `gh pr create` em sessão não-TTY).

**Lição operacional, registrada porque generaliza**: nenhuma rodada isolada —
nem a minha, nem a do Fable — encontrou tudo. Cada rodada nova achou algo que
a anterior tinha errado ou pulado. A mitigação que funcionou não foi "revisar
mais uma vez sozinho", foi comparação literal (citação exata dos dois lados,
nunca "parece coberto") **e** duas fontes independentes em paralelo sobre o
mesmo material — convergência (rodada 5 achou muito menos, e nada crítico) é
sinal de confiança crescente, não prova de zero bugs remanescentes.

## 5. Replicabilidade — Codex, Grok, Kimi (não só Claude Code)

`~/.claude/skills/commit/SKILL.md` só é descoberto automaticamente pelo
mecanismo de skills do Claude Code. Para outros agentes de terminal seguirem o
mesmo fluxo, a instrução precisa estar num arquivo que eles de fato leem —
por isso este documento mora no perfil `alealla/alealla` e é referenciado a
partir de `AGENTS.md`.

Arquivos atualizados nesta rodada com o ponteiro explícito ("leia
`~/.claude/skills/commit/SKILL.md`, depois cheque `.claude/commands/ship.md`
local"):

- `alealla/alealla/AGENTS.md` (este repositório — fonte canônica)
- `~/.codex/AGENTS.md` (regra global do Codex nesta máquina)
- `/Users/a/Code/AGENTS.md` (raiz do workspace)
- `AGENTS.md` de `luma-ultra`, `site-talita`, `inventario` (não versionados
  nesses repos — a correção já vale sem precisar de commit)
- `agents.md` de `aesthetic-ehr-final` (versionado — seção "SHIPPING"
  reescrita como parte do commit da migração)

**Por que isso funciona pra qualquer agente**: `SKILL.md` é um arquivo de
texto comum (frontmatter YAML + markdown + bash) — qualquer agente com acesso
ao filesystem local consegue lê-lo e seguir os passos, mesmo sem entender o
mecanismo de "skill" do Claude Code. A instrução no `AGENTS.md` só precisa
dizer "leia esse arquivo e siga".

## 6. Decisões de política, fechadas em 2026-09-07 (pós rodada 5)

- **Sempre PR, sem exceção.** A flag `--no-pr` foi **removida** do sistêmico —
  todo `/commit` (qualquer nível) sempre cria PR depois do push. Quem digitar
  `--no-pr` por hábito antigo é tratado como `--draft` (para antes do merge
  automático, mas a PR é criada do mesmo jeito) e avisado no relatório.
- **Merge sempre automático, produção NUNCA automática.** `gh pr merge` roda
  sozinho conforme `config.merge` em todo repo migrado — sem pedir confirmação
  a cada PR. Mas nenhum `ship.md` local pode promover pra um ambiente de
  produção distinto do que o merge em `main` já dispara sem pausa explícita e
  resposta "ok" textual do usuário. Hoje só `aesthetic-ehr-final` tem essa
  distinção (staging via merge automático, produção via `/promote` com
  confirmação); os outros 3 repos não têm ambiente de produção separado do
  deploy que o próprio merge dispara, então a regra não muda nada ali hoje —
  fica registrada pra valer automaticamente se algum ganhar essa etapa depois.

## 7. Pendências abertas

- **`new-project-standard`** (skill do Claude Code): atualizada para não gerar
  mais `commit.md`; `ship.md`/`review-checklist.md` só quando o usuário
  confirmar que o projeto precisa; `ship.config.json` sempre gerado com o
  detectado.
- Ratchets (any=0, lint=0, TS=0, CC≤5) e revisão sênior obrigatória ainda
  precisam ser retrofitados nos repos que não têm — hoje só
  `aesthetic-ehr-final` tinha os três pilares completos antes desta migração.
- Poll de deploy real (Cloud Build/Vercel/Railway) não foi executado em nenhum
  repo nesta rodada — as mudanças eram só `.claude/`, sem impacto em código de
  produto.
- Achados de baixa severidade da rodada 5 (mensagens de log exatas, `pnpm
  kill`/`--fix`/`--wait` do guard-oom fora da suíte full, argumento posicional
  de mensagem de commit, linhas de relatório específicas) foram deliberadamente
  **não corrigidos** — cosméticos/redundantes com o que já existe, registrados
  na memória de sessão pra não se perderem se algum dia importarem.

## Referências

- Worktrees de `aesthetic-ehr-final` (`lumave-blog-consolida`,
  `lumave-jornada-integrada-38`, `pix-multiplo-revisao`,
  `timesfm-forecast-proto`) e de `site-talita` (`site-talita-sonar-42`) ainda
  têm os `commit.md` antigos — convergem sozinhas quando as branches
  mergearem com `main`, não foram tocadas de propósito.
- Memória de sessão (Claude Code):
  `commit_ship_flows_por_repo.md`,
  `commit_ship_flows_audit.md`,
  `padrao_global_ship.md`,
  `migracao_commit_sistemico_2026_09.md`.
