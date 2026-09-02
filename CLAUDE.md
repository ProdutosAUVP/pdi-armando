# CLAUDE.md

Guia para o Claude Code (e humanos) trabalharem neste repositório.

## O que é

`pdi-armando` é um **site de página única** publicado no GitHub Pages. Toda a aplicação —
markup, estilos e lógica — vive em **um único arquivo: `index.htm`**. Não há build,
bundler, framework, gerenciador de pacotes nem testes versionados. O que está no arquivo
é o que vai para produção.

## Arquivos

- `index.htm` — a aplicação inteira. **Único artefato de deploy.**
- `meme_this_is_fine_dog.png` — favicon (referenciado por URL **absoluta** do GitHub raw,
  não por caminho relativo).
- `README.md` — visão geral do produto.
- `CLAUDE.md` — este guia.

⚠️ **Não mova nem renomeie `index.htm` ou o PNG.** O GitHub Pages serve `index.htm` da
raiz e o favicon usa URL absoluta apontando para `main`. Reestruturar pastas quebra a
publicação.

## Arquitetura de `index.htm`

Ordem do arquivo:

1. `<head>` — meta, favicon, fontes (Satoshi/Fontshare), **Font Awesome** (CDN),
   **Tailwind** (CDN) + `tailwind.config` inline, e um `<style>` com CSS customizado.
2. `<body>` — assistente "Clippy" (fixo), a **navbar superior única** e as `<main>` de
   cada view.
3. `<script type="module">` — app principal (Firebase, dados do cronograma/manifesto,
   `window.switchView`, filtros, tema, assistente).
4. `<script>` (JS comum) — **motor da aba Documentação** (decks Discovery/Executiva).

### Sistema de views (abas do topo)

- Cada aba é um `<main id="view-...">`; todas começam com a classe `hidden`, exceto a
  inicial (`view-presentation`).
- `window.switchView(name)` esconde todas as views do array `views` e mostra a alvo,
  atualizando o destaque dos botões `#btn-nav-*` num laço.
- Views atuais: `presentation`, `tracking`, `resultados`, `manifesto`, `documentacao`.
- **Para adicionar uma aba:** botão `#btn-nav-XYZ` com `onclick="window.switchView('XYZ')"`,
  um `<main id="view-XYZ" class="... view-enter hidden">`, e inclua `'XYZ'` no array
  `views` de `switchView`.

### Rotas / URLs compartilháveis

Cada parte do site tem a sua própria URL, via **hash** (`#/...`) — não há servidor,
então nada de rotas com `history.pushState` puro. Formato:

| URL | Abre |
| --- | --- |
| `#/pdi` | Aba PDI |
| `#/cronograma` | Cronograma (todas as categorias) |
| `#/cronograma/<categoria>` | Cronograma filtrado (`ope`, `per`, `inf`, `cul`, `outros`) |
| `#/resultados` | Resultados quinzenais (capa da apresentação mais recente) |
| `#/resultados/<data>` | Capa da apresentação de uma data (ex.: `2026-09-02`) |
| `#/resultados/<data>/<slide>` | Apresentação aberta no slide (`abertura`, `key` da entrega ou `fechamento`) |
| `#/manifesto` | Manifesto |
| `#/documentacao/<sub-aba>` | Capa do deck (`discovery`, `executiva`, `ia`) ou o Clarity |
| `#/documentacao/<deck>/<n>` | Deck aberto no slide `n` (1-based) |
| `#/documentacao/clarity/<mês>` | Clarity no mês registrado (ex.: `2026-07`) |
| `#/documentacao/clarity/<mês>/<produto>` | Clarity no mês + produto (`key` do produto) |
| `#/documentacao/testes-ab` | Propostas de teste A/B (todas as páginas) |
| `#/documentacao/testes-ab/<página>` | Proposta de uma página só (`key` do teste) |

- Motor no script comum: `buildRouteHash()`, `syncRoute(push)`, `applyRoute()`
  (exposto como `window.applyRoute`). Também aceita os ids internos das views
  (`#/tracking` = `#/cronograma`) e normaliza rotas inválidas.
