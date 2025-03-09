# 🚀 Projeto: Integração SQL com LangChain e OpenAI

## 📖 Descrição

Este projeto demonstra como integrar o LangChain com um banco de dados MySQL para converter perguntas em consultas SQL, executar as queries e transformar os resultados em respostas em linguagem natural utilizando a API da OpenAI. A abordagem é modular e facilmente personalizável.

![alt text](image.png)
---

## 🔍 Como esta solução pode ser um diferencial no mundo dos negócios

### Visão e Propósito:
  A solução foi desenvolvida para transformar a maneira como as empresas acessam e utilizam seus dados. Em um mundo onde a informação é o ativo mais valioso, permitir que qualquer colaborador – mesmo aqueles sem habilidades técnicas avançadas – faça perguntas em linguagem natural e receba respostas precisas aumenta a agilidade na tomada de decisões estratégicas.  
  - **Propósito:** Democratizar o acesso a informações essenciais, eliminando barreiras técnicas e acelerando a inovação e a competitividade.

### Integração de Tecnologias Avançadas:
  Através da combinação de Python, LangChain, MySQL e a API da OpenAI, a solução automatiza o processo de conversão de perguntas em consultas SQL, executa essas consultas e transforma os resultados em insights claros e acionáveis.  
  - **Processo Automatizado:**  
    - **Entrada:** Usuários formulam perguntas em linguagem natural.  
    - **Processamento:** O sistema converte essas perguntas em consultas SQL precisas utilizando modelos de linguagem avançados.  
    - **Execução:** A consulta é executada no banco de dados, retornando os dados relevantes.  
    - **Saída:** O resultado é convertido em uma resposta em linguagem natural, facilitando o entendimento e a ação.
- **Simplicidade e Modularidade:**  
  A estrutura em pipelines permite que a solução seja facilmente ajustada e ampliada conforme as necessidades do negócio evoluem, garantindo escalabilidade e manutenção sem complicações.

### Benefícios Diretos para o Negócio:
  - **Acesso Rápido a Insights:**  
    Permite que os gestores e equipes obtenham informações críticas do banco de dados com agilidade, sem depender de especialistas em SQL.
  - **Decisões Baseadas em Dados:**  
    Facilita a análise de dados e a extração de insights, melhorando a qualidade das decisões estratégicas e operacionais.
  - **Redução de Custos Operacionais:**  
    Automatiza processos que, de outra forma, demandariam tempo e conhecimento técnico especializado, liberando recursos para outras áreas críticas.
  - **Melhoria na Comunicação Interna:**  
    A tradução de dados brutos em linguagem natural torna as informações acessíveis a todos os níveis hierárquicos, promovendo uma cultura de dados na empresa.
  
## Solução Concreta:
  Este sistema permite que empresas, de pequeno a grande porte, integrem seus bancos de dados com ferramentas de Inteligência Artificial, transformando a forma como a informação é consultada e utilizada, tornando o processo mais intuitivo, rápido e seguro.

## 🔧 Pré-requisitos

- **Python 3.9 ou superior**
- **MySQL** com o banco de dados `chinook` configurado e rodando em `localhost:3306`
- Arquivo **`.env`** com as seguintes variáveis (substitua pelos seus valores reais):
  ```env
  OPENAI_API_KEY=your_openai_api_key_here
  DB_PASSWORD=your_mysql_password_here

## 🛠️ Instalação
 **1.Clone o repositório:**
```
git clone https://github.com/seu-usuario/seu-projeto.git
cd seu-projeto
```
**2.Crie e ative um ambiente virtual:**
```
python -m venv venv
# Linux/macOS:
source venv/bin/activate

# Windows:
.\venv\Scripts\activate
```
**3.Instale as dependências:**
```
pip install -r requirements.txt
```
## 💻 Estrutura do Código ##
A seguir, cada parte do código é explicada e comentada:

**1. Carregamento de Variáveis de Ambiente**
Utiliza o dotenv para carregar as variáveis do arquivo .env de forma segura, evitando expor informações sensíveis no código.
```
import os
from dotenv import load_dotenv, find_dotenv

