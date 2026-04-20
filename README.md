# Pipeline de Dados com Airflow, Kafka, Spark e Cassandra

## Descrição


#### (OBS)
Eu fiz questão de subir a maior parte dos arquvios pois quero facilitar o entendimento dos meus colegas da área e estudantes da área que gostariam de implementar e utilizar meu projeto em seus perfils, meu objetivo com esse projeto é me apriximar cada vez mais de atuar na área e ganhar cada vez mais confiança para engressar na área.
---


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

* Adicionar monitoramento (Prometheus/Grafana) ✅
* Implementar tratamento de falhas no pipeline
* Adicionar testes automatizados
* Criar dashboard para visualização dos dados 
* Configurar CI/CD

---
17/04/2026
## Monitoramento com Prometheus e Grafana

Foi implementada uma camada de monitoramento com **Prometheus** e **Grafana** para acompanhar a saúde dos serviços e o comportamento do banco Cassandra em tempo real.

Nesta implementação, o monitoramento do Cassandra foi realizado com **JMX Exporter em modo Standalone**, ou seja, em um container separado do banco. Essa abordagem foi escolhida para evitar adicionar mais carga diretamente na instância do Cassandra, mantendo o banco mais isolado e reduzindo o impacto no consumo de recursos.

### Componentes adicionados

* **Prometheus** para coleta de métricas
* **Grafana** para visualização dos dashboards
* **JMX Exporter Standalone** para exposição das métricas do Cassandra

### Métricas monitoradas

Entre as métricas observadas no projeto, destacam-se:

* Status do serviço (**up/down**)
* Uso de memória heap do Cassandra
* Espaço consumido pelo Cassandra
* Número de clientes conectados ao banco

### Observações sobre a implementação

* O exporter do Cassandra foi configurado em modo **Standalone**
* O Prometheus realiza o scrape das métricas expostas pelo exporter
* O Grafana consome os dados do Prometheus para geração dos dashboards
* Essa abordagem facilita a observabilidade do ambiente sem acoplar o exporter diretamente ao processo principal do banco

---

No Cassandra, a coleta de métricas foi realizada com **JMX Exporter em modo Standalone**, executado em um container separado do banco. Essa abordagem foi adotada para reduzir o acoplamento da instrumentação ao processo principal do Cassandra e evitar impacto desnecessário no consumo de recursos do banco.

### Ajustes realizados no Cassandra Exporter

Para expor métricas relacionadas ao uso de memória da JVM, foram adicionados os seguintes parâmetros no `cassandra-exporter`:

```yaml
- pattern: 'java.lang<type=Memory><HeapMemoryUsage>used:'
  name: jvm_memory_heap_used_bytes

- pattern: 'java.lang<type=Memory><HeapMemoryUsage>max:'
  name: jvm_memory_heap_max_bytes
```

## Possíveis Correções Ajustes e Pontos de Atenção

Durante a configuração do ambiente, alguns pontos podem precisar de ajuste dependendo do ambiente:

### Broker(nome do kafka) pode não ter tópicos adicionados, rode esse comando na VM do broker para fazer esse ajuste (rode caso ocorra um erro de implementação da conecção broker/airflow
Primeiro tentar realizar sem implementar o bash no broker)

```bash
    "
    /opt/kafka/bin/kafka-topics.sh --bootstrap-server broker:29092 --create --if-not-exists --topic _kafka_topic --partitions 1 --replication-factor 1 &&
    echo 'Tópico criado com sucesso!'
    "
```

## Verifique a implementação de rede o Cliente e o servidor precisam estar no mesmo ambiente de rede, para fazer o cliente começar a rodar caso ele não suba automáticamente pelo comando cmd do yaml abaixo (ele vai estar comentado dentro do dockerflile descomente essa opção) ou rode o comando abaixo dentro da pasta client:
``` yaml
# Comando de entrada padrão
#CMD ["python", "consumer_stream.py", "--mode", "append"]
```
```bash
docker run --name client -dit --network server_networkproject kafka-spark-cassandra-consumer
```

```bash (dentro do contêiner na pasta que está o arquivo .py)
python consumer_stream.py --mode initial
```

## Dentro do Entrypoint.sh você vai se deparar com as configurações e permissões do usuário admin 
```
if [ ! -f "/opt/airflow/airflow.db" ]; then
  airflow db init && \
  airflow users create \
    --username admin \
    --firstname admin \
    --lastname admin \
    --role Admin \ role de admin (não mexer)
    --email yourmail@gmail.com \ seu e-mail para configuração de smtp e envios de e-mails
    --password yourPassword teu password que vai ser usado para logar no airflow
fi
```
Autor : Gabriel Turmena

Contato: gtcarvalho2005@gmail.com
