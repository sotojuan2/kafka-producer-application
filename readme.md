

# Data Contracts
Confluent Schema Registry adds support for tags, metadata, and rules, which together support the concept of a Data Contract. 
## Set up environment
```shell
docker-compose up -d
```
## Let's generate a schema with associated rules and metadata

````
curl --silent -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{
            "schemaType": "AVRO",
            "schema": "{\"type\":\"record\",\"namespace\":\"org.matias\",\"name\":\"Message\",\"fields\":[{\"name\":\"greet\",\"type\":\"string\"}]}",
            "metadata": {
                "properties" : {
                    "owner": "Bob Jones",
                    "email": "bob@acme.com"
                }
            },
            "ruleSet": { 
                "domainRules": [
                  {
                    "name": "checkLen",
                    "kind": "CONDITION",
                    "type": "CEL",
                    "mode": "WRITE",
                    "expr": "size(message.greet) == 4",
                    "onFailure": "ERROR"
                  }
                ]}}' \
    http://localhost:8081/subjects/message-value/versions
````


## Retrieving subjects

````
curl --silent -X GET http://localhost:8081/subjects | jq
````

## Retrieving a specific schema

````
curl --silent -X GET http://localhost:8081/subjects/message-value/versions/1 | jq
````

## Find a schema where owner email address is xxx
`````
curl --silent -X GET http://localhost:8081/subjects/message-value/versions/6 | jq 'select (.metadata.properties.email == "bob@acme.com")'
`````

# Run KafkaAvroProducerApplication
The KafkaAvroProducerApplication requieres two properties:
- The configuration file. You have examples on ./configuration folder
- The input data file ./input.txt

# Event-Condition-Action Rules
## Evolving the schema with rules to send the error to DLQ

````
curl --silent -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{
            "schemaType": "AVRO",
            "schema": "{\"type\":\"record\",\"namespace\":\"org.matias\",\"name\":\"Message\",\"fields\":[{\"name\":\"greet\",\"type\":\"string\"}]}",
            "metadata": {
                "properties" : {
                    "owner": "Bob Jones",
                    "email": "bob@acme.com"
                }
            },
            "ruleSet": { 
                "domainRules": [
                  {
                    "name": "checkLen",
                    "kind": "CONDITION",
                    "type": "CEL",
                    "mode": "WRITE",
                    "expr": "size(message.greet) == 4",
                    "onFailure": "DLQ"
                  }
                ]}}' \
    http://localhost:8081/subjects/message-value/versions
````
# Read Rules
## Evolving the schema with rules executed on consumer


````
curl --silent -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '@data.json' \
    http://localhost:8081/subjects/message-value/versions
````
## Getting Into a Docker Container’s Shell
docker exec -it kafka-producer-application-schema-registry-1 bash
## Consume CLI using CEL_FIELD Rule Executor
``````
kafka-avro-console-consumer --topic message --bootstrap-server kafka:29092 --property schema.registry.url=http://localhost:8081  --property rule.executors=populateFullName --property rule.executors.populateFullName.class=io.confluent.kafka.schemaregistry.rules.cel.CelFieldExecutor --from-beginning
``````
# CP Data Contract

[Data Contract Documentation](https://docs.confluent.io/platform/current/schema-registry/fundamentals/data-contracts.html)


