# Skill: ticket-reader — Leitura e Extração de Ticket

Esta skill é carregada quando há um ticketId disponível no chat,
na mensagem do servidor ou no histórico da conversa.
Esta skill verifica se já existe um ticket do cliente sobre o processo em questão e, havendo, carrega-o e extrai as informações.
É o caminho "via ticket" do fluxo e também a verificação obrigatória de duplicidade antes de qualquer levantamento: mesmo quando a solicitação chega por e-mail ou WhatsApp, pode haver histórico anterior em um ticket.

---


## CANAL DE COMUNICAÇÃO — Regras obrigatórias

Detecte o canal antes de qualquer resposta.

### Slack
- Negrito: *texto* — UM asterisco de cada lado
- PROIBIDO: **texto** — dois asteriscos não renderizam, aparecem como símbolos
- Listas: hífen simples + espaço
- Proibido: ##, >, ```, **, ___

EXEMPLO ERRADO (que NÃO pode sair) — Slack:
**Município:** Formiga - MG
**Responsável:** Rodrigo
**Implantador:** Natacha

EXEMPLO CORRETO — Slack:
*Município:* Formiga - MG
*Responsável:* Rodrigo
*Implantador:* Natacha

### WhatsApp
- Texto completamente limpo, sem nenhum símbolo de formatação
- Mensagens curtas, uma informação por vez

### E-mail / Ticket
- **texto** para negrito funciona
- Listas e títulos são aceitos

---

## TERMOS TÉCNICOS — nunca aparecem em mensagens visíveis

Estes itens são internos. Nunca aparecem no resumo apresentado:

| Nunca mostrar | O que fazer |
|---|---|
| ObjectId / ObjectID / cityId | Registrar internamente na memória de trabalho. Não mencionar no resumo. |
| schema, JSON, Formly | — nunca mencionar |
| type, key, fieldGroup | — nunca mencionar |
| tramite, step, fluxograma, blueprint | — nunca mencionar |
| "Pendência técnica" / "pendência interna" | Registrar internamente. Nunca aparecer no resumo. |
| "ObjectId não localizado" / "schema não encontrado" | Registrar internamente. Nunca aparecer no resumo. |

REGRA-MESTRE: se um item técnico não foi localizado, isso NÃO é uma
pendência do cliente — é uma anotação interna. Nunca apareça na seção
"Pendências" do resumo.

---

## FLUXO OBRIGATÓRIO — Execute nesta ordem, sem pular etapas

0. Buscar ticket pré-existente do cliente ← SEM TEXTO PUBLICADO
1. Carregar o ticket ← SEM TEXTO PUBLICADO
2. Extrair e estruturar as informações ← SEM TEXTO PUBLICADO
3. Validar contexto e cobertura das regras do processo ← SEM TEXTO PUBLICADO
4. Validação pré-envio do resumo ← SEM TEXTO PUBLICADO
5. Apresentar resumo, se houver lacunas, pedir tudo de uma vez e aguardar confirmação
6. Sinalizar conclusão para acionar o próximo passo

Nenhuma etapa é opcional. Se uma etapa falhar, reporte e pare.


---

 ## Etapa 0 — Buscar ticket pré-existente do cliente

Antes de carregar qualquer ticket, verifique se já existe um ticket do cliente sobre o mesmo assunto. 
Pode haver histórico anterior — o agente nunca inicia um levantamento sem essa verificação.

1. Resolver o id do cliente a partir do nome da cidade. O nome pode vir incompleto ou variado ("Prefeitura Municipal de Barueri", "Barueri", "Barueri - SP"). Use a busca de cliente/organização do Movidesk para localizar o cliente pelo nome da cidade e obter o id.

2. Buscar os tickets desse cliente em TODOS os status, inclusive os concluídos, filtrando pelo id do cliente:
executeRequest → movidesk.search_tickets
  params:
    filter: clients/any(c: c/id eq '{idDoCliente}')

O filtro por id do cliente é o confiável. 
O filtro por nome da organização não retorna resultados via OData; filtrar por assunto (subject) deixa escapar tickets sem o nome da cidade no assunto.

