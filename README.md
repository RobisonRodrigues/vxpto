### **📦  Projeto: Otimização e Análise de Vendas XPTO**

#### 🧩 Cenário
A empresa XPTO está em expansão e busca modernizar suas estratégias comerciais com base em análises de dados. 

Para isso, o departamento de vendas forneceu dois arquivos:
- `base_vendas.csv`: → lista detalhada de vendas realizadas.
- `categorias_valores.csv`: → percentuais de aumento dos produtos por categoria.

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
```
### **🔍 2 → Verificação do encoding dos arquivos CSV**

**obs:** Biblioteca `chardet` foi utilizada para a verificação.
```python
rawdata = open('base_vendas.csv', 'rb').read()
encoding = chardet.detect(rawdata)['encoding']
print('Encoding detectado:', encoding)
```
![Image](https://github.com/user-attachments/assets/7fe36f13-ef26-4c8e-ae6f-3052ad469a5c)
```python
rawdata = open('categorias_valores.csv', 'rb').read()
encoding = chardet.detect(rawdata)['encoding']
print('Encoding detectado: ', encoding)
```
![Image](https://github.com/user-attachments/assets/b83dd718-74dc-46db-99e9-13e626e9f20a)
##### **2.1 → Criação do Dataframe a partir do encoding detectado**
```python
df_vendas = pd.read_csv('base_vendas.csv', encoding="ISO-8859-1", sep=';')
```
```python
df_categorias = pd.read_csv('categorias_valores.csv', encoding="utf-8", sep=',')
```
### **💻 3 → Exploração da base de dados**
```python
# Verificando as linhas iniciais
df_vendas.head()
```
![Image](https://github.com/user-attachments/assets/0fe9ee66-7ea0-4d16-9135-c8d44f752ae1)
```python
# Verificando as linhas iniciais
df_categorias.head(2)
```
![image](https://github.com/user-attachments/assets/dcc0bd84-9352-4767-afd6-4c9ebeed2822)
```python
df_categorias.rename(columns={'Categoria': 'categoria'}, inplace=True)
```
```python
# Tipos de dados das colunas
df_vendas.dtypes
```
```python
# Tipos de dados das colunas
df_categorias.dtypes
```
```python
# Informações da base de dados, que incluem: colunas, linhas e nulos
df_vendas.info()
```
```python
# Informações da base de dados, que incluem: colunas, linhas e nulos
df_categorias.info()
```
```python
# Numero de nulos
df_vendas.isnull().sum()
```
```python
# Numero de nulos
df_categorias.isnull().sum()
```
```python
# Estatística das colunas númericas de vendas
df_vendas['valor_venda'].describe()
```
```python
# Estatistica das colunas textuais do dataframe de vendas.
df_vendas.describe(include=['object'])
```
```python
# Estatística das colunas númericas de categorias
df_categorias.describe()
```
```python
# Estatistica das colunas textuais do dataframe de categorias.
df_categorias.describe(include=['object'])
```
```python
# Valores únicos da coluna 'Categoria' do dataframe de categorias.
lista_categoria = list(df_categorias['categoria'].unique())
print(lista_categoria)
```
```python
# Valores únicos da coluna 'categoria_produto' do dataframe de vendas.
lista_produto = list(df_vendas['categoria_produto'].unique())
print(lista_produto)
```
```python
# Valores únicos da coluna 'nome_vendedor' do dataframe de vendas.
lista_vendedor = list(df_vendas['nome_vendedor'].unique())
print(lista_vendedor)
```
```python
# Valores únicos da coluna 'categoria_produto' do dataframe de vendas, fazendo a contagem nos resultados.
df_vendas['categoria_produto'].value_counts()
```
```python
# Estatística da categoria do produto por valor de venda.
df_vendas.groupby('categoria_produto')['valor_venda'].describe()
```
### **📊 4 → Análise Gráfica**

Foram utilizados gráficos para visualizar os dados e identificar padrões, tendências e possíveis valores atípicos (outliers)

##### **4.1 Histograma de distribuição dos Valores de Venda**

**Resumo do fluxo**


✅ Quantidade de vendas em cada faixa de preço.
✅ Padrões de vendas, identificando faixas com maior e menor frequência.
```python
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
```
![Image](https://github.com/user-attachments/assets/24633e09-da26-4ec1-bf88-85b3d0c82233)
##### 4.2 Gráfico de barras por marca de produto

**Resumo do fluxo**

✅ Agrupamento por marca.
✅ Soma dos valores de venda por marca.
✅ Ordenamento dos valores de venda para analisar melhor.
```python
df_marca = df_vendas.groupby('marca_produto')['valor_venda'].sum().sort_values()
print(df_marca)
```
```python
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
```
![Image](https://github.com/user-attachments/assets/02186d31-ef4e-4854-a473-4a54eb30c67b)
##### 4.3 Boxplot por categoria 

✅ Visualizar dispersão dos valores. ✅ Observa padrões e variabilidade.
```python
plt.figure(figsize=(10, 5))
sns.boxplot(data=df_vendas, x='categoria_produto', y='valor_venda', palette='Set2')
plt.title('Distribuição de Valores de Venda por Categoria')
plt.xlabel('Categoria')
plt.ylabel('Valor da Venda (R$)')
plt.xticks(rotation=30)
plt.tight_layout()
plt.show()
```
![Image](https://github.com/user-attachments/assets/b90f87b1-4063-48e8-b317-b6de41a0488e)
### **🧩 5 → Alteração na tabela (df_merge).**

✅ Alteração da coluna `valor` para `percentual_aumento`.
```python
df_categorias.rename(columns={'Valor': 'percentual_aumento'}, inplace=True)

