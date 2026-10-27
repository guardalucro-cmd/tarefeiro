# Prompt: instale o Tarefeiro, seu app de tarefas e hábitos dentro do Claude

**Como usar:** no app Claude Desktop, abra **Artefatos → Painel → Chat**. Ali, conecte o **Google Sheets** (obrigatório) e o **Google Calendar** (opcional, para o celular avisar antes das tarefas com horário). Depois copie TODO o texto do bloco abaixo, cole nesse mesmo Chat e envie. Não precisa baixar nem anexar nada: o Claude busca o app pronto no GitHub, cria a sua planilha e publica o app no seu painel.

---

```
Quero que você instale para mim o Tarefeiro, um app de tarefas e hábitos que funciona dentro do Claude e guarda tudo numa planilha minha no Google Sheets. O código do app já está pronto no endereço https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.html : você NÃO escreve um app novo, só baixa e publica esse arquivo. Siga as etapas na ordem 1 → 2 → 3 → 4, sem pular nenhuma, em português do Brasil, sem me fazer perguntas (decida e me conte o que assumiu).

========================================
ETAPA 1: PLANILHA (GOOGLE SHEETS)
========================================
Faça sempre uma planilha NOVA, do zero, mesmo que exista outra parecida no meu Drive. Use exatamente as chamadas abaixo, na ordem: elas foram testadas e funcionam. Copie o JSON como está (as cores são números de 0 a 1, nunca "#hex"). Se uma chamada der erro, leia a mensagem, corrija só o que ela aponta e repita; não pare para me perguntar.

1.1 CRIAR. Crie uma planilha do Google chamada "Tarefeiro, banco de dados" (com a ferramenta do Google Sheets ou do Google Drive que cria arquivo, tipo application/vnd.google-apps.spreadsheet). Anote o ID (o trecho entre /d/ e /edit no link).

1.2 ABAS. Ferramenta update_spreadsheet do Google Sheets, com este requests (fuso America/Sao_Paulo, ou o meu fuso se for outro):
[{"updateSpreadsheetProperties":{"properties":{"locale":"pt_BR","timeZone":"America/Sao_Paulo"},"fields":"locale,timeZone"}},
{"addSheet":{"properties":{"sheetId":1,"title":"Tarefas","index":0}}},
{"addSheet":{"properties":{"sheetId":2,"title":"Listas","index":1}}},
{"addSheet":{"properties":{"sheetId":3,"title":"Habitos","index":2}}},
{"addSheet":{"properties":{"sheetId":4,"title":"Checkins","index":3}}},
{"addSheet":{"properties":{"sheetId":5,"title":"Config","index":4}}},
{"addSheet":{"properties":{"sheetId":6,"title":"Sincronizacao","index":5}}},
{"addSheet":{"properties":{"sheetId":7,"title":"Resumo","index":6}}},
{"deleteSheet":{"sheetId":0}}]
(Se a aba padrão não tiver sheetId 0, leia os sheetIds com get_spreadsheet e apague a aba que sobrou pelo id certo.)

1.3 FORMATO. update_spreadsheet com os pedidos abaixo, todos numa chamada só. T(id, colIni, colFim) significa:
{"repeatCell":{"range":{"sheetId":id,"startRowIndex":1,"startColumnIndex":colIni,"endColumnIndex":colFim},"cell":{"userEnteredFormat":{"numberFormat":{"type":"TEXT"}}},"fields":"userEnteredFormat.numberFormat"}}
Pedidos: T(1,0,19), T(2,0,6), T(5,0,3), T(6,0,5), T(3,0,6), T(3,7,13), T(4,0,3), T(4,4,5). Isso deixa as datas e horas como texto, que é o que o app espera.
Mais, para cada sheetId de 1 a 6, o cabeçalho escuro e a linha 1 congelada:
{"repeatCell":{"range":{"sheetId":ID,"startRowIndex":0,"endRowIndex":1},"cell":{"userEnteredFormat":{"backgroundColorStyle":{"rgbColor":{"red":0.055,"green":0.055,"blue":0.071}},"textFormat":{"bold":true,"foregroundColorStyle":{"rgbColor":{"red":1,"green":1,"blue":1}}}}},"fields":"userEnteredFormat.backgroundColorStyle,userEnteredFormat.textFormat"}}
{"updateSheetProperties":{"properties":{"sheetId":ID,"gridProperties":{"frozenRowCount":1}},"fields":"gridProperties.frozenRowCount"}}
Se SÓ a parte de cor e congelamento continuar falhando depois de corrigir, siga sem ela (é só visual). O formato TEXT é obrigatório.

1.4 CABEÇALHOS E LINHAS INICIAIS. Ferramenta update_values, uma chamada por aba (nomes idênticos, sem acento):
Tarefas!A1:S1 = [["id","titulo","notas","data","hora","prioridade","lista_id","etiquetas","status","concluida_em","repeticao","tarefa_mae_id","origem","google_task_id","google_list_id","google_atualizado_em","atualizado_em","criado_em","evento_id"]]
Listas!A1:F4 = [["id","nome","cor","ordem","google_list_id","atualizado_em"],["L001","Pessoal","#5A5FE8","1","",""],["L002","Trabalho","#F4B13E","2","",""],["L003","Casa","#2FA36B","3","",""]]
Habitos!A1:M1 = [["id","nome","icone","cor","frequencia","dias","meta_dia","unidade","periodo","ativo","ordem","criado_em","termina_em"]]
Checkins!A1:E1 = [["id","habito_id","data","valor","criado_em"]]
Sincronizacao!A1:E1 = [["data_hora","direcao","quantidade","status","detalhe"]]
Config!A1:C12 = [["campo","valor","descricao"],["nome_app","Tarefeiro","Nome mostrado no topo do app"],["visao_padrao","hoje","Visão que abre primeiro: hoje, amanha, 3dias, 7dias, tudo"],["inicio_semana","seg","Primeiro dia da semana no calendário: seg ou dom"],["mostrar_atrasadas","sim","Mostrar tarefas atrasadas no topo de Hoje"],["horario_resumo","07:30","Hora do resumo da manhã"],["horario_lembrete","21:00","Hora do lembrete da noite"],["google_tarefas_ligado","nao","Reservado para a ligação com o Google Tarefas"],["google_ultima_sync","","Reservado para a ligação com o Google Tarefas"],["agenda_ligada","nao","Alertas no celular pelo Google Agenda (sim ou nao)"],["agenda_minutos","15","Quantos minutos antes o celular avisa"],["lista_padrao","L001","Lista usada quando a tarefa não diz qual"]]

1.5 RESUMO. Ferramenta update_formulas (ou update_values, se não houver), range Resumo!A1:C1, com estas três células (pt_BR: ";" separa argumentos e "\" separa colunas da matriz):
A1: =IFERROR(QUERY({ARRAYFORMULA(LEFT(Tarefas!J:J;10))\Tarefas!I:I};"select Col1, count(Col2) where Col2 = 'feita' group by Col1 order by Col1 desc label Col1 'dia', count(Col2) 'tarefas feitas'";1);"Sem tarefas concluídas ainda")
B1: vazio
C1: =IFERROR(QUERY(Checkins!A:E;"select C, count(B) where C <> '' group by C order by C desc label C 'dia', count(B) 'check-ins'";1);"Sem check-ins ainda")

1.6 TESTE. append_values em Tarefas!A1 com a linha ["TESTE1","Tarefa de teste","","<hoje AAAA-MM-DD>","09:00","3","L001","","feita","<hoje AAAA-MM-DD> 09:15:00","","","app","","","","<agora AAAA-MM-DD HH:MM:SS>","<agora AAAA-MM-DD HH:MM:SS>",""]. Leia Tarefas!D2:J2 e confira que data e hora voltaram como texto, iguais ao que você gravou. Leia Resumo!A1:B2 e confira que A2 é o dia de hoje e B2 é 1. Depois apague a linha de teste inteira: update_spreadsheet com [{"deleteDimension":{"range":{"sheetId":1,"dimension":"ROWS","startIndex":1,"endIndex":2}}}]. Se o Resumo não calcular, corrija as fórmulas e repita.

========================================
ETAPA 2: O APP (ARTIFACT)
========================================
1. Baixe o arquivo com a sua ferramenta de execução de código (terminal), byte a byte, sem resumir nem reescrever:
   curl -fsSL https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.html -o tarefeiro-app.html
   curl -fsSL https://raw.githubusercontent.com/guardalucro-cmd/tarefeiro/main/tarefeiro-app.sha256 -o tarefeiro-app.sha256
   Confira com sha256sum que o arquivo bate com o código de tarefeiro-app.sha256 e que ele contém o texto COLE_AQUI_O_ID_DA_PLANILHA. NUNCA use uma ferramenta de leitura de página que resume o conteúdo: o app precisa chegar inteiro. Se o terminal não conseguir baixar (sem internet ou endereço bloqueado), pare e me peça para anexar o arquivo tarefeiro-app.html, que eu baixo em github.com/guardalucro-cmd/tarefeiro.
2. Troque o texto COLE_AQUI_O_ID_DA_PLANILHA pelo ID real da planilha da Etapa 1. Essa é a ÚNICA mudança permitida no arquivo: não reescreva, não resuma, não "melhore" nenhuma outra linha.
3. Publique o arquivo como artifact (página no claude.ai), com ícone "checklist" e com estas capabilities:
   - mcp: servidor "Google Sheets" com as ferramentas get_spreadsheet, get_values, append_values e update_values, e servidor "Google Calendar" com as ferramentas create_event, update_event e delete_event. Declare o Calendar mesmo que essas três ferramentas não apareçam para você nesta conversa: quem as usa é o app, com a minha permissão, quando eu ligar os alertas. Só publique sem o Calendar se a publicação for recusada por causa dele, e aí me avise;
   - sample (o app usa para separar várias tarefas de um texto só e para o resumo da semana).
4. Me passe o link do app.

========================================
ETAPA 3: LEMBRETES NO CELULAR
========================================
Se você tiver a ferramenta de tarefas agendadas nesta conversa, crie duas tarefas agendadas, com notificação no celular e aprovação automática (os disparos não podem parar esperando aprovação). As duas só LEEM a planilha, nunca escrevem nela. Escreva o prompt de cada uma completo e autônomo, com o ID real da planilha.

Se NÃO tiver essa ferramenta, pule esta etapa sem travar e me diga no fim que os lembretes da manhã e da noite podem ser criados depois numa conversa do Cowork com o texto abaixo. O app continua completo sem eles.

1. "Tarefeiro: resumo da manhã", todo dia às 07:30 no meu fuso.
   Prompt: descobrir a data de hoje no meu fuso; ler Tarefas!A:S, Habitos!A:M e Checkins!A:E da planilha <ID>; contar as tarefas com status "aberta" e sem tarefa_mae_id cuja data é hoje, as atrasadas (status "aberta" e data menor que hoje) e os hábitos ativos (ativo diferente de "nao" e termina_em vazio ou maior ou igual a hoje) que valem para hoje (frequencia "diario"; ou "dias" com o dia da semana de hoje na coluna dias, usando seg, ter, qua, qui, sex, sab, dom; hábitos "semana" não entram). Responder SÓ uma frase, que vira a notificação. Exemplo: "Tarefeiro: hoje são 5 tarefas (2 atrasadas) e 3 hábitos. A primeira é pagar o DAS às 10h." Se não houver nada: "Tarefeiro: dia livre hoje."
2. "Tarefeiro: lembrete da noite", todo dia às 21:00 no meu fuso.
   Prompt: mesma leitura; listar os hábitos que valem para hoje e ainda não bateram a meta (soma da coluna valor dos Checkins de hoje para o hábito menor que meta_dia) e as tarefas de hoje ainda abertas. Responder SÓ uma frase. Exemplo: "Tarefeiro: faltam 2 hábitos (leitura e água) e 1 tarefa de hoje." Se estiver tudo feito: "Tarefeiro: tudo feito hoje."

========================================
ETAPA 4: TESTE DE PONTA A PONTA (OBRIGATÓRIO)
========================================
1. Se criou os lembretes, rode uma vez, agora, o "Tarefeiro: resumo da manhã" e confira que a resposta é uma frase correta para a planilha vazia ("dia livre").
2. Acrescente na aba Tarefas uma tarefa de teste para hoje com status "aberta", leia a aba de volta e confira que ela aparece (se criou os lembretes, rode o resumo de novo: ele precisa citar 1 tarefa). Depois apague essa linha de teste removendo a linha inteira.
3. Confira que o artifact publicado contém o ID real da planilha (e não o texto COLE_AQUI_O_ID_DA_PLANILHA) e que as capabilities estão declaradas.

========================================
REGRAS GERAIS
========================================
- No fim, confirme item por item: [1] planilha criada, testada e limpa; [2] app publicado com o ID real e as capabilities; [3] dois lembretes criados com aprovação automática, ou o aviso de que ficaram para o Cowork; [4] teste da Etapa 4 aprovado. Se algum item estiver pendente, continue trabalhando em vez de encerrar. Nunca termine a resposta perguntando se pode continuar: continue até o fim.
- Depois me diga em poucas linhas como começar: abrir o link do app (no computador ou no app do Claude no celular), tocar em Permitir quando o Claude pedir acesso ao Google Sheets (escolher "Sempre permitir") e anotar a primeira tarefa no botão +. Para o celular avisar antes das tarefas com horário: aba Mais → Alertas no celular → Ligado, e permitir o Google Calendar.
- Tudo em português do Brasil.
```

---

## O que você vai ter no final

- **Um app** no seu Claude, que abre também no celular pelo app do Claude: tarefas com as visões Hoje, Amanhã, 3 dias, 7 dias e Tudo, anotação rápida em texto corrido ("pagar DAS sexta 10h"), várias tarefas de uma vez, tarefas que se repetem, subtarefas, hábitos com sequência de dias, calendário e resumo da semana.
- **Uma planilha no seu Google Sheets** como banco de dados: os dados são seus e você abre tudo no Sheets ou no Excel.
- **Dois lembretes** no celular: o resumo da manhã e o que falta à noite.

## Requisitos

1. Plano pago do Claude, usando o app Claude Desktop.
2. Conector **Google Sheets** conectado pelo Claude Desktop → Artefatos → Painel → Chat. Para os alertas no celular, conecte também o **Google Calendar**.
3. Na primeira vez que abrir o app, toque em **Permitir** quando o Claude pedir acesso ao Google Sheets (escolha "Sempre permitir").
