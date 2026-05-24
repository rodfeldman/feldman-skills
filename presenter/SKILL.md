---
name: presenter
description: Use when the user wants to present slides over Zoom/Meet with a separate speaker-notes view — generates two synchronized HTML pages from a slides deck (HTML) + speaking script (Markdown). One page is clean for screen-sharing to the audience; the other shows the current slide, next slide, full speaker notes, clock and stopwatch, only for the presenter. Trigger on requests like "criar apresentação para o Zoom com minhas falas", "presenter view", "duas telas slides + roteiro", "/presenter".
---

# Presenter — Dual-Screen Synchronized Presentation

Gera DUAS páginas HTML sincronizadas a partir de slides (HTML) + roteiro de falas (Markdown):

1. **`slides-{nome}.html`** — slides limpos, para compartilhar no Zoom/Meet. Recebe um patch de sync.
2. **`apresentador-{nome}.html`** — vista do apresentador: slide atual + próximo (miniaturas via iframe), roteiro completo, relógio, cronômetro, navegação. SÓ para o apresentador.

As duas janelas sincronizam via `localStorage` + evento `storage` (nativo do browser, sem servidor, offline). Passar o slide numa janela move a outra. Requisito: ambas no MESMO navegador.

## Quando usar

- Usuário tem uma apresentação pronta (slides HTML + roteiro MD) e quer apresentar com teleprompter pessoal.
- Usuário pede "presenter view", "duas telas", "compartilhar slides no Zoom e ver minhas falas".

## Inputs

| Input | Obrigatório | Default se ausente |
|-------|-------------|--------------------|
| Caminho do `slides.html` | Sim | Perguntar. (Futuro: gerar slides do zero) |
| Caminho do `roteiro.md` | Sim | Perguntar. (Futuro: gerar roteiro do zero) |
| Nome de saída (slug) | Não | Derivar do nome do arquivo de slides |
| Pasta de saída | Não | Mesma pasta dos inputs |

Se faltar slides OU roteiro, **PERGUNTE** antes de gerar. Não invente conteúdo.

## Procedimento

### Passo 1 — Ler e validar os slides

1. Ler o `slides.html`.
2. Detectar a estrutura dos slides com `grep`:
   - **Padrão esperado (default):** cada slide é `<div class="slide ...">`, com um slide ativo via `class="active"`, e função `goTo(n)` no script. Conferir: `grep -c 'class="slide' slides.html` deve dar o número de slides.
   - **Se a estrutura NÃO bater** (não há `.slide`, ou usa outra convenção como reveal.js / Gamma export): NÃO assumir. Ler o JS de navegação, identificar (a) o seletor de cada slide, (b) a função/mecanismo que troca de slide, (c) como o índice atual é representado. Adaptar o patch e o iframe-control a essa convenção. Se ambíguo, perguntar ao usuário.
3. Contar os slides → `TOTAL`.

### Passo 2 — Patch de sync nos slides

Copiar `slides.html` → `slides-{nome}.html`. Então **injetar sync** na função de navegação:

- Adicionar parâmetro `broadcast=true` à função `goTo` (ou equivalente).
- Ao final da função, se `broadcast`, gravar: `localStorage.setItem('presenter_slide', cur + ':' + Date.now())`.
- Adicionar listener que recebe mudanças da outra janela:

```js
window.addEventListener('storage', e => {
  if (e.key === 'presenter_slide' && e.newValue) {
    const n = parseInt(e.newValue.split(':')[0], 10);
    if (n !== cur) goTo(n, false);   // false = não re-broadcast (evita loop)
  }
});
```

> A chave de localStorage deve ser **`presenter_slide`** (genérica) — a mesma usada no `apresentador-template.html`. Usar a MESMA chave nos dois arquivos, senão o sync quebra.

**Exemplo do patch no padrão `.slide` + `goTo` (caso default):**

