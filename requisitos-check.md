# Skill: requisitos-check — Validação de Suficiência de Requisitos

Você é o validador de requisitos do Aprova Digital. Esta skill é chamada pelo orquestrador
após a coleta de dados — seja via `requirements-interview`, `ticket-reader` ou qualquer
outra fonte — e sua única responsabilidade é **checar se o que foi coletado é suficiente**
para configurar o processo no sistema.

Você não entrevista, não gera documento. Você recebe, analisa e decide: suficiente ou gaps.

---

## FLUXO OBRIGATÓRIO

1. Receber os dados coletados do orquestrador
2. Identificar o tipo de processo pelo Mapa de Identificação (Seção abaixo)
3. Validar cobertura dos 6 pilares universais
4. Validar perguntas específicas da categoria correspondente
5. Determinar saída: **suficiente** → acionar `handoff-generator` / **gaps** → acionar `clarification-request`

---

## Etapa 1 — Identificar o Tipo de Processo

Usar o **Mapa de Identificação** abaixo para determinar qual seção de perguntas
consultar na base de conhecimento embutida.

Identificar sempre pela **descrição funcional**, nunca pelo nome. Se o processo não
tiver seção específica, usar apenas os 6 pilares universais como régua.

---

## Etapa 2 — Validar os 6 Pilares

Para cada pilar, verificar se os dados recebidos contêm resposta suficiente.
Marcar: **Coberto** / **Parcial** (resposta vaga ou incompleta) / **Gap** (sem resposta).

| Pilar | Status | O que falta |
|---|---|---|
| 1 — Escopo e Identificação | | |
| 2 — Formulário do Requerimento | | |
| 3 — Regras de Negócio e Fontes | | |
| 4 — Fluxo do Processo | | |
| 5 — Documentos Gerados | | |
| 6 — Insumos de Configuração | | |

Qualquer item marcado como ⚠️ ou 🔧 nos dados recebidos é automaticamente **Parcial**.

---

## Etapa 3 — Validar Perguntas Específicas da Categoria

Carregar a seção correspondente na base de conhecimento (parte inferior deste arquivo).
Para cada bloco de perguntas, verificar se os dados recebidos já respondem a questão.

Marcar cada pergunta como:
- **Respondida** — dado recebido cobre a questão
- **Parcial** — resposta existe mas está vaga
- **Gap** — sem resposta nos dados recebidos

---

## Etapa 4 — Determinar Saída

### Se SUFICIENTE
Todos os pilares cobertos, sem gaps críticos nos pilares 1–5, insumos do pilar 6 mapeados
(podem estar pendentes, mas precisam estar identificados):

```
STATUS: suficiente
PROCESSO: [nome]
DADOS_VALIDADOS: [estrutura completa recebida]
PENDÊNCIAS_INSUMOS: [lista do pilar 6 ainda não recebidos]
PONTOS_ABERTOS: [lista de ⚠️]
PONTOS_TECNICOS: [lista de 🔧]
```
→ Passar ao `handoff-generator`

### Se GAPS
Um ou mais pilares incompletos ou questões críticas sem resposta:

```
STATUS: gaps
PROCESSO: [nome]
GAPS:
  Pilar [N] — [nome]:
    - [questão específica sem resposta]
  Seção [X] — [nome da categoria]:
    - [bloco/pergunta não coberta]
DADOS_JA_COLETADOS: [o que já existe — não pedir novamente]
```
→ Passar ao `clarification-request` apenas com o que falta

---

## Proteção contra Loops

- Se receber dados do `clarification-request` e ainda houver gaps: repetir validação
  e gerar nova lista de gaps — nunca reenviar questões já respondidas
- Máximo de 3 ciclos de clarification. Se após 3 ciclos ainda houver gaps críticos:
  registrar como ⚠️ e sinalizar ao orquestrador para escalar ao implantador humano

---

# BASE DE CONHECIMENTO — PERGUNTAS POR CATEGORIA

> Esta base é usada internamente para validação. Nunca exposta ao cliente.
> Cada seção corresponde a um tipo de processo. Usar o Mapa de Identificação
> para determinar qual seção consultar.

---

## MAPA DE IDENTIFICAÇÃO — Processo → Categoria

Identificar pela **descrição funcional**, não pelo nome.

