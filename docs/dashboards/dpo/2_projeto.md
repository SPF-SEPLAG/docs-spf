# Projeto e Cronograma Painel DPO

Este projeto se refere à elaboração do **Painel (Dashboard) de Business Intelligence (BI)** da **Diretoria de Planejamento e Orçamento (DPO)** da **Superintendência de Planejamento e Finanças (SPF)** da **Subsecretaria de Gestão e Finanças (SUBGEF)** da **Secretaria de Planejamento e Gestão do Estado de Minas Gerais (SEPLAG-MG)**.

O principal fator identificado durante a investigação realizada foi a **elevada complexidade técnica** associada à manutenção e ao uso da planilha atualmente em vigor. Utilizada há vários anos, essa planilha passou por sucessivos ajustes e adaptações, o que comprometeu sua estrutura tanto como base de dados quanto como ferramenta de consulta.  

Além disso, verificou-se que a forma de organização dos dados dificulta a compreensão por parte das áreas técnicas externas, que enfrentam obstáculos para acessar as informações de maneira clara e visualmente organizada. Isso evidencia uma lacuna entre a forma como os dados estão dispostos e sua efetiva utilidade no apoio às demandas dessas áreas.  

Os dados presentes na planilha são consumidos e utilizados por agentes de diversos níveis organizacionais — **técnicos da DPO**, **áreas demandantes** e **tomadores de decisão**. Portanto, proporcionar uma comunicação clara e acessível é fundamental.  

Dessa forma, foi identificada a necessidade de **melhoria na representação e visualização dos dados** utilizados no contexto de trabalho da Diretoria, por meio do desenvolvimento de um dashboard de Business Intelligence.

---

## Objetivos

Este projeto possui os seguintes objetivos:

- Subsidiar o desenvolvimento de um dashboard de BI para apoiar processos de trabalho e tomada de decisão.  
- Documentar o processo de trabalho realizado pelos técnicos desta Assessoria, servindo como **metodologia base** para futuros painéis ou automações.  
- Apoiar a elaboração de um **cronograma com prazos, metas e entregas previstas**.

O resultado esperado é a entrega de um painel validado pelos usuários e em conformidade com as demandas operacionais da DPO, das áreas externas e dos tomadores de decisão.

---

## Metodologia

Para a definição do escopo do projeto, foram realizadas reuniões com a DPO e estudo da planilha Excel utilizada como fonte de dados.  

O painel será estruturado para contemplar **duas abas principais**:

- **Informações Técnicas:** voltadas ao contexto operacional da DPO, para acompanhamento diário.  
- **Informações Gerenciais:** voltadas à avaliação estratégica e tomada de decisão dos gestores.  

A metodologia de desenvolvimento adotada foi **incremental**, com entregas evolutivas em **fases e ciclos**, atendendo às seguintes particularidades:

- Entregas parciais e rápidas  
- Engajamento contínuo dos envolvidos  
- Redução de retrabalho  

**Ferramentas utilizadas:**
- Power BI, Power Query, Excel Online, SharePoint  
- Python (`msal`, `Office365-REST-Python-Client`, `pandas`)  
- GitHub, GitHub Actions  
- Git Bash (recomendado)  
- VS Code (recomendado)

---

## Fases e Ciclos

**Fase 0 — Alinhamento e Planejamento Inicial**

- Conversas iniciais com a DPO (mapeamento do processo, entendimento do problema)  
- Estudo inicial da fonte de dados  
- Definição das fases do projeto e cronograma  

> Ao final, foi decidido dividir o desenvolvimento do painel em três fases:
> 1. Aba técnica (informações para áreas e técnicos da DPO)  
> 2. Aba gerencial (informações para gestores)  
> 3. Testes e homologação final  

**Fase 1 e Fase 2 — Desenvolvimento das Abas**

Etapas por ciclo:

1. **Conversas com a DPO** — definição de escopo, layout e prioridades.  
2. **Modelagem de Dados e ETL** — extração, transformação e padronização.  
3. **Desenvolvimento do Painel** — construção de indicadores e visualizações.  
4. **Validação** — testes de consistência e coleta de feedbacks.  

**Fase 3 — Validação e Homologação Final**

Essa fase garante estabilidade técnica, confiabilidade dos dados e boa usabilidade.  
Recomenda-se realizar um **teste piloto com usuários-chave** de diferentes perfis (analistas, gestores, operacionais).

Etapas:
- Coleta de feedback sobre clareza, utilidade e erros.  
- Ajustes e homologação final antes da liberação.  

---

## Cronograma

| Fase | Atividade | Responsáveis | Prazo Estimado |
|------|------------|---------------|----------------|
| **Fase 0** | Reuniões iniciais com a DPO | Assessoria SPF + DPO | 2 dias |
| **Fase 0** | Estudo da fonte de dados | Assessoria SPF | 4 dias |
| **Fase 0** | Definição de fases e cronograma | Assessoria SPF | 3 dias |
| **Fase 0** | Validação do planejamento | Assessoria SPF + DPO | 1 dia |
| **Fase 1** | Conversas com DPO (escopo, layout) | Assessoria SPF + DPO | 1 dia (por ciclo) |
| **Fase 1** | Modelagem de dados e ETL | Assessoria SPF | 3 dias (por ciclo) |
| **Fase 1** | Desenvolvimento do painel | Assessoria SPF | 2 dias (por ciclo) |
| **Fase 1** | Validação com técnicos | DPO + Assessoria SPF | 1 dia (por ciclo) |
| **Fase 2** | Conversas com gestores | Assessoria SPF + DPO | 1 dia (por ciclo) |
| **Fase 2** | Modelagem de dados gerencial | Assessoria SPF | 3 dias (por ciclo) |
| **Fase 2** | Desenvolvimento dos painéis gerenciais | Assessoria SPF | 2 dias (por ciclo) |
| **Fase 2** | Validação com gestores | DPO + Assessoria SPF | 1 dia (por ciclo) |
| **Fase 3** | Preparação para piloto | Assessoria SPF | 1 dia |
| **Fase 3** | Teste piloto com a área | Assessoria SPF + DPO | 5 dias |
| **Fase 3** | Coleta e análise de feedbacks | Assessoria SPF | 1 dia |
| **Fase 3** | Ajustes finais | Assessoria SPF | 3 dias |
| **Fase 3** | Homologação e entrega | DPO + Assessoria SPF | 1 dia |

**Tempo total estimado**

- Fase inicial: **10 dias úteis**  
- Desenvolvimento (3 ciclos por aba): **42 dias úteis**  
- Fase final de testes: **10 dias úteis**  
**Total: 62 dias úteis**

---

## Considerações Finais

O cronograma pode sofrer ajustes conforme surgirem situações imprevistas ou limitações técnicas.  
Por exemplo, nas fases iniciais, foi necessário utilizar **Python e GitHub Actions** para o tratamento automatizado dos dados devido ao alto volume e limitação do Power BI/hardware.

Essa **rotina Python automatizada** será documentada em uma seção própria, dada sua complexidade e importância para o funcionamento do painel.

---