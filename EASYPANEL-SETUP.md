# Cobalt - Configuração para EasyPanel

Este guia explica como implantar o Cobalt no EasyPanel usando o arquivo `docker-compose.yml` otimizado.

## Pré-requisitos

- Servidor com EasyPanel instalado <mcreference link="https://easypanel.io/docs" index="2">2</mcreference>
- Pelo menos 2GB de RAM disponível <mcreference link="https://easypanel.io/docs" index="2">2</mcreference>
- Portas 80 e 443 disponíveis <mcreference link="https://easypanel.io/docs" index="2">2</mcreference>

## Configuração no EasyPanel

### 1. Criar Novo Projeto

1. Faça login no EasyPanel
2. Clique em "Create New Project"
3. Nome do projeto: `cobalt`
4. Selecione "Docker Compose" como tipo de implantação <mcreference link="https://docs.lowcoder.cloud/lowcoder-documentation/setup-and-run/self-hosting/easypanel" index="4">4</mcreference>

### 2. Upload do Docker Compose

1. Na seção "Docker Compose", cole o conteúdo do arquivo `docker-compose.yml`
2. Configure as variáveis de ambiente necessárias (veja seção abaixo)

### 3. Configuração de Variáveis de Ambiente

No painel ENV do EasyPanel, configure as seguintes variáveis:

#### **Variáveis Obrigatórias:**

```env
# URL da sua instância (OBRIGATÓRIO)
API_URL=https://seu-dominio.com/

# Segredo JWT (mínimo 16 caracteres)
JWT_SECRET=seu-jwt-secret-muito-seguro-aqui
```

#### **Variáveis Opcionais de Segurança:**

```env
# Proteção Turnstile (Cloudflare)
TURNSTILE_SITEKEY=sua-sitekey-turnstile
TURNSTILE_SECRET=seu-secret-turnstile

# Autenticação por API Key
API_AUTH_REQUIRED=0
API_KEY_URL=https://seu-servidor-de-keys.com/
```

#### **Configurações de Rate Limiting:**

```env
# Limite de requisições por minuto
RATELIMIT_WINDOW=60
RATELIMIT_MAX=20

# Limite para túneis
TUNNEL_RATELIMIT_WINDOW=60
TUNNEL_RATELIMIT_MAX=40

# Limite para sessões
SESSION_RATELIMIT_WINDOW=60
SESSION_RATELIMIT_MAX=10
```

#### **Configurações de Conteúdo:**

```env
# Limite de duração em segundos (padrão: 3 horas)
DURATION_LIMIT=10800

# Tempo de vida do túnel em segundos
TUNNEL_LIFESPAN=90
```

#### **Configurações do YouTube:**

```env
# Servidor de sessão do YouTube (opcional)
YOUTUBE_SESSION_SERVER=http://yt-session-generator:8080

# Permitir áudio de melhor qualidade
YOUTUBE_ALLOW_BETTER_AUDIO=1

# Cliente Innertube customizado
CUSTOM_INNERTUBE_CLIENT=WEB
```

#### **Configurações de Clustering (Redis):**

```env
# Para múltiplas instâncias
API_INSTANCE_COUNT=1
API_REDIS_URL=redis://redis:6379
REDIS_PASSWORD=senha-redis-segura
```

#### **Configurações de Proxy:**

```env
# Proxy externo (opcional)
API_EXTERNAL_PROXY=http://seu-proxy.com:8080
HTTP_PROXY=http://proxy.exemplo.com:8080
HTTPS_PROXY=http://proxy.exemplo.com:8080
```

### 4. Configuração de Portas

Para o serviço principal (`cobalt-api`): <mcreference link="https://docs.lowcoder.cloud/lowcoder-documentation/setup-and-run/self-hosting/easypanel" index="4">4</mcreference>
- **Porta do serviço**: `9000` (ou valor de `API_PORT`)
- **Deixe apenas a porta do serviço** - o EasyPanel usa Traefik como load balancer

### 5. Configuração de Domínio

