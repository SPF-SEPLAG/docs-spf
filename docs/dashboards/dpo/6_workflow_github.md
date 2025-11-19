# Documentação do Workflow 

Após a implementação dos scripts de download da planilha, geração dos Tokens e processamento dos dados, é necessário a configuração do ambiente no github. 

Após a [criação do repositório privado](../guides/github/criar_repo.md), e [geração do PAT e dos Secrets](../guides/github/criar_secrets_pat.md), resta a elaboração do script `.yml` que irá coordenar a execução do **Github Actions**.

Este workflow do GitHub Actions é responsável por executar automaticamente os scripts Python detalhados anteriormente em horários específicos do dia, além de permitir execuções manuais. O objetivo principal é:

- Baixar a planilha do SharePoint (`download.py`)
- Processar os dados e gerar arquivos CSV (`process_data.py`)
- Fazer commit automático no repositório dos arquivos gerados

---

## 1. Agendamento das Execuções

O workflow utiliza **cron jobs** para rodar o processo de hora em hora, sempre no minuto 30, respeitando o horário de Brasília (UTC−3).

### Horários de Execução
Cada **cron job** listado abaixo representa um horário convertido para UTC:

| Horário BRT | Cron (UTC) |
|-------------|------------|
| 09:30       | `30 12 * * *` |
| 10:30       | `30 13 * * *` |
| 11:30       | `30 14 * * *` |
| 12:30       | `30 15 * * *` |
| 13:30       | `30 16 * * *` |
| 14:30       | `30 17 * * *` |
| 15:30       | `30 18 * * *` |

Além disso, existe o gatilho:

```yaml
workflow_dispatch:
```

que permite executar o workflow manualmente pelo GitHub.

---

## 2. Permissões

```yaml
permissions:
  contents: write
```

Isso permite que o workflow faça commits automáticos no repositório.

---

## 3. Job Principal

```yaml
jobs:
  run-script:
    runs-on: ubuntu-latest
```

O processo roda em uma máquina virtual **Ubuntu** fornecida pelo GitHub.

---

### 3.1. Checkout do repositório
```yaml
      - name: Checkout repository
        uses: actions/checkout@v3
```
Baixa o código do repositório para a máquina virtual.

---

### 3.2. Configuração do Python
```yaml
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'
```
Instala e ativa o Python 3.12.

---

### 3.3. Instalação das dependências
```yaml
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
```
Atualiza o pip e instala dependências se existir um arquivo `requirements.txt`.

---

### 3.4. Execução do script de download
```yaml
      - name: Run Python script
        run: python download.py
```
Executa a lógica que:
- acessa o SharePoint via MSAL
- baixa a planilha
- salva no repositório

---

### 3.5. Processamento dos dados
```yaml
      - name: Generate CSV Number and Text File
        run: python process_data.py   # ⬅️ Your custom CSV logic
```
Esse script realiza:
- Transformações com Pandas
- Geração de CSVs com os dados 

---

### 3.6. Commit e push automático
```yaml
      - name: Commit and push downloaded file
        env:
          TOKEN: ${{ secrets.PERSONAL_TOKEN }}
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m "Add downloaded file" || echo "No changes to commit"
          git remote set-url origin https://x-access-token:${TOKEN}@github.com/${{ github.repository }}
          git push origin HEAD:${{ github.ref_name }}
```

Este passo:

- configura a identidade do Git  
- adiciona os novos arquivos  
- cria um commit (se houver mudanças)  
- envia ao repositório usando um token pessoal

***************PAREI AQUI****************
```bash
git commit -m "Add downloaded file" || echo "No changes to commit"
```

Caso não existam alterações, o commit é ignorado.

⚠ **Segurança:**  
O token usado (`${{ secrets.PERSONAL_TOKEN }}`) deve ter permissões de *repo*.

---

## 🧪 4. Visão Geral do Fluxo

1. Workflow dispara automaticamente ou manualmente  
2. Python é configurado  
3. Scripts são executados  
4. Arquivos gerados são commitados no repositório  
5. Workflow finaliza

---

## 📝 Sugestão de título para o arquivo de documentação

- `workflow_execucao_automatica.md`
- `automacao_download_sharepoint.md`
- `pipeline_processamento_sharepoint.md`
