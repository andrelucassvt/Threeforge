# Threeforge

Workspace para criação e exportação de objetos 3D parametrizados com [Three.js](https://threejs.org/).

Cada objeto é um arquivo HTML standalone — sem build, sem dependências locais. Abra o `index.html` no navegador e acesse qualquer objeto com um clique.

---

## Como usar

1. Abra `index.html` no navegador
2. Clique em um objeto para abrir o viewer
3. Ajuste os parâmetros nos sliders
4. Exporte no formato desejado (GLB, OBJ, STL, PNG)

---

## Estrutura

```
.
├── index.html              ← galeria com todos os objetos
├── _template/
│   └── viewer-template.html   ← base para novos objetos
├── objetos/                ← um .html por objeto
│   └── *.html
└── exports/                ← arquivos exportados
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

## Criar um novo objeto

1. Copie `_template/viewer-template.html` para `objetos/nome-do-objeto.html`
2. Implemente a geometria em `buildGeometry()`
3. Defina os parâmetros no objeto `PARAMS`
4. Adicione um card no array `OBJECTS` dentro do `index.html`

```js
// index.html → array OBJECTS
{
  file: 'objetos/nome-do-objeto.html',
  name: 'Nome Legível',
  desc: 'Breve descrição do objeto e seus parâmetros.',
  icon: '⬡',
  tags: ['geométrico', 'orgânico'],
}
```

---

## Stack

- **Three.js r160** via CDN (importmap) — sem bundler
- **OrbitControls** — navegação 3D
- **GLTFExporter / OBJExporter / STLExporter** — exportação
- CSS e JS inline em cada HTML — zero arquivos externos
