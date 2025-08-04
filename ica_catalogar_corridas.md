## Identificar e Catalogar Amostras no ICA

<img width="1920" height="1080" alt="2" src="https://github.com/user-attachments/assets/af16289e-5bca-4c64-a2b4-a6f94bb7e0b6" />

### Execução

```bash
## Catalogar amostras no ICA
python3 gensus_ica.py -K build-db {GenSUS_StudyCode.csv}

## Gerar duas bases de dados utilizada para os demais commandos
## - gensus-ica.gensus-sample-data.{flag de data}.parquet
## - gensus-ica.sample-data.{flag de data}.parquet
```

<img width="1920" height="1080" alt="3" src="https://github.com/user-attachments/assets/647a38d6-ae1c-4dec-894b-015be12409a0" />

### Verifica de Inconsistências

```bash
## Reporta inconstências da corrida: nome desformatado e defasada
python3 gensus_ica.py -K check-basecall gensus-ica.sample-data.{flag de data}.parquet
```

```bash
## Reporta inconstências da amostra: nome desformatado, duplicados, e inconsistente com o nome da corrida
python3 gensus_ica.py -K check-sample gensus-ica.sample-data.{flag de data}.parquet
```
