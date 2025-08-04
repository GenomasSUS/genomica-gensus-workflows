## Identificar, Catalogar e Preparar Amostras no ICA

Coleção de scripts para localizar, catalogar e preparar o processamento DRAGEN de amostras de genoma completo no ambiente ICA da Illumina.
O fluxo geral de trabalho está apresentado no esquema abaixo.

<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/af16289e-5bca-4c64-a2b4-a6f94bb7e0b6" />

De forma simplificada, as corridas e amostras disponíveis no ICA são catalogadas pelo `build-db` gerando duas bases de dados (`gensus-sample-data` e `sample-data`), os quais são utilizados para avaliar e identificar inconsistências para serem revisadas (pelos comandos `check-basecall` e `check-sample`) e preparar a submissão de lote de amostras ao DRAGEN da ICA (`prep-analysis`).

### Catalogar Amostras e Corridas

O seguinte commando é utilizado para catalogar as amostras e corridas disponíveis no ICA e requer somente os seguintes argumentos:
- `APIKEY`, chave de acesso a API do ICA.
- `GenSUS_StudyCode.csv`, tabela com dados básicos do ID dos projetos e Centro Âncora responsável.

```bash
## Catalogar amostras no ICA
python3 gensus_ica.py -K {APIKEY} build-db {GenSUS_StudyCode.csv}

## Gerar duas bases de dados utilizada para os demais commandos
## - gensus-ica.gensus-sample-data.{flag de data}.parquet
## - gensus-ica.sample-data.{flag de data}.parquet
```

O script explora os basespaces disponíveis para localizar todas as chamadas de bases disponíveis.
As chamadas de base são identificadas pelo arquivo `Demultiplex_Stats.csv`, uma tabela listando as amostras inclusas e o número de reads identificados.
Estes são baixados localmente e qualificados como **defasados** caso haja outra chamada de bases mais recente com o mesmo id de corrida (`RunId`).
As amostras disponíveis são extraídas dos arquivos `Demultiplex_Stats.csv` e os Ids de amostras corrigidos quando possível.
Desta coleção de amostra são localizadas os `FASTQ` associados e as amostras de corrida _shallow_ e _full_ são consolidadas de acordo com seu GenSUS Id.
O fluxo de trabalho do script está representado no flowchart abaixo.

<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/647a38d6-ae1c-4dec-894b-015be12409a0" />

### Verificar de Inconsistências

Após catalogar as amostras e corridas, podemos verificar identificar inconsistências de nomenclatura e as corridas defasadas que podem afetar a submissão adequadas das amostras para processamento.
O comando `check-basecall` busca por problemas de identificação das corridas/chamada de bases e aponta chamadas de base defasadas.
E `check-sample` identifica inconsistências de identificação de amostras, identificadores duplicados, e conflitos com o nome da corrida.
No exemplo abaixo, executamos ambos os comando para buscar problemas e inconsistências.

```bash
## Reporta inconstências da corrida: nome desformatado e defasada
python3 gensus_ica.py -K {APIKEY} check-basecall gensus-ica.sample-data.{flag de data}.parquet

## Reporta inconstências da amostra: nome desformatado, duplicados, e inconsistente com o nome da corrida
python3 gensus_ica.py -K {APIKEY} check-sample gensus-ica.sample-data.{flag de data}.parquet
```

## Prepara Lote de Amostra para DRAGEN

Após verificar inconsistências, podemos selecionar um lote para preparar a submissão DRAGEN.
O comando `list-batch` identifica lotes de amostras com pelo menos duas corridas disponíveis, assumidas como _shallow_ e _full_.
Lotes de amostras são identificados baseado no id das corridas (e.g. GS-RP-S032_GS-RP-F032).
Exemplo abaixo lista todos os lotes prontos para submissão, o número de amostras contidas, e o número de amostra com estimativa de cobertura igual ou acima de 5x, 28x, e 30x.

```bash
## Catalogar amostras no ICA
python3 gensus_ica.py -K {APIKEY} list-batch gensus-ica.gensus-sample-data.{flag de data}.parquet
```

A submissão do lote selecionado pode ser preparada utilizando o commando `prep-analysis` conforme o exemplo abaixo.
O comando cria uma pasta no ICA dos centros âncoras responsáveis pelas amostras inclusas, submete `.fastqlist` para cada amostra e linka os arquivos `FASTQ` para execução do DRAGEN.
Ao final, o comando gera uma tabela com os dados necessários para execução das análises DRAGEN.

```bash
## Catalogar amostras no ICA
python3 gensus_ica.py -K {APIKEY} prep-analysis gensus-ica.gensus-sample-data.{flag de data}.parquet {id_lote}

## Gerar duas bases de dados utilizada para os demais commandos
## - gensus-submission.{id_lote}.{flag de data}.txt
```
