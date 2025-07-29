## Monitoramento de Análises

Este módulo verifica o status de um lote específico de análises em *modo batch* usando o mesmo script de sincronização. Ele pode operar **online** (consultando a ICA API) ou **offline** (apenas com os arquivos locais) dependendo da flag `--no-sync`. O diagrama a seguir mostra o fluxo completo:

![Fluxograma de Monitoramento](figuras/flow_monitoramento.png)

### Descrição do Fluxo

1. **Entrada**

   | Flag           | Função                                                                     |
   | -------------- | -------------------------------------------------------------------------- |
   | `-k API_KEY`   | chave de acesso à API ICA                                                  |
   | `-i BATCH_TSV` | caminho para o arquivo TSV com colunas `userReference`, `id`, `project_id` |
   | `--no-sync`    | *Opcional.* Se presente, **não** consulta a API; usa somente o cache local |
   | `-v`           | saída detalhada (verbose)                                                  |

2. **Execução do Script**

   ```bash
   Rscript src/ica_sync_analyses_status.R \
     -k $ICA_API_KEY \
     -i input_qc/unprocessed/lote.tsv \
     --no-sync      # opcional
   ```

3. **Processos Internos**

   1. **Lê o cache local**

      * `data/projects.parquet` / `.tsv`
      * `data/analyses/<slug>.parquet` / `.tsv`
   2. **Decisão `--no-sync`**

      * Se **ausente** → consulta a **ICA API** apenas para os projetos presentes no lote, atualizando (ou criando) os caches de projetos e análises.
      * Se **presente** → trabalha exclusivamente com os arquivos locais existentes.
   3. **Gera a tabela de status** para cada linha do `batch.tsv` combinando cache + API.

4. **Saída**

   * **Atualiza** os mesmos caches locais (`projects.parquet/tsv`, `analyses/<slug>.parquet/tsv`).
   * **Imprime** no *console*:

     * Tabela detalhada com colunas `userReference`, `id`, `CA`, `status`, `pipeline.code`, `startDateUTC-3`.
     * Resumo por status (contagem de `SUCCEEDED`, `FAILED`, etc.).
   * **TSV opcional** com `--output <arquivo>` contendo a mesma tabela de status.

5. **Exemplo de Saída no Console**

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

**Observação:**
Utilize `--no-sync` apenas quando tiver certeza de que o cache local está atualizado. Para mais exemplos ou detalhes de uso, entre em contato com o mantenedor do repositório.