- **Trocar de aba** entra no histórico (`pushState`); navegar **dentro** de uma aba
  (slide, mês, produto, filtro) usa `replaceState`, para o botão voltar não ter que
  desfazer slide a slide. Chamadas do mesmo gesto são agrupadas numa entrada só.
- `syncRoute` só age depois da primeira leitura da URL (`routerReady`) — por isso
  renders de inicialização não sobrescrevem a rota recebida.
- Quem chama o roteador na carga é o `DOMContentLoaded` do **módulo principal**
  (`window.applyRoute()` no lugar do antigo `switchView('presentation')`). Se mexer
  nessa inicialização, mantenha a chamada, senão links diretos param de abrir.
- `window.switchView` e `window.setFilter` (do módulo) são **envolvidos** pelo roteador
  no `DOMContentLoaded` do script comum para manter a URL em dia. Ao editar essas
  funções, preserve a assinatura.
- O botão de corrente 🔗 na navbar (`window.copyCurrentLink`) copia a URL da parte
  aberta e mostra o aviso `#link-toast`.

### Aba Documentação (integração nativa)

- Conteúdo trazido do repo `documenta_pdi_armando` e **embutido nativamente** (não é
  iframe — a versão com iframe foi removida por gerar uma segunda navbar/"puxadinho").
- Vive dentro de `view-documentacao`, com um **menu lateral de sub-abas**
  (`#tab-btn-discovery`, `#tab-btn-executiva`, `#tab-btn-ia`, `#tab-btn-clarity`,
  `#tab-btn-testes-ab`) —
  não é uma segunda navbar fixa. Os mesmos itens aparecem no dropdown da navbar
  (`#doc-dropdown`, via `window.openDoc('<tab>')`).
- Três "decks" de slides: `discovery`, `executiva` e `ia`. Cada slide é um objeto em
  `discoverySlides` / `executivaSlides` / `iaSlides` (`badge`, `themeKey`, `title`,
  `intro`, `content` HTML, `reference`).
- Funções globais do motor de decks: `switchTab`, `startDeck`, `backToIntro`,
  `switchDeckView`, `renderDeck`, `updateNavButtons`, `deckNext`, `deckPrev`.
  Estado em `decks` e `activeTab`.
- Navegação por **teclado (← →)** e **swipe** só age quando `view-documentacao` está
  visível **e** a sub-aba ativa tem um `#<tab>-view-slides` visível (há um guard
  explícito). Preserve esse guard ao mexer nos handlers — é ele que impede a aba
  Clarity (que não é deck) de reagir às setas.
- **Para editar/adicionar um slide:** altere o array correspondente. Ao trocar a
  quantidade de slides, os contadores (ex.: `01/10`) se ajustam sozinhos.

### Sub-aba Análise Clarity (registro mensal)

- Não é um deck: é um **arquivo recorrente** das leituras de mapa de calor do Clarity,
  separado por **mês** e, dentro do mês, por **produto/página**.
- Dados em `clarityReports` (script comum, junto do motor de decks): um objeto por mês
  (`id`, `month`, `year`, `focus`, `summary`, `products[]`), do **mais recente para o
  mais antigo**. Cada produto tem `key`, `name`, `page`, `icon` (Font Awesome),
  `themeKey` (paleta de `colorThemes`), `metrics[]` (opcional) e as listas `behaviors[]`
  e `conclusions[]`. Em qualquer item dessas listas, `note` é a anotação/hipótese do time
  (renderizada como bloco recuado) e `text` aceita HTML inline (`<strong>`, `<em>`).
- Render em `renderClarity()`; estado em `activeClarityMonth` / `activeClarityProduct`;
  interações globais `switchClarityMonth(id)` e `filterClarityProduct(key)`.
  Contadores da capa e do mês (meses, páginas, insights) são calculados a partir dos
  dados — não edite números na mão.
- **Para registrar um novo mês:** adicione um objeto no início de `clarityReports`.
  Nada mais precisa mudar: seletor de mês, filtros por produto e estatísticas se ajustam.

### Sub-aba Testes A/B (propostas de versão das páginas)

- Também não é deck: é o desdobramento do Clarity em **experimentos propostos**, um por
  página, na ordem sugerida de execução (a primeira do array é a primeira a rodar).
