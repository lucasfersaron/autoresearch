# autoresearch - Documentacao do Projeto

> **Nivel**: PROJETO  
> **Workspace**: `/Users/lucas/workspace`  
> **Ultima Atualizacao**: 2026-03-10

---

## Visao Geral

`autoresearch` e um experimento de pesquisa autonoma para treino de modelos pequenos com agentes iterando em `train.py` dentro de uma janela fixa de 5 minutos por experimento.

No workspace, ele foi integrado como projeto de exploracao em `2_Projetos_Andamento`. O checkout local esta preparado para inspecao, setup com `uv` e ajustes de `program.md`, mas a execucao de treino continua dependente de uma maquina com GPU NVIDIA e CUDA.

**Status**: 🟡 Andamento

---

## Tech Stack

### Core

- **Linguagem**: Python 3.10+
- **Framework**: PyTorch + scripts Python puros
- **Build Tool**: `uv`

### Principais Dependencias

- `torch`: treino e inferencia do modelo
- `kernels`: acesso aos kernels de Flash Attention
- `rustbpe` + `tiktoken`: tokenizer BPE
- `pyarrow` + `pandas`: leitura e processamento dos shards parquet

### Banco de Dados / Storage

- Cache local em `~/.cache/autoresearch`
- Dados em parquet vindos do Hugging Face

---

## Comandos Essenciais

### Setup Inicial

```bash
uv sync
```

### Desenvolvimento

```bash
# Download de shards e treinamento do tokenizer
uv run prepare.py --num-shards 10

# Treino manual de um experimento
uv run train.py
```

### Troubleshooting Comum

```bash
# Recriar ambiente virtual
rm -rf .venv
uv sync

# Verificar disponibilidade de CUDA
uv run python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

---

## Arquitetura

### Estrutura de Diretorios

```text
autoresearch/
├── prepare.py       # download de dados, tokenizer e utilitarios de runtime
├── train.py         # unico arquivo-alvo de iteracao autonoma
├── program.md       # instrucoes-base para o agente pesquisador
├── pyproject.toml   # dependencias do projeto
└── analysis.ipynb   # notebook auxiliar
```

### Padroes e Convencoes

- **Human edits**: `program.md`
- **Agent edits**: `train.py`
- **Fixed metric**: `val_bpb` (bits por byte), menor e melhor

### Fluxo de Dados

1. `prepare.py` baixa shards do dataset e treina o tokenizer.
2. `train.py` carrega tokenizer, monta o modelo e executa treino de 5 minutos.
3. O agente compara `val_bpb` entre iteracoes e decide manter ou descartar mudancas.

---

## Integracao com Workspace

### Regras Globais

Este projeto segue as regras definidas em:

- [CONSTITUICAO.md](../../_CONFIG_GLOBAL/rules/CONSTITUICAO.md)
- [WORKSPACE.md](../../../WORKSPACE.md)

### Relacao com Outros Projetos

- Nao faz parte da Regra de 3 do `1_Foco_Imediato`.
- Funciona como projeto isolado de P&D em `2_Projetos_Andamento`.

---

## Diretrizes para IA

### Prioridades

1. Preservar a estrutura minima do repo e iterar com mudancas pequenas.
2. Nao assumir suporte local a treino sem antes validar `torch.cuda.is_available()`.

### Nao Modificar

- `prepare.py`, salvo se houver decisao explicita de adaptar a pipeline
- Constantes globais sem registrar impacto no experimento

### Antes de Commits

- [ ] Executar `uv sync`
- [ ] Validar `uv run python -c "import torch; print(torch.cuda.is_available())"`
- [ ] Se houver GPU CUDA, validar `uv run train.py`

---

## Documentacao Adicional

- `README.md`: guia original do projeto
- `program.md`: baseline de instrucao para o agente
- `docs/DISC-AUTORESEARCH-BOOTSTRAP-CUDA-2026-03-10.md`: diagnostico operacional e proxima decisao de execucao
- `docs/RUNBOOK-CUDA-BASELINE.md`: procedimento DevOps para o primeiro treino real em host NVIDIA/CUDA
