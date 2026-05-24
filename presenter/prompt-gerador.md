# Prompt gerador de apresentação (para usar em qualquer LLM)

Cole este prompt em qualquer LLM (ChatGPT, Gemini, Grok, Claude). Ele gera os DOIS arquivos no formato exato que a skill `/presenter` consome: um `slides.html` no padrão `.slide` + `goTo`, e um `roteiro.md` no padrão `## SLIDE N — TÍTULO`.

Depois é só salvar os dois arquivos e mandar pro Claude Code: `/presenter` + os dois caminhos.

---

## O PROMPT (copie tudo abaixo)

```
Você vai criar uma apresentação em DOIS arquivos que serão usados juntos. Siga o CONTRATO DE FORMATO exatamente — outra ferramenta vai consumir esses arquivos automaticamente e qualquer desvio quebra a integração.

## CONTEXTO DA APRESENTAÇÃO
- Tema: [DESCREVA O TEMA AQUI]
- Público: [QUEM VAI ASSISTIR]
- Duração alvo: [EX: 15 minutos]
- Objetivo / call to action: [O QUE VOCÊ QUER QUE ACONTEÇA NO FINAL]
- Tom de voz: [EX: direto, acolhedor, líder de comunidade — sem pitch agressivo]
- Quantidade de slides: [EX: 12 a 15]
- Informações/dados que DEVEM aparecer: [LISTE FATOS, NÚMEROS, LINKS, NOMES]

## ARQUIVO 1 — slides.html
Gere um único arquivo HTML autocontido (sem dependências externas além de Google Fonts), seguindo ESTE CONTRATO técnico OBRIGATÓRIO:

1. Cada slide é uma `<div class="slide">`. O primeiro slide tem `class="slide active"`.
2. Todos os slides ficam dentro de um container `<div class="deck">`.
3. CSS: cada `.slide` ocupa a tela inteira (100vw x 100vh), `display:none` por padrão, e `.slide.active { display:flex }`. Centralize o conteúdo. Use `box-sizing:border-box`.
4. Inclua uma barra de progresso `<div id="prog"></div>` e um contador `<div id="sctr"></div>`.
5. Ao final, inclua EXATAMENTE este script de navegação (não altere os nomes goTo, slides, total, cur, prog, sctr):

```html
<script>
const slides = document.querySelectorAll('.slide');
const total  = slides.length;
let cur = 0;
function goTo(n) {
  slides[cur].classList.remove('active');
  cur = Math.max(0, Math.min(total - 1, n));
  slides[cur].classList.add('active');
  const p = document.getElementById('prog'); if(p) p.style.width = ((cur+1)/total*100) + '%';
  const s = document.getElementById('sctr'); if(s) s.textContent = (cur+1) + ' · ' + total;
}
document.addEventListener('keydown', e => {
  if(['ArrowRight','ArrowDown',' ','PageDown'].includes(e.key)) { e.preventDefault(); goTo(cur+1); }
  if(['ArrowLeft','ArrowUp','PageUp'].includes(e.key)) { e.preventDefault(); goTo(cur-1); }
  if(e.key==='Home') goTo(0);
  if(e.key==='End') goTo(total-1);
  if(e.key==='f'||e.key==='F'){ !document.fullscreenElement ? document.documentElement.requestFullscreen() : document.exitFullscreen(); }
});
goTo(0);
</script>
```

6. Design: visual profissional, alto contraste, tipografia grande (legível a 3 metros de distância numa chamada de Zoom). Pode variar fundos entre slides (claro/escuro). Não use animações que dependam de bibliotecas.
7. NÃO coloque as falas do apresentador dentro do HTML. O HTML é só o que a audiência vê.

## ARQUIVO 2 — roteiro.md
Gere um arquivo Markdown com o texto que EU vou FALAR em cada slide. Contrato OBRIGATÓRIO:

1. Um cabeçalho curto no topo (título, data, tom) — opcional, será ignorado pela ferramenta.
2. Para CADA slide, um bloco começando com: `## SLIDE N — TÍTULO` (N = número do slide, começando em 1; use travessão `—`). A ORDEM e a QUANTIDADE de blocos devem bater 1:1 com os slides do HTML.
3. O texto da fala em parágrafos normais (linguagem falada, primeira pessoa, como se eu estivesse conversando).
4. Deixas de palco (instruções pra mim, não pra falar em voz alta) entre colchetes em itálico: `*[Pausa de 2 segundos]*` ou `> *[Aguardar respostas no chat]*`. Use pra ações como "jogar link no chat", "abrir terminal", "esperar resposta".
5. Use `**negrito**` nas frases-chave que eu devo enfatizar.
6. Separe os blocos com `---`.

## REGRAS FINAIS
- A quantidade de `## SLIDE` no roteiro.md DEVE ser igual à quantidade de `<div class="slide">` no HTML.
- Entregue os dois arquivos em blocos de código separados e claramente rotulados (slides.html e roteiro.md).
- Não invente dados que eu não forneci; se faltar algo, use um placeholder visível tipo [PREENCHER].
```

---

## Como usar o output

1. Salve o primeiro bloco como `slides.html` e o segundo como `roteiro.md`.
2. No Claude Code: `/presenter` e informe os dois caminhos.
3. A skill gera `slides-{nome}.html` (compartilhar no Zoom) + `apresentador-{nome}.html` (suas falas), sincronizados.
