# DISC - autoresearch Bootstrap CUDA

> **DISC** = Diagnostico · Investigacao · Solucao · Controle
> **Data:** 2026-03-10
> **Severidade:** Alta
> **Escopo:** `projects/2_Projetos_Andamento/autoresearch` · setup local · baseline de treino · operacao em host CUDA

---

## D - DIAGNOSTICO

### Sintoma principal
O projeto `autoresearch` foi integrado ao workspace e esta pronto para leitura, edicao e preparo de dados, mas **nao pode executar treino completo nesta maquina**.

O bloqueio nao esta no codigo-base do repositorio. O bloqueio esta na combinacao:
- host macOS ARM sem GPU NVIDIA,
- pipeline de treino assumindo CUDA do inicio ao fim,
- dependencia original do `torch` apontando para wheel CUDA sem compatibilidade com Darwin.

### Evidencias coletadas

| # | Evidencia | Impacto |
|---|-----------|---------|
| 1 | `uv sync` original falhou com wheel CUDA sem suporte a macOS ARM | Setup local quebrado |
| 2 | `torch.cuda.is_available()` retorna `False` neste host | Treino impossivel aqui |
| 3 | `train.py` usa CUDA e Flash Attention 3 como precondicao estrutural | Sem fallback real para CPU/MPS |
| 4 | `prepare.py --num-shards 10` concluiu com sucesso | Dados e tokenizer estao prontos |
| 5 | Repo local esta `ahead 1` com bootstrap do workspace | Base rastreavel pronta |

### Causa raiz

```text
CAMADA 1 - AMBIENTE
  Host atual nao possui NVIDIA/CUDA

CAMADA 2 - DEPENDENCIAS
  pyproject original priorizava wheel CUDA sem compatibilidade local

CAMADA 3 - OPERACAO
  Faltava explicitar que "setup local" != "host apto para treino"
```

---

## I - INVESTIGACAO

### Estado atual confirmado

- Repositorio clonado em `projects/2_Projetos_Andamento/autoresearch`
- `.venv` criada com `uv`
- cache local preparado em `~/.cache/autoresearch`
- 11 shards baixados
- tokenizer treinado com sucesso
- fail-fast inserido para hosts sem CUDA

### O que esta pronto

1. Ler e editar `program.md`
2. Iterar documentacao e setup
3. Subir o projeto em um host CUDA sem precisar refazer descoberta
4. Rodar baseline assim que houver maquina compativel

### O que nao esta pronto neste host

1. Rodar `uv run train.py`
2. Medir `val_bpb` real
3. Iniciar loop de autoresearch noturno

### Decisao de PM

O projeto ja saiu de "integracao bruta" e entrou em **estado de handoff operacional**.
O proximo gargalo e infraestrutura de execucao, nao planejamento adicional.

---

## S - SOLUCAO

### Estrategia

Separar a iniciativa em duas fases curtas:

1. **Fase A - Bootstrap local**
   - concluir integracao no workspace
   - destravar instalacao em macOS
   - preparar dados e tokenizer

2. **Fase B - Baseline em host CUDA**
   - sincronizar repo em maquina NVIDIA
   - validar `uv sync`
   - executar `uv run train.py`
   - capturar `val_bpb`, `mfu_percent`, `peak_vram_mb`

### Entregaveis da Fase B

- baseline numerico do modelo
- registro do host usado
- copia do output final do treino
- decisao sobre proxima iteracao em `program.md`

### Nao decisao importante

Nao adaptar agora o projeto para CPU ou MPS.

Motivo:
- amplia escopo;
- desvia do objetivo original do repo;
- nao resolve o proximo passo mais valioso, que e obter baseline real em CUDA.

---

## C - CONTROLE

### KPIs

| KPI | Atual | Meta |
|-----|-------|------|
| Setup local com `uv` | ok | ok |
| Cache de dados/tokenizer | ok | ok |
| Baseline `val_bpb` | ausente | 1 primeira medicao |
| Host CUDA validado | nao | sim |
| Repo pronto para push | sim | sim |

### Gates obrigatorios

1. `uv sync` concluido no host alvo
2. `python -c "import torch; print(torch.cuda.is_available())"` retorna `True`
3. `uv run train.py` conclui sem abortar
4. output final do treino e salvo/registrado

### Squad recomendada

| Agente | Responsabilidade |
|--------|------------------|
| `@aios-pm` | congelar contexto, escopo e criterio de pronto |
| `@aios-devops` | push, sync no host CUDA, validacao operacional |
| `@aios-dev` | ajustes pequenos em `program.md` ou `train.py` apos baseline |
| `@aios-qa` | revisar consistencia dos resultados e riscos de reproducibilidade |

### Proxima acao unica

**Dar push do commit local do `autoresearch` e executar o primeiro baseline em uma maquina com GPU NVIDIA/CUDA.**

---

*DISC gerado por @pm (Morgan) · 2026-03-10*
