---
name: brainstorming
description: You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation.
---

# Brainstorming

## O que esta skill faz

Funciona como a camada de inteligência de contexto antes de qualquer implementação. Antes de criar features, modificar comportamentos ou adicionar funcionalidades:

1. **Explora a intenção** do usuário para garantir entendimento preciso do que precisa ser feito
2. **Lê os flows relevantes** em `./flow/` para entender como o sistema funciona hoje
3. **Produz um briefing de contexto** com o que é necessário saber para implementar com segurança
4. **Ao final do trabalho, garante que flows afetados sejam atualizados** se mudanças estruturais aconteceram

Esta skill é puramente de análise e contexto — não gera planos, não escreve código, não chama outras skills. Após o briefing, o próximo passo fica a critério do usuário.

---

## Fase 1 — Entender a intenção

Antes de buscar qualquer arquivo, fixe mentalmente:

- **O quê?** O que o usuário quer criar, mudar ou adicionar?
- **Por quê?** Qual problema ou objetivo isso resolve?
- **Onde?** Quais features, telas ou camadas são afetadas?
- **Impacto?** O que muda no comportamento existente? O que permanece igual?

Se o pedido for ambíguo ou tiver múltiplas interpretações possíveis, faça **uma única pergunta de clarificação** — a mais importante para desbloquear o entendimento. Não pergunte o que você pode inferir do código.

---

## Fase 2 — Carregar contexto dos flows

Verifique se existe documentação em `./flow/`:

```bash
ls ./flow/ 2>/dev/null
```

### Se não existir a pasta `./flow/` ou estiver vazia

Continue para o Fase 3 sem contexto documental. Mencione ao usuário no briefing que não há flows documentados e sugira `/flow-init` para criar uma base documental do projeto — mas não bloqueie o trabalho por isso.

### Se existir

Identifique quais flows são relevantes para o pedido do usuário:

- Mapeie as palavras-chave do pedido (ex: "login", "perfil", "pagamento", "notificação")
- Verifique os nomes dos arquivos em `./flow/` — prefira `project-structure.md` para contexto geral, e flows específicos para features afetadas
- Leia os flows relevantes **na íntegra** — não pule seções

**O que extrair de cada flow lido:**

| Informação | Para que serve |
|---|---|
| Arquivos envolvidos (tabela do flow) | Saber exatamente onde mexer |
| Passo a Passo (ordem de execução) | Não quebrar o fluxo existente |
| Regras de Negócio | Não violar contratos existentes |
| Observações / pontos frágeis | Evitar bugs conhecidos |
| Dependências Externas | Saber quais integrações são afetadas |

Se o pedido afeta múltiplas features e cada uma tem flow, leia todos.

---

## Fase 3 — Briefing de contexto

Após entender a intenção e ler os flows, apresente ao usuário um briefing estruturado:

```
## Entendimento do Pedido
[Uma frase descrevendo o que você entendeu que precisa ser feito]

## Contexto Carregado
- Flows lidos: [lista dos arquivos lidos, ou "nenhum — ./flow/ não existe"]
- Features afetadas: [lista]
- Arquivos-chave envolvidos: [caminhos reais encontrados nos flows]

## O que já existe
[2–4 frases descrevendo como a feature funciona hoje, com base nos flows.
Se não houver flows, escreva "Nenhuma documentação disponível — análise baseada no código."]

## Pontos de Atenção
[Lista de conflitos, regras de negócio que podem ser afetadas, dependências surpresa,
ou avisos encontrados nas Observações dos flows. Se não houver nada relevante, omita.]

## Flows a Revisitar Após Implementação
- `flow/<nome>.md` — revisitar se [quais seções podem mudar com as mudanças planejadas]
- (ou: "Nenhum — não há flows documentados para as features afetadas")

## Próximos Passos Sugeridos
[Sugestão breve do caminho natural: ex: "Use /writing-plan para montar o plano antes de implementar"
ou "Feature simples — pode implementar diretamente seguindo o CLAUDE.md"]
```

O briefing deve ser conciso. Não repita informações dos flows palavra por palavra — sintetize o que é relevante para o pedido atual.

---

## Fase 4 — Após a implementação: atualizar flows afetados

Esta fase acontece **depois que o trabalho for concluído**, antes de declarar a tarefa completa.

### Quando atualizar um flow

Atualize se qualquer um destes aconteceu:

- Novos arquivos foram criados em uma feature que já tem flow documentado
- Responsabilidade de uma camada mudou (ex: lógica movida do Cubit para um Service)
- Ordem de execução do fluxo mudou
- Novas regras de negócio foram adicionadas
- Arquivos foram movidos, renomeados ou removidos
- Novas dependências externas foram adicionadas ao fluxo

### Quando NÃO atualizar

Não atualize se:

- Mudança foi puramente interna sem impacto na estrutura (ex: renomear variável local, extrair método privado)
- Correção de bug que mantém exatamente o mesmo comportamento observável
- Mudança de UI sem impacto em estado, domínio ou dados

### Como atualizar

Use a skill `flow` para regenerar o flow completo, ou edite diretamente o `./flow/<feature>.md` seguindo o template e as regras de qualidade definidos nela. Não duplique a lógica de documentação aqui — a skill `flow` é a fonte de verdade sobre como escrever e atualizar flows.

### Informe o usuário

Ao final, mencione quais flows foram atualizados e o que mudou em cada um. Se nenhum flow precisou ser atualizado, não mencione esta etapa.

---

## Regras Gerais

**Seja preciso** — cite apenas arquivos que você encontrou nos flows ou no código. Não invente caminhos.

**Não bloqueie** — se não houver flows ou se os flows não cobrirem a feature pedida, o briefing ainda tem valor (entendimento da intenção + sugestão de próximo passo). Nunca impeça o trabalho por falta de documentação.

**Não duplique** — se a informação já está clara nos flows, referencie em vez de transcrever. O briefing é uma síntese, não uma cópia.

**Idioma** — use o mesmo idioma da conversa com o usuário.