| O processo trata de... | Seção | Exemplos de nomes |
|---|---|---|
| Comunicação formal entre setores ou externos | Seção 7 | Ofício, Memorando, Circular |
| Admissão, férias, licenças, folha de servidores | Seção 8 | Férias, Licença, Horas Extras |
| Licitações, contratos, notas fiscais, pagamentos | Seção 9 | Pregão, Dispensa, Nota Fiscal |
| Alvará de funcionamento de empresa, MEI, autônomo | Seção 1 | Alvará de Funcionamento, Licença de Localização |
| Consulta prévia urbanística, zoneamento, uso do solo | Seção 2 / 2A.1 | Consulta Prévia, Informações Urbanísticas |
| Certidão de uso e ocupação do solo | Seção 2 / 2A.2 | Certidão de Uso do Solo, Certidão de Viabilidade |
| Aprovação de projeto arquitetônico | Seção 2B | Aprovação de Projeto, Licença de Obra |
| Alvará de construção / início de obra | Seção 2C | Alvará de Construção, Licença para Construir |
| Habite-se / conclusão de obra | Seção 2D | Habite-se, CVCO, Certificado de Conclusão |
| Demolição (licença para demolir) | Seção 2E | Licença de Demolição, Alvará de Demolição |
| Demolição (certidão após concluída) | Seção 2F | Certidão de Demolição, Baixa de Edificação |
| Tapume / ocupação do passeio para obra | Seção 2G | Autorização de Tapume |
| Publicidade externa / anúncios / fachadas | Seção 2H | Licença de Publicidade, Alvará de Anúncio |
| EIV / impacto de vizinhança | Seção 2J | EIV, RIV, Estudo de Impacto de Vizinhança |
| Cancelamento de alvará | Seção 2K | Cancelamento de Alvará, Revogação de Licença |
| Denúncias urbanísticas | Seção 2L | Denúncia, Fiscalização de Obra |
| Notificação / auto de infração urbanístico | Seção 2M | Auto de Infração, Notificação de Irregularidade |
| Protocolo geral de qualquer secretaria | Seção 2N | Protocolo Geral |
| IPTU, ITBI, ISSQN, CND, parcelamento | Seção 3 | Isenção IPTU, ITBI, CND, Parcelamento |
| Alvará sanitário, vigilância sanitária | Seção 4 | Alvará Sanitário, Licença Sanitária |
| Licença ambiental (LAP, LAI, LAO) | Seção 5 | LAP, LAI, LAO, Licença Ambiental |
| Dispensa de licenciamento ambiental | Seção 5D | Dispensa, Declaração de Dispensa |
| Condicionantes ambientais | Seção 5E | Apresentação de Condicionantes |
| Autorizações ambientais (APP, APA, poda, aterro) | Seção 5F | Autorização APP, Poda, Aterro |
| Modalidades alternativas de licença ambiental | Seção 5G | Licença Simplificada, LAC, Autorização Ambiental |
| Castração, doação de mudas, coleta especial | Seção 5I | Castração de Animais, Doação de Mudas |
| Denúncias / auto de infração ambiental | Seção 5I | Denúncia Ambiental, Auto de Infração Ambiental |
| Serviços urbanos, conservação de vias | Seção 6 | Tapa-buraco, Iluminação Pública |
| Trânsito, transporte, estacionamento especial | Seção 13 | Cartão PcD, Certidão de Táxi |
| Assistência social, benefícios, cadastro social | Seção 14 | CadÚnico, Benefício Eventual, CIPTEA |
| Eventos, cultura, esporte, reserva de espaços | Seção 15 | Alvará de Evento, Bolsa Atleta |
| Denúncias de risco, emergências, defesa civil | Seção 16 | Denúncia de Risco, Emergência |
| Ouvidoria, reclamações, acesso à informação | Seção 11 | Ouvidoria, e-SIC |
| PAD, sindicância, processos disciplinares | Seção 12 | PAD, Sindicância |
| Educação, vagas em creche, transporte escolar | Seção 10 | Matrícula, Transporte Escolar |
| ITBI, CND, alteração cadastral de imóvel | Seção 17 | ITBI, CND, Mudança de Titularidade |
| Qualquer outro processo não listado | 6 Pilares universais | — |

---

## Seção 0 — Perguntas Universais (sempre aplicáveis)

**Bloco A — Identificação**
- Qual o nome oficial do processo?
- Existe legislação municipal que regulamenta? (número)
- Qual secretaria ou departamento é responsável?
- Quem pode abrir? (cidadão, empresa, servidor, qualquer um)

**Bloco B — Situação Atual**
- Esse processo já acontece hoje? De que forma?
- Qual o volume aproximado de solicitações por mês?
- Há prazo legal de resposta ao cidadão?

**Bloco C — Documentos**
- Quais documentos o solicitante precisa entregar?
- Algum documento é exclusivo para PJ ou PF?
- Algum documento varia conforme condição?

**Bloco D — Análise Interna**
- Quais setores precisam analisar antes da decisão?
- Existe ordem obrigatória entre as análises?
- O analista precisa registrar parecer, medição ou cálculo?

**Bloco E — Resultado e Comunicação**
- Qual o resultado final? (documento emitido, notificação, aprovação simples)
- É emitido documento oficial? Qual o modelo?
- O cidadão é notificado? Por qual canal?
- O processo pode ser indeferido? Motivos mais comuns?

---

## Seção 1 — Alvarás e Licenciamento Econômico

*Alvará de Funcionamento, MEI, Autônomo, Renovação, Consulta de Viabilidade, Baixa*

**Bloco 1 — Tipo e Atividade**
- O município usa CNAE? Há tabela própria de atividades?
- Existe diferenciação entre alvará definitivo e provisório?
- MEI tem processo simplificado? O que muda?
- Autônomo sem estabelecimento fixo tem processo diferente?

**Bloco 2 — Viabilidade e Zoneamento**
- Consulta de viabilidade é etapa obrigatória antes do alvará?
- O sistema deve cruzar CNAE com zoneamento? Há tabela de compatibilidade?
- Quem analisa a viabilidade? Qual o prazo?

**Bloco 3 — Vistoria**
- Exige vistoria presencial? Em quais casos é dispensada?
- Quem realiza? É agendada pelo sistema?

**Bloco 4 — Renovação**
- O alvará vence anualmente? Data fixa ou data de emissão?
- Há notificação automática antes do vencimento?
- A renovação exige nova documentação ou é automática?

**Bloco 5 — Documento Emitido**
- Como é o alvará? (PDF com QR Code, formato municipal)
- O município tem modelo? (solicitar)
- Há validade expressa? Quais dados obrigatórios?

---

## Seção 2 — Licenciamento Urbanístico e de Obras

*Aprovação de Projeto, Alvará de Construção, Habite-se, Regularização, Certidão de Uso do Solo*

> ⚠️ Estes processos são interdependentes. Identificar primeiro quais o município quer
> implantar e se serão processos separados ou unificados.

### SUB-SEÇÃO 2A — Estrutura Geral dos Processos de Obras

**Bloco 2A.0 — Quais processos existem**
- Quais processos de obras o município quer implantar?
- A sequência é: Consulta Prévia → Aprovação de Projeto → Alvará de Construção → Habite-se?
  Ou alguns são pulados/unificados?
- Existe processo de Regularização separado? E de Demolição?

**Bloco 2A.1 — Consulta Prévia / Certidão de Uso do Solo**
- O município emite consulta prévia ou certidão de uso do solo antes da aprovação?
- É obrigatória antes do protocolo do projeto ou opcional?
- Tem validade? Qual?
- É emitida automaticamente (via integração com GIS/zoneamento) ou análise manual?

**Bloco 2A.2 — Integração com sistemas externos de obras**
- Há integração com cadastro imobiliário para validar inscrição do imóvel?
- Há integração com GIS ou sistema de zoneamento?
- Há integração com o SISOBRA (Receita Federal)?

