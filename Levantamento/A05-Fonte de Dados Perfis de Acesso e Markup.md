# A05 — Fonte de Dados, Perfis de Acesso e Markup

**Projeto de CRM | Master
Responsável:** Sofia Medeiros do Nascimento
**Requisitos Mapeados:** RM10, RM12, RM13

## 1. Objetivo da A05

Descrever a arquitetura de integração de dados da operação, a estrutura de perfis de usuário, as regras de visibilidade financeira e o cálculo do markup no CRM. Este documento consolida o relatório de alinhamento revisado, a entrevista inicial e as validações de requisitos e governança.

## 2. Integração e Fonte de Dados (SS Plus)

A alimentação e atualização das tabelas do CRM serão sustentadas pelo sistema atual da operação (identificado nos requisitos formais como SS Plus).

| Item | Regra e Definição |
| --- | --- |
| **Integração por API** | O sistema possui integração total para suprir as necessidades de dados da aplicação. Por meio da API, é possível acessar e atualizar as tabelas de maneira rápida, garantindo fluxo contínuo. |
| **Exportação Manual** | A maior parte das atualizações operacionais hoje é conduzida por planilhas de Excel do supervisor (que não podem ser compartilhadas por sigilo). Para o andamento do projeto, relatórios específicos serão exportados nos formatos CSV e XLS. |
| **Reconciliação (RM13)** | A importação (seja via API ou CSV) deverá reconciliar clientes, produtos e vendas, garantindo o controle de duplicidade, atualização, origem e tratamento de exceções. |

**Acesso do grupo:** Como os dados reais são confidenciais, a sugestão para o protótipo é trabalhar com exportações em CSV/XLS usando dados fictícios (mantendo a mesma estrutura de colunas) e tratar o consumo direto via API como etapa de desenvolvimento.

## 3. Perfis de Acesso e Governança

Ficou definido que a estrutura contará com quatro perfis distintos: **Administrador (admin), Supervisor, Vendedor e Marketing**.

| Regra de Visibilidade | Definição |
| --- | --- |
| **Sigilo de Custo** | Como regra fundamental de negócio, **nenhum dos perfis** (nem mesmo o administrador) terá acesso aos valores brutos de custo. |
| **Exibição do Markup** | Em substituição ao custo, o sistema apresentará exclusivamente o valor fechado do markup em formato percentual. |
| **Campos Sensíveis e Auditoria** | A edição de dados sensíveis (ex.: inscrição estadual) é restrita. Exige-se governança (RNFM02) com trilha de alterações para auditoria. |
| **Validação de Backend (RM12)** | As regras de bloqueio de visão (custos) e edição não devem ser apenas visuais; o CRM precisa validar no servidor (backend) os casos autorizados e negados. |
| **Carteira e Contatos** | Cada vendedor tem carteira fixa (clientes inativos há mais de 60 dias podem ser assumidos por outros). O histórico de contatos é visível para todos. |

**Matriz consolidada de permissões**

| Perfil | Ver Custo | Ver Markup % | Carteira | Registrar Contatos e Cotações | Publicar Avisos e Campanhas (RM11) | Editar Campos Sensíveis | Gerenciar Usuários |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Admin** | Não | Sim | Todas | Sim | Sim | Sim | Sim |
| **Supervisor** | Não | Sim | Todas | Sim | Não | Não | Não |
| **Vendedor** | Não | Sim | Própria / Inativos > 60 dias | Sim | Não | Não | Não |
| **Marketing** | Não | Não | Sem acesso | Não | Sim | Não | Não |

## 4. Cálculo e Exibição do Markup (RM10)

### 4.1 Fórmula

**Markup (%) = (valor de venda ÷ valor de custo − 1) × 100**

*Exemplo: Venda a R$ 150,00 e Custo a R$ 100,00 → (1,5 - 1)*  100 = 50%.*

O markup do total da cotação usa a **soma das vendas e a soma dos custos**, e não a média dos percentuais dos itens.

### 4.2 Faixas de cor (MKP Geral)

- **Azul:** 80,00% ou acima
- **Verde:** de 70,00% a 79,99%
- **Amarelo:** de 60,00% a 69,99%
- **Vermelho:** abaixo de 60,00%

### 4.3 Regras de exibição e bloqueio

- O vendedor vê o percentual e a cor da faixa na tela de cotação.
- PDF, impressão, compartilhamento e exportação destinados ao cliente **não trazem** custo nem markup.

## 5. Alertas de Incompatibilidade e Pontos Críticos

Com o cruzamento das regras operacionais, dois bloqueios técnicos precisam ser alinhados junto à equipe:

1. **Conflito de Extração (PDF vs. Markup):** A documentação da A03 sugere que as cotações feitas no SS Plus sejam importadas para o CRM apenas como PDF. No entanto, o CRM não consegue ler itens estruturados dentro de um PDF estático para calcular o Markup. Para que o RM10 funcione (exibir o markup por item e total na tela do vendedor), a integração precisa obrigatoriamente trazer os dados da cotação de forma estruturada (via API), ou o markup já precisa vir calculado do SS Plus junto com o PDF.
2. **Imutabilidade Histórica (RN03):** Como os custos dos produtos variam com o tempo, o CRM precisará registrar um *snapshot* (fotografia) dos valores no momento em que a cotação é registrada ou enviada. Isso garante que a atualização posterior de custos na API não altere retroativamente o percentual de markup de uma negociação antiga.

## 6. Requisitos Relacionados à A05

| Requisito | Escopo abordado nesta entrega |
| --- | --- |
| **RM10** | Cálculo de markup pela fórmula (venda ÷ custo − 1) × 100, faixas de cores, ocultação absoluta do custo para todos os perfis e remoção do dado nos PDFs enviados. |
| **RM11** | Definição do perfil “Marketing” e “Admin” como responsáveis por atualizar avisos e campanhas. |
| **RM12** | Quatro perfis estabelecidos, com matriz de permissões, restrição de edição e exigência de validação no servidor (backend) para regras de acesso e custo. |
| **RM13** | Integração definida via API e CSV/XLS, com exigência de reconciliação de dados, tratamento de duplicidades e uso de dados fictícios no protótipo. |
| **RNFM02** | Exigência de governança e trilha de auditoria para a edição de campos sensíveis de clientes. |
| **RN03** | Necessidade de salvar snapshots de cotações para manter o histórico de markups imune a flutuações futuras de custos. |

## 7. Critérios de Aceite Sugeridos

- A matriz de permissões está definida para os 4 perfis, e a validação de acesso ao custo e campos sensíveis é garantida via servidor.
- **Nenhum usuário** do sistema visualiza os valores brutos de custo.
- A fonte de dados via API e CSV/XLS está definida, prevendo a reconciliação e o controle de duplicidades (RM13).
- O markup é calculado pela fórmula validada, baseado no somatório de custos e vendas (para totais), e exibido exclusivamente em percentual e cores.
- Documentos externos (PDF/impressão para clientes) omitem qualquer traço de custo ou markup.
- Os conflitos técnicos (Importação de cotação em PDF vs. Necessidade de dados estruturados para cálculo do Markup) estão mapeados para definição com a Master.