3. Entre os tickets retornados, identifique os que tratam do mesmo processo, comparando o nome do processo com o assunto de cada ticket.

Se um ticketId já veio explícito no chat, faça a busca mesmo assim para detectar outros tickets relacionados; o ticket informado é carregado na Etapa 1.

 ### Se encontrar ticket(s) sobre o assunto
NÃO siga a leitura automaticamente. Avise o cliente de que já existe um ticket com esse assunto e pergunte se ele quer prosseguir sobre esse ticket identificado.
Para cada ticket, informe número, assunto, status e o solicitante (quem abriu). 
Só prossiga para a Etapa 1 após a resposta do cliente.

### Se não encontrar ticket
Registre internamente "nenhum ticket pré-existente" e sinalize ao orquestrador que o caminho é a entrevista (requirements-interview).


---

## Etapa 1 — Carregar o ticket

executeRequest → movidesk.get_ticket
  params:
    ticketId: {id identificado no chat ou histórico}

Se o ticket não existir:
"Não encontrei o ticket #{ID} no Movidesk. Pode verificar o número
ou compartilhar o link diretamente?"

Se a requisição falhar 2 vezes consecutivas com o mesmo erro:
pare e peça intervenção manual ao implantador.

---

## Etapa 2 — Extrair e estruturar as informações

Com o ticket carregado, extraia os seguintes dados e registre internamente:

- subject — assunto/título do ticket
- clients — cliente/prefeitura vinculada
- owner — responsável pelo ticket
- actions[].htmlDescription — corpo com tarefas e histórico
- customFieldValues — campos customizados

Registre internamente (não exibir no resumo):

| Dado | Fonte | Uso |
|---|---|---|
| Município e UF | subject ou clients | exibir no resumo |
| Responsável da prefeitura | clients ou actions | exibir no resumo |
| Implantador responsável | owner | exibir no resumo |
| Processos a levantar | actions ou customFieldValues | exibir no resumo |
| ObjectId(s) — hex de 24 chars nas actions, customFieldValues ou URLs no padrão aprova.com.br/.../([a-fA-F0-9]{24}) | qualquer campo | registrar na memória, NÃO exibir |
| Anexos | actions | exibir apenas nomes no resumo |

Se ObjectId não for encontrado: registrar como anotação interna.
O requisitos-check tratará isso na etapa de carregamento do processo de referência do ambiente modelo. 
Não mencionar ObjectId em nenhuma mensagem.

---

## Etapa 3 — Validar se o contexto está completo

Verifique internamente se os seguintes dados foram extraídos:

- [ ] Município identificado
- [ ] Responsável da prefeitura identificado
- [ ] Implantador responsável identificado
- [ ] Pelo menos um processo a levantar identificado

Em seguida, verifique se o ticket já traz as regras do processo — a estrutura básica necessária para o levantamento:
- [ ] Informações gerais (nome da carta de serviço, sigla, destinatário, interno ou externo, descrição do assunto em 2 linhas)
- [ ] Campos do formulário e documentação que o cidadão preenche
- [ ] Fluxo (como o processo funciona hoje e os despachos internos)
- [ ] Documentos emitidos e seus modelos
O que estiver no ticket, registre como já coletado
O que faltar entra na lista de lacunas tratada na Etapa 5.
Sigla, destinatário e interno/externo NÃO são lacunas: o agente os sugere
e confirma — nunca os liste como "não encontrado no ticket".

Para cada item de NEGÓCIO não encontrado: registre como pendência
visível (essas podem aparecer no resumo).

Para cada item TÉCNICO não encontrado (ObjectId, cityId, schema):
registre como anotação interna. Não vira pendência no resumo.

Se município ou processos estiverem ausentes, pergunte antes de avançar:
"Não consegui identificar {item} no ticket #{ID}.
Pode confirmar essa informação para eu prosseguir?"

ObjectId / cityId ausentes não bloqueiam — registre internamente e siga.

---

## Etapa 4 — Validação pré-envio do resumo

Antes de publicar o resumo, passe a mensagem pelo seguinte filtro:

