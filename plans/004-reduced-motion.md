# 004 — Respeitar `prefers-reduced-motion` no CSS e no Lenis

- **Status**: DONE
- **Commit**: 74648b8
- **Severity**: HIGH
- **Category**: Accessibility
- **Estimated scope**: 5 arquivos HTML; ~30 linhas de CSS + ~8 linhas de JS por arquivo
- **Depende de**: 001, 002, 003 (este plano sobrescreve estados finais criados por eles; execute por último)

## Problem

Quem ativa "Reduzir movimento" no sistema ainda recebe todo o movimento do site. O JS já respeita a preferência em alguns pontos (vídeo do hero, formas do "Por que Rooker", blur do logo, avatar — `index.html:1540`), mas o CSS não tem nenhuma regra `@media (prefers-reduced-motion: reduce)` e o Lenis é criado sempre.

- 37 elementos `[data-reveal]` na home (25–29 em cada projeto) entram deslizando 40px:
  ```css
  /* index.html:933 — current (projetos: linha 902) */
  [data-reveal] {
    opacity: 0;
    transform: translateY(40px);
    transition: opacity 0.9s cubic-bezier(0.16, 1, 0.3, 1),
                transform 0.9s cubic-bezier(0.16, 1, 0.3, 1);
  }
  [data-reveal="left"]  { transform: translateX(-40px); }
  [data-reveal="right"] { transform: translateX(40px); }
  ```
- Hover com deslocamento/zoom: setas e label do `.btn`, zoom do `.case-img`, zoom de `.others-img img`, troca nome/descrição com `translateY` nos cards de processo.
- Lenis aplica inércia de 1.2s a todo scroll:
  ```js
  // index.html:1317 — current (projetos: ~1215-1232)
  const lenis = new Lenis({
    duration: 1.2,
    easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
    smoothWheel: true,
  });
  ```

## Target

Movimento reduzido = **menos e mais suave, não zero**. Mantém fades de opacidade e trocas de cor (ajudam a entender o estado); remove deslocamentos, zooms e scroll com inércia.

Bloco CSS, colocado como **última regra** dentro do `<style>` de cada página (depois da seção RESPONSIVE), para vencer na cascata:

```css
/* ── MOVIMENTO REDUZIDO ── */
@media (prefers-reduced-motion: reduce) {
  /* reveal: só fade, curto, sem atraso escalonado */
  [data-reveal],
  [data-reveal="left"],
  [data-reveal="right"],
  [data-reveal].revealed {
    transform: none;
    transition: opacity 0.4s ease;
    transition-delay: 0s;
  }

  /* botão: cores e preenchimento continuam; setas e texto não se deslocam */
  .btn .arr-in,
  .btn .arr-out,
  .btn .btn-label { transition: stroke 0.4s; }
  .btn .arr-in { display: none; }
  .btn .btn-label,
  .btn:hover .btn-label { transform: none; }
  .btn .btn-circle,
  .btn:hover .btn-circle { transform: translate(-50%,-50%) scale(1); }
  .btn:active { transform: none; }

  /* imagens: sem zoom; o blur dos cases vira só um leve fade */
  .case-img { transition: filter 0.3s ease; }
  a.case-item:hover .case-img,
  .case-item:hover .case-img { transform: none; }
  .others-card[href]:hover .others-img img { transform: none; }

  /* processo: crossfade nome ↔ descrição sem deslizar */
  .process-name,
  .process-card-desc,
  .process-card:hover .process-name,
  .process-card:hover .process-card-desc,
  .process-card:focus-visible .process-name,
  .process-card:focus-visible .process-card-desc { transform: none; }
  .process-toggle img { transition: opacity 0.3s ease; }
  .process-toggle .ico-plus,
  .process-toggle .ico-minus,
  .process-card:hover .process-toggle .ico-plus,
  .process-card:hover .process-toggle .ico-minus,
  .process-card:focus-visible .process-toggle .ico-plus,
  .process-card:focus-visible .process-toggle .ico-minus { transform: none; }
}
```

