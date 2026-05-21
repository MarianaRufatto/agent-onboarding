# Skill: handoff-generator - Gerador do Documento de Requisitos

Esta skill unificada cobre dois estágios do pipeline de implantação:

- **Fase A — Handoff** (ex-`handoff-generator`): gera o Documento de Requisitos
  nas versões v1 (aprovação do cliente) e v2 (completo, com Parte 1 para o
  cliente e Parte 2 técnica para o configurador).
- **Fase B — JSON Builder** (ex-`json-builder`): a partir do pacote v2 validado,
  produz o JSON Formly do formulário, o bloco de despachos e o relatório de TODOs.

A geração é silenciosa em ambas as fases: nenhum raciocínio, validação
intermediária ou status é publicado no chat — apenas o artefato final.

---

## REGRA DE OURO — separação cliente / técnico

Qualquer artefato destinado ao cliente (v1, Parte 1 do v2) **nunca** contém:

- Termos técnicos: JSON, schema, type, key, hideExpression, ObjectId, cityId,
  Formly, fieldGroup, expressionProperties, tramite, step, blueprint, card
- Pendências internas da Equipe Aprova
- Seções de "Decisões Arquiteturais", "Notas de Implantação" ou
  "Pontos Técnicos — Equipe Aprova"

Esses itens aparecem **apenas** na Parte 2 do v2 e no JSON Formly —
jamais compartilhados com o cliente.

---

## QUANDO CADA FASE É ACIONADA

| Fase | Gatilho |
|---|---|
| **A — v1** | Dados iniciais coletados; orquestrador indica `fase1-suficiente` |
| **A — v2** | Fase 2 completa; orquestrador indica `fase2-suficiente` |
| **B — JSON Builder** | Após v2 gerado e orquestrador autorizar o handoff técnico |

---

## FASE A — HANDOFF (Documento de Requisitos)

### Entrada esperada

Dados validados pelo `requisitos-check` com:

- Identificação do processo (nome, sigla, cidade, secretaria)
- Seções e campos do formulário com suas regras
- Documentos exigidos do cidadão
- Despachos e campos internos
- Documentos emitidos e mapa de variáveis
- Regras condicionais em linguagem natural
- Datasets / opções de seleção
- Indicação da versão a gerar (`fase1-suficiente` ou `fase2-suficiente`)

---

### DOCUMENTO V1 — Estrutura básica para aprovação do cliente

Use o template abaixo. Adapte o canal de comunicação (Slack, WhatsApp, e-mail).

---

```
FORMULÁRIO DE REQUERIMENTO — [NOME DO PROCESSO]
Município: [nome] | Data: [data]

Esta é a estrutura que vamos configurar no sistema. Confira se está tudo correto
e me diga se algo precisa ser ajustado.

─────────────────────────────────────
CAMPOS DO FORMULÁRIO
─────────────────────────────────────

[SEÇÃO: nome da seção]
• [Nome do campo] — [como é preenchido], [obrigatório / opcional]
  Aparece quando: [condição em português simples, se houver]
  Opções: [se for escolha — lista das opções]
  Pode adicionar vários: [sim/não, se for repetível]

─────────────────────────────────────
DOCUMENTOS QUE O CIDADÃO PRECISA ENVIAR
─────────────────────────────────────
• [Nome do documento] — [sempre obrigatório / obrigatório quando: condição]
  Validade: [prazo, se houver]

─────────────────────────────────────
DESPACHOS (ETAPAS INTERNAS)
─────────────────────────────────────
1. [Nome do despacho] — [Setor responsável]
   O servidor preenche:
     • [Campo] — [como é preenchido], [obrigatório / opcional]
   Decisão possível: [aprovar / devolver / negar]

─────────────────────────────────────
DOCUMENTOS EMITIDOS
─────────────────────────────────────
• [Nome do documento]
  Quando é emitido: [ao deferir / na etapa X]
  Modelo: [recebido ✓ / pendente — precisamos que você nos envie]
  Numeração: [sim, ex: ALV-2025-001 / não]
  Variações: [se houver]
  De onde vêm as informações: [resumo em linguagem natural]

─────────────────────────────────────
PRÓXIMOS PASSOS
─────────────────────────────────────
[Se houver itens pendentes de negócio]:
Ainda preciso receber de vocês:
• [item pendente 1]
• [item pendente 2]

Confira as informações acima e me diz:
1. Está tudo correto?
2. Faltou algum campo, documento ou despacho?
3. Alguma coisa precisa ser ajustada?

Após sua confirmação, seguimos para a configuração no sistema!

─────────────────────────────────────
O QUE VEM DEPOIS
─────────────────────────────────────
Após a aprovação desta estrutura, vamos trabalhar juntos nos detalhes:
• [item de fase 2 em linguagem cliente]
```

