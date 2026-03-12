# 📊 Social Media Graph Analytics com Neo4j

## 📌 Contexto do Problema

Uma startup de análise de mídias sociais deseja desenvolver um novo produto capaz de gerar **insights sobre engajamento, popularidade de conteúdo e comunidades de interesse** em uma plataforma social.

O objetivo deste projeto é construir um **protótipo funcional utilizando banco de dados em grafos (Neo4j)** para analisar dados de interações do Twitter e responder perguntas complexas sobre:

* Engajamento em conteúdos
* Popularidade de tópicos
* Sentimento dos usuários
* Comunidades de interesse

O dataset utilizado foi obtido no Kaggle e contém milhares de tweets relacionados a diferentes **entidades (marcas, jogos e empresas)**.

---

# 🧠 Por que usar Grafos?

Bancos relacionais funcionam bem para dados tabulares, porém **redes sociais são naturalmente grafos**.

Usuários, tweets e temas estão conectados por relações complexas.

O uso do **Neo4j** permite:

* Navegar facilmente entre conexões
* Detectar padrões sociais
* Identificar comunidades
* Analisar propagação de conteúdo

Grafos são amplamente usados por empresas como:

* Netflix
* LinkedIn
* Uber
* Amazon

para sistemas de recomendação e análise de redes.

---

# 📂 Dataset Utilizado

Dataset: **Twitter Sentiment Analysis**

Contém aproximadamente **74 mil tweets** com as seguintes colunas:

| Coluna      | Descrição                                 |
| ----------- | ----------------------------------------- |
| entity_id   | ID da entidade                            |
| entity_name | Nome da entidade (marca, empresa ou jogo) |
| sentiment   | Sentimento do tweet                       |
| tweet_text  | Texto do tweet                            |

Sentimentos possíveis:

* Positive
* Negative
* Neutral
* Irrelevant

---

# 🧩 Modelo de Grafo

## Nodes

```
User (opcional)
Tweet
Entity
Sentiment
Keyword
```

### Propriedades

```
(:Tweet {id, text})
(:Entity {id, name})
(:Sentiment {type})
(:Keyword {word})
```

---

## Relacionamentos

```
(:Tweet)-[:ABOUT]->(:Entity)

(:Tweet)-[:HAS_SENTIMENT]->(:Sentiment)

(:Tweet)-[:MENTIONS]->(:Keyword)
```

---

# 1️⃣ Instalar bibliotecas

```
!pip install neo4j pandas
```

---

# 2️⃣ Importar bibliotecas

```
import pandas as pd
from neo4j import GraphDatabase
```

---

# 3️⃣ Upload do dataset

No Colab você pode subir o arquivo manualmente.

```
from google.colab import files
uploaded = files.upload()
```
---

# 4️⃣ Carregar o CSV

Esse dataset não tem header, então precisamos definir manualmente.

```
columns = ["entity_id", "entity_name", "sentiment", "tweet"]
df = pd.read_csv("twitter_training.csv", names=columns)
df.head()
```

---

# 5️⃣ Verificar dados

Esse dataset não tem header, então precisamos definir manualmente.

```
df.info()
df['sentiment'].value_counts()
```

---

# 6️⃣ Conectar ao Neo4j

Se você estiver usando:
- Neo4j Desktop
- ou Neo4j AuraDB

```
uri = "bolt://localhost:7687"
user = "neo4j"
password = "password"

driver = GraphDatabase.driver(uri, auth=(user, password))
```

---

# 7️⃣ Função para inserir dados no grafo

```
def create_tweet(tx, entity, sentiment, text):

    query = """
    MERGE (e:Entity {name:$entity})
    MERGE (s:Sentiment {type:$sentiment})

    CREATE (t:Tweet {text:$text})

    MERGE (t)-[:ABOUT]->(e)
    MERGE (t)-[:HAS_SENTIMENT]->(s)
    """

    tx.run(query, entity=entity, sentiment=sentiment, text=text)
```

---

# 8️⃣ Enviar dados para o Neo4j

⚠️ Importante: não envie os 74k tweets de uma vez no Colab.
Vamos usar uma amostra.

```
sample = df.sample(2000)

with driver.session() as session:
    for index, row in sample.iterrows():
        session.execute_write(
            create_tweet,
            row["entity_name"],
            row["sentiment"],
            row["tweet"]
        )

driver.close()
```

---

# 9️⃣ Executar queries de análise

Agora podemos criar funções para consultas.

```
def top_entities(tx):

    query = """
    MATCH (:Tweet)-[:ABOUT]->(e:Entity)
    RETURN e.name AS entity, COUNT(*) AS tweets
    ORDER BY tweets DESC
    LIMIT 10
    """

    result = tx.run(query)

    return [record.data() for record in result]
```

Executar:

```
with driver.session() as session:
    result = session.read_transaction(top_entities)

result
```

---

# 🔟 Análise de sentimentos

```
def sentiment_analysis(tx):

    query = """
    MATCH (:Tweet)-[:HAS_SENTIMENT]->(s:Sentiment)
    RETURN s.type, COUNT(*) AS total
    """

    result = tx.run(query)

    return [record.data() for record in result]
```

Executar:

```
with driver.session() as session:
    result = session.read_transaction(sentiment_analysis)

result
```
