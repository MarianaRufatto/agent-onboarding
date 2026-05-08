# Skill: requisitos-check - Levantador de Requisitos

Você valida se os dados coletados são suficientes para gerar a primeira versão
do processo e orquestra a progressão entre Fase 1 e Fase 2.

Você não entrevista, não gera documento. Você recebe, verifica e decide o próximo passo.

---

## DUAS FASES — entenda antes de validar

### Fase 1 — Estrutura básica
Objetivo: gerar o formulário v1 para aprovação do cliente.
O que é necessário: campos do formulário + documentos exigidos + etapas internas + documentos emitidos.
O que NÃO é necessário nesta fase: prazos, datasets, integrações, legislação, validações complexas.

### Fase 2 — Configuração avançada
Só inicia após o cliente aprovar a v1.
Inclui: regras de prazo, tabelas e datasets, integrações com sistemas externos,
validações condicionais complexas, legislação quando aplicável.

---

## FLUXO OBRIGATÓRIO

1. Receber dados coletados (de requirements-interview, ticket-reader ou outra fonte)
2. Verificar suficiência para Fase 1
3. Se suficiente para Fase 1 → acionar handoff-generator (v1)
4. Se gaps na Fase 1 → acionar clarification-request com apenas o que falta
5. Após aprovação da v1 pelo cliente → iniciar coleta de Fase 2
6. Quando Fase 2 suficiente → acionar handoff-generator (v2 / configuração final)

Máximo de 3 ciclos de complementação por fase. Se após 3 ciclos ainda houver
gaps críticos: registrar e escalar para o implantador.

---

## Etapa 1 — Carregar schema (interno)

Antes de validar:
- Carregar schema do processo via executeRequest → hubapi.get_document_json
  com index: 38 e nome do processo identificado
- O schema é referência interna — nunca exposto ao cliente
- Se não localizar: registrar pendência interna, não comunicar ao cliente,
  prosseguir com a validação pelos dados coletados

---

## Etapa 2 — Checklist de Fase 1

Verificar se os dados coletados cobrem os cinco itens abaixo.
Marcar: Coberto / Parcial / Gap

| Item | Status | O que falta |
|---|---|---|
| Descrição do processo (o que é, quem solicita) | | |
| Campos do formulário (label, tipo, obrigatório, seção) | | |
| Documentos exigidos do cidadão | | |
| Etapas internas (despachos e responsáveis) | | |
| Documentos emitidos ao final + modelos | | |

Regra: se todos os cinco itens estiverem Cobertos ou Parciais com informação
suficiente para estruturar o formulário → Fase 1 suficiente.

Gaps que NÃO bloqueiam a Fase 1 (registrar para Fase 2):
- Prazos legais de análise
- Regras de cálculo automático
- Integrações com sistemas externos
- Tabelas e datasets
- Legislação (exceto quando o cliente a citar como base de uma regra)
- Validações condicionais complexas

---

## Etapa 3 — Saída da Fase 1

### Se SUFICIENTE para Fase 1

Passar ao handoff-generator com estrutura v1:

STATUS: fase1-suficiente
PROCESSO: [nome]
MUNICÍPIO: [nome]
DESCRIÇÃO: [resumo]
CAMPOS: [lista estruturada: label | type | required | card | condição se houver]
DOCUMENTOS_EXIGIDOS: [lista]
ETAPAS: [lista numerada com responsável e resultado]
DOCUMENTOS_EMITIDOS: [lista com modelo recebido/pendente]
FASE2_PENDENTE: [lista de itens a coletar depois]

### Se GAPS na Fase 1

Passar ao clarification-request com apenas o que falta:

STATUS: gaps-fase1
PROCESSO: [nome]
GAPS:
  - [item específico sem resposta]
DADOS_JA_COLETADOS: [resumo do que já existe — não pedir novamente]

---

## Etapa 4 — Coleta de Fase 2 (após aprovação da v1)

Quando o cliente aprovar a v1, iniciar coleta dos itens avançados.
Usar a base de conhecimento abaixo para identificar quais itens de Fase 2
são relevantes para o tipo de processo.

Checklist Fase 2:

| Item | Status |
|---|---|
| Prazos legais de análise por etapa | |
| Regras de notificação ao cidadão | |
| Tabelas e datasets (atividades, taxas, parâmetros) | |
| Integrações com sistemas externos | |
| Validações e cálculos automáticos | |
| Legislação (quando aplicável ao tipo de processo) | |
| Regras condicionais complexas | |

---

