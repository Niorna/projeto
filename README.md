import sqlite3
import datetime

# criação

banco = "estoque.db"
conexao = sqlite3.connect(banco)
cursor = conexao.cursor()

# conexão

def conectar_banco():
    return sqlite3.connect("estoque.db")

# tabela principal

def criar_tabelas():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS estoque(
        codigo TEXT PRIMARY KEY,
        nome_item TEXT NOT NULL,
        quantidade REAL NOT NULL,
        categoria TEXT NOT NULL,
        armazenar TEXT NOT NULL,
        unidade_medida TEXT NOT NULL,
        preco_unitario REAL NOT NULL,
        preco_total REAL NOT NULL,
        estoque_minimo REAL NOT NULL
    )
    """)
    # tabela de movimentação
    cursor.execute("""
    CREATE TABLE IF NOT EXISTS movimentacao(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        codigo_produto TEXT,
        tipo TEXT,
        quantidade REAL,
        data TEXT,
        observacao TEXT
    )
    """)
    conexao.commit()
    conexao.close()

# cadastro

def cadastro_produto():
    print("cadastro de produtos")
    codigo = input("Digite o código do produto tendo 3 digitos como 021: ")
    if not codigo.isdigit() or len(codigo) != 3:
        print("Erro: o código deve ocupar 3 casas. Ex: 001.")
        return
    nome_item = input("Digite o nome do produto: ")
    categoria = input("Digite a categoria do produto: ")
    armazenar = input("Digite onde o produto será armazenado (geladeira, freezer, prateleira): ")
    quantidade = float(input("Digite a quantidade inicial do produto: "))
    unidade_medida = input("Digite a unidade de medida do produto como un, kg, pacotes, L: ")
    preco_unitario = float(input("Digite o preço unitário do produto: "))
    estoque_minimo = float(input("Digite a quantidade mínima para alerta de estoque: "))
    preco_total = quantidade * preco_unitario
    conexao = conectar_banco()
    cursor = conexao.cursor()
    try:
        cursor.execute("""
        INSERT INTO estoque(
            codigo,
            nome_item,
            quantidade,
            categoria,
            armazenar,
            unidade_medida,
            preco_unitario,
            preco_total,
            estoque_minimo
        )
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
        """, (
            codigo,
            nome_item,
            quantidade,
            categoria,
            armazenar,
            unidade_medida,
            preco_unitario,
            preco_total,
            estoque_minimo
        ))
        # registra entrada inicial
        data = datetime.datetime.now().strftime("%d/%m/%Y %H:%M")
        cursor.execute("""
        INSERT INTO movimentacao(
            codigo_produto,
            tipo,
            quantidade,
            data,
            observacao)
        VALUES (?, ?, ?, ?, ?)
        """, (codigo,"entrada",quantidade,data,"cadastro inicial"))
        conexao.commit()
        print("Produto cadastrado com sucesso.")
        print(f"Preço total do estoque: R$ {preco_total:.2f}")
    except sqlite3.Error as erro:
        print("Erro ao cadastrar produto:", erro)
    conexao.close()

# entrada de produto

def entrada_produto():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    codigo = input("Digite o código do produto: ")
    cursor.execute("SELECT * FROM estoque WHERE codigo = ?", (codigo,))
    produto = cursor.fetchone()
    if produto is None:
        print("Produto não encontrado.")
        conexao.close()
        return
    quantidade = float(input("Digite a quantidade de entrada: "))
    cursor.execute("""
    UPDATE estoque
    SET quantidade = quantidade + ?
    WHERE codigo = ?
    """, (quantidade, codigo))
    # atualiza preço total
    cursor.execute("""
    UPDATE estoque
    SET preco_total = quantidade * preco_unitario
    WHERE codigo = ?
    """, (codigo,))
    data = datetime.datetime.now().strftime("%d/%m/%Y %H:%M")
    cursor.execute("""
    INSERT INTO movimentacao(codigo_produto, tipo, quantidade, data, observacao)
    VALUES (?, ?, ?, ?, ?)
    """, (codigo, "entrada", quantidade, data, "reposição de estoque"))
    conexao.commit()
    conexao.close()
    print("Entrada registrada com sucesso.")

# saída de produtos

def saida_produto():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    codigo = input("Digite o código do produto: ")
    cursor.execute("SELECT * FROM estoque WHERE codigo = ?", (codigo,))
    produto = cursor.fetchone()
    if produto is None:
        print("Produto não encontrado.")
        conexao.close()
        return
    quantidade_saida = float(input("Digite a quantidade de saída: "))
    if quantidade_saida > produto[2]:
        print("Erro: quantidade maior que o estoque.")
        conexao.close()
        return
    tipo_saida = input(
        "Digite o tipo de saída (venda, perda, transferencia): ")
    cursor.execute("""
    UPDATE estoque
    SET quantidade = quantidade - ?
    WHERE codigo = ?
    """, (quantidade_saida, codigo))
    # atualiza preço total
    cursor.execute("""
    UPDATE estoque
    SET preco_total = quantidade * preco_unitario
    WHERE codigo = ?
    """, (codigo,))
    data = datetime.datetime.now().strftime("%d/%m/%Y %H:%M")
    cursor.execute("""
    INSERT INTO movimentacao(codigo_produto, tipo, quantidade, data, observacao)
    VALUES (?, ?, ?, ?, ?)
    """, (codigo, tipo_saida, quantidade_saida, data, "saída registrada"))
    conexao.commit()
    conexao.close()
    print("Saída registrada com sucesso.")

# consultar estoque em tempo real

def consultar_estoque():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    cursor.execute("SELECT * FROM estoque")
    produtos = cursor.fetchall()
    conexao.close()
    print("\nESTOQUE ATUAL")
    if len(produtos) == 0:
        print("Não existem produtos cadastrados.")
    else:
        for produto in produtos:
            print(f"""
