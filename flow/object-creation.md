# Flow: Criação de um Novo Objeto 3D

> **Resumo:** Fluxo manual (executado pela IA ou desenvolvedor) para adicionar um objeto 3D ao workspace: copiar o template, customizar geometria e parâmetros, e registrar o objeto na galeria editando o array `OBJECTS` do `index.html`.

## Visão Geral

O projeto não tem auto-discovery — adicionar um arquivo em `src/objetos/` não basta. O processo correto, definido em `CLAUDE.md`, tem três passos obrigatórios: (1) copiar `src/_template/viewer-template.html` para `src/objetos/<nome-kebab>.html`; (2) editar o novo arquivo para conter a geometria desejada, ajustando `PARAMS`, `buildGeometry()`, os sliders no HTML e a constante `OBJECT_NAME`; (3) adicionar um item ao array `OBJECTS` no `index.html` para que o card apareça na galeria.

O resultado final é um arquivo HTML standalone funcional (sem dependências locais, tudo via CDN) que pode ser aberto direto no navegador ou pela galeria. O template já entrega scene/luzes/orbit/material/exportadores prontos — o que muda entre objetos é essencialmente a função `buildGeometry()` e a definição de `PARAMS`.

Esse fluxo é uma **convenção do projeto**, não um sistema automatizado. A obrigatoriedade vem de `CLAUDE.md` (seção "Regra Obrigatória — Onde Salvar Objetos").

## Passo a Passo

1. **Copiar o template** — origem `src/_template/viewer-template.html` → destino `src/objetos/<nome-kebab>.html`.
   - Nome em kebab-case, descritivo (`esfera-geodesica-v2.html`, `cubo-chanfrado.html`).
2. **Trocar o título da página** — `<title>` (linha 6) e `<h1>` (linha 173) → nome legível do objeto.
3. **Definir `PARAMS`** — `viewer-template.html:290–297` → ajustar chaves/valores para os parâmetros que fazem sentido para a geometria.
4. **Reescrever `buildGeometry()`** — `viewer-template.html:354–370` → trocar `THREE.BoxGeometry(...)` pela geometria real (combinações de `THREE.*Geometry`, `BufferGeometry` custom, ou loops gerando meshes filhas).
   - Manter o padrão: descartar mesh antiga (`geometry.dispose()` + `scene.remove`), criar nova `Mesh`, `castShadow = true`, posicionar Y se necessário, chamar `updateInfo()`.
5. **Ajustar sliders no HTML** — `viewer-template.html:191–214` → para cada chave de `PARAMS`, ter um `.param-row` com `id="sl-<key>"`, label `val-<key>` e `oninput="updateParam('<key>', this.value)"`.
   - Adicionar/remover linhas conforme `PARAMS` (não há geração automática).
6. **Trocar `OBJECT_NAME`** — `viewer-template.html:433` → nome usado pelos exportadores (sem extensão).
7. **Ajustar `resetParams()`** — `viewer-template.html:412–421` → atualizar para resetar as chaves reais do novo `PARAMS` (o template tem apenas width/height/depth hardcoded).
8. **Registrar na galeria** — `index.html:276–285` (array `OBJECTS`) → adicionar item:
   ```js
   {
     file: 'src/objetos/<nome-kebab>.html',
     name: 'Nome Legível',
     desc: 'Descrição curta dos parâmetros principais.',
     icon: '⬡',
     tags: ['categoria', 'estilo'],
   }
   ```
9. **Validar visualmente** — abrir `index.html` no navegador → verificar card → clicar → conferir que o viewer abre, sliders atualizam a geometria, exports funcionam.

### Caminhos alternativos

- **Esquecer de registrar no `index.html`:** o arquivo funciona individualmente (`src/objetos/<nome>.html` direto), mas não aparece na galeria.
- **Esquecer de trocar `OBJECT_NAME`:** todos os exports saem como `objeto.glb`/`objeto.obj`/etc., colidindo com outros objetos no `~/Downloads`.
- **`PARAMS` sem slider correspondente:** o parâmetro existe internamente mas é imutável pela UI; rebuild não é disparado.
- **Slider sem chave em `PARAMS`:** `updateParam` faz `PARAMS[key] = ...` mas a chave não é lida em `buildGeometry()` → mudança silenciosa que não afeta a mesh.

