# Time de Agentes de Marketing

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


Sistema de automaÃ§Ã£o de conteÃºdo para redes sociais usando **5 agentes de IA coordenados**. Os agentes pesquisam, criam textos, geram imagens, renderizam vÃ­deos e preparam a distribuiÃ§Ã£o de campanhas de marketing â€” tudo automatizado.

O projeto funciona com o **Claude Code** como motor dos agentes. Cada agente Ã© uma skill especializada que o Claude executa via CLI. Um orquestrador (BullMQ + Redis) coordena a ordem de execuÃ§Ã£o e as dependÃªncias entre agentes.

A marca de demonstraÃ§Ã£o usada Ã© a **Cold Brew Coffee Co.**, mas o sistema serve para qualquer marca â€” basta trocar os knowledge files e assets.

> DocumentaÃ§Ã£o tÃ©cnica completa da arquitetura em [`CLAUDE.md`](CLAUDE.md).

---

## Como funciona

```
VocÃª define uma campanha (JSON ou conversa)
    â†“
1. Agente de Pesquisa â†’ pesquisa mercado via Tavily
    â†“
2. TrÃªs agentes criativos rodam em paralelo:
   â”œâ”€â”€ Designer de Ads â†’ imagens para carousel + stories (Playwright)
   â”œâ”€â”€ Especialista de VÃ­deo â†’ roteiro + renderizaÃ§Ã£o (Remotion)
   â””â”€â”€ Copywriter â†’ textos para Instagram, Threads, YouTube
    â†“
3. Agente de DistribuiÃ§Ã£o â†’ sobe mÃ­dia no Supabase + monta guia de publicaÃ§Ã£o
    â†“
Resultado: pasta com tudo pronto para publicar
```

### Os 5 agentes

| Agente | O que faz | Ferramenta |
|---|---|---|
| **Pesquisa de Marketing** | Pesquisa tendÃªncias, concorrentes, pÃºblico-alvo | Tavily AI SDK |
| **Designer de Ads** | Cria imagens para carousel e stories | Playwright (HTMLâ†’PNG) |
| **Especialista de VÃ­deo** | Gera roteiro e renderiza vÃ­deos com efeitos | Remotion |
| **Copywriter** | Escreve textos adaptados por plataforma | Claude |
| **DistribuiÃ§Ã£o** | Sobe mÃ­dia e monta o guia de publicaÃ§Ã£o | Supabase |

---

## Recursos de MÃ­dia

O sistema usa mÃºltiplos providers para imagens, Ã¡udio e vÃ­deo. Escolhe automaticamente o melhor disponÃ­vel baseado nas API keys configuradas.

### Imagens

