### **📦  Projeto: Otimização e Análise de Vendas XPTO**

#### 🧩 Cenário
A empresa XPTO está em expansão e busca modernizar suas estratégias comerciais com base em análises de dados. 

Para isso, o departamento de vendas forneceu dois arquivos:
- `base_vendas.csv`: uma lista detalhada de vendas realizadas.
- `categorias_valores.csv`: com os percentuais de aumento dos produtos por categoria.

#### 🎯 Objetivos
- Aplicar os percentuais de aumento às vendas e gerar uma nova coluna com os valores ajustados.
- Comparar os resultados por categoria (antes e depois do ajuste).
- Calcular a comissão de cada vendedor (2,5% do total vendido).
- Criar gráficos visuais para análise (barras comparativas e comissões).
- Organizar os dados e simular o envio dos resultados por e-mail.

#### 🔍 Proposta
Este projeto destaca não somente o uso de Python em contexto empresarial, como também demonstra habilidades essenciais

 para transformar dados em decisões estratégicas: organização, automação de cálculos e storytelling visual.

## **🗂️ Configurações Iniciais**

#### 1º Criação do ambiente virtual. 
     python -m venv vendas_xpto 
#### 2º Ativação do ambiente virtual.
     .\vendas_xpto\Scripts\activate
#### 3º Instalação das Bibliotecas que serão utilizadas.
     pip install pandas numpy chardet matplotlib seaborn psycopg2 ipython-sql