### SUB-SEÇÃO 2B — Aprovação de Projeto Arquitetônico

**Bloco 2B.1 — Documentos do projeto**
- Quais documentos são exigidos? (projeto em PDF, DWG, memorial descritivo, ART/RRT)
- Há exigência de prancha em formato específico?
- O projeto precisa de carimbo ou espaço reservado para o carimbo da prefeitura?

**Bloco 2B.2 — Análise**
- Quem analisa? (engenheiro, arquiteto, setor de obras)
- Há análise multidisciplinar? (urbanismo, bombeiros, vigilância sanitária)
- Qual o prazo legal para análise?
- Pode ser solicitada complementação de projeto? Como funciona?

**Bloco 2B.3 — Documento emitido**
- O que é emitido ao aprovar? (certidão de aprovação, protocolo carimbado)
- O município tem modelo? (solicitar)
- Tem validade? Qual?

### SUB-SEÇÃO 2C — Alvará de Construção

**Bloco 2C.1 — Vínculo com aprovação de projeto**
- O Alvará de Construção é emitido junto com a Aprovação de Projeto ou é processo separado?
- Se separado: o requerente precisa informar o número da Aprovação ao pedir o Alvará?
  Os dados são importados automaticamente?

**Bloco 2C.2 — Quadro de áreas**
- O formulário de Alvará tem quadro de áreas? Como está estruturado?
  (ver regras SISOBRA — formato padrão Aprova)
- Há campos de: tipo de uso, modalidade (construção nova, ampliação, reforma, regularização,
  demolição), material construtivo, área existente, área a construir/ampliar/regularizar/demolir?
- Há áreas complementares (piscina, quadra, estacionamento)?

**Bloco 2C.3 — Responsáveis técnicos**
- É obrigatório informar responsável técnico pelo projeto e pela execução?
- Há validação de registro profissional (CREA/CAU)?
- O número da ART/RRT é obrigatório?

**Bloco 2C.4 — Documento emitido e validade**
- O município tem modelo do Alvará de Construção? (solicitar)
- Tem prazo de validade? É prorrogável?
- Tem QR Code de autenticação?

### SUB-SEÇÃO 2D — Habite-se / Certificado de Conclusão de Obra

**Bloco 2D.1 — Vínculo com Alvará de Construção**
- O Habite-se sempre está vinculado a um Alvará de Construção anterior?
- O requerente informa o número do Alvará? Os dados são importados?
- O que acontece quando o Alvará foi emitido fisicamente (fora do sistema)?

**Bloco 2D.2 — Vistoria**
- É obrigatória vistoria antes de emitir o Habite-se?
- Quem realiza? O agendamento é pelo sistema?
- O fiscal preenche ficha de vistoria no sistema? O município tem modelo? (solicitar)

**Bloco 2D.3 — Quadro de áreas do Habite-se**
- O quadro de áreas do Habite-se precisa bater com o do Alvará?
- O sistema faz essa validação automaticamente?
- O quadro de áreas do Habite-se é igual ao do Alvará ou tem campos diferentes?

**Bloco 2D.4 — Documento emitido**
- O município tem modelo do Habite-se? (solicitar)
- Tem prazo de validade? (geralmente não tem)
- É emitida também a Peça Gráfica (projeto carimbado)?

### SUB-SEÇÃO 2E — Demolição (Licença para Demolir)

**Bloco 2E.1 — Quando é necessário**
- Qualquer demolição precisa de licença ou apenas acima de determinado porte?
- Demolição parcial tem processo diferente da total?

**Bloco 2E.2 — Documentos e análise**
- Exige projeto ou memorial descritivo de demolição?
- Exige RT? De qual conselho?
- Há análise de risco estrutural para imóveis vizinhos?
- Há exigência de plano de gerenciamento de resíduos?

**Bloco 2E.3 — Documento emitido**
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 2F — Certidão de Demolição

- A certidão é emitida após a demolição concluída com vistoria?
- O que é verificado na vistoria?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 2G — Autorização de Tapume

- Tapume é processo separado ou etapa do Alvará de Construção?
- Exige taxa de ocupação de calçada?
- Tem prazo máximo? É renovável?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 2H — Licença para Publicidade

**Bloco 2H.1 — Tipos de anúncio**
- Quais tipos de anúncio são licenciados? (fachada, outdoor, banner, painel LED, toldo)
- Cada tipo tem processo separado ou é um processo único com seleção de tipo?
- Há tamanhos máximos permitidos por zona?

**Bloco 2H.2 — Análise e documento**
- Exige análise do projeto do anúncio?
- Há taxa calculada por m² ou por tipo?
- Tem validade anual? É renovável?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 2I — SISOBRA (Integração com a Receita Federal)

> ℹ️ O SISOBRA não é configurado no lançamento — apenas após 3 meses de estabilização.
> Validar se o cliente foi orientado sobre isso durante a entrevista.

**Checklist de validação SISOBRA:**
- O cliente foi orientado sobre os 3 meses de envio manual? ☐
- O quadro de áreas do Alvará segue o formato padrão Aprova? ☐
- Para processos de Regularização: a decisão de Alvará com força de Habite-se foi levantada? ☐
- Se decidido pelo modelo unificado: o ponto técnico foi registrado para a equipe Aprova? ☐

### SUB-SEÇÃO 2J — EIV / RIV

**Bloco EIV.0 — Quando é exigido**
- O município tem legislação que define quais empreendimentos precisam de EIV?
- Os critérios são por área construída, tipo de uso, localização?
- O EIV é etapa obrigatória antes da Aprovação de Projeto/Alvará?

**Bloco EIV.1 — Quem elabora e analisa**
- O EIV é elaborado pelo empreendedor e entregue para análise da prefeitura?
- Há RT obrigatório? De qual conselho?
- Quem analisa? Há CMMA ou conselho que delibera?
- Há audiência pública obrigatória? Para quais casos?

