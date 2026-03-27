# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PWA de rastreamento de ciclo menstrual e anticoncepcional. Aplicação 100% client-side, sem dependências npm — um único arquivo HTML com CSS e JS embutidos, hospedado no GitHub Pages. Dados armazenados exclusivamente no localStorage.

- **Repo**: `https://github.com/Raid112/ciclo-app.git`
- **Deploy**: GitHub Pages
- **Idioma da UI**: Português brasileiro

## Development

```bash
python -m http.server 8000
# ou: npx http-server
```

Não há build, lint, testes ou package.json.

## Architecture

**Arquivo único (`index.html`)** contendo HTML + CSS + JS.

### Telas
- **Calendário** — Visualização mensal com marcadores de período (intensidade) e pílula
- **Day Panel** — Overlay lateral para registrar dados do dia (período + pílula com horário)

### Módulos JS (funções no escopo global)
- `STORAGE_KEYS` — Constantes para chaves localStorage (`ciclo_data` + keys legadas para migração)
- `init()` → migração one-shot de JSONBin (se houver credenciais legadas) → `showApp()`
- `renderCalendar()` — Renderiza grid mensal com navegação mês a mês
- `openPanel()`/`closePanel()` — Painel lateral para entrada de dados do dia
- `mergeData()` / `mergeSymptoms()` — Merge inteligente, usado apenas na migração one-shot

### Persistência
- localStorage key `ciclo_data` — todos os dados do usuário
- localStorage key `ciclo_migrated` — flag indicando que migração de JSONBin já ocorreu
- Sem sync remoto — app é 100% local

### PWA
- `manifest.json` — Tema `#8b1a1a` (vermelho escuro), standalone
- `sw.js` — Cache `ciclo-v4`, estratégia cache-first

## Code Conventions

- Fontes: Cinzel (display), Raleway (body)
- Paleta: vermelho escuro (`--primary: #8b1a1a`, `--accent: #e74c3c`) sobre fundo `#0a0a0a`
- Período tem 3 intensidades visuais: light/medium/heavy com opacidades diferentes
- Ornamentos decorativos unicode (`✦ ─────── ✦`)
- Navegação de calendário por mês (prevMonth/nextMonth)
