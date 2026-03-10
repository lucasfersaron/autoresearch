# Runbook - CUDA Baseline

> **Projeto:** `autoresearch`
> **Objetivo:** executar o primeiro baseline real em host com GPU NVIDIA/CUDA
> **Ultima Atualizacao:** 2026-03-10

---

## Objetivo

Este runbook existe para tirar o projeto do estado "bootstrap concluido" e levar para o primeiro treino real, com coleta de metricas e handoff limpo para a proxima iteracao.

Saida esperada desta execucao:

- `uv sync` concluido no host CUDA
- `uv run train.py` executado ate o fim
- baseline inicial de `val_bpb` registrado
- output bruto salvo para comparacao futura

---

## Pre-condicoes

### Host

- Linux ou Windows com GPU NVIDIA
- driver NVIDIA funcional
- CUDA disponivel para o PyTorch instalado
- espaco em disco para environment + cache de dados

### Acesso

- repositorio atualizado com `origin/master`
- internet liberada para Hugging Face e PyPI/PyTorch

### Validacao minima

```bash
nvidia-smi
python3 --version
uv --version
```

---

## Procedimento

### 1. Clonar ou atualizar o repositorio

```bash
git clone https://github.com/lucasfersaron/autoresearch.git
cd autoresearch

# se o repo ja existir
git pull origin master
```

### 2. Instalar dependencias

```bash
uv sync
```

### 3. Validar CUDA no PyTorch

```bash
uv run python -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'NO CUDA')"
```

Critico:
- `torch.cuda.is_available()` precisa retornar `True`

### 4. Preparar dados e tokenizer

```bash
uv run prepare.py --num-shards 10 --download-workers 4
```

Se o cache ja existir no host, este passo pode reutilizar os artefatos.

### 5. Executar baseline

```bash
mkdir -p runs
uv run train.py | tee runs/baseline-$(date +%Y%m%d-%H%M%S).log
```

### 6. Extrair metricas finais

No final do log, confirmar a presenca de:

- `val_bpb`
- `training_seconds`
- `total_seconds`
- `peak_vram_mb`
- `mfu_percent`
- `total_tokens_M`
- `num_steps`
- `num_params_M`
- `depth`

---

## Criterios de sucesso

Uma execucao e considerada valida se:

1. o processo nao aborta por falta de CUDA;
2. o treino passa da fase inicial de compilacao/warmup;
3. o bloco final de metricas e emitido;
4. o log fica salvo em `runs/`.

---

## Falhas comuns

### `torch.cuda.is_available()` retorna `False`

Causa provavel:
- wheel errada do PyTorch
- driver/CUDA ausente
- ambiente Python inconsistente

Acao:
- recriar environment
- confirmar `nvidia-smi`
- confirmar wheel CUDA do PyTorch

### Erro de OOM

Acao:
- reduzir `DEVICE_BATCH_SIZE` em `train.py`
- manter registro da alteracao antes de comparar resultados

### Download lento ou falho

Acao:
- repetir `uv run prepare.py`
- validar acesso ao Hugging Face

---

## Artefatos a devolver

Ao concluir no host CUDA, devolver:

1. nome do host e GPU usada
2. commit exato executado
3. caminho do log em `runs/`
4. valor de `val_bpb`
5. observacoes sobre throughput, VRAM e falhas

---

## Proxima decisao depois do baseline

Com o baseline em maos:

1. congelar resultado como referencia
2. decidir se a primeira iteracao sera em `program.md` ou `train.py`
3. abrir nova task focada em melhoria de `val_bpb` sem perder comparabilidade
