🚀 Projeto: Integração SQL com LangChain e OpenAI
📚 Descrição
Este projeto demonstra como integrar o LangChain com um banco de dados MySQL para converter perguntas em consultas SQL, executar essas consultas e retornar respostas em linguagem natural utilizando o ChatOpenAI. A estrutura modular permite expandir ou adaptar o pipeline conforme a necessidade.

🛠️ Funcionalidades
Carregamento seguro de variáveis de ambiente (API keys, senhas, etc.)
Geração dinâmica de consultas SQL a partir de perguntas em linguagem natural
Execução de queries no banco de dados MySQL
Geração de respostas finais em linguagem natural

🔍 Estrutura do Código

import os
from dotenv import load_dotenv, find_dotenv
# Importa classes do LangChain (versões customizadas/forks)
from langchain_core.prompts import ChatPromptTemplate
from langchain_community.utilities import SQLDatabase
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

# 🚀 Carregar variáveis de ambiente
# Carrega o arquivo .env para obter as chaves de API e senhas sem expô-las no código
load_dotenv(find_dotenv())
OPENAI_KEY = os.environ['OPENAI_API_KEY']   # Chave da API da OpenAI
DB_PASSWORD = os.environ['DB_PASSWORD']       # Senha do MySQL

# (Opcional) Importação do prompt do prompt_toolkit - evite conflito de nomes se não for utilizado
from prompt_toolkit import prompt

# 💬 Definição do template para gerar a consulta SQL
# O template recebe o esquema da tabela e a pergunta do usuário para formular a consulta SQL
template = """
Based on the table schema below, write a SQL query that would answer the user's question:
{schema}

Question: {question}
SQL Query
"""
# Cria o objeto de prompt a partir do template
prompt = ChatPromptTemplate.from_template(template)
# Exemplo de formatação do template (apenas para teste)
prompt.format(schema='my schema', question="how many users are there!")

# 🔗 Configuração da conexão com o MySQL
# Monta a URI de conexão utilizando o usuário root e a senha carregada do .env
db_uri = f"mysql+mysqlconnector://root:{DB_PASSWORD}@localhost:3306/chinook"
# Inicializa a conexão com o banco utilizando a classe SQLDatabase
db = SQLDatabase.from_uri(db_uri)

# 🔍 Função para obter o esquema das tabelas do banco de dados
def get_schema(_):
    return db.get_table_info()

# 🤖 Inicialização do modelo de linguagem da OpenAI
llm = ChatOpenAI()

# 🛠️ Criação do pipeline SQL para gerar a consulta
sql_chain = (
    # Passa o esquema do banco para o pipeline
    RunnablePassthrough.assign(schema=get_schema)
    # Aplica o template para gerar a consulta SQL
    | prompt
    # Utiliza o modelo de linguagem para completar a consulta (parando antes de retornar o resultado)
    | llm.bind(stop="\nSQL Result:")
    # Converte a saída para uma string
    | StrOutputParser()
)
# Exemplo de invocação do pipeline para gerar uma consulta com uma pergunta específica
sql_chain.invoke({"question": "how many artists are there?"})

# 💡 Definição de um template para gerar a resposta final em linguagem natural
# Esse template usa o esquema, a pergunta, a query gerada e a resposta SQL para formular uma resposta natural
template = """Based on the table schema below, question, sql query, and sql response, write a natural language response:
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}"""
# Cria o objeto de prompt para a resposta final
prompt_response = ChatPromptTemplate.from_template(template)

# 🔧 Função que executa a query construída anteriormente no banco de dados
def run_query(query):
    return db.run(query)

# 🔄 Criação do pipeline completo que integra a consulta SQL e a resposta em linguagem natural
full_chain = (
    # Passa a consulta SQL gerada anteriormente para o pipeline
    RunnablePassthrough.assign(query=sql_chain).assign(
        # Re-obtem o esquema da base de dados
        schema=get_schema,
        # Executa a query e obtém a resposta do banco
        response=lambda variables: run_query(variables["query"]),
    )
    # Aplica o template para gerar a resposta final
    | prompt_response
    # Utiliza o modelo de linguagem para formatar a resposta
    | llm
)

# 🚀 Execução do pipeline completo com a pergunta final do usuário
full_chain.invoke({"question": "Quanto foi o faturamento total da empresa?"})
📌 Considerações Finais
Segurança: Variáveis sensíveis são carregadas via .env, evitando exposição direta no código.
Modularidade: O uso de pipelines facilita a manutenção e a expansão do projeto.
Personalização: Você pode alterar os templates e funções auxiliares para se adequar a diferentes tipos de queries ou bancos de dados.
🎯 Como Executar
Certifique-se de ter o arquivo .env com as seguintes variáveis:

OPENAI_API_KEY=your_openai_api_key_here
DB_PASSWORD=your_mysql_password_here
Instale as dependências necessárias (por exemplo, pip install python-dotenv mysql-connector-python langchain ...).

Execute o script:

python seu_script.py

Agradecimento especial ao professor [@EduardoInocencio](https://github.com/EduardoVitorInocencio)