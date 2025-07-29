# Emedgene CSV File Builder

Scripts voltados para gerar o arquivo CSV necessário para criação de casos na [Emedgene](https://help.emg.illumina.com/emedgene-analyze-manual/creating_multiple_cases/csv_format_requirements). 

### Authors

* Felipe de Azevedo Oliveira

### Requirements

* Python 3
* Token de acesso a API do RedCap GenSUS
* Token de acesso a API do ICA GenSUS

### Packages used

#### Python
* requests;
* logging;
* pandas;
* io;
* docopt;
* datetime;
* math;
* multiprocessing;
* os;
* sys.

# Usage

O fluxo para criação do arquivo CSV consiste em três scripts distintos: 
* `create_csv_emedgene.py`: Script principal. Responsável por fazer as chamadas e orquestrar as tasks.
* `csvUtils.py`: Fornece as funções que serão utilizadas pelo `create_csv_emedgene.py`. Essas funções tem como objetivo a checagem e captura de diversas informações em cada amostra, como, por exemplo, HPO, informações de parentesco, métricas de qualidade etc.
* `LoggerUtils.py`: Fornece as estruturas de log a serem gerados.

### Flowchart

<img src = "./fig/Fluxograma Fluxo Emedgene.svg">

### Step 1

Confira se possui todas as dependências e pacotes instaladas. Para esse script também é necessário ter os Tokens de acesso para a API da conta GenSUS no ICA e no RedCap.

### Step 2

Criação do arquivo de input. Um arquivo de exemplo com formato esperado está disponível em `example/input_table_example.txt`. Todas as colunas do arquivo devem estar presentes, porém, caso necessário, somente as colunas `GensusCA` (Nome do Centro Âncora) e `GensusSample` (GenSUS ID da amostra) precisam estar preenchidas.

### Step 3
Execução do script `create_csv_emedgene.py`. Para isso são esperados 6 argumentos:

* `--input SAMPLES_TABLE_PATH`: Caminho e nome para o arquivo de input;
* `--output CSV_EMEDGENE_OUTPUT_PATH`: Caminho e nome para criação do arquivo CSV de output. Não adicione a extensão do arquivo.;
* `--ica ICA_TOKEN`: Token de acesso para API do ICA;
* `--redcap REDCAP_TOKEN`: Token de acesso para API do RedCap;
* `--log LOG_OUTPUT_FOLDER`: Diretório onde os arquivos de log devem ser salvos;
* `--parallel NUMBER_PROCESSES`: Número de processos a serem rodados em paralelo.

E existem dois parâmetros opcionais:
* `--qcfile QC_FILE`: Caminho para arquivo de métricas de qualidade;
* `--batchskip BATCH_LIST`: Lista de batche IDs a serem pulados durante a checagem. Os IDs devem ser separados por vírgula.

Exemplo de comando de execução:

```
python3 create_csv_emedgene.py --input exemple/input_table_example.txt --output output/batch_GS-TEST1-F003_GS-TEST1-S003 --ica TOKEN_ICA --redcap TOKEN_REDCAP --log /log/PA-BE --parallel 10
```

Além dos argumentos obrigatórios existe um parâmetro opcional: `--qcfile`. Para essa opção é esperado o caminho para um arquivo de métricas de qualidade obtidas através do processamento com a Dragen. Ao utilizar esta opção, o script não irá buscar os dados no RedCap. Esse arquivo precisa conter todas as amostras já processadas. Um exemplo para a formatação do arquivo esperado por ser encontrada em `example/qc_table_example.csv`.

Exemplo de uso com o parâmetro `--qcfile`:

```
python3 create_csv_emedgene.py --input exemple/input_table_example.txt --output output/batch_GS-TEST1-F003_GS-TEST1-S003 --ica TOKEN_ICA --redcap TOKEN_REDCAP --log /log/PA-BE --parallel 10 --qcfile example/qc_table_example.csv
```

Também é possível passar através do parâmetro opcional `--batchskip` uma lista de batch IDs a serem pulados durante a checagem. Todas as amostras que estão presentes nestes batches automaticamente vão passar sem erros pela checagem de batch.

Exemplo de uso com o parâmetro `--batchskip`:
```
python3 create_csv_emedgene.py --input exemple/input_table_example.txt --output output/batch_GS-TEST1-F003_GS-TEST1-S003 --ica TOKEN_ICA --redcap TOKEN_REDCAP --log /log/PA-BE --parallel 10 --batchskip GS-TEST1-S033_GS-TEST1-F033,GS-TEST2-S040_GS-TEST2-F040
```

Em caso de dúvidas, utilize o comando `python3 create_csv_emedgene.py --help` para mais detalhes.

# Output
Caso alguma das amostras presentes no arquivo de input passe em todas as checagens, um arquivo CSV será gerado, no diretório e com o nome conforme indicado na execução do script. Caso todas as amostras apresentem alguma pendência/conflito, o arquivo não será gerado.

Além do arquivo CSV, também são gerados diversos arquivos de log:
* Log geral (`stdout.log`): Esse log reune todas as informações e linhas dos outros logs;
* Logs individuais para cada amostra (`[ID Amostra].log`): Nesses arquivos estarão presentes as checagens feitas e o seu status para uma única amostra;
* Log Family ID (`family_id_check.log`): Após a checagem por amostra, é feita uma checagem por Family ID. Isso é feito para verificar se todos os membros de um determinado Family ID passaram com sucesso pelas outras checagens. Caso alguma amostra dentro do Family ID tenha falhado, todas as outras serão removidas. Neste log fica descrito quais Family ID seguirão ou não para a análise.

Por último, também é gerado o arquivo `table_errors.tsv`. Este arquivo possui duas colunas: `Sample` e `Avisos`. A primeira indica o GenSUS ID da amostra e a segunda possui uma mensagem indicando o motivo pelo qual aquela amostra falhou em alguma das checagens. Como as amostras podem falhar em múltiplas checagens, é provável que a mesma amostra esteja presente em múltiplas linhas, mas com avisos diferentes.