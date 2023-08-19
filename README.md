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
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}

```
