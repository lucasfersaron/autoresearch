# Task: Integracao inicial do autoresearch

## Objetivo

Adicionar o projeto `autoresearch` ao workspace, deixar o ambiente preparado e registrar os limites operacionais desta maquina.

## Checklist

### Workspace

- [x] Clonar repositório em `projects/2_Projetos_Andamento/autoresearch`
- [x] Criar `PROJETO.md`
- [x] Criar `task.md`
- [x] Atualizar `projects/PROJECTS_STATUS.md`

### Setup local

- [x] Validar `uv` instalado
- [x] Ajustar instalação para macOS ARM
- [x] Criar `.venv` com `uv sync`
- [x] Rodar `uv run prepare.py --num-shards 10`
- [ ] Rodar `uv run train.py`

### Handoff operacional

- [x] Criar DISC de bootstrap/CUDA
- [x] Criar runbook para baseline em host CUDA
- [ ] Executar baseline em host CUDA

### Bloqueios atuais

- [x] Registrar que treino requer GPU NVIDIA/CUDA
- [x] Adicionar fail-fast claro em `train.py` para hosts sem CUDA
- [x] Preparar cache local em `~/.cache/autoresearch`
