# Skill: json-builder — Geração do JSON Formly de Configuração

Você recebe a Parte 1 (Regras de Negócio) validada pelo cliente e a Parte 2
(Configuração Técnica) gerada pelo handoff-generator, e produz o JSON Formly
do formulário, pronto para o configurador (imp-config-agent) aplicar no
ambiente da cidade.

Esta skill é estritamente técnica. Não interage com o cliente. Sua saída é
apenas o JSON e, opcionalmente, um relatório curto ao orquestrador com
itens que ficaram como TODO.

---

## QUANDO É ACIONADA

Depois que o handoff-generator concluiu a Fase 2 (gerou Parte 1 + Parte 2)
e o orquestrador autorizou o handoff.

---

## ENTRADA ESPERADA

Pacote estruturado do handoff-generator com pelo menos:

- Identificação do processo (nome, sigla, cidade, secretaria)
- ObjectId / cityId (quando localizados — se pendentes, deixar placeholder)
- Lista de seções (cards) com seus campos detalhados
- Lista de documentos exigidos do cidadão (uploads)
- Lista de despachos com seus campos internos
- Lista de documentos emitidos com mapa de variáveis
- Regras condicionais em IF/THEN (em linguagem natural)
- Datasets / opções de seleção

---

## ESTRUTURA MÍNIMA DO JSON

O JSON segue o padrão dos processos existentes da Aprova Digital. Estrutura
de topo:

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
    "templates": [ ... ],
    "form_tramites": [],
    "fluxograma": { "fields": [] },
    "tenance": {},
    "initialReceiverConfig": {}
  },
  "city_id": "<cityId>",
  "form": [
    {
      "fieldGroup": [ <cards> ]
    }
  ],
  "tramite": [],
  "step": { "config": [], "shared": {} },
  "options": { "showStepViewControl": true }
}
```

Notas:
- `form_tramites` permanece como `[]` — despachos vão para o blueprint, não
  para o schema do formulário.
- `tramite` e `step.config` permanecem como listas vazias por padrão.
- Quando ObjectId / cityId estiverem pendentes, gravar um placeholder
  identificável (`"<OBJECTID_PENDENTE>"`, `"<CITYID_PENDENTE>"`) e listar
  como TODO no relatório.

---

## ESTRUTURA DE UM CARD (SEÇÃO DO FORMULÁRIO)

Cada item do array `form[0].fieldGroup` é um card:

```json
{
  "wrappers": ["text-right", "step"],
  "templateOptions": {
    "title": "<Nome da seção visível ao cidadão>",
    "subtitle": "",
    "textHtml": "<HTML de orientação ao cidadão — opcional>"
  },
  "fieldGroup": [ <campos> ],
  "expressionProperties": {}
}
```

Notas:
- `templateOptions.title` é o que aparece ao cidadão como nome da etapa.
- `textHtml` recebe o bloco de orientação convertido em HTML (use `<ol>`,
  `<ul>`, `<b>`, `<br>` — evite estilos inline complexos quando não houver
  necessidade).
- Para cards de área interna (escondidos do cidadão), usar `wrappers` sem
  "step" e aplicar `hideExpression` no card todo.

---

## TRADUÇÃO TIPO HUMANO → TYPE FORMLY

Tabela canônica de conversão. Quando a entrada disser um dos termos da
coluna esquerda, gere um campo do `type` correspondente.

| Termo coletado | `type` Formly | Notas |
|---|---|---|
| Texto livre, "digitando texto" | `next-input` | `templateOptions.type` = "text" |
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

## ESTRUTURA DE CAMPO

Padrão de um campo (substituir conforme o tipo):

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

Para `radio` / `select` / `multicheckbox` adicionar:

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

Para `next-upload` adicionar:

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
    "fieldGroup": [ <campos do item repetível> ]
  }
}
```

---

## REGRAS CONDICIONAIS — `hideExpression`

