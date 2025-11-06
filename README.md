# ETL Pipeline — Fusão de Dados de Duas Empresas

Este repositório contém um projeto prático de ETL (Extract, Transform, Load) que une dados de duas empresas (Empresa A e Empresa B) em um único arquivo combinado. O projeto foi desenvolvido como exercício durante um curso na Alura.

## Objetivo

Unir e padronizar dados vindos de duas fontes diferentes (JSON e CSV), aplicando transformações nas colunas quando necessário, e gerar um CSV final pronto para análise.

## Estrutura do repositório

- `data_raw/` — dados de entrada:
  - `dados_empresaA.json` (ex.: lista de dicionários JSON)
  - `dados_empresaB.csv` (CSV com cabeçalho)
- `scripts/` — scripts do pipeline:
  - `processamento_dados.py` — classe `Dados` com métodos de leitura, transformação, união e gravação
  - `fusao_mercado_fev.py` — script que usa a classe para executar o ETL e produzir o resultado
- `data_processed/` — saída do pipeline:
  - `dados_combinados.csv` (gerado pelo pipeline)
- `notebooks/` — análise/exploração (opcional)

## Fluxo ETL (resumido)

1. Extract
   - `Dados.leitura_dados(path, tipo)` lê arquivos JSON ou CSV e retorna uma instância `Dados`.
2. Transform
   - `rename_coluns(mapping)` aplica o mapeamento de nomes de colunas (ex.: padronizar nomes entre as empresas).
   - `join(dadosA, dadosB)` concatena os registros das duas fontes.
3. Load
   - `salvando_dados(path)` grava o resultado final em `data_processed/dados_combinados.csv`.

## Como usar

Recomenda-se usar um ambiente virtual (venv). O projeto usa apenas módulos da biblioteca padrão (`json`, `csv`), portanto não há dependências externas obrigatórias.

No Windows PowerShell:

```powershell
# criar e ativar virtualenv (opcional)
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# executar o script principal
python .\scripts\fusao_mercado_fev.py
```

Após execução, verifique `data_processed/dados_combinados.csv`.

## Observações e problema conhecido

Durante a execução original do pipeline pode ocorrer um problema em que a coluna "Data da Venda" desaparece do conjunto final — você notou que o resultado tinha 5 colunas quando deveriam ser 6. A causa provável é que a implementação atual de `__get_coluns` em `processamento_dados.py` obtém apenas as chaves do último registro lido (por exemplo, `list(self.dados[-1].keys())`). Se apenas alguns registros (da outra fonte) tiverem a chave "Data da Venda", ela não aparecerá nas colunas.

Correção recomendada (resumida): computar a união das chaves de todos os registros preservando a ordem de aparecimento. Também tornar o `rename_coluns` tolerante a chaves faltantes. Exemplo de implementação a ser aplicada em `processamento_dados.py`:

```python
# obter colunas como união de todas as chaves (preservando ordem)
def __get_coluns(self):
    colunas = []
    vistos = set()
    for registro in self.dados:
        for k in registro.keys():
            if k not in vistos:
                vistos.add(k)
                colunas.append(k)
    return colunas

# rename_coluns mais resiliente (mantém chaves não mapeadas)
def rename_coluns(self, key_mapping):
    new_dados = []
    for old_dict in self.dados:
        dict_temp = {}
        for old_key, value in old_dict.items():
            novo_nome = key_mapping.get(old_key, old_key)
            dict_temp[novo_nome] = value
        new_dados.append(dict_temp)
    self.dados = new_dados
    self.nome_colunas = self.__get_coluns()
```

Com essa alteração, a coluna "Data da Venda" (ou qualquer outra coluna que apareça apenas em uma das fontes) será incluída em `nome_colunas` e preservada na saída final. Registros que não possuírem um valor para uma coluna específica receberão o valor `Indisponível` no CSV final (comportamento já implementado).

## Boas práticas e próximos passos sugeridos

- Adicionar logging para acompanhar o progresso do ETL (`logging` da stdlib).
- Criar testes unitários simples (ex.: leitura, renomeação e join) usando `pytest`.
- Validar e normalizar tipos (datas, números) durante a transformação.
- Adicionar um `requirements.txt` se decidir usar bibliotecas externas (pandas, pytest, etc.).
- Incluir um exemplo mínimo de `notebooks/` demonstrando leitura do `dados_combinados.csv` e análise rápida.

