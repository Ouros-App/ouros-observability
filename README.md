# Ouros Observability

<!-- REPO-METADATA:START -->
<div align="center">

[![Repo Size](https://img.shields.io/github/repo-size/Ouros-App/ouros-observability?style=flat-square&label=REPO%20SIZE)](https://github.com/Ouros-App/ouros-observability)
[![Languages](https://img.shields.io/github/languages/count/Ouros-App/ouros-observability?style=flat-square&label=LANGUAGES)](https://github.com/Ouros-App/ouros-observability/languages)
[![Forks](https://img.shields.io/github/forks/Ouros-App/ouros-observability?style=flat-square&label=FORKS)](https://github.com/Ouros-App/ouros-observability/network/members)
[![Issues](https://img.shields.io/github/issues/Ouros-App/ouros-observability?style=flat-square&label=ISSUES)](https://github.com/Ouros-App/ouros-observability/issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/Ouros-App/ouros-observability?style=flat-square&label=PULL%20REQUESTS)](https://github.com/Ouros-App/ouros-observability/pulls)

</div>
<!-- REPO-METADATA:END -->

Infraestrutura central de observabilidade do Ouros. Este repositório concentra a configuração do Prometheus e, futuramente, exporters, regras e demais recursos necessários para coletar e disponibilizar métricas operacionais dos serviços da plataforma.

## Status e escopo

O repositório está em fase inicial de bootstrap. A implementação da stack de observabilidade ainda será adicionada.

O escopo planejado inclui:

- Prometheus executado no homelab do Ouros;
- coleta periódica dos endpoints `/metrics` dos microserviços;
- autenticação machine-to-machine via Keycloak para endpoints protegidos;
- retenção local de séries temporais;
- regras e queries PromQL versionadas;
- integração do Prometheus como fonte de dados do `ms-telemetry-dashboard-service`;
- exporters de infraestrutura quando necessários, como Node Exporter e PostgreSQL Exporter;
- configuração reproduzível por Docker Compose e arquivos versionados.

Grafana não é requisito para o produto. O `ms-telemetry-dashboard-service` poderá consultar a API HTTP do Prometheus e renderizar os dashboards consumidos pelos clientes Ouros. Grafana pode ser adicionado futuramente como ferramenta interna de engenharia.

## Arquitetura planejada

```text
ms-spring-api ------------------\
ms-auth-service -----------------\
ms-ai-server ---------------------+--> Prometheus (homelab)
ms-mcp-server-ouros-knowledge ---/          |
ms-telemetry-dashboard-service -/           |
Keycloak / exporters -----------/            |
                                             v
                              ms-telemetry-dashboard-service
                                             |
                                   Chart.js / PNG / JSON
                                             |
                                      Web / Mobile
```

O Prometheus atua como coletor e banco de séries temporais. Os serviços expõem métricas e o Prometheus realiza o scrape em intervalos definidos.

## Autenticação

A estratégia planejada para endpoints protegidos é usar uma identidade própria de infraestrutura no Keycloak.

```text
Prometheus
    |
    | OAuth2 Client Credentials
    v
Keycloak
    |
    | JWT de service account
    v
/metrics
```

O client de infraestrutura deve ter apenas as permissões necessárias para leitura de métricas. Credenciais de usuários finais não devem ser usadas para scraping.

A declaração do client e das permissões permanece responsabilidade do repositório `ouros-keycloak`. Este repositório deve armazenar apenas a configuração necessária para o Prometheus consumir essa identidade em runtime, sem versionar secrets.

## Estrutura planejada

```text
.
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
├── secrets/
│   └── .gitkeep
├── scripts/
├── .env.example
└── README.md
```

A estrutura pode evoluir conforme novos exporters e integrações forem adicionados.

## Recursos previstos

A configuração inicial será dimensionada para o ambiente atual do Ouros:

| Recurso | Configuração inicial |
| --- | --- |
| Memória | 1 GB |
| CPU | até 1 vCPU |
| Volume | 8 GB |
| Retenção lógica | até 6 GB |
| Retenção temporal | 30 dias |
| Scrape interval | 15 s |

Os limites devem ser revistos conforme a quantidade de séries ativas e a cardinalidade das labels crescerem.

## Integração com dashboards

O `ms-telemetry-dashboard-service` já possui abstrações para descoberta de dashboards, execução de consultas e renderização em Chart.js ou PNG.

A evolução planejada é adicionar um provider Prometheus ao lado do provider Databricks:

```text
DashboardService
    |
    +--> DatabricksDashboardProvider
    |
    +--> PrometheusDashboardProvider
             |
             +--> /api/v1/query
             +--> /api/v1/query_range
```

Isso permite combinar métricas operacionais do Prometheus com dados analíticos do Databricks sem exigir Grafana no caminho do usuário final.

## Segurança

- Nunca versionar client secrets, tokens ou credenciais do Keycloak.
- Não expor a porta administrativa do Prometheus diretamente à internet.
- Preferir acesso privado entre o `ms-telemetry-dashboard-service` e o homelab.
- Evitar labels de alta cardinalidade como `user_id`, `thread_id`, `request_id`, e-mail ou conteúdo de prompts.
- Manter as permissões do service account de observabilidade com privilégio mínimo.

## Execução

A stack ainda não foi implementada neste repositório.

Quando o bootstrap inicial estiver disponível, a execução local será documentada aqui e deverá permanecer reproduzível por Docker Compose.

## Testes e qualidade

A CI do repositório deve validar, conforme os arquivos forem adicionados:

- sintaxe e consistência do Docker Compose;
- configuração do Prometheus com `promtool`;
- regras PromQL versionadas;
- ausência de secrets acidentalmente versionados.

## Licença

Este projeto está sob a licença MIT, conforme o arquivo [LICENSE](LICENSE).

## Principais contribuidores

<!-- CONTRIBUTORS:START -->
- Nenhum contribuidor individual identificado ainda.
<!-- CONTRIBUTORS:END -->

> Atualizado automaticamente semanalmente pelo workflow de metadados do README.
