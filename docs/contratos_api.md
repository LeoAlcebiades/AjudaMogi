# Contratos de Integração da API REST

Este documento define os padrões arquiteturais, especificações de endpoints, estruturas JSON de requisição e resposta, e políticas de segurança da API RESTful da plataforma **AjudaMogi**, em conformidade com o Relatório Técnico-Científico do Trabalho de Graduação (FATEC Mogi das Cruzes, 2026).

---

## 1. Diretrizes Globais da API

* **Estilo Arquitetural:** REST (*Representational State Transfer*), sem estado (*stateless*).
* **Protocolo de Transporte:** Exclusivamente **HTTPS com TLS 1.3** (RNF07).
* **Formato de Troca de Dados:** JSON (*JavaScript Object Notation*, RFC 8259), com codificação UTF-8, exceto para submissão de arquivos binários de mídia que utilizam `multipart/form-data`.
* **Autenticação:** Padrão **Bearer Token** baseado em **JSON Web Token (JWT)** no cabeçalho `Authorization`:
  ```http
  Authorization: Bearer <TOKEN_JWT>
  ```
* **Controle de Vazão (Rate Limiting):** A API aplica limitação de requisições por IP e token de sessão. Quando excedido, responde `HTTP 429 Too Many Requests` com os cabeçalhos:
  * `X-RateLimit-Limit`: Quantidade máxima permitida na janela.
  * `X-RateLimit-Remaining`: Requisições restantes na janela atual.
  * `X-RateLimit-Reset`: Tempo em segundos UTC até a renovação da cota.

---

## 2. Endpoints Centrais da Especificação

### 2.1. Submissão de Denúncia (Cidadão / Reportador)

Recebe os dados coletados em campo pelo aplicativo móvel nativo, dispara a verificação espacial de proximidade, persiste a ocorrência e agenda as notificações assíncronas via mensageria.

* **Rota:** `POST /api/v1/denuncias`
* **Autenticação:** Obrigatória (Bearer Token / JWT do Reportador)
* **Formato do Payload:** `multipart/form-data` (recomendado para transmissão eficiente do binário comprimido) ou `application/json` (com imagem em base64 pré-validada).

#### Estrutura Lógica da Requisição (Quadro 5 do TG)
```json
{
  "categoria_id": 4,
  "descricao_livre": "Buraco profundo na faixa da direita, causando danos aos pneus.",
  "coordenadas": {
    "latitude": -23.550520,
    "longitude": -46.633308
  },
  "foto_base64_ou_multipart": "<ARQUIVO_BINARIO_COMPRIMIDO>"
}
```

#### Regras de Validação no Endpoint:
1. **Fotografia Obrigatória (RN01):** Rejeita com `HTTP 400` se nenhum binário de mídia for submetido.
2. **Tamanho Máximo (RNF02):** A imagem deve ter sido previamente comprimida pelo app móvel para no máximo **2 MB**. O gateway corta payloads que excedam esse limite.
3. **Remoção de EXIF:** O aplicativo deve realizar a expurgação das tags EXIF (*EXIF stripping*) antes do envio. A API valida a higienização dos metadados antes de direcionar a foto ao *Object Storage* (S3/MinIO).
4. **Coordenadas Válidas:** `latitude` (-90.0 a 90.0) e `longitude` (-180.0 a 180.0) devem ser numéricas.

#### Resposta de Sucesso (`HTTP 201 Created` - Quadro 6 do TG)
```json
{
  "sucesso": true,
  "mensagem": "Denúncia registrada e encaminhada para análise.",
  "dados": {
    "protocolo_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "status": "PENDENTE",
    "criado_em": "2026-09-29T14:17:05Z"
  }
}
```

#### Comportamento Interno Desencadeado:
* **Upload S3:** Arquivo é armazenado no *Object Storage* gerando uma URL segura.
* **Geocodificação Reversa:** Conversão das coordenadas em logradouro e bairro (protegido por *Circuit Breaker*).
* **Deduplicação Espacial (RN05 / RN07):** Consulta `ST_DWithin` no PostGIS para checar se há denúncias ativas da mesma categoria num raio de 30m:
  * *Distância <= 30m:* Rotulado como `Reforço` (`denuncia_pai_id` preenchido, prioridade `BAIXA`).
  * *Distância > 30m:* Rotulado como `Novo Problema` (`denuncia_pai_id` nulo, prioridade `ALTA`).
* **Mensageria BullMQ:** Publicação desacoplada do evento `denuncia_submetida` para que o worker dispare a notificação FCM em segundo plano, liberando a API em menos de 2 segundos (RNF05).

---

### 2.2. Consulta de Mapa de Calor (Visualização Pública)

