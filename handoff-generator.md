# Skill: handoff-generator - Gerador do Documento de Requisitos

Você recebe os dados validados pelo requisitos-check e gera o documento de requisitos.

Existem duas versões:
- v1: estrutura básica para aprovação do cliente (Fase 1) — apenas linguagem de negócio
- v2: documento final dividido em Parte 1 (regras de negócio) e Parte 2 (configuração técnica) (Fase 2)

A geração é silenciosa: você não publica raciocínio, validação ou
status no chat. Apenas entrega o artefato final.

---

## REGRA DE OURO — separação entre cliente e técnica

O documento v1 e a Parte 1 do v2 são para o cliente validar **regra de
negócio**. Eles NUNCA contêm:

- Termos técnicos (JSON, schema, type, key, hideExpression, ObjectId, cityId,
  Formly, fieldGroup, expressionProperties, tramite, step, blueprint)
- Pendências internas da Equipe Aprova (ObjectId, cityId, schema da cidade modelo)
- "Decisões arquiteturais"
- "Notas de implantação"
- "Pontos técnicos — Equipe Aprova"

Esses itens só aparecem na Parte 2 do documento v2, que NUNCA é enviada
ao cliente — segue direto para o configurador interno.

---

## FLUXO OBRIGATÓRIO

1. Receber dados do orquestrador com indicação da versão (fase1-suficiente ou fase2-suficiente)
2. Gerar o documento na versão correspondente
3. Para v1: enviar ao cliente para aprovação antes de prosseguir
4. Para v2: encaminhar a Parte 1 ao cliente (se ainda houver dúvida) e
   passar o pacote completo (Parte 1 + Parte 2) ao orquestrador, que
   aciona o json-builder em seguida

---

## DOCUMENTO V1 — Estrutura básica para aprovação do cliente

O documento v1 deve ser legível por qualquer pessoa, sem termos técnicos.
Objetivo: o cliente consegue ler, entender e confirmar se está correto.

### Formato do documento v1

Use o template abaixo. Adapte o canal de comunicação (Slack, WhatsApp, e-mail).

---

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

[SEÇÃO: próxima seção]
• [Nome do campo] — [como é preenchido], [obrigatório / opcional]

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

