# PDI Armando — AUVP

Página única (SPA) que apresenta o **Plano de Desenvolvimento Individual (PDI)** do
pirata Armando, do time de Produto/Design da AUVP. Reúne, em um só lugar, o plano de
desenvolvimento, o cronograma de acompanhamento, o manifesto de design e a
documentação tática (playbooks de Discovery e de Defesa de Design).

🔗 **Publicação:** GitHub Pages — https://produtosauvp.github.io/pdi-armando/

---

## Sumário do repositório

| Arquivo | Descrição |
| --- | --- |
| `index.htm` | Aplicação completa (single-page). HTML + Tailwind (via CDN) + JavaScript inline. É o **único artefato de deploy**. |
| `meme_this_is_fine_dog.png` | Imagem usada como favicon (referenciada por URL absoluta do GitHub raw). |
| `README.md` | Este documento. |
| `CLAUDE.md` | Guia de arquitetura e convenções para o Claude Code / contribuintes. |

> ⚠️ O site é servido a partir de `index.htm` na **raiz** do repositório. Não mova nem
> renomeie esse arquivo (nem o PNG do favicon) sem atualizar as referências — isso
> quebraria a publicação no GitHub Pages.

---

## Estrutura da aplicação

Toda a experiência vive em `index.htm` e é organizada como um SPA por *views*, alternadas
via JavaScript (sem recarregar a página). O menu superior fixo (única navbar do sistema)
controla as quatro abas principais:

| Aba (menu) | `id` da view | Conteúdo |
| --- | --- | --- |
| **PDI** | `view-presentation` | Plano de desenvolvimento no modelo 70‑20‑10, com as 4 dimensões e TAGS de rastreamento. |
| **Cronograma** | `view-tracking` | Roadmap por mês/semana, com checklist persistente e filtros por categoria. |
| **Manifesto** | `view-manifesto` | Manifesto de design (bom designer × mau designer), injetado via JS. |
| **Documentação** | `view-documentacao` | Playbooks interativos de **Discovery** e **Apresentação Executiva** (decks de slides). |

A aba **Documentação** foi integrada a partir do repositório
[`documenta_pdi_armando`](https://github.com/ProdutosAUVP/documenta_pdi_armando) e roda
nativamente dentro do SPA (mesma navbar, mesmo tema). Internamente ela tem um seletor
de sub-abas (Discovery / Executiva) e um motor de decks com navegação por botões,
teclado (setas ← →) e swipe no mobile.

Além das views, há um **assistente estilo Clippy** (canto inferior direito) com um chat
de dicas sobre o PDI.

---

## Stack técnica

- **HTML único** (`index.htm`), sem build step.
- **Tailwind CSS** via CDN (`cdn.tailwindcss.com`) com `tailwind.config` inline
  (tema `darkMode: 'class'`, paleta editorial e keyframes/animações customizadas).
- **Font Awesome** via CDN (ícones da aba Documentação).
- **Fonte Satoshi** via Fontshare.
- **Firebase** (opcional) — Auth anônimo + Firestore para **persistir o progresso do
  cronograma** entre dispositivos. Sem configuração de Firebase, o progresso cai
  automaticamente para o `localStorage` (chave `pdi_armando_v9`).

Como tudo depende de CDNs externas, é necessário acesso à internet para renderizar a
página com estilo completo.

---

## Rodando localmente

Não há dependências para instalar. Basta servir a pasta por HTTP (o `<iframe>` foi
removido, mas ainda é recomendável usar um servidor para o Firebase/fontes):

```bash
# a partir da raiz do repositório
python3 -m http.server 8000
# abra http://localhost:8000/index.htm
```

Abrir o arquivo direto com `file://` também funciona para a maior parte da UI, mas
recursos externos (CDNs, Firebase) exigem rede.

---

## Deploy

O deploy é automático via **GitHub Pages** a partir da branch padrão. Qualquer alteração
mesclada em `main` publica o novo `index.htm`.

---

## Como adicionar uma nova aba no menu superior

1. Adicione um `<button id="btn-nav-XYZ" onclick="window.switchView('XYZ')">` na navbar.
2. Crie o container `<main id="view-XYZ" class="... view-enter hidden">` com o conteúdo.
3. Inclua `'XYZ'` no array `views` dentro de `window.switchView(...)`.

O `switchView` cuida de esconder as demais views e de destacar o botão ativo.
