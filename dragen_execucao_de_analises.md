## Processamento de Novas Análises DRAGEN

Este módulo recebe um lote de amostras em formato TSV, envia cada análise ao pipeline **DRAGEN / Nextflow** na ICA, e gera artefatos para rastreabilidade. O diagrama abaixo resume as etapas.

<img width="3718" height="2168" alt="dragen_submission" src="https://github.com/user-attachments/assets/962ce42a-972b-4dca-a8bb-ce3e9b58d0e4" />


---

### 1. Entrada

| Flag / Arquivo | Descrição                                                                                                                                      |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `-k API_KEY`   | Chave de acesso à ICA (obrigatório)                                                                                                            |
| `-i INPUT_TSV` | TSV validado, salvo em input_qc/unprocessed/, contendo, por linha, os campos:<br>`RunName`, `GensusCAId`, `GensusCA`, `GensusSample`, `FastqId`, `FastqListId`, `BatchFolderId` |
| `--dry-run`    | Opcional. Prepara o payload mas **não** envia para a API                                                                                       |
| `--verbose`    | Opcional. Exibe logs detalhados                                                                                                                |

### 2. Execução do Script

```bash
Rscript src/run_dragen_analyses.R \
  -k $ICA_API_KEY \
  -i input_qc/unprocessed/batch.tsv \
  --verbose    # opcional
  # --dry-run  # simulação, se necessário
```

### 3. O que acontece por dentro

1. **Leitura & Validação** do TSV.
2. **Construção do payload JSON** via `build_dragen_nextflow_body()`.
3. **Envio** à rota `POST /analysis:nextflow` com `send_dragen_nextflow()` (retry exponencial p/ time‑outs ou 5xx).
4. **Geração de artefatos** para cada `RunName`:

   * `data/sent_analyses/<RunName>/<RunName>.tsv`
   * Flag‑files `.SUCCESS`, `.ERROR`, `.DRYRUN`
   * Logs em `data/sent_analyses/<RunName>/logs/`
5. **Movimentação do TSV do lote**: se houve sucesso (e não for *dry‑run*), o arquivo original é movido de `input_dragen/unprocessed/` para `input_dragen/processed/`.

### 4. Saídas

| Tipo                | Localização                                                  |
| ------------------- | ------------------------------------------------------------ |
| **TSV por RunName** | `data/sent_analyses/<RunName>/<RunName>.tsv`                 |
| **TSV (backup)**    | `input_qc/processed/<RunName>.tsv`                           |
| **Flag‑files**      | `.SUCCESS`, `.ERROR` ou `.DRYRUN` na pasta de cada `RunName` |
| **Logs**            | Arquivos `.log` timestampados em `.../logs/`                 |
| **Console**         | Linha de progresso por amostra + resumo final                |

### 5. Exemplo resumido de execução real

```bash
Rscript src/run_dragen_analyses.R -k xxxx \
  -i input_dragen/unprocessed/gensus-submission.GS-RP-S061_GS-RP-F061.20250724.txt
```

Saída (trecho):

```
ℹ Preparando envio de 128 amostras
ℹ Preparando envio de análise para projeto SP-RP (128 amostras)
✔ [1/128] … ID=d14bdd83-bb97-450b-9271-da3314a57fdd
✔ [2/128] … ID=b39abd9e-b55c-46da-8717-b7d2df85adf3
⋯
✔ [128/128] … ID=03d567d6-f8db-4e80-9917-93abeb8df8a2
ℹ Todas 128 análises enviadas com sucesso.
✔ Movido input para: .../input_dragen/processed/gensus-submission.GS-RP-S061_GS-RP-F061.20250724.txt
```

No *dry‑run*, as IDs são marcadas como `dry-run` e nenhum POST é realizado.

---

**Observação**

* Use `--dry-run` para validar o lote sem consumos de créditos.
* Os artefatos seguem a estrutura por **RunName** para facilitar auditoria.
