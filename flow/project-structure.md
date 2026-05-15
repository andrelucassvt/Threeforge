# Estrutura do Projeto: Threeforge

> **Resumo:** Workspace estático (HTML/JS) para criação, visualização e exportação de objetos 3D parametrizados em Three.js. Cada objeto é um arquivo HTML standalone com viewer, painel de parâmetros e exportadores (GLB/OBJ/STL/PNG); o `index.html` na raiz é uma galeria que lista manualmente os objetos em `src/objetos/`.

## Stack e Tecnologias

| Elemento | Valor |
|----------|-------|
| Linguagem | HTML + JavaScript (ES Modules) + CSS inline |
| Framework | Three.js r0.160.0 (via CDN, importmap) |
| Gerenciador de pacotes | Nenhum — sem `package.json`, sem build, sem `node_modules` |
| Principais dependências | `three@0.160.0`, addons: `OrbitControls`, `GLTFExporter`, `OBJExporter`, `STLExporter` |
| Distribuição | `index.html` aberto direto no navegador (file:// ou servidor estático) |
| Fontes | Google Fonts — `DM Mono` |

## Arquitetura

Arquitetura **flat / file-per-object**: não há camadas tradicionais (presentation/domain/data), backend, build system ou roteador. Cada objeto 3D é um arquivo HTML **autossuficiente** que carrega Three.js via CDN, monta uma cena, expõe sliders de parâmetros e botões de exportação. O `index.html` raiz é a única "página" que conhece o conjunto — ele renderiza cards a partir de um array `OBJECTS` declarado inline no próprio arquivo.

```
index.html (galeria)
    │
    ▼  [link]
src/objetos/<nome>.html (viewer standalone)
    │
    ├── Three.js Scene  ──►  buildGeometry(PARAMS)
    │                          │
    │                          ▼
    │                       Mesh + Material
    │
    └── Exportadores  ──►  GLB / OBJ / STL / PNG  ──►  download local
```

### Regras de dependência

- **Cada `objetos/*.html` é independente** — não importa nada de outros arquivos do projeto. Todo CSS/JS/Three.js vem inline ou de CDN.
- **`index.html` não carrega os objetos** — apenas lista links (`<a href="objetos/...">`). A navegação é via `target="_blank"`.
- **Sem servidor / sem build** — qualquer mudança é refletida ao recarregar o HTML no navegador.

## Features

> **Nota:** As "features" abaixo são as funcionalidades estruturais do workspace. Os viewers existentes ficam em `src/objetos/` e só aparecem na galeria quando também são cadastrados no array `OBJECTS`.

| Feature | Caminho principal | Descrição resumida |
|---------|------------------|-------------------|
| Galeria de objetos | `index.html` | Página inicial; lista objetos a partir do array `OBJECTS`, com busca e filtro por tags |
| Template de viewer | `src/_template/viewer-template.html` | Base copiada para criar novos objetos — já contém scene, sliders, exportadores e material |
| Objetos 3D | `src/objetos/*.html` | Cada arquivo é um viewer parametrizado de um objeto específico, como `cubo.html`, `bolha-de-sabao.html`, `detetive-homens-de-preto.html` e `tree.html` |
| Exportações | `src/exports/` | Diretório-alvo sugerido para arquivos exportados pelos viewers |

## Camadas / Módulos Compartilhados

Não há código compartilhado entre arquivos — cada HTML é autossuficiente por design. O que se repete entre eles são **padrões** (não imports):

| Tipo | Onde vive | Responsabilidade |
|------|-----------|-----------------|
| Importmap do Three.js | Inline em cada HTML | Aponta `three` e `three/addons/` para `cdn.jsdelivr.net/npm/three@0.160.0/` |
| Iluminação padrão | Inline em cada viewer | `AmbientLight(0.4)` + `DirectionalLight(1.2)` com sombras + `fill` azulado |
| Setup de cena | Inline em cada viewer | `PerspectiveCamera(45°)`, `OrbitControls` com damping, `GridHelper`, `ShadowMaterial` plane |
| Função `downloadBlob` | Inline em cada viewer | Helper que cria `<a download>` e dispara o save |
| Paleta de cores | CSS inline | `--bg #0d0d0d`, `--surface #111`, `--accent #00ff88`, `--text #eee`, monoespaçada |

## Configuração

| Componente | Arquivo | Responsabilidade |
|-----------|---------|-----------------|
| Lista de objetos da galeria | `index.html` (array `OBJECTS`) | Cada item: `{ file, name, desc, icon, tags }`. A IA atualiza este array ao criar novo objeto. |
| Parâmetros de cada objeto | `src/objetos/<nome>.html` (objeto `PARAMS`) | Map de valores numéricos (width, height, etc.) lidos por `buildGeometry()` |
| CDN do Three.js | Importmap inline | Versão fixa `0.160.0` — mudar exige editar cada HTML manualmente |
| Convenções do projeto | `CLAUDE.md`, `AGENTS.md` | Regras de criação de objetos, layout UI, exportadores, paleta de cores |
| Sincronização de instruções | `sync-brain.sh` (gitignored) | Script auxiliar (não versionado) — uso fora do escopo do código |

## Dependências Externas Principais

| Pacote / Biblioteca | Versão | Uso no projeto |
|--------------------|--------|---------------|
| `three` | 0.160.0 | Engine 3D — `Scene`, `Mesh`, `Geometry`, `Material`, `WebGLRenderer` |
| `three/addons/controls/OrbitControls` | 0.160.0 | Câmera orbital com damping |
| `three/addons/exporters/GLTFExporter` | 0.160.0 | Exporta para `.glb` (binary GLTF) |
| `three/addons/exporters/OBJExporter` | 0.160.0 | Exporta para `.obj` |
| `three/addons/exporters/STLExporter` | 0.160.0 | Exporta para `.stl` (binary) |
| `DM Mono` (Google Fonts) | — | Fonte monoespaçada usada em toda UI |

## Observações

- **Objetos cadastrados manualmente.** A galeria não faz auto-discovery; cada novo viewer em `src/objetos/` precisa ser adicionado ao array `OBJECTS` em `index.html`.
- **`AGENTS.md` e `CLAUDE.md` são idênticos.** Mesmo conteúdo (8206 bytes cada), provavelmente mantidos em paralelo para suportar diferentes ferramentas/agents. Se editar um, sincronize o outro.
- **README descreve uma pasta `_lib/` que não existe.** `CLAUDE.md` menciona "libs locais (opcional, fallback offline)" em `_lib/`, mas a pasta não foi criada. Hoje, sem internet, os viewers quebram (CDN não responde).
- **`exports/` é convenção, não enforcement.** Os exportadores fazem download via `<a download="...">` — o navegador salva em `~/Downloads` por padrão; mover para `exports/` é manual.
- **`OBJECT_NAME` precisa ser editado por objeto.** No template há `const OBJECT_NAME = 'objeto'` — quem criar um novo objeto deve trocar pelo nome real, senão todos os exports saem com o mesmo nome.
- **Sem testes, sem CI, sem lint.** Projeto puramente artesanal; validação é visual (abre no navegador e olha).
