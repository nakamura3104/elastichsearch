### .evtx ファイルを XML に変換する方法

1. 必要なモジュールをインストールします：

    `requirements.txt`:
    ```
    evtx
    ```

    インストールコマンド:
    ```
    pip install -r requirements.txt
    ```

2. `.evtx` ファイルを XML に変換するスクリプト：

    `evtx_to_xml.py`:
    ```python
    import sys
    import evtx

    def evtx_to_xml(evtx_file_path, output_xml_file_path):
        with open(output_xml_file_path, 'w', encoding='utf-8') as xml_output:
            with evtx.Evtx(evtx_file_path) as log:
                xml_output.write(log.xml())

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