df_merge = df_vendas.merge(df_categorias, left_on="categoria_produto", right_on="categoria", how="left")
df_merge.head()
```
✅ Inserção de uma nova coluna no `df_merge` chamado `venda_final` aplicando o percentual de aumento.
```python
df_merge['venda_final'] = df_merge['valor_venda'] * (1 + df_merge['percentual_aumento'] / 100)
df_merge.head()
```
✅ Visualização agrupado por `categoria` gerando um gráfico comparando os valores originais e com aumento.
```python
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
```
![Image](https://github.com/user-attachments/assets/78fb9b63-1267-4aec-8296-d1be8bae43da)
✅ Calculo e adição de uma nova coluna `comissão` com 2,5% para cada vendedor.
```python
# Agrupamento por nome do vendedor.

df_vendedor = df_merge.groupby('nome_vendedor')[['valor_venda']].sum().reset_index()
df_vendedor
```
![Image](https://github.com/user-attachments/assets/c5d42bbf-490f-42a9-ae44-892cec9d4cc8)
```python
# Calculo da comissão de 2,5% para cada vendedor

df_vendedor['comissao'] = df_vendedor['valor_venda'] * 0.025
df_vendedor
```
![Image](https://github.com/user-attachments/assets/13eb7785-2eb2-47d8-862b-775957eabbac)
✅ Gráfico com a comissão dos vendedores de acordo com o cálculo anterior.
```python
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
```
![Image](https://github.com/user-attachments/assets/afa6d16d-9835-4ce4-8729-0a377112bff9)

✅ Transformação da coluna `data_venda` em formato de data e adição de uma nova coluna `ano` com o ano da venda.
```python
df_merge['data_venda'] = pd.to_datetime(df_merge['data_venda'], format='%d/%m/%Y') # Converte em um formato conhecido pelo Pandas e expecifica o formato da data na coluna indicada
df_merge['ano'] = df_merge['data_venda'].dt.year # Extrai apenas o ano da coluna indicada
df_merge[['data_venda', 'ano']].head()
```
![Image](https://github.com/user-attachments/assets/5ca34ed4-72af-40fc-bdfa-c809e3d9375e)
✅ Cálculo do total de vendas por ano.
```python
# Agrupamento da coluna ano para o cálculo  do `valor_venda`.

df_merge.groupby('ano')['valor_venda'].sum().reset_index()
```
![Image](https://github.com/user-attachments/assets/3ea7022e-9050-48ee-ae12-3ceaa4829d2e)
✅ Gráfico de pizza com a distribuição percentual de vendas por categorias.
```python
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
```
![Image](https://github.com/user-attachments/assets/0c0f6cc3-8fc3-4c33-9e5d-e1187f5934eb)
**✅ Insights baseados nas análises realizadas anteriormente**
```python
# Ditribuição de vendas concentrada entre os valores: 0 a 500 reais e 1500 a 2000
        # O que sugere uma maior aceitação desses produtos
        # Acima de 3000 reais, pela baixa verificara sugere um público mais especifico ou 
            # menor demanda.