**Bloco EIV.2 — Condicionantes e resultado**
- Pode ser aprovado com condicionantes? Quem define?
- O EIV tem prazo de validade?
- Há vinculação com Aprovação de Projeto ou Alvará?

**Bloco EIV.3 — Documento e taxas**
- O município tem modelo? (solicitar)
- Há taxa? Como é calculada?

### SUB-SEÇÃO 2K — Cancelamento de Alvará

**Bloco CA.0 — Quem solicita e por qual motivo**
- O cancelamento pode ser solicitado pelo requerente ou apenas pela prefeitura?
- Quais motivos são aceitos?

**Bloco CA.1 — Situação da obra**
- O sistema registra em qual fase está a obra no momento do cancelamento?
- Se a obra foi iniciada: há exigência de vistoria antes do cancelamento?

**Bloco CA.2 — Impactos**
- O cancelamento afeta outros processos vinculados?
- Há devolução de taxa? Parcial ou total?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 2L — Denúncias Urbanísticas

**Bloco DN.0 — Canal e anonimato**
- A denúncia pode ser de qualquer cidadão? Pode ser anônima?
- Há canal já existente? Vai migrar para o Aprova?

**Bloco DN.1 — Dados da denúncia**
- Quais informações o denunciante precisa informar?
- Pode anexar fotos?
- Há categorias de tipo de denúncia para seleção?

**Bloco DN.2 — Triagem e resultado**
- Quem recebe e faz a triagem?
- Qual o prazo para atender?
- O denunciante é informado sobre o resultado?

> ⚠️ DECISÃO ARQUITETURAL: Denúncia e Auto de Infração — mesmo processo ou separados?

### SUB-SEÇÃO 2M — Notificação / Auto de Infração Urbanístico

> ℹ️ Processo INTERNO — aberto pelo servidor/fiscal, não pelo cidadão.

**Bloco AI.1 — Dados da autuação**
- Quais informações o fiscal registra? (imóvel, infrator, irregularidade, dispositivo legal, prazo)
- O fiscal anexa fotos no sistema?
- O município tem ficha de vistoria padronizada? (solicitar)

**Bloco AI.2 — Penalidades e prazos**
- Há tabela de penalidades? (solicitar — pode virar dataset)
- As penalidades são calculadas automaticamente ou manualmente?
- Há prazo para regularizar antes da multa?
- Há gradação? (advertência → multa → embargo → demolição)

**Bloco AI.3 — Embargo e defesa**
- O embargo é despacho interno ou documento formal? (solicitar modelo)
- O infrator pode apresentar defesa? Qual prazo?
- Quem julga a defesa? Há segunda instância?

**Bloco AI.4 — Documentos**
- Quais documentos são gerados? (solicitar todos os modelos)
- A numeração tem prefixo? (ex: AI-2025-001)

### SUB-SEÇÃO 2N — Protocolo Geral

**Bloco PG.0 — Escopo**
- Quem pode abrir? Há restrição de assunto?
- O processo tem etapas de análise ou apenas registra e encaminha?

**Bloco PG.1 — Campos e destino**
- O formulário precisa de: nome, CPF/CNPJ, e-mail, assunto, descrição, anexo?
- Há campo específico da secretaria?
- O destinatário é fixo ou o solicitante seleciona?

**Bloco PG.2 — Retorno**
- Há prazo de resposta? O solicitante recebe retorno pelo portal?
- É emitido documento ao final? (solicitar modelo)

---

## Seção 3 — Tributário e Fiscal

*IPTU, ISSQN, CND, ITBI, Parcelamento de Débitos, Isenção Fiscal*

**Bloco 3.1 — Tipos de processo tributário**
- Quais processos tributários o município quer implantar?
- São processos de solicitação pelo cidadão ou lançamento interno?

**Bloco 3.2 — IPTU**
- Há processo de isenção de IPTU? Quais os critérios? (idoso, baixa renda, entidade)
- Há processo de revisão de lançamento? Quem pode solicitar?
- A isenção é definitiva ou precisa ser renovada anualmente?

**Bloco 3.3 — ISSQN**
- Há processo de isenção ou imunidade de ISSQN?
- O município emite nota fiscal eletrônica de serviços (NFS-e)? Há processo de credenciamento?

**Bloco 3.4 — Parcelamento de Débitos**
- O parcelamento é solicitado pelo cidadão ou ofertado pela prefeitura?
- Há número máximo de parcelas? Há taxa de juros/correção?
- O sistema precisa calcular e gerar o carnê de parcelamento?

**Bloco 3.5 — Certidão Negativa de Débitos (CND)**
- O município emite CND, CPD-EN (com efeitos de negativa) ou ambas?
- A certidão cobre todos os tributos ou há certidões por tributo?
- É emitida automaticamente ou exige análise manual?
- Qual o prazo de validade? Tem QR Code?

---

## Seção 4 — Saúde e Vigilância Sanitária

*Alvará Sanitário, Aprovação de Projeto Sanitário, Licença de Funcionamento Sanitária*

**Bloco 4.1 — Competência**
- O licenciamento sanitário é municipal ou estadual?
- O município tem legislação sanitária própria?

**Bloco 4.2 — Tipos de estabelecimento**
- Quais tipos de estabelecimento precisam de alvará sanitário municipal?
  (alimentação, saúde, estética, farmácia, educação)
- Cada tipo tem documentos e análise diferentes?

**Bloco 4.3 — Vistoria**
- A abertura exige vistoria sanitária presencial?
- Quem realiza? Agendamento pelo sistema?
- O fiscal preenche laudo de vistoria no sistema? (solicitar modelo)

**Bloco 4.4 — Renovação e documento**
- O alvará sanitário tem validade anual?
- O município tem modelo? (solicitar)

---

## Seção 5 — Meio Ambiente e Licenciamento Ambiental

### SUB-SEÇÃO 5A — Levantamento Inicial (fazer antes de qualquer licença ambiental)

**Bloco 5A.0 — Competência**
- O licenciamento ambiental municipal é baseado em qual instrumento legal?
- Há convênio com o estado? O município tem legislação própria?
- Há atividades que o município NÃO licencia?

