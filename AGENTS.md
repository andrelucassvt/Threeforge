# Threeforge

Workspace estático de objetos 3D parametrizados em Three.js: cada objeto é um HTML standalone com viewer + sliders + exportadores (GLB/OBJ/STL/PNG). `index.html` na raiz é a galeria.

## Stack

- Three.js **r0.160.0** via importmap (CDN jsDelivr) — versão fixa em cada HTML
- Addons usados: `OrbitControls`, `GLTFExporter`, `OBJExporter`, `STLExporter`
- Sem `package.json`, sem build, sem bundler, sem servidor — abre no navegador
- Fonte: `DM Mono` (Google Fonts), fallback `monospace`
- Paleta: bg `#0a0a0a`/surface `#161618`/accent `#00ff88`/text `#f0f0f0`/muted `#6e6e73`/border `rgba(255,255,255,0.07)` — estilo Apple dark minimalista; accent usado com moderação

## Estrutura

- `index.html` — galeria; array `OBJECTS` hardcoded é a fonte da listagem
- `src/_template/viewer-template.html` — base obrigatória para novos objetos (não editar; copiar)
- `src/objetos/<nome-kebab>.html` — cada objeto vive aqui, autossuficiente
- `src/exports/` — convenção (manual) para arquivos exportados; não é alvo automático do download
- `flow/` — documentação de fluxos (ver seção final)
- `AGENTS.md` — duplicata de `CLAUDE.md` para outras ferramentas; manter sincronizado

## Comandos

Não há scripts. Para validar mudanças: abrir `index.html` no navegador (ou um servidor estático local) e clicar no objeto.

## Convenções

- **Salvar objetos em `src/objetos/<nome-kebab>.html`** — `index.html` lista a partir do array `OBJECTS`; não há auto-discovery
- **Registrar todo objeto novo no array `OBJECTS` de `index.html`** (`file`, `name`, `desc`, `icon`, `tags`) — sem isso o card não aparece
- **Trocar `OBJECT_NAME` no novo viewer** (linha ~433 do template) — define o nome de todos os exports (`.glb/.obj/.stl/.png`); padrão `'objeto'` causa colisão
- **`buildGeometry()` sempre faz dispose antes de recriar** — `mesh.geometry.dispose()` + `scene.remove(mesh)` antes de instanciar nova `Mesh`; senão vaza memória GPU em sliders rápidos
- **Material é criado uma vez fora de `buildGeometry()`** — sliders de cor/rugosidade/metalicidade mutam o material direto, sem rebuild
- **Manter iluminação padrão** (Ambient 0.4 + Directional 1.2 com shadow + fill azulado 0.3) — definida em todo viewer
- **Tudo inline em cada HTML** — sem CSS/JS externos, sem imports locais entre arquivos; duplicação é aceitável por design
- **Adicionar slider exige editar HTML + `PARAMS` + `buildGeometry()` em paralelo** — não há geração automática
- **Editar `CLAUDE.md` e `AGENTS.md` juntos** — são idênticos hoje

## Gotchas

- **CDN é a única fonte do Three.js** — offline, o viewer quebra. O `_lib/` mencionado em README/CLAUDE antigo não existe; não há fallback local.
- **`resetParams()` no template só reseta width/height/depth** — ao customizar `PARAMS`, atualizar manualmente o handler de reset.
- **`updateInfo()` assume geometria não-indexada** — usa `position.count / 3` para triângulos; geometrias com `index` reportarão contagem errada — corrigir para `geometry.index.count / 3` quando aplicável.
- **PNG redimensiona o canvas e restaura via `onResize()` no callback** — geometrias densas mostram o canvas esticado por um instante.
- **`setPixelRatio(window.devicePixelRatio)` sem cap** — em displays HiDPI pode travar; considerar `Math.min(devicePixelRatio, 2)` se vier a doer.

## Não fazer

- Não salvar objetos fora de `src/objetos/` — quebra a expectativa do `index.html`
- Não esquecer de registrar no array `OBJECTS` — o arquivo funciona standalone, mas não aparece na galeria
- Não compartilhar CSS/JS entre objetos via arquivos externos — cada HTML é autônomo por design
- Não trocar a paleta nem a fonte em objetos individuais — coesão visual do workspace depende disso
- Não esquecer de trocar `OBJECT_NAME` ao criar um novo viewer
- Não criar features, telas, widgets ou objetos sem antes invocar a skill `brainstorming` (regra obrigatória do projeto em `.claude/rules/brainstorming.instructions.md`)

## 📖 Documentação de Flows

Para qualquer feature ou fluxo, verifique a pasta `./flow/`: leia os títulos dos arquivos `.md` disponíveis e, se algum for relevante para a tarefa atual, leia-o antes de implementar ou debugar. Use `/flow <nome>` para criar ou atualizar flows individuais.

Disponíveis hoje:
- `flow/project-structure.md` — estrutura geral, stack, camadas
- `flow/gallery.md` — como `index.html` lista, busca e filtra objetos
- `flow/viewer.md` — ciclo de vida de um viewer (cena, params, rebuild)
- `flow/export.md` — pipeline de exportação GLB/OBJ/STL/PNG
- `flow/object-creation.md` — fluxo manual de criar um novo objeto
