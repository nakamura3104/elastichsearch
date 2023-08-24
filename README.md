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
 - マッチしないinputがあった場合の処理

```
output {
  if "_grokparsefailure" in [tags] {
    elasticsearch {
      hosts => ["your-elasticsearch-host:9200"]
      index => "grok-failures"
    }
  } else {
    elasticsearch {
      hosts => ["your-elasticsearch-host:9200"]
      index => "normal-index"
    }
  }
}

```

 - ログのタイムスタンプからの取得

```
filter {
  grok {
    match => { "message" => "%{NUMBER:id}  %{TIMESTAMP_ISO8601:logdate} %{HOSTNAME:host} %{WORD:event_source} - - -" }
  }
  date {
    match => [ "logdate", "ISO8601" ]
    target => "@timestamp"
  }
}


# 別のパターン
filter {
  date {
    match => ["year-month-day hour:minute:second", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
  }
}

```


 - マルチラインを１行にする

```
input {
  # ここでは例としてfileプラグインを使用していますが、実際の環境に合わせて適切なプラグインを選択してください。
  file {
    path => "/path/to/your/logfile.log"
    start_position => "beginning"
    sincedb_path => "/dev/null" # テスト用途の場合、常に最初からファイルを読むようにする

    # multilineの設定
    codec => multiline {
      # 行が yyyy-mm-dd で始まる場合、新しいイベントの開始として認識します。
      pattern => "^\d{4}-\d{2}-\d{2}"
      negate => true
      what => "previous"
    }
  }
}

filter {
  # 必要に応じてフィルタリングを追加
}

output {
  # 出力設定
}

```

```
#すでにinputでcodecを使っている場合
input {
    file {
        path => "your_log_path.log"
        codec => "plain"
    }
}

filter {
    multiline {
        pattern => "YOUR_PATTERN_HERE"
        what => "previous"
    }
}


```

- ファイル名を追加

```
filter {
  mutate {
    add_field => { "filename" => "%{[@metadata][s3][key]}" }
  }
}

```


OpenSearch（および元のElasticsearch）で.keywordというサフィックスが付いたフィールドを見ることがよくあります。これは、テキストフィールドのデータを異なる方法でインデックス化したものです。以下に詳細を説明します。

テキストフィールド (text):

textフィールドタイプは、フルテキスト検索のために最適化されています。
このフィールドタイプは、入力データを個別の単語（トークン）に分割し、それらをインデックス化します。この分割・インデックス化の過程を「トークン化」と呼びます。
トークン化されたデータは、部分一致検索や検索時のテキスト解析に適していますが、ソートやアグリゲーションには適していません。
キーワードフィールド (keyword):

keywordフィールドタイプは、データをそのままの形でインデックス化します。つまり、トークン化の過程を経ずに、全体としての文字列として保存されます。
これにより、ソート、アグリゲーション、フィルタリング、正確な値の一致検索などの操作が可能になります。
.keywordの役割:

デフォルトのマッピングテンプレートを使用すると、テキストフィールドは通常、textタイプとしてインデックス化されますが、同時にそのフィールドの.keywordという名前でkeywordタイプのマルチフィールドも作成されます。
このようにして、同じデータに対してフルテキスト検索と、ソートやアグリゲーションなどの操作の両方が可能になります。
例えば、messageというフィールドがある場合、そのフィールドをフルテキスト検索するためにはmessageを直接使用し、ソートやアグリゲーションを行うためにはmessage.keywordを使用します。



```
  # _dateparsefailure タグが存在する場合にダミー値を設定
  if "_dateparsefailure" in [tags] {
    ruby {
      code => "
        event.set('@timestamp', LogStash::Timestamp.new(Time.parse('2023-01-01T00:00:00Z')))
      "
    }
  }
```