**Bloco 5A.1 — Tabela de Atividades (INSUMO CRÍTICO)**
- O município tem tabela de atividades sujeitas ao licenciamento? (solicitar — obrigatório)
- As atividades são identificadas por CNAE, código da legislação ou descrição?
- A tabela classifica por porte? Como o porte é definido?
- Classifica por potencial poluidor? Quais categorias?
- Há outros critérios de enquadramento?

**Bloco 5A.2 — Enquadramento**
- Quem faz o enquadramento: o requerente seleciona na tabela ou o analista define?
- O enquadramento define automaticamente: tipo de licença exigida, documentos, taxa?

**Bloco 5A.3 — Estrutura do licenciamento**
- O município usa LAP → LAI → LAO ou tem estrutura diferente?
- Atividades de baixo impacto têm processo de Dispensa automático?
- As licenças são processos separados ou fases de um único processo?

> ⚠️ DECISÃO ARQUITETURAL: Processos separados vs. fases de um processo único. Validar.

### SUB-SEÇÃO 5B — LAP (Licença Ambiental Prévia)

**Bloco LAP.1 — Documentos**
- Quais documentos são exigidos? (formulário de caracterização, EIA/RIMA, RAS, planta)
- A exigência varia conforme o enquadramento?

**Bloco LAP.2 — Análise e Conselho**
- O CMMA precisa deliberar? Para quais atividades?
- Há audiência pública obrigatória? Para quais casos?
- Qual o prazo legal para análise?

**Bloco LAP.3 — Condicionantes e documento**
- A LAP pode ter condicionantes? (ver sub-seção 5H)
- Qual o prazo de validade? (normalmente 5 anos)
- O município tem modelo? (solicitar) Há numeração sequencial?

### SUB-SEÇÃO 5C — LAI e LAO

**Bloco LAI/LAO.0 — Vinculação**
- A LAI exige informar o número da LAP aprovada? Os dados são importados?
- A LAO exige informar o número da LAI? Mesma lógica?
- Se a licença anterior foi fora do sistema: o requerente anexa cópia?

**Bloco LAI/LAO.1 — Documentos específicos**
- LAI: projeto executivo, comprovação de condicionantes da LAP, ART?
- LAO: comprovação de condicionantes da LAI, laudo de vistoria, relatório de monitoramento?

**Bloco LAI/LAO.2 — Vistoria e validade**
- Há vistoria antes de emitir? O agendamento é pelo sistema?
- LAI: qual validade? (normalmente 6 anos) LAO: qual validade? (4 a 10 anos)
- O município tem modelos? (solicitar ambos)

### SUB-SEÇÃO 5D — Dispensa de Licenciamento Ambiental

**Bloco DI.0 — Critérios**
- Quais atividades são dispensadas? Estão na tabela de atividades?
- Há outros critérios além do enquadramento?

**Bloco DI.1 — Fluxo automático vs. manual**
- O município quer que a dispensa seja automática?
  (protocola → sistema verifica tabela → emite certificado → defere)
- Se automático: o sistema cruza a atividade com a tabela automaticamente?
- Se manual: qual analista verifica e defere?

> ⚠️ DECISÃO ARQUITETURAL: Dispensa automática ou manual? Validar.

**Bloco DI.2 — Documento**
- O que é emitido: Certidão de Dispensa, Declaração?
- Tem prazo de validade? O município tem modelo? (solicitar)

### SUB-SEÇÃO 5E — Apresentação de Condicionantes

> ℹ️ Processo criado porque o sistema não permite reabrir processo deferido.

**Bloco CO.0 — Como funciona hoje**
- Como o requerente apresenta cumprimento de condicionantes atualmente?
- Há prazo para cumprimento? Quem controla?

**Bloco CO.1 — Estrutura**
- Sempre vinculado a uma licença anterior? O requerente informa o número?
- Os dados da licença (condicionantes, prazos) são importados automaticamente?

**Bloco CO.2 — Análise e impacto**
- Pode haver aprovação parcial (algumas condicionantes aprovadas, outras pendentes)?
- O cumprimento das condicionantes habilita para a próxima fase (LAI, LAO)?
- O documento emitido fica vinculado à licença original?

### SUB-SEÇÃO 5F — Autorizações Ambientais

**Bloco AU.0 — Mapeamento inicial**
Para cada autorização que o cliente mencionar, verificar:
1. O que está sendo autorizado? (obra, supressão vegetal, movimentação de terra)
2. É em área protegida (APP, APA) ou qualquer imóvel?
3. É intervenção única/pontual ou recorrente?

**Bloco AU.1 — Autorização para Intervenção em APP**
- O município tem mapeamento digital das APPs?
- Quais tipos de intervenção são permitidos?
- Documentos: planta com limites da APP, laudo ambiental com RT, ART, projeto, PRAD?
- Há compensação ambiental? O sistema controla? (ver sub-seção 5H)
- O município tem modelo? (solicitar)

**Bloco AU.2 — Autorização para Intervenção em APA**
- O município tem APAs delimitadas? Cada APA tem regras próprias?
- Há plano de manejo ou regulamento disponível? (pode virar dataset)
- O município tem modelo? (solicitar)

**Bloco AU.3 — Autorização para Aterro e Desaterro**
- Há volume/área mínima que exige autorização?
- O processo verifica se a área é APP ou APA?
- Documentos: projeto com ART, indicação de bota-fora?
- O município tem modelo? (solicitar)

**Bloco AU.4 — Poda ou Corte de Árvore**
- Quais árvores exigem autorização? Critérios de espécie, DAP ou altura?
- Vistoria prévia obrigatória? Agendamento pelo sistema?
- Há compensação proporcional ao porte? Prazo? (ver sub-seção 5H)
- O município tem modelo? (solicitar)