---

#### Regras para gerar o v1

- Descrever tipos de campo em linguagem humana:

  | Type técnico | Linguagem humana |
  |---|---|
  | input | campo de texto livre |
  | select | seleção de uma lista suspensa |
  | radio | escolha única entre opções |
  | upload | envio de arquivo |
  | date | seleção de data |
  | repeat | pode adicionar vários |
  | multicheckbox | marcação de uma ou mais opções |
  | rich-text | campo de texto com formatação |
  | cpf-cnpj | CPF ou CNPJ |
  | processo | vincular outro processo do sistema |

- Condições de aparecimento: explicar em português simples
  ("Aparece quando 'Tipo de solicitação' = 'Construção'")
- Modelo de documento não recebido: marcar como pendente e solicitar nos próximos passos
- Campos com informação incompleta: incluir com marcação `[a confirmar]` em vez de omitir
- **Nunca usar**: JSON, schema, type, key, hideExpression, fieldGroup, card,
  ObjectId, cityId, expressionProperties, dataset, tramite, step, blueprint

---

### DOCUMENTO V2 — Documento final em duas partes

Gerado após Fase 2 completa. A **Parte 1** é enviada ao cliente para validação
final. A **Parte 2** segue para o configurador (`imp-config-agent`), nunca
para o cliente.

#### Cabeçalho geral

```markdown
# Documento de Requisitos — [NOME DO PROCESSO]

**Município:** [nome]
**Secretaria:** [nome]
**Data do levantamento:** [data]
**Status:** [completo / pendente validação]

> Este documento tem duas partes.
> A **Parte 1 — Regras de Negócio** descreve o processo como o cliente o
> entende e é o que deve ser validado pela prefeitura.
> A **Parte 2 — Configuração Técnica** é de uso interno da Equipe Aprova.
```

---

#### PARTE 1 — Regras de Negócio (para o cliente revisar)

**1. Descrição do processo**
O que é, quem solicita, o que a prefeitura faz, o que é emitido — 3 a 5 frases simples.

**2. Escopo**
- Quem pode solicitar: [pessoa física, jurídica, ambos]
- Modalidades cobertas: [lista]
- Processos pré-requisito: [lista ou nenhum]

**3. Base legal (se aplicável)**

| Instrumento | Número/Data | Resumo |
|---|---|---|

**4. Campos do formulário** — por seção, em linguagem natural:

```
#### Seção: [nome]
- **[Nome do campo]** — [como é preenchido], [obrigatório / opcional]
  - Aparece quando: [condição, se houver]
  - Opções: [se for escolha]
  - Pode adicionar vários: [se repetível]
  - Cálculo: [se calculado]
  - Validação: [se houver regra — em linguagem simples]
```

**5. Regras condicionais** — IF/THEN em português simples:
- Se [campo] = [valor], então [efeito]

**6. Documentos exigidos do cidadão**

| Documento | Obrigatório? | Condição | Validade |
|---|---|---|---|

**7. Despachos (etapas internas)**

```
**[Nome do despacho]** — [Setor responsável]
- Campos que o servidor preenche:
  - [Nome do campo] — [como é preenchido], [obrigatório / opcional]
- Decisão possível: [aprovar / devolver / negar]
```

**8. Documentos emitidos**

```
**[Nome do documento]**
- Quando é emitido: [momento no fluxo]
- Modelo: [recebido / pendente]
- Numeração: [padrão, se houver]
- Validade: [prazo, se houver]
- Variações: [por condição, em linguagem simples]
- Origem das informações: [mapa em linguagem natural]
```

**9. Pontos em aberto (que dependem da prefeitura)**

| # | Ponto | Prazo sugerido |
|---|---|---|

Apenas pontos que o cliente precisa resolver. Pendências técnicas da Aprova NÃO entram aqui.

**10. Nota SISOBRA** (apenas se processo de obras)

> "O envio automático ao SISOBRA será ativado 3 meses após o lançamento.
> Durante esse período, o envio é feito manualmente pela secretaria para
> evitar multas."
>
> Se Alvará de Regularização com força de Habite-se: "Requer comunicação
> prévia à Receita Federal. A Equipe Aprova fornecerá um modelo de e-mail."

