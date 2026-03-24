# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PWA de rastreamento de ciclo menstrual e anticoncepcional. Aplicação 100% client-side com sync remoto via JSONBin.io, sem dependências npm — um único arquivo HTML (~1138 linhas) com CSS e JS embutidos, hospedado no GitHub Pages.

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
- **Setup Screen** — Configuração inicial de credenciais JSONBin (Master Key, Access Key, Bin ID)
- **Calendário** — Visualização mensal com marcadores de período (intensidade) e pílula
- **Day Panel** — Overlay lateral para registrar dados do dia (período + pílula com horário)

### Módulos JS (funções no escopo global)
- `STORAGE_KEYS` — Constantes para chaves localStorage (`ciclo_data`, `ciclo_master_key`, `ciclo_access_key`, `ciclo_bin_id`, `ciclo_last_sync`)
- `init()` → verifica credenciais JSONBin → `showApp()` ou exibe setup
- `renderCalendar()` — Renderiza grid mensal com navegação mês a mês
- `openPanel()`/`closePanel()` — Painel lateral para entrada de dados do dia
- `syncToRemote()`/`syncFromRemote()` — Sync bidirecional com JSONBin.io
- `mergeData()` — Merge inteligente de dados locais/remotos (prioriza dado mais completo, resolve conflito de período por nível mais alto)

### Sync remoto (JSONBin.io)
- Indicador visual de status: synced (verde), syncing (amarelo), error (vermelho)
- Cache-first no service worker, mas exclui chamadas a `jsonbin.io`
- Credenciais armazenadas no localStorage (não hardcoded)

### PWA
- `manifest.json` — Tema `#8b1a1a` (vermelho escuro), standalone
- `sw.js` — Cache `ciclo-v1`, estratégia cache-first, exclui jsonbin.io

## Code Conventions

- Fontes: Cinzel (display), Raleway (body)
- Paleta: vermelho escuro (`--primary: #8b1a1a`, `--accent: #e74c3c`) sobre fundo `#0a0a0a`
- Período tem 3 intensidades visuais: light/medium/heavy com opacidades diferentes
- Ornamentos decorativos unicode (`✦ ─────── ✦`)
- Navegação de calendário por mês (prevMonth/nextMonth)