### **📥 1 → Importação das bibliotecas**
```python
import pandas as pd
import numpy as np
import chardet
import matplotlib.pyplot as plt
import seaborn as sns
import psycopg2
### **🔍 2 → Verificar encoding dos arquivos CSV**

**obs:** Biblioteca `chardet` foi utilizada para a verificação.
rawdata = open('base_vendas.csv', 'rb').read()
encoding = chardet.detect(rawdata)['encoding']
print('Encoding detectado:', encoding)
rawdata = open('categorias_valores.csv', 'rb').read()
encoding = chardet.detect(rawdata)['encoding']
print('Encoding detectado: ', encoding)
##### **2.1 → Criação do Dataframe a partir do encoding detectado**
df_vendas = pd.read_csv('base_vendas.csv', encoding="ISO-8859-1", sep=';')
df_categorias = pd.read_csv('categorias_valores.csv', encoding="utf-8", sep=',')
### **💻 3 → Exploração da base de dados**
# Verificando as linhas iniciais
df_vendas.head()
# Verificando as linhas iniciais
df_categorias.head(2)
df_categorias.rename(columns={'Categoria': 'categoria'}, inplace=True)
# Tipos de dados das colunas
df_vendas.dtypes
# Tipos de dados das colunas
df_categorias.dtypes
# Informações da base de dados, que incluem: colunas, linhas e nulos
df_vendas.info()
# Informações da base de dados, que incluem: colunas, linhas e nulos
df_categorias.info()
# Numero de nulos
df_vendas.isnull().sum()
# Numero de nulos
df_categorias.isnull().sum()
# Estatística das colunas númericas de vendas
df_vendas['valor_venda'].describe()
# Estatistica das colunas textuais do dataframe de vendas.
df_vendas.describe(include=['object'])
# Estatística das colunas númericas de categorias
df_categorias.describe()
# Estatistica das colunas textuais do dataframe de categorias.
df_categorias.describe(include=['object'])
# Valores únicos da coluna 'Categoria' do dataframe de categorias.
lista_categoria = list(df_categorias['categoria'].unique())
print(lista_categoria)
# Valores únicos da coluna 'categoria_produto' do dataframe de vendas.
lista_produto = list(df_vendas['categoria_produto'].unique())
print(lista_produto)
# Valores únicos da coluna 'nome_vendedor' do dataframe de vendas.
lista_vendedor = list(df_vendas['nome_vendedor'].unique())
print(lista_vendedor)
# Valores únicos da coluna 'categoria_produto' do dataframe de vendas, fazendo a contagem nos resultados.
df_vendas['categoria_produto'].value_counts()
# Estatística da categoria do produto por valor de venda.
df_vendas.groupby('categoria_produto')['valor_venda'].describe()
### **📊 4 → Análise Gráfica**

Nesta etapa, utilizamos gráficos para visualizar os dados e identificar padrões, tendências e possíveis valores atípicos (outliers)

##### **4.1 Histograma de distribuição dos Valores de Venda**

**Resumo do fluxo**


✅ Quantidade de vendas em cada faixa de preço.
✅ Padrões de vendas, identificando faixas com maior e menor frequência.
plt.figure(figsize=(10, 5))
n, bins, patches = plt.hist(df_vendas['valor_venda'], bins=20, color='skyblue', edgecolor='black')

plt.title('Distribuição dos Valores de Venda')
plt.xlabel('Valor (R$)')
plt.ylabel('Frequência')

# Adicionando rótulos nas barras
for i in range(len(patches)):
    plt.text(patches[i].get_x() + patches[i].get_width()/2,
             n[i] + 2,  # posição acima da barra
             int(n[i]),
             ha='center', fontsize=9)

plt.tight_layout()
plt.show()
##### 4.2 Gráfico de barras por marca de produto

**Resumo do fluxo**

✅ Agrupamento por marca.
✅ Soma dos valores de venda por marca.
✅ Ordenamento dos valores de venda para analisar melhor.
df_marca = df_vendas.groupby('marca_produto')['valor_venda'].sum().sort_values()
print(df_marca)
fig, ax = plt.subplots(figsize=(10, 5))
barras = ax.bar(df_marca.index, df_marca.values, color='teal')
ax.set_title('Total de Vendas por Marca')
ax.set_xlabel('Marca')
ax.set_ylabel('Valor Total (R$)')
plt.xticks(rotation=45)

# Rótulos
for barra in barras:
    altura = barra.get_height()
    ax.text(barra.get_x() + barra.get_width()/2, altura + 1000,
            f"R$ {altura:,.0f}", ha='center', fontsize=8)

plt.tight_layout()
plt.show()
##### 4.3 Boxplot por categoria 

✅ Visualizar dispersão dos valores. ✅ Observa padrões e variabilidade.

plt.figure(figsize=(10, 5))
sns.boxplot(data=df_vendas, x='categoria_produto', y='valor_venda', palette='Set2')
plt.title('Distribuição de Valores de Venda por Categoria')
plt.xlabel('Categoria')
plt.ylabel('Valor da Venda (R$)')
plt.xticks(rotation=30)
plt.tight_layout()
plt.show()
#### **🧩 5 → Alteração na tabela (df_merge).**

✅ Alteração da coluna `valor` para `percentual_aumento`.

df_categorias.rename(columns={'Valor': 'percentual_aumento'}, inplace=True)

df_merge = df_vendas.merge(df_categorias, left_on="categoria_produto", right_on="categoria", how="left")
df_merge.head()
✅ Inserção de uma nova coluna no `df_merge` chamado `venda_final` aplicando o percentual de aumento.
df_merge['venda_final'] = df_merge['valor_venda'] * (1 + df_merge['percentual_aumento'] / 100)
df_merge.head()
✅ Visualização agrupado por `categoria` gerando um gráfico comparando os valores originais e com aumento.
# Agrupando os dados
df_categoria = df_merge.groupby('categoria_produto')[['valor_venda', 'venda_final']].sum().reset_index()

# Parâmetros do gráfico
x = np.arange(len(df_categoria))
largura_barra = 0.35

fig, ax = plt.subplots(figsize=(10, 5))

# Barras lado a lado
barras1 = ax.bar(x - largura_barra/2, df_categoria['valor_venda'], width=largura_barra, label='Valor Original', color='steelblue')
barras2 = ax.bar(x + largura_barra/2, df_categoria['venda_final'], width=largura_barra, label='Com Aumento', color='darkorange')

# Títulos e eixos
ax.set_title('Comparativo de Vendas por Categoria', fontsize=14)
ax.set_xlabel('Categoria')
ax.set_ylabel('Valor em R$')
ax.set_xticks(x)
ax.set_xticklabels(df_categoria['categoria_produto'], rotation=30)
ax.legend()

# Adicionando rótulos nas barras
for barra in barras1:
    altura = barra.get_height()
    ax.text(barra.get_x() + barra.get_width()/2, altura + 2000, f'{altura:,.0f}', ha='center', fontsize=8, color='black')

for barra in barras2:
    altura = barra.get_height()
    ax.text(barra.get_x() + barra.get_width()/2, altura + 2000, f'{altura:,.0f}', ha='center', fontsize=8, color='black')

plt.tight_layout()
plt.show()
✅ Calculo e adição de uma nova coluna `comissão` com 2,5% para cada vendedor.
# Agrupamento por nome do vendedor.

df_vendedor = df_merge.groupby('nome_vendedor')[['valor_venda']].sum().reset_index()
df_vendedor
# Calculo da comissão de 2,5% para cada vendedor

df_vendedor['comissao'] = df_vendedor['valor_venda'] * 0.025
df_vendedor
✅ Gráfico com a comissão dos vendedores de acordo com o cálculo anterior.
# Ordena os dados
df_vendedor_ordenado = df_vendedor.sort_values(by='comissao', ascending=False)

# Cria um mapa de cores com base na comissão
norm = plt.Normalize(df_vendedor_ordenado['comissao'].min(), df_vendedor_ordenado['comissao'].max())
cmap = plt.cm.get_cmap('RdYlGn')  # ou 'viridis', 'coolwarm', etc.
colors = cmap(norm(df_vendedor_ordenado['comissao'].values))

# Gráfico
fig, ax = plt.subplots(figsize=(12, 6))
barras = ax.bar(df_vendedor_ordenado['nome_vendedor'], df_vendedor_ordenado['comissao'], color=colors)

# Títulos e eixos
ax.set_title('💼 Comissão por Vendedor (2,5%)', fontsize=16, pad=20)
ax.set_ylabel('Comissão (R$)', fontsize=12)
ax.set_xlabel('')
ax.set_ylim(0, df_vendedor_ordenado['comissao'].max() + 500)
plt.xticks(rotation=30, ha='right', fontsize=10)

# Rótulos nas barras
for barra in barras:
    altura = barra.get_height()
    ax.text(barra.get_x() + barra.get_width()/2, altura + 50,
            f"R$ {altura:,.2f}", ha='center', fontsize=9, color='black')

# Grid leve
ax.yaxis.grid(True, linestyle='--', alpha=0.3)
ax.set_axisbelow(True)

plt.tight_layout()
plt.show()
✅ Transformação da coluna `data_venda` em formato de data e adição de uma nova coluna `ano` com o ano da venda.
df_merge['data_venda'] = pd.to_datetime(df_merge['data_venda'], format='%d/%m/%Y') # Converte em um formato conhecido pelo Pandas e expecifica o formato da data na coluna indicada
df_merge['ano'] = df_merge['data_venda'].dt.year # Extrai apenas o ano da coluna indicada
df_merge[['data_venda', 'ano']].head()
✅ Cálculo do total de vendas por ano.
# Agrupamento da coluna ano para o cálculo  do `valor_venda`.

df_merge.groupby('ano')['valor_venda'].sum().reset_index()

✅ Gráfico de pizza com a distribuição percentual de vendas por categorias.
# Agrupar os dados de vendas por categoria
df_categoria_pizza = df_merge.groupby('categoria_produto')['valor_venda'].sum().sort_values(ascending=False)

# Cores e explosão na maior fatia
cores = ['#2ca02c', '#1f77b4', '#ff7f0e', '#d62728']
explode = [0.05 if i == 0 else 0 for i in range(len(df_categoria_pizza))]

# Plot do gráfico de pizza
plt.figure(figsize=(9, 7))
plt.pie(
    df_categoria_pizza,
    labels=df_categoria_pizza.index,
    explode=explode,
    autopct=lambda p: f'{p:.1f}%\n(R$ {p * df_categoria_pizza.sum() / 100:,.0f})',
    startangle=90,
    colors=cores,
    textprops={'fontsize': 11}
)

plt.title('Distribuição de Vendas por Categoria', fontsize=15)
plt.axis('equal')
plt.tight_layout()
plt.show()
**✅ Insights baseados nas análises realizadas anteriormente**
# Ditribuição de vendas concentrada entre os valores: 0 a 500 reais e 1500 a 2000
        # O que sugere uma maior aceitação desses produtos
        # Acima de 3000 reais, pela baixa verificara sugere um público mais especifico ou 
            # menor demanda.
# 3 marcas mais expressivas → Brastemp(R$ 99.208), Samsung (R$ 76.490) e Consul (R$ 60.687)
# 3 marcas menos expressivas → LG (R$ 679), Sony (R$ 699), Philco (R$ 1.937)

    # Desempenho das marcas apresentam grande variação, indicando possíveis problemas:
        # Público-alvo
        # Preço
        # Marketing
        # Baixa aceitação
# Diferença na comissão dos vendedores pode indicar:
    # Tipo de produto vendido
    # Região a qual está ligada

# OBS: investigar quais produtos mais lucrativos de cada região 
#      e se estão sendo estimulados pelos vendedores
# Eletroportáteis representam pouco mais de 5% das vendas, esse problema pode ser devido:
    # Público-alvo
    # Marketing
    # Preço

# Uma opção de solução podeira ser:
    # Promoções sazonais
    # Adição de combos
    # Outro incetivos, como: beneficios, como utilizar...
df_vendedor.to_csv('relatorio_vendedor.csv', index=False)
df_categoria.to_csv('relatorio_categoria.csv', index=False)
#### **🛠️ 5 → Estruturação utilizando banco de dados relacional**
✅ Bibliotecas → `PostgreSQL`, `psycopg2`, `ipython-sql`
✅ Conexão ao PostgreSQL 
✅ Consultas com SQL   
# Conexão com banco de dados
try:
    conn = psycopg2.connect(
        host="localhost",
        database="vendas_xpto",
        user="postgres",
        password="%Robim1997%"
    )
    print("Conexão bem sucedida!")
except Exception as e:
    print("Erro ao conectar ao banco de dados: ", e) 
#criando um cursor

crsr = conn.cursor()
✅ Criação da tabelas `vendas_final` via SQL  
✅ Inserção dos dados de venda linha a linha a partir do DataFrame
# Carregar extensões SQL 
%reload_ext sql

# Conexão com o banco de dados
%sql postgresql://postgres:%Robim1997%@localhost/vendas_xpto
%%sql

CREATE TABLE IF NOT EXISTS public.vendas_final 
(
    id serial PRIMARY KEY,
    cod_produto TEXT NOT NULL,            
    nome_produto TEXT NOT NULL,           
    Categoria TEXT NOT NULL,              
    valor_venda numeric(18, 2) NOT NULL,  
    venda_final numeric(18, 2),           
    nome_vendedor TEXT NOT NULL,          
    data_venda DATE NOT NULL
);

tabela = [df_merge]

for df in tabela:  
    for index, row in df.iterrows():  
        try:          
            query = """
                INSERT INTO vendas_final (cod_produto, nome_produto, categoria, valor_venda, venda_final, nome_vendedor, data_venda)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
            """
            crsr.execute(query, (
                row['cod_produto'], row['nome_produto'], row['categoria'],
                row['valor_venda'], row['venda_final'], row['nome_vendedor'], row['data_venda']
            ))
        except Exception as e:
            print(f"Erro na linha {index} do DataFrame: {e}")

conn.commit()
print("Dados salvos com sucesso!")

crsr.close()
# Desfaz as mudanças em caso de erro → `conn.rollback()`
# O ipython-sql foi desenvolvido para funcionar com versões anteriores do PrettyTable.
# pip install prettytable==2.4.0

import prettytable
print(prettytable.__version__)
**5.1 → Consultas ao banco de dados `vendas_final`**
# total de vendas por categoria
%%sql
SELECT categoria, SUM(valor_venda) AS total_vendas
FROM vendas_final
GROUP BY categoria
ORDER BY total_vendas DESC; 
# 3 vendedores que mais venderam
%%sql
SELECT nome_vendedor, SUM(valor_venda) AS total_vendas
FROM vendas_final
GROUP BY nome_vendedor
ORDER BY total_vendas DESC
LIMIT 3;
# Média final de venda
%%sql
SELECT AVG(venda_final)::numeric(18,2) AS media_valor_final
FROM vendas_final;
# Produtos com maior faturamento
%%sql
SELECT nome_produto, SUM(venda_final) AS total_venda_final
FROM vendas_final
GROUP BY nome_produto
ORDER BY total_venda_final DESC
LIMIT 5;
# Receita por ano
%%sql
SELECT 
    EXTRACT(YEAR FROM data_venda) AS ano,
    SUM(venda_final)::numeric(10,2) AS total_venda_final
FROM vendas_final
GROUP BY ano
ORDER BY ano DESC;