---

#### PARTE 2 — Configuração Técnica (uso interno Aprova)

**1. Identificação técnica**

| Item | Valor / Status |
|---|---|
| ObjectId do processo (cidade modelo) | [valor ou pendente] |
| cityId | [valor ou pendente] |
| Sigla sugerida do processo | [ex: LICR] |
| Texto da ação de aprovação | [ex: Deferir] |

**2. Mapa Campo → Estrutura técnica**

| Seção (card) | Label cliente | key sugerida | type Formly | obrigatório | hideExpression | expressionProperties | repetível |
|---|---|---|---|---|---|---|---|

**3. Mapa Template → Variáveis**

```
**[Nome do documento]** → arquivo `[Cidade/nome_arquivo.html]`

| Variável no template | Origem (campo / despacho) | Condição de exibição |
|---|---|---|
```

**4. Despachos (para o blueprint)**

| Nome | Setor | Campos internos (label / key / type) | Decisão |
|---|---|---|---|

**5. Datasets e tabelas**

| Dataset | Descrição | Arquivo recebido? |
|---|---|---|

**6. Decisões arquiteturais**

| Decisão | Opção escolhida | Impacto na configuração |
|---|---|---|

**7. Insumos de configuração**

| Insumo | Status | Responsável |
|---|---|---|
| Modelo de [documento] | recebido / pendente | Prefeitura |
| Dataset de [...] | recebido / pendente | Prefeitura |
| ObjectId do processo | localizado / pendente | Equipe Aprova |
| cityId | localizado / pendente | Equipe Aprova |

**8. Configuração técnica posterior (Equipe Aprova resolve)**
- Permissões por setor
- Fluxograma de transições entre despachos
- Steps de assinatura
- Integrações com sistemas externos
- Notificações automáticas ao cidadão

**9. Notas de implantação**
[Contexto operacional para o configurador — volume estimado, peculiaridades,
resistências, observações — uso interno]

**10. SISOBRA — configuração técnica** (se aplicável)
[Regras de envio, modelo de e-mail à Receita Federal, prazo de envio manual]

---

### Encerramento e handoff (após v2)

1. Registrar na memória de trabalho:
   - Processo concluído, decisões arquiteturais reutilizáveis, padrões identificados
2. Sinalizar ao orquestrador:
   - Documento v2 gerado (Parte 1 + Parte 2)
   - Insumos ainda pendentes
   - **Próximo passo:** orquestrador aciona a Fase B desta skill (JSON Builder)

---

## FASE B — JSON BUILDER (Geração do JSON Formly)

### Entrada esperada

Pacote estruturado do orquestrador com pelo menos:

- Identificação do processo (nome, sigla, cidade, secretaria)
- ObjectId / cityId (ou marcadores de pendência)
- Lista de seções (cards) com campos detalhados (vinda da Parte 2 do v2)
- Lista de documentos exigidos do cidadão
- Lista de despachos com campos internos
- Lista de documentos emitidos com mapa de variáveis
- Regras condicionais em IF/THEN (linguagem natural)
- Datasets / opções de seleção

---

### Estrutura mínima do JSON Formly

```json
{
  "_id": "<ObjectId ou placeholder>",
  "__v": 0,
  "config": {
    "title": "<Nome do processo>",
    "descricao": "<Descrição curta para o cidadão>",
    "cidade": "<cityId>",
    "sigla": "<Sigla — ex: LICR>",
    "approveActionText": "<Deferir / Aprovar / Conceder>",
    "devolverAoUltimo": true,
    "experimentalMode": true,
    "newSignatureStrategy": true,
    "templates": [ ],
    "form_tramites": [],
    "fluxograma": { "fields": [] },
    "tenance": {},
    "initialReceiverConfig": {}
  },
  "city_id": "<cityId>",
  "form": [
    {
      "fieldGroup": [ ]
    }
  ],
  "tramite": [],
  "step": { "config": [], "shared": {} },
  "options": { "showStepViewControl": true }
}
```

> `form_tramites` permanece `[]` — despachos vão para o blueprint.
> `tramite` e `step.config` permanecem listas vazias por padrão.
> Quando ObjectId / cityId estiverem pendentes, gravar placeholder
> identificável (`"<OBJECTID_PENDENTE>"`, `"<CITYID_PENDENTE>"`) e listar
> como TODO no relatório.

---

### Estrutura de um card (seção do formulário)

