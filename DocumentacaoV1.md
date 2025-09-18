# Documentação da Solução - Sistema de Previsão de Vendas

## Visão Geral

Esta solução implementa um sistema completo de previsão de vendas semanais utilizando machine learning para competições de hackathon. O objetivo é prever as quantidades de venda por combinação PDV-Produto para as primeiras 5 semanas de Janeiro de 2023, baseando-se exclusivamente em dados históricos de 2022.

## Arquitetura da Solução

### Stack Tecnológica

A solução utiliza uma combinação de tecnologias otimizada para processamento eficiente de grandes volumes de dados:

- **DuckDB**: Banco de dados analítico em memória para processamento rápido de consultas complexas
- **LightGBM**: Algoritmo de gradient boosting otimizado para problemas de regressão
- **Pandas**: Manipulação e análise de dados estruturados
- **NumPy**: Operações matemáticas de alta performance
- **Scikit-learn**: Ferramentas de pré-processamento e métricas de avaliação

### Componentes Principais

O sistema é estruturado em uma classe principal `PrevisaoVendasHackathon` que encapsula todo o pipeline de machine learning, desde o carregamento dos dados até a geração das previsões finais.

## Pipeline de Processamento

### 1. Carregamento e Integração de Dados

O sistema integra três fontes de dados principais:
- Dados de lojas (PDVs) com informações geográficas e categóricas
- Transações históricas com quantidades, valores e datas
- Catálogo de produtos com hierarquias categóricas

A integração é realizada através de joins SQL otimizados no DuckDB, garantindo consistência e performance mesmo com volumes grandes de dados. O sistema automaticamente converte identificadores para tipos apropriados (BIGINT) para suportar IDs de grande magnitude.

### 2. Agregação Temporal

Os dados transacionais diários são agregados em períodos semanais, criando as seguintes métricas:
- Quantidade total vendida por semana
- Número de transações e dias únicos de venda
- Valores monetários agregados
- Preço unitário médio calculado dinamicamente

### 3. Engenharia de Features

O sistema cria um conjunto abrangente de características para alimentar o modelo de machine learning:

**Features Temporais:**
- Codificação cíclica de semanas e meses usando funções seno/cosseno
- Características de sazonalidade e tendência

**Features de Lag:**
- Vendas das 1, 2 e 4 semanas anteriores
- Médias móveis de 4 e 8 semanas
- Features de momentum baseadas em diferenças temporais

**Features Cross-sectional:**
- Médias de venda por produto across todas as lojas
- Médias de venda por loja across todos os produtos
- Intensidade transacional e padrões de atividade

**Features Categóricas:**
- Encoding automático de variáveis categóricas usando LabelEncoder
- Tratamento robusto de categorias não observadas

### 4. Validação Temporal

A solução implementa uma estratégia de validação temporal rigorosa:
- Treino: semanas 1-48 de 2022
- Validação: semanas 49-52 de 2022
- Métrica principal: WMAPE (Weighted Mean Absolute Percentage Error)

Esta abordagem garante que o modelo seja avaliado em condições realísticas, respeitando a natureza temporal dos dados de vendas.

### 5. Modelagem com LightGBM

O modelo utiliza configurações otimizadas para problemas de previsão de vendas:

**Configurações Principais:**
- Objetivo: Poisson (adequado para dados de contagem)
- Número de folhas: 127 (balanço entre complexidade e overfitting)
- Taxa de aprendizado: 0.05 (conservadora para estabilidade)
- Feature fraction: 0.8 (regularização através de amostragem)
- Early stopping: 50 rounds (prevenção de overfitting)

**Estratégia de Treinamento:**
- Treinamento inicial com validação para otimização de hiperparâmetros
- Re-treinamento final em todo o dataset de 2022 para maximizar informação

### 6. Geração de Previsões

O sistema gera previsões para todas as combinações PDV-Produto observadas nos dados históricos:

**Processo de Inferência:**
- Extração do estado mais recente de cada combinação PDV-Produto
- Construção de features para as semanas 1-5 de Janeiro 2023
- Processamento em batches para otimização de memória
- Garantia de não-negatividade nas previsões

**Otimizações de Performance:**
- Processamento vetorizado usando pandas
- Batching inteligente para gerenciamento de memória
- Reutilização de cálculos de features temporais

## Qualidade e Robustez

### Tratamento de Dados

O sistema implementa tratamento robusto de situações comuns em dados reais:
- Valores faltantes são imputados com estratégias específicas por tipo de feature
- Categorias não observadas são mapeadas para valores especiais
- Transações com quantidade zero ou negativa são filtradas
- IDs de grande magnitude são suportados através de tipos apropriados

### Validação de Qualidade

Multiple camadas de validação garantem a qualidade da solução:
- Verificação de integridade dos dados de entrada
- Validação de formatos de saída conforme especificações do hackathon
- Checagem de consistência estatística das previsões
- Verificação de não-negatividade e tipos de dados

### Monitoramento e Debugging

O sistema fornece feedback detalhado durante a execução:
- Estatísticas de carregamento e processamento de dados
- Métricas de performance do modelo
- Progresso de geração de previsões
- Alertas para situações anômalas

## Formato de Saída

As previsões são exportadas no formato exato requerido pelo hackathon:
- Formato CSV com separador ponto-e-vírgula
- Encoding UTF-8
- Colunas: semana, pdv, produto, quantidade
- Todas as quantidades como números inteiros não-negativos

## Performance e Escalabilidade

A solução é otimizada para datasets de grande escala:

**Otimizações de Memória:**
- Uso do DuckDB para processamento out-of-core
- Processamento em batches durante a inferência
- Limpeza automática de objetos intermediários

**Otimizações de CPU:**
- Paralelização automática via LightGBM
- Uso eficiente de múltiplos cores do DuckDB
- Operações vetorizadas com NumPy/Pandas

**Configurabilidade:**
- Limites de memória ajustáveis
- Número de threads configurável
- Tamanho de batches adaptável

## Limitações e Considerações

**Dependências de Dados:**
- Requer dados históricos consistentes de 2022
- Assume continuidade das combinações PDV-Produto
- Previsões limitadas a combinações observadas historicamente

**Assumptions do Modelo:**
- Padrões de sazonalidade se mantêm consistentes
- Não considera eventos externos ou promoções futuras
- Assume estabilidade do mix de produtos por loja

**Restrições Técnicas:**
- Uso obrigatório do LightGBM conforme regras do hackathon
- Formato de saída fixo e não-negociável
- Processamento limitado a dados de 2022 apenas

## Manutenção e Extensões Futuras

A arquitetura modular permite extensões futuras:
- Adição de novas fontes de dados (promoções, eventos, clima)
- Incorporação de features externas (feriados, sazonalidade regional)
- Ensemble de múltiplos modelos
- Otimização de hiperparâmetros automatizada
- Deploy em ambiente de produção com API REST