Endpoint de alta performance projetado para renderizar o adensamento térmico das ocorrências validadas na cidade sem retornar coordenadas brutas pontuais, poupando a largura de banda móvel e protegendo o banco relacional.

* **Rota:** `GET /api/v1/mapa/calor`
* **Autenticação:** Obrigatória (Bearer Token / JWT)
* **Parâmetros de Busca (Query Parameters - Opcionais):**
  * `categoria_id` (inteiro): Filtra anomalias por categoria específica (ex: apenas iluminação ou buracos).
  * `raio_km` (numérico): Delimita a amplitude geográfica a partir do centro da visualização.

#### Exemplo de Chamada:
```http
GET /api/v1/mapa/calor?categoria_id=4&raio_km=5 HTTP/1.1
Host: api.ajudamogi.sp.gov.br
Authorization: Bearer <TOKEN_JWT>
```

#### Resposta de Sucesso (`HTTP 200 OK` - Quadro 7 do TG)
```json
{
  "sucesso": true,
  "dados": {
    "resolucao_h3": 9,
    "clusters": [
      {
        "h3_index": "89283082803ffff",
        "peso": 15,
        "nivel_gravidade": "ALTO"
      },
      {
        "h3_index": "89283082807ffff",
        "peso": 3,
        "nivel_gravidade": "BAIXO"
      }
    ]
  }
}
```

#### Mecanismos de Otimização e Cache:
* **Filtro Estrito (RN03 / RN04):** Somente denúncias com status `VALIDADA` compõem a métrica `peso` dos hexágonos H3. Ocorrências `PENDENTE` ou `REJEITADA` são sumariamente desconsideradas.
* **Cache em Memória (Redis):** A resposta agregada é armazenada no Redis com TTL de **5 a 15 minutos** (*Cache-Aside*).
* **Invalidação Reativa:** Eventos de transição de estado (`PENDENTE` -> `VALIDADA` ou `VALIDADA` -> `RESOLVIDA`) no painel administrativo acionam a invalidação imediata da chave no Redis via fila assíncrona.

---

## 3. Endpoints Complementares do Ecossistema

Para cobrir a integralidade dos requisitos funcionais previstos no DRS (RF01 a RF09) e a conformidade com a LGPD, a arquitetura padroniza os seguintes contratos complementares:

### 3.1. Autenticação e Gestão de Acesso

#### Criar Conta de Cidadão / Agente de Limpeza (RF01)
* **Rota:** `POST /api/v1/auth/register`
* **Corpo da Requisição:**
  ```json
  {
    "nome": "Cidadão Mogi",
    "email": "cidadao@exemplo.com.br",
    "senha": "SenhaForteSegura#2026"
  }
  ```
* **Resposta (`HTTP 201 Created`):**
  ```json
  {
    "sucesso": true,
    "mensagem": "Usuário cadastrado com sucesso.",
    "dados": {
      "usuario_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "email": "cidadao@exemplo.com.br"
    }
  }
  ```

#### Login e Obtenção de Token JWT (RF01 / RF07)
* **Rota:** `POST /api/v1/auth/login`
* **Corpo da Requisição:**
  ```json
  {
    "email": "cidadao@exemplo.com.br",
    "senha": "SenhaForteSegura#2026"
  }
  ```
* **Resposta (`HTTP 200 OK`):**
  ```json
  {
    "sucesso": true,
    "dados": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "tipo": "Bearer",
      "expira_em": 86400,
      "perfil": "REPORTADOR"
    }
  }
  ```

---

### 3.2. Histórico Pessoal do Munícipe (RF05 / LGPD)

* **Rota:** `GET /api/v1/denuncias/minhas`
* **Autenticação:** Obrigatória (Bearer Token / JWT do cidadão).
* **Segurança:** A API extrai o `UUID` do usuário contido no token e restringe a busca unicamente às ocorrências de sua autoria.
* **Resposta (`HTTP 200 OK`):**
  ```json
  {
    "sucesso": true,
    "dados": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "categoria": "Buraco",
        "logradouro": "Avenida Vereador Narciso Yague Guimarães",
        "bairro": "Centro Cívico",
        "status": "VALIDADA",
        "foto_url": "https://s3.amazonaws.com/ajudamogi-evidencias/f47ac10b.jpg",
        "criado_em": "2026-09-29T14:17:05Z",
        "atualizado_em": "2026-09-29T15:30:00Z"
      }
    ]
  }
  ```

---

### 3.3. Painel Administrativo de Triagem (RF08 / RF09 / Payload Masking)

