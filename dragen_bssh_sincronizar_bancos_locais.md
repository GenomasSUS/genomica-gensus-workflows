## Sincronização de Bancos Locais

O script `ica_sync_analyses_status.R` mantém **projetos** e **análises** locais sempre alinhados com a plataforma ICA, nos modos **dragen** e **bs**. Ele:

1. Lê o cache existente (`projects.parquet/tsv` e `analyses/<slug>.parquet/tsv`). — se não existir, cria do zero.
2. Consulta a **ICA API** para os projetos‑alvo, detectando novidades ou atualizações.
3. Atualiza os mesmos arquivos Parquet/TSV e gera um **resumo** por status + um TSV consolidado (`dragen.tsv` ou `bssh.tsv`).

<img width="2892" height="1195" alt="sync_dbs" src="https://github.com/user-attachments/assets/1f1d635f-52b7-4681-a993-caa360cb98b5" />


### 1. Entrada

| Flag         | Descrição             |
| ------------ | --------------------- |
| `-k API_KEY` | Chave de acesso à ICA |
| `-i MODE`    | `dragen` ou `bs`      |
| `-v`         | Verbose opcional      |

### 2. Execução

```bash
Rscript src/ica_sync_analyses_status.R \
  -k $ICA_API_KEY \
  -i dragen \
  -v
```

### 3. O que acontece por dentro

| Etapa                 | Ação                                                                                                          |
| --------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Carrega cache**     | Lê `projects.parquet/tsv` e todos os Parquets/TSVs em `data/analyses/`                                        |
| **`sync_projects()`** | Busca a lista de projetos via API e regrava `data/projects.{parquet,tsv}`                                     |
| **`sync_analyses()`** | Para cada projeto:<br>• Consulta `list_project_analyses()`<br>• Atualiza/gera `analyses/<slug>.{parquet,tsv}` |
| **Gera resumo**       | Concatena todas as análises e grava:<br>• `data/analyses/dragen.tsv` ou `bssh.tsv`                            |
| **Log no console**    | Imprime progresso por CA + contagem de status                                                                 |

### 4. Saídas

| Tipo                | Arquivo/Pasta                                |
| ------------------- | -------------------------------------------- |
| **Projetos**        | `data/projects.parquet` & `projects.tsv`     |
| **Análises por CA** | `data/analyses/<slug>.parquet` & `.tsv`      |
| **Resumo do modo**  | `data/analyses/dragen.tsv` **ou** `bssh.tsv` |

### 5. Exemplo de Log (modo DRAGEN)

```
ℹ Sincronizando projetos DRAGEN: MG-BH, PA-BE, PE-RE, ...
ℹ [1/8] MG-BH: 1111 análises (+0 novas e 0 atualizadas)
⋯
✔ Resultados DRAGEN salvos em .../data/analyses/dragen.tsv

ℹ Resumo de status DRAGEN:
• SUCCEEDED: 11500
• FAILED:     220
• ABORTED:    130
```

---

**Dica** Use `--no-sync` quando quiser apenas consultar o cache local sem chamar a API. Para detalhes completos de uso, veja a ajuda (`-h`) ou contate o mantenedor.