1. Na seção "Domains" do EasyPanel:
2. Adicione seu domínio
3. Selecione o container `cobalt-api`
4. Configure a porta `9000`
5. Ative SSL automático (Let's Encrypt) <mcreference link="https://easypanel.io/docs/services/app" index="5">5</mcreference>

### 6. Volumes e Persistência

O docker-compose inclui volumes para: <mcreference link="https://easypanel.io/docs/services/app" index="5">5</mcreference>
- `cobalt-data`: Dados da aplicação
- `cobalt-logs`: Logs da aplicação
- `redis-data`: Dados do Redis (se habilitado)
- `redis-config`: Configuração do Redis

### 7. Serviços Opcionais

O compose inclui perfis para serviços opcionais:

#### **Redis (para clustering):**
```env
# Ativar Redis
DOCKER_COMPOSE_PROFILES=redis
REDIS_VERSION=7-alpine
REDIS_PASSWORD=senha-muito-segura
REDIS_MAX_MEMORY=256mb
```

#### **YouTube Session Generator:**
```env
# Ativar gerador de sessão do YouTube
DOCKER_COMPOSE_PROFILES=youtube-session
YT_SESSION_VERSION=webserver
YT_SESSION_PORT=8080
```

#### **Watchtower (atualizações automáticas):**
```env
# Ativar atualizações automáticas
DOCKER_COMPOSE_PROFILES=auto-update
WATCHTOWER_VERSION=latest
WATCHTOWER_INTERVAL=900
```

## Configurações Avançadas

### Cookies para Serviços Autenticados

Para suportar serviços que requerem autenticação:

1. Crie um arquivo `cookies.json` com o formato:
```json
{
  "youtube": "cookie_string_here",
  "instagram": "cookie_string_here"
}
```

2. Configure a variável:
```env
COOKIE_PATH=/cookies.json
COOKIE_FILE_PATH=./cookies.json
```

### Limites de Recursos

```env
# Limites de memória e CPU
MEMORY_LIMIT=1G
CPU_LIMIT=1.0
MEMORY_RESERVATION=512M
CPU_RESERVATION=0.5

# Limites para Redis
REDIS_MEMORY_LIMIT=512M
REDIS_CPU_LIMIT=0.5
```

### Desabilitar Serviços

```env
# Desabilitar serviços específicos (separados por vírgula)
DISABLED_SERVICES=tiktok,instagram
```

## Implantação

1. Configure todas as variáveis necessárias
2. Clique em "Deploy" no EasyPanel <mcreference link="https://docs.lowcoder.cloud/lowcoder-documentation/setup-and-run/self-hosting/easypanel" index="4">4</mcreference>
3. Aguarde a conclusão (pode levar alguns minutos)
4. Verifique os logs na seção "Deployments"
5. Configure o domínio na seção "Domains"

## Verificação

Após a implantação:

1. Acesse `https://seu-dominio.com/api/serverInfo`
2. Deve retornar informações do servidor
3. Teste a interface web em `https://seu-dominio.com/`

## Troubleshooting

### Problemas Comuns:

1. **Erro 500**: Verifique se `JWT_SECRET` tem pelo menos 16 caracteres
2. **Conexão recusada**: Verifique se a porta está configurada corretamente
3. **Problemas de proxy**: Configure `API_URL` com a URL correta
4. **Erro de Redis**: Se usar clustering, verifique `API_REDIS_URL`

### Logs:

Para verificar logs no EasyPanel:
1. Vá para "Deployments"
2. Clique no serviço
3. Veja a aba "Logs"

## Atualizações

Para atualizar:
1. Ative o perfil `auto-update` para atualizações automáticas
2. Ou atualize manualmente alterando `COBALT_VERSION`

## Suporte

- [Documentação oficial do Cobalt](https://github.com/imputnet/cobalt)
- [Documentação do EasyPanel](https://easypanel.io/docs)
- [Issues no GitHub](https://github.com/imputnet/cobalt/issues)

---

**Nota**: Este arquivo foi otimizado especificamente para EasyPanel. Para outras plataformas, use o `docker-compose.example.yml` original.