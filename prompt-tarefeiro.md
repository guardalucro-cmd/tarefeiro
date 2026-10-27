# Prompt: instale o Tarefeiro, seu app de tarefas e hábitos dentro do Claude

**Como usar:** abra uma conversa nova no Cowork (app Claude Desktop), copie TODO o texto do bloco abaixo, cole na conversa e envie. Não precisa baixar nem anexar nada: o Claude busca o app sozinho. Antes disso, conecte os conectores **Google Sheets** e **Google Calendar** na sua conta (Claude Desktop → Artefatos → Painel → Chat). O Google Calendar é opcional: serve para o celular avisar antes das tarefas com horário. O Claude cria a planilha, publica o app e liga os lembretes para você.

---

```
Quero que você instale para mim o Tarefeiro, um app de tarefas e hábitos que funciona dentro do Claude e guarda tudo numa planilha minha no Google Sheets. O código do app já está pronto no endereço https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.html : você NÃO escreve um app novo, só baixa e publica esse arquivo. Siga as etapas na ordem 1 → 2 → 3 → 4, sem pular nenhuma, em português do Brasil, sem me fazer perguntas (decida e me conte o que assumiu).

========================================
ETAPA 1: PLANILHA (GOOGLE SHEETS)
========================================
Crie no meu Google Drive uma planilha chamada "Tarefeiro, banco de dados". Configure a planilha com fuso America/Sao_Paulo (ou o meu fuso, se for outro) e localidade pt_BR.

Crie exatamente estas abas, com estes cabeçalhos na linha 1 (linha 1 congelada, fundo escuro #0E0E12, texto branco em negrito). Os nomes das abas e das colunas precisam ser idênticos, sem acento onde não tem:

1. Tarefas (19 colunas): id | titulo | notas | data | hora | prioridade | lista_id | etiquetas | status | concluida_em | repeticao | tarefa_mae_id | origem | google_task_id | google_list_id | google_atualizado_em | atualizado_em | criado_em | evento_id
2. Listas: id | nome | cor | ordem | google_list_id | atualizado_em
   Linhas iniciais: L001 | Pessoal | #5A5FE8 | 1 ; L002 | Trabalho | #F4B13E | 2 ; L003 | Casa | #2FA36B | 3
3. Habitos (13 colunas): id | nome | icone | cor | frequencia | dias | meta_dia | unidade | periodo | ativo | ordem | criado_em | termina_em
4. Checkins: id | habito_id | data | valor | criado_em
5. Config: campo | valor | descricao, com estas linhas:
   nome_app | Tarefeiro | Nome mostrado no topo do app
   visao_padrao | hoje | Visão que abre primeiro: hoje, amanha, 3dias, 7dias, tudo
   inicio_semana | seg | Primeiro dia da semana no calendário: seg ou dom
   mostrar_atrasadas | sim | Mostrar tarefas atrasadas no topo de Hoje
   horario_resumo | 07:30 | Hora do resumo da manhã
   horario_lembrete | 21:00 | Hora do lembrete da noite
   google_tarefas_ligado | nao | Reservado para a ligação com o Google Tarefas
   google_ultima_sync | (vazio) | Reservado para a ligação com o Google Tarefas
   agenda_ligada | nao | Alertas no celular pelo Google Agenda (sim ou nao)
   agenda_ligada | nao | Reservado para o Google Agenda
   lista_padrao | L001 | Lista usada quando a tarefa não diz qual
6. Sincronizacao: data_hora | direcao | quantidade | status | detalhe
7. Resumo: sem cabeçalho. Na célula A1 e na célula C1, estas fórmulas (planilha em pt_BR: separador ";" e "\" entre colunas da matriz). Use colunas inteiras como abaixo, nunca intervalos que começam na linha 2:
   A1: =IFERROR(QUERY({ARRAYFORMULA(LEFT(Tarefas!J:J;10))\Tarefas!I:I};"select Col1, count(Col2) where Col2 = 'feita' group by Col1 order by Col1 desc label Col1 'dia', count(Col2) 'tarefas feitas'";1);"Sem tarefas concluídas ainda")
   C1: =IFERROR(QUERY(Checkins!A:E;"select C, count(B) where C <> '' group by C order by C desc label C 'dia', count(B) 'check-ins'";1);"Sem check-ins ainda")

Formate como TEXTO (numberFormat TEXT) todas as células da linha 2 para baixo nas abas Tarefas, Listas, Config e Sincronizacao, e nas colunas A a F e H a M da aba Habitos e A a C e E da aba Checkins. Isso impede o Sheets de transformar datas e horas.

Apague a aba vazia que vem por padrão, se ela sobrar.

TESTE da planilha: acrescente (append) na aba Tarefas uma linha [TESTE1, Tarefa de teste, , <hoje AAAA-MM-DD>, 09:00, 3, L001, , feita, <hoje AAAA-MM-DD> 09:15:00, , , app, , , , <agora>, <agora>, ]. Leia Tarefas de volta e confira que data e hora continuam texto. Leia Resumo!A1:B2 e confira que apareceu o dia de hoje com 1. Depois APAGUE a linha de teste removendo a linha inteira (deleteDimension), não só limpando o conteúdo. Se o Resumo não calcular, corrija as fórmulas e repita.

Anote o ID da planilha (o trecho entre /d/ e /edit no link).

========================================
ETAPA 2: O APP (ARTIFACT)
========================================
1. Baixe o arquivo com o terminal, byte a byte, sem resumir nem reescrever:
   curl -fsSL https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.html -o tarefeiro-app.html
   curl -fsSL https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.sha256 -o tarefeiro-app.sha256
   Confira com sha256sum que o arquivo bate com o código de tarefeiro-app.sha256 e que ele contém o texto COLE_AQUI_O_ID_DA_PLANILHA. NUNCA use uma ferramenta de leitura de página que resume o conteúdo: o app precisa chegar inteiro. Se o terminal não conseguir baixar (sem internet ou endereço bloqueado), pare e me peça para anexar o arquivo tarefeiro-app.html, que eu baixo em github.com/guardalucro-cmd/tarefeiro.
2. Troque o texto COLE_AQUI_O_ID_DA_PLANILHA pelo ID real da planilha da Etapa 1. Essa é a ÚNICA mudança permitida no arquivo: não reescreva, não resuma, não "melhore" nenhuma outra linha.
3. Publique o arquivo como artifact (página no claude.ai), com ícone "checklist" e com estas capabilities:
   - mcp: servidor "Google Sheets" com as ferramentas get_spreadsheet, get_values, append_values e update_values, e servidor "Google Calendar" com as ferramentas create_event, update_event e delete_event (se o Google Calendar não estiver conectado, publique só com o Google Sheets e me avise);
   - sample (o app usa para separar várias tarefas de um texto só e para o resumo da semana).
4. Me passe o link do app.

========================================
ETAPA 3: LEMBRETES NO CELULAR
========================================
Crie duas tarefas agendadas, com notificação no celular e aprovação automática (os disparos não podem parar esperando aprovação). As duas só LEEM a planilha, nunca escrevem nela. Escreva o prompt de cada uma completo e autônomo, com o ID real da planilha.

1. "Tarefeiro: resumo da manhã", todo dia às 07:30 no meu fuso.
   Prompt: descobrir a data de hoje no meu fuso; ler Tarefas!A:S, Habitos!A:M e Checkins!A:E da planilha <ID>; contar as tarefas com status "aberta" e sem tarefa_mae_id cuja data é hoje, as atrasadas (status "aberta" e data menor que hoje) e os hábitos ativos (ativo diferente de "nao" e termina_em vazio ou maior ou igual a hoje) que valem para hoje (frequencia "diario"; ou "dias" com o dia da semana de hoje na coluna dias, usando seg, ter, qua, qui, sex, sab, dom; hábitos "semana" não entram). Responder SÓ uma frase, que vira a notificação. Exemplo: "Tarefeiro: hoje são 5 tarefas (2 atrasadas) e 3 hábitos. A primeira é pagar o DAS às 10h." Se não houver nada: "Tarefeiro: dia livre hoje."
2. "Tarefeiro: lembrete da noite", todo dia às 21:00 no meu fuso.
   Prompt: mesma leitura; listar os hábitos que valem para hoje e ainda não bateram a meta (soma da coluna valor dos Checkins de hoje para o hábito menor que meta_dia) e as tarefas de hoje ainda abertas. Responder SÓ uma frase. Exemplo: "Tarefeiro: faltam 2 hábitos (leitura e água) e 1 tarefa de hoje." Se estiver tudo feito: "Tarefeiro: tudo feito hoje."

========================================
ETAPA 4: TESTE DE PONTA A PONTA (OBRIGATÓRIO)
========================================
1. Rode uma vez, agora, o "Tarefeiro: resumo da manhã" e confira que a resposta é uma frase correta para a planilha vazia ("dia livre").
2. Acrescente na aba Tarefas uma tarefa de teste para hoje com status "aberta" e rode o resumo de novo: ele precisa citar 1 tarefa. Depois apague essa linha de teste removendo a linha inteira.
3. Confira que o artifact publicado contém o ID real da planilha (e não o texto COLE_AQUI_O_ID_DA_PLANILHA) e que as capabilities estão declaradas.

========================================
REGRAS GERAIS
========================================
- No fim, confirme item por item: [1] planilha criada, testada e limpa; [2] app publicado com o ID real e as capabilities; [3] dois lembretes criados com aprovação automática; [4] teste da Etapa 4 aprovado. Se algum item estiver pendente, continue trabalhando em vez de encerrar.
- Depois me diga em poucas linhas como começar: abrir o link do app (no computador ou no app do Claude no celular), tocar em Permitir quando o Claude pedir acesso ao Google Sheets (escolher "Sempre permitir") e anotar a primeira tarefa no botão +. Para o celular avisar antes das tarefas com horário: aba Mais → Alertas no celular → Ligado, e permitir o Google Calendar.
- Tudo em português do Brasil.
```

---

## O que você vai ter no final

- **Um app** no seu Claude, que abre também no celular pelo app do Claude: tarefas com as visões Hoje, Amanhã, 3 dias, 7 dias e Tudo, anotação rápida em texto corrido ("pagar DAS sexta 10h"), várias tarefas de uma vez, tarefas que se repetem, subtarefas, hábitos com sequência de dias, calendário e resumo da semana.
- **Uma planilha no seu Google Sheets** como banco de dados: os dados são seus e você abre tudo no Sheets ou no Excel.
- **Dois lembretes** no celular: o resumo da manhã e o que falta à noite.

## Requisitos

1. Conta no Claude com Cowork e Artefatos.
2. Conector **Google Sheets** conectado pelo Claude Desktop → Artefatos → Painel → Chat. Para os alertas no celular, conecte também o **Google Calendar**.
3. Na primeira vez que abrir o app, toque em **Permitir** quando o Claude pedir acesso ao Google Sheets (escolha "Sempre permitir").
