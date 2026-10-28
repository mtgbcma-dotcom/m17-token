# Continuidade do Bookstore — autenticação

Dei continuidade ao [módulo 16 — Paginação](https://github.com/mtgbcma-dotcom/m16-paginacao/pull/1), branch `paginacao`, commit `24f54666fe781ee505115442defc0ca0e71f7c4e`.

## Ambiente preservado

Recuperei `pyproject.toml`, `poetry.lock` e `requirements.txt` integralmente dessa base. Mantive o nome do projeto, as faixas de dependências e as versões do lock. Utilizei Poetry 2.5.1 para conferir o manifesto e instalar o ambiente. Não substituí o Poetry por pip.

## Funcionalidades anteriores preservadas

Mantive sem alterações os modelos, serializers, migração inicial, testes de serializers, testes de paginação e comando de demonstração. Os ViewSets de categorias e produtos mantêm a autenticação anterior. Acrescentei TokenAuthentication apenas aos pedidos, o app `rest_framework.authtoken` e a rota `/api/token/`.

Adaptei somente a expectativa de permissão nos testes de ViewSets para HTTP 401 nos pedidos sem token, preservando o HTTP 403 nas outras ViewSets. A suíte mantém os 65 testes anteriores e acrescenta 21 de autenticação, totalizando 86 testes.

## Reproduzir a validação

```powershell
poetry check --lock
poetry install --no-root --no-interaction
poetry run python manage.py check
poetry run python manage.py makemigrations --check --dry-run
poetry run python manage.py test --verbosity 2
poetry run python manage.py migrate --noinput
```

O banco local e o ambiente virtual não são versionados. Os arquivos necessários para recriá-los estão incluídos nesta entrega.

## Resultado da validação em ambiente novo

Validei esta correção em 28/10/2026, com Python 3.13 e Poetry 2.5.1:

- `poetry check --lock`: válido.
- `poetry install --no-root --no-interaction`: dependências instaladas a partir do lock em um ambiente virtual novo.
- `poetry run python manage.py check`: nenhum problema.
- `poetry run python manage.py makemigrations --check --dry-run`: nenhuma mudança pendente.
- `poetry run python manage.py test --verbosity 2`: 86 testes aprovados.
- `poetry run python manage.py migrate --noinput`: migrações aplicadas com sucesso.