- Dados em `abTests` (script comum, logo abaixo do motor do Clarity). Cada objeto tem
  `key`, `name`, `page`, `icon`, `themeKey` (paleta de `colorThemes`), os chips
  `priority` / `effort` / `recommended`, `evidence[]`, `hypothesis` (`se`, `entao`,
  `porque`), `variants[]`, `metrics` (`primary`, `secondary[]`, `guardrail`) e `success`.
- Cada variante tem `id` (`A` = controle, `B` = variante), `tag`, `title`, `summary`,
  o esquema de tela `wireframe[]` e — na variante — `changes[]` (que reaproveita
  `clarityListHtml`, então aceita `note`).
- Cada bloco do `wireframe` aceita `label` (HTML inline), `state` (`mantido`,
  `ajustado`, `subiu`, `desceu`, `novo`, `removido`, `destaque` — estilos em
  `abBlockStates`), `size: 'lg'`, `alert` (rótulo vermelho do ponto quente) e
  `fold: true`, que desenha a linha da dobra logo abaixo do bloco.
- Render em `renderAbTests()`; estado em `activeAbTest`; interação global
  `switchAbTest(key)`. Contadores da capa e a legenda saem dos dados — não edite números
  na mão.
- **Para propor um novo teste:** adicione um objeto em `abTests`. Filtros, contadores e
  legenda se ajustam sozinhos.

### Aba Resultados Quinzenais (apresentação de slides com cronômetro)

- Aba do topo (`view-resultados`): arquivo recorrente das apresentações de resultados,
  marcado pela **data em que foi apresentada** — e rodado como **deck de slides**.
- Dois modos, como os decks da Documentação: a **capa** (`resultados-view-intro`, com
  seletor de data, resumo e roteiro clicável) e os **slides**
  (`resultados-view-slides`). `switchResultadosView('intro' | 'slides')` alterna.
- Dados em `apresentacoesQuinzenais` (script comum, logo acima do roteador): um objeto
  por apresentação (`id` no formato `AAAA-MM-DD` — é o que vai para a URL —, `date`,
  `label`, `period`, `focus`, `summary`, `deliveries[]`), do **mais recente para o mais
  antigo**.
- Cada entrega em `deliveries[]` tem `key`, `name`, `page`, `kind` (chip do tipo:
  Conceituação, Atualização, Materiais, Processo, Experimento…), `stage` (chip de
  estágio), `icon` (Font Awesome), `themeKey` (paleta de `colorThemes`), `summary`
  (HTML inline) e as três listas obrigatórias — `what[]` (o que foi feito), `why[]`
  (por que é um bom produto) e `revenue[]` (como pode gerar faturamento) — mais
  `next[]` (próximos passos), opcional.
- As listas reaproveitam `clarityListHtml`, então cada item aceita `text` com HTML
  inline (`<strong>`, `<em>`) e `note` como anotação recuada do time.
- **Roteiro dos slides:** `getResultadoSlides()` monta `abertura` + uma entrega por
  slide + `fechamento` (que agrega os `next[]` de todas as entregas). A `key` de cada
  slide é o que vai para a URL — não há número de slide na rota.
- Render: `renderResultados()` (capa) e `renderResultadoSlide()` (slide atual).
  Estado em `activeQuinzena` / `activeQuinzenaSlide`. Interações globais:
  `switchQuinzena(id)`, `startResultadosDeck(slideKey?)`, `backToResultadosIntro()`,
  `resultadosNext()`, `resultadosPrev()`, `goToResultadoSlide(i)` e
  `abrirEntregaQuinzena(key)`.
- **Cronômetro:** a apresentação tem **duração máxima fixa de 10 minutos**, na constante
  `RESULTADOS_LIMITE_MIN` (é o único lugar a mudar se o limite mudar). O alvo por slide
  é esse limite ÷ nº de slides (`resultadosSlideTargetMs()`), então muda sozinho quando
  a quinzena tem mais ou menos entregas. Estado em `resultadosTimer` (`running`,
  `totalMs`, `slideMs`). `tickResultadosTimer()` roda a cada 250 ms e **só acumula com
  os slides visíveis** (`resultadosSlidesVisible()`), então trocar de aba ou voltar para
  a capa segura o tempo sozinho. Trocar de slide zera o tempo do slide, não o total.
  Globais: `toggleResultadosTimer()` e `resetResultadosTimer()`; desenho em
  `renderResultadosTimer()`, que mostra total, tempo do slide e o **restante** dos 10
  minutos (laranja no último minuto, vermelho e com `+` depois de estourar).