codigo: {produto[0]}
nome: {produto[1]}
quantidade: {produto[2]}
categoria: {produto[3]}
armazenamento: {produto[4]}
unidade de medida: {produto[5]}
preço unitário: R$ {produto[6]:.2f}
preço total: R$ {produto[7]:.2f}
estoque mínimo: {produto[8]} """)
            
# alerta inteligente

def alerta_estoque():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    cursor.execute("""
    SELECT * FROM estoque
    WHERE quantidade <= estoque_minimo
    """)
    produtos = cursor.fetchall()
    conexao.close()
    print("\nALERTAS DE ESTOQUE")
    if len(produtos) == 0:
        print("Nenhum produto com estoque baixo.")
    else:
        for produto in produtos:
            print(f"""
ALERTA: o produto {produto[1]} está com estoque baixo.
Quantidade atual: {produto[2]}
Quantidade mínima: {produto[8]}
Necessidade de reposição.""")

# relatório gerencial

def relatorio_gerencial():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    cursor.execute("SELECT SUM(preco_total) FROM estoque")
    valor_total = cursor.fetchone()[0]
    if valor_total is None:
        valor_total = 0
    cursor.execute("SELECT COUNT(*) FROM movimentacao")
    total_movimentacoes = cursor.fetchone()[0]
# indicadores de desempenho
    estoque_medio = valor_total / 2
    if estoque_medio != 0:
        giro_estoque = valor_total / estoque_medio
    else:
        giro_estoque = 0
    conexao.close()
    print("RELATÓRIO GERENCIAL")
    print(f"Valor total do estoque: R$ {valor_total:.2f}")
    print(f"Total de movimentações: {total_movimentacoes}")
    print(f"Giro de estoque: {giro_estoque:.2f}")
    print("""
Indicadores de desempenho:
- Giro de estoque
- Tempo de reposição
- Nível de serviço
- Custos de manutenção""")

# histórico de movimentação

def historico_movimentacao():
    conexao = conectar_banco()
    cursor = conexao.cursor()
    cursor.execute("SELECT * FROM movimentacao")
    movimentacoes = cursor.fetchall()
    conexao.close()
    print("\nHISTÓRICO DE MOVIMENTAÇÃO")
    if len(movimentacoes) == 0:
        print("Nenhuma movimentação registrada.")
    else:
        for mov in movimentacoes:
            print(f"""
id: {mov[0]}
codigo produto: {mov[1]}
tipo: {mov[2]}
quantidade: {mov[3]}
data: {mov[4]}
observação: {mov[5]}
""")

# menu principal

def menu():
    criar_tabelas()
    while True:
        print("""
SISTEMA DE ESTOQUE

1 - Cadastrar produto
2 - Adicionar produtos
3 - Retirar produtos
4 - Consultar em estoque
5 - Alerta de estoque
6 - Histórico de movimentação
7 - Relatório gerencial
0 - Sair""")
        opcao = input("Digite o número que indica a ação: ")
        if opcao == "1":
            cadastro_produto()
        elif opcao == "2":
            entrada_produto()
        elif opcao == "3":
            saida_produto()
        elif opcao == "4":
            consultar_estoque()
        elif opcao == "5":
            alerta_estoque()
        elif opcao == "6":
            historico_movimentacao()
        elif opcao == "7":
            relatorio_gerencial()
        elif opcao == "0":
            print("Saindo do sistema.")
            break
        else:
            print("alternativa invalida")
menu()
