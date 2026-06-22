🌊 QualidadeAmbiental AWS Data Lake

Contract-driven cloud ingestion pipeline for environmental water quality monitoring data.

From SQL Server to Parquet — validated, cataloged, and ready for Power BI.


Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem
Mostrar Imagem

</div>

📖 Sobre o Projeto

QualidadeAmbiental AWS Data Lake é um pipeline de engenharia de dados cloud-native derivado do contrato externo de dados v2.1.0 do projeto QualidadeAmbiental_SQLServer. Ele automatiza a ingestão, validação, catalogação e transformação de dados de monitoramento de qualidade da água e esgoto em um Data Lake na AWS.

Valor técnico: O projeto implementa um pipeline de dados com contrato rígido de 30 colunas, validação local pré-ingestão, arquitetura medallion (raw → curated) e exposição analítica via Athena + Power BI — tudo sem armazenar credenciais no repositório.

Valor de negócio: Permite que engenheiros ambientais e analistas de dados acessem dados históricos de conformidade hídrica de forma confiável, rastreável e escalável, diretamente em ferramentas de BI corporativo.


Escopo v0.1.0: Automação de validação e ingestão de novos lotes após o provisionamento da infraestrutura AWS. Provisionamento de infraestrutura e substituição destrutiva de partições estão fora do escopo desta versão.




🏗️ Arquitetura

text┌─────────────────────────────────────┐
│   SQL Server View / CSV Contratado  │
│          (Fonte de dados)           │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│    Python — Validação de Contrato   │
│  30 colunas · ISO dates · unicidade │
│  flags de conformidade · nulos      │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│         S3 Raw Zone (CSV)           │
│   Partição: /ingestion_date=YYYY-   │
│   MM-DD/                            │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│   AWS Glue Crawler + Data Catalog   │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│    Amazon Athena INSERT INTO        │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│      S3 Curated Zone (Parquet)      │
│         Compressão: Snappy          │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│    Amazon Athena → Power BI Import  │
│         (Camada analítica)          │
└─────────────────────────────────────┘


🎬 Demonstração


📌 Placeholder: Adicione aqui um GIF ou vídeo do pipeline em execução — por exemplo, a saída do terminal ao rodar qa-datalake ingest com validação, upload para S3, e confirmação de contagem no Athena.



[ GIF / Screencast do pipeline aqui ]
Sugestão: grave com asciinema ou OBS e converta para GIF via gifski.


✨ Principais Funcionalidades


Validação de contrato pré-ingestão — garante 30 colunas, tipos ISO, decimais, identifiers, unicidade e flags de conformidade antes de qualquer chamada à AWS.
Arquitetura medallion com S3 — zonas raw (CSV) e curated (Parquet/Snappy), particionadas por ingestion_date.
Proteção contra ingestão duplicada — partições existentes em raw ou curated são rejeitadas automaticamente.
Contagem dupla de integridade — a contagem Athena raw deve igualar a contagem local antes de carregar curated; após o carregamento, a contagem curated também deve igualar.
CLI ergonômica — comandos normalize, validate, plan e ingest com flags claras.
Credenciais zero no repositório — resolução exclusiva via AWS SDK credential chain (IAM Identity Center, perfil CLI ou variáveis de ambiente).
Suite de testes independente de nuvem — sem dependências de AWS; roda com unittest puro.
Baseline reproduzível — lote didático v2.1.0 com contagens verificadas para onboarding e CI.



🔢 Baseline Validado

O lote didático v2.1.0 é mantido como amostra reproduzível e deve retornar exatamente:

IndicadorValorResultados analíticos72Resultados com limite de referência57Resultados sem limite de referência15Resultados conformes com limite50Resultados não-conformes com limite7


O baseline é explícito e opcional. Lotes válidos futuros podem ter totais diferentes.




🛡️ Propriedades de Segurança

PropriedadeGarantiaValidação localExecuta antes de qualquer chamada à AWSContrato rígidoExatamente 30 colunas obrigatóriasTipos validadosISO dates, decimais, identifiers, nulos, unicidade, flagsAnti-duplicataPartições existentes (raw ou curated) são rejeitadasIntegridade rawContagem Athena raw = contagem local CSV antes do curatedIntegridade curatedContagem Athena curated = contagem local CSV após cargaSem sobrescritaNenhuma opção --force ou --overwrite expostaCredenciaisResolvidas pelo AWS SDK chain; nunca armazenadas no projeto


⚙️ Pré-requisitos

Certifique-se de ter os itens abaixo instalados e configurados:


Python >= 3.11
AWS CLI configurado com um perfil IAM Identity Center (ou equivalente)
Acesso AWS com permissões para S3, Glue, e Athena nos recursos declarados no .env
Infraestrutura AWS já provisionada (buckets S3, Glue Crawler, banco de dados Athena)



