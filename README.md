# Docker Secrets para PostgreSQL

A forma mais segura de passar variáveis para o serviço do Banco de Dados é através dos **secrets** do Docker. Essa alternativa armazena, no host, as variáveis `POSTGRES_DB`, `POSTGRES_USER` e `POSTGRES_PASSWORD` de maneira segura.

## Passos para Utilizar Secrets

### 1. Criar os secrets
Cada secret deve ser criado com o valor desejado:

```bash
echo "senha_aqui" | docker secret create postgres_password -
# ou
# docker secret create postgres_password arquivo.txt  # senha no arquivo
```
A ideia é a mesma para as outras variáveis sensíveis.
### 2. Utilizar a tag `secrets` no serviço
Adicione a tag `secrets:` logo após a tag `environment:` no serviço do banco:

```yaml
secrets:
  - postgres_user
  - postgres_password
  - postgres_db
```

### 3. Definir os secrets (devem existir no Swarm)

```yaml
secrets:
  postgres_user:
    external: true  
  postgres_password:
    external: true
  postgres_db:
    external: true
```

## Configuração do Serviço

No serviço do banco, use as variáveis com sufixo `_FILE` para mapear os arquivos dos secrets:

```yaml
environment:
  - POSTGRES_USER_FILE=/run/secrets/postgres_user
  - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password
  - POSTGRES_DB_FILE=/run/secrets/postgres_db
```

Assim, as credenciais são lidas de arquivos (os secrets) de maneira criptografada, garantindo:
- Segurança (não expostas em logs ou inspect)
- Acesso restrito apenas ao container

## Exemplo Completo de `docker-stack.yml` para PostgreSQL utilizando Secrets

```yaml
version: '3.8'
services:
  db:
    image: postgres:14.18
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER_FILE=/run/secrets/postgres_user
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password
      - POSTGRES_DB_FILE=/run/secrets/postgres_db
    secrets:
      - postgres_user
      - postgres_password
      - postgres_db
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - db_network
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
networks:
  db_network:
    driver: overlay
secrets:
  postgres_user:
    external: true
  postgres_password:
    external: true
  postgres_db:
    external: true
volumes:
  db_data:
    driver: local
```
