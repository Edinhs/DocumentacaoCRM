# A05 — Fonte de Dados, Perfis de Acesso e Markup

**Projeto de CRM | Master**
**Responsável:** Sofia Medeiros do Nascimento
**Requisitos Mapeados:** RM10, RM11, RM12, RM13, RNFM02, RN03

## 1. Objetivo da A05
Descrever a fonte de dados do CRM, a estrutura de perfis de usuário, as regras de visibilidade financeira e o fluxo de cálculo do markup. Este documento consolida a entrevista inicial e as respostas de Jean enviadas por áudio.

## 2. Integração e Fonte de Dados (SS Plus)

A alimentação e atualização das tabelas do CRM tem como origem o sistema atual da operação (SS Plus).

| Item | Regra e Definição |
| :--- | :--- |
| **Integração por API** | Jean informou que a API é completa: acessa e atualiza as tabelas do sistema, e a integração já foi validada antes pela equipe de integração da Master. A documentação e o acesso à API não foram fornecidos ao grupo. |
| **Exportação de relatórios** | Relatórios específicos podem ser exportados em CSV e XLS. A planilha geral do supervisor, que concentra a maior parte das atualizações, não pode ser compartilhada por sigilo. |
| **Relatórios existentes** | Movimentação geral, notas fiscais, vendas por cliente e produtos vendidos. Também existem o relatório semanal do percentual da meta atingida por cliente (A04) e o relatório de perdas de vendas (A03). |
| **Reconciliação (RM13)** | A importação deve reconciliar clientes, produtos e vendas, com controle de duplicidade, atualização, origem e tratamento de exceções. |

**Dados do protótipo:** o grupo não teve acesso ao SS Plus nem a dados reais. O protótipo usa uma planilha com valores fictícios, na mesma lógica dos relatórios exportados em CSV/XLS. Estrutura sugerida, que cobre também o histórico, as metas e os sinais de queda (RM01, RM04 e RM05):

| Data da venda | Vendedor | Cliente | Perfil | Meta mensal do cliente | Código interno | Produto | Marca | Qtd | Preço de venda | Custo (não exibido) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 05/10/2026 | Vendedor 1 | Transportadora Exemplo | Transportadora | R$ 10.000,00 | P001 | Item A (exemplo) | Marca X | 1 | R$ 150,00 | R$ 100,00 |
| 05/10/2026 | Vendedor 1 | Transportadora Exemplo | Transportadora | R$ 10.000,00 | P002 | Item B (exemplo) | Marca Y | 1 | R$ 80,00 | R$ 50,00 |

**Sugestão de mapeamento entre dados e requisitos**

Jean citou os relatórios, mas não disse qual alimenta cada funcionalidade. A relação abaixo é proposta do grupo.

| Requisito | Dado necessário | Relatório de origem |
| :--- | :--- | :--- |
| RM01 — Histórico | Vendas por cliente, período e produto | Vendas por cliente; produtos vendidos |
| RM04 — Metas | Meta mensal do cliente e valor realizado | Percentual da meta atingida; vendas por cliente ou notas fiscais, conforme a base do realizado definida na A04 |
| RM05 — Queda e mix | Vendas dos últimos 3 meses, média do cliente, produtos comprados e data da última compra | Vendas por cliente; produtos vendidos; movimentação geral |
| RM10 — Markup | Itens e preços de venda da cotação; custo | Cotação em PDF (A03); custo, que não é exibido |
| Carteira e perdas (A03) | Cliente por vendedor; perdas de vendas | Cadastro de clientes; relatório de perdas |

## 3. Perfis de Acesso e Governança

Jean definiu quatro perfis: **Administrador (admin), Supervisor, Vendedor e Marketing**.

| Regra de Visibilidade | Definição |
| :--- | :--- |
| **Sigilo de Custo** | Jean afirmou que "o custo ninguém vê, só o markup fechado". Como ele não separou os perfis, adota-se que **nenhum perfil** (nem o administrador) vê o custo bruto dentro do CRM. |
| **Exibição do Markup** | Em substituição ao custo, o sistema apresenta exclusivamente o markup fechado, em percentual. |
| **Campos Sensíveis e Auditoria** | A edição de dados sensíveis (ex.: inscrição estadual) é restrita. Propõe-se trilha de alterações para auditoria (RNFM02). |
| **Validação no servidor (RM12)** | O bloqueio de visão (custo) e de edição não deve ser apenas visual: o CRM valida no servidor os casos autorizados e negados. |
| **Carteira e Contatos** | Cada vendedor tem carteira fixa. Clientes inativos há mais de 60 dias podem ser assumidos por outro vendedor. O histórico de contatos é visível para todos (A03). |

**Matriz consolidada de permissões**

Custo, markup em percentual, carteira do vendedor, publicação de avisos pelo marketing e acompanhamento de metas por vendedores e supervisores (A04) foram informados pelo cliente. As demais permissões são proposta do grupo.

| Perfil | Ver Custo | Ver Markup % | Carteira | Metas, histórico e sinais de queda (RM01, RM04, RM05) | Registrar Contatos e Cotações | Publicar Avisos e Campanhas (RM11) | Editar Campos Sensíveis | Gerenciar Usuários |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Admin** | Não | Sim | Todas | Todas | Sim | Sim | Sim | Sim |
| **Supervisor** | Não | Sim | Todas | Todas | Sim | Não | Não | Não |
| **Vendedor** | Não | Sim | Própria / Inativos > 60 dias | Própria carteira | Sim | Não | Não | Não |
| **Marketing** | Não | Não | Sem acesso | Não | Não | Sim | Não | Não |

