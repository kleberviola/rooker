# 003 — Acordeão de serviços: animar sem max-height e torná-lo acessível

- **Status**: DONE
- **Commit**: 74648b8
- **Severity**: MEDIUM
- **Category**: Performance / Interruptibility
- **Estimated scope**: 1 arquivo (`index.html`): ~25 linhas de CSS, 4 trechos de HTML, 1 função JS. Nos `projeto-*.html` o CSS do acordeão existe mas não há markup — não mexer.

## Problem

O acordeão "Serviços" (home, seção /03) anima `max-height` de `0` a `300px` e `padding`, com `0.5s`. Problemas:
- `max-height: 300px` é um teto chutado: o tempo real da animação depende do texto (abre rápido demais com texto curto) e textos longos em tela estreita podem ser cortados.
- `max-height`/`padding` são propriedades de layout.
- O gatilho é `<div onclick>`: não recebe foco por Tab nem abre com Enter/Espaço, e leitor de tela não sabe que é expansível.

```css
/* index.html:536 — current */
.service-body {
  max-height: 0;
  overflow: hidden;
  font-size: 16px;
  color: var(--gray);
  line-height: 1.6;
  max-width: 680px;
  transition: max-height 0.5s cubic-bezier(0.4, 0, 0.2, 1),
              padding 0.5s cubic-bezier(0.4, 0, 0.2, 1),
              opacity 0.4s ease;
  opacity: 0;
  padding: 0;
}
.service-item.active .service-body {
  max-height: 300px;
  padding: 0 0 28px;
  opacity: 1;
}
```

```html
<!-- index.html:1119 — current (padrão repetido 4×: svc-branding, svc-rebranding, svc-evolucao, svc-site) -->
<div class="service-item active" id="svc-branding" data-reveal data-delay="1">
  <div class="service-toggle" onclick="toggleService('svc-branding')">
    <span class="service-name">Branding</span>
    <img src="assets/plus-icon.svg" class="service-plus" alt="" />
  </div>
  <div class="service-body">
    Para quem está construindo agora …
  </div>
</div>
```

```js
// index.html:1643 — current
function toggleService(id) {
  const item = document.getElementById(id);
  const isActive = item.classList.contains('active');
  document.querySelectorAll('.service-item').forEach(el => el.classList.remove('active'));
  if (!isActive) item.classList.add('active');
}
```

## Target

Técnica `grid-template-rows: 0fr → 1fr` — anima até a altura real do conteúdo, sem teto. Curva ease-out forte em 300ms (abrir/fechar = entrar/sair); a opacidade do texto acompanha. Transições CSS retomam do ponto atual se o usuário clicar no meio da animação (interrompível).

```css
/* target */
.service-body {
  display: grid;
  grid-template-rows: 0fr;
  opacity: 0;
  transition: grid-template-rows 300ms cubic-bezier(0.23, 1, 0.32, 1),
              opacity 200ms cubic-bezier(0.23, 1, 0.32, 1);
}
.service-body-inner {
  overflow: hidden;
  min-height: 0;
  font-size: 16px;
  color: var(--gray);
  line-height: 1.6;
  max-width: 680px;
}
.service-body-inner p { padding: 0 0 28px; }
.service-item.active .service-body {
  grid-template-rows: 1fr;
  opacity: 1;
}

.service-toggle {
  /* propriedades existentes + reset de botão: */
  width: 100%;
  background: none;
  border: 0;
  font: inherit;
  color: inherit;
  text-align: left;
}
.service-toggle:focus-visible { outline: 2px solid var(--black); outline-offset: 4px; }

/* ícone + : mesma curva do corpo */
.service-plus { transition: transform 300ms cubic-bezier(0.23, 1, 0.32, 1); }
```

```html
<!-- target (exemplo do primeiro item; repetir o padrão nos 4) -->
<div class="service-item active" id="svc-branding" data-reveal data-delay="1">
  <button type="button" class="service-toggle" aria-expanded="true" aria-controls="svc-branding-body" onclick="toggleService('svc-branding')">
    <span class="service-name">Branding</span>
    <img src="assets/plus-icon.svg" class="service-plus" alt="" />
  </button>
  <div class="service-body" id="svc-branding-body">
    <div class="service-body-inner"><p>Para quem está construindo agora …</p></div>
  </div>
</div>
```

```js
// target
function toggleService(id) {
  const item = document.getElementById(id);
  const isActive = item.classList.contains('active');
  document.querySelectorAll('.service-item').forEach(el => {
    el.classList.remove('active');
    el.querySelector('.service-toggle').setAttribute('aria-expanded', 'false');
  });
  if (!isActive) {
    item.classList.add('active');
    item.querySelector('.service-toggle').setAttribute('aria-expanded', 'true');
  }
}
```

## Repo conventions to follow

- CSS inline no `<style>` da página; JS inline no `<script>` do fim do `body`.
- O padrão de foco visível já existe nos cards de processo (`.process-card:focus-visible`, `outline-offset: 2px`) — siga o mesmo estilo.
- O JS do cursor (`index.html:1629`) já inclui `.service-toggle` na lista de "link" — continua funcionando com `<button>`.

## Steps

1. `index.html`, bloco `.service-toggle { … }` (linha ~508): acrescente as 6 propriedades de reset de botão do Target (`width`, `background`, `border`, `font`, `color`, `text-align`). Adicione logo abaixo a regra `.service-toggle:focus-visible`.
2. Bloco `.service-plus`: troque `transition: transform 0.4s ease;` por `transition: transform 300ms cubic-bezier(0.23, 1, 0.32, 1);`.
3. Substitua os blocos `.service-body { … }` e `.service-item.active .service-body { … }` pelo CSS do Target (incluindo `.service-body-inner` e `.service-body-inner p`).
4. No HTML (index.html ~1119-1158), para cada um dos 4 itens:
   - `<div class="service-toggle" onclick=…>` → `<button type="button" class="service-toggle" aria-expanded="…" aria-controls="<id>-body" onclick=…>` e o `</div>` correspondente → `</button>`. `aria-expanded="true"` só em `svc-branding` (o único que começa com `active`); nos outros `"false"`.
   - `<div class="service-body">` ganha `id="<id>-body"`, e o texto interno passa a ficar em `<div class="service-body-inner"><p>…</p></div>`. O texto não muda.
5. Substitua `function toggleService` pelo JS do Target.

## Boundaries

- Só `index.html`. NÃO toque no CSS de acordeão dos `projeto-*.html` (sem markup lá).
- NÃO altere textos, ícones, tamanho de fonte ou cores.
- NÃO mude `data-reveal` / `data-delay` dos itens.
- Se o código não bater com os trechos "current", PARE e reporte.

## Verification

- **Mechanical**: abrir `index.html` no navegador; console sem erros. `grep -n "max-height" index.html` não retorna nada na seção de serviços.
- **Feel check**:
  - Clicar em cada serviço: o corpo abre até a altura exata do texto, sem "pausa" no fim (sinal de teto de max-height) e sem cortar o texto em 375px de largura.
  - Clicar rápido duas vezes no mesmo item: a animação reverte do ponto onde estava, não recomeça do zero.
  - DevTools → Animations a 10%: abrir e fechar parecem rápidos no início e suaves no fim (ease-out); o `+` gira junto com a abertura.
  - Teclado: Tab chega em cada título, mostra outline, Enter e Espaço abrem/fecham.
  - Leitor de tela (VoiceOver: Cmd+F5): anuncia "botão, recolhido/expandido".
- **Done when**: os 4 itens abrem/fecham com altura exata, operáveis por teclado, com `aria-expanded` correto.
