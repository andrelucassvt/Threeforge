# ThreeJS Objetos — Workspace

> Projeto de criação e exportação de objetos 3D usando Three.js.
> Cada objeto é um arquivo HTML standalone com viewer interativo e painel de exportação.

---

## Objetivo

Criar objetos 3D parametrizados em Three.js com:
- Visualizador 3D interativo (orbit controls, iluminação, grid)
- Painel lateral com controles de parâmetros do objeto
- Exportação para múltiplos formatos: **GLB**, **OBJ**, **STL**, **PNG**

---

## Estrutura de Arquivos

```
ThreeJS objetos/
├── CLAUDE.md                  ← este arquivo
├── _template/
│   └── viewer-template.html   ← base para novos objetos
├── _lib/                      ← libs locais (opcional, fallback offline)
├── objetos/
│   ├── cubo-chanfrado.html
│   ├── esfera-geodesica.html
│   ├── torre-modular.html
│   └── ...
└── exports/                   ← arquivos exportados (GLB, OBJ, STL, PNG)
```

---

## Template Padrão de Objeto

Cada arquivo em `objetos/` deve seguir este padrão:

### Stack
- **Three.js r160+** via CDN (importmap ou script tag)
- **OrbitControls** para navegação
- **GLTFExporter** para exportar `.glb`
- **OBJExporter** para exportar `.obj`
- **STLExporter** para exportar `.stl`
- **CSS**: tudo inline no HTML, sem arquivos externos

### Layout UI
```
┌─────────────────────────────────────────┐
│  [Nome do Objeto]          [?] [Reset]  │  ← header
├──────────────────┬──────────────────────┤
│                  │  PARÂMETROS          │
│                  │  ─────────────────   │
│   VIEWER 3D      │  Largura: [slider]   │
│   (canvas)       │  Altura:  [slider]   │
│                  │  ...                 │
│                  │  ─────────────────   │
│                  │  EXPORTAR            │
│                  │  [GLB] [OBJ] [STL]  │
│                  │  [PNG 2x] [PNG 4x]  │
└──────────────────┴──────────────────────┘
```

### Cores e Estilo
- Background viewer: `#1a1a1a`
- Background painel: `#111`
- Accent: `#00ff88` (verde neon)
- Texto: `#eee`
- Bordas: `#333`
- Font: `monospace` (DM Mono ou similar via Google Fonts)

### Iluminação Padrão
```js
const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
const dirLight = new THREE.DirectionalLight(0xffffff, 1.2);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;

const fillLight = new THREE.DirectionalLight(0x8888ff, 0.3);
fillLight.position.set(-5, -2, -5);
```

### Grid e Helpers
- `GridHelper` no plano Y=0
- `AxesHelper` pequeno no canto (opcional)
- Sombras habilitadas no renderer

---

## ⚠️ Regra Obrigatória — Onde Salvar Objetos

**Todo objeto criado pela IA DEVE ser salvo em `objetos/nome-kebab.html`.**

Isso é obrigatório porque:
- O `index.html` na raiz lista automaticamente todos os arquivos de `objetos/`
- O usuário abre `index.html` no navegador e de lá acessa qualquer objeto com um clique
- Nunca salve objetos na raiz da pasta nem em outro local

### Fluxo de Criação

```
1. IA cria o objeto  →  salva em objetos/nome-do-objeto.html
2. IA atualiza o index.html adicionando o novo objeto na lista
3. Usuário abre index.html no navegador → clica no objeto → visualiza
```

### Atualizar o index.html

Sempre que um novo objeto for criado, adicione um card no array `OBJECTS` dentro do `index.html`:

```js
const OBJECTS = [
  // ... objetos existentes ...
  {
    file: 'objetos/nome-do-objeto.html',
    name: 'Nome Legível do Objeto',
    desc: 'Breve descrição do que é o objeto e seus parâmetros.',
    tags: ['tag1', 'tag2'],  // ex: ['orgânico', 'modular', 'arquitetura']
  },
];
```

---

## Como Criar um Novo Objeto

1. **Copie** `_template/viewer-template.html`
2. **Renomeie** para `objetos/nome-do-objeto.html`
3. **Implemente** a geometria na função `buildGeometry(params)`
4. **Defina** os parâmetros no objeto `PARAMS` com min/max/step
5. **Conecte** os sliders ao rebuild automático

