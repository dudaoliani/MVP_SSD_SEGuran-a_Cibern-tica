# MVP: Segurança Cibernética: Panorama de Vulnerabilidades (NVD/CVE)

Trabalho final da disciplina **Sistemas de Suporte à Decisão**, Universidade de Brasília (UnB).
Professor: André Luiz Marques Serrano.
Aluna: Maria Eduarda Oliani.

## Objetivo

**Problema de negócio:** apoiar a priorização de esforços de gestão de vulnerabilidades (patch management) a partir de dados públicos de segurança cibernética, identificando onde a severidade, o tempo de resposta e a concentração de risco são maiores.

**Perguntas de negócio:**

1. Quais categorias de vulnerabilidade (CWE) apresentam maior severidade média (CVSS) e como essa severidade evolui ao longo do tempo?
2. Qual o tempo médio entre a publicação de uma CVE e sua última atualização, segmentado por nível de severidade?
3. Quais fornecedores e produtos concentram o maior número de vulnerabilidades críticas (CVSS ≥ 9.0)?
4. Existe sazonalidade mensal na publicação de vulnerabilidades críticas?
5. Qual a proporção de vulnerabilidades ainda sem pontuação CVSS atribuída, e como essa proporção varia por status de análise?

## Fonte de dados

- **Base:** National Vulnerability Database (NVD), mantida pelo NIST.
- **Acesso:** API pública REST 2.0 — `https://services.nvd.nist.gov/rest/json/cves/2.0`.
- **Licença:** dados de domínio público, com atribuição ao NIST ([FAQ oficial](https://nvd.nist.gov/general/faq)). Sem restrição de redistribuição.
- **Período coletado:** 2023-01-01 a 2025-01-01.
- **Formato original:** JSON.
- **Data da coleta:** ver `catalogo/metadados_coleta.json`.

## Metodologia

O pipeline foi construído em Python, executado no Google Colab, e segue as etapas abaixo.

1. **Busca e Coleta** — consumo da API pública da NVD, com paginação, respeito ao rate limit, e nova tentativa automática (retry com espera progressiva) em caso de erros transitórios do servidor (429/500/502/503/504) ou timeout de conexão.
2. **Modelagem** — Esquema Estrela, com granularidade de uma linha por CVE na tabela fato:
   - `fato_vulnerabilidade`: severidade (CVSS), datas do ciclo de vida, status, categoria CWE primária.
   - `dim_cwe`: categorias de fraqueza (Common Weakness Enumeration).
   - `dim_produto`: fornecedores e produtos afetados.
   - `ponte_vulnerabilidade_produto`: relação muitos-para-muitos entre CVEs e produtos.
   - `dim_tempo`: granularidade diária, para agregações temporais.
   - Documentação completa de cada atributo em `catalogo/catalogo_dados.csv`.
3. **Carga** — persistência em banco PostgreSQL gerenciado (Supabase), com recarga completa das tabelas a cada execução.
4. **Análise** — avaliação de qualidade dos dados (completude, unicidade, consistência, conformidade, acurácia) e resposta às cinco perguntas de negócio por meio de consultas SQL executadas diretamente sobre o banco na nuvem.

O notebook completo, com todas as etapas documentadas célula a célula, está em `notebooks/MVP_Seguranca_Cibernetica.ipynb`.

## Resultados

Vulnerabilidades de execução de código (injeção, desserialização, upload de arquivo) têm as maiores severidades médias, com destaque para CWE-434 em alta em 2024. Linux concentra o maior número de CVEs críticas. Tempo até a última atualização não varia por severidade — limitação do dado, não achado de negócio. Sazonalidade mensal é moderada, sem padrão robusto. Score ausente concentra-se apenas no status Rejected.

## Qualidade dos dados

*(preencher com o resumo das checagens de completude, unicidade, consistência, conformidade e acurácia realizadas na seção 5.1 do notebook)*

## Autoavaliação

1. **Quais das perguntas de negócio formuladas no objetivo foram respondidas e quais não foram? Por quê?**
   Três das cinco perguntas foram respondidas de forma acionável: as categorias CWE mais severas e sua evolução (P1), a concentração de vulnerabilidades críticas por fornecedor (P3) e a proporção de CVEs sem pontuação por status (P5). As Perguntas 2 e 4 tiveram resposta parcial: o tempo até a última atualização foi calculado, mas não reflete o tempo de resposta do fornecedor, pois a métrica da NVD mistura isso com reanálises internas do catálogo; e a sazonalidade mensal não mostrou padrão robusto, dado o curto período de coleta.

2. **Que limitações dos dados condicionaram os resultados obtidos?**
   O campo de última modificação da NVD não diferencia correção do fornecedor de reanálise interna, comprometendo a Pergunta 2. A janela de dois anos limita conclusões sazonais (Pergunta 4). E a modelagem por CPE gerou duplicidade em alguns casos, com o mesmo produto aparecendo separadamente como hardware e firmware.

3. **Quais decisões técnicas seriam tomadas de outra forma caso o trabalho fosse reiniciado?**
   Coletaria também o campo de referências externas de cada CVE, para aproximar melhor o momento de disponibilização de um patch. Na modelagem, unificaria pares de hardware e firmware como um único produto. E optaria desde o início por uma plataforma de nuvem sem exigência de conta de faturamento — a tentativa inicial com o BigQuery consumiu tempo desproporcional ao benefício, o que levou à migração para um Postgres gerenciado.

4. **Que extensões seriam necessárias para transformar este MVP em uma solução de uso contínuo?**
   Carga incremental em vez de recarga completa; execução agendada automaticamente (ex: GitHub Actions), sem depender de rodar o notebook manualmente; uso do campo de referências externas para estimar o tempo real de patch por fornecedor; e um painel conectado diretamente ao banco, para consulta contínua sem reexecutar o notebook.

## Estrutura do repositório
├── README.md
├── LICENSE
├── notebooks/
│ └── MVP_Seguranca_Cibernetica.ipynb
├── catalogo/
│ ├── catalogo_dados.csv
│ └── metadados_coleta.json
└── evidencias/
├── supabase_table_editor.png
└── resultados_consultas.png


## Como reproduzir

1. Criar um projeto gratuito no [Supabase](https://supabase.com) e obter a connection string (Connect > Transaction Pooler, formato URI).
2. Abrir `notebooks/MVP_Seguranca_Cibernetica.ipynb` no Google Colab.
3. Substituir a variável `DATABASE_URL` pela connection string obtida no passo 1.
4. Executar as células em ordem, da Busca e Coleta até a Análise.
