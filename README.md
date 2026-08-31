# 📦 Sistema de Gerenciamento de Estoque em Python & SQLite

Um sistema simples e eficiente de controle de estoque via linha de comando (CLI) desenvolvido em **Python** utilizando **SQLite** para persistência de dados.

O sistema permite cadastrar produtos, controlar entradas e saídas, emitir alertas de estoque mínimo, consultar histórico de movimentações e gerar relatórios gerenciais com indicadores de desempenho.

---

## 🚀 Funcionalidades

- **📦 Cadastro de Produtos**: Permite cadastrar novos itens validando o código de 3 dígitos, definindo categoria, local de armazenamento, preços e nível de estoque mínimo.
- **📥 Entrada de Produtos**: Registro de reposição de itens no estoque com atualização automática do valor total acumulado e histórico.
- **📤 Saída de Produtos**: Registro de vendas, perdas ou transferências com validação de saldo disponível.
- **🔍 Consulta de Estoque**: Visualização detalhada de todos os itens em estoque em tempo real.
- **⚠️ Alerta Inteligente de Estoque**: Notificação de produtos com quantidade abaixo ou igual ao estoque mínimo configurado.
- **📜 Histórico de Movimentações**: Registro auditável de todas as operações (entradas, saídas, cadastros) com data, hora e observações.
- **📊 Relatório Gerencial**: Apresentação de estatísticas como valor total investido no estoque, quantidade de movimentações e giro de estoque.

---

## 🛠️ Tecnologias Utilizadas

- **[Python 3](https://www.python.org/)** — Linguagem base do sistema.
- **[SQLite3](https://www.sqlite.org/)** — Banco de dados relacional embarcado (não requer instalação adicional).

---

## 📁 Estrutura do Banco de Dados

O sistema cria automaticamente o banco de dados `estoque.db` com as seguintes tabelas:

### 1. Tabela `estoque`
| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `codigo` | `TEXT` *(PK)* | Código único de 3 dígitos (ex: `001`, `021`) |
| `nome_item` | `TEXT` | Nome do produto |
| `quantidade` | `REAL` | Quantidade disponível |
| `categoria` | `TEXT` | Categoria do produto |
| `armazenar` | `TEXT` | Local de armazenamento (ex: geladeira, prateleira) |
| `unidade_medida` | `TEXT` | Unidade (ex: kg, un, L, pacotes) |
| `preco_unitario` | `REAL` | Preço de custo/venda por unidade |
| `preco_total` | `REAL` | Quantidade × Preço Unitário |
| `estoque_minimo` | `REAL` | Quantidade mínima para disparo de alertas |

### 2. Tabela `movimentacao`
| Coluna | Tipo | Descrição |
| :--- | :--- | :--- |
| `id` | `INTEGER` *(PK AI)* | Identificador único da movimentação |
| `codigo_produto` | `TEXT` | Código do produto movimentado |
| `tipo` | `TEXT` | Tipo de operação (`entrada`, `venda`, `perda`, etc.) |
| `quantidade` | `REAL` | Quantidade movimentada |
| `data` | `TEXT` | Data e hora do registro (`DD/MM/AAAA HH:MM`) |
| `observacao` | `TEXT` | Detalhes ou justificativa da movimentação |

---

## ⚡ Como Executar o Projeto

### Pré-requisitos
- Ter o **Python 3.x** instalado no seu computador.

### Passo a Passo

1. **Clone o repositório ou baixe o arquivo:**
   ```bash
   git clone [https://github.com/seu-usuario/seu-repositorio.git](https://github.com/seu-usuario/seu-repositorio.git)
   cd seu-repositorio