A entrada vem em linguagem natural ("Aparece quando 'Tipo de solicitação'
= 'Construção'"). Traduzir para `hideExpression`, que ESCONDE o campo
quando a expressão for verdadeira (note a inversão lógica).

| Regra de negócio | hideExpression |
|---|---|
| Campo só aparece quando `solicitacao` = "construcao" | `"model['solicitacao'] !== 'construcao'"` |
| Campo só aparece quando `solicitacao` contém "construcao" (multi-check) | `"!model['solicitacao'] || model['solicitacao'].indexOf('construcao') === -1"` |
| Campo só aparece quando `tipo_pessoa` = "fisica" | `"model['tipo_pessoa'] !== 'fisica'"` |
| Campo sempre visível | omitir `hideExpression` |

Sempre referencie campos pela `key`, nunca pelo label.

---

## REGRAS DE OBRIGATORIEDADE CONDICIONAL — `expressionProperties`

Quando "campo é obrigatório APENAS em alguma situação":

```json
"expressionProperties": {
  "templateOptions.required": "model['tipo_pessoa'] === 'juridica'"
}
```

Para validações dinâmicas (ex: máximo de 10 caracteres):

```json
"expressionProperties": {
  "templateOptions|||required": "model['numero_predial']?.length > 10"
}
```

---

## DOCUMENTOS EMITIDOS — `config.templates`

Para cada documento emitido pela Parte 1, adicionar uma entrada em
`config.templates`:

```json
{
  "icon": "ion-ios-print",
  "text_button": "<Nome curto do botão>",
  "template": "<Cidade>/<nome_arquivo>.html",
  "function": "processComplete",
  "internal": false,
  "permission": "process:viewAll:all",
  "official": true,
  "possui_prazo_validade": <true se houver validade, false caso contrário>,
  "viewed": true
}
```

Notas:
- Pasta da cidade segue PascalCase + UF (ex: `RioDasOstrasRJ`,
  `FormigaMG`). Se não houver convenção definida, gerar placeholder
  `<Cidade><UF>` e marcar como TODO.
- Nome do arquivo em snake_case sem acento (ex:
  `laudo_vistoria_ambiental.html`).
- Para peças gráficas, adicionar `"wk": true`.

---

## DESPACHOS — NÃO ENTRAM NO JSON DO FORMULÁRIO

Despachos coletados na Fase 1 NÃO viram entradas em `form_tramites`. Eles
serão configurados no blueprint pelo configurador.

A saída desta skill deve incluir, junto ao JSON Formly, um bloco
complementar `despachos[]` em formato estruturado para o blueprint:

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

Esse bloco vai como anexo ao handoff técnico, não dentro do JSON do
formulário.

---

## FLUXO DE GERAÇÃO

1. Receber pacote do orquestrador
2. Identificar processo, cidade, sigla, ObjectId/cityId (ou placeholders)
3. Construir `config` (título, descrição, cidade, sigla, templates)
4. Para cada seção da Parte 1, construir um card em `form[0].fieldGroup`
5. Para cada campo da Parte 1, construir o objeto de campo:
   a. Resolver `type` pela tabela de tradução
   b. Resolver `key` (snake_case, único no schema)
   c. Resolver `templateOptions` (label, placeholder, required, options, accept, etc.)
   d. Resolver `hideExpression` a partir da regra condicional, se houver
   e. Resolver `expressionProperties` para required condicional ou validação
6. Construir blocos de instrução (`next-html`) a partir dos textos de
   orientação coletados
7. Para cada documento emitido, criar entrada em `config.templates`
8. Para cada despacho, gerar entrada no bloco complementar `despachos[]`
9. Marcar TODOs:
   - ObjectId pendente
   - cityId pendente
   - Modelos de template pendentes
   - Datasets pendentes
   - Pasta da cidade não definida
10. Entregar ao orquestrador: JSON + `despachos[]` + relatório de TODOs

---

## ENTREGÁVEIS

A saída desta skill é composta por três artefatos, todos para uso interno:

1. `<sigla>_<cidade>.json` — JSON Formly do formulário
2. `<sigla>_<cidade>_despachos.yaml` — Lista estruturada de despachos
   para o blueprint
3. Relatório curto ao orquestrador:
   - Itens que viraram placeholder (ObjectId, cityId, pasta de templates)
   - Modelos de documento ainda pendentes
   - Regras condicionais que precisaram de aproximação por ambiguidade
   - Quaisquer suposições feitas

---

## PROTEÇÃO CONTRA ERROS

- Nunca inventar ObjectId / cityId — usar placeholder identificável
- Nunca inventar pasta de templates — usar placeholder e listar como TODO
- Nunca inventar opções de select / radio — se faltar, marcar como TODO
- Antes de entregar, validar:
  - JSON é parseable
  - Todo campo tem `key` única
  - Todo `hideExpression` referencia uma `key` existente no schema
  - Todo template em `config.templates` tem `template` e `text_button`
  - Toda condicional ambígua está documentada no relatório de TODOs

---

## ESCALONAMENTO

Esta skill nunca pergunta ao cliente. Se uma informação técnica crítica
faltar e impedir a geração, escalar ao implantador (não ao cliente):

"Para gerar o JSON final preciso do(s) seguinte(s) item(ns):
- [item]
Posso prosseguir com placeholder e marcar como TODO, ou prefere resolver
antes?"
