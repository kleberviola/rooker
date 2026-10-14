# 005 — Transição do card para a capa do projeto (View Transitions entre páginas)

- **Status**: DONE
- **Commit**: be34a3d
- **Severity**: LOW (oportunidade nova, não é correção)
- **Category**: Missed opportunities, spatial consistency
- **Estimated scope**: 5 arquivos HTML; ~40 linhas de CSS + ~25 de JS por arquivo; 1 atributo removido + 1 estilo inline na capa de cada projeto

## Problem

Clicar num case da home (`index.html:1204-1222`) ou num card de "Outros projetos" (`projeto-*.html`, seção `.project-others`) troca de página de uma vez. A capa do projeto (`<img class="project-cover" data-reveal …>`, ex. `projeto-belief.html:1219`) aparece com o reveal genérico, sem nenhuma ligação com o card clicado.

## Target

- `@view-transition { navigation: auto; }` nas 5 páginas.
- Imagem do card e capa do projeto compartilham `view-transition-name: cover-<slug>` e `view-transition-class: cover`:
  - A imagem vai do card até a capa (e volta no "voltar") em **400ms, `cubic-bezier(0.77, 0, 0.175, 1)`** (movimento na tela → ease-in-out), com `object-fit: cover` para não distorcer.
  - O resto da página (root) faz um fade de **200ms, `cubic-bezier(0.23, 1, 0.32, 1)`**.
- O menu (`#nav`) recebe `view-transition-name: site-nav` para não piscar.
- **`pageswap`**: ao sair, remove o blur e o zoom do hover da imagem antes da captura (senão a captura sai borrada). Ao voltar pelo bfcache (`pageshow`), a classe é retirada.
- **`pagereveal`**: ao chegar por transição, os `[data-reveal]` que já estão na tela são marcados `.revealed` sem transição. Senão o card de destino estaria com `opacity: 0` no momento da captura.
- A capa do projeto perde o `data-reveal`: quem anima a chegada é a transição.
- **Movimento reduzido**: sem morph (`view-transition-name: none` nas imagens), só o fade de 200ms.
- Navegadores sem suporte (Firefox): navegação normal, como hoje.

```css
/* ── TRANSIÇÃO ENTRE PÁGINAS ── */
@view-transition { navigation: auto; }

#nav { view-transition-name: site-nav; }

a[href="projeto-fabiola-molina.html"]   :is(.case-img, .others-img img) { view-transition-name: cover-fabiola-molina; }
a[href="projeto-casa-lemham.html"]      :is(.case-img, .others-img img) { view-transition-name: cover-casa-lemham; }
a[href="projeto-belief.html"]           :is(.case-img, .others-img img) { view-transition-name: cover-belief; }
a[href="projeto-inspire-seguros.html"]  :is(.case-img, .others-img img) { view-transition-name: cover-inspire-seguros; }
.case-img, .others-img img, .project-cover { view-transition-class: cover; }

::view-transition-group(*.cover),
::view-transition-old(*.cover),
::view-transition-new(*.cover) {
  animation-duration: 400ms;
  animation-timing-function: cubic-bezier(0.77, 0, 0.175, 1);
}
::view-transition-image-pair(*.cover) { overflow: clip; }
::view-transition-old(*.cover),
::view-transition-new(*.cover) {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 200ms;
  animation-timing-function: cubic-bezier(0.23, 1, 0.32, 1);
}

/* captura sem o blur/zoom do hover */
.vt-leaving .case-img,
.vt-leaving .others-img img {
  filter: none !important;
  transform: none !important;
  transition: none !important;
}

@media (prefers-reduced-motion: reduce) {
  .case-img, .others-img img, .project-cover { view-transition-name: none !important; }
}
```

```html
<!-- capa de cada projeto: sem data-reveal, com o nome da transição -->
<img class="project-cover" style="view-transition-name: cover-belief" src="…" alt="…" />
```

```js
// ── TRANSIÇÃO ENTRE PÁGINAS ── (no <head>, para registrar antes do primeiro render)
window.addEventListener('pageswap', (e) => {
  if (e.viewTransition) document.documentElement.classList.add('vt-leaving');
});
window.addEventListener('pageshow', () => document.documentElement.classList.remove('vt-leaving'));
window.addEventListener('pagereveal', (e) => {
  if (!e.viewTransition) return;
  const vh = window.innerHeight;
  document.querySelectorAll('[data-reveal]:not(.revealed)').forEach((el) => {
    const r = el.getBoundingClientRect();
    // cards com imagem de transição sempre: no "voltar" o scroll é restaurado depois deste evento
    if ((r.bottom > 0 && r.top < vh) || el.querySelector('.case-img, .others-img')) {
      el.style.transition = 'none';
      el.classList.add('revealed');
      requestAnimationFrame(() => requestAnimationFrame(() => { el.style.transition = ''; }));
    }
  });
});
```

## Boundaries

- NÃO mude os reveals, hovers ou o resto das transições.
- NÃO adicione dependências.

## Verification

- Home → clicar num case: a imagem do card cresce até virar a capa; o resto da página faz fade.
- Voltar (botão do navegador ou logo): a capa encolhe de volta para o card, que está visível (não com opacity 0).
- "Outros projetos" → outro projeto: o mesmo morph.
- Com o mouse em cima do case no clique: a imagem que voa está nítida, sem blur.
- Reduzir movimento: só fade, sem morph.
- Firefox: navegação normal, sem erros.
