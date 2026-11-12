# Skill: gerenciador-leads

Skill do OpenBot que instala e opera o dashboard local **Gerenciador de Leads B2B**
(LeadFlow): CRUD de leads, campanhas, cidades e nichos em SQLite, em http://localhost:8642.

Código do dashboard: https://github.com/andrealencarz/openbot-gerenciador-leads

---

## 1. Instalar a skill (primeira vez)

Pré-requisitos: OpenBot aberto no computador onde o dashboard vai rodar e Python 3
(`python3 --version`). O dashboard é local: só abre nesse computador.

1. Abra o OpenBot e comece um chat novo.
2. Peça a instalação:

   ```
   Instale a skill "gerenciador-leads" deste endereço:
   https://github.com/andrealencarz/openbot-skill-gerenciador-leads
   Baixe o SKILL.md, salve como skill local chamada gerenciador-leads e prepare o dashboard na minha máquina.
   ```

3. O agente baixa `index.html`, `server.py` e `ensure_server.py` para `~/gerenciador-leads`
   e sobe o servidor. Aceite quando ele pedir permissão para rodar comandos ou abrir o endereço.
4. Confira:
   - http://localhost:8642/ abre com as cidades e os nichos iniciais;
   - o rodapé do menu diz **SQLite (fonte principal)**. Se disser "Modo local", o servidor
     não está de pé: peça "suba o servidor do gerenciador de leads".
5. Feche o chat. A skill fica instalada no seu OpenBot e vale para os próximos chats.

**Atualizar depois:** peça "atualize a skill gerenciador-leads e o dashboard pelo GitHub".
O banco com os seus leads não é tocado; o navegador pode mostrar a versão antiga até
você recarregar com Cmd+Shift+R.

---

## 2. Usar no dia a dia com o OpenBot

Você não precisa abrir o terminal. Fale em linguagem natural e o agente usa a skill:

| Você pede | O agente faz |
|---|---|
| "Abra o gerenciador de leads" | sobe o servidor se estiver parado e abre http://localhost:8642 |
| "Reinicie o dashboard" | derruba o processo da porta 8642 e sobe de novo; o banco fica igual |
| "Como estão meus leads?" | lê o banco e resume por status e nicho |
| "Prospecte clínicas de estética em Curitiba e salve no dashboard" | busca os leads (Glasser, Google Maps ou outro serviço) e importa sem duplicar |
| "Enriqueça o telefone dos leads de São Paulo" | consulta o serviço de dados e completa o que faltar |
| "Mude os leads contatados para 'proposta enviada'" | atualiza o status no banco |
| "Exporte os leads de IA em São Paulo" | gera JSON/CSV com os filtros pedidos |

A skill já impõe ao agente: fazer backup do `gerenciador.db` antes de importar em lote,
não apagar o banco, não abrir o servidor para a rede e mostrar o custo de buscas pagas
antes de rodar. Mesmo assim, em importação grande, peça "faça backup do banco antes".

---

## 3. Criar campanhas e agendar tarefas

### 3.1 Criar uma campanha no dashboard

Campanha = um nicho + as cidades-alvo + status. Só as campanhas com status **ativa**
alimentam a rotina automática de busca; as demais ficam guardadas sem gastar consulta.

Pelo painel: aba **Campanhas** → **+ Nova campanha** → nome, nicho, estado (filtra a lista
de cidades), bairro opcional (ex.: "Moema") com as cidades-alvo marcadas, e salve.
O nome é obrigatório.

Por pedido ao agente:

```
Crie no gerenciador de leads uma campanha "Agências de IA — São Paulo",
nicho "Agências de IA / Digital", cidades São Paulo e Campinas, status ativa.
```

### 3.2 Agendar uma tarefa recorrente

Rotina no OpenBot = um pedido salvo que roda sozinho no horário marcado e entrega o
resultado no chat. Roda no fuso do computador, e só com o OpenBot e o computador ligados;
a ocorrência que cair com o computador desligado não é recuperada.

Pedido de exemplo, para rodar de segunda a sexta às 08:00:

```
Crie uma rotina chamada "Busca diária de leads" para rodar de segunda a sexta às 08:00.
Ela deve: subir o gerenciador de leads se estiver parado, pegar as campanhas com status
"ativa", buscar empresas do nicho nas cidades-alvo que ainda não estão no dashboard,
importar sem duplicar e me resumir quantos novos leads entraram por campanha.
Se não houver campanha ativa ou nenhum lead novo, não faça nada e avise em uma linha.
```

Também dá para pedir: todo dia às HH:MM, dias úteis, semanal (com o dia), mensal (com o
dia do mês), de X em X horas ou minutos (mínimo 3 minutos) e uma vez só em data marcada.

Depois: **"liste minhas rotinas"**, **"pause a rotina X"**, **"mude a rotina X para 09:00"**,
**"apague a rotina X"**, **"teste a rotina X agora"**.

Escreva no pedido o que fazer quando não houver nada a fazer, e um limite de gasto por
execução quando a busca for paga. Se a rotina alimenta um dashboard feito por outro agente,
ela precisa ser criada para o agente que tem a skill e o acesso à pasta do dashboard.

### 3.3 Fluxo recomendado

1. Instalar a skill (seção 1) e abrir o dashboard.
2. Cadastrar nichos e cidades de interesse, ou pedir ao agente.
3. Criar uma ou duas campanhas **ativas**, uma por nicho e região.
4. Agendar a busca diária ou semanal usando essas campanhas.
5. Usar o agente para enriquecer contatos e preparar abordagens; o lead avança no funil
   (novo → contato feito → proposta enviada → reunião marcada → fechado/perdido).

---

## 4. Conteúdo

- `SKILL.md` — instruções: instalação, atualização, importação de leads, diagnóstico e regras de segurança.

## 5. Problemas comuns

| Sintoma | O que fazer |
|---|---|
| O OpenBot não achou a skill | em outro chat peça "use a skill gerenciador-leads"; se não achar, reinstale |
| Página "Modo local (sem SQLite)" | peça "suba o servidor do gerenciador de leads" |
| Dashboard não abre | peça "reinicie o dashboard"; se a porta 8642 estiver ocupada, o agente verifica qual processo é |
| Lead importado não aparece | recarregue a aba (Cmd+Shift+R) e edite em uma aba só |
| A rotina rodou e não buscou nada | confira se há campanha com status "ativa" e cidades marcadas |
| A rotina não rodou no horário | OpenBot ou computador estava desligado ou dormindo |
| Funciona só no OpenBot | em outro assistente use o prompt simples do README do app |
