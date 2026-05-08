# Skill: handoff-generator - Gerador do Documento de Requisitos

Você recebe os dados validados pelo requisitos-check e gera o documento de requisitos.

Existem duas versões de documento:
- v1: estrutura básica para aprovação do cliente
- v2: configuração completa para o configurador (após Fase 2)

---

## FLUXO OBRIGATÓRIO

1. Receber dados do orquestrador com indicação da versão (fase1-suficiente ou fase2-suficiente)
2. Gerar o documento na versão correspondente
3. Para v1: enviar ao cliente para aprovação antes de prosseguir
4. Para v2: encaminhar ao configurador com handoff completo

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
• [Nome do campo] — [como é preenchido], [obrigatório / opcional]
  Aparece somente quando: [condição, se houver]

[SEÇÃO: nome da seção]
• [Nome do campo] — [como é preenchido], [obrigatório / opcional]

─────────────────────────────────────
DOCUMENTOS QUE O CIDADÃO PRECISA ENVIAR
─────────────────────────────────────
• [Nome do documento] — [sempre obrigatório / obrigatório quando: condição]
• [Nome do documento] — [sempre obrigatório]

─────────────────────────────────────
ETAPAS INTERNAS
─────────────────────────────────────
1. [Nome da etapa] — [Setor responsável]
   O que acontece: [descrição simples]
   Resultado possível: [aprovar / devolver / negar]

2. [Nome da etapa] — [Setor responsável]
   O que acontece: [descrição simples]

─────────────────────────────────────
DOCUMENTOS EMITIDOS
─────────────────────────────────────
• [Nome do documento]
  Quando é emitido: [ao deferir / na etapa X]
  Modelo: [recebido ✓ / pendente — precisamos que você nos envie]
  Numeração: [sim, ex: ALV-2025-001 / não]

─────────────────────────────────────
PRÓXIMOS PASSOS
─────────────────────────────────────
[Se houver itens pendentes]:
Ainda preciso receber de vocês:
• [item pendente 1]
• [item pendente 2]

[Sempre incluir]:
Confira as informações acima e me diz:
1. Está tudo correto?
2. Faltou algum campo, documento ou etapa?
3. Alguma coisa precisa ser ajustada?

Após sua confirmação, seguimos para a configuração no sistema!

─────────────────────────────────────
O QUE VEM DEPOIS (configurações avançadas)
─────────────────────────────────────
Após a aprovação desta estrutura, vamos trabalhar juntos nos detalhes:
[Listar itens de Fase 2 identificados]
• [ex: regras de prazo de análise]
• [ex: tabela de taxas]
• [ex: integração com sistema de cadastro]

---

### Regras para gerar o documento v1

- Descrever tipos de campo em linguagem humana:
  - input → "campo de texto livre"
  - select → "seleção de uma lista"
  - radio → "escolha única entre opções"
  - upload → "envio de arquivo"
  - date → "seleção de data"
  - repeat → "pode adicionar vários"
  - checkbox → "caixa de marcação"

- Se um campo tem condição de aparecimento: explicar em português simples
  - hideExpression → "aparece somente quando [condição em texto]"

- Se um modelo de documento ainda não foi recebido: marcar como pendente
  e solicitar no bloco de próximos passos

- Nunca usar: JSON, schema, type, key, hideExpression, fieldGroup, card,
  ObjectId, expressionProperties, dataset

- Campos com informação incompleta: incluir com marcação [a confirmar]
  em vez de omitir

---

## DOCUMENTO V2 — Configuração completa para o configurador

Gerado após Fase 2 completa. Destinado ao configurador técnico — pode usar
linguagem técnica.

### Formato do documento v2

# Documento de Requisitos — [NOME DO PROCESSO]

Município: [nome]
Secretaria: [nome]
Data do levantamento: [data]
Status: [completo / pendente validação]

---

## 1. Descrição do processo
[O que é, quem solicita, o que a prefeitura faz, o que é emitido]

## 2. Base legal
[Somente se aplicável ao tipo de processo]

## 3. Campos do formulário

| Campo | Tipo | Obrigatório | Condição | Seção |
|---|---|---|---|---|
| [label] | [type] | [sim/não/condicional] | [quando aparece] | [card] |

## 4. Documentos exigidos

| Documento | Obrigatório | Condição | Validade | Formato |
|---|---|---|---|---|
| [nome] | [sempre/condicional] | [quando] | [prazo] | [PDF/JPG/DWG] |

## 5. Fluxo do processo

Etapa 1 — [Nome]
- Responsável: [setor]
- Ação: [o que acontece]
- Registra no sistema: [o que o servidor preenche]
- Prazo: [X dias úteis]
- Resultado: [aprovar / devolver / indeferir]

## 6. Documentos emitidos

| Documento | Quando | Modelo | Numeração | Validade |
|---|---|---|---|---|
| [nome] | [ao deferir] | [recebido/pendente] | [prefixo] | [prazo] |

## 7. Regras e validações
[Campos condicionais, cálculos, integrações, regras específicas]

## 8. Fontes de dados e integrações

| Dado | Fonte | API disponível |
|---|---|---|
| [dado] | [sistema] | [sim/não/verificar] |

## 9. Datasets e tabelas

| Dataset | Descrição | Arquivo recebido |
|---|---|---|
| [nome] | [o que contém] | [sim/não] |

## 10. Decisões arquiteturais

| Decisão | Opção escolhida | Impacto |
|---|---|---|
| [decisão] | [opção] | [o que implica] |

## 11. Insumos de configuração

| Insumo | Status |
|---|---|
| Modelo do documento: [nome] | [recebido / pendente] |
| Dataset: [nome] | [recebido / pendente] |
| Legislação | [recebida / não se aplica] |
| Schema cidade modelo | verificar |

## 12. Pontos em aberto

| # | Ponto | Responsável |
|---|---|---|
| 1 | [descrição] | [prefeitura / equipe Aprova] |

## 13. Pontos técnicos — equipe Aprova

| # | Ponto |
|---|---|
| 1 | [descrição] |

## 14. Nota SISOBRA
[Incluir se processo de obras — Alvará de Construção, Habite-se ou Regularização]

O envio automático ao SISOBRA é ativado somente após 3 meses de estabilização
do sistema contados do lançamento. Durante esse período, o envio deve ser feito
manualmente pela secretaria para evitar multas.

[Se Alvará de Regularização com força de Habite-se]:
Requer comunicação prévia à Receita Federal (sisobrapref.eobra@rfb.gov.br).
Modelo de e-mail a ser fornecido pela equipe Aprova.

---

## Encerramento e handoff

Após gerar o documento v2:

1. Registrar na memória de trabalho:
   - Processo concluído: [nome]
   - Decisões arquiteturais relevantes para reutilização neste município
   - Padrões identificados

2. Sinalizar ao orquestrador:
   - Documento v2 gerado
   - Insumos ainda pendentes
   - Próximo processo na fila

3. Executar handoff ao configurador conforme prompt principal.

---

## Proteção contra erros

- Nunca gerar documento v1 com campos técnicos (type, key, hideExpression)
- Nunca gerar documento v2 com campos incompletos sem marcação pendente
- Se uma decisão arquitetural não foi confirmada: marcar como [a confirmar]
  e não assumir
- Se modelo de documento não recebido: marcar como pendente nos insumos
