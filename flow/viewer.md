# Flow: Viewer 3D (ciclo de vida do objeto)

> **Resumo:** Cada arquivo em `objetos/*.html` é um viewer standalone que monta uma cena Three.js, expõe sliders de parâmetros e reconstrói a geometria em tempo real conforme o usuário ajusta valores.

## Visão Geral

Quando o usuário abre um viewer (`objetos/<nome>.html` ou o `_template/viewer-template.html`), o navegador carrega Three.js via importmap, monta o DOM em duas regiões (`#viewer` à esquerda com o canvas, `<aside>` à direita com os controles) e dispara o script de módulo. O script inicializa renderer, cena, câmera com `OrbitControls`, luzes (ambient + direcional com sombras + fill azulado), grid, plano de sombra, material padrão e um loop `requestAnimationFrame` contínuo.

A geometria é gerada pela função `buildGeometry()`, que lê o objeto `PARAMS` (declarado inline no início do script) e cria/recria a `Mesh`. Toda interação do usuário com os sliders chama um handler (`updateParam`, `updateMaterial`, `updateColor`) exposto em `window`; esses handlers atualizam `PARAMS`/material e, no caso de mudança de geometria, descartam a mesh antiga (`geometry.dispose()` + `scene.remove`) e chamam `buildGeometry()` novamente.

A responsividade é tratada por um `ResizeObserver` que ouve o elemento `#viewer` e ajusta `renderer.setSize` + `camera.aspect` automaticamente. O ciclo de animação roda em todo frame, atualiza `OrbitControls.update()` (necessário pelo damping) e renderiza a cena.

## Passo a Passo

1. **Importmap define `three`** — `_template/viewer-template.html:161–168` → mapeia `three` e `three/addons/` para `cdn.jsdelivr.net/npm/three@0.160.0/`.
2. **Imports do módulo** — `viewer-template.html:280–285` → `THREE`, `OrbitControls`, `GLTFExporter`, `OBJExporter`, `STLExporter`.
3. **Declaração de `PARAMS`** — `viewer-template.html:290–297` → `width`, `height`, `depth`, `roughness`, `metalness`, `color`.
4. **Setup do renderer** — `viewer-template.html:302–307` → `WebGLRenderer({ canvas, antialias: true })` com `pixelRatio`, shadow map `PCFSoftShadowMap` e clear color `0x1a1a1a`.
5. **Cena + câmera** — `viewer-template.html:309–311` → `Scene`, `PerspectiveCamera(45°)` posicionada em `(5, 4, 7)`.
6. **OrbitControls** — `viewer-template.html:313–315` → damping habilitado com `dampingFactor = 0.06`.
7. **Iluminação** — `viewer-template.html:318–329` → `AmbientLight(0.4)` + `DirectionalLight(1.2)` com `castShadow` em `(5,10,7)` + `fill` azulado em `(-5,-2,-5)`.
8. **Grid e plano de sombra** — `viewer-template.html:332–342` → `GridHelper(20,20)` e `ShadowMaterial` em plano rotacionado para Y=0.
9. **Material compartilhado** — `viewer-template.html:348–352` → `MeshStandardMaterial({ color, roughness, metalness })` criado uma vez.
10. **Construção inicial** — `viewer-template.html:487` → `buildGeometry()` cria a primeira `Mesh` (geometria padrão = `BoxGeometry`), atribui sombra, posiciona em `y = height/2` e chama `updateInfo()`.
11. **Loop de animação** — `viewer-template.html:480–484, 488` → `animate()` registra `requestAnimationFrame`, atualiza `controls` e renderiza a cena a cada frame.
12. **Usuário move um slider de parâmetro** — handler `oninput` no DOM dispara `updateParam(key, val)` exposto em `window`.
13. **`updateParam` reconstrói geometria** — `viewer-template.html:382–386` → atualiza `PARAMS[key]`, atualiza o label `val-<key>` e chama `buildGeometry()` que **descarta** a geometria anterior (`mesh.geometry.dispose()` + `scene.remove(mesh)`) antes de criar a nova.
14. **Usuário ajusta material** — `viewer-template.html:388–396` → `updateMaterial`/`updateColor` mutam o `MeshStandardMaterial` diretamente (sem rebuild de geometria).
15. **Usuário troca fundo** — `viewer-template.html:398–406` → `setBackground` ajusta `setClearAlpha`/`setClearColor`; suporta `transparent`.
16. **Resize do viewport** — `viewer-template.html:466–475` → `ResizeObserver` no `#viewer` chama `onResize()`, que ajusta `renderer.setSize`, `camera.aspect` e `updateProjectionMatrix`.