### Padrão de Geometria Parametrizada
```js
const PARAMS = {
  width:  { value: 2, min: 0.1, max: 10, step: 0.1 },
  height: { value: 3, min: 0.1, max: 10, step: 0.1 },
  // adicionar mais parâmetros aqui
};

function buildGeometry(p) {
  // sempre descarte a geometria anterior
  if (mesh) {
    mesh.geometry.dispose();
    scene.remove(mesh);
  }

  const geo = new THREE.BoxGeometry(p.width.value, p.height.value, p.width.value);
  const mat = new THREE.MeshStandardMaterial({ color: 0x00ff88, roughness: 0.4, metalness: 0.2 });
  mesh = new THREE.Mesh(geo, mat);
  mesh.castShadow = true;
  scene.add(mesh);
}
```

---

## Exportação

### GLB (Three.js GLTFExporter)
```js
import { GLTFExporter } from 'three/addons/exporters/GLTFExporter.js';
const exporter = new GLTFExporter();
exporter.parse(mesh, (gltf) => {
  const blob = new Blob([gltf], { type: 'application/octet-stream' });
  downloadBlob(blob, 'objeto.glb');
}, { binary: true });
```

### OBJ (Three.js OBJExporter)
```js
import { OBJExporter } from 'three/addons/exporters/OBJExporter.js';
const result = new OBJExporter().parse(mesh);
downloadBlob(new Blob([result], { type: 'text/plain' }), 'objeto.obj');
```

### STL (Three.js STLExporter)
```js
import { STLExporter } from 'three/addons/exporters/STLExporter.js';
const result = new STLExporter().parse(mesh, { binary: true });
downloadBlob(new Blob([result]), 'objeto.stl');
```

### PNG (canvas toBlob)
```js
renderer.render(scene, camera);
renderer.domElement.toBlob((blob) => {
  downloadBlob(blob, 'objeto.png');
}, 'image/png');
```

### Helper de Download
```js
function downloadBlob(blob, filename) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}
```

---

## CDN Imports (importmap recomendado)

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
  }
}
</script>
<script type="module">
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
  import { GLTFExporter } from 'three/addons/exporters/GLTFExporter.js';
  import { OBJExporter }  from 'three/addons/exporters/OBJExporter.js';
  import { STLExporter }  from 'three/addons/exporters/STLExporter.js';
</script>
```

---

## Boas Práticas

- **Um arquivo HTML por objeto** — tudo inline (CSS + JS), zero dependências locais
- **Sempre dispose** geometrias e materiais ao reconstruir (`geo.dispose()`, `mat.dispose()`)
- **ResizeObserver** no canvas para responsividade (`renderer.setSize`, `camera.aspect`)
- **requestAnimationFrame** loop contínuo (não renderizar só no evento de mouse)
- **Parâmetros no URL hash** para compartilhamento: `#w=2&h=3`
- **Nomes de arquivo**: kebab-case, descritivos (`esfera-geodesica-v2.html`)

---

## Formatos de Saída Suportados

| Formato | Uso | Exporter |
|---------|-----|----------|
| `.glb`  | Game engines, Blender, web AR/VR | GLTFExporter (binary) |
| `.gltf` | Intercâmbio legível por humanos | GLTFExporter (JSON) |
| `.obj`  | CAD, impressão 3D, compatibilidade ampla | OBJExporter |
| `.stl`  | Impressão 3D (fatiadores) | STLExporter |
| `.png`  | Preview, portfólio, documentação | canvas.toBlob |

---

## Checklist para Cada Objeto Novo

- [ ] Geometria parametrizada com sliders funcionando
- [ ] Rebuild automático ao mover slider (sem lag)
- [ ] Iluminação padrão aplicada
- [ ] Sombras habilitadas
- [ ] OrbitControls configurado (damping on)
- [ ] Exportação GLB funcionando
- [ ] Exportação OBJ funcionando  
- [ ] Exportação STL funcionando
- [ ] Export PNG (render atual)
- [ ] Responsivo (resize do canvas)
- [ ] Arquivo salvo em `objetos/nome-kebab.html`