- **Teclado e swipe:** os handlers globais checam `resultadosSlidesVisible()` **antes**
  do guard da Documentação — setas ← → navegam e **espaço pausa/retoma** o cronômetro
  (com `preventDefault`, então o espaço não rola a página durante a apresentação).
  Preserve essa ordem ao mexer nos handlers.
- **Para registrar uma nova quinzena:** adicione um objeto **no início** de
  `apresentacoesQuinzenais`. Seletor de data, roteiro, slides, contadores e o alvo por
  slide do cronômetro se ajustam sozinhos.

### Tema (dark/light)

- Tailwind `darkMode: 'class'`; o tema é a classe `dark` no `<html>`.
- `window.toggleTheme` alterna o tema. **Existe apenas uma** definição de `toggleTheme`
  (a do módulo principal). O `toggleTheme` que vinha do documenta foi removido na
  integração — não reintroduza um segundo.

### Persistência (cronograma)

- Se as variáveis globais `__firebase_config` / `__app_id` / `__initial_auth_token`
  existirem (ambiente estilo Canvas), usa **Firebase** (auth anônimo + Firestore) para
  salvar o progresso do checklist entre dispositivos.
- Sem Firebase, cai para `localStorage` (chave `pdi_armando_v9`).
- Dados que dirigem a UI: `roadmapTasks` (cronograma) e o conteúdo do manifesto,
  injetados por `window.renderRoadmap` / `window.renderManifesto`.

## Convenções

- **Idioma:** **tudo em pt-BR** — conteúdo do site, comentários de código, mensagens de
  commit e, obrigatoriamente, **títulos, descrições e comentários de pull request**.
  Nenhum PR deve ser aberto (ou respondido) em inglês. Mantenha.
- **Estilo:** classes utilitárias do Tailwind inline. Paleta do tema:
  `paper`, `ink`, `accent`, `danger`, `neutral`, `brand.{orange,blue,pink}`.
  Reaproveite essas cores em vez de introduzir hex avulsos.
- **Ícones:** Font Awesome (`fa-solid` / `fa-regular`).
- **Sem dependências novas de build.** Continue tudo em `index.htm`; use CDN quando
  precisar de algo externo (como já é feito com Tailwind/FA/Firebase).
- Todas as funções chamadas por `onclick` no HTML precisam ser **globais**
  (`window.*` no módulo, ou `function` no script comum).

## Git e Pull Requests

- **Todo PR é em pt-BR** — título, corpo, checklists e respostas a revisões. Se um
  template de PR vier em inglês, preencha o conteúdo em pt-BR mesmo assim.
- Título curto e no imperativo, descrevendo o que muda para quem usa o site
  (ex.: `Adiciona sub-aba "Análise Clarity" na Documentação`).
- No corpo, explique **o que mudou** e **por quê**; como o deploy é direto para o
  GitHub Pages, cite o que precisa ser conferido no navegador após o merge.
- Mensagens de commit seguem a mesma regra de idioma.

## Testando alterações

Não há suíte de testes no repositório. Para validar comportamento (troca de abas, decks,
tema), sirva localmente e verifique no navegador:

```bash
python3 -m http.server 8000   # http://localhost:8000/index.htm
```

Como Tailwind/Font Awesome/Firebase vêm de CDNs, ambientes **sem acesso a essas CDNs**
renderizam a página sem estilo e podem quebrar o `<script type="module">` (o import do
Firebase falha e impede a definição de `window.switchView`). Isso é artefato de rede do
ambiente, não bug do código. Se precisar testar a lógica de forma isolada nesse cenário,
faça stub das CDNs (Tailwind e Firebase) antes de carregar a página.

## Deploy

Merge em `main` → publicação automática no GitHub Pages. Não há passo manual.
