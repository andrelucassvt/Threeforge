# Threeforge

![Threeforge preview](assets/image1.png)

Workspace para criação e exportação de objetos 3D parametrizados com [Three.js](https://threejs.org/). Cada objeto é um arquivo HTML standalone — sem build, sem dependências locais. Abra o `index.html` no navegador e acesse qualquer objeto com um clique.

---

## Como funciona

1. `index.html` na raiz é a galeria — lista todos os objetos cadastrados
2. Cada objeto vive em `src/objetos/<nome>.html`, autossuficiente (HTML + CSS + JS inline)
3. Ao abrir um objeto, você tem um viewer 3D interativo com sliders de parâmetros e botões de exportação

O viewer de cada objeto:
- Renderiza a geometria com Three.js (WebGL)
- Expõe sliders ligados ao objeto `PARAMS` — alterar um slider chama `buildGeometry()` e reconstrói a malha em tempo real
- Permite exportar nos formatos GLB, OBJ, STL e PNG sem nenhum servidor

---

## Como criar um novo objeto

Use a skill `/create-object <nome>` no Claude Code. Ela executa o fluxo completo:

1. Copia `src/_template/viewer-template.html` para `src/objetos/<nome>.html`
2. Customiza a geometria, os `PARAMS` e os sliders
3. Registra o card no array `OBJECTS` do `index.html`

> Antes de criar, invoque `/brainstorming` — é obrigatório para alinhar o design do objeto antes de implementar.

---

## Fluxo resumido

```
index.html (galeria)
  └── src/objetos/<nome>.html (viewer standalone)
        ├── PARAMS          → valores dos sliders
        ├── buildGeometry() → reconstrói a malha a cada mudança
        └── export buttons  → GLB / OBJ / STL / PNG
```

---

## Formatos de exportação

| Formato | Uso |
|---------|-----|
| `.glb`  | Game engines, Blender, AR/VR |
| `.obj`  | CAD, impressão 3D, compatibilidade ampla |
| `.stl`  | Impressão 3D (fatiadores como Cura, PrusaSlicer) |
| `.png`  | Preview, portfólio, documentação |

---

## Estrutura de arquivos

```
.
├── index.html                        ← galeria
├── src/
│   ├── _template/
│   │   └── viewer-template.html      ← base para novos objetos (não editar; copiar)
│   └── objetos/
│       ├── cubo.html
│       └── bolha-de-sabao.html
├── assets/                           ← imagens e recursos estáticos
├── flow/                             ← documentação de fluxos
└── exports/                          ← convenção para arquivos exportados
```

---

## Stack

- **Three.js r160** via CDN (importmap) — sem bundler
- **OrbitControls** — navegação 3D
- **GLTFExporter / OBJExporter / STLExporter** — exportação
- CSS e JS inline em cada HTML — zero arquivos externos
- Fonte: `DM Mono`, paleta dark minimalista estilo Apple
