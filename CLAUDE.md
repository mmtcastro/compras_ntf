# Projeto de Migracao TDec - HCL Domino para Java/Vaadin

## Visao Geral

Este workspace contem as bases de dados HCL Domino/Notes (NSF) da TDec, exportadas como ODP (On-Disk Project), que serao migradas para uma aplicacao moderna baseada em Java e Vaadin.

A TDec (TDec Redes de Computadores) e uma empresa brasileira de tecnologia e servicos de TI que utiliza um ERP customizado em Lotus Notes/Domino com multiplos modulos integrados.

## Bases de Dados e Seus Dominios

### Core Business
| Repositorio | Modulo | Descricao |
|---|---|---|
| **sales_ntf** | Vendas (CRM) | Negocios, propostas, forecast, comissoes, margens, cambio. 166 views, 47 agentes |
| **empresas_ntf** | Cadastro (MDM) | Empresas, contatos, grupos economicos. Fonte autoritativa de dados mestres |
| **servicos_ntf** | Servicos | Chamados, contratos, projetos, SLA, tarifacao, tecnicos |
| **compras_ntf** | Compras | Pedidos, cotacoes, baixas, NFe XML, compliance tributario |

### Financeiro
| Repositorio | Modulo | Descricao |
|---|---|---|
| **caixa_ntf** | Financeiro | Despesas, cheques, borderos, conciliacao bancaria (Bradesco, Santander, PIX) |
| **estoquemain_ntf** | Estoque | Catalogo, movimentacoes, remessas, RMA, custos com impostos |
| **listas_ntf** | Listas | Itens, servicos, planilhas, fretes, intermediacoes vinculadas a negocios |

### RH e Gestao
| Repositorio | Modulo | Descricao |
|---|---|---|
| **adm_ntf** | Administracao/RH | Colaboradores, folha, ferias, requisicoes, contratos, ASOs |
| **aprimoramento_ntf** | Desenvolvimento | Treinamentos, certificacoes, PDI, premiacoes |

### Plataforma
| Repositorio | Modulo | Descricao |
|---|---|---|
| **intra_ntf** | Intranet | Portal unificado com todos os modulos, 394 XPages, APIs REST, Zabbix, RD Station |
| **bi_ntf** | BI | Business Intelligence, resumos diarios, dashboards, indicadores |

## Mapa de Integracoes Entre Bases

```
                    ┌──────────┐
                    │  intra   │ (Portal Unificado)
                    │  _ntf    │
                    └────┬─────┘
                         │ agrega todos os modulos
    ┌────────┬───────┬───┴───┬────────┬──────────┐
    │        │       │       │        │          │
┌───▼──┐ ┌──▼───┐ ┌─▼──┐ ┌──▼──┐ ┌───▼───┐ ┌───▼────┐
│sales │ │servi-│ │adm │ │caixa│ │aprimo-│ │  bi    │
│_ntf  │ │cos   │ │_ntf│ │_ntf │ │ramento│ │ _ntf   │
└──┬───┘ └──┬───┘ └─┬──┘ └──┬──┘ └───────┘ └───┬────┘
   │        │       │       │                   │
   │   ┌────┴───────┴───────┘                   │
   │   │                                        │
┌──▼───▼──┐  ┌─────────┐  ┌────────┐           │
│empresas │  │compras  │  │estoque │           │
│  _ntf   │  │ _ntf    │  │main_ntf│    consulta todas
└────┬────┘  └────┬────┘  └───┬────┘    as bases ───┘
     │            │           │
     └────────────┼───────────┘
                  │
            ┌─────▼─────┐
            │ listas_ntf│
            └───────────┘
```

### Dependencias Principais
- **empresas_ntf** -> sales_ntf, compras_ntf, faturas.nsf, importacao.nsf (validacao referencial)
- **sales_ntf** -> TimeSheet.nsf, servicos_ntf (validacao), compras_ntf (pedidos)
- **compras_ntf** -> sales_ntf (cambio), empresas_ntf (excecoes tributarias), listas_ntf
- **caixa_ntf** -> compras_ntf (baixas), faturas.nsf (recebiveis), adm_ntf (feriados)
- **estoquemain_ntf** -> sales_ntf, adm_ntf, compras_ntf, faturas.nsf, listas_ntf
- **bi_ntf** -> sales_ntf, servicos_ntf, empresas_ntf, faturas.nsf (todas como leitura)
- **intra_ntf** -> todas as bases (portal unificado)

## Padroes do Sistema Legado

### Convencoes de Nomenclatura Notes
- **Forms** (.form) - Definem entidades de dados (equivalente a tabelas)
- **Views** (.view) - Indices e consultas pre-definidas
- **Agents** (.lsa/.ja/.fa/.aa) - Logica de negocio automatizada
- **Script Libraries** (.lss/.javalib/.jss) - Codigo compartilhado
- **XPages** (.xsp) - Interface web moderna (JSF-based)
- **Custom Controls** (.xsp) - Componentes reutilizaveis de UI

