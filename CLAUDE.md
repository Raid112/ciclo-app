# CLAUDE.md — ciclo-app

## Project Overview

PWA de rastreamento de anticoncepcional em **regime contínuo (desogestrel, minipílula)** + diário de sangramento. Aplicação 100% client-side, sem dependências npm — um único arquivo HTML com CSS e JS embutidos, hospedado no GitHub Pages. Dados armazenados exclusivamente no localStorage.

> Histórico: o app nasceu para uma pílula combinada de cartela **24+4** (fase ativa/pausa, menstruação programada, previsão de ciclo, janela fértil). Em 29/05/2026 a usuária trocou para **desogestrel contínuo** (sem pausa). Todo o motor 24+4 foi removido — desogestrel não tem fase/pausa, é anovulatório e o sangramento é individual/imprevisível (amenorreia, escape ou irregular). Tolera atraso de só **12h** na tomada.

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
- **Calendário** — Visualização mensal: marca pílula (dot), sangramento (intensidade), atraso > 12h (dot âmbar) e janela de escape prevista (tracejado). Dados anteriores a 29/05 aparecem esmaecidos (`legacy-bleed`).
- **Day Panel** — Overlay para registrar o dia: pílula (com horário, sempre disponível — sem pausa), sangramento (Nenhum/Leve/Moderado/Intenso) e sintomas.
- **Status card** — Mês de uso, selo de adaptação, streak de aderência, classificação do padrão, previsão (quando houver) e alertas.

### Módulos JS (funções no escopo global)
- **Constantes tunáveis** (bloco no topo do `<script>`): `REGIME_START='2026-05-29'`, `PILL_TARGET='09:00'`, `DELAY_HOURS=12`, `ADAPTATION_DAYS=90`, `EWMA_ALPHA=0.4`, `CV_THRESHOLD=0.30`, `PROLONGED_DAYS=10`, `MIN_INTERVALS=3`. São chutes iniciais; só calibram com ~6 meses de dado real.
- `init()` → migração one-shot de JSONBin (se houver credenciais legadas) → `showApp()`.
- `renderCalendar()` — grid mensal; calcula `predictNextBleeding()` uma vez por render.
- `renderStatusCard()` — card de status (substituiu o antigo `renderCycleCard`).
- `openPanel()`/`closePanel()` — painel de entrada do dia.
- **Aderência**: `getPillDelayStatus()` (atraso vs `PILL_TARGET`), `getAdherence()` (streak, dias perdidos, janelas de risco; **dia corrente = pendente, não esquecido**).
- **Motor de padrão**: `getBleedingEpisodes()` (agrupa dias consecutivos ≥ `REGIME_START`), `getBleedingStats()`, `classifyPattern()` (amenorreia/esporádico/frequente/prolongado), `predictNextBleeding()` (EWMA dos intervalos + coef. de variação + gates; retorna `null` durante adaptação, amostra < 4 episódios, ou CV alto).
- `mergeData()` / `mergeSymptoms()` — merge usado apenas na migração one-shot.

### Persistência
- localStorage key `ciclo_data` — todos os dados do usuário. Estrutura por dia: `{ pill:{taken,time}, period:{level}, symptoms:{...} }`. O campo `period` é mantido (evita migração) mas agora representa **sangramento/escape**, não menstruação programada.
- localStorage key `ciclo_migrated` — flag de migração de JSONBin.
- **Separação de regime**: o motor de padrão só analisa registros com data ≥ `REGIME_START`; dados anteriores ficam como histórico.
- Sem sync remoto — app é 100% local.

### PWA
- `manifest.json` — Tema `#8b1a1a` (vermelho escuro), standalone.
- `sw.js` — Cache `ciclo-v6`, estratégia cache-first. **Bump do `CACHE_NAME` a cada release** do `index.html`.

## Code Conventions

- Fontes: Cinzel (display), Raleway (body)
- Paleta: vermelho escuro (`--primary: #8b1a1a`, `--accent: #e74c3c`) sobre fundo `#0a0a0a`; atraso de risco em âmbar (`#e6a23c`)
- Sangramento tem 3 intensidades visuais: light/medium/heavy com opacidades diferentes
- Ornamentos decorativos unicode (`✦ ─────── ✦`)
- Navegação de calendário por mês (prevMonth/nextMonth)
