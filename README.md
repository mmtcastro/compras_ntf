# Compras NTF - Compras e Aquisicoes

Base de dados HCL Domino/Notes para gestao de compras e aquisicoes da TDec.

## Finalidade

Sistema completo de gestao de compras que gerencia pedidos, cotacoes, recebimento de mercadorias (baixas), notas fiscais eletronicas (NFe) e compliance tributario brasileiro. Suporta operacoes multi-moeda e integra-se com estoque, vendas e financeiro.

## Entidades Principais (Formularios)

### Compras
- **Pedido** - Pedidos de compra (itens, quantidades, datas de entrega, garantia)
- **Cotacao** - Cotacoes de fornecedores (precos, prazos, condicoes de pagamento)
- **Compra** - Itens individuais de compra (documento-resposta ao Pedido)
- **Fornecedor** - Cadastro de fornecedores

### Recebimento e Fiscal
- **Baixa** - Recebimento de mercadorias e notas fiscais
  - Calculo de ICMS, IPI, PIS/COFINS
  - Importacao de XML NFe
  - Integracao com ativo fixo
- **Cancelamento** - Cancelamento de pedidos/itens
- **MemoNFe** - Comunicacoes sobre NFe

### Outros
- **TrocaReserva** - Trocas e reservas de estoque
- **Renovacao** - Renovacao de licencas de software e manutencao
- **Licenca** - Contratos de licenca/manutencao
- **Codigo** - Dados mestres de produtos

## Views Principais (45+ views)

### Pedidos
- Por data, fornecedor, codigo, entrega, status, NFe, negocio
- Pendentes/parciais, por negocio, consumo, cliente

### Baixas
- Por data, fornecedor, NF, numero de controle, DocID, criacao
- NFTs a emitir, codigo espelho para ativo fixo

### Cotacoes e Fornecedores
- Por codigo, pendentes/parciais, fornecedor

## Ciclo de Vida do Pedido

1. **Criacao** - Pedido de compra criado
2. **Cotacao** - Cotacoes recebidas como documentos-resposta
3. **Compra** - Itens convertidos em linhas de compra
4. **Rastreamento** - Acompanhamento de entrega (previsto vs. entregue)
5. **Recebimento** - Baixa com nota fiscal e calculo de impostos
6. **NFe** - Validacao de XML da nota fiscal eletronica
7. **Cancelamento/Troca** - Cancelamento ou troca de reserva

## Automacoes (Agents)

### Processamento Batch
- **BaixasBatch** - Agendado a cada 5 minutos no servidor ebix; processa baixas em lote
  - Gera movimentos, atualiza catalogo, cria historico, valida XML
- **CalculaTotaisPedidos** - Recalcula totais de todos os pedidos pendentes
- **AtualizaMovimento** - Atualiza registros de movimentacao de estoque

### Acoes
- **CkBaixa / CkAviso / CkAvisoXML** - Validacoes de recebimento
- **CkProcessamento** - Processamento de workflow
- **NFe** - Gestao de nota fiscal eletronica
- **PopulaRegrasFiscais** - Carrega regras fiscais
- **ckPedidosPendenteParcial** - Monitora pedidos pendentes

## Script Libraries Principais

| Biblioteca | Tamanho | Funcao |
|---|---|---|
| **Negocios.lss** | 921 KB | Faturamento, calculo de impostos, margem, totais |
| **Baixas.lss** | 127 KB | Recebimento, NFe XML, movimento, custos, regras fiscais |
| **Pedidos.lss** | 45 KB | Processamento de pedidos, moedas, saldos |
| **Cotacoes.lss** | 44 KB | Processamento de cotacoes |
| **Planilha.lss** | 174 KB | Calculos de planilha/matriz |
| **Troca de Reserva.lss** | 69 KB | Logica de troca e reserva |

## Compliance Tributario

- **ICMS** - Imposto sobre Circulacao de Mercadorias
- **IPI** - Imposto sobre Produtos Industrializados
- **PIS/COFINS** - Contribuicoes sociais
- **ICMS-ST** - Substituicao tributaria
- **IRRF** - Imposto de renda retido na fonte
- **SINTEGRA** - Relatorio para fisco
- **NFe** - Nota fiscal eletronica (importacao/validacao XML)
- **Suframa** - Incentivos da Zona Franca

## Integracoes

| Base de Dados | Integracao |
|---|---|
| **sales.nsf** | Cotacoes de moeda (USD), taxas de cambio diarias |
| **empresas.nsf** | Excecoes tributarias, incentivos fiscais |
| **listas.nsf** | Listas de itens, intermediacao, planilhas |

## Tecnologia

- HCL Domino/Notes
- LotusScript (logica de negocio principal)
- XPages: form_home.xsp (interface web minima)
- 15 formularios, 45+ views, 19 agentes, 14+ script libraries
