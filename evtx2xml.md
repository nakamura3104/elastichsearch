### .evtx ファイルを XML に変換する方法

1. 必要なモジュールをインストールします：

    `requirements.txt`:
    ```
    python-evtx
    ```

    インストールコマンド:
    ```
    pip install -r requirements.txt
    ```

2. `.evtx` ファイルを XML に変換するスクリプト：

    `evtx_to_xml.py`:
    ```python
    import sys
    from Evtx.Evtx import Evtx

    def evtx_to_xml(evtx_file_path, output_xml_file_path):
        with open(output_xml_file_path, 'w', encoding='utf-8') as xml_output:
            with Evtx(evtx_file_path) as log:
                for record in log.records():
                    xml_output.write(record.xml())

    if __name__ == "__main__":
        if len(sys.argv) < 3:
            print(f"Usage: {sys.argv[0]} [input.evtx] [output.xml]")
            sys.exit(1)

        input_evtx = sys.argv[1]
        output_xml = sys.argv[2]

        evtx_to_xml(input_evtx, output_xml)
    ```

    ```
    import sys
    from Evtx.Evtx import Evtx
    from concurrent.futures import ProcessPoolExecutor

    def record_to_xml(record):
        return record.xml()

    def evtx_to_xml(evtx_file_path, output_xml_file_path):
        xml_list = []
        with Evtx(evtx_file_path) as log:
            with ProcessPoolExecutor() as executor:
                xml_list = list(executor.map(record_to_xml, log.records()))

        with open(output_xml_file_path, 'w', encoding='utf-8') as xml_output:
            for xml_record in xml_list:
                xml_output.write(xml_record)

    if __name__ == "__main__":
        if len(sys.argv) < 3:
            print(f"Usage: {sys.argv[0]} [input.evtx] [output.xml]")
            sys.exit(1)

        input_evtx = sys.argv[1]
        output_xml = sys.argv[2]

        evtx_to_xml(input_evtx, output_xml)
    ```

    実行コマンド:
    ```
    python evtx_to_xml.py your_input.evtx your_output.xml
    ```


4. logstashのフィルター

```
filter {
  xml {
    source => "message"
    target => "parsed_xml"
    force_array => false
  }
}

```