Cada item do array `form[0].fieldGroup` é um card:

```json
{
  "wrappers": ["text-right", "step"],
  "templateOptions": {
    "title": "<Nome da seção visível ao cidadão>",
    "subtitle": "",
    "textHtml": "<HTML de orientação ao cidadão — opcional>"
  },
  "fieldGroup": [ ],
  "expressionProperties": {}
}
```

> Para cards de área interna (escondidos do cidadão), usar `wrappers` sem
> `"step"` e aplicar `hideExpression` no card todo.
> `textHtml` usa `<ol>`, `<ul>`, `<b>`, `<br>` — evitar estilos inline
> complexos sem necessidade.

---

### Tradução tipo humano → type Formly

| Termo coletado | `type` Formly | Notas |
|---|---|---|
| Texto livre | `next-input` | `templateOptions.type` = "text" |
| Número | `next-input` | `templateOptions.type` = "number" |
| Editor de texto rich | `next-ck` | Para observações longas |
| Lista suspensa | `select` ou `next-select` | options no `templateOptions` |
| Escolha única visível | `radio` | options no `templateOptions` |
| Caixas de marcação | `multicheckbox` | options no `templateOptions` |
| Envio de arquivo | `next-upload` | aceita 1 ou múltiplos |
| Data | `next-input-data` | |
| CPF ou CNPJ | `next-cpf-cnpj` | validação embutida |
| Pode adicionar vários | `next-repeat` | wrap em torno de subcampos |
| Vincular outro processo | `next-processo` | referência a processo existente |
| Bloco de instrução / aviso (HTML) | `next-html` | usa `textHtml` |
| Taxas | `next-taxas` | usa `taxa` interno |
| Remetente | `next-from` | metadados do solicitante |
| Destinatário | `next-destinatario` | |
| Setor | `sector` | |

---

### Estrutura de campo

Padrão base:

```json
{
  "key": "<key_sugerida_snake_case>",
  "type": "next-input",
  "wrappers": ["form-field"],
  "templateOptions": {
    "label": "<Label visível>",
    "placeholder": "<Placeholder>",
    "focus": "false",
    "type": "text",
    "required": true
  }
}
```

Para `radio` / `select` / `multicheckbox`:

```json
"templateOptions": {
  "label": "...",
  "required": true,
  "options": [
    { "value": "construcao", "label": "Construção" },
    { "value": "regularizacao", "label": "Regularização" }
  ]
}
```

Para `next-upload`:

```json
"templateOptions": {
  "label": "Registro de Imóvel",
  "required": true,
  "max": 1,
  "accept": ".pdf"
}
```

Para `next-repeat`:

```json
{
  "key": "responsavel_tecnico",
  "type": "next-repeat",
  "templateOptions": { "label": "Responsável Técnico" },
  "fieldArray": {
    "fieldGroup": [ ]
  }
}
```

---

### Regras condicionais — `hideExpression`

`hideExpression` **esconde** o campo quando a expressão é verdadeira
(inversão lógica em relação à regra de negócio):

| Regra de negócio | `hideExpression` |
|---|---|
| Aparece quando `solicitacao` = "construcao" | `"model['solicitacao'] !== 'construcao'"` |
| Aparece quando `solicitacao` contém "construcao" (multi-check) | `"!model['solicitacao'] \|\| model['solicitacao'].indexOf('construcao') === -1"` |
| Aparece quando `tipo_pessoa` = "fisica" | `"model['tipo_pessoa'] !== 'fisica'"` |
| Sempre visível | omitir `hideExpression` |

Sempre referenciar campos pela `key`, nunca pelo label.

---

### Obrigatoriedade condicional — `expressionProperties`

Quando o campo é obrigatório apenas em alguma situação:

```json
"expressionProperties": {
  "templateOptions.required": "model['tipo_pessoa'] === 'juridica'"
}
```

Para validações dinâmicas:

```json
"expressionProperties": {
  "templateOptions|||required": "model['numero_predial']?.length > 10"
}
```

---

### Documentos emitidos — `config.templates`

Para cada documento emitido, adicionar uma entrada em `config.templates`:

```json
{
  "icon": "ion-ios-print",
  "text_button": "<Nome curto do botão>",
  "template": "<Cidade>/<nome_arquivo>.html",
  "function": "processComplete",
  "internal": false,
  "permission": "process:viewAll:all",
  "official": true,
  "possui_prazo_validade": false,
  "viewed": true
}
```

