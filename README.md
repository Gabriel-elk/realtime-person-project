# Pipeline de Dados com Airflow, Kafka, Spark e Cassandra

## Descrição

Este projeto implementa um pipeline de dados completo utilizando:

* Apache Airflow para orquestração
* Apache Kafka para ingestão e streaming
* Apache Spark para processamento
* Apache Cassandra como banco de dados não relacional

A aplicação consome dados da API pública:
https://randomuser.me/api/

Os dados são coletados em tempo real durante um período de 60 segundos (configurável), processados e persistidos no Cassandra.

---

## Arquitetura

O fluxo do pipeline segue as etapas abaixo:

1. Airflow agenda e executa a DAG
2. Dados são consumidos da API RandomUser
3. Kafka recebe os dados e atua como buffer de streaming
4. Spark processa os dados
5. Os dados são armazenados no Cassandra

Todos os serviços são containerizados com Docker e compartilham a mesma rede:
networkproject

---

## Tecnologias Utilizadas

* Apache Airflow
* Apache Kafka
* Apache Spark
* Apache Cassandra
* Docker
* Python

---

## Estrutura do Projeto

A estrutura inclui:

* DAGs do Airflow
* Scripts de consumo e processamento
* Configuração do Kafka
* Configuração do Cassandra
* Dockerfile e docker-compose.yml

---

## Como Executar o Projeto

### 1. Inicializar os containers

Certifique-se de que o Docker Desktop está em execução. (todos os comandos devem ser executados nas respectivas pastas em que os dockerfiles estão)

```bash
docker-compose up -d
```

---

### 2. Parar os containers

```bash
docker-compose down
```

---

### 3. Rebuild dos containers (após alterações)

```bash
docker-compose up --build -d
```

---

## Execução do Consumer

### 1. Build da imagem

```bash
docker build -t kafka-spark-cassandra-consumer .
```

### 2. Criar o container

```bash
docker run --name client -dit --network server_networkproject kafka-spark-cassandra-consumer
```

### 3. Executar o consumer

```bash
python consumer_stream.py --mode initial
```

---

## Verificação no Cassandra

Acesse o Cassandra:

```bash
docker exec -it cassandra cqlsh
```

Execute:

```sql
USE dados_usuarios;
SELECT * FROM tb_usuarios;
```

---

## Observações

* O tempo de coleta padrão é de 60 segundos, mas pode ser ajustado.
* A API RandomUser possui limitações de uso na versão gratuita.
* Todos os serviços estão na mesma rede Docker para comunicação interna.
* Para automatizar a execução do consumer, descomente a última linha do Dockerfile e recrie a imagem.

---

## Possíveis Melhorias

* Adicionar monitoramento (Prometheus/Grafana)
* Implementar tratamento de falhas no pipeline
* Adicionar testes automatizados
* Criar dashboard para visualização dos dados
* Configurar CI/CD

---

## Autor

Gabriel Turmena
