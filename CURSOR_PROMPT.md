# Prompt mestre pro Cursor

Cole isso no chat do Cursor com o repo aberto:

---

## Tarefa

Implementar o pipeline Tourverse em 4 estágios neste repositório. Lê o README.md primeiro — ele tem a especificação completa.

## Estágio 1 — Scraper (já existe em outro repo)

O scraper vive em `maxteel122343/google-maps-scraper-kit`. **Não reimplementar.** Só criar um script aqui que:
1. Chama o scraper via subprocess ou lê o JSON de saída dele
2. Aceita uma URL de loja do Google Maps ou um arquivo de keywords
3. Salva o JSON resultante em `output/<timestamp>-loja.json`

Exemplo de interface:
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
3. Exporta o resultado como `.spz` (mais leve que .ply) em `assets/<loja>/splat.spz`
4. Mostra progresso e tempo estimado

Comando interno esperado (ajustar conforme a CLI real do gsplat):
```bash
gsplat train --data-dir assets/loja-tal --output assets/loja-tal/splat.spz
```

Se a CLI do gsplat for diferente, adaptar. Prioridade: funcionar com o mínimo de flags.

## Estágio 4 — Viewer PlayCanvas

Criar uma página React em `src/` (o frontend do fork do scraper já tem estrutura em `src/` com App.tsx, ScrapeForm, LeadsView — reutilizar):
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

## Ordem de implementação

1. `scripts/run_scraper.py`
2. `scripts/download_photos.py`
3. `scripts/train_splat.py`
4. `src/pages/LojaPage.tsx` + rota
5. Testar o fluxo completo com UMA loja real

## Critério de pronto

Rodar os 4 comandos em sequência e abrir `localhost:3000/loja/1` vendo o splat 3D navegável + dados da loja na tela.

---

Quando terminar cada estágio, roda e mostra o output. Se algo quebrar, para e explica o erro antes de continuar.
