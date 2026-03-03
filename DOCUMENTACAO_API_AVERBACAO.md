# e-Reobot API — Averbação de Exportação

**Versão:** 1.0.0
**Data:** 03 de Março de 2026

---

## Sumário

1. [Visão Geral](#visão-geral)
2. [Autenticação](#autenticação)
3. [Endpoints](#endpoints)
   - [Averbação](#averbação)
   - [Empresa](#empresa)
   - [Certificado](#certificado)
4. [Modelos de Dados](#modelos-de-dados)
5. [Códigos de Status HTTP](#códigos-de-status-http)
6. [Exemplos de Integração](#exemplos-de-integração)
7. [Rate Limiting](#rate-limiting)
8. [Suporte](#suporte)

---

## Visão Geral

A **e-Reobot API de Averbação** é a interface dedicada para consulta e download de Eventos de Averbação de Exportação vinculados a NF-es.

### Documentação Interativa (Swagger)

```
GET /swagger-ui/index.html?urls.primaryName=averbacao-cliente
```

### Características

- Autenticação JWT (Bearer Token)
- Consulta por chave de acesso (44 dígitos)
- Listagem paginada com filtros por CNPJ e período de `dataHoraEvento`
- Consultas assíncronas para grandes volumes
- Download em lote (ZIP)
- Webhook para notificações em tempo real

> Todas as requisições devem usar HTTPS.

---

## Autenticação

### Obter Token JWT

```
POST /api/v1/login
```

**Request Body:**
```json
{
  "login": "seu.email@empresa.com",
  "senha": "SuaSenhaSegura123"
}
```

**Response (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "login": "seu.email@empresa.com",
  "name": "Nome do Usuário",
  "autenticado": true,
  "message": "Autenticação realizada com sucesso"
}
```

**Response (401 Unauthorized):**
```json
{
  "error": "Credenciais inválidas",
  "message": "Usuário ou senha incorretos"
}
```

### Usando o Token

Inclua o token em todas as requisições no header:

```
Authorization: Bearer {token}
```

---

## Endpoints

### Averbação

#### GET /api/v1/averbacao/{chaveAcesso}

Retorna os dados e XML de um evento de averbação vinculado a uma NF-e.

**Parâmetros:**

| Parâmetro | Local | Tipo | Obrigatório | Descrição |
|-----------|-------|------|-------------|-----------|
| `chaveAcesso` | path | String | Sim | Chave de acesso da NF-e (44 dígitos) |

**Response (200 OK):**
```json
{
  "chaveAcesso": "35200812345678000195550010000000011234567890",
  "tipoEvento": "110130",
  "cnpjEmitente": "12345678000195",
  "cnpjDestinatario": null,
  "xml": "<procEventoNFe>...</procEventoNFe>"
}
```

**Códigos de Status:**
- `200` — Evento encontrado
- `400` — Chave de acesso inválida (não tem 44 dígitos)
- `401` — Token ausente ou inválido
- `404` — Evento não encontrado

---

#### GET /api/v1/averbacao

Lista eventos de averbação com filtros, retornando resultados paginados.

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `cnpjEmpresa` | String | Sim | CNPJ da empresa (14 dígitos) | `12345678000195` |
| `dataHoraEventoDe` | String | Não | Data do evento a partir de (YYYY-MM-DD) | `2025-01-01` |
| `dataHoraEventoAte` | String | Não | Data do evento até (YYYY-MM-DD) | `2025-12-31` |
| `page` | Integer | Não | Número da página, inicia em 0 (padrão: 0) | `0` |
| `size` | Integer | Não | Itens por página (padrão: 20, máx: 100) | `20` |

**Response (200 OK):**
```json
{
  "content": [
    {
      "chaveAcesso": "35200812345678000195550010000000011234567890",
      "tipoEvento": "110130",
      "cnpjEmitente": "12345678000195",
      "cnpjDestinatario": null,
      "xml": "<procEventoNFe>...</procEventoNFe>"
    }
  ],
  "totalElements": 100,
  "totalPages": 5,
  "number": 0,
  "size": 20
}
```

**Códigos de Status:**
- `200` — Consulta realizada com sucesso
- `400` — Parâmetros inválidos ou CNPJ sem permissão
- `401` — Token ausente ou inválido
- `429` — Rate limit excedido

---

#### POST /api/v1/averbacao/consultar-async

Cria uma consulta assíncrona para grandes volumes. O processamento ocorre em background e os resultados ficam disponíveis para download após conclusão.

**Request Body:**
```json
{
  "cnpjCpfEmitente": "12345678000195",
  "dataEmissaoDe": "2025-01-01",
  "dataEmissaoAte": "2025-12-31"
}
```

**Response (202 Accepted):**
```json
{
  "idProtocolo": "AVERB-550e8400-e29b-41d4-a716-446655440000",
  "status": "AGUARDANDO",
  "mensagem": "Consulta criada com sucesso. Use o idProtocolo para acompanhar o status."
}
```

**Códigos de Status:**
- `202` — Consulta criada e enfileirada
- `400` — Parâmetros inválidos
- `401` — Não autenticado

---

#### GET /api/v1/averbacao/status/{idProtocolo}

Consulta o status de processamento de uma consulta assíncrona.

**Parâmetros:**

| Parâmetro | Local | Tipo | Obrigatório | Descrição |
|-----------|-------|------|-------------|-----------|
| `idProtocolo` | path | String | Sim | ID retornado pelo endpoint de consulta assíncrona |

**Response (200 OK):**
```json
{
  "idProtocolo": "AVERB-550e8400-e29b-41d4-a716-446655440000",
  "status": "CONCLUIDO",
  "totalDocumentos": 42,
  "mensagem": "Processamento concluído com sucesso"
}
```

**Valores possíveis de `status`:**

| Status | Descrição |
|--------|-----------|
| `AGUARDANDO` | Na fila de processamento |
| `PROCESSANDO` | Execução em andamento |
| `CONCLUIDO` | Concluído, XMLs disponíveis para download |
| `ERRO` | Falha no processamento |

**Códigos de Status:**
- `200` — Status retornado
- `404` — Protocolo não encontrado

---

#### POST /api/v1/averbacao/download

Baixa múltiplos XMLs de averbação compactados em um único arquivo ZIP.

**Request Body:**
```json
[
  "35200812345678000195550010000000011234567890",
  "35200812345678000195550010000000021234567891"
]
```

**Response (200 OK):**

Arquivo `averbacoes.zip` (`Content-Type: application/octet-stream`) contendo um XML por chave de acesso.

**Códigos de Status:**
- `200` — Arquivo ZIP gerado
- `400` — Lista de chaves inválida
- `401` — Não autenticado
- `404` — Nenhum documento encontrado para as chaves informadas

---

#### POST /api/v1/averbacao/webhook

Registra uma URL para receber notificações via HTTP POST quando novas averbações forem detectadas para o CNPJ informado.

**Request Body:**
```json
{
  "cnpjEmpresa": "12345678000195",
  "url": "https://seu-sistema.com.br/webhook/averbacao"
}
```

**Response (200 OK):** Webhook registrado com sucesso.

**Payload enviado ao webhook (exemplo):**
```json
{
  "evento": "AVERBACAO_DETECTADA",
  "cnpjEmpresa": "12345678000195",
  "chaveAcesso": "35200812345678000195550010000000011234567890",
  "dataHoraEvento": "2025-06-07T04:52:16"
}
```

**Códigos de Status:**
- `200` — Webhook registrado
- `400` — CNPJ ou URL ausente
- `403` — Usuário sem permissão para o CNPJ informado

---

### Empresa

#### GET /api/v1/empresa/{cnpj}

Consulta dados cadastrais de uma empresa.

**Parâmetros:**

| Parâmetro | Local | Tipo | Obrigatório | Descrição |
|-----------|-------|------|-------------|-----------|
| `cnpj` | path | String | Sim | CNPJ da empresa (14 dígitos) |

**Response (200 OK):**
```json
{
  "cnpj": "12345678000195",
  "razaoSocial": "Empresa Exemplo LTDA",
  "fantasia": "Empresa Exemplo",
  "uf": "SP"
}
```

---

### Certificado

#### POST /api/v1/certificado/{cnpj}

Cadastra ou substitui o certificado digital A1 associado à empresa.

**Parâmetros:**

| Parâmetro | Local | Tipo | Obrigatório | Descrição |
|-----------|-------|------|-------------|-----------|
| `cnpj` | path | String | Sim | CNPJ da empresa (14 dígitos) |

**Request Body (multipart/form-data):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `file` | File | Sim | Arquivo do certificado digital A1 (.pfx / .p12) |
| `senha` | String | Sim | Senha do certificado |

**Response (200 OK):**
```json
{
  "cnpj": "12345678000195",
  "validoAPartirDe": "2024-01-01T00:00:00.000+00:00",
  "validoAte": "2026-12-31T23:59:59.000+00:00"
}
```

---

## Modelos de Dados

### AverbacaoRequest (GET /api/v1/averbacao — query params)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `cnpjEmpresa` | String | Sim | CNPJ da empresa cliente |
| `dataHoraEventoDe` | String (YYYY-MM-DD) | Não | Início do range de `dataHoraEvento` |
| `dataHoraEventoAte` | String (YYYY-MM-DD) | Não | Fim do range de `dataHoraEvento` |
| `page` | Integer | Não | Página (padrão: 0) |
| `size` | Integer | Não | Tamanho (padrão: 20) |

### NFeEventoDTO (response de averbação)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `chaveAcesso` | String | Chave de acesso da NF-e (44 dígitos) |
| `tipoEvento` | String | Código do tipo de evento (ex: `110130`) |
| `cnpjEmitente` | String | CNPJ do emitente da NF-e |
| `cnpjDestinatario` | String | CNPJ do destinatário (pode ser null) |
| `xml` | String | XML completo do evento de averbação |

---

## Códigos de Status HTTP

| Código | Significado |
|--------|-------------|
| `200` | Sucesso |
| `202` | Aceito (consulta assíncrona criada) |
| `400` | Requisição inválida — verifique os parâmetros |
| `401` | Não autenticado — token ausente, expirado ou inválido |
| `403` | Sem permissão para acessar o CNPJ informado |
| `404` | Recurso não encontrado |
| `429` | Rate limit excedido — aguarde antes de tentar novamente |
| `500` | Erro interno do servidor |

**Estrutura de erro padrão:**
```json
{
  "status": "BAD_REQUEST",
  "timestamp": "2025-10-18T10:15:30",
  "message": "Descrição do erro"
}
```

---

## Exemplos de Integração

### curl

**Autenticar:**
```bash
TOKEN=$(curl -s -X POST https://api.ereobot.com.br/api/v1/login \
  -H "Content-Type: application/json" \
  -d '{"login":"seu@email.com","senha":"suasenha"}' \
  | jq -r '.token')
```

**Listar averbações do mês:**
```bash
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  "https://api.ereobot.com.br/api/v1/averbacao?cnpjEmpresa=12345678000195&dataHoraEventoDe=2025-01-01&dataHoraEventoAte=2025-01-31&page=0&size=20"
```

**Buscar por chave de acesso:**
```bash
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  "https://api.ereobot.com.br/api/v1/averbacao/35200812345678000195550010000000011234567890"
```

**Download em lote:**
```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '["35200812345678000195550010000000011234567890"]' \
  "https://api.ereobot.com.br/api/v1/averbacao/download" \
  --output averbacoes.zip
```

---

### Python

```python
import requests

BASE_URL = "https://api.ereobot.com.br"

# Autenticar
resp = requests.post(f"{BASE_URL}/api/v1/login", json={
    "login": "seu@email.com",
    "senha": "suasenha"
})
token = resp.json()["token"]
headers = {"Authorization": f"Bearer {token}"}

# Listar averbações
params = {
    "cnpjEmpresa": "12345678000195",
    "dataHoraEventoDe": "2025-01-01",
    "dataHoraEventoAte": "2025-12-31",
    "page": 0,
    "size": 20
}
resp = requests.get(f"{BASE_URL}/api/v1/averbacao", headers=headers, params=params)
data = resp.json()

print(f"Total de eventos: {data['totalElements']}")
for evento in data['content']:
    print(f"  Chave: {evento['chaveAcesso']} | Tipo: {evento['tipoEvento']}")
```

---

### Java

```java
import org.springframework.web.client.RestTemplate;
import org.springframework.http.*;
import java.util.Map;

public class AverbacaoClient {

    private static final String BASE_URL = "https://api.ereobot.com.br";
    private final RestTemplate restTemplate = new RestTemplate();

    public String login(String email, String senha) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        String body = String.format("{\"login\":\"%s\",\"senha\":\"%s\"}", email, senha);
        ResponseEntity<Map> resp = restTemplate.exchange(
            BASE_URL + "/api/v1/login",
            HttpMethod.POST,
            new HttpEntity<>(body, headers),
            Map.class
        );
        return (String) resp.getBody().get("token");
    }

    public Map listarAverbacoes(String token, String cnpjEmpresa, String de, String ate) {
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        String url = BASE_URL + "/api/v1/averbacao?cnpjEmpresa={cnpj}&dataHoraEventoDe={de}&dataHoraEventoAte={ate}";
        ResponseEntity<Map> resp = restTemplate.exchange(
            url, HttpMethod.GET,
            new HttpEntity<>(headers), Map.class,
            cnpjEmpresa, de, ate
        );
        return resp.getBody();
    }
}
```

---

## Rate Limiting

| Janela | Limite |
|--------|--------|
| Por minuto | 60 requisições |
| Por hora | 1.000 requisições |

Quando o limite é excedido, a API retorna `429 Too Many Requests`.

```json
{
  "status": "TOO_MANY_REQUESTS",
  "timestamp": "2025-10-18T10:15:30",
  "message": "Rate limit excedido. Aguarde antes de fazer novas requisições."
}
```

---

## Suporte

- **Email:** suporte@ereobot.com
- **Swagger interativo:** `/swagger-ui/index.html?urls.primaryName=averbacao-cliente`
- **Documentação geral:** `/swagger-ui/index.html?urls.primaryName=geral`

---

**Última atualização:** 03 de Março de 2026
**Versão:** 1.0.0
**e-Reobot — Automação de Documentos Fiscais** © 2026
