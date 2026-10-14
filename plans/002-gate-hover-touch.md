# 002 — Restringir efeitos de hover a dispositivos com mouse

- **Status**: DONE
- **Commit**: 74648b8
- **Severity**: MEDIUM
- **Category**: Accessibility
- **Estimated scope**: 5 arquivos HTML (CSS inline), ~40 linhas por arquivo
- **Depende de**: 001 (os seletores `.btn:hover` foram reescritos lá)

## Problem

Em toque, o navegador dispara `:hover` no tap e o estado fica preso até tocar em outro lugar. Resultado no celular:
- Card de case fica borrado (`filter: blur(5px)`) e ampliado por um instante antes de navegar.
- Card de processo fica preto, preso no estado de hover, depois do toque.
- Botão `.btn` fica preenchido de preto depois do toque.

```css
/* index.html:401 — current (nos projetos: `.case-item:hover .case-img`, linha 370) */
a.case-item:hover .case-img {
  filter: blur(5px);
  transform: scale(1.03);
}

/* index.html:650 — current (nos projetos: linha 619) */
.process-card:hover,
.process-card:focus-visible {
  background: var(--black);
  border-color: var(--black);
  color: var(--white);
}
/* …e as demais regras `.process-card:hover …, .process-card:focus-visible …` até index.html:747 */

/* projetos, ex. projeto-belief.html:1045 — current */
.others-card[href]:hover .others-img img { transform: scale(1.04); }
```

## Target

Toda regra com `:hover` que muda transform, filter ou cores de fundo passa para dentro de:

```css
@media (hover: hover) and (pointer: fine) { … }
```

As variantes `:focus-visible` (teclado) **ficam fora** da media query — o foco por teclado deve continuar funcionando em qualquer dispositivo.

## Repo conventions to follow

- O JS do site já usa exatamente essa media query para o cursor customizado: `index.html:1619` → `window.matchMedia('(hover: hover) and (pointer: fine)')`. Use a mesma string em CSS.
- CSS inline em cada página; repetir nos 5 arquivos.

## Steps

Para cada arquivo (`index.html`, `projeto-belief.html`, `projeto-casa-lemham.html`, `projeto-fabiola-molina.html`, `projeto-inspire-seguros.html`):

1. **Cases**: remova a regra `a.case-item:hover .case-img { … }` (ou `.case-item:hover .case-img` nos projetos) de onde está e recrie-a, idêntica, dentro de um bloco `@media (hover: hover) and (pointer: fine) { … }` logo abaixo de `.case-img { … }`.
2. **Botão**: mova as quatro regras `.btn:hover`, `.btn:hover .arr-in`, `.btn:hover .btn-circle`, `.btn:hover .arr-out` e `.btn:hover .btn-label` para um bloco `@media (hover: hover) and (pointer: fine) { … }` ao final da seção FLOW BUTTON (antes de `.btn-sm`). Conteúdo das regras inalterado.
3. **Processo**: cada regra hoje no formato
   ```css
   .process-card:hover X,
   .process-card:focus-visible X { … }
   ```
   vira duas: a versão `:focus-visible X { … }` fica no lugar original; a versão `:hover X { … }` (mesmo corpo, incluindo `transition-delay: 0.12s` onde existir) vai para um bloco `@media (hover: hover) and (pointer: fine) { … }` ao final da seção `/* ── PROCESSO ── */`. São 6 regras: o card, `.process-shapes svg`, `.ico-plus`, `.ico-minus`, `.process-name`, `.process-card-desc`.
4. **Outros projetos** (só nos 4 arquivos `projeto-*.html`): mova `.others-card[href]:hover .others-img img { transform: scale(1.04); }` para dentro de um bloco `@media (hover: hover) and (pointer: fine) { … }` logo abaixo.
5. Não mexa em `.nav-links a:hover`, `.btn-text:hover`, `.footer-links .btn-text:hover` (só opacidade — inofensivo).

## Boundaries

- NÃO mude valores, curvas, durações ou cores — apenas mova regras.
- NÃO remova as variantes `:focus-visible`.
- NÃO altere HTML nem JS.
- Se encontrar drift em relação ao trecho "current", PARE e reporte.

## Verification

- **Mechanical**: `grep -nE ":hover" index.html projeto-*.html` — toda linha restante fora de media query deve ser só de `.nav-links`, `.btn-text` ou `.footer-links`.
- **Feel check**:
  - Desktop com mouse: todos os hovers (botão, case, processo, outros projetos) idênticos ao antes.
  - DevTools → modo dispositivo (iPhone) e recarregar: tocar num card de processo **não** deixa ele preto; tocar num case navega sem blur.
  - Teclado (Tab) na home: ao focar um card de processo ele continua ficando preto com a troca nome → descrição.
- **Done when**: nenhum efeito de transform/filter/fundo dispara por toque, e foco por teclado continua funcionando.