> Pasta da cidade em PascalCase + UF (ex: `RioDasOstrasRJ`, `FormigaMG`).
> Se não houver convenção definida, gerar placeholder `<Cidade><UF>` e
> marcar como TODO.
> Nome do arquivo em snake_case sem acento (ex: `laudo_vistoria_ambiental.html`).
> Para peças gráficas, adicionar `"wk": true`.

---

### Despachos — bloco complementar para o blueprint

Despachos **não entram** em `form_tramites`. São gerados como bloco YAML
separado para o blueprint:

```yaml
despachos:
  - nome: "Análise Documental"
    setor: "Protocolo"
    campos:
      - key: "conferencia_docs"
        type: "next-ck"
        label: "Conferência da documentação"
        required: true
    decisoes:
      - "Aceitar para vistoria"
      - "Devolver para complementação"
```

---

### Fluxo de geração do JSON

1. Receber pacote do orquestrador
2. Identificar processo, cidade, sigla, ObjectId/cityId (ou placeholders)
3. Construir `config` (título, descrição, cidade, sigla, templates)
4. Para cada seção da Parte 1, construir um card em `form[0].fieldGroup`
5. Para cada campo:
   - a. Resolver `type` pela tabela de tradução
   - b. Resolver `key` (snake_case, único no schema)
   - c. Resolver `templateOptions` (label, placeholder, required, options, accept…)
   - d. Resolver `hideExpression` a partir da regra condicional, se houver
   - e. Resolver `expressionProperties` para required condicional ou validação
6. Construir blocos de instrução (`next-html`) a partir dos textos de orientação
7. Para cada documento emitido, criar entrada em `config.templates`
8. Para cada despacho, gerar entrada no bloco `despachos[]` YAML
9. Marcar TODOs (ver seção de proteção abaixo)
10. Entregar: JSON + `despachos[]` YAML + relatório de TODOs

---

### Entregáveis da Fase B

| Artefato | Nome | Destino |
|---|---|---|
| JSON Formly do formulário | `<sigla>_<cidade>.json` | Configurador (imp-config-agent) |
| Despachos para o blueprint | `<sigla>_<cidade>_despachos.yaml` | Configurador (blueprint) |
| Relatório de TODOs | (inline ao orquestrador) | Orquestrador |

O relatório de TODOs inclui:
- Itens que viraram placeholder (ObjectId, cityId, pasta de templates)
- Modelos de documento ainda pendentes
- Regras condicionais que precisaram de aproximação por ambiguidade
- Quaisquer suposições feitas

---

## PROTEÇÃO CONTRA ERROS — ambas as fases

### Fase A (Handoff)
- Nunca gerar Parte 1 com campos técnicos (type, key, hideExpression)
- Nunca incluir ObjectId, cityId ou pendência interna na Parte 1
- Nunca gerar Parte 2 com campos incompletos sem marcação `[pendente]`
- Se decisão arquitetural não confirmada: marcar como `[a confirmar]`
- Antes de entregar: validar que nenhum termo técnico vazou para a Parte 1

### Fase B (JSON Builder)
- Nunca inventar ObjectId / cityId — usar placeholder identificável
- Nunca inventar pasta de templates — usar placeholder e listar como TODO
- Nunca inventar opções de select / radio — se faltar, marcar como TODO
- Antes de entregar, validar:
  - JSON é parseable
  - Todo campo tem `key` única
  - Todo `hideExpression` referencia uma `key` existente no schema
  - Todo template em `config.templates` tem `template` e `text_button`
  - Toda condicional ambígua está documentada no relatório de TODOs

### Escalonamento (Fase B)
Esta fase nunca pergunta ao cliente. Se uma informação técnica crítica faltar
e impedir a geração, escalar ao **implantador** (não ao cliente):

> "Para gerar o JSON final preciso do(s) seguinte(s) item(ns):
> - [item]
> Posso prosseguir com placeholder e marcar como TODO, ou prefere resolver antes?"

---

## FLUXO COMPLETO DA SKILL

```
orquestrador
    │
    ├─ fase1-suficiente ──► Gerar v1 ──► enviar ao cliente ──► aguardar aprovação
    │
    ├─ fase2-suficiente ──► Gerar v2
    │                           ├─ Parte 1 ──► cliente (validação final)
    │                           └─ Parte 2 ──► orquestrador
    │
    └─ handoff autorizado ──► Gerar JSON Formly + despachos YAML + TODOs
                                    └─► imp-config-agent
```