## Arquivos Envolvidos

| Camada | Arquivo | Responsabilidade |
|--------|---------|------------------|
| Template | `src/_template/viewer-template.html` | Fonte da cópia; contém scene, sliders de exemplo, exportadores |
| Documento de regras | `CLAUDE.md`, `AGENTS.md` | Define a obrigatoriedade de salvar em `src/objetos/` e atualizar `index.html` |
| Saída (novo arquivo) | `src/objetos/<nome-kebab>.html` | Viewer customizado; standalone |
| Registro na galeria | `index.html` (array `OBJECTS`, linhas 276–285) | Item descritivo com `file`, `name`, `desc`, `icon`, `tags` |

## Regras de Negócio Relevantes

- **Localização obrigatória** — `CLAUDE.md` (seção "Convenções"): todo objeto deve estar em `src/objetos/`, em kebab-case. Salvar em outro lugar quebra a expectativa do `index.html`.
- **Cada objeto é autônomo** — herdado de `CLAUDE.md` ("Um arquivo HTML por objeto — tudo inline, zero dependências locais"). Não criar arquivos compartilhados; duplicação é aceita.
- **Dispose ao rebuildar é não-negociável** — `viewer-template.html:355–358`: a regra de `geometry.dispose()` + `scene.remove(mesh)` antes de criar nova mesh é parte do checklist obrigatório em `CLAUDE.md`.
- **Iluminação padrão deve ser mantida** — `CLAUDE.md` (seção "Iluminação Padrão"): novos objetos devem preservar `AmbientLight(0.4)` + `DirectionalLight(1.2)` + fill azulado. Customizações são exceção, não regra.
- **Paleta visual é fixa** — `bg #1a1a1a`, painel `#111`, accent `#00ff88`, texto `#eee`, monospace. Mudar a paleta de um objeto isoladamente quebra a coesão visual do workspace.
- **Checklist em `CLAUDE.md`** — antes de considerar o objeto pronto, verificar: geometria parametrizada, rebuild sem lag, sombras, OrbitControls com damping, 4 exports funcionando, responsivo, salvo em `objetos/`, registrado na galeria.

## Dependências Externas

- **CDN jsDelivr** — herdado do template; cada novo objeto também depende.
- **`CLAUDE.md` / `AGENTS.md`** — documento operacional que define o fluxo. Se editar um, sincronizar o outro (são idênticos hoje).

## Observações

- **`AGENTS.md` e `CLAUDE.md` duplicam o mesmo conteúdo.** Provavelmente para suportar ferramentas diferentes (Claude Code vs outros agents). Mudanças neste fluxo precisam ser refletidas nos dois arquivos.
- **Sem gerador / template engine.** O passo "copiar template e editar" é manual — não há comando tipo `npm run new-object`. Um script bash trivial (`cp _template/viewer-template.html objetos/$1.html`) resolveria, mas não foi implementado.
- **A lista `OBJECTS` no `index.html` é hardcoded.** Em projetos maiores faria sentido extrair para um JSON separado ou auto-gerar a partir de `objetos/*.html`, mas hoje é convenção manual.
- **Não há validação automática do checklist.** O `CLAUDE.md` lista 11 critérios ("✓ Geometria parametrizada...") mas nada force-verifica. É responsabilidade do criador.
- **Diretriz futura não-implementada:** `CLAUDE.md` menciona "Parâmetros no URL hash para compartilhamento (`#w=2&h=3`)" — não está no template. Quem implementar primeiro deve atualizar o template, não só seu objeto.
- **Risco de colisão de `OBJECT_NAME`.** Sem disciplina, todos os exports podem cair como `objeto.glb` em `~/Downloads`. O criador deve sempre trocar a constante.
