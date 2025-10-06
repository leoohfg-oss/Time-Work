# Time-Work

Aplicativo simples de registro de ponto com FastAPI. Inclui:

- Autenticação por PIN com JWT.
- Registro de marcações de entrada/saída com geolocalização opcional.
- Painel administrativo para cadastro de colaboradores e consulta de eventos.

## Como executar

1. Crie um ambiente virtual e instale as dependências:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
2. Inicie a API:
   ```bash
   uvicorn main:app --reload
   ```
3. Acesse `http://localhost:8000/docs` para testar as rotas ou visite `http://localhost:8000/` para uma visão rápida.

O banco de dados `sqlite` (`ponto.db`) é criado automaticamente na primeira execução, bem como um usuário administrador padrão (`admin` / PIN `1234`). Altere o PIN após o primeiro acesso.

## Fluxo rápido de uso

Se a dúvida for “o que faço com isso?”, siga o roteiro abaixo para ver o sistema funcionando de ponta a ponta usando apenas `curl`.

1. **Faça login como administrador**
   ```bash
   curl -X POST http://localhost:8000/login \
     -d 'username=admin' \
     -d 'password=1234' \
     -d 'grant_type=' \
     -H 'Content-Type: application/x-www-form-urlencoded'
   ```
   O retorno inclui um `access_token`. Guarde-o para usar nos próximos passos:
   ```bash
   export TOKEN="<valor-do-access-token>"
   ```

2. **Cadastre um colaborador**
   ```bash
   curl -X POST http://localhost:8000/admin/employees \
     -H "Authorization: Bearer $TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"name": "Maria", "pin": "4321", "role": "Financeiro"}'
   ```

3. **Faça login como colaborador**
   ```bash
   curl -X POST http://localhost:8000/login \
     -d 'username=Maria' \
     -d 'password=4321' \
     -d 'grant_type=' \
     -H 'Content-Type: application/x-www-form-urlencoded'
   export EMP_TOKEN="<token-da-maria>"
   ```

4. **Bata o ponto**
   ```bash
   curl -X POST http://localhost:8000/clock \
     -H "Authorization: Bearer $EMP_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"type": "IN", "device": "Portaria"}'
   ```

5. **Consulte os registros**
   - Como colaborador:
     ```bash
     curl http://localhost:8000/me/events \
       -H "Authorization: Bearer $EMP_TOKEN"
     ```
   - Como administrador (todos os registros):
     ```bash
     curl "http://localhost:8000/admin/events" \
       -H "Authorization: Bearer $TOKEN"
     ```

Seguindo esse roteiro você percorre todo o ciclo de autenticação, cadastro e marcação de ponto.
