# Tourverse Pipeline

Protótipo do **Tourverse**: tour virtual 3D cinematográfico de lojas locais.

## Visão

O usuário entra num tour virtual 3D de uma loja real. Vê a cena navegável feita das fotos do estabelecimento, lê os dados (nome, telefone, endereço, avaliação), e conversa com uma IA que guia a experiência — tipo um guia turístico dentro do próprio lugar.

Inspiração visual: vídeo do Marcin AI (site 3D imobiliário cinematográfico) e o repo `eyosushisaobernardo` (tour virtual com 3DVista + Gaussian splats + WebXR).

## Pipeline em 4 estágios

### Estágio 1 — Scraper (dados + fotos)
- Repo: `maxteel122343/google-maps-scraper-kit` (fork do Mahanaicoach)
- Motor: `gosom/google-maps-scraper` via Docker (`docker compose up -d`)
- Entrada: URL direta de uma loja ou busca por categoria/cidade
- Saída: JSON com nome, telefone, email, site, endereço, rating, coordenadas e URLs de fotos (thumbnail + campo `images`)
- Comando exemplo: `python3 scripts/scrape.py --keywords-file loja.txt --full`
- **Não precisa de API key** — o `SCRAPER_API_KEY` do AI Studio é irrelevante aqui

### Estágio 2 — Download das fotos
- Script Python lê o JSON do scraper
- Baixa as imagens das URLs do Google
- Salva em `assets/<nome-da-loja>/foto-01.jpg`, etc.
- Filtra URLs que falharam ou expiraram
- Nota: fotos do Google Maps são ângulo amplo e baixa resolução — o resultado 3D vai ser limitado

### Estágio 3 — Reconstrução 3D
- **Opção A (recomendada pro protótipo): gsplat** — biblioteca Python CLI que o Brush usa por baixo. Roda headless, sem interface gráfica.
  - `gsplat train --data-dir assets/loja-tal --output assets/loja-tal/splat.ply`
- **Opção B: Brush** — open source, roda local, mas é app com GUI (precisa automatizar ou usar a engine por baixo)
- **Opção C: Luma AI** — nuvem, sem GPU local, mas pago acima do gratuito
- Saída: arquivo `.ply` ou `.spz` (Gaussian splat)
- Tempo: 20 fotos ≈ minutos a 1 hora na GPU

### Estágio 4 — Exibição
- Viewer nativo de splats do **PlayCanvas** (repo `supersplat` no GitHub do usuário)
- Embutido numa página React (o frontend do fork do scraper já existe em `src/`)
- Página por loja: splat 3D navegável + dados do scraper ao lado (nome, telefone, menu, botão de IA)
- Futuro: IA (Gemini) como guia por voz, multiplayer, Street View

## Stack

| Peça | Tecnologia | Repo |
|------|-----------|------|
| Scraper | Python + Docker | `google-maps-scraper-kit` |
| Download | Python | este repo |
| Treinamento 3D | gsplat (Python CLI) | este repo |
| Viewer | PlayCanvas SuperSplat | `supersplat` |
| Frontend | React + Vite + Tailwind | fork do scraper |
| IA guia | Gemini (futuro) | — |

## Como rodar (quando implementado)

```bash
# 1. Scraper
docker compose up -d
python3 scripts/scrape.py --keywords-file loja.txt --full

# 2. Download
python3 scripts/download_photos.py output/loja.json

# 3. Treinar splat
gsplat train --data-dir assets/loja-tal -o assets/loja-tal/splat.spz

# 4. Servir
npm run dev  # abre http://localhost:3000
```

## Limitações conhecidas

- Fotos do Google Maps geram splats borrados/com buracos — protótipo de validação, não demo de cliente
- gsplat precisa de GPU CUDA
- Sem modo headless nativo no Brush (por isso gsplat)
- PlayCanvas viewer de splat é o caminho mais direto pro navegador

## Próximos passos (pós-protótipo)

1. IA guia por voz (Gemini Live API)
2. Multiplayer no mesmo local
3. Integração Street View
4. App Android (já existe base em `Aluga-Avatar-3d-VERSAO-2.0`)
5. Scan com iPhone (repo `3d-model-anything`) como versão evoluída
