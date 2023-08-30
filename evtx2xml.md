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

```python
import xml.dom.minidom
import argparse
from concurrent.futures import ProcessPoolExecutor
from Evtx.Evtx import Evtx
from Evtx.Views import evtx_file_xml_view

def record_to_xml(record):
    return xml.dom.minidom.parseString(record.xml()).toprettyxml()

def evtx_to_xml(evtx_file_path, output_xml_file_path):
    with Evtx(evtx_file_path) as log:
        with open(output_xml_file_path, 'w', encoding='utf-8') as xml_output:
            with ProcessPoolExecutor() as executor:
                for xml_record in executor.map(record_to_xml, log.records()):
                    xml_output.write(xml_record)

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Convert EVTX to XML")
    parser.add_argument("input_evtx", help="Path to the input EVTX file")
    parser.add_argument("output_xml", help="Path to the output XML file")

    args = parser.parse_args()

    evtx_to_xml(args.input_evtx, args.output_xml)
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
