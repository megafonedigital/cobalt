# Documentação dos Endpoints da API Cobalt

Esta documentação apresenta todos os endpoints disponíveis na API Cobalt, incluindo métodos HTTP, parâmetros, corpo de requisição e funcionalidades.

## Índice
- [Autenticação](#autenticação)
- [Endpoints Principais](#endpoints-principais)
  - [POST /](#post-)
  - [POST /session](#post-session)
  - [GET /](#get-)
  - [GET /tunnel](#get-tunnel)
  - [GET /itunnel](#get-itunnel)
- [Headers Obrigatórios](#headers-obrigatórios)
- [Rate Limiting](#rate-limiting)
- [Códigos de Status HTTP](#códigos-de-status-http)

## Autenticação

A API suporta dois métodos de autenticação opcionais:

### 1. API Key Authentication
```
Authorization: Api-Key aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
```

### 2. Bearer Token Authentication (JWT)
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Endpoints Principais

### POST `/`
**Funcionalidade:** Endpoint principal de processamento da API Cobalt para download de mídia.

**Método:** `POST`

**Headers Obrigatórios:**
```
Accept: application/json
Content-Type: application/json
```

**Corpo da Requisição:**
```json
{
  "url": "string (obrigatório)",
  "audioBitrate": "320|256|128|96|64|8",
  "audioFormat": "best|mp3|ogg|wav|opus",
  "downloadMode": "auto|audio|mute",
  "filenameStyle": "classic|pretty|basic|nerdy",
  "videoQuality": "max|4320|2160|1440|1080|720|480|360|240|144",
  "localProcessing": "disabled|preferred|forced",
  "disableMetadata": "boolean",
  "alwaysProxy": "boolean",
  "subtitleLang": "string (ISO 639-1)",
  "youtubeVideoCodec": "h264|av1|vp9",
  "youtubeVideoContainer": "auto|mp4|webm|mkv",
  "youtubeDubLang": "string (ISO 639-1)",
  "convertGif": "boolean",
  "allowH265": "boolean",
  "tiktokFullAudio": "boolean",
  "youtubeBetterAudio": "boolean",
  "youtubeHLS": "boolean"
}
```

**Parâmetros Detalhados:**
- `url`: URL da mídia a ser processada (obrigatório)
- `audioBitrate`: Taxa de bits do áudio em kbps (padrão: "128")
- `audioFormat`: Formato de áudio de saída (padrão: "mp3")
- `downloadMode`: Modo de download (padrão: "auto")
- `filenameStyle`: Estilo do nome do arquivo (padrão: "basic")
- `videoQuality`: Qualidade do vídeo (padrão: "1080")
- `localProcessing`: Configuração de processamento local (padrão: "disabled")
- `disableMetadata`: Desabilitar metadados no arquivo (padrão: false)
- `alwaysProxy`: Sempre usar proxy para arquivos (padrão: false)
- `subtitleLang`: Idioma das legendas (opcional)
- `youtubeVideoCodec`: Codec de vídeo para YouTube (padrão: "h264")
- `youtubeVideoContainer`: Container de vídeo para YouTube (padrão: "auto")
- `youtubeDubLang`: Idioma de dublagem do YouTube (opcional)
- `convertGif`: Converter GIFs do Twitter para formato GIF real (padrão: true)
- `allowH265`: Permitir vídeos H265/HEVC (padrão: false)
- `tiktokFullAudio`: Baixar áudio original do TikTok (padrão: false)
- `youtubeBetterAudio`: Preferir áudio de maior qualidade do YouTube (padrão: false)
- `youtubeHLS`: Usar formatos HLS do YouTube (padrão: false)

**Tipos de Resposta:**
1. **tunnel**: Cobalt está fazendo proxy/remux/transcodificação
2. **local-processing**: Cobalt faz proxy, mas processamento local necessário
3. **redirect**: Redirecionamento para URL direta do serviço
4. **picker**: Múltiplos itens para escolher
5. **error**: Erro ocorreu

---

### POST `/session`
**Funcionalidade:** Gerar tokens JWT para autenticação (se habilitado).

**Método:** `POST`

**Headers Obrigatórios:**
```
cf-turnstile-response: <turnstile_response_token>
```

**Corpo da Requisição:** Vazio

**Resposta de Sucesso:**
```json
{
  "token": "string",
  "exp": "number"
}
```

**Parâmetros de Resposta:**
- `token`: Token Bearer para autenticação posterior
- `exp`: Tempo de vida do token em segundos

---

### GET `/`
**Funcionalidade:** Obter informações básicas da instância da API.

**Método:** `GET`

**Parâmetros:** Nenhum

**Resposta:**
```json
{
  "cobalt": {
    "version": "string",
    "url": "string",
    "startTime": "string",
    "turnstileSitekey": "string (opcional)",
    "services": ["string"]
  },
  "git": {
    "commit": "string",
    "branch": "string",
    "remote": "string"
  }
}
```

**Parâmetros de Resposta:**
- `cobalt.version`: Versão do Cobalt
- `cobalt.url`: URL da instância
- `cobalt.startTime`: Tempo de início em milissegundos Unix
- `cobalt.turnstileSitekey`: Chave do site Turnstile (se configurado)
- `cobalt.services`: Array de serviços suportados
- `git.commit`: Hash do commit
- `git.branch`: Branch do Git
- `git.remote`: Remote do Git

---

### GET `/tunnel`
**Funcionalidade:** Endpoint para túneis de arquivo (proxy/remux/transcode).

**Método:** `GET`

**Parâmetros de Query:**
- `id`: ID do túnel (21 caracteres, obrigatório)
- `exp`: Timestamp de expiração (13 caracteres, obrigatório)
- `sig`: Assinatura (43 caracteres, obrigatório)
- `sec`: Chave de segurança (43 caracteres, obrigatório)
- `iv`: Vetor de inicialização (22 caracteres, obrigatório)
- `p`: Parâmetro de ping (opcional)

**Resposta:** Stream de arquivo

**Headers de Resposta:**
- `Content-Length`: Tamanho do arquivo em bytes (quando conhecido)
- `Estimated-Content-Length`: Tamanho estimado em bytes
- `Content-Disposition`: Informações de disposição do arquivo

---

### GET `/itunnel`
**Funcionalidade:** Túnel interno para streaming (apenas localhost).

**Método:** `GET`

**Restrições:** Apenas acessível de 127.0.0.1

**Parâmetros de Query:**
- `id`: ID do túnel interno (21 caracteres, obrigatório)

**Resposta:** Stream de arquivo interno

---

### GET `/favicon.ico`
**Funcionalidade:** Endpoint para favicon (retorna 404).

**Método:** `GET`

**Resposta:** HTTP 404

---

### GET `/*` (Wildcard)
**Funcionalidade:** Redireciona qualquer rota não encontrada para a raiz.

**Método:** `GET`

**Resposta:** Redirecionamento para `/`

## Headers Obrigatórios

Para requisições POST principais:
```
Accept: application/json
Content-Type: application/json
```

Para autenticação (opcional):
```
Authorization: <scheme> <token>
```

Para sessões Turnstile:
```
cf-turnstile-response: <response_token>
```

## Rate Limiting

Todos os endpoints (exceto `GET /`) são limitados por taxa e retornam headers de status:
- `RateLimit-Limit`: Limite de requisições
- `RateLimit-Policy`: Política de rate limiting
- `RateLimit-Remaining`: Requisições restantes
- `RateLimit-Reset`: Tempo para reset do limite

**Tipos de Rate Limiting:**
1. **Session Limiter**: Para endpoint `/session`
2. **API Limiter**: Para endpoint principal `/`
3. **Tunnel Limiter**: Para endpoint `/tunnel`

## Códigos de Status HTTP

### Códigos de Sucesso
- `200`: OK - Requisição bem-sucedida

### Códigos de Erro
- `400`: Bad Request - Requisição inválida
- `401`: Unauthorized - Não autorizado
- `403`: Forbidden - Acesso negado
- `404`: Not Found - Recurso não encontrado
- `429`: Too Many Requests - Limite de taxa excedido
- `500`: Internal Server Error - Erro interno do servidor

## Serviços Suportados

A API Cobalt suporta download de mídia dos seguintes serviços:
- YouTube
- TikTok
- Twitter/X
- Instagram
- Facebook
- Reddit
- Twitch
- SoundCloud
- Vimeo
- Dailymotion
- Bilibili
- VK
- OK.ru
- Snapchat
- Streamable
- Pinterest
- Tumblr
- Loom
- Newgrounds
- Rutube
- Bluesky
- Xiaohongshu (Little Red Book)

## Notas Importantes

1. **CORS**: A API suporta CORS com métodos GET e POST
2. **Proxy Trust**: A aplicação confia em proxies loopback e uniquelocal
3. **Limite de Corpo**: Requisições JSON limitadas a 1024 bytes
4. **Criptografia**: Cifras são randomizadas a cada 30 minutos
5. **Instâncias Múltiplas**: Suporte para múltiplas instâncias com reusePort
6. **Monitoramento**: Arquivo de ambiente pode ser monitorado para mudanças

---

*Documentação gerada automaticamente baseada na análise do código fonte da API Cobalt.*