### Caminhos alternativos

- **Toggle wireframe:** `viewer-template.html:408–410` → `toggleWireframe` inverte `material.wireframe` (não reconstrói geometria).
- **Reset de parâmetros:** `viewer-template.html:412–421` → `resetParams` zera `PARAMS.width/height/depth`, atualiza valores dos sliders no DOM e dispara rebuild.
- **PNG export altera tamanho temporariamente:** `viewer-template.html:451–461` → `renderer.setSize(w*scale, h*scale)` é chamado antes do `toBlob`; o `onResize()` é chamado no callback para restaurar — durante o instante do export o canvas fica esticado.

## Arquivos Envolvidos

| Camada | Arquivo | Responsabilidade |
|--------|---------|------------------|
| Template | `_template/viewer-template.html` | Base copiada para cada novo viewer; contém todo o setup descrito acima |
| Viewer (instância) | `objetos/<nome>.html` (a criar) | Cópia do template com `buildGeometry()`, `PARAMS` e `OBJECT_NAME` customizados |
| Markup do canvas | `viewer-template.html:182–185` | `<div id="viewer"><canvas id="c"></canvas></div>` — alvo do renderer e do ResizeObserver |
| Markup dos controles | `viewer-template.html:188–276` | `<aside>` com sliders, color picker, select de fundo, botões de export |
| CSS inline | `viewer-template.html:9–158` | Layout flex split-pane, paleta `#0d0d0d`/`#00ff88`/monospaced |

## Regras de Negócio Relevantes

- **Sempre dispose ao rebuildar** — `viewer-template.html:355–358`: `mesh.geometry.dispose()` + `scene.remove(mesh)` antes de criar nova `Mesh`. Esquecer disso causa vazamento de memória GPU em sliders rápidos.
- **Material é compartilhado, geometria é descartável** — `viewer-template.html:348` cria o material **fora** de `buildGeometry()`. Mudanças de cor/rugosidade/metalicidade não disparam rebuild.
- **Mesh é posicionada por altura** — `viewer-template.html:366`: `mesh.position.y = PARAMS.height / 2` deixa a base apoiada no grid Y=0. Geometrias customizadas devem reavaliar esse offset.
- **Sombras dependem do plano** — `viewer-template.html:336–342`: `ShadowMaterial` no plano + `mesh.castShadow = true` + `dirLight.castShadow = true`. Remover qualquer um quebra sombras.
- **Reset não restaura material/cor** — `viewer-template.html:412–421`: `resetParams()` só reseta dimensões; cor, rugosidade, metalicidade e fundo permanecem como o usuário deixou.
- **`updateInfo` lê só `position`** — `viewer-template.html:372–377`: contagem de vértices vem de `geometry.attributes.position.count` e triângulos é `count/3`. Geometrias indexadas (`BufferGeometry` com `index`) terão contagem de triângulos incorreta — usar `geometry.index.count / 3` quando aplicável.

## Dependências Externas

- **CDN jsDelivr** — `three@0.160.0` e seus addons. Sem internet o viewer não carrega (não há fallback local; a pasta `_lib/` mencionada no `CLAUDE.md` não existe).
- **Google Fonts** — `DM Mono`. Fallback para `monospace` quando offline.
- **WebGL no navegador** — necessário para o `WebGLRenderer`. Navegadores muito antigos ou ambientes sem GPU falham na criação do contexto.

## Observações

- **`OBJECT_NAME` precisa ser editado.** `viewer-template.html:433` tem `const OBJECT_NAME = 'objeto'`. Se o criador esquecer de trocar, todos os exports (GLB/OBJ/STL/PNG) saem com nome genérico — colisão silenciosa no `~/Downloads`.
- **Sem `pixelRatio` cap.** `setPixelRatio(window.devicePixelRatio)` em displays HiDPI pode penalizar performance em geometrias densas — alguns projetos limitam a `Math.min(devicePixelRatio, 2)`.
- **`onResize` não trata canvas com aspecto zero.** Se `#viewer` ficar com `clientHeight = 0` (ex: aside colapsada), `camera.aspect` vira `Infinity`. Não há proteção.
- **Sliders são duplicados manualmente no HTML.** Não há geração dinâmica a partir de `PARAMS` — adicionar parâmetro novo exige editar HTML + JS lado a lado.
- **Nenhum `localStorage` ou hash URL.** O `CLAUDE.md` sugere "Parâmetros no URL hash para compartilhamento (`#w=2&h=3`)", mas o template **não implementa** isso. É uma diretriz futura, não comportamento atual.