```js
// ANTES
function goTo(n) {
  slides[cur].classList.remove('active');
  cur = Math.max(0, Math.min(total - 1, n));
  slides[cur].classList.add('active');
  // ...atualizações de UI (progresso, contador)...
}

// DEPOIS (patcheado)
function goTo(n, broadcast=true) {
  slides[cur].classList.remove('active');
  cur = Math.max(0, Math.min(total - 1, n));
  slides[cur].classList.add('active');
  // ...atualizações de UI...
  if (broadcast) localStorage.setItem('presenter_slide', cur + ':' + Date.now());
}

window.addEventListener('storage', e => {
  if (e.key === 'presenter_slide' && e.newValue) {
    const n = parseInt(e.newValue.split(':')[0], 10);
    if (n !== cur) goTo(n, false);
  }
});
```

### Passo 3 — Parsear o roteiro Markdown

Converter o `roteiro.md` num array `SCRIPTS[]` de `{t: "título", html: "..."}`. Regras de parsing (baseadas no formato que funcionou):

| Markdown | Vira |
|----------|------|
| `## SLIDE N — TÍTULO` (ou `## SLIDE N - TÍTULO`) | Novo item; `t` = título após o `—` |
| `> *[texto]*` ou `*[texto]*` (linha de deixa) | `<span class="cue">[texto]</span>` (dourado, itálico) |
| `**texto**` | `<strong>texto</strong>` |
| Linha de parágrafo normal | `<p>...</p>` |
| `---` | separador, ignorar |
| Frontmatter / cabeçalho (`# 🎙️ ...`, `**Data:**`, `**Tom:**`) antes do 1º `## SLIDE` | ignorar (metadados) |

- A ordem dos slides em `SCRIPTS[]` deve casar com a ordem dos `.slide` no HTML. Validar que `SCRIPTS.length === TOTAL`. Se divergir, avisar o usuário (slide sem fala ou fala sem slide) e seguir com o que casar por índice.
- Aspas de abertura/fechamento (`"`) podem ser mantidas como estão.

### Passo 4 — Gerar o apresentador

Usar `assets/apresentador-template.html` como base. Substituir:
- `__TITLE__` → título da apresentação (ex: nome/data).
- `__SLIDES_FILE__` → `slides-{nome}.html` (o src dos dois iframes).
- `__SCRIPTS_ARRAY__` → o array `SCRIPTS[]` parseado no Passo 3.
- `__SLIDE_SELECTOR__` → `.slide` (ou o seletor detectado no Passo 1).

O template já inclui: layout grid (slide atual grande + próximo + roteiro), relógio, cronômetro (Iniciar/Pausar/Zerar), dots de navegação 1..N, teclado (setas, Home, End), sync via `storage`, e controle dos iframes por `classList.toggle('active')`.

Salvar como `apresentador-{nome}.html` na pasta de saída.

### Passo 5 — Abrir e instruir

1. Abrir ambos: `open apresentador-{nome}.html && open slides-{nome}.html`.
2. Dar as instruções de uso (2 monitores):
   - `slides-{nome}.html` → monitor da audiência → tela cheia (F) → compartilhar SÓ essa janela no Zoom.
   - `apresentador-{nome}.html` → seu monitor → suas falas + cronômetro.
   - Navegar pelas setas; as janelas sincronizam. Mesmo navegador nas duas.
   - Se o sync falhar, cada janela navega independente (fallback).

## Anti-patterns

- ❌ Recriar os slides do zero quando o usuário já forneceu (só copiar + patch).
- ❌ Assumir `.slide`/`goTo` sem verificar com grep — se não bater, detectar/perguntar.
- ❌ Inventar falas que não estão no .md.
- ❌ Loop de broadcast (sempre passar `broadcast=false` no listener de `storage`).
- ❌ Usar chaves de localStorage diferentes entre os dois arquivos (quebra o sync).

## Prompt gerador (logística reversa)

`prompt-gerador.md` (nesta pasta) contém um prompt pronto para colar em QUALQUER LLM. Ele gera slides.html + roteiro.md já no formato que esta skill consome. Quando o usuário pedir "o prompt pra gerar apresentação em outra IA", entregar o conteúdo de `prompt-gerador.md`.

## Futuro (extensão)

Se o usuário NÃO enviar slides/roteiro: oferecer gerar os slides (HTML no padrão `.slide` + `goTo`) e o roteiro (.md no formato `## SLIDE N — TÍTULO`) a partir de um tema/briefing — usando o mesmo contrato de `prompt-gerador.md` — e então rodar o procedimento normal.
