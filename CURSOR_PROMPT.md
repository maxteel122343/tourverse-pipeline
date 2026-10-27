# Prompt mestre pro Cursor

Cole isso no chat do Cursor com o repo aberto:

---

## Tarefa

Implementar o pipeline Tourverse em 4 estágios neste repositório. Lê o README.md primeiro — ele tem a especificação completa.

## Passo 0 — Clonar repositórios de referência

**Obrigatório antes de qualquer código.** Criar pasta `references/` e clonar os três repos:

```bash
mkdir -p references && cd references
git clone https://github.com/maxteel122343/google-maps-scraper-kit.git
git clone https://github.com/ArthurBrussee/brush.git
git clone https://github.com/maxteel122343/supersplat.git
cd ..
```

Estes repos são a base de tudo:
- `google-maps-scraper-kit`: scraper + Docker. É de onde vêm os dados e as fotos.
- `brush`: engine de reconstrução 3D. Estudar o código, mas o pipeline usa **gsplat** (CLI) — o Brush é GUI.
- `supersplat`: viewer de splats do PlayCanvas. É a base do frontend de exibição.

Não reimplementar nada desses repos. Reutilizar, importar, ou chamar via subprocess.

## Estágio 1 — Scraper

Usar `references/google-maps-scraper-kit`. Criar `scripts/run_scraper.py` que:
1. Sobe o Docker do kit (`docker compose up -d` dentro da pasta dele) ou verifica se já tá rodando
2. Aceita `--url` (URL de uma loja) ou `--keywords-file` + `--city`
3. Chama o scraper do kit via subprocess
4. Salva o JSON em `output/<timestamp>-loja.json`

Exemplo:
```bash
python3 scripts/run_scraper.py --url "https://www.google.com/maps/place/..."
python3 scripts/run_scraper.py --keywords-file loja.txt --city "São Paulo, SP"
```

## Estágio 2 — Download de fotos

Criar `scripts/download_photos.py`:
1. Lê o JSON do estágio 1
2. Baixa todas as URLs de `images` + `thumbnail`
3. Salva em `assets/<nome-sanitizado-da-loja>/foto-01.jpg`...
4. Pula URLs que retornam erro ou redirecionam pra placeholder
5. Gera um `manifest.json` na pasta da loja listando as fotos baixadas com sucesso

## Estágio 3 — Treinamento 3D com gsplat

Criar `scripts/train_splat.py`:
1. Verifica se gsplat está instalado (`pip show gsplat`), se não, instrui a instalação
2. Roda o treinamento apontando pra pasta `assets/<loja>/`
3. Exporta o resultado como `.spz` em `assets/<loja>/splat.spz`
4. Mostra progresso e tempo estimado

Comando interno esperado (ajustar conforme a CLI real do gsplat):
```bash
gsplat train --data-dir assets/loja-tal --output assets/loja-tal/splat.spz
```

Se a CLI do gsplat for diferente, adaptar. Prioridade: funcionar com o mínimo de flags.

## Estágio 4 — Viewer PlayCanvas

Criar uma página React em `src/` (reutilizar a estrutura do fork do scraper: App.tsx, ScrapeForm, LeadsView):
1. Instalar `@playcanvas/supersplat-viewer` ou embutir o viewer via iframe/CDN
2. Criar rota `/loja/:id` que carrega o `splat.spz` da loja
3. Ao lado do splat, mostrar os dados do JSON: nome, telefone, endereço, rating, site
4. Botão "Falar com a IA" (placeholder por agora — a integração com Gemini vem depois)
5. Manter o visual cinematográfico: tela cheia, controles de câmera suaves, sem UI poluída

## Restrições

- Python 3.11+, Node 20+
- Não adicionar dependências pagas
- Não commitar fotos baixadas (adicionar `assets/` ao .gitignore, exceto um `.gitkeep`)
- Código em português nos comentários, inglês nos nomes de variáveis/funções
- Cada estágio deve rodar independente: se o 3 falhar, 1 e 2 já têm output útil
- Os três repos de `references/` devem estar clonados e o Cursor deve **ler o código deles** antes de escrever qualquer coisa

## Ordem de implementação

0. Clonar os 3 repos de referência em `references/`
1. `scripts/run_scraper.py`
2. `scripts/download_photos.py`
3. `scripts/train_splat.py`
4. `src/pages/LojaPage.tsx` + rota
5. Testar o fluxo completo com UMA loja real

## Critério de pronto

Rodar os 4 comandos em sequência e abrir `localhost:3000/loja/1` vendo o splat 3D navegável + dados da loja na tela.

---

Quando terminar cada estágio, roda e mostra o output. Se algo quebrar, para e explica o erro antes de continuar.
