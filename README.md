# Bookstore — Autenticação por Token

Atividade do Módulo 17 de Continuação Back-End Python da EBAC.

Adicionei a autenticação por token do Django REST Framework somente à `OrderViewSet`. A API mantém o cadastro de categorias, produtos e pedidos, os serializers e a paginação de 5 registros por página.


## Continuidade do projeto e do ambiente

Dei continuidade à entrega do [módulo 16 — Paginação](https://github.com/mtgbcma-dotcom/m16-paginacao/pull/1). Mantive os modelos, os serializers, a migração inicial e os testes de serializers do Bookstore, além das funcionalidades acrescentadas nas etapas anteriores.

Nos módulos 15 e 16, utilizei Poetry com `pyproject.toml` e `poetry.lock`. A partir do módulo 17, passei a declarar as dependências em `requirements.txt` e a instalar com pip. Nas etapas com Docker, o próprio Dockerfile executa essa instalação durante a construção da imagem. Documentei essa mudança de ferramenta para deixar explícita a continuidade do mesmo projeto.

Para reproduzir esta entrega, utilizei os comandos e os arquivos de dependências desta versão, descritos abaixo. A instalação com pip segue as faixas registradas em `requirements.txt`; o lock do Poetry das etapas anteriores continua disponível nos respectivos repositórios.

## Implementação

- Inclusão de `rest_framework.authtoken` nos aplicativos instalados.
- Endpoint `POST /api/token/` para obter um token com usuário e senha válidos.
- `authentication_classes = [TokenAuthentication]` somente em `OrderViewSet`.
- Cabeçalho exigido em pedidos: `Authorization: Token <token>`.
- Pedidos sem token, com token inválido ou de usuário inativo retornam HTTP 401.
- A permissão de gestão `IsAdminUser` foi mantida: o usuário precisa ser da equipe, como o superusuário criado para os testes manuais.
- Categorias e produtos continuam com autenticação por sessão ou Basic Authentication; não receberam autenticação por token.
- Paginação e operações de cadastro, consulta, atualização e exclusão preservadas.

A autenticação identifica o usuário; a permissão verifica se ele pode acessar a gestão. Um token válido de usuário comum retorna HTTP 403 nos pedidos, enquanto um token válido de administrador permite o acesso.

## Tecnologias

- Python
- Django
- Django REST Framework
- SQLite

## Executar no Windows

Com Python 3.10 ou mais recente instalado, abra a pasta que contém `manage.py` e execute:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py createsuperuser
.\.venv\Scripts\python.exe manage.py seed_demo
.\.venv\Scripts\python.exe manage.py runserver
```

O comando opcional `seed_demo` cria 12 categorias e 12 produtos para conferir a paginação. Ele não cria usuários nem pedidos e não duplica os mesmos registros quando executado novamente.

## Acessos pelo navegador

- Raiz da API: http://127.0.0.1:8000/api/
- Categorias: http://127.0.0.1:8000/api/categories/
- Produtos: http://127.0.0.1:8000/api/products/
- Administração: http://127.0.0.1:8000/admin/

Em categorias e produtos, clique em **Log in** e use o superusuário criado.

Abrir `http://127.0.0.1:8000/api/orders/` no navegador sem enviar o token retorna HTTP 401, mesmo que exista uma sessão de login. Esse é o comportamento implementado para demonstrar a exigência de token somente nos pedidos.

## Testar pedidos com token no PowerShell

Mantenha o servidor rodando e abra outro terminal.

### 1. Conferir o bloqueio sem token

```powershell
try {
    (Invoke-WebRequest -Uri "http://127.0.0.1:8000/api/orders/" -UseBasicParsing).StatusCode
} catch {
    [int]$_.Exception.Response.StatusCode
}
```

Resultado esperado: `401`.

### 2. Obter um token

```powershell
$credencial = Get-Credential -Message "Informe o superusuario criado no projeto local"

$dadosLogin = @{
    username = $credencial.UserName
    password = $credencial.GetNetworkCredential().Password
}

$autenticacao = Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/token/" -Method Post -ContentType "application/x-www-form-urlencoded" -Body $dadosLogin

$cabecalhos = @{ Authorization = "Token $($autenticacao.token)" }

Remove-Variable dadosLogin, credencial
```

O token fica na variável da sessão do PowerShell. Não o inclua em prints, arquivos ou commits.

### 3. Consultar pedidos autenticados

```powershell
(Invoke-WebRequest -Uri "http://127.0.0.1:8000/api/orders/" -Headers $cabecalhos -UseBasicParsing).StatusCode

$pedidos = Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/orders/" -Headers $cabecalhos
$pedidos | ConvertTo-Json -Depth 8
```

Resultado esperado: HTTP `200` e a resposta paginada com `count`, `next`, `previous` e `results`. Em um banco sem pedidos, `count` será `0` e `results` será uma lista vazia.

### 4. Cadastrar um pedido

O exemplo abaixo utiliza usuário e produto com ID `1`. Confira os IDs existentes no seu banco e ajuste os valores antes de executar. O ID de um usuário pode ser consultado no painel administrativo.

```powershell
$novoPedido = @{
    user = 1
    product_ids = @(1)
    status = "pending"
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/orders/" -Method Post -Headers $cabecalhos -ContentType "application/json" -Body $novoPedido | ConvertTo-Json -Depth 8
```

### 5. Atualizar um pedido

Use o ID retornado no cadastro. Exemplo para o pedido `1`:

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:8000/api/orders/1/" -Method Patch -Headers $cabecalhos -ContentType "application/json" -Body '{"status":"completed"}' | ConvertTo-Json -Depth 8
```

## Endpoints

| Recurso | Endereço | Autenticação |
|---|---|---|
| Obter token | `POST /api/token/` | Usuário e senha válidos |
| Categorias | `/api/categories/` e `/api/categories/{id}/` | Sessão ou Basic Authentication |
| Produtos | `/api/products/` e `/api/products/{id}/` | Sessão ou Basic Authentication |
| Pedidos | `/api/orders/` e `/api/orders/{id}/` | Somente Token Authentication |

Os três recursos de gestão exigem um usuário da equipe. Os campos de escrita de produto e pedido continuam sendo `category_id`, `user` e `product_ids`.

## Verificações

```powershell
.\.venv\Scripts\python.exe manage.py check
.\.venv\Scripts\python.exe manage.py makemigrations --check --dry-run
.\.venv\Scripts\python.exe manage.py test --verbosity 2
```

A suíte contém **86 testes**. Além dos serializers, operações da API e paginação, os testes verificam obtenção de token, credenciais incorretas, usuário inativo, token inválido ou removido, bloqueio sem token, operações autenticadas e preservação da autenticação das outras ViewSets.

## Arquivos principais

- `api/views.py`: autenticação por token somente na `OrderViewSet`.
- `bookstore/settings.py`: aplicativo `authtoken` e paginação global.
- `bookstore/urls.py`: endpoint para obter token.
- `api/tests/test_token_authentication.py`: testes das requisições com e sem token.
- `api/tests/test_viewsets.py`: testes das operações e permissões.
- `api/tests/test_pagination.py`: testes da paginação.
- `api/tests/test_serializers.py`: testes dos serializers.

## Referência

[TokenAuthentication — Django REST Framework](https://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication)

## Autor

Bruno Costa Macedo de Almeida
