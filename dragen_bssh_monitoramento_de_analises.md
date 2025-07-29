## Monitoramento de Análises

Este módulo usa o mesmo script `ica_sync_analyses_status.R`, agora em **modo batch**, para conferir o status de um conjunto de amostras listadas num TSV.  O fluxo foi revisado para destacar a decisão de atualizar ou não o banco local de projetos/analyses.

<img width="2090" height="1046" alt="image" src="https://github.com/user-attachments/assets/d967bcd8-5f08-43d3-ac5a-7a577f3766f3" />

---

### 1. Entrada

| Flag / Arquivo | Função                                                                                                                              |   |                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------- | - | --------------------------------------------------------------- |
| `-k API_KEY`   | chave de acesso à ICA (Illumina Connected Analytics)                                                                                |   |                                                                 |
| `-i BATCH_TSV` | TSV com colunas `userReference`, `id`, `project_id` — criado na submissão de um lote Dragen/ICA                                     |   |                                                                 |
| `--no-sync`    | *Opcional.* **Pula** a sincronização e usa apenas o cache local                                                                     |   | *Opcional.* **Pula** a sincronização e usa apenas o cache local |
| `-v`           | modo verboso                                                                                                                        |   |                                                                 |

> **Origem do `BATCH_TSV`** — este arquivo é gerado especificamente para o *monitoramento*, durante a submissão de um lote ao Dragen/ICA, e salvo em `input_qc/unprocessed/`.

### 2. Execução típica. 

```bash
Rscript src/ica_sync_analyses_status.R \
  -k $ICA_API_KEY \
  -i input_qc/unprocessed/lote.tsv
  # adicione --no-sync se não quiser consultar a API
```

### 3. Passos internos

| Etapa                          | Descrição                                                                                                                                 |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Leitura inicial**            | Carrega `projects.parquet/tsv` e `analyses/<slug>.parquet/tsv`                                                                            |
| **Decisão – Atualizar banco?** | Se `--no-sync` **não** estiver presente, chama a **ICA API** para só os projetos do lote (via *sync interna*) e regrava os Parquets/TSVs. |
| **Consulta de status**         | Para cada linha do lote consulta o cache atualizado e gera a tabela de status.                                                            |

### 4. Saídas

| Saída                  | Onde aparece                                                           |
| ---------------------- | ---------------------------------------------------------------------- |
| **Caches atualizados** | `data/projects.{parquet,tsv}`  e  `data/analyses/<slug>.{parquet,tsv}` |
| **Status detalhado**   | Impresso no console (e salvo com `--output <arquivo.tsv>`)             |
| **Resumo por status**  | Contagem de SUCCEEDED / FAILED / … no console                          |

### 5. Exemplo de log (*offline* `--no-sync`)

```
ℹ Pulando sincronização (--no-sync).
ℹ Última sync de pr_gu.parquet: 2025‑07‑29 10:08:27

userReference   id                                   CA    status    pipeline.code                         startDateUTC-3
GS-008‑001681‑1 77298c2c‑7374‑445b‑8fb1‑120d493b93c5 PR-GU SUCCEEDED DRAGEN_Germline_Whole_Genome_4‑3‑13   2025‑07‑28 11:27:31
...
ℹ Resumo de status BATCH:
• SUCCEEDED: 48
• FAILED:    0
• ABORTED:   0
```

---

**Dicas**

* Rode **sem** `--no-sync` quando quiser garantir que o cache está fresco.
* Use `--output lote_status.tsv` para exportar a tabela detalhada e anexar ao ticket ou planilha.
