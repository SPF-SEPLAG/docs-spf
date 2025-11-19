# Pré-processamento dos dados

Como detalhado nas seções anteriores, devido à complexidade da planilha utilizada no dia-a-dia de trabalho da DPO e por limitações do PBI ao processar o grande volume de dados foi necessário realizar o tratamento inicial dos dados utilizando outra solução que não fosse o Power Query ou o DAX. 

Para isso, foi feito um pré-processamento por meio do python, utilizando a lib do `pandas`, amplamente utilizada para leitura, limpeza, transformação e análise de dados.  

## Acesso
- Para acessar os scripts, acesse o [repo](https://github.com/SPF-SEPLAG/painel-dpo) pelo github.
> Caso não consiga visualizar o repo, peça para o responsável adicioná-lo como colaborador.

## Pré-processamento da planilha da DPO (process_data.csv)

### Etapa 1 - Filtrar abas relevantes 
- A função `get_tables()` identifica quais abas da planilha contêm dados válidos.
- Ela ignora abas desnecessárias como “Menu” e mantém apenas aquelas com 4 caracteres.

```python
def get_tables(excel_file):
  # Get all sheet names
  xls = pd.ExcelFile(excel_file)
  sheet_names = xls.sheet_names
  filtered_sheets = []
  # Filter sheet names:
  # - Keep only 4-character names
  # - Remove 'Menu'
  # - Remove names starting with '44'
  for sheet in sheet_names:
      if len(sheet) == 4 and sheet not in ['Menu', '44xx', '4488', '4489']:
          filtered_sheets.append(sheet)

  return filtered_sheets
```
- **Saída**: uma lista de nomes de abas filtradas.

### Etapa 2 - Processar abas textuais
- A função `process_tables_text()` trata colunas de texto, como “Objeto” e “Programação”.
**Principais ações**
1. Encontra a linha onde começa a tabela (procura “Programação”).
2. Remove linhas e colunas extras.
3. "Limpa" a coluna "Objeto" para evitar erros de atualização por conta do preenchimento dos dados da planilha pelos técnicos (padroniza os textos -> tudo em minúsculo, tira acentos e pontuação, etc).
4. Remove linhas baseado em valores específicos de GMIFP.

```python
def process_tables_text(excel_file, filtered_sheets):
  treated_dfs = []
  removed_ids = []
  df = pd.DataFrame()

  for sheet in filtered_sheets:
      df = pd.read_excel(excel_file, sheet_name=sheet)
      # Find the row index where first column contains 'Programação'
      target_row = df[df.iloc[:, 0].str.contains('Programação', case=False, na=False)].index[0]
      # Get all data starting from the target row
      df = df.iloc[target_row:]
      # Drop rows with empty values in the first column
      df = df.dropna(subset=[df.columns[0]])
      # Drop last row
      df = df[:-1]
      # Keep only the first 9 columns
      df = df.iloc[:, :9]
      # Promote first row to headers
      df.columns = df.iloc[0]
      df = df.iloc[1:]  # Remove the first row since it's now the header
      # Add sheet name as 'Ação'
      df['Ação'] = sheet

      # Cleans column Objeto
      df["Objeto"] = (
          df["Objeto"]
          .fillna("")
          .str.strip()
          .str.lower()
          .apply(lambda x: re.sub(r'[^\w\s]', " ", x, flags=re.UNICODE))
          .str.split()
          .str.join(" ")
          .str.title()
      )

      #GMIFPs to remove
      gmis_to_remove = ["1.90.0.10.1", "1.91.0.10.1", "3.90.0.10.7"]

      # Get which rows will be removed
      rows_to_remove = df[df['GMIFP'].isin(gmis_to_remove)]

      # Extract the IDs of those rows for later processing by process_tables_num
      removed_ids.extend(rows_to_remove['Programação'].tolist())

      # Effectively remove rows from df by GMIFP
      df = df[~df['GMIFP'].isin(gmis_to_remove)]

      print("Tratando aba:", sheet)
      treated_dfs.append(df)

  if treated_dfs:
    final_df = pd.concat(treated_dfs, ignore_index=True)
    output_file = 'processed_data_text.csv'
    final_df.to_csv(output_file, index=False, encoding='utf-8-sig')
    print(f"\nData exported to {output_file}")

    return removed_ids
```
- **Saída**: exporta os dados tratados para `processed_data_text.csv` e retorna uma lista de IDs das linhas removidas.

### Etapa 3 — Processar abas numéricas
- A função `process_tables_num()` trata os valores financeiros.
- Ela usa os IDs removidos na etapa anterior para excluir registros indesejados.
**Principais ações**
1. Localiza a linha “Programação” e ajusta o início da tabela
2. Remove colunas desnecessárias (como “TOTAIS”)
3. Transpõe os dados (linhas ↔ colunas) e em seguida faz o unpivot e pivot para reorganizar os valores.
> O passo 3 é imprescindível para associar corretamente o número de programação aos meses e às respectivas colunas de valores, quais sejam "Descentralizado", "Empenhado", "Liquidado", "Programado" e "Realizado".
4. Converte colunas para números.
5. Remove as linhas com os respectivos IDs do passo anterior (associados aos GMIFPs que devem ser removidos).

```python
def process_tables_num(excel_file, filtered_sheets, removed_ids):
  treated_dfs = []

  for worksheet in filtered_sheets:
      df = pd.read_excel(excel_file, sheet_name=worksheet)

      # 1. Find the row index where first column contains 'programação'
      target_row = df[df.iloc[:, 0].str.contains('Programação', case=False, na=False)].index[0]

      # 2. Filter the DataFrame to start 2 rows before the target row
      df = df.iloc[target_row-2:]

      # 3. Remove rows where first column is null, but keep the first row
      first_row = df.iloc[0:1]  # Keep first row
      rest_of_df = df.iloc[1:].dropna(subset=[df.columns[0]])  # Remove nulls from rest
      df = pd.concat([first_row, rest_of_df])  # Combine them back

      # 4. Remove the last row
      df = df.iloc[:-1]

      # 5. Find the column with 'TOTAIS' in the second row and remove it and all columns after it
      total_col_idx = df.iloc[1].str.contains('TOTAIS', case=False, na=False).idxmax()
      total_col_num = df.columns.get_loc(total_col_idx)
      df = df.iloc[:, :total_col_num]  # Keep all columns up to (but not including) the TOTAIS column

      # 6. Remove columns 2 through 9
      df = pd.concat([df.iloc[:, 0:1], df.iloc[:, 9:]], axis=1)

      # 7. Transpose the DataFrame
      df = df.transpose()

      # 8. Remove the second column
      df = df.drop(df.columns[1], axis=1)

      # 9. Promote first row to headers
      df.columns = df.iloc[0]  # Set column names to first row
      df = df.iloc[1:]  # Remove the first row since it's now the header

      # 10. Rename the first two columns
      df = df.rename(columns={df.columns[0]: 'Mês', df.columns[1]: 'Status'})

      # 11. Unpivot the DataFrame
      id_vars = ['Mês', 'Status']  # Columns to keep as identifiers
      df = pd.melt(df, 
                  id_vars=id_vars,
                  var_name='Programação',  # Name for the column containing the former column headers
                  value_name='Valor')    # Name for the column containing the values

      # 12. Add Ação column with worksheet name
      df['Ação'] = worksheet

      # 13. Pivot the table back using Status and Valor
      df = df.pivot(index=['Mês', 'Programação', 'Ação'], 
                  columns='Status', 
                  values='Valor').reset_index()

      # 14. Force columns to be number types (filter unexpected characters in number columns)
      columns_to_clean = ['Descentralizado', 'Empenhado', 'Liquidado', "Programado", "Realizado"]

      for col in columns_to_clean:
          # Remove tudo que não é número e converte para float, ou NaN se falhar
          df[col] = pd.to_numeric(df[col], errors='coerce').fillna(0)

      # 15. Remove rows where gmifp == ["1.90.0.10.1", "1.91.0.10.1", "3.90.0.10.7"] by n. Programação
      df = df[~df['Programação'].isin(removed_ids)]

      # Add to list of DataFrames
      treated_dfs.append(df)
      print(f"Processed worksheet: {worksheet}")

  # Combine all DataFrames
  if treated_dfs:
      final_df = pd.concat(treated_dfs, ignore_index=True)

      # Export to CSV
      output_file = 'processed_data_num.csv'
      final_df.to_csv(output_file, index=False, encoding='utf-8-sig')
      print(f"\nData exported to {output_file}")
```
- **Saída**: exporta o resultado para `processed_data_num.csv`.

### Execução do Script

- Por fim, basta passar o nome da planilha e chamar as funções na sequência:

```python
excel_file = 'Execução Orçamentária 2025.xlsx'

filtered_sheets = get_tables(excel_file)
removed_ids = process_tables_text(excel_file, filtered_sheets)
process_tables_num(excel_file, filtered_sheets, removed_ids)
```

### Resultado esperado
- `processed_data_text.csv`
- `processed_data_num.csv`

Esses arquivos serão integrados via **GitHub Actions** e então usados pelo **Power BI**.