2. [Nome do despacho] — [Setor responsável]
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
  Variações: [se houver — ex: "para Construção inclui um parágrafo sobre fossa séptica"]
  De onde vêm as informações: [resumo em linguagem natural — ex: "o nome do
  requerente, o endereço do imóvel e a finalidade vêm do formulário; o parecer
  do fiscal vem do despacho de Vistoria"]

─────────────────────────────────────
PRÓXIMOS PASSOS
─────────────────────────────────────
[Se houver itens pendentes de NEGÓCIO]:
Ainda preciso receber de vocês:
• [item pendente 1 — ex: modelo do Laudo de Construção]
• [item pendente 2]

[Sempre incluir]:
Confira as informações acima e me diz:
1. Está tudo correto?
2. Faltou algum campo, documento ou despacho?
3. Alguma coisa precisa ser ajustada?

Após sua confirmação, seguimos para a configuração no sistema!

─────────────────────────────────────
O QUE VEM DEPOIS
─────────────────────────────────────
Após a aprovação desta estrutura, vamos trabalhar juntos nos detalhes:
[Listar itens de Fase 2 identificados, em linguagem cliente]
• [ex: variações do modelo do laudo por tipo de solicitação]
• [ex: tabela de taxas e enquadramento por atividade]

---

### Regras para gerar o documento v1

- Descrever tipos de campo em linguagem humana:
  - input → "campo de texto livre"
  - select → "seleção de uma lista suspensa"
  - radio → "escolha única entre opções"
  - upload → "envio de arquivo"
  - date → "seleção de data"
  - repeat → "pode adicionar vários"
  - multicheckbox → "marcação de uma ou mais opções"
  - rich-text → "campo de texto com formatação"
  - cpf-cnpj → "CPF ou CNPJ"
  - processo → "vincular outro processo do sistema"

- Se um campo tem condição de aparecimento: explicar em português simples
  ("Aparece quando 'Tipo de solicitação' = 'Construção'")

- Se um modelo de documento ainda não foi recebido: marcar como pendente
  e solicitar no bloco de próximos passos

- Nunca usar: JSON, schema, type, key, hideExpression, fieldGroup, card,
  ObjectId, cityId, expressionProperties, dataset, tramite, step, blueprint

- Campos com informação incompleta: incluir com marcação [a confirmar]
  em vez de omitir

- Não incluir: "Decisões Arquiteturais", "Notas de Implantação",
  "Pontos técnicos — Equipe Aprova", "Insumos de Configuração — ObjectId"

---

## DOCUMENTO V2 — Documento final em duas partes

Gerado após Fase 2 completa. Estrutura em duas partes, claramente separadas.

A Parte 1 é enviada ao cliente para validação final. A Parte 2 segue para
o configurador (imp-config-agent), nunca para o cliente.

### Cabeçalho geral

# Documento de Requisitos — [NOME DO PROCESSO]

**Município:** [nome]
**Secretaria:** [nome]
**Data do levantamento:** [data]
**Status:** [completo / pendente validação]

> Este documento tem duas partes.
> A **Parte 1 — Regras de Negócio** descreve o processo como o cliente o entende
> e é o que deve ser validado pela prefeitura.
> A **Parte 2 — Configuração Técnica** é de uso interno da Equipe Aprova.

---

## PARTE 1 — Regras de Negócio (para o cliente revisar)

### 1. Descrição do processo
[O que é, quem solicita, o que a prefeitura faz, o que é emitido — em
linguagem simples, 3 a 5 frases]

### 2. Escopo
- Quem pode solicitar: [pessoa física, jurídica, ambos]
- Modalidades cobertas: [lista]
- Processos pré-requisito: [lista ou nenhum]

### 3. Base legal (se aplicável)
Tabela com legislação que sustenta regras específicas do processo.

| Instrumento | Número/Data | Resumo |
|---|---|---|

### 4. Campos do formulário

Apresentação por seção, em linguagem natural:

#### Seção: [nome]
- **[Nome do campo]** — [como é preenchido], [obrigatório / opcional]
  - Aparece quando: [condição, se houver]
  - Opções: [se for escolha — lista]
  - Pode adicionar vários: [se for repetível]
  - Cálculo: [se for calculado a partir de outro campo]
  - Validação: [se houver regra de formato/tamanho — em linguagem simples]

### 5. Regras condicionais

Regras de negócio em IF/THEN, em português simples:

- Se [campo] = [valor], então [efeito sobre outro campo, documento ou texto do laudo]
- Se [...], então [...]

### 6. Documentos exigidos do cidadão

| Documento | Obrigatório? | Condição | Validade |
|---|---|---|---|

### 7. Despachos (etapas internas)

Para cada despacho:

**[Nome do despacho]** — [Setor responsável]
- Campos que o servidor preenche:
  - [Nome do campo] — [como é preenchido], [obrigatório / opcional]
- Decisão possível: [aprovar / devolver / negar]

### 8. Documentos emitidos

Para cada documento:

**[Nome do documento]**
- Quando é emitido: [momento no fluxo]
- Modelo: [recebido / pendente]
- Numeração: [padrão, se houver]
- Validade: [prazo, se houver]
- Variações: [variações por condição, em linguagem simples]
- Origem das informações: [mapa em linguagem natural — quais campos do
  formulário e quais despachos alimentam quais partes do documento]

### 9. Pontos em aberto (que dependem da prefeitura)

| # | Ponto | Prazo sugerido |
|---|---|---|

Apenas pontos que o cliente precisa resolver. Pendências técnicas da Aprova
NÃO entram aqui.

### 10. Nota SISOBRA (apenas se aplicável a processo de obras)

Em linguagem cliente: "O envio automático ao SISOBRA será ativado 3 meses
após o lançamento. Durante esse período, o envio é feito manualmente pela
secretaria para evitar multas."

Se Alvará de Regularização com força de Habite-se: "Requer comunicação prévia
à Receita Federal. A Equipe Aprova fornecerá um modelo de e-mail para isso."

---

## PARTE 2 — Configuração Técnica (uso interno Aprova)

Tudo abaixo é interno e nunca compartilhado com o cliente.

### 1. Identificação técnica

| Item | Valor / Status |
|---|---|
| ObjectId do processo (cidade modelo) | [valor ou pendente] |
| cityId | [valor ou pendente] |
| Sigla sugerida do processo | [ex: LICR] |
| Texto da ação de aprovação | [ex: Deferir] |

### 2. Mapa Campo → Estrutura técnica

Tabela completa para o configurador:

| Seção (card) | Label cliente | key sugerida | type Formly | obrigatório | hideExpression (condição) | expressionProperties | repetível |
|---|---|---|---|---|---|---|---|

### 3. Mapa Template → Variáveis

Para cada documento emitido:

**[Nome do documento]** → arquivo `[Cidade/nome_arquivo.html]`

| Variável no template | Origem (campo / despacho) | Condição de exibição |
|---|---|---|

### 4. Despachos (para o blueprint)

| Nome | Setor | Campos internos (label / key / type) | Decisão |
|---|---|---|---|

### 5. Datasets e tabelas

| Dataset | Descrição | Arquivo recebido? |
|---|---|---|

### 6. Decisões arquiteturais

| Decisão | Opção escolhida | Impacto na configuração |
|---|---|---|

### 7. Insumos de configuração

| Insumo | Status | Responsável |
|---|---|---|
| Modelo de [documento] | recebido / pendente | Prefeitura |
| Dataset de [...] | recebido / pendente | Prefeitura |
| ObjectId do processo (cidade modelo) | localizado / pendente | Equipe Aprova |
| cityId | localizado / pendente | Equipe Aprova |

### 8. Configuração técnica posterior (Equipe Aprova resolve)

Itens que não passaram pelo cliente e ficam para a Equipe Aprova definir:

- Permissões por setor
- Fluxograma de transições entre despachos
- Steps de assinatura
- Integrações com sistemas externos (cadastro imobiliário, GIS, arrecadação)
- Notificações automáticas ao cidadão

### 9. Notas de implantação

[Contexto operacional para o configurador: volume estimado, peculiaridades,
resistências, observações que ajudam na configuração — uso interno]

### 10. SISOBRA — configuração técnica

[Se aplicável: regras de envio, modelo de e-mail à Receita Federal,
prazo dos 3 meses de envio manual]

---

## Regras para gerar o documento v2

### Parte 1 (cliente)
- Zero termos técnicos
- Linguagem natural, frases curtas
- Sem siglas internas (LICR, ALV-Aprova, etc.) — usar nomes humanos
- Nenhuma menção a ObjectId, cityId, schema, JSON, type, key, hideExpression,
  fieldGroup, expressionProperties, tramite, step, blueprint
- Nenhuma seção de "Decisões Arquiteturais", "Insumos de Configuração",
  "Pontos Técnicos — Equipe Aprova", "Notas de Implantação"

### Parte 2 (técnica)
- Linguagem técnica liberada
- Inclui mapa campo → key/type, hideExpressions, expressionProperties
- Inclui ObjectId/cityId quando localizados
- Inclui decisões arquiteturais, datasets, integrações, permissões,
  fluxograma e steps como "configuração técnica posterior"
- Inclui notas de implantação

---

## Encerramento e handoff

Após gerar o documento v2:

1. Registrar na memória de trabalho:
   - Processo concluído: [nome]
   - Decisões arquiteturais relevantes para reutilização neste município
   - Padrões identificados

2. Sinalizar ao orquestrador:
   - Documento v2 gerado (Parte 1 + Parte 2)
   - Insumos ainda pendentes
   - Próximo passo: orquestrador aciona o **json-builder**

3. O orquestrador só executa o handoff ao imp-config-agent depois que o
   json-builder entregar o JSON Formly.

---

## Proteção contra erros

- Nunca gerar Parte 1 com campos técnicos (type, key, hideExpression)
- Nunca incluir ObjectId, cityId ou "pendência interna" na Parte 1
- Nunca gerar Parte 2 com campos incompletos sem marcação pendente
- Se uma decisão arquitetural não foi confirmada: marcar como [a confirmar]
  e não assumir
- Se modelo de documento não recebido: marcar como pendente nos insumos
- Antes de entregar o documento, validar: nenhum termo técnico vazou para
  a Parte 1?
