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

| Repositório | Resultado | Observações |
|---|---|---|
| `aesthetic-ehr-final` | Commitado local (branch `feature/pix-multiplo`, não pushado — branch com outro trabalho real em andamento) | Achou e corrigiu 2 problemas reais no processo: link markdown quebrado apontando pro `commit.md` apagado; gate estático `check-commit-commands.mjs` que protegia a arquitetura antiga de 3 arquivos, obsoleto sem ela — removido junto com `.claude/commit-invariantes.md`. `ship.md` absorveu os EXTRAs que viviam em `commit.md`/`commit-all.md`; checklist de domínio foi para `review-checklist.md`. |
| `luma-ultra` | Commit → push → **PR #34 → MERGED** → limpeza pós-merge | Único ajuste: Prettier reformatou `ship.config.json` (cosmético). Ganhou `ship.md` que não existia antes: trava LGPD (padrões migrados literalmente), colisão/segurança de migration Postgres, versionamento clínico 0.1/0.2/1.0, poll de Cloud Build. |
| `site-talita` | **Bloqueado, não commitado** | `verify:content-governance-manifest` varre a árvore inteira (`git status --porcelain`, não só staged) e reprovou por ~200 arquivos de trabalho real não commitado (pipeline do Fascículo 11: artigos científicos, PDFs, imagens). Mudanças da migração ficaram staged, preservadas, aguardando o usuário resolver a pendência de governança editorial — decisão dele, não da migração. |
| `inventario` | Commit → push → **PR #1 → MERGED** → limpeza pós-merge | Achado à parte (fora do escopo de mexer): `Relatorio_*.docx/.xlsx` reais (dados de inventário hospitalar) soltos e **não rastreados** na raiz do repo — risco real se algum dia rodar `git add -A` ali. |

**Garantia verificada em todos os 4**: nenhum trabalho alheio foi perdido,
misturado ou commitado por engano — cada commit tocou exatamente os arquivos
da migração (`git diff --stat`/`--cached --stat` conferido antes de cada
commit); nunca `git add -A`/`.`; trabalho de outras sessões (código, relatórios
sensíveis, arquivos de tracking) permaneceu intocado nas árvores de trabalho.

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

## 6. Pendências abertas

- **`site-talita`**: fechar a pendência de governança editorial do Fascículo
  11 pra poder completar o commit da migração ali.
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
