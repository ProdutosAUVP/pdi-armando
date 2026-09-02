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
controla as cinco abas principais:

| Aba (menu) | `id` da view | Conteúdo |
| --- | --- | --- |
| **PDI** | `view-presentation` | Plano de desenvolvimento no modelo 70‑20‑10, com as 4 dimensões e TAGS de rastreamento. |
| **Cronograma** | `view-tracking` | Roadmap por mês/semana, com checklist persistente e filtros por categoria. |
| **Resultados** | `view-resultados` | Apresentação quinzenal de resultados em **slides, com cronômetro de 10 minutos**, arquivada por data: cada entrega com o que foi feito, por que é um bom produto e como pode gerar faturamento. |
| **Manifesto** | `view-manifesto` | Manifesto de design (bom designer × mau designer), injetado via JS. |
| **Documentação** | `view-documentacao` | Playbooks interativos de **Discovery**, **Apresentação Executiva** e **Bons Usos de IA** (decks de slides), o arquivo mensal de **Análise Clarity** e as propostas de **Testes A/B**. |

A aba **Documentação** foi integrada a partir do repositório
[`documenta_pdi_armando`](https://github.com/ProdutosAUVP/documenta_pdi_armando) e roda
nativamente dentro do SPA (mesma navbar, mesmo tema). Internamente ela tem um menu
lateral de sub-abas e um motor de decks com navegação por botões, teclado (setas ← →)
e swipe no mobile.

### Links diretos

Toda parte do site tem URL própria, então dá para mandar o link exato do que se quer
mostrar (o botão 🔗 na navbar copia o link da tela aberta):

| Link | Abre |
| --- | --- |
| `.../#/pdi` | Aba PDI |
| `.../#/cronograma/per` | Cronograma filtrado pela categoria Performance |
| `.../#/resultados/2026-09-02` | Capa da apresentação de resultados de 02/09/2026 |
| `.../#/resultados/2026-09-02/quiz-etfs` | Mesma apresentação, aberta no slide do quiz de ETFs |
| `.../#/manifesto` | Manifesto |
| `.../#/documentacao/discovery/4` | Playbook de Discovery no slide 4 |
| `.../#/documentacao/clarity/2026-07/pro` | Análise Clarity de julho, produto AUVP PRO |
| `.../#/documentacao/testes-ab/escola` | Proposta de teste A/B da página da AUVP Escola |

O botão **voltar** do navegador desfaz a troca de aba; navegar dentro de uma aba
(slide, mês, filtro) só atualiza a URL.

A sub-aba **Análise Clarity** foge do formato de deck: é um registro **recorrente
mensal** das leituras de mapa de calor das nossas páginas, com um seletor de mês e
filtro por produto. Cada página analisada traz os *comportamentos do usuário*
observados e as *principais conclusões*, com as anotações do time em destaque.

A sub-aba **Testes A/B** é a continuação natural dessa leitura: cada gargalo registrado
no Clarity vira uma proposta fechada de experimento — evidência, hipótese (se / então /
porque), o **controle (A)** e a **variante (B)** com esquema de tela lado a lado
(incluindo a linha da dobra), as mudanças propostas e como medir (métrica primária,
secundárias, métrica de proteção e critério de sucesso). É o material de apoio da ação
**PER.1** do cronograma.

A aba **Resultados** é o registro **quinzenal** das entregas, rodado como
**apresentação de slides**: a capa traz o seletor de data e o roteiro, e o botão
*Iniciar apresentação* abre o deck — abertura, um slide por entrega e o fechamento com
os próximos passos. Cada entrega responde sempre às mesmas três perguntas — *o que foi
feito*, *por que é um bom produto* e *como pode gerar faturamento*.

O deck tem **cronômetro**: a apresentação tem duração máxima de **10 minutos**, e o
cronômetro mostra o tempo total, o tempo do slide atual contra a média (10 min ÷ nº de
slides) e quanto ainda **resta** do limite — com as barras virando laranja perto do fim
e vermelhas ao estourar. As setas ← → passam os slides, a **barra de espaço**
pausa e retoma, e o cronômetro **segura sozinho** quando você sai dos slides (volta à
capa ou troca de aba). É modular: registrar uma nova quinzena é adicionar um objeto no
início do array `apresentacoesQuinzenais`.

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
