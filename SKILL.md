---
name: gerenciador-leads
description: Instala, atualiza e opera o dashboard local "Gerenciador de Leads B2B" (LeadFlow), um CRUD de leads, campanhas, cidades e nichos com SQLite, que roda em http://localhost:8642. Usa quando o usuário pedir para criar, instalar, atualizar, abrir, reiniciar ou usar o gerenciador de leads/ dashboard de prospecção, ou importar leads nele.
example-prompt: Instale a skill "gerenciador-leads" e prepare o dashboard na minha máquina.
---

# Gerenciador de Leads B2B (LeadFlow)

Dashboard local de prospecção: leads, campanhas, cidades/bairros e nichos, com busca,
filtros (cidade, UF, nicho, status), status no funil, links clicáveis por lead
(e-mail, site, LinkedIn, WhatsApp) e exportar/importar JSON. Dados em SQLite.

**Resultado esperado:** o dashboard aberto em http://localhost:8642 com a base carregada,
e o usuário sabendo como reabrir e onde ficam os dados.

## Código-fonte

Repositório do app: https://github.com/andrealencarz/openbot-gerenciador-leads
(branch `main`; arquivos `index.html`, `server.py`, `ensure_server.py`).

## Instalação (primeira vez)

1. Confirme que não há uma instalação: `curl -s -m 2 http://127.0.0.1:8642/api/ping`.
   Se responder `{"ok": ...}`, pule para **Rodar e verificar**.
2. Verifique Python 3: `python3 --version`. Se faltar, avise o usuário — o app não tem
   outra dependência.
3. Pegue os três arquivos em `~/gerenciador-leads` (a pasta do README do app):
   ```sh
   mkdir -p ~/gerenciador-leads && cd ~/gerenciador-leads
   for f in index.html server.py ensure_server.py; do
     curl -fsSL "https://raw.githubusercontent.com/andrealencarz/openbot-gerenciador-leads/main/$f" -o "$f" || exit 1
   done
   ```
   Se o usuário já tem a pasta com o app, use-a e só atualize os arquivos.
4. Rode `python3 ensure_server.py` dentro da pasta. Ele sobe o servidor em segundo plano
   (sessão própria, sobrevive ao fim do shell) e responde
   `{"status":"online|iniciado"}` quando o servidor responde.
5. **Rodar e verificar**: `curl -s http://127.0.0.1:8642/api/state` e confirme JSON com
   `cidades` (20 na primeira execução), `nichos` (5 ou mais), `leads`, `campanhas`.
   Abra http://localhost:8642/ no navegador do usuário e confira o rodapé do menu:
   deve dizer "SQLite (fonte principal)"; se disser "Modo local", o servidor não está de pé.
6. Diga ao usuário: o endereço, que o banco fica em `~/gerenciador-leads/gerenciador.db`
   (é o que guarda os dados, não limpe), e como reabrir depois.

## Atualizar uma instalação existente

Refaça o passo 3 (baixa os arquivos atuais) e reinicie o servidor: `lsof -ti tcp:8642 | xargs kill`
e em seguida `python3 ensure_server.py`. O banco não é tocado. Deixe claro que o navegador
pode servir a versão antiga: peça um recarregamento com Cmd+Shift+R.

## Importar leads para o dashboard

O servidor tem um endpoint que insere sem duplicar (chave: telefone ou nome+cidade):

```sh
curl -s -X POST http://127.0.0.1:8642/api/leads/import \
  -H 'Content-Type: application/json' \
  -d '{"leads":[{"nome":"Empresa","telefone":"","email":"contato@empresa.com",
       "cidade":"São Paulo","nicho":"Agências de IA / Digital","site":"https://empresa.com",
       "status":"novo","obs":"origem"}]}'
```

Resposta: `{"novos":N,"duplicados":N,"invalidos":N,"cidades_atualizadas":[...],"leads":[...]}`.
Campos extras aceitos pelo banco: `apresentacao` (URL do site, quando `site` guarda o LinkedIn)
e `linkedin` (URL do perfil). Faça backup do `gerenciador.db` antes de importar em lote.

## Como o app funciona (para diagnosticar)

- `server.py`: HTTP em `127.0.0.1:8642`; rotas `/api/state` (GET/POST), `/api/ping`,
  `/api/leads/import`. O POST de `/api/state` **substitui** o conjunto de registros enviado.
- `index.html`: tudo embutido. Carrega de `/api/state` ao abrir e ao voltar para a aba;
  o `localStorage` é só cópia de apoio. Filtros e menus vêm dos dados do banco.
- Os selects são menus próprios (`.cs`): lista compacta presa ao botão, sem fechar ao rolar.

## Regras de segurança

- Não apague nem sobrescreva `gerenciador.db` sem pedir: é a base de prospecção do usuário.
- Não exponha o servidor em outra interface de rede sem que o usuário peça e entenda o risco
  (hoje ele escuta só em `127.0.0.1`).
- Os dados contêm contatos comerciais; não os copie para fóruns, issues ou tabelas públicas.

## Problemas comuns

| Sintoma | Causa provável | O que fazer |
|---|---|---|
| Página diz "Modo local (sem SQLite)" | servidor fora do ar | `python3 ensure_server.py` |
| Mudança aparece só depois de Cmd+Shift+R | HTML em cache | normal; peça recarregar |
| Alteração feita no banco não aparece | aba antiga aberta | recarregue a aba; não edite em duas abas ao mesmo tempo |
| Porta 8642 ocupada por outro processo | outra instância | `lsof -nP -iTCP:8642 -sTCP:LISTEN` e decida com o usuário |
