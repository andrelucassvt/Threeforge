---
name: create-character
description: Creates a humanoid character in T-pose in Threeforge. Builds a multi-part 3D figure (head, torso, arms, legs) from proportional parameters with sliders. Trigger whenever the user asks to create a person, humanoid, character, figure, or T-pose model — including "quero um personagem", "cria uma pessoa 3D", "faz um humanoide", "adiciona um personagem", "cria uma figura humana". If the user describes any kind of human or humanoid form they want to visualize, this is the skill to use.
---

# create-character

## O que esta skill faz

Executa o fluxo completo de criação de um personagem humanoide em T-pose no workspace Threeforge:

1. **Coleta** nome, estilo visual, proporções e ícone do personagem
2. **Cria** `src/objetos/<nome-kebab>.html` a partir do template
3. **Adapta** o padrão single-mesh para multi-mesh com `Group`
4. **Implementa** proporções humanoides com parâmetros ajustáveis por slider
5. **Registra** o card no array `OBJECTS` de `index.html`

Esta skill é especializada em personagens. Para objetos geométricos simples (caixas, esferas, etc.), use `create-object`.

---

## Por que é diferente do create-object

O template usa uma única `mesh` com uma única `geometry`. Um personagem é um `THREE.Group` com múltiplas meshes (cabeça, torso, membros). Isso muda três coisas:

| Ponto | create-object | create-character |
|---|---|---|
| Variável principal | `let mesh` | `let character` (Group) |
| Dispose | `mesh.geometry.dispose()` | traverse + dispose cada filho |
| updateInfo | `position.count / 3` de uma geometry | somar triângulos de todos os filhos |
| Exporters | `parse(mesh, ...)` | `parse(character, ...)` |

O material ainda pode ser único e compartilhado entre todas as partes — o template de material do create-object funciona direto.

---

## Proporções humanoides (referência)

Use estas proporções como base para calcular posições e tamanhos a partir de um `height` central:

```
headH      = height / 7.5      // cabeça ≈ 1/7.5 da altura total
neckH      = height * 0.04
torsoH     = height * 0.30
upperArmH  = height * 0.19
lowerArmH  = height * 0.15
upperLegH  = height * 0.25
lowerLegH  = height * 0.22
footH      = height * 0.05

// Y das partes (pés em y=0, tudo sobe)
footTop    = footH
lowerLegY  = footH + lowerLegH / 2
upperLegY  = footH + lowerLegH + upperLegH / 2
hipY       = footH + lowerLegH + upperLegH        // base do torso
torsoMidY  = hipY + torsoH / 2
shoulderY  = hipY + torsoH                         // onde os braços começam
neckMidY   = shoulderY + neckH / 2
headMidY   = shoulderY + neckH + headH / 2

// X das pernas (separação)
legX       = shoulderWidth * 0.25

// X dos braços (T-pose: braços ao longo do eixo X)
upperArmX  = shoulderWidth / 2 + upperArmH / 2
lowerArmX  = upperArmX + upperArmH / 2 + lowerArmH / 2
```

Adapte conforme o estilo desejado. Proporções clássicas são boas para um humano neutro; personagens estilizados podem exagerar cabeça, encurtar pernas, etc.

---

## PARAMS para personagens

```js
const PARAMS = {
  // proporções
  height:       1.8,   // altura total em unidades
  headSize:     0.5,   // escala relativa da cabeça (1 = proporcional)
  shoulderWidth: 0.5,  // largura dos ombros
  limbThickness: 0.12, // espessura dos membros

  // material (manter sempre — usados pelos sliders do template)
  roughness: 0.6,
  metalness: 0.1,
  color:     0x00ff88,
};
```

Ajuste os defaults para o personagem específico. Não remova `roughness`, `metalness` e `color` — os sliders de material do template dependem deles.

---

## buildGeometry() para Group