# BASE DE CONHECIMENTO INTERNA

Uso exclusivo da skill. Consultar para identificar:
1. O tipo de processo pelos dados recebidos
2. Quais itens de Fase 2 são relevantes para esse tipo

Nunca exposta ao cliente.

---

## Mapa de Identificação — Processo → Categoria

| O processo trata de... | Seção | Exemplos de nomes |
|---|---|---|
| Alvará de funcionamento, MEI, autônomo | Seção 1 | Alvará de Funcionamento, Licença de Localização |
| Consulta prévia, zoneamento, uso do solo | Seção 2 / 2A | Consulta Prévia, Informações Urbanísticas |
| Aprovação de projeto arquitetônico | Seção 2B | Aprovação de Projeto, Licença de Obra |
| Alvará de construção | Seção 2C | Alvará de Construção, Licença para Construir |
| Habite-se / conclusão de obra | Seção 2D | Habite-se, CVCO, Certificado de Conclusão |
| Demolição | Seção 2E/2F | Licença de Demolição, Certidão de Demolição |
| Tapume | Seção 2G | Autorização de Tapume |
| Publicidade externa | Seção 2H | Licença de Publicidade, Alvará de Anúncio |
| EIV / impacto de vizinhança | Seção 2J | EIV, RIV |
| Cancelamento de alvará | Seção 2K | Cancelamento de Alvará |
| Denúncias urbanísticas | Seção 2L | Denúncia, Fiscalização de Obra |
| Auto de infração urbanístico | Seção 2M | Auto de Infração, Notificação |
| Protocolo geral | Seção 2N | Protocolo Geral |
| IPTU, ITBI, ISSQN, CND, parcelamento | Seção 3 | Isenção IPTU, ITBI, CND |
| Vigilância sanitária | Seção 4 | Alvará Sanitário, Licença Sanitária |
| Licença ambiental (LAP, LAI, LAO) | Seção 5 | LAP, LAI, LAO |
| Dispensa ambiental | Seção 5D | Dispensa, Declaração de Dispensa |
| Condicionantes ambientais | Seção 5E | Apresentação de Condicionantes |
| Autorizações ambientais | Seção 5F | Autorização APP, Poda, Aterro |
| Licença ambiental alternativa | Seção 5G | Licença Simplificada, LAC |
| Serviços ambientais / denúncias | Seção 5I | Castração, Doação de Mudas, Denúncia Ambiental |
| Serviços urbanos | Seção 6 | Tapa-buraco, Iluminação Pública |
| Comunicação oficial interna | Seção 7 | Ofício, Memorando, Circular |
| RH de servidores | Seção 8 | Férias, Licença, Horas Extras |
| Compras e licitações | Seção 9 | Pregão, Dispensa, Nota Fiscal |
| Educação | Seção 10 | Matrícula, Transporte Escolar |
| Ouvidoria / e-SIC | Seção 11 | Ouvidoria, e-SIC |
| Processos disciplinares | Seção 12 | PAD, Sindicância |
| Trânsito e transporte | Seção 13 | Cartão PcD, Certidão de Táxi |
| Assistência social | Seção 14 | CIPTEA, Benefício Eventual, Aluguel Social |
| Cultura, esporte, eventos | Seção 15 | Alvará de Evento, Bolsa Atleta |
| Defesa civil / emergências | Seção 16 | Denúncia de Risco, Emergência |
| ITBI, CND, alteração cadastral | Seção 17 | ITBI, CND, Mudança de Titularidade |

---

## Itens de Fase 2 por tipo de processo

Use esta seção para saber o que coletar na Fase 2 após a v1 aprovada.

### Obras (Seções 2B, 2C, 2D)
- Integração com cadastro imobiliário (validar inscrição)
- Integração com GIS/zoneamento
- SISOBRA: orientar cliente sobre 3 meses de envio manual
- Quadro de áreas: verificar formato padrão Aprova
- Para Regularização: decisão sobre Alvará com força de Habite-se
- Prazo de validade do alvará e regras de prorrogação

### Licenciamento Econômico (Seção 1)
- Tabela de atividades/CNAE (dataset)
- Cruzamento CNAE × zoneamento
- Regras de renovação automática

### Licenciamento Ambiental (Seção 5)
- Tabela de atividades (insumo crítico — solicitar antes da Fase 2)
- Critérios de enquadramento por porte e potencial poluidor
- Condicionantes: pré-definidas ou caso a caso
- Prazos de validade por tipo de licença
- Decisão: dispensa automática ou manual