## 4. Cálculo e Exibição do Markup (RM10)

### 4.1 Fluxo da cotação
Por orientação da gerência (A03), as cotações são geradas no SS Plus e importadas no CRM em PDF, evitando uma cotação paralela.

No protótipo, as cotações são carregadas pela planilha com valores fictícios (venda e custo por item). O CRM calcula o markup, aplica a faixa de cor e não exibe o custo. A leitura de cotações em PDF e a origem do cálculo em uma versão real (SS Plus ou CRM) ficam para a etapa de desenvolvimento.

### 4.2 Fórmula
**Markup (%) = (valor de venda ÷ valor de custo − 1) × 100**

*Exemplo: venda a R$ 150,00 e custo a R$ 100,00 → (1,5 − 1) × 100 = 50%.*

O indicador exibido é o markup, e não a margem de contribuição (que considera despesas), conforme a distinção feita por Jean.

O markup do total da cotação usa a **soma das vendas e a soma dos custos**, e não a média dos percentuais dos itens.

| Caso | Venda | Custo | Markup |
| :--- | :--- | :--- | :--- |
| Item A | R$ 150,00 | R$ 100,00 | 50% |
| Item B | R$ 80,00 | R$ 50,00 | 60% |
| **Total (A + B)** | R$ 230,00 | R$ 150,00 | **53,33%** |

### 4.3 Faixas de cor (MKP Geral)
* **Azul:** 80,00% ou acima
* **Verde:** de 70,00% a 79,99%
* **Amarelo:** de 60,00% a 69,99%
* **Vermelho:** abaixo de 60,00%

Exemplos: item A (50%) vermelho, item B (60%) amarelo, total A + B (53,33%) vermelho.

Essas cores são do markup e não se confundem com as cores de queda de compras da A04. Na interface, cada indicador deve ter rótulo próprio ("Markup" e "Queda de compras"). As faixas devem ser parametrizáveis.

### 4.4 Regras de exibição e bloqueio
* O vendedor vê o percentual e a cor da faixa na tela de cotação.
* PDF, impressão, compartilhamento e exportação destinados ao cliente **não trazem** custo nem markup.
* Se o custo for zero ou não existir, o markup não é calculado e o CRM indica "sem base de cálculo", seguindo a mesma regra da A04 para valores de referência zerados.

## 5. Pontos Críticos de Arquitetura

**Custo oculto x cálculo do markup:** para calcular o markup o sistema precisa conhecer o custo, mas nenhum perfil pode vê-lo. A solução é calcular e validar no servidor: o navegador recebe apenas o percentual e a cor da faixa, nunca o custo.

**Histórico imutável (RN03):** os custos mudam com o tempo. Para que uma atualização posterior não altere o markup de uma negociação já enviada, o CRM registra, no momento do envio da cotação, o resultado: valores de venda, markup (%) e data. Isso preserva o histórico sem guardar o custo bruto, o que mantém o sigilo da seção 3.

## 6. Requisitos Relacionados à A05

| Requisito | Escopo abordado nesta entrega |
| :--- | :--- |
| **RM10** | Fórmula do markup, faixas de cor, custo não exibido e remoção do dado nos documentos enviados ao cliente. |
| **RM11** | Perfis Marketing e Admin como responsáveis por atualizar avisos e campanhas. |
| **RM12** | Quatro perfis, matriz de permissões, restrição de edição e validação no servidor para as regras de acesso e custo. |
| **RM13** | Fonte de dados por API e CSV/XLS, com reconciliação e tratamento de duplicidades; protótipo com planilha fictícia. Telas com dados do SS Plus exibem a origem e a data da última atualização (RM01). |
| **RNFM02** | Governança e trilha de auditoria para edição de campos sensíveis. |
| **RN03** | Registro do resultado da cotação no envio, para manter o histórico de markup. |

## 7. Critérios de Aceite Sugeridos

* A matriz de permissões está definida para os quatro perfis, e a validação de acesso a custo e campos sensíveis é feita no servidor.
* Nenhum usuário visualiza o custo bruto no CRM.
* O protótipo carrega a planilha com valores fictícios e reconcilia clientes, produtos e vendas, sem duplicidades (RM13).
* As telas com dados vindos do SS Plus exibem a origem e a data da última atualização.
* O markup é calculado pela fórmula definida, inclusive no total da cotação, e exibido em percentual com a cor da faixa.
* PDF e impressão destinados ao cliente não trazem custo nem markup.

## 8. Premissas Adotadas

Como o levantamento com o cliente está encerrado, as definições abaixo valem para as próximas etapas:

* O protótipo usa planilha com valores fictícios. Integração por API e leitura de cotações em PDF ficam para a etapa de desenvolvimento.
* O CRM calcula o markup no protótipo a partir da planilha, sem exibir o custo.
* O custo não é exibido a nenhum perfil, inclusive o admin.
* O marketing não visualiza o markup.
* As faixas de cor valem para cada item e para o total da cotação.
* O valor de 80,00% exato entra como azul.
* As faixas de cor do markup são parametrizáveis, como os demais critérios ainda não confirmados (A04).
* As permissões não informadas por Jean (matriz da seção 3) são proposta do grupo.
