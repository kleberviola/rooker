# Planos de animação — site Rooker

Gerados pela auditoria `improve-animations` no commit `74648b8`. Cada plano é autocontido: pode ser executado por qualquer agente sem o contexto da conversa.

| # | Plano | Severidade | Status |
|---|---|---|---|
| 001 | [Botão `.btn`: transform no lugar de layout](001-btn-hover-transform.md) | HIGH | DONE |
| 002 | [Hover só em dispositivos com mouse](002-gate-hover-touch.md) | MEDIUM | DONE |
| 003 | [Acordeão de serviços: grid-rows + acessível](003-services-accordion.md) | MEDIUM | DONE |
| 004 | [Respeitar `prefers-reduced-motion`](004-reduced-motion.md) | HIGH | DONE |

## Ordem recomendada

1. **001** — reescreve as regras do `.btn`.
2. **002** — move os `:hover` (inclusive os do `.btn` reescritos em 001) para `@media (hover: hover) and (pointer: fine)`.
3. **003** — independente; só `index.html`.
4. **004** — por último: o bloco de movimento reduzido sobrescreve estados finais criados em 001–003.

## Dependências

- 002 → depende de 001
- 004 → depende de 001, 002, 003
- 003 → nenhuma

## Ficaram fora (auditoria, prioridade baixa)

- `globalRAF` escreve `top`/`left`/`width` do logo e dos cursores em todo quadro (MEDIUM).
- Hovers de 0.6–0.8s em botão/processo; 5 curvas ease-out quase iguais sem token; `.fade-up` sem uso; cursor "VER" nascendo de `scale(0)` (LOW).
- Oportunidade: View Transitions entre o card da home e a capa da página do projeto.