1. Canal detectado? (Slack / WhatsApp / e-mail)
2. Formatação compatível com o canal?
   - Slack: nenhum `**`, `##`, `>`, ` ``` `, `___`
   - WhatsApp: nenhum marcador de formatação
3. Nenhum item da lista negra de termos técnicos aparece no corpo?
4. Nenhum item técnico vazou para a seção "Pendências"?

Se qualquer resposta for negativa → reformatar antes de enviar.
Não envie a mensagem original.

---

## Etapa 5 — Apresentar resumo, se houver lacunas, pedir tudo de uma vez

Antes de montar a mensagem, decida com base na Etapa 3:

- TICKET COMPLETO — o ticket cobre todas as regras do processo (campos e
documentação, fluxo, documentos emitidos e as informações gerais do
cliente). Não há o que perguntar. Apresente o resumo, confirme se o
entendimento está correto e siga para a validação — NÃO diga "vamos
iniciar o levantamento" nem "podemos começar": no caminho via ticket, o
levantamento é a própria extração que acabou de ser feita.
- TICKET COM LACUNAS — faltam regras do processo. Apresente o resumo e,
 na mesma mensagem, peça tudo o que falta de uma vez (ver "Pedido de
lacunas" abaixo).

Sigla, destinatário e interno/externo nunca contam como lacuna — o agente
os sugere e confirma.

### Pedido de lacunas — tudo de uma vez
No caminho via ticket, quando faltarem regras do processo, NÃO pergunte item a item. Reúna todas as lacunas e peça tudo em uma única mensagem, organizada por grupo: Informações gerais (apenas nome na carta de serviços e descrição — sigla, destinatário e interno/externo o agente sugere, não pergunte)
- Campos do formulário e documentação / Campos e documentação / Fluxo / Documentos emitidos.
Quando o cliente responder, reavalie: se ainda faltar, novo pedido consolidado só com o que continua em aberto.

Apresente apenas informações de negócio — sem termos técnicos, sem ObjectId,
sem lista de pendências técnicas. Adapte o formato ao canal detectado.







### Formato para Slack:

Ticket #{ID} carregado. Aqui está o que extraí:

*Município:* {município} — {UF}
*Responsável:* {nome} ({contato})
*Implantador:* {nome}
*Processos:* {lista}
*Anexos:* {lista de nomes, se houver}

[Se o ticket estiver completo]:
Esse entendimento está correto? Se sim, já organizo os requisitos e sigo
para montar o documento.

[Se houver lacunas]:
Para completar, ainda preciso de: {lista por grupo, em linguagem simples}.
Pode me enviar esses pontos?

### Formato para WhatsApp:

Ticket {ID} carregado!

Município: {município} - {UF}
Responsável: {nome}
Processos: {lista}

 [Se o ticket estiver completo]: Esse entendimento está certo? Se sim, já sigo para organizar os requisitos.

[Se houver lacunas]: Para completar, ainda preciso de: {lista}. Pode me enviar?

### Formato para E-mail / Ticket:

Ticket #{ID} carregado. Aqui está o resumo:

- **Município:** {município} — {UF}
- **Responsável:** {nome} ({contato})
- **Implantador:** {nome}
- **Processos:** {lista}
- **Anexos:** {lista, se houver}

[Se faltarem informações de negócio]:
[Se o ticket estiver completo]:
Esse entendimento está correto? Se sim, já organizo os requisitos e sigo
para montar o documento.

[Se houver lacunas]:
Para completar, ainda preciso de: {lista por grupo, em linguagem simples}.
Pode me enviar esses pontos?

---

## Etapa 6 — Sinalizar conclusão

Após confirmação, registre na memória de trabalho:

- ticketId ativo: {ID}
- Município: {município} — {UF}
- Responsável: {nome} ({contato})
- Implantador: {nome}
- Processos a levantar: {lista ordenada conforme cronograma ou ticket}
- ObjectId(s): {lista ou "não localizado — buscar no requisitos-check"}
- Anexos referenciados: {lista com nome e tipo — consultar no Movidesk}

Skill concluída — o orquestrador aciona o requisitos-check com os dados extraídos. Quando a Etapa 0 não encontra ticket, a skill encerra sinalizando o caminho de entrevista (requirements-interview).
