# Index Routing

You can use a pattern in your index name based on the value of one of your fields. Here we use the value of the type field in order to name the index:

```bash
output {  
    stdout {codec => rubydebug}  
    elasticsearch {  
        host => "localhost"  
        protocol => "http"  
        index => "%{type}_indexer"
    }
}
```

You can also use several elasticsearch outputs either to the same ES host or to different ES hosts:

```bash
output {  
    stdout {codec => rubydebug}  
    elasticsearch {  
        host => "localhost"  
        protocol => "http"  
        index => "trial_indexer"
    }
    elasticsearch {  
        host => "localhost"  
        protocol => "http"  
        index => "movie_indexer"
    }
}
```

Or maybe you want to route your documents to different indices based on some variable:

```bash
output {  
    stdout {codec => rubydebug}
    if [type] == "trial" {
        elasticsearch {  
            host => "localhost"  
            protocol => "http"  
            index => "trial_indexer"
        }
    } else {
        elasticsearch {  
            host => "localhost"  
            protocol => "http"  
            index => "movie_indexer"
        }
    }
}
```

UPDATE

The syntax has changed a little bit in Logstash 2 and 5:

```bash
output {  
    stdout {codec => rubydebug}
    if [type] == "trial" {
        elasticsearch {  
            hosts => "localhost:9200"  
            index => "trial_indexer"        }
    } else {
        elasticsearch {  
            hosts => "localhost:9200"  
            index => "movie_indexer"
        }
    }
}
```
