# CLAUDE.md — Mapa de Territórios Rabele

## Projeto
- Mapa SVG de territórios do Paraná, arquivo único `index.html` (~3.800 linhas, HTML + CSS + JS inline)
- Produção: https://territorios-fleischmann.netlify.app
- Deploy = `git push` na `main` (Netlify publica automático, costuma levar < 1 min)
- Repo: github.com/evandroadada/mapa-fleischmann
- Arquivos soltos não versionados (não commitar): `CONTEXTO_PROJETO.md`, `cidades_coferpan.csv`, `geojs-41-mun.json`, `index_old.html`

## Estrutura do código (linhas aproximadas — confirmar com grep)
- CSS tema/vars `:root` e `body.dark` ~10–50 · layout `body` flex ~51 · `#mapa-wrapper` ~150
- `@media print` principal ~248 · estilos da legenda de print fora do media ~280
- `@media print` duplicado dentro do SVG (`svgTerritorios`) ~919 — **é o que vence** (inserido depois); alterar os dois
- Header/cards das nativas ~395–425 · menu ⚙ `#menu-config` ~384
- `popCidades` ~471 (nome → hab)
- `cidadesRabeleFleischHoje`, `cidadesFleischmannRabele`, `coferpanExcluir` ~875–877
- `svgTerritorios` ~900 (polígonos `.mun` com `fill` base + `data-cidade`) · `<g id="labels">` ~1338 (54 labels fixos)
- `allMuns` ~1402 · `gerarLabelsCidadesGrandes()` ~1406
- Impressão: `abrirMenuImpressao()` ~1482 (sempre sintético) · `imprimirMapa(modo)` ~1492 · `gerarLegendaPrint()` ~1538
  · `medirLegendaPrint()` / `reservarEspacoLegendaPrint()` ~1667 · constantes `PRINT_*` ~1660
- Excel: `exportarExcelCustom()` ~1682 · `exportarExcel()` ~1746
- `toggleTheme()` ~1812 · `window load` (injeta SVG, allMuns, labels, overrides, carrega visões) ~1820
- Pintura: `resetCores()` ~1922 · `destacar()` ~1954 · `destacarPorChave()` ~1969 · `destacarCustom()` ~1988
  · `renderLabelsVendedor()` ~2008 (labels de região `.vendedor-label`)
- `SEL_PALETTE` ~2347
- Supabase ~2380: `SUPABASE_URL`, `SB_REST`, `supabaseSalvarVisao` ~2412, `supabaseExcluirVisao` ~2432,
  `supabaseCarregarVisoes` ~2445, `supabaseMigrarFixas` ~2474, nativas: `aplicarOverridesNativas` ~2513,
  `carregarNativasSupabase` ~2545
- Ordem dos cards: `salvarOrdemCards` ~2570 · `carregarOrdemCards` ~2601 · `ativarReordenar` ~2621
- localStorage: `salvarVisoesLS` ~2683 · `carregarVisoesLS` ~2700 (Supabase → fallback LS/fixas)
- Clone fixo: `CLONE_ID` / `garantirCloneRabelePotencial()` ~2758 (não alterar)
- `VISAO_COR` / `VISAO_NOME` ~2810 · `pedirSenha()` ~2831
- Menu ⚙: `abrirMenuConfig` ~2853 · `configNovaVisao` / `configEditarRegioes` / `configReordenar` ~2864 · `configClonarVisao` ~2869
- Seletor (Editar Regiões): `abrirSeletorNovaVisao` ~2989 · `abrirSeletor` ~3015 · `seletorConstruirSVG` ~3081
  · `seletorCarregarVisao` ~3141 · `seletorPaint` ~3185 · `seletorBuscar` ~3347 · `seletorSalvarEAplicar` ~3448
  · `seletorCriarNovaVisao` ~3516 · `seletorAplicarAoMapa` ~3696
- Cards: `ajustarContrasteCard` ~3570 · `criarCardNovaVisao` ~3600 (clique ativa, dblclick exclui custom)

## Modelo de dados
- Visão ativa: `travadoTerr` (chave ou null)
- Nativas (chave fixa): `coferpan`, `fermisul`, `rabele-fleisch-hoje`, `rabele`, `fleisch-rabele`, `coferpan-apos`
- Custom: chave `custom_<timestamp>` → `window._customVisoes[chave] = { nome, regioes: [{nome, cor, cids:Set, pop, meta, tipo}] }`
  - `tipo`: `'vendedor'` | `'naoatendida'`
  - qualquer lógica por prefixo `custom_` vale para clones e novas visões
- Nomes/cores globais: `VISAO_NOME[chave]`, `VISAO_COR[chave]`
- Overrides das nativas: `window._overrideCoferpanAdd`, `_overrideAposAdd`, `_overrideFermisul{add,rem}`, `_overrideRabele{add,rem}`, `_overrideApos`

