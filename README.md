# Como Rodar o Notebook - Sistema de Previsão de Vendas

## Pré-requisitos

Antes de começar, você precisa ter algumas coisas instaladas no seu sistema:

- **Python 3.12**
- **uv** - gerenciador de pacotes ultrarrápido ([instalar aqui](https://docs.astral.sh/uv/getting-started/installation/))
- **Jupyter** ou **VS Code** com extensão Python/Jupyter

## Configuração do Ambiente

### 1. Criando o Ambiente Virtual

Primeiro, você precisa criar um ambiente virtual isolado. Isso evita conflitos entre diferentes projetos e mantém suas dependências organizadas:

```bash
# No terminal, navegue até a pasta do seu projeto
cd caminho/para/seu/projeto

# Crie um ambiente virtual com uv
uv venv venv-vendas

# Ative o ambiente virtual
# No Windows:
venv-vendas\Scripts\activate
# No Linux/Mac:
source venv-vendas/bin/activate
```

### 2. Instalando as Dependências

Agora que seu ambiente virtual está ativo, você pode instalar todas as bibliotecas necessárias. Esta é sua **primeira célula** do notebook:

```python
# Célula 1: Instalação das Dependências
!uv add pandas numpy lightgbm duckdb ipywidgets scikit-learn --active
!uv sync --active
```

## Preparação dos Dados

### 3. Estrutura de Pastas Recomendada

Organize seus arquivos assim para evitar dor de cabeça:

```
projeto-vendas/
├── data/
│   └── raw/
│       ├── file1.parquet  # Dados das lojas
│       ├── file2.parquet  # Dados das transações
│       └── file3.parquet  # Dados dos produtos
├── notebooks/
│   └── previsao_vendas.ipynb  # Seu notebook
│   └── hackathon_submission.csv  # Resultado final
└── venv-vendas/  # Ambiente virtual
```

### 4. Configuração dos Caminhos dos Arquivos

Na célula onde você executa o sistema, ajuste estes caminhos para apontar para seus arquivos:

```python
# Ajuste estes caminhos conforme sua estrutura
arquivo_loja = "../data/raw/file1.parquet"
arquivo_transacao = "../data/raw/file2.parquet" 
arquivo_produto = "../data/raw/file3.parquet"
arquivo_saida = "../outputs/hackathon_submission.csv"
```

## Executando o Sistema

### 5. Ordem de Execução das Células

Execute as células nesta ordem:

1. **Instalação**
2. **Imports principais** (imports do sistema)
3. **Classe PrevisaoVendasHackathon** (definição da classe)
4. **Função principal** (executar_previsao_hackathon)
5. **Funções utilitárias** (validação)
6. **Execução** (script principal)

### 6. Monitoramento do Progresso

O sistema vai te dar feedback constante. Você vai ver mensagens como:

```
Inicializando DuckDB para previsão do hackathon...
Carregando e juntando conjuntos de dados...
   Registros: 2,450,123
   PDVs: 1,234
   Produtos: 5,678
Criando agregação semanal...
Treinando modelo LightGBM...
```

## Resolução de Problemas Comuns

### Erro de Memória
Se você receber erro de falta de memória, ajuste na classe:
```python
self.inicializar_duckdb(memory_limit='8GB', threads=6)  # Reduza conforme sua máquina
```

### Arquivos Não Encontrados
Verifique se:
- Os caminhos dos arquivos estão corretos
- Os arquivos realmente existem
- Você tem permissão de leitura