| Provider | Custo | Tipo | Link para API key |
|---|---|---|---|
| **Kie.ai Z-Image** (padrÃ£o) | ~$0.004/img | GeraÃ§Ã£o por IA | [kie.ai/api-key](https://kie.ai/api-key) |
| DALL-E 3 | ~$0.04/img | GeraÃ§Ã£o por IA | [platform.openai.com](https://platform.openai.com/api-keys) |
| Stability AI | ~$0.003/img | GeraÃ§Ã£o por IA | [platform.stability.ai](https://platform.stability.ai/account/keys) |
| **Pexels** | GrÃ¡tis | Fotos stock | [pexels.com/api](https://www.pexels.com/api/) |
| Unsplash | GrÃ¡tis | Fotos stock | [unsplash.com/developers](https://unsplash.com/oauth/applications) |
| **Pixabay** | GrÃ¡tis | Fotos + Ã¡udio + mÃºsica | [pixabay.com/api/docs](https://pixabay.com/api/docs/) |

### NarraÃ§Ã£o / Voz (TTS)

| Provider | Custo | Qualidade | Link |
|---|---|---|---|
| **ElevenLabs** | $0.30/1k chars | Excelente | [elevenlabs.io](https://elevenlabs.io/app/settings/api-keys) |
| OpenAI TTS | $0.015/1k chars | Alta | [platform.openai.com](https://platform.openai.com/api-keys) |
| MiniMax | ~$0.01/1k chars | Alta | [api.minimax.chat](https://api.minimax.chat) |
| **Piper (local)** | GrÃ¡tis | Boa | `pip install piper-tts` |

### Efeitos Sonoros e MÃºsica

| Provider | Custo | Tipo | Link |
|---|---|---|---|
| **Pixabay** | GrÃ¡tis | SFX + MÃºsica | [pixabay.com/api/docs](https://pixabay.com/api/docs/) |
| Freesound | GrÃ¡tis | SFX (500k+ sons) | [freesound.org/apiv2/apply](https://freesound.org/apiv2/apply) |
| ElevenLabs | Pago | GeraÃ§Ã£o de mÃºsica por IA | [elevenlabs.io](https://elevenlabs.io) |
| Suno AI | ~$0.05/track | GeraÃ§Ã£o de mÃºsica por IA | [suno.ai](https://suno.ai) |

### VÃ­deos (Remotion)

| ComposiÃ§Ã£o | Formato | Uso |
|---|---|---|
| **DynamicAd** | 1080x1920 (9:16) | Reels, Stories, Shorts |
| **DynamicAdSquare** | 1080x1080 (1:1) | Feed Instagram |
| ColdBrewAd | 1080x1080 (1:1) | Template fixo (fallback) |

### SeleÃ§Ã£o automÃ¡tica de provider

O sistema escolhe o provider baseado nas keys configuradas:

```
Imagem â†’ Kie.ai â†’ DALL-E â†’ Stability â†’ Pexels/Unsplash/Pixabay (fallback grÃ¡tis)
Voz    â†’ ElevenLabs â†’ OpenAI TTS â†’ MiniMax â†’ Piper local (fallback grÃ¡tis)
MÃºsica â†’ Pixabay (grÃ¡tis) ou Suno/ElevenLabs (IA paga)
```

Verificar quais estÃ£o ativos: `npm run media:status`

---

## Efeitos de CÃ¢mera nos VÃ­deos

Movimentos aplicados sobre fotos de fundo simulando cÃ¢mera cinematogrÃ¡fica:

| Efeito | DescriÃ§Ã£o | Uso automÃ¡tico |
|---|---|---|
| `push-in` | Zoom rÃ¡pido para o centro | Hooks, impacto |
| `pull-out` | Zoom afastando | RevelaÃ§Ãµes |
| `ken-burns-in` | Zoom lento e suave | Produto, close-ups |
| `ken-burns-out` | Zoom afastando suave | Flashbacks |
| `pan-left` / `pan-right` | PanorÃ¢mica horizontal | Narrativa |
| `pan-up` / `pan-down` | PanorÃ¢mica vertical | EsperanÃ§a, calma |
| `drift` | Movimento aleatÃ³rio sutil | Emocional, onÃ­rico |
| `parallax-zoom` | Zoom com drift vertical | DinÃ¢mico |
| `tilt-shift` | Zoom com rotaÃ§Ã£o | ArtÃ­stico |
| `breathe` | Pulso de escala sutil | CTAs, finais |

### AnimaÃ§Ãµes de Texto

| AnimaÃ§Ã£o | DescriÃ§Ã£o | Uso automÃ¡tico |
|---|---|---|
| `blur-in` | Surge de desfocado para nÃ­tido | Hooks |
| `slide-up` / `slide-down` | Desliza vertical | Produto |
| `per-word` | Cada palavra surge separadamente | ConexÃ£o, benefÃ­cios |
| `punch-in` | Palavras com spring de impacto | Presentes, reveals |
| `bounce-in` | Palavras com bounce | CTAs |
| `typewriter` | Letra por letra com cursor | Flashbacks, memÃ³rias |
| `scale-up` | Cresce de pequeno para normal | Destaque |
| `fade` | Fade in simples | Close-ups |

### SeleÃ§Ã£o automÃ¡tica por tipo de cena

| Tipo de Cena | CÃ¢mera | Overlay | Texto |
|---|---|---|---|
| `hook` | `push-in` | escuro | `blur-in` |
| `produto_em_acao` | `ken-burns-in` | frio | `slide-up` |
| `conexao_emocional` | `drift` | quente | `per-word` |
| `flashback_infancia` | `ken-burns-out` | sÃ©pia | `typewriter` |
| `flashback_adolescencia` | `pan-left` | sÃ©pia | `typewriter` |
| `presente` | `push-in` | escuro | `punch-in` |
| `cta` | `breathe` | claro | `bounce-in` |

O sistema resolve nesta ordem: valor explÃ­cito no JSON â†’ detecÃ§Ã£o por palavras no roteiro â†’ mapeamento pelo tipo â†’ default.

---

## Regras de Texto sobre Imagens

Quando fotos reais sÃ£o usadas como fundo, o sistema segue estas regras:

1. **Analisar a imagem antes** de posicionar texto
2. **Nunca cobrir rostos** â€” texto sÃ³ em Ã¡reas livres
3. **Texto deve caber no frame** â€” padding 40px, overflow hidden
4. **Gradientes localizados** â€” sÃ³ na zona do texto, opacidade 0.4-0.7

| Imagem com... | Onde colocar texto |
|---|---|
| Pessoas no centro | RodapÃ© ou topo |
| Pessoas na esquerda | Direita |
| Produto no centro | Topo ou rodapÃ© |
| Texto jÃ¡ na imagem | NÃ£o duplicar na mesma Ã¡rea |

---

## Fluxo de GeraÃ§Ã£o de VÃ­deos

```
1. Agente de VÃ­deo (Claude) gera scene_plan.json
   â†“
2. Scene plan define: cenas, timing, texto, cores, cÃ¢mera, imagens
   â†“
3. render-video.js passa o JSON como props para o Remotion
   â†“
4. DynamicAd.tsx monta <Sequence> por cena com:
   - Foto de fundo + efeito de cÃ¢mera
   - Texto animado
   - Produto com glow (se houver)
   - NarraÃ§Ã£o contÃ­nua + mÃºsica de fundo
   â†“
5. Output: ad_01.mp4, ad_02.mp4, etc.
```

### Ãudio nos vÃ­deos

```json
{
  "narration_file": "audio/v1_full.mp3",
  "narration_volume": 1,
  "background_music": "audio/bgm_v1.mp3",
  "background_music_volume": 0.2
}
```

- NarraÃ§Ã£o Ã© um Ã¡udio Ãºnico e contÃ­nuo (nÃ£o cortado por cena)
- DuraÃ§Ã£o das cenas acompanha o ritmo da voz
- CTA final fica com hold longo (5-9s) depois da voz terminar
- MÃºsica de fundo a 20% do volume para nÃ£o competir

---

## PublicaÃ§Ã£o no Instagram

Para publicar via API, Ã© necessÃ¡rio:

1. **Conta Business ou Creator** no Instagram vinculada a uma Facebook Page
2. **App no Meta Developer** com o produto "Instagram Graph API"
3. **Credenciais**:
   - `INSTAGRAM_ACCOUNT_ID` â€” ID numÃ©rico da conta (`GET /me?fields=instagram_business_account`)
   - `INSTAGRAM_ACCESS_TOKEN` â€” token com permissÃµes `instagram_business_basic` + `instagram_business_content_publish`
4. Token expira em **60 dias** â€” renovar antes
5. Imagens devem estar em **URL pÃºblica** (o sistema usa Supabase)

---

## Supabase

O projeto usa o Supabase **apenas como Storage** para hospedar os arquivos de mÃ­dia das campanhas. NÃ£o hÃ¡ tabelas no banco de dados â€” todos os metadados (captions, research, layouts, URLs) ficam em arquivos locais na pasta `outputs/`.

### O que Ã© armazenado

O Agente de DistribuiÃ§Ã£o faz upload de imagens (PNG) e vÃ­deos (MP4) para o bucket e gera URLs pÃºblicas que sÃ£o usadas para publicaÃ§Ã£o via API (Instagram, YouTube).

```
Campanha gerada
    â†“
Upload de PNGs e MP4s â†’ Supabase Storage (bucket: campaign-uploads)
    â†“
URLs pÃºblicas salvas em media_urls.json (local)
    â†“
URLs usadas para publicar via Instagram Graph API / YouTube Data API
```

### ConfiguraÃ§Ã£o

1. Criar um bucket chamado `campaign-uploads` marcado como **pÃºblico**
2. Adicionar no `.env`:

```
SUPABASE_URL=sua-url
SUPABASE_SERVICE_ROLE_KEY=sua-key
```

### Exemplo de media_urls.json

```json
{
  "campanha_carousel_01.png": "https://xxx.supabase.co/storage/v1/object/public/campaign-uploads/campanha_carousel_01.png",
  "campanha_ad_01.mp4": "https://xxx.supabase.co/storage/v1/object/public/campaign-uploads/campanha_ad_01.mp4"
}
```

### Publish MD â€” O guia de publicaÃ§Ã£o

ApÃ³s o upload no Supabase, o Agente de DistribuiÃ§Ã£o gera um arquivo `Publish <task_name> <date>.md` com tudo pronto para publicar. Esse arquivo funciona como um **guia completo da campanha** e contÃ©m:

| SeÃ§Ã£o | O que tem |
|---|---|
| Status | Checklist por plataforma (Instagram, YouTube, Threads) |
| MÃ­dias hospedadas | Tabela com todos os arquivos, tipo, plataforma e URL pÃºblica |
| Instagram â€” Carrossel | Slides na ordem exata + caption com hashtags |
| Instagram â€” Stories | SequÃªncia de stories com texto e instruÃ§Ãµes de formato |
| YouTube | TÃ­tulo, descriÃ§Ã£o, tags e URL do vÃ­deo por vÃ­deo |
| Threads | Texto pronto para colar (publicaÃ§Ã£o manual) |
| Agendamento | CalendÃ¡rio da semana com datas, horÃ¡rios e justificativa estratÃ©gica |
| InstruÃ§Ãµes de execuÃ§Ã£o | Como acionar a publicaÃ§Ã£o via API |

### Trava de publicaÃ§Ã£o

A publicaÃ§Ã£o real nas APIs (Instagram Graph API, YouTube Data API) **sÃ³ acontece quando o usuÃ¡rio referencia o Publish MD pelo nome**:

```
Executar Publish dia_das_maes 2026-05-10.md
```

Sem essa referÃªncia explÃ­cita, nenhuma chamada de API Ã© feita. Isso garante que o usuÃ¡rio sempre revisa o conteÃºdo antes de publicar.

```
Pipeline completo
    â†“
Publish MD gerado com todo o conteÃºdo
    â†“
UsuÃ¡rio revisa captions, hashtags, agendamento
    â†“
UsuÃ¡rio referencia o arquivo pelo nome â†’ publicaÃ§Ã£o executa
    â†“
Instagram: Graph API (container â†’ publish)
YouTube: Data API (upload com tÃ­tulo/descriÃ§Ã£o/tags)
Threads: manual (sem API pÃºblica)
```

---

## Estrutura do Projeto

```
timesmkt/
â”œâ”€â”€ assets/                  # Imagens da marca (fotos de produto)
â”œâ”€â”€ knowledge/               # Identidade, guidelines, briefing
â”œâ”€â”€ skills/                  # DefiniÃ§Ãµes dos 5 agentes + media-help
â”œâ”€â”€ media/                   # MÃ³dulo multi-provider (imagem, voz, SFX, mÃºsica)
â”œâ”€â”€ pipeline/                # Orquestrador, worker, render de vÃ­deo
â”‚   â”œâ”€â”€ orchestrator.js      # Enfileira jobs no BullMQ
â”‚   â”œâ”€â”€ worker.js            # Executa agentes via Claude CLI
â”‚   â”œâ”€â”€ render-video.js      # Renderiza vÃ­deo com Remotion
â”‚   â””â”€â”€ payloads/            # JSONs de campanha
â”œâ”€â”€ remotion-ad/             # Projeto Remotion (vÃ­deo)
â”‚   â”œâ”€â”€ src/DynamicAd.tsx    # ComposiÃ§Ã£o dinÃ¢mica
â”‚   â”œâ”€â”€ src/scenes/          # Cenas por tipo
â”‚   â”œâ”€â”€ src/components/      # CameraMotion, TextOverlay, etc.
â”‚   â””â”€â”€ public/              # Assets e Ã¡udios
â”œâ”€â”€ outputs/                 # Campanhas geradas
â”œâ”€â”€ .env.example             # Template de variÃ¡veis
â””â”€â”€ CLAUDE.md                # DocumentaÃ§Ã£o tÃ©cnica completa
```

---

## Exemplo de Campanha â€” Dia das MÃ£es

Campanha gerada com fotos, vÃ­deos com narraÃ§Ã£o e mÃºsica.

### VÃ­deo 1 â€” "Domingo com Ela" (21s)

| Cena | Tempo | Background | CÃ¢mera | NarraÃ§Ã£o |
|---|---|---|---|---|
| Hook | 0-3s | GrÃ£os de cafÃ© | `push-in` | *"Domingos tÃªm um cheiro especial."* |
| ConexÃ£o | 3-6.5s | MÃ£e e filha | `drift` | *"Ela ensinou tudo. AtÃ© o cafÃ© perfeito."* |
| Produto | 6.5-9.5s | Cold brew | `ken-burns-in` | *"Suave. Gelado. Perfeito."* |
| Brinde | 9.5-12s | Cozinha | `parallax-zoom` | *"Um brinde pra ela."* |
| **CTA** | **12-21s** | **AbraÃ§o** | `breathe` | *"Presente perfeito para quem te deu tudo."* + **hold** |

### VÃ­deo 2 â€” "Anos de CafÃ©" (19s)

| Cena | Tempo | Background | CÃ¢mera | NarraÃ§Ã£o |
|---|---|---|---|---|
| Hook | 0-2.7s | GrÃ£os de cafÃ© | `push-in` | *"Ela sempre teve um cafÃ© pra te oferecer."* |
| InfÃ¢ncia | 2.7-5.8s | Cozinha (sÃ©pia) | `ken-burns-out` | *"Quando vocÃª tinha cinco anos..."* |
| AdolescÃªncia | 5.8-8.8s | MÃ£e e filha (sÃ©pia) | `pan-left` | *"...e quando vocÃª tinha dezessete."* |
| Presente | 8.8-12s | Cold brew (flash) | `push-in` | *"Hoje Ã© a sua vez de cuidar dela."* |
| **CTA** | **12-19s** | **AbraÃ§o** | `breathe` | *"Cada gole Ã© um abraÃ§o."* + **hold** |

---

## Problemas Comuns

| Problema | SoluÃ§Ã£o |
|---|---|
| `npm install` falha | Instalar Node.js v18+ |
| Playwright nÃ£o renderiza | `npx playwright install chromium` |
| Redis nÃ£o conecta | `docker run -d --name redis -p 6379:6379 redis:alpine` |
| Remotion falha | `cd remotion-ad && npm install` |
| ElevenLabs 401 | Verificar key e permissÃµes em elevenlabs.io |
| VÃ­deo sai igual ao template fixo | Passar o scene_plan.json como segundo argumento no render-video.js |
| Texto sobre rosto nas imagens | Seguir regras de posicionamento (analisar imagem antes) |

---

## ComeÃ§ando do Zero

### 1. Instalar

```bash
npm install
cd remotion-ad && npm install && cd ..
npx playwright install chromium
```

### 2. Redis

```bash
docker run -d --name redis -p 6379:6379 redis:alpine
```

### 3. Configurar .env

```bash
cp .env.example .env
```

Preencher pelo menos:

```
TAVILY_API_KEY=sua-key
SUPABASE_URL=sua-url
SUPABASE_SERVICE_ROLE_KEY=sua-key
```

Recomendados:

```
KIE_API_KEY=sua-key
PEXELS_API_KEY=sua-key
PIXABAY_API_KEY=sua-key
ELEVENLABS_API_KEY=sua-key
```

### 4. Rodar

```bash
# Terminal 1
node pipeline/worker.js

# Terminal 2
node pipeline/orchestrator.js --file pipeline/payloads/dia_das_maes_2026.json
```

### 5. Renderizar vÃ­deo avulso

```bash
node pipeline/render-video.js output.mp4 scene_plan.json
```

### 6. Ver providers ativos

```bash
npm run media:status
```

O pipeline roda os 5 agentes e salva tudo em `outputs/`.

---

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="32" />
  <p><strong>Arckhé AI</strong></p>
</div>