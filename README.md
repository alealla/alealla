# Instruções globais

Este repositório reúne instruções globais de trabalho para agentes Codex.

As regras definem `gpt-6-astra` com `reasoning_effort low` para planejamento, coordenação e revisão, e `gpt-5.6-sol` com `reasoning_effort low` para execução, implementação e trabalho manual pesado. O fluxo mantém um único alvo principal ativo; qualquer paralelismo deve convergir para concluí-lo.

O arquivo deste repositório não é aplicado automaticamente a todos os projetos. A instalação local é manual e deve preservar — nunca sobrescrever — instruções já existentes no destino.

## Commit/ship sistêmico (supra-repos)

Desde 2026-09-07, `/commit` é sempre sistêmico — nenhum repositório do usuário deve ter `.claude/commands/commit.md` próprio. Arquitetura completa, decisões e o histórico da migração: [`docs/2026-09-07-commit-sistemico.md`](docs/2026-09-07-commit-sistemico.md).