#### Listagem da Fila de Triagem (RF08)
* **Rota:** `GET /api/v1/admin/denuncias/triagem`
* **Autenticação:** Obrigatória (Perfil `ADMIN`).
* **Regra de Mascaramento (LGPD):** O `usuario_id`, e-mail e nome do cidadão são estritamente omitidos da resposta para preservar o anonimato perante o servidor público.
* **Ordenação:** Prioridade `ALTA` (Novos Problemas) precede prioridade `BAIXA` (Reforços).
* **Resposta (`HTTP 200 OK`):**
  ```json
  {
    "sucesso": true,
    "dados": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "categoria": "Buraco",
        "descricao_livre": "Buraco profundo na faixa da direita",
        "foto_url": "https://s3.amazonaws.com/ajudamogi-evidencias/f47ac10b.jpg",
        "latitude": -23.550520,
        "longitude": -46.633308,
        "logradouro": "Av. Vereador Narciso Yague Guimarães",
        "bairro": "Centro Cívico",
        "prioridade": "ALTA",
        "rotulo": "Novo Problema",
        "criado_em": "2026-09-29T14:17:05Z"
      }
    ]
  }
  ```

#### Transição de Status / Triagem Humana (RF09)
* **Rota:** `PATCH /api/v1/admin/denuncias/{id}/status`
* **Autenticação:** Obrigatória (Perfil `ADMIN`).
* **Corpo da Requisição:**
  ```json
  {
    "novo_status": "VALIDADA",
    "categoria_id": 4,
    "justificativa": "Evidência confirmada em via pública de alto fluxo."
  }
  ```
* **Valores Permitidos para `novo_status`:** `"VALIDADA"`, `"REJEITADA"`, `"RESOLVIDA"`.
* **Resposta (`HTTP 200 OK`):**
  ```json
  {
    "sucesso": true,
    "mensagem": "Status da ocorrência atualizado para VALIDADA com sucesso.",
    "dados": {
      "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "status": "VALIDADA",
      "atualizado_em": "2026-09-29T15:30:00Z"
    }
  }
  ```
* **Efeitos Colaterais no Servidor:**
  1. Invalida cache do Redis para o mapa de calor.
  2. Publica evento no BullMQ para envio de push notification ao munícipe (FCM).

---

### 3.4. Marcadores do Mapa Interativo (RF03)

Retorna as ocorrências pontuais individuais aprovadas que configuram anomalias ativas (excluindo os reforços duplicados, que apenas agregam no mapa de calor).

* **Rota:** `GET /api/v1/mapa/ocorrencias`
* **Autenticação:** Obrigatória (Bearer Token).
* **Filtros de Query:** `?min_lat=-23.6&max_lat=-23.4&min_lng=-46.7&max_lng=-46.5`
* **Resposta (`HTTP 200 OK`):**
  ```json
  {
    "sucesso": true,
    "dados": [
      {
        "id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
        "categoria_id": 4,
        "categoria_nome": "Buraco",
        "latitude": -23.550520,
        "longitude": -46.633308,
        "quantidade_reforcos": 3,
        "criado_em": "2026-09-29T14:17:05Z"
      }
    ]
  }
  ```

---

## 4. Padrão de Respostas de Erro

Todas as respostas de falha da API seguem um formato consistente e legível por máquina:

```json
{
  "sucesso": false,
  "codigo_erro": "EVIDENCIA_OBRIGATORIA",
  "mensagem": "É estritamente proibido o registro de uma denúncia sem o anexo de fotografia capturada.",
  "detalhes": {
    "campo": "foto_base64_ou_multipart",
    "regra": "RN01"
  }
}
```

### Matriz de Códigos de Status HTTP:
| Código | Significado | Aplicação |
| :--- | :--- | :--- |
| **`200 OK`** | Sucesso | Consultas (mapa de calor, histórico) e atualizações com retorno. |
| **`201 Created`** | Criado | Submissão de denúncia ou criação de novo usuário. |
| **`400 Bad Request`** | Erro de Validação | Violação de regras de negócio (sem foto, coordenadas ausentes, tamanho > 2 MB). |
| **`401 Unauthorized`** | Não Autenticado | Ausência de Bearer Token JWT ou token expirado/inválido. |
| **`403 Forbidden`** | Não Autorizado | Cidadão tentando acessar rotas administrativas (`/admin/*`). |
| **`404 Not Found`** | Não Encontrado | Ocorrência, usuário ou rota inexistente. |
| **`429 Too Many Requests`** | Cota Excedida | Violação dos limites estritos de *Rate Limiting* por IP ou conta. |
| **`500 Internal Server Error`** | Falha de Servidor | Falha não tratada da aplicação. |
| **`503 Service Unavailable`** | Indisponibilidade | Disjuntor (*Circuit Breaker*) aberto em dependências críticas essenciais. |
