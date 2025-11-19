# Download da planilha do Sharepoint e geração dos Tokens
Para que a rotina funcione automaticamente via GitHub Actions, é necessário **gerar o Access Token e o Refresh Token uma única vez**, manualmente. Esses tokens ficam salvos em um arquivo de cache e serão reutilizados nas execuções seguintes.

As próximas execuções serão automáticas e silenciosas, sem necessidade de login manual.

## 1. Visão Geral do Processo
- Na **primeira execução**, o usuário realiza login interativo no navegador.
- O script salva os tokens no arquivo `msal_cache.bin`.
- Nas execuções seguintes, o script utiliza o **Refresh Token** para obter o Access Token silenciosamente.
- O token obtido permite requisitar a API do SharePoint e realizar o download da planilha.

## 2. Arquivo principal: `download.py`

```python
from sharepoint_utils import download_sharepoint_file, get_sharepoint_token

base_url = "https://cecad365.sharepoint.com/sites/DPO"
folder_path = "/sites/DPO/Documentos Compartilhados/1 Orçamento/1 Acompanhamento da Execução Orçamentária/Execução 2025"

download_sharepoint_file(base_url, folder_path, "Execução Orçamentária 2025.xlsx", "Execução Orçamentária 2025.xlsx")
```

## 3. Função de Download 
```python
def download_sharepoint_file(base_url, folder_path, file_name, local_filename):
    """
    Downloads a file from SharePoint using dynamic components for the URL.
    """
```

**Parâmetros:**
- `base_url` — URL base do site SharePoint.
- `folder_path` — caminho interno da pasta onde o arquivo está.
- `file_name` — nome do arquivo no SharePoint.
- `local_filename` — nome com que será salvo localmente.

- Antes do download, a função chama `get_sharepoint_token()`, responsável por obter o Access Token.

## 4. Configurações de Autenticação

```python
config = {
  "authority": "https://login.microsoftonline.com/e5d3ae7c-9b38-48de-a087-f6734a287574",
  "client_id": "d44a05d5-c6a5-4bbb-82d2-443123722380",
  "scope": ["https://cecad365.sharepoint.com/.default"], #["Group.ReadWrite.All"],
  "username": "username@ca.mg.gov.br",
  "endpoint": "https://login.microsoftonline.com/common/oauth2/v2.0/authorize"
}
```
- Somente o campo `username` deve ser alterado
- Trocar: `username@ca.mg.gov.br` → por um login CA válido do responsável pela rotina.

# 5. Geração dos Tokens: get_sharepoint_token()
## 5.1. Carregamento do Cache
```python
def get_sharepoint_token():
    # ✅ Load or create token cache
    cache = msal.SerializableTokenCache()
    if os.path.exists(CACHE_FILE):
        cache.deserialize(open(CACHE_FILE, "r").read())
```
- Se existir um cache, ele é carregado, junto com o Access Token e o Refresh Token.
- Na primeira execução, o cache estará vazio.

## 5.2. Inicialização do aplicativo MSAL
```python
    app = msal.PublicClientApplication(
        config["client_id"], authority=config["authority"], token_cache=cache
        )
```
O aplicativo é carregado com:
- credenciais da organização,
- tokens existentes no cache (se houver).

## 5.3. Verificação de contas já autenticadas
```python
    accounts = app.get_accounts(username=config.get("username"))
```
- Verifica se já existe token válido para o usuário informado.
- **Na primeira execução: estará vazio.**

## 5.4. Aquisição silenciosa do Access Token
```python
    if accounts:
        result = app.acquire_token_silent(config["scope"], account=accounts[0])
```
Se o cache contém um Refresh Token válido:
- O Access Token é obtido **silenciosamente.**
- É esse processo que permite a automação via GitHub Actions.

## 5.5. Aquisição interativa (primeira execução)
```python
    if not result:
        # So no suitable token exists in cache. Let's get a new one from Azure AD.
        result = app.acquire_token_interactive(scopes=config["scope"])
```
Caso não exista token válido:
- Abre o navegador.
- Usuário faz login com a conta organizacional.
- Novos tokens são gerados.

## 5.6. Salvamento do Cache
```python
    # ✅ Save updated cache
    if cache.has_state_changed:
        with open(CACHE_FILE, "w") as f:
            f.write(cache.serialize())
        print("💾 Token cache saved.")
```
- Tokens novos ou atualizados são gravados no arquivo `msal_cache.bin`.
- Se o arquivo não existir, ele será criado.

## 6. Conclusão
A função `get_sharepoint_token()` retorna o Access Token obtido, seja:
- silenciosamente via Refresh Token, ou
- manualmente na primeira execução.
Esse token é então usado pela função `download_sharepoint_file()` para baixar a planilha do SharePoint.

## 7. Comentários Finais
- A versão original da função de autenticação fornecida pela equipe da SPLOR estava com falhas e foi redesenhada pela equipe de automação da SPF.
- O foco deste documento está na parte mais sensível e complexa do fluxo: a geração e gerenciamento dos Tokens.
- Para detalhes da implementação do download em si, consultar o script original.