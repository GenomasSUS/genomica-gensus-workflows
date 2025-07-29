## Sincronização de Bancos Locais

Este módulo mantém sempre atualizada a cópia local dos projetos e análises disponíveis na plataforma ICA, nos modos **dragen** e **bs**. O diagrama abaixo ilustra, de maneira resumida, o fluxo de execução:

![Fluxograma de Sincronização](figuras/flow_sync.png)

### Descrição do Fluxo

1. **Entrada**

   * `-k API_KEY` : chave de acesso à API ICA
   * `-i MODE` : modo de sincronização (`dragen` ou `bs`)
   * `-v` : habilita saída detalhada (verbose)

2. **Execução do Script**

   ```bash
   Rscript src/ica_sync_analyses_status.R \
     -k $ICA_API_KEY \
     -i dragen \
     -v
   ```

3. **Processos Internos**

   * `sync_projects()`  – busca projetos na API, filtra pelos nomes‑padrão e grava cache `data/projects.parquet` e `.tsv`.
   * `sync_analyses()`  – para cada projeto:

     * consulta análises na API (`list_project_analyses()`),
     * cria ou atualiza Parquet/TSV em `data/analyses/<slug>.{parquet,tsv}`.

   * Gera um **TSV contendo informações sobre todas as análises do modo**:

     * `data/analyses/dragen.tsv` (se `-i dragen`)
     * `data/analyses/bssh.tsv`  (se `-i bs`)

4. **Saída**

   Após a execução, o diretório `data/` conterá:

   * **Projetos**

     * `data/projects.parquet`
     * `data/projects.tsv`
   * **Análises (por CA)**

     * `data/analyses/<slug>.parquet`
     * `data/analyses/<slug>.tsv`

   * **Análises do modo**

     * `data/analyses/dragen.tsv` *ou* `data/analyses/bssh.tsv`

5. **Resumo de Status (console)**

   O script imprime um log de progresso e, ao final, um resumo por status. Exemplo (modo DRAGEN):

   ```
   ℹ Sincronizando projetos DRAGEN: MG-BH, PA-BE, ...
   ℹ [1/8] MG-BH: 1111 análises (+0 novas e 0 atualizadas)
   ...
   ✔ Resultados DRAGEN salvos em .../data/analyses/dragen.tsv

   ℹ Resumo de status DRAGEN:
   • SUCCEEDED: 11500
   • FAILED: 220
   • ABORTED: 130
   ```

   Esse resumo facilita a verificação rápida das condições das análises.

---

**Observação:**
Para executar apenas a leitura do cache sem consultar a API, utilize a flag `--no-sync`. Para mais exemplos ou informações, entre em contato com o mantenedor do repositório.