🚀 Instalação

1. Clone o repositório

bashgit clone https://github.com/engambientalucas-design/QualidadeAmbiental_AWSDataLake.git
cd QualidadeAmbiental_AWSDataLake

2. Crie e ative o ambiente virtual

bashpython -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

3. Instale o pacote com dependências de desenvolvimento

bashpip install -e ".[dev]"

4. Configure as variáveis de ambiente

bashcp .env.example .env
# Abra .env e preencha apenas os nomes dos recursos AWS.
# NÃO coloque Access Keys no .env — use um perfil AWS CLI.


📦 Configuração

Copie .env.example para .env e preencha os nomes dos seus recursos:

dotenv# .env — apenas nomes de recursos, sem segredos
QA_S3_RAW_BUCKET=meu-bucket-raw
QA_S3_CURATED_BUCKET=meu-bucket-curated
QA_GLUE_DATABASE=qualidade_ambiental
QA_ATHENA_OUTPUT_LOCATION=s3://meu-bucket-athena-results/


Para desenvolvimento local, prefira um perfil AWS CLI com IAM Identity Center.
A identidade IAM restrita do Power BI não é uma identidade de ingestão.




💻 Exemplos de Uso

Normalizar um export bruto do SSMS:

bashqa-datalake normalize data/input/export_ssms.csv data/output/dados_conformidade.csv

Validar contra o baseline didático v2.1.0:

bashqa-datalake validate data/sample/dados_conformidade_v2_1_0.csv --baseline

Simular (dry-run) uma ingestão sem escrever na AWS:

bashqa-datalake plan data/sample/dados_conformidade_v2_1_0.csv --ingestion-date 2026-06-22

Ingerir um novo lote:

bashqa-datalake ingest data/output/dados_conformidade.csv --ingestion-date 2026-07-01


⚠️ Não execute a ingestão de amostra contra 2026-06-22: essa partição já existe no ambiente validado e o pipeline irá rejeitá-la corretamente.




🧪 Testes

Execute a suite de testes sem dependências de nuvem:

bashPYTHONPATH=src python -m unittest discover -s tests -v

Os testes cobrem validação de contrato e geração de SQL. Nenhuma credencial ou chamada AWS é necessária.


🗂️ Estrutura do Projeto

textQualidadeAmbiental_AWSDataLake/
│
├── src/
│   └── qa_datalake/          # Pacote Python e CLI (normalize, validate, plan, ingest)
│
├── tests/                    # Suite de testes de contrato e geração SQL
│
├── docs/
│   └── runbook.md            # Procedimento de operação em lote e recuperação
│
├── data/
│   └── sample/               # Dataset didático contratado (v2.1.0)
│
├── .env.example              # Configuração de nomes de recursos sem segredos
├── pyproject.toml
└── README.md


🛠️ Stack Utilizada

CamadaTecnologiaMostrar ImagemLinguagem principal, CLI, validação de contratoMostrar ImagemArmazenamento raw (CSV) e curated (Parquet/Snappy)Mostrar ImagemCrawler e Data CatalogMostrar ImagemQueries SQL serverless sobre S3Mostrar ImagemCamada de relatórios e BIMostrar ImagemFonte de dados (projeto externo contratado)


🗺️ Roadmap


Acompanhe o progresso via GitHub Issues.




 Validação de contrato local (30 colunas, tipos, unicidade)
 Ingestão raw para S3 com particionamento por ingestion_date
 Catalogação via AWS Glue Crawler
 Transformação raw → curated (Parquet/Snappy via Athena)
 Suite de testes independente de nuvem
 Orquestração automática de lotes (ex: AWS Step Functions ou Airflow)
 Provisionamento de infraestrutura como código (Terraform / CDK)
 Monitoramento e alertas de qualidade de dados (ex: AWS CloudWatch)
 Suporte a múltiplas ETAs e sistemas de abastecimento
 Dashboard Power BI publicado como template



🤝 Contribuição

Contribuições são bem-vindas! Para contribuir:


Faça um fork do repositório
Crie uma branch descritiva: git checkout -b feat/nome-da-feature
Faça suas alterações e adicione testes quando aplicável
Certifique-se de que python -m unittest discover -s tests -v passa sem erros
Abra um Pull Request com descrição clara da mudança e motivação


Para reportar bugs ou sugerir melhorias, abra uma Issue.


📄 Licença

Distribuído sob a licença MIT. Consulte o arquivo LICENSE para mais detalhes.


<div align="center">
Desenvolvido por Lucas · Engenharia Ambiental & Engenharia de Dados

Mostrar Imagem

</div>
