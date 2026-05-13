# Skill: ticket-reader — Leitura e Extração de Ticket

Esta skill é carregada quando há um ticketId disponível no chat,
na mensagem do servidor ou no histórico da conversa.

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

1. Carregar o ticket ← SEM TEXTO PUBLICADO
2. Extrair e estruturar as informações ← SEM TEXTO PUBLICADO
3. Validar se o contexto está completo ← SEM TEXTO PUBLICADO
4. Validação pré-envio do resumo ← SEM TEXTO PUBLICADO
5. Apresentar resumo e aguardar confirmação
6. Sinalizar conclusão para acionar o próximo passo

Nenhuma etapa é opcional. Se uma etapa falhar, reporte e pare.

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
O requisitos-check tratará isso na etapa de carregamento do processo
da cidade modelo. Não mencionar ObjectId em nenhuma mensagem.

---

## Etapa 3 — Validar se o contexto está completo

Verifique internamente se os seguintes dados foram extraídos:

- [ ] Município identificado
- [ ] Responsável da prefeitura identificado
- [ ] Implantador responsável identificado
- [ ] Pelo menos um processo a levantar identificado

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

## Etapa 5 — Apresentar resumo e aguardar confirmação

Apresente apenas informações de negócio — sem termos técnicos, sem ObjectId,
sem lista de pendências técnicas. Adapte o formato ao canal detectado.

### Formato para Slack:

Ticket #{ID} carregado. Aqui está o que extraí:

*Município:* {município} — {UF}
*Responsável:* {nome} ({contato})
*Implantador:* {nome}
*Processos:* {lista}
*Anexos:* {lista de nomes, se houver}

[Se houver informações de negócio faltando no ticket]:
Não encontrei no ticket: {lista em linguagem simples, ex: "contato do responsável"}.
Podemos prosseguir assim ou prefere complementar?

Podemos iniciar o levantamento?

### Formato para WhatsApp:

Ticket {ID} carregado!

Município: {município} - {UF}
Responsável: {nome}
Processos: {lista}

[Se faltarem informações de negócio]: Não encontrei {item} no ticket. Podemos seguir assim?

Podemos começar?

### Formato para E-mail / Ticket:

Ticket #{ID} carregado. Aqui está o resumo:

- **Município:** {município} — {UF}
- **Responsável:** {nome} ({contato})
- **Implantador:** {nome}
- **Processos:** {lista}
- **Anexos:** {lista, se houver}

[Se faltarem informações de negócio]:
Informações não encontradas no ticket: {lista simples}.
Podemos prosseguir ou prefere complementar?

Podemos iniciar o levantamento?

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

Skill concluída — orquestrador aciona o requisitos-check ou o
requirements-interview conforme o caso.
