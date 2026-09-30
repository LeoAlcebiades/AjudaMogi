# Glossário e Lista de Siglas Técnicas

Este documento reúne o glossário terminológico e a lista de siglas e abreviações da plataforma **AjudaMogi**, extraídos e consolidados a partir do Relatório Técnico-Científico do Trabalho de Graduação (FATEC Mogi das Cruzes, 2026).

---

## 1. Glossário de Termos

* **Agente Administrativo:** Perfil de usuário da equipe de zeladoria urbana que acessa o painel web seguro para validar, rejeitar ou recategorizar as denúncias submetidas.
* **Back-end:** Parte da aplicação executada no servidor, responsável pelas regras de negócio, pelo acesso ao banco de dados e pela integração com serviços de terceiros.
* **Bearer Token:** Token de acesso que concede autorização aos recursos protegidos a qualquer entidade portadora de sua credencial válida.
* **Cache:** Mecanismo de armazenamento temporário de dados em memória de acesso ultra-rápido, utilizado para acelerar consultas repetidas e preservar recursos do banco principal.
* **Circuit Breaker:** Padrão de resiliência de software que monitora chamadas a serviços remotos e interrompe temporariamente novas requisições em caso de taxa elevada de falhas, prevenindo falhas em cascata no ecossistema.
* **Deduplicação Espacial:** Mecanismo algorítmico que identifica ocorrências da mesma categoria registradas a uma distância geográfica predeterminada (raio de 30 metros), agrupando registros redundantes como reforços de prioridade.
* **Denúncia:** Registro formal de anomalia ou problema de infraestrutura urbana efetuado por um Reportador, contemplando categoria, evidência fotográfica obrigatória e coordenadas de GPS.
* **Endpoint:** Combinação específica de método HTTP e rota exposta pela API REST para execução de uma operação de negócio.
* **EXIF Stripping:** Técnica de privacidade aplicada no lado do cliente que expurga metadados Exchangeable Image File Format de uma imagem antes do despacho ao servidor, suprimindo detalhes de hardware, fabricante, modelo e lentes.
* **Geocodificação Reversa:** Processo computacional de converter coordenadas geográficas brutas (latitude e longitude) em uma descrição textual de endereço urbano (logradouro, bairro, cidade).
* **H3 (Hexagonal Hierarchical Spatial Index):** Sistema de grade geoespacial hierárquica e global baseado em polígonos hexagonais, desenvolvido pela Uber para particionamento do globo e agregações térmicas.
* **Mapa de Calor (*Heat Map*):** Representação cartográfica temática na qual a densidade, gravidade e volume de fenômenos são destacados através de variações gradativas em uma escala de cores térmicas.
* **Message Broker:** Componente intermediário de mensageria que recebe eventos de componentes produtores e os encaminha a filas de consumidores (*workers*), viabilizando processamento assíncrono.
* **Novo Problema:** Rótulo sistêmico concedido a uma denúncia que não possui anomalias ativas da mesma categoria em um raio de 30 metros; recebe prioridade alta (urgência) na esteira de triagem.
* **Object Storage:** Modelo de armazenamento em nuvem voltado a arquivos binários não estruturados (ex: fotos) persistidos como objetos dentro de contêineres (*buckets*), como AWS S3 ou MinIO.
* **Offline-First:** Paradigma de desenvolvimento de software em que a aplicação armazena seu estado em banco local nativo e opera normalmente sem acesso à internet, sincronizando os dados em segundo plano assim que a conectividade for restabelecida.
* **Payload:** Conjunto de dados transportado no corpo (*body*) de uma requisição ou resposta de rede.
* **Payload Masking:** Técnica de proteção de dados que suprime ou ofusca campos confidenciais da carga de dados enviada a determinadas interfaces (ex: mascaramento de dados civis e identificadores de cidadãos no painel da prefeitura).
* **PostGIS:** Extensão espacial avançada para o banco relacional PostgreSQL, que habilita tipos de dados geográficos e primitivas analíticas espaciais indexadas.
* **Rate Limiting:** Política de proteção e controle de fluxo que restringe o volume de requisições que um determinado cliente ou endereço IP pode submeter em uma janela temporal definida.
* **Reforço:** Rótulo atribuído a uma denúncia registrada em um raio de até 30 metros de uma ocorrência ativa preexistente da mesma categoria; recebe prioridade baixa na triagem e incrementa o peso térmico da respectiva célula H3 quando validada.
* **Reportador:** Perfil de usuário (munícipe ou agente de limpeza pública) responsável por coletar e submeter problemas urbanos em campo via aplicativo móvel nativo.
* **Token:** Credencial computacional codificada e assinada criptograficamente, concedida ao cliente após autenticação para identificação de sessões.
* **TTL (*Time to Live*):** Janela temporal de validade de um determinado registro mantido em cache, após a qual o item é compulsoriamente descartado.
* **Worker:** Processo ou serviço operando em segundo plano (*background*) encarregado de processar tarefas demoradas retiradas de filas assíncronas sem reter a resposta da API principal.
* **Zeladoria Urbana:** Esfera de serviços públicos municipais dedicada à preservação, manutenção, conservação e reparação da infraestrutura das cidades (pavimentação, iluminação, drenagem, limpeza e podas).

