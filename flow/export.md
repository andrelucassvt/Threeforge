# Flow: Exportação (GLB / OBJ / STL / PNG)

> **Resumo:** A partir de qualquer viewer, o usuário pode exportar a mesh atual em quatro formatos. Cada botão chama um exporter do Three.js (ou `canvas.toBlob` para PNG), embala o resultado em `Blob` e dispara um download local via `<a download>`.

## Visão Geral

A seção `EXPORTAR` do painel lateral expõe seis botões: `GLB`, `OBJ`, `STL`, `PNG 1×`, `PNG 2×`, `PNG 4×`. Cada um chama uma função exposta em `window` (`exportGLB`, `exportOBJ`, `exportSTL`, `exportPNG`). Todas usam o helper `downloadBlob(blob, filename)` que cria um `<a>` temporário com `URL.createObjectURL`, dispara `.click()` e libera a URL.

GLB, OBJ e STL operam sobre `mesh` (variável de módulo populada por `buildGeometry()`). GLB roda **assíncrono** via callback do `GLTFExporter.parse`; OBJ e STL são **síncronos**. PNG não usa exporter da geometria — ele aumenta temporariamente o tamanho do canvas (`renderer.setSize(w*scale, h*scale)`), renderiza a cena no novo tamanho, captura o pixel buffer com `canvas.toBlob`, faz o download e então restaura o tamanho original chamando `onResize()`.

Nenhum dos exports pergunta caminho — o navegador salva direto em `~/Downloads` (ou no diretório padrão configurado). A convenção do projeto é mover manualmente para a pasta `exports/` depois.

## Passo a Passo

### GLB
1. **Botão clicado** — `_template/viewer-template.html:259` → `onclick="exportGLB()"`.
2. **Handler** — `viewer-template.html:435–439` → `exportGLB` instancia `new GLTFExporter()`, chama `.parse(mesh, callback, errorCb, { binary: true })`.
3. **Callback recebe ArrayBuffer** — `viewer-template.html:436–437` → `new Blob([buf], { type: 'application/octet-stream' })` → `downloadBlob(blob, '${OBJECT_NAME}.glb')`.

### OBJ
1. **Botão clicado** — `viewer-template.html:260` → `onclick="exportOBJ()"`.
2. **Handler síncrono** — `viewer-template.html:441–444` → `new OBJExporter().parse(mesh)` retorna `string` → `Blob([result], { type: 'text/plain' })` → download.

### STL
1. **Botão clicado** — `viewer-template.html:261` → `onclick="exportSTL()"`.
2. **Handler síncrono** — `viewer-template.html:446–449` → `new STLExporter().parse(mesh, { binary: true })` retorna `DataView` → `Blob([result], { type: 'application/octet-stream' })` → download como `${OBJECT_NAME}.stl`.

### PNG
1. **Botão clicado** — `viewer-template.html:262–264` → `onclick="exportPNG(1|2|4)"`.
2. **Resize do canvas** — `viewer-template.html:452–454` → calcula `w = canvas.width * scale`, `h = canvas.height * scale`, chama `renderer.setSize(w, h)`.
3. **Render single-frame** — `viewer-template.html:455` → `renderer.render(scene, camera)` desenha no canvas redimensionado.
4. **Captura blob** — `viewer-template.html:456–459` → `canvas.toBlob(callback, 'image/png')`.
5. **Download e restore** — callback dispara `downloadBlob(blob, '${OBJECT_NAME}-${scale}x.png')` e em seguida chama `onResize()` para retornar o canvas ao tamanho do `#viewer`.

### Helper de download (compartilhado)
1. `downloadBlob(blob, filename)` — `viewer-template.html:426–431`:
   - `URL.createObjectURL(blob)` → cria URL temporária.
   - Cria `<a>` em memória com `href` e `download = filename`, dispara `.click()`.
   - `URL.revokeObjectURL(url)` libera a referência.

### Caminhos alternativos