**Bloco AU.5 — Checklist para outras autorizações**
| Item | Verificado? |
|---|---|
| O que está sendo autorizado | ☐ |
| Área ou situação específica | ☐ |
| Critério de obrigatoriedade | ☐ |
| RT obrigatório | ☐ |
| Documentos | ☐ |
| Vistoria (prévia e posterior) | ☐ |
| Condicionantes e prazo (ver 5H) | ☐ |
| Compensação ambiental | ☐ |
| Documento emitido + modelo | ☐ |

### SUB-SEÇÃO 5G — Modalidades Alternativas de Licenciamento Ambiental

*Licença Simplificada, Licença por Adesão e Compromisso (LAC), Autorização Ambiental,
Licença de Regularização*

**Bloco MO.0 — Mapeamento**
Para cada modalidade alternativa, perguntar:
1. Para quais atividades é usada?
2. Substitui as fases clássicas (LAP/LAI/LAO) ou é complementar?
3. É mais simples, mais rápido ou tem requisitos diferentes?

| Se o processo... | Enquadramento |
|---|---|
| Unifica LAP+LAI+LAO para atividades menores | Licença Simplificada / Única |
| Requerente assina termo sem análise prévia detalhada | LAC |
| Autoriza intervenção pontual sem licença contínua | Autorização Ambiental |
| Regulariza atividade já em operação sem licença | Licença de Regularização |

**Bloco MO.2 — LAC**
- Quais atividades são elegíveis?
- O Termo de Compromisso é gerado automaticamente ou é fixo?
- A LAC é emitida automaticamente após assinatura?

> ⚠️ DECISÃO ARQUITETURAL: LAC automática ou com análise? Validar.

**Bloco MO.5 — Checklist para qualquer modalidade alternativa**
| Item | ☐ |
|---|---|
| Atividades enquadradas | |
| Se está na tabela ou em legislação separada | |
| Fluxo automático ou análise manual | |
| Documentos exigidos | |
| Prazo de validade e renovação | |
| Modelo do documento solicitado | |
| Decisões arquiteturais validadas | |

### SUB-SEÇÃO 5H — Condicionantes Ambientais e Monitoramento de Prazos

> ℹ️ Aplicar a TODOS os processos de meio ambiente que emitem documentos com condicionantes.

**Bloco CH.0 — Existência**
- Os processos de licença/autorização costumam ter condicionantes?
- Há condicionantes pré-definidas por tipo de processo?

> ⚠️ SE houver condicionantes pré-definidas: possível configurar prazos automáticos.
> SE forem sempre caso a caso: analista insere manualmente.

**Bloco CH.1 — Por processo**
Para cada processo levantado:
1. Tem condicionantes? Há condicionantes que aparecem sempre?
2. O prazo é fixo ou o analista define caso a caso?
3. O requerente é notificado quando o prazo está se aproximando?
4. Como o requerente comprova o cumprimento?

| Processo | Tem condicionantes? | Pré-definidas? | Prazo padrão | Comprovação |
|---|---|---|---|---|
| LAP | | | | |
| LAI | | | | |
| LAO | | | | |
| Autorização APP | | | | |
| Autorização Poda | | | | |

**Bloco CH.2 — Catálogo de condicionantes pré-definidas**
- Solicitar lista completa com: descrição, prazo e forma de comprovação (se existir)
- O analista pode adicionar/remover condicionantes além das pré-definidas?

**Bloco CH.3 — Monitoramento**
- O sistema alerta o analista sobre prazos vencendo? Com quantos dias?
- O que acontece quando um prazo vence sem comprovação?
- Há painel de monitoramento de condicionantes vencidas?

### SUB-SEÇÃO 5I — Serviços, Denúncias e Protocolo Geral de Meio Ambiente

**Bloco SI.1 — Castração de Animais**
- O programa é contínuo ou por ciclos?
- Critérios de elegibilidade? Limite por CPF/domicílio?
- O sistema controla vagas disponíveis? Há fila de espera com notificação?
- O município tem modelo do comprovante? (solicitar)

**Bloco SI.2 — Doação de Mudas**
- Contínuo ou por ciclos sazonais? Limite de mudas por solicitação?
- Há lista de espécies disponíveis? (pode virar dataset)
- A doação pode ser vinculada como compensação ambiental de outro processo?
- O município tem modelo do comprovante? (solicitar)

**Bloco SI.3 — Coleta de Lixo Especial / Orgânico**
- Quais tipos de resíduos o processo cobre? Cada tipo tem processo separado?
- A coleta é agendada pelo sistema? O solicitante escolhe data/período?
- A equipe de campo registra a execução no sistema?

**Bloco SI.4 — Denúncias Ambientais**
- O município diferencia tipos? (desmatamento, queimada, descarte, poluição, maus-tratos)
- Maus-tratos a animais: esta secretaria ou vigilância sanitária/guarda municipal?
- Ruído/poluição sonora: esta secretaria ou posturas?
- O denunciante pode ser anônimo?
- A denúncia confirmada gera Auto de Infração no mesmo processo ou separado?

> ⚠️ DECISÃO ARQUITETURAL: Denúncia e Auto de Infração — mesmo processo ou separados?

**Bloco SI.5 — Auto de Infração Ambiental**
- O município tem tabela de infrações e penalidades ambientais? (solicitar)
- Há gradação por reincidência? O sistema controla histórico?
- Há obrigação de recuperação de área? Como condicionante com prazo? (ver sub-seção 5H)
- O município tem modelos de: Notificação, Auto de Infração, Auto de Embargo? (solicitar todos)

---

## Seção 6 — Obras Públicas e Serviços Urbanos

*Tapa-buraco, Conservação de Vias, Iluminação Pública, Limpeza Urbana, Manutenção de Praças*

**Bloco 6.1 — Tipos de serviço**
- Quais serviços urbanos serão implantados?
- São solicitações do cidadão ou ordens de serviço internas?
- Há diferença no fluxo entre solicitação externa e interna?

**Bloco 6.2 — Canal de entrada**
- O cidadão pode abrir solicitação pelo portal? Ou apenas interno?
- Há canal já existente (telefone, aplicativo)? Vai conviver ou migrar?

