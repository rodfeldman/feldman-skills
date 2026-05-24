# Presenter Skill — Instalação

Skill para Claude Code que gera duas páginas HTML sincronizadas a partir de slides + roteiro: uma limpa pra compartilhar no Zoom/Meet, outra com vista do apresentador (slide atual, próximo, falas, relógio, cronômetro).

## Pré-requisitos

- [Claude Code](https://claude.com/claude-code) instalado
- macOS, Linux ou Windows (WSL)

## Instalação (1 minuto)

1. Descompacte o zip
2. Mova a pasta `presenter/` inteira para a pasta de skills do Claude Code:

   **macOS / Linux:**
   ```bash
   mv presenter ~/.claude/skills/
   ```

   **Windows (PowerShell):**
   ```powershell
   Move-Item presenter $env:USERPROFILE\.claude\skills\
   ```

3. Pronto. Abra qualquer sessão do Claude Code e digite `/presenter` — a skill aparece.

## Como usar

Dentro do Claude Code:

```
/presenter
```

Ou descreva em linguagem natural:
- "criar apresentação pro Zoom com minhas falas"
- "duas telas: slides + roteiro"
- "presenter view"

A skill pede dois inputs:
1. **Slides** — um arquivo HTML com seus slides (qualquer framework: reveal.js, impress.js, HTML puro com `<section>`, etc.)
2. **Roteiro** — um Markdown com as falas, uma seção por slide

E gera dois arquivos:
- `slides-{nome}.html` — limpo, pra screen-share
- `apresentador-{nome}.html` — sua vista privada

Abra os dois no mesmo navegador, em janelas separadas. Mudar de slide numa sincroniza a outra (via `localStorage`, sem servidor).

## Estrutura da skill

```
presenter/
├── SKILL.md                      # instruções pro Claude Code
├── prompt-gerador.md             # template de prompt usado internamente
├── apresentador-template.html    # template da vista do apresentador
└── INSTALL.md                    # este arquivo
```

## Atualizando

Se receber uma versão nova, sobrescreva a pasta `~/.claude/skills/presenter/` inteira.

## Removendo

```bash
rm -rf ~/.claude/skills/presenter
```

## Autor

Criada por Rodrigo Feldman (Academia Lendária). Reportar problemas ou sugestões: pedagogico@academialendaria.ai