### Prefixo _intra em Views
Views com prefixo `_intra` sao views internas de suporte:
- `_intraCodigos` - Indice por codigo
- `_intraStatus` - Indice por status
- `_intraForms` - Indice por tipo de formulario
- `_intraResponsaveis` - Indice por responsavel
- `_intraCriacao` - Indice por data de criacao

### Padrao de Identificacao
- Todas as bases usam `GeraJavaId` para gerar IDs unicos no formato: `{dbname}_{form}_{randomId}`
- Campo `Codigo` e o identificador principal de negocio na maioria das entidades

### Padrao MVC (Java - intra_ntf)
O intra_ntf possui a arquitetura Java mais madura com classes abstratas:
- `AbstractModelDoc` -> Entidades de dados
- `AbstractDaoNsf` -> Acesso a dados via OpenNTF Domino API
- `AbstractViewDoc / AbstractViewLista` -> Controllers de UI
- `AbstractApp` -> Contexto de aplicacao

## Estrategia de Migracao para Java/Vaadin

### Ordem Sugerida de Migracao (por dependencia)
1. **empresas_ntf** - Base cadastral (menos dependencias, mais referenciada)
2. **listas_ntf** - Estrutura simples de itens/servicos
3. **adm_ntf** - RH e administracao (relativamente independente)
4. **aprimoramento_ntf** - Desenvolvimento profissional (depende de adm)
5. **sales_ntf** - Vendas/CRM (depende de empresas)
6. **compras_ntf** - Compras (depende de empresas, listas)
7. **estoquemain_ntf** - Estoque (depende de compras, vendas)
8. **caixa_ntf** - Financeiro (depende de compras)
9. **servicos_ntf** - Servicos (depende de empresas)
10. **bi_ntf** - BI (depende de todas)
11. **intra_ntf** - Portal (integra tudo)

### Mapeamento de Conceitos Notes -> Java/Vaadin

| Conceito Notes | Equivalente Java/Vaadin |
|---|---|
| Form | JPA Entity / Spring Data |
| View (Notes) | Repository + JPA Query / Criteria API |
| Agent (agendado) | Spring @Scheduled / Quartz Job |
| Agent (acao) | Service method / REST endpoint |
| Script Library | Service class / Utility class |
| XPage | Vaadin View (Route) |
| Custom Control | Vaadin Component / Composite |
| Document Link | Foreign Key / @ManyToOne |
| Response Document | @OneToMany relationship |
| ACL / Roles | Spring Security roles |
| Full Text Search | Hibernate Search / Elasticsearch |

### Compliance Tributario Brasileiro

Todas as bases lidam com impostos brasileiros que devem ser preservados:
- **ICMS** - Imposto sobre Circulacao de Mercadorias (por UF)
- **IPI** - Imposto sobre Produtos Industrializados
- **PIS/COFINS** - Contribuicoes sociais
- **ICMS-ST** - Substituicao tributaria
- **IRRF** - Imposto de renda retido na fonte
- **ISS** - Imposto sobre servicos
- **CSLL** - Contribuicao social sobre lucro liquido
- **INSS** - Previdencia social
- **NFe** - Nota fiscal eletronica (XML)
- **SINTEGRA** - Relatorio para fisco
- **Suframa** - Incentivos Zona Franca de Manaus

### Stack Tecnologico Alvo
- **Backend**: Java 17+, Spring Boot, Spring Data JPA, Spring Security
- **Frontend**: Vaadin Flow (componentes Java-side)
- **Banco de Dados**: PostgreSQL (ou outro relacional)
- **Build**: Maven ou Gradle
- **Autenticacao**: Spring Security com roles mapeadas do legado
- **Agendamento**: Spring @Scheduled ou Quartz
- **Email**: Spring Mail (substituindo Domino Memo)
- **Relatorios**: JasperReports ou similar
- **Busca**: Hibernate Search ou Elasticsearch

## Servidores do Ambiente Legado
- **ebix.tdec.com.br** - Execucao principal de agentes agendados
- **lexapro.tdec.com.br** - Servidor de aplicacao e on-behalf-of
- **luvox.tdec.com.br** - Servidor secundario
- **intra.tdec.com.br** - Portal web (intranet)

## Convencoes para Desenvolvimento

- Manter README.md atualizado em cada repositorio _ntf
- Documentar decisoes de mapeamento de entidades
- Priorizar migracao de regras de negocio sobre formatacao visual
- Preservar calculos tributarios com testes unitarios
- Manter rastreabilidade entre entidades Notes e entidades JPA