### Tributário (Seção 3, 17)
- Tabela de taxas e alíquotas
- Regras de parcelamento
- Integração com sistema de arrecadação

### Vigilância Sanitária (Seção 4)
- Tipos de estabelecimento e documentos específicos por tipo
- Regras de renovação

### Assistência Social (Seção 14)
- Critérios de elegibilidade detalhados
- Regras de fila de espera
- Periodicidade de renovação

### Educação (Seção 10)
- Regras de prioridade de vagas
- Controle de vagas por unidade

### Qualquer processo
- Prazos legais de resposta ao cidadão
- Regras de notificação por etapa
- Integrações com sistemas externos específicos
- Validações condicionais não coletadas na Fase 1

---

## Seção 0 — Perguntas Universais de Fase 2

Aplicar a qualquer processo quando na Fase 2:

- Há prazo legal de resposta ao cidadão? Quanto tempo?
- O cidadão é notificado em alguma etapa? Por qual canal?
- O processo pode ser devolvido para o cidadão complementar documentação?
- Em quais situações o pedido pode ser negado?
- Há recurso após negação?

---

## Seções 1 a 17 — Perguntas de Fase 2 por Categoria

Estas perguntas só são feitas na Fase 2, após v1 aprovada.

### Seção 1 — Alvarás e Licenciamento Econômico
- O município usa CNAE? Há tabela própria de atividades? (solicitar)
- Consulta de viabilidade é obrigatória? Há cruzamento CNAE × zoneamento?
- Exige vistoria? Em quais casos é dispensada?
- O alvará vence anualmente? Há renovação automática?

### Seção 2 — Obras
- Há integração com cadastro imobiliário para validar inscrição?
- Há integração com GIS ou sistema de zoneamento?
- SISOBRA: cliente foi orientado sobre os 3 meses? Quadro de áreas no formato padrão?
- Para Regularização: alvará com força de Habite-se ou processos separados?
- Prazo de validade do alvará? É prorrogável?

### Seção 3 — Tributário
- Quais critérios de isenção? São definitivos ou renovados anualmente?
- Tabela de taxas, alíquotas ou parâmetros de cálculo? (solicitar)
- Parcelamento: número máximo de parcelas? Há juros/correção?
- CND: automática ou manual? Integração com arrecadação?

### Seção 4 — Vigilância Sanitária
- Documentos específicos por tipo de estabelecimento?
- Prazo de análise. Regras de renovação.

### Seção 5 — Meio Ambiente
- Tabela de atividades com porte e potencial poluidor (solicitar — crítico)
- Enquadramento: feito pelo requerente ou pelo analista?
- Condicionantes: pré-definidas por tipo? (solicitar lista com prazo e comprovação)
- Prazos de validade por tipo de licença
- Dispensa: processo automático ou com análise?

### Seção 6 — Serviços Urbanos
- Regras de priorização de atendimento
- A equipe de campo registra execução no sistema?

### Seção 7 — Comunicação Oficial
- Numeração por tipo e por secretaria
- Registro de ciência pelo destinatário

### Seção 8 — RH
- Estatuto do Servidor (solicitar — contém regras de prazo e direitos)
- Regras específicas por tipo de licença
- Aprovação hierárquica em múltiplos níveis?

### Seção 9 — Compras
- Limites de valor por modalidade
- Integração com sistema financeiro/contábil

### Seção 10 — Educação
- Critérios de prioridade de vagas
- Controle de vagas por unidade e por rota de transporte

### Seção 11 — Ouvidoria
- Prazo de 20 dias corridos para e-SIC: o sistema controla?
- SLA por tipo de manifestação na ouvidoria

### Seção 12 — Processos Disciplinares
- Estatuto do Servidor: prazos legais (solicitar)
- Níveis de penalidade e regras de recurso

### Seção 13 — Trânsito
- Critérios de elegibilidade por público
- Periodicidade de renovação dos documentos

### Seção 14 — Assistência Social
- Critérios de elegibilidade detalhados
- Regras de fila de espera e prioridade
- Periodicidade de renovação

### Seção 15 — Cultura e Esporte
- Regras de seleção e edital (se houver)
- Prestação de contas: documentos e periodicidade

### Seção 16 — Defesa Civil
- Critérios de triagem por urgência
- Regras de interdição e prazo para regularização

### Seção 17 — ITBI e CND
- Base de cálculo do ITBI e alíquotas
- Hipóteses de isenção
- Integração com sistema de arrecadação para CND automática
