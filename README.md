MVP - Segurança Cibernética: Panorama de Vulnerabilidades (NVD/CVE)

Trabalho final da disciplina Sistemas de Suporte à Decisão, Universidade de Brasília (UnB). Professor: André Luiz Marques Serrano. Aluna: Maria Eduarda Oliani

Objetivo

Problema de negócio: apoiar a priorização de esforços de gestão de vulnerabilidades (patch management) a partir de dados públicos de segurança cibernética, identificando onde a severidade, o tempo de resposta e a concentração de risco são maiores.

Perguntas de negócio:

Quais categorias de vulnerabilidade (CWE) apresentam maior severidade média (CVSS) e como essa severidade evolui ao longo do tempo?
Qual o tempo médio entre a publicação de uma CVE e sua última atualização, segmentado por nível de severidade?
Quais fornecedores e produtos concentram o maior número de vulnerabilidades críticas (CVSS ≥ 9.0)?
Existe sazonalidade mensal na publicação de vulnerabilidades críticas?
Qual a proporção de vulnerabilidades ainda sem pontuação CVSS atribuída, e como essa proporção varia por status de análise?
Fonte de dados
Base: National Vulnerability Database (NVD), mantida pelo NIST.
Acesso: API pública REST 2.0 — https://services.nvd.nist.gov/rest/json/cves/2.0.
Licença: dados de domínio público, com atribuição ao NIST (FAQ oficial). Sem restrição de redistribuição.
Período coletado: 2023-01-01 a 2025-01-01.
Formato original: JSON.
Data da coleta: ver catalogo/metadados_coleta.json.
Metodologia

O pipeline foi construído em Python, executado no Google Colab, e segue as etapas abaixo.

Busca e Coleta — consumo da API pública da NVD, com paginação, respeito ao rate limit, e nova tentativa automática (retry com espera progressiva) em caso de erros transitórios do servidor (429/500/502/503/504) ou timeout de conexão.
Modelagem — Esquema Estrela, com granularidade de uma linha por CVE na tabela fato:
fato_vulnerabilidade: severidade (CVSS), datas do ciclo de vida, status, categoria CWE primária.
dim_cwe: categorias de fraqueza (Common Weakness Enumeration).
dim_produto: fornecedores e produtos afetados.
ponte_vulnerabilidade_produto: relação muitos-para-muitos entre CVEs e produtos.
dim_tempo: granularidade diária, para agregações temporais.
Documentação completa de cada atributo em catalogo/catalogo_dados.csv.
Carga — persistência em banco PostgreSQL gerenciado (Supabase), com recarga completa das tabelas a cada execução.
Análise — avaliação de qualidade dos dados (completude, unicidade, consistência, conformidade, acurácia) e resposta às cinco perguntas de negócio por meio de consultas SQL executadas diretamente sobre o banco na nuvem.

O notebook completo, com todas as etapas documentadas célula a célula, está em notebooks/MVP_Seguranca_Cibernetica.ipynb.

Resultados

Vulnerabilidades de execução de código (injeção, desserialização, upload de arquivo) têm as maiores severidades médias, com destaque para CWE-434 em alta em 2024. Linux concentra o maior número de CVEs críticas. Tempo até a última atualização não varia por severidade — limitação do dado, não achado de negócio. Sazonalidade mensal é moderada, sem padrão robusto. Score ausente concentra-se apenas no status Rejected.

Qualidade dos dados

(preencher com o resumo das checagens de completude, unicidade, consistência, conformidade e acurácia realizadas na seção 5.1 do notebook)

Autoavaliação
Quais das perguntas de negócio formuladas no objetivo foram respondidas e quais não? Por quê? Três das cinco (1, 3 e 5) foram respondidas de forma acionável. As Perguntas 2 e 4 tiveram resposta parcial, limitada pela granularidade temporal da NVD e pelo curto período coletado.
Que limitações dos dados condicionaram os resultados obtidos? O campo de última modificação da NVD mistura reanálise interna e patch do fornecedor. A janela de dois anos limita conclusões sazonais. Produtos ficaram duplicados entre hardware/firmware em alguns casos.
Quais decisões técnicas seriam tomadas de outra forma caso o trabalho fosse reiniciado? (personalizar — ex: coletar também as referências externas de cada CVE; unificar produtos hardware/firmware na modelagem; escolher desde o início uma plataforma sem exigência de conta de faturamento)
Que extensões seriam necessárias para transformar este MVP em uma solução de uso contínuo? Carga incremental em vez de completa, execução agendada (ex: GitHub Actions), uso do campo de referências para estimar tempo real de patch, e um painel conectado diretamente ao banco.
Estrutura do repositório
├── README.md
├── LICENSE
├── notebooks/
│   └── MVP_Seguranca_Cibernetica.ipynb
├── catalogo/
│   ├── catalogo_dados.csv
│   └── metadados_coleta.json
└── evidencias/
    ├── supabase_table_editor.png
    └── resultados_consultas.png
Como reproduzir
Criar um projeto gratuito no Supabase e obter a connection string (Connect > Transaction Pooler, formato URI).
Abrir notebooks/MVP_Seguranca_Cibernetica.ipynb no Google Colab.
Substituir a variável DATABASE_URL pela connection string obtida no passo 1.
Executar as células em ordem, da Busca e Coleta até a Análise.
