# Flow: Galeria de Objetos

> **Resumo:** Página inicial do workspace que lista os objetos 3D disponíveis em formato de grid, com busca textual e filtro por tags, abrindo o viewer escolhido em uma nova aba.

## Visão Geral

O usuário abre `index.html` no navegador. A página renderiza um header com contagem de objetos, uma toolbar com campo de busca e botões de tag, e um grid de cards. A fonte de dados é um array literal `OBJECTS` declarado **inline** no próprio HTML — cada item descreve um objeto (arquivo, nome, descrição, ícone, tags). Não há fetch, banco ou backend: editar o array é a única forma de adicionar/remover objetos da galeria.

No `init`, o script atualiza o contador total, monta dinamicamente os botões de tag a partir de todas as tags únicas e chama `render(OBJECTS)` para criar os cards. Cada card é um `<a target="_blank">` apontando para `objetos/<nome>.html` — clicar abre o viewer correspondente em uma aba nova, sem navegação client-side, sem router. A galeria não conhece o conteúdo dos viewers; apenas lista links.

A busca (`oninput` no campo) e o filtro de tags (`onclick` em cada `.tag-btn`) chamam `filter()`, que reaplica os critérios sobre o array em memória e re-renderiza o grid. Quando `OBJECTS` está vazio ou quando o filtro não encontra resultados, um placeholder `.empty` é exibido.

## Passo a Passo

1. **HTML carrega** — `index.html` → `<body>` renderiza header, toolbar e um grid contendo apenas o placeholder `#empty`.
2. **Script inline executa** — `index.html` → bloco `<script>` na raiz do `<body>`.
3. **Atualiza contador** — `index.html:374` → `document.getElementById('total-count').textContent = OBJECTS.length`.
4. **Decide branch inicial** — `index.html:376–381` → se `OBJECTS.length === 0`, mostra `.empty`; senão chama `buildTags()` e `render(OBJECTS)`.
5. **Constrói tags (se há objetos)** — `index.html:351–371` → `buildTags()` coleta `new Set(OBJECTS.flatMap(o => o.tags))` e injeta um `<button class="tag-btn">` para cada uma.
6. **Renderiza cards** — `index.html:292–332` → `render(items)` remove cards existentes (preservando `#empty`), itera `items` e cria um `<a class="card">` com preview/ícone, nome, descrição, tags e footer.
7. **Usuário interage** — digita no input ou clica em uma tag → handler dispara `filter()`.
8. **Aplica filtros** — `index.html:334–349` → `filter()` parte de `OBJECTS`, aplica filtro por `activeTag` se houver, depois aplica filtro textual contra `name`/`desc`/`tags`, e chama `render(items)` com o resultado.
9. **Usuário clica em um card** — o `<a href="objetos/<nome>.html" target="_blank">` abre o viewer em nova aba; nenhum código JS adicional roda neste passo.

### Caminhos alternativos

- **Galeria vazia:** `OBJECTS.length === 0` exibe `.empty` com mensagem "Nenhum objeto encontrado". O mesmo placeholder também aparece quando a busca/filtro não encontra resultados.
- **Busca sem resultado:** `filter()` retorna lista vazia → `render([])` reexibe `.empty` (linha 299–303).
- **Tag clicada duas vezes:** o handler em `buildTags` desativa a tag (`activeTag = null`) e remove a classe `.active` (linhas 359–367).

## Arquivos Envolvidos

| Camada | Arquivo | Responsabilidade |
|--------|---------|------------------|
| Apresentação (HTML) | `index.html` | Markup do header, toolbar e grid; CSS inline com a paleta do projeto |
| Dados (inline) | `index.html` (array `OBJECTS`, linhas 276–285) | Lista de objetos 3D — única "source of truth" da galeria |
| Lógica (inline) | `index.html` (`render`, `filter`, `buildTags`) | Filtragem em memória e geração de DOM |
| Destino dos links | `objetos/*.html` | Viewers individuais — abertos em nova aba, não importados pela galeria |

## Regras de Negócio Relevantes

- **Lista é hard-coded** — `index.html:276`: a IA (ou desenvolvedor) deve adicionar manualmente um item ao array `OBJECTS` toda vez que criar um objeto novo. Não há varredura de diretório.
- **Filtro textual cobre 3 campos** — `index.html:341–346`: a query é comparada contra `name`, `desc` e cada `tag` (case-insensitive). Ícone e nome do arquivo **não** entram na busca.
- **Tag única ativa por vez** — `index.html:357–367`: ao clicar em uma segunda tag, a primeira é desativada visualmente, mas o `activeTag` é trocado, não combinado. Não há filtro multi-tag.
- **Links abrem em nova aba** — `index.html:309`: `card.target = '_blank'` é fixo; não há suporte a navegação no mesmo tab.

## Dependências Externas

- **Google Fonts** — `index.html:7` carrega `DM Mono` via `fonts.googleapis.com`. Offline, a fonte cai para o fallback `monospace`.
- Nenhuma outra dependência externa nesta página (Three.js só é carregado pelos viewers).

## Observações

- **Sem auto-discovery.** Adicionar um arquivo em `objetos/` não o faz aparecer na galeria — é obrigatório editar o array `OBJECTS`. Um script de geração poderia listar `objetos/*.html` automaticamente, mas hoje a regra explícita em `CLAUDE.md` é manual.
- **Exemplo comentado no array.** As linhas 277–284 contêm um exemplo (`Cubo Chanfrado`) em comentário. Manter como referência ao adicionar o primeiro item real.
- **Sem persistência de filtros.** Recarregar a página reseta busca e tag ativa — não há query string nem `localStorage`.
- **Acessibilidade limitada.** Não há `aria-label` nos botões de tag, e o placeholder do input só comunica em português.