# 🚀 Carrega as variáveis de ambiente do arquivo .env (override=True sobrescreve variáveis já definidas)
load_dotenv(find_dotenv(), override=True)
OPENAI_KEY = os.environ['OPENAI_API_KEY']   # Chave da API da OpenAI
DB_PASSWORD = os.environ['DB_PASSWORD']       # Senha do banco MySQL
```

**2. Configuração da Conexão com o Banco de Dados**
Monta a string de conexão (URI) para o MySQL e inicializa a conexão com o banco chinook.
```
from langchain_community.utilities import SQLDatabase

# Monta a string de conexão com o banco MySQL
db_uri = f"mysql+mysqlconnector://root:{DB_PASSWORD}@localhost:3306/chinook"
# Cria a conexão com o banco
db = SQLDatabase.from_uri(db_uri)

# Função que retorna informações sobre as tabelas do banco (o "esquema")
def get_schema(_):
    return db.get_table_info()
```
**O que isso faz:**
Ele configura e abre uma conexão com o banco de dados, e define uma função para pegar informações (esquema) do banco.

## 3. Gerando Consultas SQL com LangChain ##
Utilizamos um modelo de linguagem (LLM) para transformar uma pergunta em uma consulta SQL. Para isso, definimos um template que orienta o modelo.
```
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI

# Template para gerar a query SQL a partir do esquema do banco e da pergunta
template = """
Based on the table schema below, write a SQL query that would answer the user's question:
{schema}

Question: {question}
SQL Query
"""
# Criamos o objeto que vai formatar o template
chat_prompt = ChatPromptTemplate.from_template(template)

# Inicializa o modelo de linguagem da OpenAI
llm = ChatOpenAI()

# Cria o pipeline que gera a consulta SQL
sql_chain = (
    RunnablePassthrough.assign(schema=get_schema)  # Injeta o esquema do banco
    | chat_prompt                                   # Preenche o template com a pergunta
    | llm.bind(stop="\nSQL Result:")                # Gera a query SQL com a ajuda do modelo
    | StrOutputParser()                             # Converte a saída para uma string
)

# Exemplo de como gerar uma query
sql_chain.invoke({"question": "how many artists are there?"})
```
**O que isso faz:**
O código usa o modelo da OpenAI para "traduzir" uma pergunta (por exemplo, "Quantos artistas existem?") em uma consulta SQL que pode ser executada no banco de dados.

## 4. Executando a Query e Gerando Resposta Natural ##
Depois de gerar a consulta SQL, executamos essa query no banco e usamos outro template para transformar a resposta em uma explicação em linguagem natural.

```
# Template para criar a resposta final em linguagem natural
template = """Based on the table schema below, question, sql query, and sql response, write a natural language response:
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}"""
prompt_response = ChatPromptTemplate.from_template(template)

# Função que executa a query no banco e retorna o resultado
def run_query(query):
    return db.run(query)

# Cria um pipeline completo que junta geração da query, execução e resposta final
full_chain = (
    RunnablePassthrough.assign(query=sql_chain).assign(
        schema=get_schema,                                  # Injeta o esquema do banco novamente
        response=lambda variables: run_query(variables["query"]),  # Executa a query e pega o resultado
    )
    | prompt_response                                       # Preenche o template final
    | llm                                                 # Gera a resposta natural com o LLM
)

# Exemplo: Gera uma resposta final para a pergunta
full_chain.invoke({"question": "Quanto foi o faturamento total da empresa?"})
```
**O que isso faz:**
Ele executa a consulta SQL que foi gerada, pega o resultado do banco e usa o modelo para criar uma resposta em linguagem natural. Assim, você tem a pergunta, a consulta SQL e a resposta de forma integrada.

Agredecimento especial ao professor [@EduardoInocencio](https://github.com/EduardoVitorInocencio) pelo artigo compartilhado que possibilitou o entendimento e a aplicação deste projeto.

[Artigo professor Eduardo](https://www.linkedin.com/pulse/automatizando-consultas-sql-com-python-langchain-e-mysql-inocencio-oubpf/)