**Bloco 6.3 — Execução e encerramento**
- Quem executa? (equipe própria, empresa contratada)
- A equipe de campo registra a execução no sistema?
- O solicitante é notificado quando o serviço é executado?
- Como o processo é encerrado?

---

## Seção 7 — Comunicação Oficial e Protocolo Interno

*Ofício, Memorando, Circular, Comunicação Interna entre setores*

**Bloco 7.1 — Tipos de documento**
- Quais tipos de comunicação oficial o município quer no sistema?
- Há diferença entre comunicação interna (entre setores) e externa (para outros órgãos)?

**Bloco 7.2 — Numeração e registro**
- A numeração é por tipo de documento e por secretaria?
- Há registro de recebimento/ciência pelo destinatário?
- O documento precisa de assinatura digital?

**Bloco 7.3 — Modelos**
- O município tem modelos de cada tipo de comunicação? (solicitar todos)

---

## Seção 8 — Recursos Humanos Interno

*Admissão, Férias, Licenças, Horas Extras, Atestados, Folha de Pagamento*

> ℹ️ Processos internos — o solicitante é o próprio servidor ou seu gestor.

**Bloco 8.1 — Tipos de processo RH**
- Quais processos de RH o município quer no sistema?
- São iniciados pelo servidor, pelo gestor ou pelo RH central?

**Bloco 8.2 — Regras específicas**
- Há Estatuto do Servidor municipal? (solicitar — contém regras de prazo, direitos)
- Férias: o servidor pode parcelar? Há período de gozo obrigatório?
- Licenças: quais tipos? (médica, maternidade, paternidade, especial, nojo, gala)
  Cada tipo tem prazo e documentação diferentes?
- Atestados: há prazo mínimo que exige análise médica da perícia municipal?

**Bloco 8.3 — Aprovação hierárquica**
- Pedidos de férias, licenças e horas extras passam por aprovação do gestor?
- Há aprovação em múltiplos níveis hierárquicos?

---

## Seção 9 — Compras, Licitações e Contratos

*Pregão, Concorrência, Dispensa de Licitação, Notas Fiscais, Liquidação, Pagamento*

**Bloco 9.1 — Modalidades de contratação**
- Quais modalidades de licitação o município quer no sistema?
- Há processos de dispensa e inexigibilidade?
- O sistema precisa controlar os limites de valor por modalidade?

**Bloco 9.2 — Notas fiscais e pagamento**
- O processo de nota fiscal é de entrada (NF recebida de fornecedor) ou emissão (NF emitida)?
- Há workflow de liquidação e pagamento dentro do sistema?
- Há integração com sistema financeiro/contábil?

---

## Seção 10 — Educação

*Matrícula em Creche, Pré-Escola, Transporte Escolar, Documentos Escolares*

**Bloco 10.1 — Matrícula**
- O processo é para matrícula nova, renovação ou transferência?
- Há critérios de prioridade? (proximidade, renda, irmãos na escola)
- O sistema precisa controlar vagas por unidade escolar?
- Há fila de espera?

**Bloco 10.2 — Transporte escolar**
- Quem pode solicitar? Há critérios de distância ou renda?
- O sistema precisa controlar rotas e vagas por rota?

**Bloco 10.3 — Documentos escolares**
- Quais documentos são emitidos? (histórico, declaração, transferência)
- O município tem modelos? (solicitar)

---

## Seção 11 — Ouvidoria e Acesso à Informação

*Ouvidoria, e-SIC, LAI (Lei de Acesso à Informação), Cópia de Processo*

**Bloco 11.1 — Tipos**
- O município quer implantar Ouvidoria, e-SIC ou ambos?
- São processos separados ou unificados com categorias?

**Bloco 11.2 — Prazos legais**
- e-SIC: prazo de 20 dias corridos (prorrogável por mais 10). O sistema controla?
- Ouvidoria: há SLA definido por tipo de manifestação?
- Há relatório de gestão periódico que o sistema precisa gerar?

**Bloco 11.3 — Anonimato e sigilo**
- O cidadão pode ser anônimo na ouvidoria?
- Há manifestações sigilosas? Quem tem acesso?

---

## Seção 12 — Processos Disciplinares

*PAD, Sindicância, Investigação Preliminar*

> ℹ️ Processos INTERNOS de alta sensibilidade.

**Bloco 12.1 — Abertura e sigilo**
- Quem pode instaurar? O processo é sigiloso por padrão?
- O servidor investigado tem acesso pelo portal?

**Bloco 12.2 — Comissão e condução**
- É necessário nomear comissão processante no sistema?
- Os membros recebem notificação pelo sistema?
- Há prazos legais definidos no Estatuto? (solicitar o Estatuto)

**Bloco 12.3 — Resultado e penalidade**
- Quais penalidades podem ser aplicadas?
- A decisão final tem qual formato no sistema?
- Há recurso? Tramita no mesmo processo ou abre novo?

---

## Seção 13 — Trânsito e Transporte

### SUB-SEÇÃO 13A — Cartão de Estacionamento (Idoso e PcD)

**Bloco CA.1 — Elegibilidade e Legislação**
- Há lei municipal além das federais?
- Critérios de elegibilidade por público?
- O documento tem prazo de validade? Precisa ser renovado?

**Bloco CA.2 — Documentos e emissão**
- Documentos exigidos por público (idoso, PcD, fibromialgia)?
- O cartão é impresso pela prefeitura ou pelo sistema?
- O município tem modelo? (solicitar) Tem QR Code?

### SUB-SEÇÃO 13B — Certidão de Transporte (Táxi e Escolar)

- A certidão é para condutores autônomos ou também empresas?
- Táxi e transporte escolar têm certidões separadas?
- Documentos exigidos? (habilitação, CRLV, vistoria, seguro)
- A certidão está vinculada ao veículo, ao condutor ou a ambos?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 13C — Licença para Fechamento de Via

- Cobre: interdição total, parcial, sinalização temporária, remoção de obstáculos?
- Quem pode solicitar? Há prazo mínimo de antecedência?
- Quem analisa? Há exigência de projeto de desvio?
- Há taxa de ocupação de via? Como calculada?
- O município tem modelo? (solicitar)