Observação: em `index.html` as regras de hover de `.btn`, `.case-img` e `.process-card` estarão dentro de `@media (hover: hover) and (pointer: fine)` após o plano 002. Os seletores acima têm especificidade igual ou maior e vêm depois na cascata — vencem. Se algum seletor não existir num arquivo (ex.: `.others-card` só existe nos projetos; `.process-card` só tem markup na home), deixe a regra mesmo assim — é inofensiva e mantém o bloco idêntico nos 5 arquivos.

JS — Lenis só para quem não pediu menos movimento:

```js
// target
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
const lenis = prefersReducedMotion ? null : new Lenis({
  duration: 1.2,
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  smoothWheel: true,
});
```
- Toda chamada `lenis.raf(time)` → `if (lenis) lenis.raf(time);`
- Todo `lenis.scrollTo(target, { offset: -80 });` → 
  ```js
  if (lenis) lenis.scrollTo(target, { offset: -80 });
  else window.scrollTo({ top: target.getBoundingClientRect().top + window.scrollY - 80, behavior: 'auto' });
  ```

## Repo conventions to follow

- O JS já detecta a preferência com `window.matchMedia('(prefers-reduced-motion: reduce)')` (index.html:1540, projeto-belief.html:1254). **Não** reutilize a constante `reduceMotion` existente: ela é declarada com `const` *depois* da criação do Lenis, e usá-la antes causa erro (temporal dead zone). Por isso a nova constante `prefersReducedMotion`.
- Comentários de seção no CSS seguem o formato `/* ── NOME ── */`.

## Steps

Para cada arquivo (`index.html`, `projeto-belief.html`, `projeto-casa-lemham.html`, `projeto-fabiola-molina.html`, `projeto-inspire-seguros.html`):

1. Cole o bloco CSS `/* ── MOVIMENTO REDUZIDO ── */` do Target imediatamente antes de `</style>`.
2. No `<script>` do fim do `body`, substitua `const lenis = new Lenis({ … });` pela versão do Target (com `prefersReducedMotion`).
3. Procure toda ocorrência de `lenis.raf(` e `lenis.scrollTo(` no arquivo (`grep -n "lenis\." <arquivo>`) e aplique a guarda `if (lenis)` conforme o Target. Na home: `globalRAF` (~1765) e a função `lenisRAF` (~1323); nos projetos: a função `loop` e o handler dos links âncora.

## Boundaries

- NÃO remova os tratamentos de reduced motion que já existem no JS (vídeo, formas, blur, avatar).
- NÃO altere o comportamento para quem **não** ativou a preferência — tudo deve ficar idêntico.
- NÃO adicione dependências.
- Se a criação do Lenis ou as chamadas não baterem com os trechos "current", PARE e reporte.

## Verification

- **Mechanical**: abrir as 5 páginas com e sem a preferência; console sem erros (em especial, nenhum `Cannot read properties of null (reading 'raf')`).
- **Feel check** — DevTools → Rendering → "Emulate CSS media feature prefers-reduced-motion: reduce", recarregar:
  - Rolar a página: scroll nativo, sem inércia; seções aparecem só com fade curto, sem deslizar e sem atraso escalonado.
  - Hover no botão: o preenchimento preto aparece por fade e o texto muda de cor; nada desliza.
  - Hover em case: blur leve, sem zoom. Card de processo: nome e descrição trocam por fade, sem subir/descer.
  - Clicar num link âncora (ex. "Marcar uma conversa" → #contato): pula para a seção com o mesmo offset de 80px.
  - Desligar a emulação e recarregar: tudo volta exatamente como antes.
- **Done when**: com reduced motion, nenhuma animação de posição/escala no site (exceto o necessário para revelar o círculo do botão por fade) e scroll nativo; sem a preferência, sem nenhuma diferença.
