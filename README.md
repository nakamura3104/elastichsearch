# elastichsearch

# logstash 
```
input {
  file {
    path => "/path/to/your/csvfile.csv"
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}

filter {
  # ダブルクォーテーションの削除
  mutate {
    gsub => [
      "message", "\"", ""
    ]
  }

  # CSVとしてパース
  csv {
    separator => ","
    columns => ["column1", "column2", "column3"]
  }

  mutate {
    split => { "column2" => " " }
    add_field => {
      "field1" => "%{[column2][0]}"
      "field2" => "%{[column2][1]}"
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}

```