```js
let character;
const material = new THREE.MeshStandardMaterial({
  color:     PARAMS.color,
  roughness: PARAMS.roughness,
  metalness: PARAMS.metalness,
});

function buildGeometry() {
  // Dispose: percorre o grupo anterior e libera cada geometry
  if (character) {
    character.traverse(child => {
      if (child.isMesh) child.geometry.dispose();
    });
    scene.remove(character);
  }

  character = new THREE.Group();

  // ── calcule dimensões a partir de PARAMS ──────────────────
  const h = PARAMS.height;
  const headH     = (h / 7.5) * PARAMS.headSize;
  const torsoH    = h * 0.30;
  const upperArmH = h * 0.19;
  const lowerArmH = h * 0.15;
  const upperLegH = h * 0.25;
  const lowerLegH = h * 0.22;
  const footH     = h * 0.05;
  const neckH     = h * 0.04;
  const lx        = PARAMS.shoulderWidth * 0.25; // separação das pernas
  const sw        = PARAMS.shoulderWidth;
  const lt        = PARAMS.limbThickness;

  const hipY      = footH + lowerLegH + upperLegH;
  const shoulderY = hipY + torsoH;

  // ── helper: cria mesh, define castShadow, adiciona ao group ─
  function part(geo, y, x = 0, rotZ = 0) {
    const m = new THREE.Mesh(geo, material);
    m.castShadow = true;
    m.position.set(x, y, 0);
    if (rotZ) m.rotation.z = rotZ;
    character.add(m);
  }

  // cabeça
  part(new THREE.SphereGeometry(headH / 2, 16, 12),
       shoulderY + neckH + headH / 2);

  // pescoço
  part(new THREE.CylinderGeometry(lt * 0.5, lt * 0.5, neckH, 8),
       shoulderY + neckH / 2);

  // torso
  part(new THREE.BoxGeometry(sw, torsoH, sw * 0.55),
       hipY + torsoH / 2);

  // braços — T-pose: rotZ = ±Math.PI/2 para apontar ao longo de X
  const upperArmX = sw / 2 + upperArmH / 2;
  const lowerArmX = upperArmX + upperArmH / 2 + lowerArmH / 2;
  const armY      = shoulderY - lt * 0.5;

  part(new THREE.CylinderGeometry(lt / 2, lt / 2, upperArmH, 8),
       armY, +upperArmX, -Math.PI / 2);
  part(new THREE.CylinderGeometry(lt * 0.43, lt / 2, lowerArmH, 8),
       armY, +lowerArmX, -Math.PI / 2);
  part(new THREE.CylinderGeometry(lt / 2, lt / 2, upperArmH, 8),
       armY, -upperArmX, Math.PI / 2);
  part(new THREE.CylinderGeometry(lt * 0.43, lt / 2, lowerArmH, 8),
       armY, -lowerArmX, Math.PI / 2);

  // pernas
  part(new THREE.CylinderGeometry(lt * 0.7, lt * 0.6, upperLegH, 8),
       footH + lowerLegH + upperLegH / 2, +lx);
  part(new THREE.CylinderGeometry(lt * 0.55, lt * 0.5, lowerLegH, 8),
       footH + lowerLegH / 2, +lx);
  part(new THREE.CylinderGeometry(lt * 0.7, lt * 0.6, upperLegH, 8),
       footH + lowerLegH + upperLegH / 2, -lx);
  part(new THREE.CylinderGeometry(lt * 0.55, lt * 0.5, lowerLegH, 8),
       footH + lowerLegH / 2, -lx);

  // pés
  part(new THREE.BoxGeometry(lt * 1.2, footH, lt * 2),
       footH / 2, +lx);
  part(new THREE.BoxGeometry(lt * 1.2, footH, lt * 2),
       footH / 2, -lx);

  scene.add(character);
  updateInfo();
}
```

O helper `part()` evita repetição ao criar cada segmento — adapte conforme o personagem pede mais ou menos partes.

---

## updateInfo() para Group

O `updateInfo()` do template assume uma única geometry. Para o grupo, some os triângulos de todos os filhos:

```js
function updateInfo() {
  if (!character) return;
  let verts = 0, tris = 0;
  character.traverse(child => {
    if (!child.isMesh) return;
    const pos = child.geometry.attributes.position;
    verts += pos.count;
    tris  += child.geometry.index
      ? child.geometry.index.count / 3
      : Math.floor(pos.count / 3);
  });
  document.getElementById('info-verts').textContent = verts;
  document.getElementById('info-tris').textContent  = tris;
}
```

---

## Exporters — trocar mesh por character