## Regras que não podem quebrar
- Nunca `setAttribute('fill')` em polígono — pintar só via `destacarPorChave` / `destacarCustom` (usam `style.fill`)
- `fill` do polígono = cor base da nativa; predicados de `destacarPorChave` dependem dele
- Cidades com apóstrofo (ex: PÉROLA D'OESTE) usam aspas duplas em strings JS
- `coferpanExcluir` = PITANGA (fill azul `#3498DB`) e GUAMIRANGA (fill base `#85C1A0`): fora do Coferpan Total Hoje
- Layout: `body` flex column 100dvh, `#mapa-wrapper` `flex:1 1 auto; min-height:0` — nunca altura fixa
- `destacarPorChave` remove todos os `.vendedor-label` antes de pintar (nativa não tem label de região)
- `seletorBuscar`: chama `seletorPaint()` antes e destaca o match com stroke dourado
  - ⚠ o match ainda **não** sobe no z-order (pendente)
- Labels de cidade:
  - 54 fixos em `#labels` (26 são polos < 50k escolhidos à mão — não remover)
  - `gerarLabelsCidadesGrandes()` cria no load os ≥ 50k que faltam (marcados `data-label-dinamico`)
  - estilo: ≥ 200k 9/700, senão 7/500, `#222`, Arial, halo branco
- Labels de região (`renderLabelsVendedor`): desviam de nome de cidade (+9/+16, depois −9/−16), stroke 1.2/1.0, letter-spacing 0.35, quebra no "/"
- `SEL_PALETTE`: 10 cores, ≥ 3:1 contra branco **e** `#222`, ΔE ≥ 31 entre si e do azul Rabele `#3498DB`; sem pastel/neon
- Cores salvas nas regiões nunca migram automaticamente (usuário re-escolhe em Editar Regiões)
- Número do card: clareado em HSL (`ajustarContrasteCard`), alvo 4.0:1, teto +15 L, piso 3.0:1; a barra mantém a cor original
- Impressão:
  - nomes de cidade sempre visíveis (classe `print-ocultar-cidades` existe mas não é aplicada)
  - sintético oculta `.vendedor-label`
  - fundo branco forçado nos dois temas
  - legenda mede altura real (var `--print-leg-h`, mín. 290 / 130 curta); vira 2–3 colunas se não couber
  - `@page { size: A4 landscape; margin: 8mm }`
- Menu ⚙ pede senha (`pedirSenha`) antes de abrir; opções novas entram no mesmo menu
- `garantirCloneRabelePotencial()` / `CLONE_ID`: não alterar

## Como testar
- Servidor local: `python -m http.server 8765` → `http://localhost:8765/index.html?v=N` (trocar `N` p/ furar cache)
- Syntax check dos `<script>` inline:
  `node -e "const h=require('fs').readFileSync('index.html','utf8');const re=/<script>([\s\S]*?)<\/script>/g;let m;while((m=re.exec(h)))new Function(m[1]);console.log('ok')"`
- Supabase é produção compartilhada (localhost também grava lá):
  - testes que criam/excluem visões: `getSB = () => null` no console antes (desliga writes; LS vira fonte local)
  - teste real de persistência: aguardar a promise de `supabaseSalvarVisao` antes do reload e **apagar os dados de teste depois**
- `pedirSenha` usa `prompt()` e bloqueia automação → chamar direto `abrirSeletor()`, `configClonarVisao()` etc.
- `alert/confirm` também bloqueiam → sobrescrever `window.alert` / `window.confirm` no console
- Print: diálogo real bloqueia automação. Simulação:
  - copiar regras `@media print` de `document.styleSheets` para um `<style>` de tela
  - sobrescrever `window.print`; reaplicar `document.body.className` capturado no print (o código remove as classes após 500 ms)
  - página A4: `body{transform:translateZ(0);width:1062px;height:733px;overflow:hidden}` (fixed passa a ser relativo ao body)
- Resize de janela não muda o viewport nesta máquina — usar a simulação acima
- Checklist mínimo antes de push:
  - 9 visões × tema claro/escuro (sem label vazando, card legível)
  - print sintético e analítico (custom com muitas regiões + nativa)
  - Editar Regiões abre, salva, recarrega
- Produção após push: poll até o HTML novo conter um marcador do commit, depois abrir com `?nc=<hash>`

## Dados
- Supabase: `https://duzvlebxyrnksfvcsuae.supabase.co`, tabela `visoes` (REST `/rest/v1/visoes`), anon key em `SUPABASE_ANON`
  - linha = `{ id, nome, regioes (json), atualizado_em }`, upsert com `Prefer: resolution=merge-duplicates`
  - ids especiais (não viram card): `nativas_overrides` (`NATIVAS_ID`), `card_order` (`ORDEM_ID`)
- localStorage:
  - `rabele_visoes_custom_v1` (`LS_VISOES_KEY`) — backup das custom
  - `rabele_nativas_overrides_v1` — edições das nativas (aplicadas síncrono no load)
  - `rabele_card_order_v1` — ordem dos cards
  - `flTheme` — `'dark'` | `'light'`
- Ids especiais de visão:
  - `CLONE_ID = 'custom_clone_rabele_potencial'`
  - `ORIGEM_CLONE_ID = 'custom_1781613887310'` (Rabele Potencial)
  - `custom_1789823054105` = Rabele Expensão imediata
- Ordem de carga: overrides LS → `carregarVisoesLS()` (Supabase → fallback) → `carregarOrdemCards()` após 300 ms → `carregarNativasSupabase()`
