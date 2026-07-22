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
- Views atuais: `presentation`, `tracking`, `manifesto`, `documentacao`.
- **Para adicionar uma aba:** botão `#btn-nav-XYZ` com `onclick="window.switchView('XYZ')"`,
  um `<main id="view-XYZ" class="... view-enter hidden">`, e inclua `'XYZ'` no array
  `views` de `switchView`.
- **URLs por ambiente (roteamento por hash):** cada view tem rota própria — `#/pdi`
  (presentation), `#/cronograma` (tracking), `#/manifesto` e
  `#/documentacao/<discovery|executiva|ia>`. O roteador vive no **script comum** (o da
  Documentação): `window.updateRouteHash` grava a rota (chamado por `switchView` e
  `switchTab`) e `window.applyRoute` lê o hash no `hashchange`; a rota inicial é
  aplicada pela inicialização do módulo principal (que chama `applyRoute` quando há
  hash na URL). Uma URL sem hash permanece limpa até a primeira navegação.
  Ao criar uma view nova, adicione o slug em `routeToView`/`viewToRoute`; sub-abas de
  documentação entram automaticamente ao serem registradas em `decks`.

### Aba Documentação (integração nativa)

- Conteúdo trazido do repo `documenta_pdi_armando` e **embutido nativamente** (não é
  iframe — a versão com iframe foi removida por gerar uma segunda navbar/"puxadinho").
- Vive dentro de `view-documentacao`, começando por um **seletor inline de sub-abas**
  (`#tab-btn-discovery` / `#tab-btn-executiva`) — não é uma segunda navbar fixa.
- Dois "decks": `discovery` e `executiva`. Cada slide é um objeto em `discoverySlides` /
  `executivaSlides` (`badge`, `themeKey`, `title`, `intro`, `content` HTML, `reference`).
- Funções globais do motor: `switchTab`, `startDeck`, `backToIntro`, `switchDeckView`,
  `renderDeck`, `updateNavButtons`, `deckNext`, `deckPrev`. Estado em `decks` e `activeTab`.
- Navegação por **teclado (← →)** e **swipe** só age quando `view-documentacao` está
  visível (há um guard explícito). Preserve esse guard ao mexer nos handlers.
- **Para editar/adicionar um slide:** altere o array correspondente. Ao trocar a
  quantidade de slides, os contadores (ex.: `01/10`) se ajustam sozinhos.

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

- **Idioma:** todo o conteúdo e os comentários são em **pt-BR**. Mantenha.
- **Estilo:** classes utilitárias do Tailwind inline. Paleta do tema:
  `paper`, `ink`, `accent`, `danger`, `neutral`, `brand.{orange,blue,pink}`.
  Reaproveite essas cores em vez de introduzir hex avulsos.
- **Ícones:** Font Awesome (`fa-solid` / `fa-regular`).
- **Sem dependências novas de build.** Continue tudo em `index.htm`; use CDN quando
  precisar de algo externo (como já é feito com Tailwind/FA/Firebase).
- Todas as funções chamadas por `onclick` no HTML precisam ser **globais**
  (`window.*` no módulo, ou `function` no script comum).

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
