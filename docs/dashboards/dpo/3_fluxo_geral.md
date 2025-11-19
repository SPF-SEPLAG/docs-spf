# Integração das ferramentas e fluxo geral

Para garantir o processamento e a atualização dos dados em tempo real, foi necessário definir um fluxo de **ETL (Extract, Transform, Load)** que englobasse todo o processo: download automático da planilha, tratamento e modelagem dos dados via `pandas`, e carregamento dos resultados no painel do Power BI — tudo de forma totalmente automatizada.  

Para isso, foram investigadas quais ferramentas melhor atenderiam a cada etapa e como integrá-las em um único fluxo contínuo. Em linhas gerais, são utilizadas as seguintes ferramentas, de forma integrada: **Sharepoint, python, Github e PBI**.

Abaixo está uma explicação resumida de cada tecnologia envolvida e do fluxo geral da rotina. 

---

## Git e GitHub

Para saber o que é o **Git e o GitHub**, [leia aqui](https://docs.github.com/en/get-started/start-your-journey/about-github-and-git).

O GitHub permite armazenar e versionar código na nuvem, além de integrar ferramentas para automação, como o **GitHub Actions**.  
O projeto desenvolvido utiliza o **GitHub** como repositório central e controle de versionamento via **Git**.

---

## GitHub Actions

Para execução automática dos scripts foi utilizado o **GitHub Actions**.

> Para saber o que é o **GitHub Actions**, [leia aqui](../guides/github/github_actions.md).

As **VMs** disponibilizadas pelo GitHub Actions permitem a automação completa dos scripts de **download e tratamento de dados** utilizados pela DPO.  
Após o processamento, os arquivos `.csv` gerados são automaticamente commitados de volta ao repositório, prontos para o carregamento no Power BI.

---

## Secrets e Personal Access Token (PAT)

Por motivos de segurança, o repositório GitHub foi configurado como **privado**, permitindo acesso apenas a usuários autorizados.

Para que a rotina automatizada funcione corretamente — especialmente nos momentos em que a VM precisa **enviar commits** ao repositório e **carregar dados** no Power BI — é necessário o uso de credenciais de acesso.  

Essas credenciais são configuradas por meio dos **Secrets** e **Personal Access Token (PAT)**, definidos dentro do próprio GitHub.  
O passo a passo para criação dessas credenciais está descrito [neste guia](../guides/github/criar_secrets_pat.md).

---

## msal e Office365-REST-Python-Client

### Download automático da planilha do SharePoint

Uma das etapas mais importantes é o **download automatizado da planilha** hospedada no SharePoint, utilizando Python.  

Para isso, foram usadas as bibliotecas: `msal` e `Office365-REST-Python-Client`.

Ambas são mantidas pela Microsoft e permitem a integração com os serviços do **Office 365** via API, utilizando credenciais corporativas.

Com elas, foi possível construir um script Python capaz de **baixar automaticamente** a planilha diretamente do SharePoint.

---

### Criação do Access Token e Refresh Token

Durante a elaboração do script de download, identificou-se um desafio: a necessidade de **login manual** com a conta organizacional.  
Isso impediria a execução automatizada do fluxo completo.

Para resolver esse problema, foi implementada uma **autenticação automática** com o SharePoint, utilizando a geração de **Access Tokens** e **Refresh Tokens** via `msal`.

> Para entender o que são **Access Token** e **Refresh Token**, [leia aqui](../guides/github/access_refresh_tokens.md).

Na primeira execução do script, o login é feito manualmente, e os tokens gerados são armazenados em um arquivo de cache (`msal_cache.bin`). 
Esse arquivo é então commitado no repositório privado do GitHub.

Nas execuções seguintes, o fluxo usa o **Refresh Token armazenado** para gerar automaticamente um novo **Access Token**, sem necessidade de login manual.  
Assim, o **GitHub Actions** executa todo o processo de autenticação e download sem intervenção humana.

> O Refresh Token também possui tempo de expiração.  
> Ainda não há uma definição exata sobre sua validade, portanto é necessário **monitorar as execuções** dos workflows do GitHub Actions e **regerar manualmente** os tokens quando necessário.
---

### Obtenção dos scripts de download, credenciais organizacionais API sharepoint e gerações dos Tokens

Deve-se ressaltar que a elaboração da rotina descrita aqui contou com suporte da equipe da **Assessoria da Inteligência de Dados da Subsecretaria Central de Planejamento e Orçamento da SEPLAG/MG**. Além do auxílio para projetar e implementar a rotina, foi cedido o script **sharepoint_utils.py** que contém as funções de gerar os Tokens e de realizar download e upload de arquivos do Sharepoint, além das credenciais organizacionais de acesso aos serviços do **Office 365** via `API`. 

## Fluxo geral

<div class="mermaid">
flowchart TD
A["Início do processo"] --> B["Execução manual inicial do script Python<br>de download da planilha da DPO"]
B --> C["Login manual na conta organizacional (MSAL)"]
C --> D["Geração do Access Token e Refresh Token"]
D --> E["Armazenamento em cache local (msal_cache.bin)"]
E --> F["Commit e push manual do cache para o repositório privado do GitHub"]
F --> G["Execução automática da rotina via GitHub Actions"]
G --> H["Download automatizado da planilha do SharePoint"]
H --> I["Tratamento e modelagem dos dados com pandas"]
I --> J["Exportação dos CSVs tratados"]
J --> K["Commit automático dos CSVs no repositório via workflow.yml"]
K --> L["Carregamento dos dados no Power BI"]
L --> M["Fim do processo"]
</div>

---

## Comentários sobre o fluxo

- A **primeira execução** requer login, commit e push manualmente para gerar os tokens iniciais e o arquivo cache.
- As **execuções seguintes** são 100% automáticas, até o vencimento do Refresh Token.  
- O **GitHub Actions** atua como o motor de automação, orquestrando todo o fluxo ETL.

---