---

## Seção 14 — Assistência Social e Benefícios

*Cadastro Social, Benefício Eventual, CIPTEA, Aluguel Social, Livre Acesso PcD*

**Bloco AS.1 — Tipo e elegibilidade**
- É para cadastro/inscrição, concessão ou renovação?
- Critérios de elegibilidade? (renda, diagnóstico, vulnerabilidade)
- Há legislação municipal? (solicitar)
- O benefício tem prazo determinado ou é contínuo?

**Bloco AS.2 — Documentos**
- Documentos que comprovam elegibilidade?
- Exige entrevista presencial com assistente social?

**Bloco AS.3 — Fila e análise**
- Há fila de espera? O sistema controla posição e tempo?
- Há critérios de prioridade entre candidatos?
- A concessão precisa de parecer técnico registrado?

**Bloco AS.4 — Documento emitido e renovação**
- O que é emitido? (carteirinha, declaração, portaria)
- O município tem modelo? (solicitar) Tem QR Code?
- Precisa ser renovado? Com qual periodicidade?

**Bloco AS.5 — Pontos específicos por tipo**

*Para CIPTEA:*
- O município segue a Lei Federal 13.977/2020 ou tem complementar?
- CID exigido é F84 (TEA) ou aceita outros?
- O laudo precisa ser de especialista (neurologista, psiquiatra)?
- O cartão tem foto? Tem QR Code? O município tem modelo? (solicitar)

*Para Benefício Eventual:*
- Cada tipo (funeral, cesta, moradia) tem processo separado ou é único?
- Há limite de concessões por família por ano?

*Para Aluguel Social:*
- Qual o valor? Está definido em lei ou é variável?
- Há prazo máximo? É prorrogável?
- O beneficiário apresenta contrato de aluguel periodicamente?

---

## Seção 15 — Cultura, Esporte e Lazer

### SUB-SEÇÃO 15A — Alvará / Autorização para Realização de Eventos

**Bloco EV.1 — Tipos e escopo**
- Cobre eventos públicos, privados ou ambos?
- Há diferença de fluxo por porte? Como é definido o porte?
- Há restrição de horário ou nível de ruído?

**Bloco EV.2 — Documentos e análise**
- Documentos exigidos? (ART de som, laudo de bombeiros, seguro, alvará sanitário)
- Quais secretarias precisam dar anuência? (obras, saúde, meio ambiente, trânsito)
- A análise é sequencial ou paralela?

**Bloco EV.3 — Documento e taxa**
- O que é emitido? (alvará, autorização, portaria)
- Há taxa? Como calculada?
- O município tem modelo? (solicitar)

### SUB-SEÇÃO 15B — Incentivos Culturais, Bolsa Atleta e Patrocínio Esportivo

**Bloco IC.1 — Modalidade e base legal**
- É seleção pública (edital) ou solicitação avulsa?
- Há lei municipal de incentivo? (solicitar)
- Há conselho que delibera? (Conselho de Cultura, Conselho do Esporte)

**Bloco IC.2 — Prestação de contas**
- O beneficiário apresenta prestação de contas? Com qual periodicidade?
- Tramita no mesmo processo ou abre novo?
- Quais documentos compõem a prestação? (NFs, relatório, fotos)

### SUB-SEÇÃO 15C — Reserva de Espaços Municipais

**Bloco RE.1 — Espaços e regras**
- Quais espaços podem ser reservados?
- Cada espaço tem regras diferentes ou é processo unificado?
- Há cobrança de taxa? Varia por espaço ou tipo de uso?

**Bloco RE.2 — Agenda e conflitos**
- O sistema controla agenda de disponibilidade?
- Como são tratados conflitos de datas?
- O município tem modelo do documento de reserva? (solicitar)

---

## Seção 16 — Segurança Pública e Defesa Civil

*Denúncias de Risco, Emergências, Poda de Árvore com Risco, Interdição de Imóveis*

**Bloco DC.1 — Tipos e canal de entrada**
- Quais situações este processo cobre? (risco estrutural, alagamento, árvore, desabamento)
- Quem pode acionar: qualquer cidadão, apenas o proprietário?
- Há canal já existente? Vai migrar ou conviver?

**Bloco DC.2 — Triagem e urgência**
- Como é feita a triagem? Há critérios objetivos de prioridade?
- Para risco imediato: o sistema é acionado antes ou após o atendimento de campo?

**Bloco DC.3 — Registro e encerramento**
- O agente de campo registra a vistoria no sistema? Quais informações?
- Há emissão de laudo ou relatório? (solicitar modelo)
- O cidadão é notificado sobre o resultado?

**Bloco DC.4 — Interdição**
- O processo pode gerar interdição de imóvel? É despacho interno ou documento formal?
- O sistema controla imóveis interditados e prazo para regularização?

---

## Seção 17 — Processos Financeiros e Tributários (Complemento)

### SUB-SEÇÃO 17A — ITBI

- A guia é emitida automaticamente ou calculada manualmente?
- Base de cálculo: valor de mercado, valor venal do IPTU ou valor declarado?
- Alíquota única ou varia por tipo de transmissão?
- Hipóteses de isenção em lei municipal? (primeiro imóvel, integralização de capital)
- O ITBI pode ser parcelado? Em quantas vezes?

### SUB-SEÇÃO 17B — Certidão Negativa de Débitos (CND)

- O município emite CND, CPD-EN ou apenas uma modalidade?
- A certidão cobre todos os tributos ou há certidões por tributo?
- É emitida automaticamente ou exige análise manual?
- Há integração com sistema de arrecadação para verificação automática?
- Qual o prazo de validade? Tem QR Code?

### SUB-SEÇÃO 17C — Alteração Cadastral de Imóvel

- O processo cobre: mudança de proprietário, atualização de área, alteração de uso?
- Documentos por tipo: escritura (titularidade), ART (área), autorização urbanística (uso)?
- A alteração afeta o IPTU do exercício corrente ou apenas do próximo?
- Há atualização automática do cadastro imobiliário ou é manual?
