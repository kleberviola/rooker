# 001 — Botão `.btn`: trocar animação de layout por transform

- **Status**: DONE
- **Commit**: 74648b8
- **Severity**: HIGH
- **Category**: Performance
- **Estimated scope**: 5 arquivos HTML (CSS inline), ~25 linhas por arquivo, edição idêntica em todos

## Problem

O hover do botão `.btn` (usado 18× na home e 6× em cada página de projeto) anima propriedades de layout: `width`/`height` do círculo de fundo (16px → 340px) e `left`/`right` das setas. Cada quadro da transição de 0.8s força layout + paint. Em máquina lenta ou com o loop de rAF da página rodando, o botão engasga.

Mesmo CSS em `index.html:252-305` e em `projeto-belief.html`, `projeto-casa-lemham.html`, `projeto-fabiola-molina.html`, `projeto-inspire-seguros.html` em `:222-275`.

```css
/* index.html:252 — current */
.btn .arr-in {
  position: absolute;
  left: -25%;
  width: 16px; height: 16px;
  z-index: 9;
  transition: left 0.8s cubic-bezier(0.34,1.56,0.64,1),
              stroke 0.4s;
  stroke: var(--black);
  fill: none;
  flex-shrink: 0;
}
.btn:hover .arr-in { left: 16px; stroke: var(--white); }

/* index.html:276 — current */
.btn .btn-circle {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%,-50%);
  width: 16px; height: 16px;
  background: var(--black);
  border-radius: 50%;
  opacity: 0;
  transition: width 0.8s cubic-bezier(0.19,1,0.22,1),
              height 0.8s cubic-bezier(0.19,1,0.22,1),
              opacity 0.8s cubic-bezier(0.19,1,0.22,1);
}
.btn:hover .btn-circle {
  width: 340px; height: 340px;
  opacity: 1;
}

/* index.html:293 — current */
.btn .arr-out {
  position: absolute;
  right: 16px;
  width: 16px; height: 16px;
  z-index: 9;
  stroke: var(--black);
  fill: none;
  flex-shrink: 0;
  transition: right 0.8s cubic-bezier(0.34,1.56,0.64,1),
              stroke 0.4s;
}
.btn:hover .arr-out { right: -25%; stroke: var(--white); }
```

Além disso, `.btn:active { transform: scale(0.95); }` (index.html:249) não tem transição de `transform`, então o clique "pula".

## Target

Mesmo visual, mesmas curvas e durações (a personalidade com overshoot é intencional — **não mude curvas nem durações**). Só a propriedade animada muda:

- Círculo: tamanho fixo de 340px, cresce por `scale` de `0.047` (= 16/340) até `1`.
- Setas: posição fixa, deslocam por `translateX`. `.btn` já tem `overflow: hidden`, então a seta fora da área continua escondida.
- Press: `scale(0.97)` com `transition: transform 160ms cubic-bezier(0.23, 1, 0.32, 1)`.

```css
/* target */
.btn {
  /* …propriedades existentes… */
  transition: border-color 0.6s cubic-bezier(0.23,1,0.32,1),
              color 0.6s cubic-bezier(0.23,1,0.32,1),
              border-radius 0.6s cubic-bezier(0.23,1,0.32,1),
              transform 160ms cubic-bezier(0.23,1,0.32,1);
}
.btn:active { transform: scale(0.97); }

.btn .arr-in {
  position: absolute;
  left: 16px;
  width: 16px; height: 16px;
  z-index: 9;
  transform: translateX(-72px);
  transition: transform 0.8s cubic-bezier(0.34,1.56,0.64,1),
              stroke 0.4s;
  stroke: var(--black);
  fill: none;
  flex-shrink: 0;
}
.btn:hover .arr-in { transform: translateX(0); stroke: var(--white); }

.btn .btn-circle {
  position: absolute;
  top: 50%; left: 50%;
  width: 340px; height: 340px;
  transform: translate(-50%,-50%) scale(0.047);
  background: var(--black);
  border-radius: 50%;
  opacity: 0;
  transition: transform 0.8s cubic-bezier(0.19,1,0.22,1),
              opacity 0.8s cubic-bezier(0.19,1,0.22,1);
}
.btn:hover .btn-circle {
  transform: translate(-50%,-50%) scale(1);
  opacity: 1;
}

.btn .arr-out {
  position: absolute;
  right: 16px;
  width: 16px; height: 16px;
  z-index: 9;
  stroke: var(--black);
  fill: none;
  flex-shrink: 0;
  transition: transform 0.8s cubic-bezier(0.34,1.56,0.64,1),
              stroke 0.4s;
}
.btn:hover .arr-out { transform: translateX(72px); stroke: var(--white); }
```

## Repo conventions to follow

- Todo o CSS é inline no `<style>` de cada página; não existe arquivo de CSS compartilhado. A mesma edição precisa ser repetida nos 5 arquivos.
- Valores escritos à mão, sem tokens (exceto `--ease-soft`). Mantenha as curvas literais como estão.
- Exemplar de animação já correta via transform: `.btn .btn-label` (index.html:266-273) usa `translateX`.

## Steps

Para cada um dos arquivos `index.html`, `projeto-belief.html`, `projeto-casa-lemham.html`, `projeto-fabiola-molina.html`, `projeto-inspire-seguros.html`:

1. No bloco `.btn { … }` (seção `/* ── FLOW BUTTON ── */`), acrescente `transform 160ms cubic-bezier(0.23,1,0.32,1)` como quarto item da lista `transition`.
2. Troque `.btn:active { transform: scale(0.95); }` por `.btn:active { transform: scale(0.97); }`.
3. Substitua os blocos `.btn .arr-in`, `.btn:hover .arr-in`, `.btn .btn-circle`, `.btn:hover .btn-circle`, `.btn .arr-out`, `.btn:hover .arr-out` pelo código de **Target** acima.
4. Não mexa em `.btn .btn-label` nem em `.btn-sm`.

## Boundaries

- NÃO altere curvas, durações ou cores.
- NÃO altere o HTML dos botões.
- NÃO toque em `.btn-text`, `.nav-links` nem outros componentes.
- NÃO adicione dependências.
- Se o CSS encontrado não bater com o trecho "current" (drift desde 74648b8), PARE e reporte.

## Verification

- **Mechanical**: `grep -nE "transition: (left|right|width)" index.html projeto-*.html` não deve retornar linhas da seção FLOW BUTTON. Abrir cada página no navegador sem erros no console.
- **Feel check**: passe o mouse no botão "Marcar uma conversa" (home) e em um `.btn` de cada página de projeto:
  - A seta da esquerda entra com o mesmo leve overshoot de antes; a da direita sai pela borda.
  - O círculo preto cresce do centro e cobre o botão inteiro (incluindo botões largos — se algum botão mostrar canto não coberto, reporte).
  - DevTools → Animations a 10%: o círculo cresce suave, sem bordas serrilhadas no início.
  - DevTools → Performance, gravar o hover: sem eventos roxos de "Layout" durante a transição.
  - Clicar e segurar: o botão encolhe sutilmente (0.97) de forma suave, não instantânea.
- **Done when**: nenhuma propriedade de layout em `transition` no `.btn` e seus filhos, nos 5 arquivos, e o hover visualmente idêntico ao anterior.
