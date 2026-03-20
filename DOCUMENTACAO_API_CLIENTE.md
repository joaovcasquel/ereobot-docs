# e-Reobot API Pública - Documentação Completa

**Versão:** 2.4.0
**Data:** 19 de Março de 2026

---

## 📋 Sumário

1. [Visão Geral](#-visão-geral)
2. [Autenticação](#-autenticação)
3. [Endpoints Disponíveis](#-endpoints-disponíveis)
   - [NF-e (Nota Fiscal Eletrônica)](#nf-e---nota-fiscal-eletrônica)
   - [CT-e (Conhecimento de Transporte Eletrônico)](#ct-e---conhecimento-de-transporte-eletrônico)
   - [NFS-e (Nota Fiscal de Serviço Eletrônica)](#nfs-e---nota-fiscal-de-serviço-eletrônica)
   - [NFS-e Nacional (Padrão Nacional)](#nfs-e-nacional---padrão-nacional)
   - [Municípios Habilitados (NFS-e)](#2-listar-municípios-habilitados)
   - [NFC-e (Nota Fiscal do Consumidor Eletrônica)](#nfc-e---nota-fiscal-do-consumidor-eletrônica)
   - [Certificado](#certificado)
   - [Empresa](#empresa)
   - [Averbação de Exportação](#averbação-de-exportação)
   - [Extrato de Captura de Averbações por Empresa](#7-extrato-de-captura-de-averbações-por-empresa)
   - [Assinatura de Averbação](#8-listar-assinaturas-de-averbação)
4. [Modelos de Dados](#-modelos-de-dados)
5. [Códigos de Status HTTP](#-códigos-de-status-http)
6. [Exemplos de Integração](#-exemplos-de-integração)
7. [Boas Práticas](#-boas-práticas)
8. [Rate Limiting](#-rate-limiting)
9. [Suporte e Contato](#-suporte-e-contato)

---

## 🎯 Visão Geral

A **e-Reobot API Pública** é uma API RESTful que permite a consulta e download de documentos fiscais eletrônicos brasileiros (NF-e, CT-e, NFS-e e NFC-e) de forma programática.

### Características Principais

- ✅ **Autenticação JWT**: Segurança baseada em tokens Bearer
- ✅ **RESTful**: Seguindo os padrões REST
- ✅ **Paginação**: Consultas retornam resultados paginados
- ✅ **Filtros Avançados**: Múltiplos critérios de busca
- ✅ **Download em Lote**: Suporte a download de múltiplos XMLs compactados
- ✅ **Consultas Assíncronas**: Processamento em background para grandes volumes
- ✅ **Controle de Acesso**: Baseado em CNPJ/CPF autorizado

> **Nota:** Todas as requisições devem usar HTTPS para garantir a segurança dos dados.

### Documentação Interativa (Swagger)

```
GET /swagger-ui/index.html
```

---

## 🔐 Autenticação

Todos os endpoints (exceto o login) exigem autenticação via token JWT.

### Obter Token de Acesso

**Endpoint:**
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
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
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

Inclua o token em todas as requisições subsequentes no header:

```
Authorization: Bearer {seu_token_jwt}
```

**Exemplo:**
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR..." \
     https://api.ereobot.com.br/api/v1/nfe/35200812345678000195550010000000011234567890
```

---

## 📡 Endpoints Disponíveis

### NF-e - Nota Fiscal Eletrônica

#### 1. Consultar NF-e por Chave de Acesso

Retorna os dados completos e XML de uma NF-e específica.

**Endpoint:**
```
GET /api/v1/nfe/{chaveAcesso}
```

**Parâmetros:**
- `chaveAcesso` (path) - String de 44 dígitos

**Response (200 OK):**
```json
{
  "id": "507f1f77bcf86cd799439011",
  "chaveAcesso": "35200812345678000195550010000000011234567890",
  "numeroNota": "1",
  "serie": "1",
  "dataEmissao": "2020-08-15T10:30:00",
  "valorTotal": 1500.50,
  "cnpjCpfEmitente": "12345678000195",
  "razaoSocialEmitente": "Empresa Emitente LTDA",
  "cnpjCpfDestinatario": "98765432000110",
  "razaoSocialDestinatario": "Empresa Destinatária LTDA",
  "status": "AUTORIZADA",
  "xml": "<nfeProc xmlns=\"http://www.portalfiscal.inf.br/nfe\">...</nfeProc>"
}
```

**Códigos de Status:**
- `200` - NF-e encontrada
- `403` - Sem permissão para acessar esta NF-e
- `404` - NF-e não encontrada

---

#### 2. Listar NF-es com Filtros

Retorna uma lista paginada de NF-es.

**Endpoint:**
```
GET /api/v1/nfe
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `cnpjCpfEmitente` | String | Não | CNPJ/CPF do emitente | 12345678000195 |
| `cnpjCpfDestinatario` | String | Não | CNPJ/CPF do destinatário | 98765432000110 |
| `dataEmissaoDe` | String | Não | Data inicial (YYYY-MM-DD) | 2024-01-01 |
| `dataEmissaoAte` | String | Não | Data final (YYYY-MM-DD) | 2024-12-31 |
| `numNotaDe` | Integer | Não | Número inicial da nota | 100 |
| `numNotaAte` | Integer | Não | Número final da nota | 200 |
| `pageNum` | Integer | Não | Número da página (inicia em 0) | 0 |

**Exemplo de Requisição:**
```
GET /api/v1/nfe?cnpjCpfDestinatario=12345678000195&dataEmissaoDe=2024-01-01&dataEmissaoAte=2024-12-31&pageNum=0
```

**Response (200 OK):**
```json
{
  "content": [
    {
      "id": "507f1f77bcf86cd799439011",
      "chaveAcesso": "35200812345678000195550010000000011234567890",
      "numeroNota": "1",
      "serie": "1",
      "dataEmissao": "2020-08-15T10:30:00",
      "valorTotal": 1500.50,
      "cnpjCpfEmitente": "12345678000195",
      "razaoSocialEmitente": "Empresa Emitente LTDA",
      "cnpjCpfDestinatario": "98765432000110",
      "razaoSocialDestinatario": "Empresa Destinatária LTDA",
      "status": "AUTORIZADA"
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20,
    "sort": {
      "sorted": false,
      "unsorted": true,
      "empty": true
    }
  },
  "totalElements": 150,
  "totalPages": 8,
  "last": false,
  "size": 20,
  "number": 0,
  "first": true,
  "numberOfElements": 20,
  "empty": false
}
```

---

#### 3. Criar Consulta Assíncrona de NF-es

Cria um job para processamento em background de múltiplas NF-es.

**Endpoint:**
```
POST /api/v1/nfe/async
```

**Request Body:**
```json
{
  "cnpjEmpresa": "12345678000195",
  "chavesDeAcesso": [
    "35200812345678000195550010000000011234567890",
    "35200812345678000195550010000000021234567891",
    "35200812345678000195550010000000031234567892"
  ]
}
```

**Response (202 Accepted):**
```json
{
  "idExecucao": "NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "PENDING",
  "mensagem": "Consulta criada com sucesso. Processamento iniciado.",
  "chavesDeAcesso": [
    "35200812345678000195550010000000011234567890",
    "35200812345678000195550010000000021234567891",
    "35200812345678000195550010000000031234567892"
  ],
  "criadoEm": "2025-11-15T10:30:00.000Z"
}
```

> **Nota:** O campo `idExecucao` retorna um identificador único no formato `NFE-{UUID}`. Este identificador deve ser usado para consultar o status e fazer o download dos resultados.

**Códigos de Status:**
- `202` - Job criado com sucesso
- `400` - Requisição inválida
- `404` - CNPJ não encontrado

> **💡 Sobre o Identificador de Execução (`idExecucao`):**
> 
> O sistema retorna um identificador único no formato `NFE-{UUID}` (exemplo: `NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890`).
> Este formato oferece:
> - ✅ **Segurança:** Não expõe estrutura interna do banco de dados
> - ✅ **Não-previsibilidade:** Impossível enumerar ou adivinhar IDs de outros jobs
> - ✅ **Rastreabilidade:** Prefixo `NFE-` facilita identificação em logs
>
> **Importante:** Guarde este identificador para consultar status e fazer download dos resultados.

---

#### 4. Consultar Status de Job Assíncrono

Verifica o andamento do processamento.

**Endpoint:**
```
GET /api/v1/nfe/async/{idExecucao}
```

**Parâmetros:**
- `idExecucao` (path) - Identificador único retornado na criação do job (formato: `NFE-{UUID}`)

**Response (200 OK):**
```json
{
  "idExecucao": "NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "cnpjEmpresa": "12345678000195",
  "status": "COMPLETED",
  "progressoAtual": 3,
  "totalChaves": 3,
  "chavesDeAcesso": [
    "35200812345678000195550010000000011234567890",
    "35200812345678000195550010000000021234567891",
    "35200812345678000195550010000000031234567892"
  ],
  "tentativas": 0,
  "mensagemErro": null,
  "criadoEm": "2025-11-15T10:30:00.000Z",
  "atualizadoEm": "2025-11-15T10:35:00.000Z",
  "concluidoEm": "2025-11-15T10:35:00.000Z",
  "expiraEm": "2025-11-22T10:30:00.000Z"
}
```

**Status Possíveis:**
- `PENDING` - Aguardando processamento
- `PROCESSING` - Em processamento
- `COMPLETED` - Concluído com sucesso
- `FAILED` - Falha no processamento
- `CANCELLED` - Cancelado

> **⚠️ Importante - Expiração de Jobs:**
> 
> Jobs concluídos ficam disponíveis por **7 dias** após a conclusão. Após este período, os dados são removidos automaticamente.
> Certifique-se de fazer o download dos resultados dentro deste prazo. O campo `expiraEm` na resposta indica a data/hora de expiração.

---

#### 5. Download de XMLs em Lote (por ID de Execução)

Baixa arquivo compactado (GZIP) com os XMLs processados.

**Endpoint:**
```
GET /api/v1/nfe/async/{idExecucao}/download
```

**Parâmetros:**
- `idExecucao` (path) - Identificador único do job (formato: `NFE-{UUID}`)

**Response (200 OK):**
- Content-Type: `application/octet-stream`
- Content-Disposition: `attachment; filename=nfe_{idExecucao}.tar.gz`
- Body: Arquivo binário GZIP

**Códigos de Status:**
- `200` - Download iniciado
- `400` - Job ainda não completado
- `404` - Job não encontrado

**Exemplo:**
```bash
curl -H "Authorization: Bearer {token}" \
     -o nfes.tar.gz \
     https://api.ereobot.com.br/api/v1/nfe/async/NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890/download
```

---

#### 6. Download de XMLs por Lista de Chaves

Baixa XMLs específicos sem criar job assíncrono.

**Endpoint:**
```
POST /api/v1/nfe/download
```

**Request Body:**
```json
[
  "35200812345678000195550010000000011234567890",
  "35200812345678000195550010000000021234567891"
]
```

**Response (200 OK):**
- Content-Type: `application/octet-stream`
- Content-Disposition: `attachment; filename=nfe_chaves.tar.gz`
- Body: Arquivo binário GZIP

---

### CT-e - Conhecimento de Transporte Eletrônico

#### 1. Consultar CT-e por Chave de Acesso

**Endpoint:**
```
GET /api/v1/cte/{chaveAcesso}
```

**Parâmetros:**
- `chaveAcesso` (path) - String de 44 dígitos

**Response:** Similar ao NF-e, com campos específicos de CT-e.

---

#### 2. Listar CT-es com Filtros

**Endpoint:**
```
GET /api/v1/cte
```

**Parâmetros de Query:** Mesmos da NF-e

**Response:** Lista paginada de CT-es

---

### NFS-e - Nota Fiscal de Serviço Eletrônica

#### 1. Listar NFS-es com Filtros

**Endpoint:**
```
GET /api/v1/nfse
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `tipoNota` | String | Sim | Tipo da nota: `tomada` ou `prestada` | `tomada` |
| `cnpjCpfTomadorPrestador` | String | Sim | CNPJ/CPF do tomador (quando `tomada`) ou prestador (quando `prestada`) | `12345678000195` |
| `codigoMun` | Integer | Sim | Código IBGE do município | `3550308` |
| `dataEmissaoDe` | String | Sim | Data inicial (YYYY-MM-DD) | `2025-12-01` |
| `dataEmissaoAte` | String | Sim | Data final (YYYY-MM-DD) | `2025-12-31` |
| `numNotaDe` | Integer | Não | Número inicial da nota | `100` |
| `numNotaAte` | Integer | Não | Número final da nota | `200` |
| `pageNum` | Integer | Não | Número da página (inicia em 0) | `0` |

> **Regra:** intervalo máximo entre `dataEmissaoDe` e `dataEmissaoAte` é de 31 dias.

**Response (200 OK):** Lista paginada de `NFSeDTO`.

**Códigos de Status:**
- `200` — Consulta realizada com sucesso
- `400` — Parâmetros inválidos
- `403` — Sem permissão para os CNPJs/CPFs informados

---

#### 2. Listar Municípios Habilitados

**Endpoint:**
```
GET /api/v1/nfse/nfse/municipios-habilitados
```

**Response (200 OK):**
```json
{
  "municipiosHabilitados": [
    {
      "codigoMunicipio": 3550308,
      "nome": "São Paulo",
      "uf": "SP"
    }
  ]
}
```

**Códigos de Status:**
- `200` — Consulta realizada com sucesso
- `401` — Não autenticado
- `500` — Erro interno

---

### NFS-e Nacional - Padrão Nacional

#### 1. Listar NFS-e Nacional com Filtros

**Endpoint:**
```
GET /api/v1/nfse-nacional
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `cnpjCpfTomador` | String | Não* | CNPJ/CPF do tomador | `35150145000153` |
| `cnpjCpfPrestador` | String | Não* | CNPJ/CPF do prestador | `35150145000153` |
| `dataEmissaoDe` | String | Sim | Data inicial (YYYY-MM-DD) | `2025-10-01` |
| `dataEmissaoAte` | String | Sim | Data final (YYYY-MM-DD) | `2025-10-31` |
| `chaveAcesso` | String | Não | Filtra por chave de acesso | `35200812345678000195550010000000011234567890` |
| `pageNum` | Integer | Não | Número da página (inicia em 0) | `0` |

\* É obrigatório informar ao menos um entre `cnpjCpfTomador` e `cnpjCpfPrestador`.

> **Regra:** intervalo máximo entre `dataEmissaoDe` e `dataEmissaoAte` é de 31 dias.

**Response (200 OK):** Lista paginada de `NFSeNacionalDTO`.

**Códigos de Status:**
- `200` — Consulta realizada com sucesso
- `400` — Parâmetros inválidos
- `403` — Sem permissão para os CNPJs/CPFs informados

---

### NFC-e - Nota Fiscal do Consumidor Eletrônica

#### 1. Consultar NFC-e por Chave de Acesso

**Endpoint:**
```
GET /api/v1/nfce/{chaveAcesso}
```

**Response:** Dados completos da NFC-e

---

#### 2. Listar NFC-es com Filtros

**Endpoint:**
```
GET /api/v1/nfce
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| `cnpjEmitente` | String | CNPJ do emitente |
| `cnpjCpfDestinatario` | String | CNPJ/CPF do destinatário |
| `dataEmissaoDe` | String | Data inicial |
| `dataEmissaoAte` | String | Data final |
| `numNotaDe` | Integer | Número inicial |
| `numNotaAte` | Integer | Número final |

---

### Certificado

Gerenciamento do certificado digital A1 associado às empresas do usuário autenticado.

#### Cadastrar / Atualizar Certificado Digital

**Endpoint:**
```
POST /api/v1/certificado/{cnpj}
```

**Parâmetros:**
- `cnpj` (path) — CNPJ da empresa (14 dígitos)

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

**Códigos de Status:**
- `200` — Certificado cadastrado com sucesso
- `400` — CNPJ inválido ou arquivo não suportado
- `401` — Não autenticado
- `403` — Empresa não pertence ao usuário autenticado

---

### Empresa

Gerenciamento das empresas vinculadas ao grupo do usuário autenticado.

#### 1. Consultar Empresa

**Endpoint:**
```
GET /api/v1/empresa/{cnpjCpf}
```

**Parâmetros:**
- `cnpjCpf` (path) — CNPJ/CPF da empresa

**Response (200 OK):**
```json
{
  "cnpj": "12345678000195",
  "ativa": true,
  "nome": "Empresa Exemplo LTDA",
  "codigoUf": 35,
  "modulos": {
    "nfe": true,
    "cte": false,
    "nfse": false,
    "nfce": false
  },
  "certificado": {
    "cnpj": "12345678000195",
    "validoAPartirDe": "2024-01-01T00:00:00.000+00:00",
    "validoAte": "2026-12-31T23:59:59.000+00:00"
  }
}
```

**Códigos de Status:**
- `200` — Empresa encontrada
- `403` — Empresa não pertence ao usuário autenticado
- `404` — Empresa não encontrada

---

#### 2. Criar Empresa

**Endpoint:**
```
POST /api/v1/empresa
```

**Request Body (form-data ou JSON):**

| Campo | Tipo | Obrigatório | Descrição | Exemplo |
|-------|------|-------------|-----------|---------|
| `cnpjCpf` | String | Sim | CNPJ/CPF da empresa | `12345678000195` |
| `nome` | String | Sim | Nome / Razão Social da empresa | `Empresa Exemplo LTDA` |
| `ativa` | Boolean | Não | Se a empresa está ativa (padrão: false) | `true` |
| `codigoUf` | Integer | Não | Código UF do IBGE | `35` |
| `inscricaoEstadual` | String | Não | Campo usado somente quando o cadastro for CPF | `123456789` |

**Response (200 OK):** `EmpresaResponse` (mesmo formato do GET).

**Códigos de Status:**
- `200` — Empresa criada
- `400` — Dados inválidos
- `401` — Não autenticado

---

#### 3. Atualizar Empresa

**Endpoint:**
```
PATCH /api/v1/empresa/{cnpjCpf}
```

**Parâmetros:**
- `cnpjCpf` (path) — CNPJ/CPF da empresa

**Request Body (form-data ou JSON):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `nome` | String | Sim | Nome / Razão Social da empresa |
| `ativa` | Boolean | Não | Se a empresa está ativa |
| `codigoUf` | Integer | Não | Código UF do IBGE |

**Response (200 OK):** `EmpresaResponse` (mesmo formato do GET).

**Códigos de Status:**
- `200` — Empresa atualizada
- `403` — Empresa não pertence ao usuário autenticado
- `404` — Empresa não encontrada

---

### Averbação de Exportação

Consulta e download de Eventos de Averbação de Exportação vinculados a NF-es.

#### 1. Consultar Averbação por Chave de Acesso

**Endpoint:**
```
GET /api/v1/averbacao/{chaveAcesso}
```

**Parâmetros:**
- `chaveAcesso` (path) — Chave de acesso da NF-e (44 dígitos)

**Response (200 OK):**
```json
{
  "chaveAcesso": "35200812345678000195550010000000011234567890",
  "tipoEvento": "110130",
  "cnpjCpfEmitente": "12345678000195",
  "cnpjCpfDestinatario": null,
  "xml": "<procEventoNFe>...</procEventoNFe>"
}
```

**Códigos de Status:**
- `200` — Evento encontrado
- `400` — Chave de acesso inválida (não tem 44 dígitos)
- `401` — Token ausente ou inválido
- `404` — Evento não encontrado

---

#### 2. Listar Averbações com Filtros

**Endpoint:**
```
GET /api/v1/averbacao
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `cnpjEmpresa` | String | Sim | CNPJ da empresa (14 dígitos) | `12345678000195` |
| `dataHoraEventoDe` | String | Não | Data do evento a partir de (YYYY-MM-DD) | `2025-01-01` |
| `dataHoraEventoAte` | String | Não | Data do evento até (YYYY-MM-DD) | `2025-12-31` |
| `pageNum` | Integer | Não | Número da página, inicia em 0 | `0` |

**Response (200 OK):**
```json
{
  "content": [
    {
      "chaveAcesso": "35200812345678000195550010000000011234567890",
      "tipoEvento": "110130",
      "cnpjCpfEmitente": "12345678000195",
      "cnpjCpfDestinatario": null,
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

---

#### 3. Criar Consulta Assíncrona de Averbações

**Endpoint:**
```
POST /api/v1/averbacao/consultar-async
```

**Request Body:**
```json
{
  "cnpjEmpresa": "12345678000195",
  "chavesDeAcesso": [
    "35200812345678000195550010000000011234567890",
    "35200812345678000195550010000000021234567891"
  ]
}
```

**Response (202 Accepted):**
```json
{
  "idExecucao": "AVERB-550e8400-e29b-41d4-a716-446655440000",
  "status": "PENDING",
  "mensagem": "Consulta criada com sucesso. Processamento iniciado.",
  "chavesDeAcesso": [
    "35200812345678000195550010000000011234567890",
    "35200812345678000195550010000000021234567891"
  ],
  "criadoEm": "2026-03-19T10:30:00.000Z"
}
```

**Códigos de Status:**
- `202` — Consulta criada e enfileirada
- `400` — Parâmetros inválidos
- `401` — Não autenticado

---

#### 4. Consultar Status da Consulta Assíncrona

**Endpoint:**
```
GET /api/v1/averbacao/status/{idProtocolo}
```

**Parâmetros:**
- `idProtocolo` (path) — ID retornado pelo endpoint de consulta assíncrona

**Response (200 OK):**
```json
{
  "idExecucao": "AVERB-550e8400-e29b-41d4-a716-446655440000",
  "cnpjEmpresa": "12345678000195",
  "status": "COMPLETED",
  "progressoAtual": 2,
  "totalChaves": 2,
  "tentativas": 1,
  "mensagemErro": null,
  "criadoEm": "2026-03-19T10:30:00.000Z",
  "atualizadoEm": "2026-03-19T10:35:00.000Z"
}
```

**Valores possíveis de `status`:** `PENDING`, `PROCESSING`, `RETRYING`, `FAILED`, `COMPLETED`

**Códigos de Status:**
- `200` — Status retornado
- `404` — Protocolo não encontrado

---

#### 5. Download em Lote de XMLs de Averbação

**Endpoint:**
```
POST /api/v1/averbacao/download
```

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

#### 6. Registrar Webhook de Averbação

**Endpoint:**
```
POST /api/v1/averbacao/webhook
```

**Request Body:**
```json
{
  "cnpjEmpresa": "12345678000195",
  "url": "https://seu-sistema.com.br/webhook/averbacao"
}
```

**Response (200 OK):** Webhook registrado com sucesso.

**Payload enviado ao webhook:**
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

#### 7. Extrato de Captura de Averbações por Empresa

**Endpoint:**
```
GET /api/v1/averbacao/extrato-captura-averbacao/{mes}/{ano}
```

**Parâmetros:**
- `mes` (path) — Mês de referência (1 a 12)
- `ano` (path) — Ano de referência

**Response (200 OK):**
```json
[
  {
    "cnpjEmpresa": "12345678000195",
    "nomeEmpresa": "Empresa Exemplo LTDA",
    "qtdAverbadas": 152,
    "qtdNaoAverbadas": 8
  }
]
```

**Códigos de Status:**
- `200` — Extrato retornado com sucesso
- `401` — Não autenticado
- `403` — Sem permissão de acesso
- `500` — Erro interno

---

#### 8. Listar Assinaturas de Averbação

**Endpoint:**
```
GET /api/v1/averbacao/assinatura
```

**Parâmetros de Query:**

| Parâmetro | Tipo | Obrigatório | Descrição | Exemplo |
|-----------|------|-------------|-----------|---------|
| `idEmpresa` | String | Sim | CNPJ/CPF da empresa | `12345678000195` |
| `ano` | Integer | Sim | Ano da assinatura | `2026` |
| `mes` | Integer | Sim | Mês da assinatura (1 a 12) | `3` |

**Response (200 OK):**
```json
[
  {
    "id": "67f4e1ddc49f4429c9aa10ff",
    "ano": 2026,
    "mes": 3,
    "creditos": 5000,
    "valor": 199.90,
    "idEmpresa": "12345678000195",
    "createdAt": "2026-03-15T10:30:00"
  }
]
```

**Códigos de Status:**
- `200` — Assinaturas listadas com sucesso
- `403` — Sem permissão para a empresa informada

---

#### 9. Criar Assinatura de Averbação

**Endpoint:**
```
POST /api/v1/averbacao/assinatura
```

**Request Body:**
```json
{
  "ano": 2026,
  "mes": 3,
  "creditos": 5000,
  "valor": 199.90,
  "idEmpresa": "12345678000195"
}
```

**Response (201 Created):**
```json
{
  "id": "67f4e1ddc49f4429c9aa10ff",
  "ano": 2026,
  "mes": 3,
  "creditos": 5000,
  "valor": 199.90,
  "idEmpresa": "12345678000195",
  "createdAt": "2026-03-19T11:05:00"
}
```

**Códigos de Status:**
- `201` — Assinatura criada com sucesso
- `400` — Erro de validação nos campos
- `403` — Sem permissão para a empresa informada

---

#### 10. Atualizar Assinatura de Averbação

**Endpoint:**
```
PATCH /api/v1/averbacao/assinatura/{id}
```

**Parâmetros:**
- `id` (path) — ID da assinatura

**Request Body (campos opcionais):**
```json
{
  "ano": 2026,
  "mes": 4,
  "creditos": 6000,
  "valor": 229.90,
  "idEmpresa": "12345678000195"
}
```

**Response (200 OK):** Retorna a assinatura atualizada.

**Códigos de Status:**
- `200` — Assinatura atualizada com sucesso
- `403` — Sem permissão para a assinatura
- `404` — Assinatura não encontrada

---

#### 11. Deletar Assinatura de Averbação

**Endpoint:**
```
DELETE /api/v1/averbacao/assinatura/{id}
```

**Parâmetros:**
- `id` (path) — ID da assinatura

**Response (204 No Content):** Assinatura removida com sucesso.

**Códigos de Status:**
- `204` — Assinatura removida
- `403` — Sem permissão para a assinatura
- `404` — Assinatura não encontrada

---

## 📊 Modelos de Dados

### NFeCTeDTO

```json
{
  "id": "string",
  "chaveAcesso": "string (44 caracteres)",
  "numeroNota": "string",
  "serie": "string",
  "dataEmissao": "datetime (ISO 8601)",
  "valorTotal": "number",
  "cnpjCpfEmitente": "string",
  "razaoSocialEmitente": "string",
  "cnpjCpfDestinatario": "string",
  "razaoSocialDestinatario": "string",
  "status": "string",
  "xml": "string (XML completo)"
}
```

### NFSeDTO

```json
{
  "id": "string",
  "numeroNota": "string",
  "codigoVerificacao": "string",
  "dataEmissao": "datetime",
  "valorServicos": "number",
  "cnpjCpfPrestador": "string",
  "razaoSocialPrestador": "string",
  "cnpjCpfTomador": "string",
  "razaoSocialTomador": "string",
  "municipio": "string",
  "codigoMunicipio": "integer",
  "xml": "string"
}
```

### NFCeDTO

```json
{
  "id": "string",
  "chaveAcesso": "string (44 caracteres)",
  "numeroNota": "string",
  "serie": "string",
  "dataEmissao": "datetime",
  "valorTotal": "number",
  "cnpjEmitente": "string",
  "razaoSocialEmitente": "string",
  "status": "string",
  "xml": "string"
}
```

### NFeEventoDTO (Averbação)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `chaveAcesso` | String | Chave de acesso da NF-e (44 dígitos) |
| `tipoEvento` | String | Código do tipo de evento (ex: `110130`) |
| `cnpjCpfEmitente` | String | CNPJ/CPF do emitente da NF-e |
| `cnpjCpfDestinatario` | String | CNPJ/CPF do destinatário (pode ser null) |
| `xml` | String | XML completo do evento de averbação |

### NFSeNacionalDTO

```json
{
  "chaveAcesso": "string",
  "numero": 12345,
  "situacao": "AUTORIZADA",
  "cnpjCpfEmitente": "12345678000195",
  "nomeEmitente": "Prestador Exemplo LTDA",
  "cnpjCpfTomador": "98765432000110",
  "dataEmissao": "2026-03-01T10:30:00",
  "municipio": "São Paulo",
  "valorTotal": 1500.75,
  "xmlGzip": "H4sIAAAAA..."
}
```

### MunicipiosHabilitadosResponse

```json
{
  "municipiosHabilitados": [
    {
      "codigoMunicipio": 3550308,
      "nome": "São Paulo",
      "uf": "SP"
    }
  ]
}
```

### AssinaturaAverbacao

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `id` | String | Identificador da assinatura |
| `ano` | Integer | Ano da assinatura (mínimo: 2026) |
| `mes` | Integer | Mês da assinatura (1 a 12) |
| `creditos` | Integer | Quantidade de créditos (mínimo: 1) |
| `valor` | Decimal | Valor da assinatura (mínimo: 0.01) |
| `idEmpresa` | String | CNPJ/CPF da empresa |
| `createdAt` | DateTime | Data de criação |

### AverbacaoEmpresasCountDTO

```json
{
  "cnpjEmpresa": "12345678000195",
  "nomeEmpresa": "Empresa Exemplo LTDA",
  "qtdAverbadas": 152,
  "qtdNaoAverbadas": 8
}
```

### Página de Resultados

```json
{
  "content": [],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 20
  },
  "totalElements": 0,
  "totalPages": 0,
  "last": false,
  "first": true,
  "numberOfElements": 0,
  "empty": false
}
```

---

## 🚦 Códigos de Status HTTP

| Código | Significado | Descrição |
|--------|-------------|-----------|
| 200 | OK | Requisição bem-sucedida |
| 201 | Created | Recurso criado com sucesso |
| 202 | Accepted | Job assíncrono criado com sucesso |
| 204 | No Content | Recurso removido com sucesso |
| 400 | Bad Request | Requisição inválida ou parâmetros incorretos |
| 401 | Unauthorized | Token ausente, inválido ou expirado |
| 403 | Forbidden | Sem permissão para acessar o recurso solicitado |
| 404 | Not Found | Recurso não encontrado |
| 429 | Too Many Requests | Limite de requisições excedido |
| 500 | Internal Server Error | Erro interno do servidor |

### Exemplos de Respostas de Erro

**401 - Não Autorizado:**
```json
{
  "error": "Unauthorized",
  "message": "Token inválido ou expirado"
}
```

**403 - Sem Permissão:**
```json
{
  "error": "Acesso negado",
  "message": "Você não possui permissão para acessar esta NF-e"
}
```

**404 - Não Encontrado:**
```json
{
  "error": "NF-e não encontrada",
  "message": "Não foi encontrada NF-e com a chave de acesso informada"
}
```

**429 - Rate Limit:**
```json
{
  "error": "Too Many Requests",
  "message": "Você excedeu o limite de 60 requisições por minuto",
  "retryAfter": 30
}
```

---

## 💻 Exemplos de Integração

### JavaScript (Axios)

```javascript
const axios = require('axios');

const API_BASE_URL = 'https://api.ereobot.com.br';
let authToken = null;

// 1. Fazer login
async function login() {
  try {
    const response = await axios.post(`${API_BASE_URL}/api/v1/login`, {
      login: 'seu.email@empresa.com',
      senha: 'SuaSenha123'
    });
    
    authToken = response.data.token;
    console.log('Login realizado com sucesso!');
    return authToken;
  } catch (error) {
    console.error('Erro no login:', error.response?.data);
    throw error;
  }
}

// 2. Buscar NF-e por chave
async function buscarNFe(chaveAcesso) {
  try {
    const response = await axios.get(
      `${API_BASE_URL}/api/v1/nfe/${chaveAcesso}`,
      {
        headers: {
          'Authorization': `Bearer ${authToken}`
        }
      }
    );
    
    return response.data;
  } catch (error) {
    console.error('Erro ao buscar NF-e:', error.response?.data);
    throw error;
  }
}

// 3. Listar NF-es com filtros
async function listarNFes(filtros) {
  try {
    const response = await axios.get(`${API_BASE_URL}/api/v1/nfe`, {
      headers: {
        'Authorization': `Bearer ${authToken}`
      },
      params: {
        cnpjCpfDestinatario: filtros.cnpj,
        dataEmissaoDe: filtros.dataInicio,
        dataEmissaoAte: filtros.dataFim,
        pageNum: filtros.pagina || 0
      }
    });
    
    return response.data;
  } catch (error) {
    console.error('Erro ao listar NF-es:', error.response?.data);
    throw error;
  }
}

// 4. Consulta assíncrona
async function consultaAssincrona(cnpj, chaves) {
  try {
    // Criar job
    const jobResponse = await axios.post(
      `${API_BASE_URL}/api/v1/nfe/async`,
      {
        cnpjEmpresa: cnpj,
        chavesDeAcesso: chaves
      },
      {
        headers: {
          'Authorization': `Bearer ${authToken}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    const idExecucao = jobResponse.data.idExecucao;
    console.log('Job criado:', idExecucao);
    // Exemplo de ID retornado: NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890
    
    // Aguardar conclusão
    let status = 'PENDING';
    while (status !== 'COMPLETED' && status !== 'FAILED') {
      await new Promise(resolve => setTimeout(resolve, 5000)); // Aguarda 5s
      
      const statusResponse = await axios.get(
        `${API_BASE_URL}/api/v1/nfe/async/${idExecucao}`,
        {
          headers: {
            'Authorization': `Bearer ${authToken}`
          }
        }
      );
      
      status = statusResponse.data.status;
      console.log('Status atual:', status);
    }
    
    if (status === 'COMPLETED') {
      // Fazer download
      const downloadResponse = await axios.get(
        `${API_BASE_URL}/api/v1/nfe/async/${idExecucao}/download`,
        {
          headers: {
            'Authorization': `Bearer ${authToken}`
          },
          responseType: 'arraybuffer'
        }
      );
      
      // Salvar arquivo
      const fs = require('fs');
      fs.writeFileSync(`nfes_${idExecucao}.tar.gz`, downloadResponse.data);
      console.log('Download concluído!');
    }
    
    return idExecucao;
  } catch (error) {
    console.error('Erro na consulta assíncrona:', error.response?.data);
    throw error;
  }
}

// Exemplo de uso
(async () => {
  await login();
  
  // Buscar uma NF-e específica
  const nfe = await buscarNFe('35200812345678000195550010000000011234567890');
  console.log('NF-e encontrada:', nfe.numeroNota);
  
  // Listar NF-es
  const lista = await listarNFes({
    cnpj: '12345678000195',
    dataInicio: '2024-01-01',
    dataFim: '2024-12-31',
    pagina: 0
  });
  console.log('Total de NF-es:', lista.totalElements);
})();
```

---

### Python (Requests)

```python
import requests
import time
from typing import List, Dict, Optional

API_BASE_URL = 'https://api.ereobot.com.br'

class EReobotAPI:
    def __init__(self):
        self.token = None
        self.session = requests.Session()
    
    def login(self, email: str, senha: str) -> str:
        """Realiza login e retorna o token JWT"""
        response = self.session.post(
            f'{API_BASE_URL}/api/v1/login',
            json={
                'login': email,
                'senha': senha
            }
        )
        response.raise_for_status()
        
        data = response.json()
        self.token = data['token']
        self.session.headers.update({
            'Authorization': f'Bearer {self.token}'
        })
        
        print('Login realizado com sucesso!')
        return self.token
    
    def buscar_nfe(self, chave_acesso: str) -> Dict:
        """Busca NF-e por chave de acesso"""
        response = self.session.get(
            f'{API_BASE_URL}/api/v1/nfe/{chave_acesso}'
        )
        response.raise_for_status()
        return response.json()
    
    def listar_nfes(self, 
                    cnpj_destinatario: Optional[str] = None,
                    cnpj_emitente: Optional[str] = None,
                    data_inicio: Optional[str] = None,
                    data_fim: Optional[str] = None,
                    pagina: int = 0) -> Dict:
        """Lista NF-es com filtros"""
        params = {
            'pageNum': pagina
        }
        
        if cnpj_destinatario:
            params['cnpjCpfDestinatario'] = cnpj_destinatario
        if cnpj_emitente:
            params['cnpjCpfEmitente'] = cnpj_emitente
        if data_inicio:
            params['dataEmissaoDe'] = data_inicio
        if data_fim:
            params['dataEmissaoAte'] = data_fim
        
        response = self.session.get(
            f'{API_BASE_URL}/api/v1/nfe',
            params=params
        )
        response.raise_for_status()
        return response.json()
    
    def consulta_assincrona(self, 
                           cnpj: str, 
                           chaves: List[str]) -> str:
        """Cria job de consulta assíncrona"""
        response = self.session.post(
            f'{API_BASE_URL}/api/v1/nfe/async',
            json={
                'cnpjEmpresa': cnpj,
                'chavesDeAcesso': chaves
            }
        )
        response.raise_for_status()
        
        data = response.json()
        id_execucao = data['idExecucao']
        print(f'Job criado: {id_execucao}')
        # Exemplo: NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890
        return id_execucao
    
    def verificar_status_job(self, id_execucao: str) -> Dict:
        """Verifica status de um job assíncrono"""
        response = self.session.get(
            f'{API_BASE_URL}/api/v1/nfe/async/{id_execucao}'
        )
        response.raise_for_status()
        return response.json()
    
    def aguardar_conclusao_job(self, 
                               id_execucao: str, 
                               intervalo: int = 5) -> Dict:
        """Aguarda conclusão de um job"""
        while True:
            status_data = self.verificar_status_job(id_execucao)
            status = status_data['status']
            
            print(f'Status: {status}')
            
            if status in ['COMPLETED', 'FAILED']:
                return status_data
            
            time.sleep(intervalo)
    
    def download_job(self, 
                     id_execucao: str, 
                     nome_arquivo: str = None) -> str:
        """Faz download dos XMLs de um job"""
        if not nome_arquivo:
            nome_arquivo = f'nfes_{id_execucao}.tar.gz'
        
        response = self.session.get(
            f'{API_BASE_URL}/api/v1/nfe/async/{id_execucao}/download',
            stream=True
        )
        response.raise_for_status()
        
        with open(nome_arquivo, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        
        print(f'Download concluído: {nome_arquivo}')
        return nome_arquivo
    
    def download_por_chaves(self, 
                           chaves: List[str], 
                           nome_arquivo: str = 'nfes.tar.gz') -> str:
        """Faz download de XMLs por lista de chaves"""
        response = self.session.post(
            f'{API_BASE_URL}/api/v1/nfe/download',
            json=chaves,
            stream=True
        )
        response.raise_for_status()
        
        with open(nome_arquivo, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        
        print(f'Download concluído: {nome_arquivo}')
        return nome_arquivo


# Exemplo de uso
if __name__ == '__main__':
    api = EReobotAPI()
    
    # 1. Login
    api.login('seu.email@empresa.com', 'SuaSenha123')
    
    # 2. Buscar NF-e específica
    nfe = api.buscar_nfe('35200812345678000195550010000000011234567890')
    print(f'NF-e encontrada: {nfe["numeroNota"]}')
    
    # 3. Listar NF-es
    resultado = api.listar_nfes(
        cnpj_destinatario='12345678000195',
        data_inicio='2024-01-01',
        data_fim='2024-12-31',
        pagina=0
    )
    print(f'Total de NF-es: {resultado["totalElements"]}')
    
    # 4. Consulta assíncrona
    chaves = [
        '35200812345678000195550010000000011234567890',
        '35200812345678000195550010000000021234567891'
    ]
    id_job = api.consulta_assincrona('12345678000195', chaves)
    
    # 5. Aguardar conclusão
    status_final = api.aguardar_conclusao_job(id_job)
    
    # 6. Download
    if status_final['status'] == 'COMPLETED':
        api.download_job(id_job)
```

---

### Java (Spring RestTemplate)

```java
import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import org.springframework.core.io.Resource;
import java.util.*;

public class EReobotApiClient {
    
    private static final String API_BASE_URL = "https://api.ereobot.com.br";
    private String token;
    private RestTemplate restTemplate;
    
    public EReobotApiClient() {
        this.restTemplate = new RestTemplate();
    }
    
    // 1. Login
    public String login(String email, String senha) {
        String url = API_BASE_URL + "/api/v1/login";
        
        Map<String, String> request = new HashMap<>();
        request.put("login", email);
        request.put("senha", senha);
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<Map<String, String>> entity = new HttpEntity<>(request, headers);
        
        ResponseEntity<Map> response = restTemplate.postForEntity(url, entity, Map.class);
        
        this.token = (String) response.getBody().get("token");
        System.out.println("Login realizado com sucesso!");
        
        return this.token;
    }
    
    // 2. Buscar NF-e
    public Map<String, Object> buscarNFe(String chaveAcesso) {
        String url = API_BASE_URL + "/api/v1/nfe/" + chaveAcesso;
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        
        ResponseEntity<Map> response = restTemplate.exchange(
            url, 
            HttpMethod.GET, 
            entity, 
            Map.class
        );
        
        return response.getBody();
    }
    
    // 3. Listar NF-es
    public Map<String, Object> listarNFes(String cnpjDestinatario, 
                                          String dataInicio, 
                                          String dataFim,
                                          int pagina) {
        String url = String.format(
            "%s/api/v1/nfe?cnpjCpfDestinatario=%s&dataEmissaoDe=%s&dataEmissaoAte=%s&pageNum=%d",
            API_BASE_URL, cnpjDestinatario, dataInicio, dataFim, pagina
        );
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        
        ResponseEntity<Map> response = restTemplate.exchange(
            url, 
            HttpMethod.GET, 
            entity, 
            Map.class
        );
        
        return response.getBody();
    }
    
    // 4. Consulta assíncrona
    public String consultaAssincrona(String cnpj, List<String> chaves) {
        String url = API_BASE_URL + "/api/v1/nfe/async";
        
        Map<String, Object> request = new HashMap<>();
        request.put("cnpjEmpresa", cnpj);
        request.put("chavesDeAcesso", chaves);
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<Map<String, Object>> entity = new HttpEntity<>(request, headers);
        
        ResponseEntity<Map> response = restTemplate.postForEntity(url, entity, Map.class);
        
        String idExecucao = (String) response.getBody().get("idExecucao");
        System.out.println("Job criado: " + idExecucao);
        // Exemplo: NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890
        
        return idExecucao;
    }
    
    // 5. Verificar status
    public Map<String, Object> verificarStatusJob(String idExecucao) {
        String url = API_BASE_URL + "/api/v1/nfe/async/" + idExecucao;
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        
        ResponseEntity<Map> response = restTemplate.exchange(
            url, 
            HttpMethod.GET, 
            entity, 
            Map.class
        );
        
        return response.getBody();
    }
    
    // 6. Download
    public byte[] downloadJob(String idExecucao) {
        String url = API_BASE_URL + "/api/v1/nfe/async/" + idExecucao + "/download";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(token);
        
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        
        ResponseEntity<Resource> response = restTemplate.exchange(
            url, 
            HttpMethod.GET, 
            entity, 
            Resource.class
        );
        
        try {
            return response.getBody().getInputStream().readAllBytes();
        } catch (Exception e) {
            throw new RuntimeException("Erro ao ler arquivo", e);
        }
    }
    
    // Exemplo de uso
    public static void main(String[] args) {
        EReobotApiClient client = new EReobotApiClient();
        
        // Login
        client.login("seu.email@empresa.com", "SuaSenha123");
        
        // Buscar NF-e
        Map<String, Object> nfe = client.buscarNFe("35200812345678000195550010000000011234567890");
        System.out.println("NF-e: " + nfe.get("numeroNota"));
        
        // Listar
        Map<String, Object> lista = client.listarNFes(
            "12345678000195", 
            "2024-01-01", 
            "2024-12-31", 
            0
        );
        System.out.println("Total: " + lista.get("totalElements"));
    }
}
```

---

### C# (.NET)

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;
using System.Collections.Generic;

public class EReobotApiClient
{
    private const string API_BASE_URL = "https://api.ereobot.com.br";
    private readonly HttpClient _httpClient;
    private string _token;

    public EReobotApiClient()
    {
        _httpClient = new HttpClient();
    }

    // 1. Login
    public async Task<string> LoginAsync(string email, string senha)
    {
        var request = new
        {
            login = email,
            senha = senha
        };

        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _httpClient.PostAsync(
            $"{API_BASE_URL}/api/v1/login",
            content
        );

        response.EnsureSuccessStatusCode();

        var responseData = await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(
            await response.Content.ReadAsStreamAsync()
        );

        _token = responseData["token"].ToString();
        _httpClient.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", _token);

        Console.WriteLine("Login realizado com sucesso!");
        return _token;
    }

    // 2. Buscar NF-e
    public async Task<Dictionary<string, object>> BuscarNFeAsync(string chaveAcesso)
    {
        var response = await _httpClient.GetAsync(
            $"{API_BASE_URL}/api/v1/nfe/" + chaveAcesso
        );

        response.EnsureSuccessStatusCode();

        return await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(
            await response.Content.ReadAsStreamAsync()
        );
    }

    // 3. Listar NF-es
    public async Task<Dictionary<string, object>> ListarNFesAsync(
        string cnpjDestinatario,
        string dataInicio,
        string dataFim,
        int pagina = 0)
    {
        var url = $"{API_BASE_URL}/api/v1/nfe?" +
                  $"cnpjCpfDestinatario={cnpjDestinatario}" +
                  $"&dataEmissaoDe={dataInicio}" +
                  $"&dataEmissaoAte={dataFim}" +
                  $"&pageNum={pagina}";

        var response = await _httpClient.GetAsync(url);
        response.EnsureSuccessStatusCode();

        return await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(
            await response.Content.ReadAsStreamAsync()
        );
    }

    // 4. Consulta assíncrona
    public async Task<string> ConsultaAssincronaAsync(string cnpj, List<string> chaves)
    {
        var request = new
        {
            cnpjEmpresa = cnpj,
            chavesDeAcesso = chaves
        };

        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _httpClient.PostAsync(
            $"{API_BASE_URL}/api/v1/nfe/async",
            content
        );

        response.EnsureSuccessStatusCode();

        var responseData = await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(
            await response.Content.ReadAsStreamAsync()
        );

        var idExecucao = responseData["idExecucao"].ToString();
        Console.WriteLine($"Job criado: {idExecucao}");
        // Exemplo: NFE-a1b2c3d4-e5f6-7890-abcd-ef1234567890

        return idExecucao;
    }

    // 5. Verificar status
    public async Task<Dictionary<string, object>> VerificarStatusJobAsync(string idExecucao)
    {
        var response = await _httpClient.GetAsync(
            $"{API_BASE_URL}/api/v1/nfe/async/" + idExecucao
        );

        response.EnsureSuccessStatusCode();

        return await JsonSerializer.DeserializeAsync<Dictionary<string, object>>(
            await response.Content.ReadAsStreamAsync()
        );
    }

    // 6. Download
    public async Task<byte[]> DownloadJobAsync(string idExecucao)
    {
        var response = await _httpClient.GetAsync(
            $"{API_BASE_URL}/api/v1/nfe/async/" + idExecucao + "/download"
        );

        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsByteArrayAsync();
    }

    // Exemplo de uso
    public static async Task Main(string[] args)
    {
        var client = new EReobotApiClient();

        // Login
        await client.LoginAsync("seu.email@empresa.com", "SuaSenha123");

        // Buscar NF-e
        var nfe = await client.BuscarNFeAsync("35200812345678000195550010000000011234567890");
        Console.WriteLine($"NF-e: {nfe["numeroNota"]}");

        // Listar
        var lista = await client.ListarNFesAsync(
            "12345678000195",
            "2024-01-01",
            "2024-12-31",
            0
        );
        Console.WriteLine($"Total: {lista["totalElements"]}");
    }
}
```

---

## ✅ Boas Práticas

### 1. Gerenciamento de Token

- ✅ **Armazene o token de forma segura** (variáveis de ambiente, cofres de secrets)
- ✅ **Renove o token antes de expirar**
- ✅ **Não exponha o ‘token’ em ‘logs’ ou URLs**

### 2. Tratamento de Erros

```javascript
try {
  const response = await api.buscarNFe(chave);
} catch (error) {
  if (error.response) {
    // Erro de resposta do servidor
    switch (error.response.status) {
      case 401:
        console.log('Token inválido - fazer login novamente');
        break;
      case 403:
        console.log('Sem permissão para acessar este recurso');
        break;
      case 404:
        console.log('Recurso não encontrado');
        break;
      case 429:
        console.log('Limite de requisições excedido - aguardar');
        break;
      default:
        console.log('Erro desconhecido:', error.response.data);
    }
  } else if (error.request) {
    // Erro de rede
    console.log('Erro de conexão com a API');
  } else {
    console.log('Erro:', error.message);
  }
}
```

### 3. Paginação Eficiente

```javascript
async function buscarTodasNFes(filtros) {
  let pagina = 0;
  let todasNFes = [];
  let temMaisPaginas = true;
  
  while (temMaisPaginas) {
    const resultado = await api.listarNFes({
      ...filtros,
      pagina: pagina
    });
    
    todasNFes = todasNFes.concat(resultado.content);
    
    temMaisPaginas = !resultado.last;
    pagina++;
    
    // Evitar rate limiting
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return todasNFes;
}
```

### 4. Uso de Consultas Assíncronas

**Use quando:**
- Processar mais de 50 chaves de acesso
- Fazer downloads em lote
- Integração batch/agendada

**Não use quando:**
- Precisa de resposta imediata
- Poucas chaves (< 10)
- Interface de usuário interativa

**Armazenamento do ID de Execução:**

```javascript
// ✅ CORRETO - Armazenar o ID de execução
const jobResponse = await criarJobAsync(cnpj, chaves);
const idExecucao = jobResponse.data.idExecucao;

// Salvar em banco de dados para consulta posterior
await database.salvarJob({
  idExecucao: idExecucao,
  cnpj: cnpj,
  status: 'PENDING',
  criadoEm: new Date()
});

// ❌ INCORRETO - Não tentar "adivinhar" ou modificar o ID
// O ID é único e não segue padrão sequencial
```

**Consulta de Status:**

```javascript
// ✅ CORRETO - Consultar status periodicamente
async function aguardarConclusao(idExecucao, maxTentativas = 60) {
  for (let i = 0; i < maxTentativas; i++) {
    const status = await consultarStatus(idExecucao);
    
    if (status.status === 'COMPLETED') {
      return status;
    }
    
    if (status.status === 'FAILED') {
      throw new Error('Job falhou: ' + status.mensagemErro);
    }
    
    // Aguardar 5 segundos entre consultas
    await sleep(5000);
  }
  
  throw new Error('Timeout ao aguardar conclusão do job');
}
```

### 5. Validação de Dados

```javascript
function validarChaveAcesso(chave) {
  // Chave deve ter 44 dígitos
  const regex = /^\d{44}$/;
  return regex.test(chave);
}

function validarCNPJ(cnpj) {
  // Remove caracteres não numéricos
  cnpj = cnpj.replace(/\D/g, '');
  // CNPJ deve ter 14 dígitos
  return cnpj.length === 14;
}
```

### 6. Retry Logic

```javascript
async function requisicaoComRetry(fn, maxTentativas = 3) {
  for (let tentativa = 1; tentativa <= maxTentativas; tentativa++) {
    try {
      return await fn();
    } catch (error) {
      if (tentativa === maxTentativas) throw error;
      
      // Backoff exponencial
      const delay = Math.pow(2, tentativa) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Uso
const nfe = await requisicaoComRetry(() => 
  api.buscarNFe('35200812345678000195550010000000011234567890')
);
```

---

## ⏱️ Rate Limiting

A API implementa limite de requisições para garantir disponibilidade e performance.

### Limites

- **Requisições por minuto:** 60
- **Requisições por hora:** 1000
- **Download simultâneos:** 5

### Headers de Resposta

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1699876543
```

### Resposta de Limite Excedido (429)

```json
{
  "error": "Too Many Requests",
  "message": "Você excedeu o limite de 60 requisições por minuto",
  "retryAfter": 30
}
```

### Como Lidar

```javascript
async function requisicaoComRateLimit(fn) {
  try {
    return await fn();
  } catch (error) {
    if (error.response?.status === 429) {
      const retryAfter = error.response.data.retryAfter || 60;
      console.log(`Rate limit excedido. Aguardando ${retryAfter}s...`);
      
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      
      // Tentar novamente
      return await fn();
    }
    throw error;
  }
}
```

---

## 🔒 Segurança

### Recomendações

1. **Use HTTPS:** Todas as requisições devem usar HTTPS
2. **Não exponha credenciais:** Use variáveis de ambiente
3. **Rotacione senhas regularmente**
4. **Monitore acessos não autorizados**
5. **Implemente timeout em requisições**
6. **Valide certificados SSL**

### Exemplo de Configuração Segura

```javascript
// .env
API_EMAIL=seu.email@empresa.com
API_SENHA=senha_segura_aqui
API_BASE_URL=https://api.ereobot.com.br

// config.js
require('dotenv').config();

module.exports = {
  apiEmail: process.env.API_EMAIL,
  apiSenha: process.env.API_SENHA,
  apiBaseUrl: process.env.API_BASE_URL
};
```

---

## 📞 Suporte e Contato

### Canais de Suporte

- **Email:** XXXXXXXXXX
- **Telefone:** +55 (XX) XXXX-XXXX
- **Horário de Atendimento:** Segunda a Sexta, 9h às 18h (horário de Brasília)

### Documentação Adicional

- **Swagger UI:** `/swagger-ui/index.html`
- **OpenAPI Spec:** `/v3/api-docs`
- **Portal do Cliente:** https://app.ereobot.com.br/#/login

### Reportar Problemas

Ao reportar um problema, inclua:

1. Endpoint acessado
2. Timestamp da requisição
3. Mensagem de erro completa
4. Request/Response (sem dados sensíveis)
5. Seu email de login (não a senha)

### Changelog

**Versão 2.4.0** (19 de Março de 2026) - Sincronização com Swagger
- ✅ **Novos endpoints documentados**: NFS-e Nacional, Municípios Habilitados (NFS-e), Extrato de Captura e Assinaturas de Averbação (GET/POST/PATCH/DELETE)
- ✅ **Regra atualizada de Empresa**: atualização via `PATCH /api/v1/empresa/{cnpjCpf}`
- ✅ **Paginação atualizada**: uso de `pageNum` na documentação e nos snippets de integração
- ✅ **Modelos ampliados**: inclusão de `NFSeNacionalDTO`, `MunicipiosHabilitadosResponse`, `AssinaturaAverbacao` e `AverbacaoEmpresasCountDTO`

**Versão 2.3.0** (03 de Março de 2026) - Averbação de Exportação
- ✅ **Averbação de Exportação**: suporte completo a consulta por chave, listagem paginada, consulta assíncrona, download em lote e webhook
- ✅ **Documentação unificada**: todos os endpoints em um único Swagger

**Versão 2.2.0** (15 de Novembro de 2025) - Lançamento Inicial
- ✅ **API RESTful completa** para consulta de documentos fiscais eletrônicos
- ✅ **Autenticação JWT** com segurança baseada em tokens Bearer
- ✅ **Consultas síncronas** para NF-e, CT-e, NFS-e e NFC-e
- ✅ **Consultas assíncronas** para processamento em background de múltiplos documentos
- ✅ **Sistema de download em lote** com arquivos compactados
- ✅ **Paginação avançada** com filtros por data, CNPJ, número de nota
- ✅ **Rate limiting** para garantir disponibilidade e performance
- ✅ **Identificadores seguros** no formato `NFE-{UUID}` para jobs assíncronos

---

## 📄 Licença e Termos de Uso

O uso desta API está sujeito aos termos de contrato estabelecidos com a e-Reobot.

- ✅ Uso permitido apenas para fins contratuais
- ❌ Proibido revenda ou redistribuição de dados
- ❌ Proibido uso abusivo ou tentativas de invasão
- ✅ Dados devem ser armazenados de forma segura
- ✅ Conformidade com LGPD obrigatória

---

## 📚 Glossário

| Termo | Definição |
|-------|-----------|
| **NF-e** | Nota Fiscal Eletrônica - documento fiscal digital de produtos |
| **CT-e** | Conhecimento de Transporte Eletrônico - documento de transporte de cargas |
| **NFS-e** | Nota Fiscal de Serviço Eletrônica - documento fiscal de serviços |
| **NFC-e** | Nota Fiscal do Consumidor Eletrônica - cupom fiscal eletrônico |
| **Chave de Acesso** | Código de 44 dígitos que identifica unicamente um documento fiscal |
| **CNPJ** | Cadastro Nacional de Pessoa Jurídica (14 dígitos) |
| **CPF** | Cadastro de Pessoa Física (11 dígitos) |
| **JWT** | JSON Web Token - token de autenticação |
| **GZIP** | Formato de compactação de arquivos |
| **Rate Limit** | Limite de requisições por período |

---

**Última atualização:** 19 de Março de 2026
**Versão da Documentação:** 2.4.0
**e-Reobot - Automação de Documentos Fiscais** ©️ 2026