O template exporta `mesh`. Troque para `character` nas três funções de export:

```js
// GLB
new GLTFExporter().parse(character, (buf) => { ... });

// OBJ
const result = new OBJExporter().parse(character);

// STL
const result = new STLExporter().parse(character, { binary: true });
```

---

## Sliders no HTML

```html
<!-- altura total -->
<div class="param-row">
  <div class="param-label">Altura <span id="val-height">1.8</span></div>
  <input type="range" id="sl-height" min="0.5" max="3.0" step="0.1" value="1.8"
         oninput="updateParam('height', this.value)" />
</div>

<!-- tamanho da cabeça -->
<div class="param-row">
  <div class="param-label">Cabeça <span id="val-headSize">1.0</span></div>
  <input type="range" id="sl-headSize" min="0.5" max="2.0" step="0.05" value="1.0"
         oninput="updateParam('headSize', this.value)" />
</div>

<!-- largura dos ombros -->
<div class="param-row">
  <div class="param-label">Ombros <span id="val-shoulderWidth">0.5</span></div>
  <input type="range" id="sl-shoulderWidth" min="0.2" max="1.2" step="0.05" value="0.5"
         oninput="updateParam('shoulderWidth', this.value)" />
</div>

<!-- espessura dos membros -->
<div class="param-row">
  <div class="param-label">Membros <span id="val-limbThickness">0.1</span></div>
  <input type="range" id="sl-limbThickness" min="0.04" max="0.3" step="0.01" value="0.12"
         oninput="updateParam('limbThickness', this.value)" />
</div>
```

---

## resetParams()

```js
window.resetParams = () => {
  PARAMS.height        = 1.8;
  PARAMS.headSize      = 1.0;
  PARAMS.shoulderWidth = 0.5;
  PARAMS.limbThickness = 0.12;
  document.getElementById('sl-height').value           = 1.8;
  document.getElementById('sl-headSize').value         = 1.0;
  document.getElementById('sl-shoulderWidth').value    = 0.5;
  document.getElementById('sl-limbThickness').value    = 0.12;
  document.getElementById('val-height').textContent        = '1.8';
  document.getElementById('val-headSize').textContent      = '1.0';
  document.getElementById('val-shoulderWidth').textContent = '0.5';
  document.getElementById('val-limbThickness').textContent = '0.1';
  buildGeometry();
};
```

---

## Fases de criação

Siga o mesmo fluxo de 5 fases do `create-object` (coletar → verificar conflito → criar arquivo → registrar na galeria → confirmar), com estas diferenças:

- **Fase 1**: além de nome/ícone/tags, pergunte sobre estilo (proporcional realista, cabeça grande estilizado, robusto/magro, etc.)
- **Fase 3**: substitua os blocos de `mesh` pelo padrão `Group` descrito acima; adapte proporções ao estilo pedido
- **Checklist adicional**: veja abaixo

---

## Skills de referência

| Necessidade | Skill |
|---|---|
| Geometria de cada parte (variações de shape) | **`threejs-geometry`** |
| Material diferente por parte do corpo (ex: pele + roupa) | **`threejs-materials`** |
| Animação do personagem (caminhada, idle, etc.) | **`threejs-animation`** |
| Shader customizado (toon, pele subsurface, etc.) | **`threejs-shaders`** |
| Texturas no corpo ou roupa | **`threejs-textures`** |

---

## Checklist antes de declarar pronto

- [ ] Variável `character` (Group), não `mesh`
- [ ] dispose com `traverse` + `child.isMesh`
- [ ] `updateInfo()` somando triângulos de todos os filhos
- [ ] Exporters trocados para `character` (não `mesh`)
- [ ] Pés em `y ≈ 0`, topo da cabeça em `y ≈ PARAMS.height`
- [ ] Braços horizontais (T-pose): `rotation.z = ±Math.PI/2`
- [ ] Um slider por parâmetro de geometria (ids conferindo com PARAMS)
- [ ] `resetParams()` reseta todos os parâmetros de geometria
- [ ] `OBJECT_NAME` trocado (não é `'objeto'`)
- [ ] Item adicionado ao `OBJECTS` em `index.html`
- [ ] Iluminação padrão preservada
