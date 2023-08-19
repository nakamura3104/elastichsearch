# elastichsearch

# logstash 
```
input {
  s3 {
    access_key_id => "YOUR_ACCESS_KEY"
    secret_access_key => "YOUR_SECRET_KEY"
    region => "us-west-1" # 適切なリージョンを指定してください
    bucket => "your-s3-bucket-name"
    interval => 60 # ポーリング間隔(秒)
    # その他のオプションも設定可能
  }

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
  }

  # 分割結果を新しいフィールドに保存
  if [column2][0] {
    mutate {
      add_field => { "field1" => "%{[column2][0]}" }
    }
  }
  if [column2][1] {
    mutate {
      add_field => { "field2" => "%{[column2][1]}" }
    }
  }

  grok {
    match => { "message" => "%{HOSTNAME:host} %{INT:port}: \[%{WORD:country_code}\] %{DAY:day} %{MONTH:month} %{MONTHDAY:monthday} %{TIME:time} %{YEAR:year}: %{GREEDYDATA:first_message}  \(%{INT:event_id}\) - %{GREEDYDATA:second_message}" }
  }


  # もし不要なら分割されたフィールドを削除
  mutate {
    remove_field => ["column2"]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}

```