---

## 2. Lista de Siglas e Abreviações

| Sigla | Significado em Português | Nome Original / Contexto Técnico |
| :--- | :--- | :--- |
| **API** | Interface de Programação de Aplicações | *Application Programming Interface* |
| **AWS** | Serviços Web da Amazon | *Amazon Web Services* |
| **CIPA** | Associação de Produtos de Câmera e Imagem | *Camera & Imaging Products Association* |
| **CPF** | Cadastro de Pessoas Físicas | Documento de identificação fiscal individual do Brasil |
| **DDoS** | Negação de Serviço Distribuída | *Distributed Denial of Service* |
| **DER** | Diagrama Entidade-Relacionamento | Modelagem conceitual e lógica de banco de dados |
| **DRS** | Documento de Requisitos de Software | Especificação normativa de requisitos do sistema |
| **EXIF** | Formato de Arquivo de Imagens Intercambiáveis | *Exchangeable Image File Format* |
| **FATEC** | Faculdade de Tecnologia de Mogi das Cruzes | Instituição de ensino superior de tecnologia |
| **FCM** | Mensageria em Nuvem do Firebase | *Firebase Cloud Messaging* |
| **GPS** | Sistema de Posicionamento Global | *Global Positioning System* |
| **HEIC** | Formato de Imagem de Alta Eficiência | *High Efficiency Image Container* |
| **HTTP** | Protocolo de Transferência de Hipertexto | *Hypertext Transfer Protocol* |
| **HTTPS** | Protocolo Seguro de Transferência de Hipertexto | *Hypertext Transfer Protocol Secure* |
| **IP** | Protocolo de Internet | *Internet Protocol* |
| **JPEG** | Grupo Conjunto de Especialistas Fotográficos | *Joint Photographic Experts Group* |
| **JSON** | Notação de Objetos JavaScript | *JavaScript Object Notation* |
| **JWT** | Token Web JSON | *JSON Web Token* |
| **LGPD** | Lei Geral de Proteção de Dados Pessoais | Lei Federal nº 13.709/2018 |
| **MVP** | Produto Mínimo Viável | *Minimum Viable Product* |
| **OWASP** | Projeto Aberto de Segurança em Aplicações Web | *Open Worldwide Application Security Project* |
| **REST** | Transferência de Estado Representacional | *Representational State Transfer* |
| **RF** | Requisito Funcional | Identificador de requisito de comportamento do sistema |
| **RN** | Regra de Negócio | Identificador de premissa normativa de domínio |
| **RNF** | Requisito Não Funcional | Identificador de restrição de qualidade ou arquitetura |
| **S3** | Serviço de Armazenamento Simples | *Simple Storage Service* (AWS) |
| **SRID** | Identificador de Sistema de Referência Espacial | *Spatial Reference System Identifier* |
| **TG** | Trabalho de Graduação | Monografia / Relatório Técnico-Científico acadêmico |
| **TLS** | Segurança da Camada de Transporte | *Transport Layer Security* |
| **TTL** | Tempo de Vida | *Time to Live* |
| **UML** | Linguagem de Modelagem Unificada | *Unified Modeling Language* |
| **URL** | Localizador Uniforme de Recursos | *Uniform Resource Locator* |
| **UUID** | Identificador Único Universal | *Universally Unique Identifier* |
| **WGS 84** | Sistema Geodésico Mundial de 1984 | *World Geodetic System 1984* (SRID 4326) |