```
```python
# 3 marcas mais expressivas → Brastemp(R$ 99.208), Samsung (R$ 76.490) e Consul (R$ 60.687)
# 3 marcas menos expressivas → LG (R$ 679), Sony (R$ 699), Philco (R$ 1.937)

    # Desempenho das marcas apresentam grande variação, indicando possíveis problemas:
        # Público-alvo
        # Preço
        # Marketing
        # Baixa aceitação
```
```python
# Diferença na comissão dos vendedores pode indicar:
    # Tipo de produto vendido
    # Região a qual está ligada

# OBS: investigar quais produtos mais lucrativos de cada região 
#      e se estão sendo estimulados pelos vendedores
```
```python
# Eletroportáteis representam pouco mais de 5% das vendas, esse problema pode ser devido:
    # Público-alvo
    # Marketing
    # Preço

# Uma opção de solução podeira ser:
    # Promoções sazonais
    # Adição de combos
    # Outro incetivos, como: beneficios, como utilizar...
```
```python
df_vendedor.to_csv('relatorio_vendedor.csv', index=False)
df_categoria.to_csv('relatorio_categoria.csv', index=False)
```
#### **🛠️ 6 → Estruturação utilizando banco de dados relacional**
✅ Bibliotecas → `PostgreSQL`, `psycopg2`, `ipython-sql`
✅ Conexão ao PostgreSQL 
✅ Consultas com SQL
```python
# Conexão com banco de dados (PostgreSQL)
try:
    conn = psycopg2.connect(
        host="localhost",
        database="nome_base_dados",
        user="postgres",
        password="sua_senha"
    )
    print("Conexão bem sucedida!")
except Exception as e:
    print("Erro ao conectar ao banco de dados: ", e)
```
```python
#criando um cursor

crsr = conn.cursor()
```
✅ Criação da tabelas `vendas_final` via SQL  
✅ Inserção dos dados de venda linha a linha a partir do DataFrame
```python
# Carregar extensões SQL 
%load_ext sql
```
```python
# Conexão com o banco de dados
%sql postgresql://postgres:´senha´@localhost/´nome_banco_dados´
%%sql
```
```python
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
```
```python
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
```
```python
# Desfaz as mudanças em caso de erro → `conn.rollback()`
```
```python
# O ipython-sql foi desenvolvido para funcionar com versões anteriores do PrettyTable.
# pip install prettytable==2.4.0

import prettytable
print(prettytable.__version__)
```
**6.1 → Consultas ao banco de dados `vendas_final`**
```python
# total de vendas por categoria
```
```python
%%sql
SELECT categoria, SUM(valor_venda) AS total_vendas
FROM vendas_final
GROUP BY categoria
ORDER BY total_vendas DESC;
```
![Image](https://github.com/user-attachments/assets/73ac5d1b-8c20-4f92-97bd-3555a6ac446f)
```python
# 3 vendedores que mais venderam
```
```python
%%sql
SELECT nome_vendedor, SUM(valor_venda) AS total_vendas
FROM vendas_final
GROUP BY nome_vendedor
ORDER BY total_vendas DESC
LIMIT 3;
```
![Image](https://github.com/user-attachments/assets/d5b54b38-fdc1-4c7a-826a-415e4cb9adfb)
```python
# Média final de venda
```
```python
%%sql
SELECT AVG(venda_final)::numeric(18,2) AS media_valor_final
FROM vendas_final;
```
![Image](https://github.com/user-attachments/assets/ac23e10e-14d9-4576-9e34-4cb67da5a689)
```python
# Produtos com maior faturamento
```
```python
%%sql
SELECT nome_produto, SUM(venda_final) AS total_venda_final
FROM vendas_final
GROUP BY nome_produto
ORDER BY total_venda_final DESC
LIMIT 5;
```
![Image](https://github.com/user-attachments/assets/1d957643-e8af-4f8e-af13-4742f0c11be0)
```python
# Receita por ano
```
```python
%%sql
SELECT 
    EXTRACT(YEAR FROM data_venda) AS ano,
    SUM(venda_final)::numeric(10,2) AS total_venda_final
FROM vendas_final
GROUP BY ano
ORDER BY ano DESC;
```
![Image](https://github.com/user-attachments/assets/fecc71e2-bf40-43f8-856a-615d88aa2628)
