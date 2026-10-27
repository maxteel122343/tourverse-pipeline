# Tourverse Pipeline

Protótipo do **Tourverse**: tour virtual 3D cinematográfico de lojas locais.

## Visão

O usuário entra num tour virtual 3D de uma loja real. Vê a cena navegável feita das fotos do estabelecimento, lê os dados (nome, telefone, endereço, avaliação), e conversa com uma IA que guia a experiência — tipo um guia turístico dentro do próprio lugar.

Inspiração visual: vídeo do Marcin AI (site 3D imobiliário cinematográfico) e o repo `eyosushisaobernardo` (tour virtual com 3DVista + Gaussian splats + WebXR).

## Repositórios de referência (clonar antes de começar)

Estes repos **não são submodules** — são referências que o Cursor deve clonar localmente pra entender a API e reutilizar código. Cada um vai numa pasta `references/`:

```bash
mkdir -p references
cd references

# 1. Scraper kit (dados + fotos das lojas, motor Docker)
git clone https://github.com/maxteel122343/google-maps-scraper-kit.git

# 2. Brush (reconstrução 3D open source — GUI, mas a engine por baixo é o gsplat)
git clone https://github.com/ArthurBrussee/brush.git

# 3. PlayCanvas SuperSplat (viewer de splats no navegador)
git clone https://github.com/maxteel122343/supersplat.git
```

### Por que cada um

| Repo | Pasta | Papel |
|------|-------|-------|
| `google-maps-scraper-kit` | `references/google-maps-scraper-kit` | Estágio 1: extrai dados e fotos. Tem `docker compose` pro motor `gosom/google-maps-scraper`. **Não precisa de API key.** |
| `brush` | `references/brush` | Estágio 3: referência de como o treinamento 3D funciona. É GUI — o Cursor deve estudar o código e usar o **gsplat** (CLI) no pipeline, não o Brush direto. |
| `supersplat` | `references/supersplat` | Estágio 4: viewer de Gaussian splats nativo do PlayCanvas. É a base do frontend de exibição. |

## Pipeline em 4 estágios

### Estágio 1 — Scraper (dados + fotos)
- Repo de referência: `references/google-maps-scraper-kit`
- Motor: `gosom/google-maps-scraper` via Docker (`docker compose up -d` dentro da pasta do kit)
- Entrada: URL direta de uma loja ou busca por categoria/cidade
- Saída: JSON com nome, telefone, email, site, endereço, rating, coordenadas e URLs de fotos (thumbnail + campo `images`)
- Comando exemplo: `python3 scripts/scrape.py --keywords-file loja.txt --full`
- **Não precisa de API key** — o `SCRAPER_API_KEY` do AI Studio é irrelevante aqui

### Estágio 2 — Download das fotos
- Script Python lê o JSON do scraper
- Baixa as imagens das URLs do Google
- Salva em `assets/<nome-da-loja>/foto-01.jpg`, etc.
- Filtra URLs que falharam ou expiraram
- Gera `manifest.json` na pasta da loja

### Estágio 3 — Reconstrução 3D
- **Opção A (recomendada pro protótipo): gsplat** — biblioteca Python CLI. Roda headless, sem interface gráfica. É o que o Brush usa por baixo.
  - `gsplat train --data-dir assets/loja-tal --output assets/loja-tal/splat.spz`
- **Opção B: Brush** — open source, roda local, mas é app com GUI. Usar só como referência de código.
- **Opção C: Luma AI** — nuvem, sem GPU local, mas pago acima do gratuito
- Saída: arquivo `.ply` ou `.spz` (Gaussian splat)
- Tempo: 20 fotos ≈ minutos a 1 hora na GPU

### Estágio 4 — Exibição
- Viewer nativo de splats do **PlayCanvas** (repo `references/supersplat`)
- Embutido numa página React (o frontend do fork do scraper já existe em `src/`)
- Página por loja: splat 3D navegável + dados do scraper ao lado (nome, telefone, menu, botão de IA)
- Futuro: IA (Gemini) como guia por voz, multiplayer, Street View

## Stack

| Peça | Tecnologia | Onde |
|------|-----------|------|
| Scraper | Python + Docker | `references/google-maps-scraper-kit` |
| Download | Python | este repo (`scripts/`) |
| Treinamento 3D | gsplat (Python CLI) | este repo (`scripts/`) |
| Viewer | PlayCanvas SuperSplat | `references/supersplat` + este repo (`src/`) |
| Frontend | React + Vite + Tailwind | fork do scraper + este repo |
| IA guia | Gemini (futuro) | — |

## Como rodar (quando implementado)

```bash
# 0. Clonar referências
mkdir -p references && cd references
git clone https://github.com/maxteel122343/google-maps-scraper-kit.git
git clone https://github.com/ArthurBrussee/brush.git
git clone https://github.com/maxteel122343/supersplat.git
cd ..

# 1. Scraper
cd references/google-maps-scraper-kit
docker compose up -d
python3 scripts/scrape.py --keywords-file loja.txt --full
cd ../..

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
- URLs de fotos do Google podem expirar — baixar logo após o scrape

## Próximos passos (pós-protótipo)

1. IA guia por voz (Gemini Live API)
2. Multiplayer no mesmo local
3. Integração Street View
4. App Android (já existe base em `Aluga-Avatar-3d-VERSAO-2.0`)
5. Scan com iPhone (repo `3d-model-anything`) como versão evoluída
