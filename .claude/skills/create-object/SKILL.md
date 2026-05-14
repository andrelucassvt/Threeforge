---
name: create-object
description: Creates a new 3D parametric object in Threeforge. Copies the viewer template, customizes geometry/params/OBJECT_NAME/sliders, and registers the card in index.html. Trigger whenever the user asks to create, add, or build a new 3D object — including Portuguese requests like "quero uma esfera geodésica", "adiciona um cubo", "faz um objeto novo", "cria um viewer pra X". If the user describes any 3D shape, geometry, or parametric concept they want to visualize, this is the skill to use.
---

# create-object

## O que esta skill faz

Executa o fluxo completo de criação de um objeto 3D no workspace Threeforge:

1. **Coleta** nome, geometria, parâmetros, ícone e tags do objeto
2. **Cria** `src/objetos/<nome-kebab>.html` a partir do template
3. **Customiza** título, `OBJECT_NAME`, `PARAMS`, `buildGeometry()`, sliders e `resetParams()`
4. **Registra** o card no array `OBJECTS` de `index.html`

Ao final o objeto está funcional e visível na galeria — pronto para abrir no navegador.

---

## Fase 1 — Coletar informações

Leia os ARGUMENTS. Extraia o máximo possível deles antes de perguntar.

Você precisa de:

| Campo | Exemplo | Derivação |
|---|---|---|
| `nome_legivel` | "Esfera Geodésica" | do usuário ou dos args |
| `nome_kebab` | `esfera-geodesica` | derive do nome legível; confirme se ambíguo |
| `object_name` | `esfera-geodesica` | igual ao kebab, sem extensão |
| `descricao` | "Esfera subdividida parametricamente..." | do usuário ou dos args |
| `icone` | `🔮` | do usuário ou dos args; default `◻` |
| `tags` | `['geodésico', 'esfera']` | do usuário ou dos args; mínimo 1 |
| `conceito_geometria` | "IcosahedronGeometry com detail controlado por slider" | do usuário ou args |
| `params_necessarios` | `{ detail: 2, radius: 1 }` | derivado do conceito |

**Se faltar informação essencial** (nome ou conceito da geometria), faça uma única pergunta cobrindo o máximo de lacunas de uma vez. Não faça múltiplas rodadas de perguntas.

---

## Fase 2 — Verificar conflito

Antes de criar qualquer arquivo:

1. Verifique se `src/objetos/<nome-kebab>.html` já existe.
   - Se existir: informe o usuário e pergunte se quer sobrescrever ou usar outro nome.
2. Verifique se `OBJECT_NAME = '<nome>'` já aparece em outro arquivo em `src/objetos/`.
   - Se houver conflito de nome de export: avise o usuário (colisão silenciosa em `~/Downloads`).

---

## Skills de referência

Antes de implementar, consulte a skill especializada correspondente ao que o objeto exige:

| Necessidade | Skill |
|---|---|
| Geometria nativa (Box, Sphere, Torus, etc.) ou BufferGeometry customizada | **`threejs-geometry`** |
| Material além do padrão (MeshPhysical, MeshToon, dois lados, wireframe, blending) | **`threejs-materials`** |
| Efeitos visuais via GLSL (vertex distortion, fragment gradients, noise) | **`threejs-shaders`** |
| Animação procedural no loop ou baseada em clock | **`threejs-animation`** |
| Texturas, mapas UV, cubemaps ou ambientes HDR | **`threejs-textures`** |
| Iluminação além do padrão Ambient + Dir + Fill | **`threejs-lighting`** |
| Bloom, DOF ou outros efeitos de pós-processamento | **`threejs-postprocessing`** |
| Carregamento de modelos externos (GLB, OBJ) ou assets | **`threejs-loaders`** |
| Interação customizada além do OrbitControls (raycasting, drag) | **`threejs-interaction`** |
| Dúvidas sobre cena, câmera, renderer ou hierarquia de objetos | **`threejs-fundamentals`** |

> Todo objeto novo usa pelo menos **`threejs-geometry`** e **`threejs-materials`**. Consulte as demais conforme as features planejadas no brainstorming.

---

## Fase 3 — Criar o arquivo

Leia o template completo: `src/_template/viewer-template.html`

Crie `src/objetos/<nome-kebab>.html` com as seguintes modificações:

### 3.1 — Título e cabeçalho

- `<title>` → `<nome_legivel> — ThreeJS Viewer`
- `<h1>` → `<nome_legivel>` (em uppercase se seguir a convenção do template)

### 3.2 — OBJECT_NAME

```js
const OBJECT_NAME = '<nome_kebab>';
```

### 3.3 — PARAMS

Substitua o bloco `PARAMS` pelos parâmetros reais do objeto. Inclua sempre `roughness`, `metalness` e `color` (usados pelo material padrão). Adicione os parâmetros de geometria:

