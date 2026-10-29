# Continuidade do Bookstore — autenticação

Dei continuidade ao [módulo 16 — Paginação](https://github.com/mtgbcma-dotcom/m16-paginacao/pull/1), branch `paginacao`, commit `24f54666fe781ee505115442defc0ca0e71f7c4e`.

## Ambiente preservado

Recuperei `pyproject.toml`, `poetry.lock` e `requirements.txt` integralmente dessa base. Mantive o nome do projeto, as faixas de dependências e as versões do lock. Utilizei Poetry 2.5.1 para conferir o manifesto e instalar o ambiente. Não substituí o Poetry por pip.

## Funcionalidades anteriores preservadas

Mantive os modelos, serializers, migração inicial, cenários anteriores de paginação e comando de demonstração. As categorias mantêm a autenticação anterior. Acrescentei TokenAuthentication apenas aos pedidos, o app `rest_framework.authtoken` e a rota `/api/token/`.

Ajustei a leitura de produtos conforme a orientação do tutor: listagem e detalhes são públicos. Mantive a escrita de produtos restrita à equipe e os pedidos protegidos por token. Atualizei as expectativas de leitura de produtos e acrescentei três testes para a consulta anônima, a paginação pública e o bloqueio de todas as operações de escrita para visitantes e usuários comuns. A suíte contém 89 testes.

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

## Correção da consulta pública de produtos — 29/10/2026

Atendi à orientação do tutor liberando a consulta da listagem e dos detalhes de produtos sem login. Mantive a paginação, a representação da categoria, as restrições de escrita dos produtos e a autenticação por token dos pedidos.

Reproduzi o bloqueio anterior nos novos testes de consulta anônima: as requisições retornavam HTTP 403. Após ajustar a permissão de leitura da ProductViewSet, a consulta e a navegação entre páginas passaram a retornar HTTP 200. Os pedidos sem token continuam retornando HTTP 401.

Validei o manifesto e o lock com `poetry check --lock`, a configuração com `poetry run python manage.py check` e as migrações com `poetry run python manage.py makemigrations --check --dry-run`. Executei `poetry run python manage.py test --verbosity 2 --noinput`: **89 testes aprovados**, incluindo os cenários de autenticação e de escrita protegida. Mantive `pyproject.toml` e `poetry.lock` sem alterações.
