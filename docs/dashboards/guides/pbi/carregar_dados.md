## Carregamento de dados de Planilha do Sharepoint no Power BI 
- Obter dados no Power BI 
![Obter dados](../../../images/carregar_dados/obter-dados.png)

- Extrair links da planilha no Sharepoint
![Obter dados](../../../images/carregar_dados/extrair-link1.png)
![Obter dados](../../../images/carregar_dados/extrair-link2.png)

- Colar link no Power BI
![Obter dados](../../../images//carregar_dados/colar-link.png)

## Carregamento de dados do Github no Power BI
- Extrair links das bases
![Obter dados](../../../images/carregar_dados/extrair-link-github1.png)
![Obter dados](../../../images/carregar_dados/extrair-link-github2.png)
![Obter dados](../../../images/carregar_dados/extrair-link-github3.png)

- Criar parâmetro no PBI e copiar e colar a hash do Personal Access Token (PAT) do Github, criado em momento prévio
![Obter dados](../../../images/carregar_dados/criar-token-pbi1.png)

- Criar consulta nula que irá armazenar os dados

![Obter dados](../../../images/carregar_dados/consulta-nula.png)

- Carregar dados via **Power Query M**, copiando e colando o código abaixo no PBI, alterando a URL

![Obter dados](../../../images/carregar_dados/power-query.png)

> O código citado serve para consumir os dados via API do próprio github, sendo passado no cabeçalho da requisição o tipo Bearer e o PAT.

```powerquery
url = "https://api.github.com/repos/SPF-SEPLAG/painel-dpo/contents/processed_data_num.csv",
authHeader = "Bearer " & GitHubToken,
response = Json.Document(Web.Contents(url, [
    Headers = [Authorization = authHeader]
])),
base64 = response[content],
binary = Binary.FromText(base64, BinaryEncoding.Base64),
csv = Csv.Document(binary, [Delimiter = ",", Encoding = 65001, QuoteStyle = QuoteStyle.Csv]),
table = Table.PromoteHeaders(csv),
```

