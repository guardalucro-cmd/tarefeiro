# Tarefeiro

App de tarefas e hábitos que funciona dentro do Claude e guarda tudo numa planilha sua no Google Sheets. Mostra as tarefas de hoje, acompanha os hábitos e avisa no celular pelo Google Agenda antes do horário.

## Como instalar

1. No app Claude Desktop, conecte o **Google Sheets** (obrigatório) e o **Google Calendar** (opcional, para os alertas no celular) em Artefatos → Painel → Chat.
2. Copie o bloco de texto de [prompt-painel.md](prompt-painel.md), cole nesse mesmo Chat do painel de Artefatos e envie.

O Claude cria a planilha, baixa o app deste repositório e publica no seu painel de Artefatos. Para instalar pelo Cowork, com os lembretes da manhã e da noite, use [prompt-tarefeiro.md](prompt-tarefeiro.md).

## Arquivos

- `tarefeiro-app.html`: o app pronto. Na instalação, o Claude troca `COLE_AQUI_O_ID_DA_PLANILHA` pelo ID da sua planilha.
- `tarefeiro-app.sha256`: código para o Claude conferir que o arquivo chegou inteiro.
- `prompt-painel.md`: o prompt de instalação para o Chat do painel de Artefatos.
- `prompt-tarefeiro.md`: o prompt de instalação para o Cowork.

Feito por José Morais · Guarda Lucro. Mais ferramentas no [Cofre de Ferramentas](https://guardalucro.com.br/cofre/?sck=gh_tarefeiro).
