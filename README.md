# AjudaMogi

Sistema distribuído de zeladoria urbana baseado em geolocalização e mapas de calor, permitindo que cidadãos e agentes reportem anomalias de infraestrutura em tempo real.

## Visão Geral
A plataforma é composta por aplicativos nativos (iOS/Android) para coleta de dados em campo, um painel administrativo web para triagem e uma API REST assíncrona. O sistema utiliza indexação espacial hexagonal (H3) e deduplicação espacial para gerar mapas de calor otimizados.

## Stack Tecnológica
* **Mobile:** Kotlin (Android) / Swift (iOS)
* **Back-end:** API RESTful (Node.js/Python a definir)
* **Persistência Relacional:** PostgreSQL + PostGIS
* **Cache:** Redis
* **Mensageria (Broker):** BullMQ
* **Storage:** AWS S3 (ou MinIO local)
* **Notificações:** Firebase Cloud Messaging (FCM)

## Documentação de Engenharia
A documentação arquitetural e as regras de negócio estão versionadas como código na pasta `/docs`. Consulte os links abaixo para detalhes técnicos:

* [Arquitetura do Sistema e Modelagem de Dados](docs/arquitetura.md)
* [Regras de Negócio e Máquina de Estados](docs/regras_de_negocio.md)
* [Contratos da API REST](docs/contratos_api.md)
* [Glossário e Siglas Técnicas](docs/glossario.md)

## Como Executar Localmente
*(Esta seção será preenchida futuramente com os comandos do Docker para subir o banco de dados e a API).*