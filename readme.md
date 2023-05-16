

# Data Contracts
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


````
curl --silent -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '@data.json' \
    http://localhost:8081/subjects/message-value/versions
````

docker exec -it kafka-producer-application-schema-registry-1 bash

kafka-avro-console-consumer --topic message --bootstrap-server kafka:29092 --property schema.registry.url=http://localhost:8081  --property rule.executors=populateFullName --property rule.executors.populateFullName.class=io.confluent.kafka.schemaregistry.rules.cel.CelFieldExecutor --from-beginning

## Evolving the schema with rules

````
curl --silent -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data @data.json \
    http://localhost:8081/subjects/message-value/versions
````

