# Bennie Flow — protótipo conceitual

Protótipo front-end para validar a experiência de um sistema de gestão de encomendas e capacidade de uma gráfica/estúdio de produção.

## O que o protótipo demonstra

- dashboard de pedidos, capacidade e alertas;
- pedidos com múltiplos produtos e controle de quantidade produzida/perdida;
- cotação com cálculo conceitual de preço, consumo e primeira data segura;
- cronograma por recurso/processo;
- margem de segurança adicionada ao lead time;
- estoque total, reservado e livre;
- estrutura de produto (BOM) com matéria-prima e roteiro de fabricação;
- etapas de pré-impressão, impressão, laminação/corte, acabamento e conferência;
- espaço conceitual para anexar imagens/referências e comentários ao item do pedido;
- interface responsiva em branco, vinho/vermelho escuro, com linguagem visual delicada e coelho original simplificado.

## Regra central de planejamento

Quando um pedido for confirmado, o backend deverá executar uma transação que:

1. congela a versão da estrutura de cada item do pedido;
2. calcula e reserva a matéria-prima necessária;
3. calcula o tempo de setup + tempo variável de cada operação;
4. procura janelas livres por recurso/máquina, permitindo operações de pedidos diferentes em paralelo quando os recursos não conflitam;
5. respeita dependências entre operações do mesmo item;
6. adiciona margem de segurança configurável;
7. calcula a primeira data de entrega realmente disponível;
8. impede promessas anteriores à data segura;
9. cria as tarefas de chão de fábrica;
10. recalcula capacidade e necessidade futura de materiais.

Perdas não devem simplesmente diminuir a quantidade boa. Cada item deve guardar `quantidade_pedida`, `quantidade_produzida`, `quantidade_aprovada`, `quantidade_perdida` e o motivo da perda. Se for necessária reposição, o planejador gera quantidade adicional e atualiza consumo e agenda.

## Modelo de domínio sugerido

- `customers`
- `quotes`, `quote_items`
- `orders`, `order_items`
- `item_attachments`, `item_comments`
- `products`, `product_versions`
- `materials`, `bom_items`
- `processes`, `product_routes`, `route_operations`
- `resources` (máquinas, bancadas ou grupos de capacidade)
- `work_centers`, `calendars`, `resource_downtimes`
- `production_jobs`, `job_operations`
- `material_stock`, `material_movements`, `material_reservations`
- `production_entries`, `scrap_entries`
- `shipments`
- `status_history`

## Arquitetura futura sugerida

```text
Web/PWA
   ↓
Node.js API (TypeScript)
   ├── Pedidos e cotações
   ├── Catálogo / BOM / roteiros
   ├── Estoque e reservas
   ├── Planejador de capacidade finita
   ├── Produção / apontamentos / perdas
   ├── Arquivos e comentários
   └── Portal do cliente
          ↓
      PostgreSQL
          +
Object Storage (imagens/arquivos)
```

Uma implementação adequada pode usar Node.js + TypeScript, PostgreSQL e um ORM. O planejador deve ficar isolado como módulo de domínio para que depois seja possível substituir a heurística inicial por um algoritmo de programação/otimização sem reescrever pedidos e estoque.

## Algoritmo inicial de agenda

Para o MVP, usar **forward finite-capacity scheduling**:

- operações ordenadas pelas dependências do roteiro;
- cada operação procura a primeira janela livre no calendário do recurso elegível;
- duração = setup + quantidade × tempo unitário;
- recursos diferentes podem trabalhar em paralelo;
- a operação seguinte só começa quando suas predecessoras terminarem;
- prazo comercial = término planejado + buffer configurável;
- pedidos prioritários podem ter peso maior, mas nunca ocupar a mesma capacidade simultaneamente.

Mais adiante, o planejador pode considerar lote econômico, agrupamento de impressões por material/cor, troca de setup, terceirização, turnos e otimização global.

## Executar o conceito

É estático: basta abrir `index.html`. Os dados atuais são simulados em `app.js` e serão substituídos pela API Node quando a arquitetura de backend for iniciada.