- **Erro no GLTFExporter:** `viewer-template.html:438` → o callback de erro é `(err) => console.error(err)`. Nada visível ao usuário; o download simplesmente não acontece.
- **Mesh sem `.geometry`:** ainda não acontece no template (a mesh é sempre populada em `buildGeometry()` no init), mas se chamado antes do init, todos os exportadores quebram (não há guard).
- **Wireframe ativo:** `wireframe = true` afeta apenas a renderização (PNG sai com wireframe); GLB/OBJ/STL exportam a geometria sólida regardless.

## Arquivos Envolvidos

| Camada | Arquivo | Responsabilidade |
|--------|---------|------------------|
| UI | `_template/viewer-template.html` (linhas 256–266) | Botões de export na seção `.export-grid` |
| Handlers | `viewer-template.html` (`exportGLB/OBJ/STL/PNG`, linhas 435–461) | Expostos em `window` para os `onclick` inline |
| Helper | `viewer-template.html:426–431` (`downloadBlob`) | Cria `<a>` temporário e dispara download |
| Configuração | `viewer-template.html:433` (`const OBJECT_NAME`) | Nome usado em todos os arquivos exportados |
| Dependências | CDN (importmap em `viewer-template.html:161–168`) | `GLTFExporter`, `OBJExporter`, `STLExporter` |

## Regras de Negócio Relevantes

- **GLB é binário** — `viewer-template.html:438`: `{ binary: true }` produz `.glb` (compactado, single-file). Sem essa flag o exporter geraria `.gltf` (JSON).
- **STL é binário** — `viewer-template.html:447`: `{ binary: true }` retorna `DataView`. Sem a flag, STL ASCII (string) é gerado e funciona em ferramentas, mas arquivo fica muito maior.
- **PNG redimensiona o canvas e depois restaura** — `viewer-template.html:454, 459`: durante o instante do export o canvas fica esticado; o `onResize()` no callback corrige. Se o callback do `toBlob` demorar (geometrias densas), o usuário vê o canvas distorcido brevemente.
- **Nome do arquivo é fixo** — `viewer-template.html:433`: `OBJECT_NAME` é uma constante de módulo; todos os exports daquele viewer usam o mesmo nome base. Trocar para múltiplos objetos exige editar o código.
- **PNG não tem extensões adicionais** — não há suporte a JPEG, WebP, transparência configurável por export (a transparência depende do fundo escolhido em `setBackground`).

## Dependências Externas

- **CDN jsDelivr** para `three/addons/exporters/*`. Offline, todos os exports quebram (módulos não carregam).
- **API `URL.createObjectURL` + `<a download>`** — comportamento padrão de navegador moderno. Em sandboxes restritos (alguns Electron sem permissões, file://, iframes cross-origin) pode falhar silenciosamente.
- **API `canvas.toBlob`** — disponível em todos os navegadores modernos. Pode retornar `null` se o canvas estiver "tainted" (não é o caso aqui, pois não há textura cross-origin).

## Observações

- **Nenhum feedback visual.** Os botões não mostram loading nem sucesso/erro. Se o GLB demorar (geometria pesada), o usuário não sabe que está exportando.
- **`OBJECT_NAME` é a única forma de nomear.** Não há prompt do tipo "salvar como" — quem cria um objeto precisa lembrar de trocar a constante.
- **`exports/` não é usado pelos handlers.** O download cai em `~/Downloads`. A pasta `exports/` no repo é só convenção (e está vazia).
- **PNG transparente exige fundo `transparent`.** `setBackground('transparent')` em `viewer-template.html:399–402` é o único caminho — não há flag dedicada no botão.
- **Sem export de cena completa.** Os exporters recebem só `mesh`; luzes, câmera e grid **não** são incluídos no GLB/OBJ/STL. Isso é intencional (são objetos isolados), mas vale documentar.
- **PNG não respeita o aspect ratio escolhido.** O resize aplica `scale` sobre o tamanho atual do canvas, então a proporção sai igual à da janela do navegador no momento do export — não é possível pedir 1024×1024 fixo sem refatorar.
