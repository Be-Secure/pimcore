# Configuration Configuration

Following aspects need to be considered in index configuration:  

## General Configuration Options
In the `config_options` area general elasticsearch settings can be made - like hosts, index settings, etc. 

##### `client_config`
- `logging`: (deprecated, for Elasticsearch 7 only) `true`/`false` to activate logging of elasticsearch client
- `indexName`: index name to be used, if not provided tenant name is used as index name 

##### `index_settings`
Index settings that are used when creating a new index. They are passed 1:1 as 
settings param to the body of the create index command. Details see 
also [elasticsearch Docs](https://www.elastic.co/guide/en/elasticsearch/client/php-api/current/_index_management_operations.html). 

#### `es_client_name` (for Elasticsearch 8 only)
Elasticsearch 8 client configuration takes place via 
[Pimcore Elasticsearch Client Bundle](https://github.com/pimcore/elasticsearch-client) and has two parts.

1) Configuring an elasticsearch client in separate configuration
```yaml
# Configure an elasticsearch client 
pimcore_elasticsearch_client:
    es_clients:
        default:
            hosts: ['elastic:9200']
            username: 'elastic'
            password: 'somethingsecret'
            logger_channel: 'pimcore.elasticsearch'    
```

2) Define the client name to be used by an elasticsearch tenant. This will be done via the `es_client_name` configuration 
   in the `config_options`. 

##### `es_client_params` (deprecated, for Elasticsearch 7 only)
- `hosts`: Array of hosts of the elasticsearch cluster to use. 
- `timeoutMs`: optional parameter for setting the client timeout (frontend) in milliseconds.
- `timeoutMsBackend`: optional parameter for setting the client timeout (CLI) in milliseconds. This value is typically higher than ``timeoutMs``.

##### `synonym_providers`
Specify synonym providers for synonym filters defined in filter section of index settings. 
For details see [Synonyms](./02_Synonyms.md).

#### Sample Config
```yml
pimcore_ecommerce_framework:
    index_service:
        tenants:
            MyEsTenant:
                worker_id: Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Worker\ElasticSearch\DefaultElasticSearch8
                config_id: Pimcore\Bundle\EcommerceFrameworkBundle\IndexService\Config\ElasticSearch
                
                config_options:
                    client_config:
                        logging: false
                        indexName: 'ecommerce-demo-elasticsearch'

                    # elasticsearch client name, for Elasticsearch 8 only
                    es_client_name: default
                    
                    # deprecated, for Elasticsearch 7 only
                    es_client_params:
                        hosts:
                            - '%elasticsearch.host%'
                        timeoutMs: 20000, # 20 seconds
                        timeoutMsBackend: 120000 # 2 minutes

                    index_settings:
                        number_of_shards: 5
                        number_of_replicas: 0
                        max_ngram_diff: 30
                        analysis:
                            analyzer:
                                my_ngram_analyzer:
                                    tokenizer: my_ngram_tokenizer
                                allowlist_analyzer:
                                    tokenizer: standard
                                    filter:
                                      - allow_list_filter
                            tokenizer:
                                my_ngram_tokenizer:
                                    type: nGram
                                    min_gram: 2
                                    max_gram: 15
                                    token_chars: [letter, digit]
                            filter:
                                allow_list_filter:
                                    type: keep
                                    keep_words:
                                      - was
                                      - WAS
```


## Data Types for attributes
The type of the data attributes needs to be set to elasticsearch data types..

```yml
pimcore_ecommerce_framework:
    index_service:
        tenants:
            MyEsTenant:
                attributes:
                    name:
                        locale: '%%locale%%'
                        type: keyword
```

In addition to the `type` configuration, you also can provide custom mappings for a field. If provided, these mapping 
configurations are used for creating the mapping of the elasticsearch index.

You can also skip the `type` and `mapping`, then ES will try to create dynamic mapping. 

```yml

pimcore_ecommerce_framework:
    index_service:
        tenants:
            MyEsTenant:
                attributes:
                    name:
                        locale: '%%locale%%'
                        type: text
                        options:
                            mapping:
                                type: text
                                store: true
                                index: not_analyzed
                                fields:
                                    analyzed:
                                        type: text
                                        analyzer: german
                                    analyzed_ngram:
                                        type: text
                                        analyzer: my_ngram_analyzer
``` 