```js
const PARAMS = {
  // parâmetros de geometria
  <chave>: <valor_default>,
  // parâmetros de material (manter sempre)
  roughness: 0.4,
  metalness: 0.2,
  color:     0x00ff88,
};
```

### 3.4 — buildGeometry()

Reescreva o corpo da função mantendo o padrão obrigatório:

```js
function buildGeometry() {
  if (mesh) {
    mesh.geometry.dispose();   // ← obrigatório: evita vazamento GPU
    scene.remove(mesh);
  }

  // ↓ geometria real do objeto usando PARAMS
  const geo = new THREE.<Geometry>(PARAMS.<chave>, ...);

  mesh = new THREE.Mesh(geo, material);
  mesh.castShadow = true;
  mesh.position.y = <offset_y>;  // geralmente PARAMS.height / 2 ou 0 para esferas
  scene.add(mesh);

  updateInfo();
}
```

### 3.5 — Sliders no HTML

Para cada parâmetro de **geometria** em `PARAMS`, adicione um `.param-row` na seção `Parâmetros`:

```html
<div class="param-row">
  <div class="param-label"><nome_label> <span id="val-<chave>"><valor_default></span></div>
  <input type="range" id="sl-<chave>" min="<min>" max="<max>" step="<step>" value="<valor_default>"
         oninput="updateParam('<chave>', this.value)" />
</div>
```

Regras:
- `id="sl-<chave>"` e `id="val-<chave>"` devem bater exatamente com a chave em `PARAMS`
- Parâmetros de material (`roughness`, `metalness`, `color`) já têm sliders no template — **não duplique**
- Ajuste `min`, `max` e `step` para valores que façam sentido visual (ex: radius 0.1–5 step 0.1)

### 3.6 — resetParams()

Atualize `resetParams()` para resetar exatamente os parâmetros de **geometria** (não material/cor):

```js
window.resetParams = () => {
  // para cada parâmetro de geometria:
  PARAMS.<chave> = <valor_default>;
  document.getElementById('sl-<chave>').value = <valor_default>;
  document.getElementById('val-<chave>').textContent = '<valor_formatado>';
  // ...
  buildGeometry();
};
```

---

## Fase 4 — Registrar na galeria

Edite `index.html` e adicione um item ao array `OBJECTS`:

```js
{
  file: 'src/objetos/<nome-kebab>.html',
  name: '<nome_legivel>',
  desc: '<descricao>',
  icon: '<icone>',
  tags: [<tags>],
},
```

Insira antes do `];` que fecha o array. Mantenha a vírgula no item anterior se houver.

---

## Fase 5 — Confirmação

Ao terminar, informe ao usuário:

```
Objeto criado:
  Arquivo:  src/objetos/<nome-kebab>.html
  Galeria:  item adicionado ao array OBJECTS em index.html
  Exports:  <nome-kebab>.glb / .obj / .stl / .png

Para validar: abra index.html no navegador → clique no card → teste os sliders e os exports.
```

Se houver qualquer desvio das convenções (ex: parâmetro sem slider, offset Y fixo por limitação da geometria), mencione explicitamente.

---

## Checklist antes de declarar pronto

- [ ] Arquivo em `src/objetos/<nome-kebab>.html` (não em outro lugar)
- [ ] `<title>` e `<h1>` com o nome do objeto
- [ ] `OBJECT_NAME` trocado (não é `'objeto'`)
- [ ] `PARAMS` com as chaves corretas
- [ ] `buildGeometry()` com `dispose()` + `scene.remove()` antes do rebuild
- [ ] `mesh.castShadow = true` presente
- [ ] Um slider por parâmetro de geometria (ids conferindo)
- [ ] `resetParams()` reseta os parâmetros reais (não apenas width/height/depth)
- [ ] Item adicionado ao `OBJECTS` em `index.html`
- [ ] Iluminação padrão preservada (não remover ambient/dir/fill do template)

---

## Regras Gerais

**Salve o arquivo em `src/objetos/`**: o `index.html` aponta para esse diretório por convenção — arquivos em outro lugar não aparecem na galeria e confundem quem abre o projeto depois.

**Registre sempre em `index.html`**: o arquivo funciona standalone ao abrir diretamente no navegador, mas sem a entrada no array `OBJECTS` o card não aparece para ninguém que usa a galeria.

**Preserve o dispose em todo rebuild**: `geometry.dispose()` + `scene.remove(mesh)` antes de criar nova geometria evita vazamento de memória GPU — em sliders rápidos o browser pode travar sem isso.

**Mantenha a paleta e a fonte do template**: o CSS do template define a identidade visual do workspace — mudar cores ou fonte num objeto individual quebra a coesão da galeria inteira.

**Idioma**: use o mesmo idioma da conversa com o usuário.
