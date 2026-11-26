ï»¿# Time de Agentes de Marketing

<div align="center">
  <img src="capa/logo.svg" alt="Arckhé AI" height="48" />
  <p><strong>Arckhé AI</strong></p>
</div>


Sistema de automaêÂ§êÂ£o de conteêÂºdo para redes sociais usando **5 agentes de IA coordenados**. Os agentes pesquisam, criam textos, geram imagens, renderizam vêÂ­deos e preparam a distribuiêÂ§êÂ£o de campanhas de marketing ê¢â¬â tudo automatizado.

O projeto funciona com o **Claude Code** como motor dos agentes. Cada agente êÂ© uma skill especializada que o Claude executa via CLI. Um orquestrador (BullMQ + Redis) coordena a ordem de execuêÂ§êÂ£o e as dependêÂªncias entre agentes.

A marca de demonstraêÂ§êÂ£o usada êÂ© a **Cold Brew Coffee Co.**, mas o sistema serve para qualquer marca ê¢â¬â basta trocar os knowledge files e assets.

> DocumentaêÂ§êÂ£o têÂ©cnica completa da arquitetura em [`CLAUDE.md`](CLAUDE.md).

---

## Como funciona

```
VocêÂª define uma campanha (JSON ou conversa)
    ê¢â â
1. Agente de Pesquisa ê¢â â pesquisa mercado via Tavily
    ê¢â â
2. TrêÂªs agentes criativos rodam em paralelo:
   ê¢âÅê¢ââ¬ê¢ââ¬ Designer de Ads ê¢â â imagens para carousel + stories (Playwright)
   ê¢âÅê¢ââ¬ê¢ââ¬ Especialista de VêÂ­deo ê¢â â roteiro + renderizaêÂ§êÂ£o (Remotion)
   ê¢ââê¢ââ¬ê¢ââ¬ Copywriter ê¢â â textos para Instagram, Threads, YouTube
    ê¢â â
3. Agente de DistribuiêÂ§êÂ£o ê¢â â sobe mêÂ­dia no Supabase + monta guia de publicaêÂ§êÂ£o
    ê¢â â
Resultado: pasta com tudo pronto para publicar
```

### Os 5 agentes

| Agente | O que faz | Ferramenta |
|---|---|---|
| **Pesquisa de Marketing** | Pesquisa tendêÂªncias, concorrentes, pêÂºblico-alvo | Tavily AI SDK |
| **Designer de Ads** | Cria imagens para carousel e stories | Playwright (HTMLê¢â âPNG) |
| **Especialista de VêÂ­deo** | Gera roteiro e renderiza vêÂ­deos com efeitos | Remotion |
| **Copywriter** | Escreve textos adaptados por plataforma | Claude |
| **DistribuiêÂ§êÂ£o** | Sobe mêÂ­dia e monta o guia de publicaêÂ§êÂ£o | Supabase |

---

## Recursos de MêÂ­dia

O sistema usa mêÂºltiplos providers para imagens, êÂ¡udio e vêÂ­deo. Escolhe automaticamente o melhor disponêÂ­vel baseado nas API keys configuradas.

### Imagens

| Provider | Custo | Tipo | Link para API key |
|---|---|---|---|
| **Kie.ai Z-Image** (padrêÂ£o) | ~$0.004/img | GeraêÂ§êÂ£o por IA | [kie.ai/api-key](https://kie.ai/api-key) |
| DALL-E 3 | ~$0.04/img | GeraêÂ§êÂ£o por IA | [platform.openai.com](https://platform.openai.com/api-keys) |
| Stability AI | ~$0.003/img | GeraêÂ§êÂ£o por IA | [platform.stability.ai](https://platform.stability.ai/account/keys) |
| **Pexels** | GrêÂ¡tis | Fotos stock | [pexels.com/api](https://www.pexels.com/api/) |
| Unsplash | GrêÂ¡tis | Fotos stock | [unsplash.com/developers](https://unsplash.com/oauth/applications) |
| **Pixabay** | GrêÂ¡tis | Fotos + êÂ¡udio + mêÂºsica | [pixabay.com/api/docs](https://pixabay.com/api/docs/) |

### NarraêÂ§êÂ£o / Voz (TTS)

| Provider | Custo | Qualidade | Link |
|---|---|---|---|
| **ElevenLabs** | $0.30/1k chars | Excelente | [elevenlabs.io](https://elevenlabs.io/app/settings/api-keys) |
| OpenAI TTS | $0.015/1k chars | Alta | [platform.openai.com](https://platform.openai.com/api-keys) |
| MiniMax | ~$0.01/1k chars | Alta | [api.minimax.chat](https://api.minimax.chat) |
| **Piper (local)** | GrêÂ¡tis | Boa | `pip install piper-tts` |

### Efeitos Sonoros e MêÂºsica

| Provider | Custo | Tipo | Link |
|---|---|---|---|
| **Pixabay** | GrêÂ¡tis | SFX + MêÂºsica | [pixabay.com/api/docs](https://pixabay.com/api/docs/) |
| Freesound | GrêÂ¡tis | SFX (500k+ sons) | [freesound.org/apiv2/apply](https://freesound.org/apiv2/apply) |
| ElevenLabs | Pago | GeraêÂ§êÂ£o de mêÂºsica por IA | [elevenlabs.io](https://elevenlabs.io) |
| Suno AI | ~$0.05/track | GeraêÂ§êÂ£o de mêÂºsica por IA | [suno.ai](https://suno.ai) |

### VêÂ­deos (Remotion)

| ComposiêÂ§êÂ£o | Formato | Uso |
|---|---|---|
| **DynamicAd** | 1080x1920 (9:16) | Reels, Stories, Shorts |
| **DynamicAdSquare** | 1080x1080 (1:1) | Feed Instagram |
| ColdBrewAd | 1080x1080 (1:1) | Template fixo (fallback) |

### SeleêÂ§êÂ£o automêÂ¡tica de provider

O sistema escolhe o provider baseado nas keys configuradas:

```
Imagem ê¢â â Kie.ai ê¢â â DALL-E ê¢â â Stability ê¢â â Pexels/Unsplash/Pixabay (fallback grêÂ¡tis)
Voz    ê¢â â ElevenLabs ê¢â â OpenAI TTS ê¢â â MiniMax ê¢â â Piper local (fallback grêÂ¡tis)
MêÂºsica ê¢â â Pixabay (grêÂ¡tis) ou Suno/ElevenLabs (IA paga)
```

Verificar quais estêÂ£o ativos: `npm run media:status`

---

## Efeitos de CêÂ¢mera nos VêÂ­deos

Movimentos aplicados sobre fotos de fundo simulando cêÂ¢mera cARCKHÉtogrêÂ¡fica:

| Efeito | DescriêÂ§êÂ£o | Uso automêÂ¡tico |
|---|---|---|
| `push-in` | Zoom rêÂ¡pido para o centro | Hooks, impacto |
| `pull-out` | Zoom afastando | RevelaêÂ§êÂµes |
| `ken-burns-in` | Zoom lento e suave | Produto, close-ups |
| `ken-burns-out` | Zoom afastando suave | Flashbacks |
| `pan-left` / `pan-right` | PanorêÂ¢mica horizontal | Narrativa |
| `pan-up` / `pan-down` | PanorêÂ¢mica vertical | EsperanêÂ§a, calma |
| `drift` | Movimento aleatêÂ³rio sutil | Emocional, onêÂ­rico |
| `parallax-zoom` | Zoom com drift vertical | DinêÂ¢mico |
| `tilt-shift` | Zoom com rotaêÂ§êÂ£o | ArtêÂ­stico |
| `breathe` | Pulso de escala sutil | CTAs, finais |

### AnimaêÂ§êÂµes de Texto

| AnimaêÂ§êÂ£o | DescriêÂ§êÂ£o | Uso automêÂ¡tico |
|---|---|---|
| `blur-in` | Surge de desfocado para nêÂ­tido | Hooks |
| `slide-up` / `slide-down` | Desliza vertical | Produto |
| `per-word` | Cada palavra surge separadamente | ConexêÂ£o, benefêÂ­cios |
| `punch-in` | Palavras com spring de impacto | Presentes, reveals |
| `bounce-in` | Palavras com bounce | CTAs |
| `typewriter` | Letra por letra com cursor | Flashbacks, memêÂ³rias |
| `scale-up` | Cresce de pequeno para normal | Destaque |
| `fade` | Fade in simples | Close-ups |

### SeleêÂ§êÂ£o automêÂ¡tica por tipo de cena

| Tipo de Cena | CêÂ¢mera | Overlay | Texto |
|---|---|---|---|
| `hook` | `push-in` | escuro | `blur-in` |
| `produto_em_acao` | `ken-burns-in` | frio | `slide-up` |
| `conexao_emocional` | `drift` | quente | `per-word` |
| `flashback_infancia` | `ken-burns-out` | sêÂ©pia | `typewriter` |
| `flashback_adolescencia` | `pan-left` | sêÂ©pia | `typewriter` |
| `presente` | `push-in` | escuro | `punch-in` |
| `cta` | `breathe` | claro | `bounce-in` |

O sistema resolve nesta ordem: valor explêÂ­cito no JSON ê¢â â detecêÂ§êÂ£o por palavras no roteiro ê¢â â mapeamento pelo tipo ê¢â â default.

---

## Regras de Texto sobre Imagens

Quando fotos reais sêÂ£o usadas como fundo, o sistema segue estas regras:

1. **Analisar a imagem antes** de posicionar texto
2. **Nunca cobrir rostos** ê¢â¬â texto sêÂ³ em êÂ¡reas livres
3. **Texto deve caber no frame** ê¢â¬â padding 40px, overflow hidden
4. **Gradientes localizados** ê¢â¬â sêÂ³ na zona do texto, opacidade 0.4-0.7

| Imagem com... | Onde colocar texto |
|---|---|
| Pessoas no centro | RodapêÂ© ou topo |
| Pessoas na esquerda | Direita |
| Produto no centro | Topo ou rodapêÂ© |
| Texto jêÂ¡ na imagem | NêÂ£o duplicar na mesma êÂ¡rea |

---

## Fluxo de GeraêÂ§êÂ£o de VêÂ­deos

```
1. Agente de VêÂ­deo (Claude) gera scene_plan.json
   ê¢â â
2. Scene plan define: cenas, timing, texto, cores, cêÂ¢mera, imagens
   ê¢â â
3. render-video.js passa o JSON como props para o Remotion
   ê¢â â
4. DynamicAd.tsx monta <Sequence> por cena com:
   - Foto de fundo + efeito de cêÂ¢mera
   - Texto animado
   - Produto com glow (se houver)
   - NarraêÂ§êÂ£o contêÂ­nua + mêÂºsica de fundo
   ê¢â â
5. Output: ad_01.mp4, ad_02.mp4, etc.
```

### êÂudio nos vêÂ­deos

```json
{
  "narration_file": "audio/v1_full.mp3",
  "narration_volume": 1,
  "background_music": "audio/bgm_v1.mp3",
  "background_music_volume": 0.2
}
```

- NarraêÂ§êÂ£o êÂ© um êÂ¡udio êÂºnico e contêÂ­nuo (nêÂ£o cortado por cena)
- DuraêÂ§êÂ£o das cenas acompanha o ritmo da voz
- CTA final fica com hold longo (5-9s) depois da voz terminar
- MêÂºsica de fundo a 20% do volume para nêÂ£o competir

---

## PublicaêÂ§êÂ£o no Instagram

Para publicar via API, êÂ© necessêÂ¡rio:

1. **Conta Business ou Creator** no Instagram vinculada a uma Facebook Page
2. **App no Meta Developer** com o produto "Instagram Graph API"
3. **Credenciais**:
   - `INSTAGRAM_ACCOUNT_ID` ê¢â¬â ID numêÂ©rico da conta (`GET /me?fields=instagram_business_account`)
   - `INSTAGRAM_ACCESS_TOKEN` ê¢â¬â token com permissêÂµes `instagram_business_basic` + `instagram_business_content_publish`
4. Token expira em **60 dias** ê¢â¬â renovar antes
5. Imagens devem estar em **URL pêÂºblica** (o sistema usa Supabase)

---

## Supabase

O projeto usa o Supabase **apenas como Storage** para hospedar os arquivos de mêÂ­dia das campanhas. NêÂ£o hêÂ¡ tabelas no banco de dados ê¢â¬â todos os metadados (captions, research, layouts, URLs) ficam em arquivos locais na pasta `outputs/`.

### O que êÂ© armazenado

O Agente de DistribuiêÂ§êÂ£o faz upload de imagens (PNG) e vêÂ­deos (MP4) para o bucket e gera URLs pêÂºblicas que sêÂ£o usadas para publicaêÂ§êÂ£o via API (Instagram, YouTube).

```
Campanha gerada
    ê¢â â
Upload de PNGs e MP4s ê¢â â Supabase Storage (bucket: campaign-uploads)
    ê¢â â
URLs pêÂºblicas salvas em media_urls.json (local)
    ê¢â â
URLs usadas para publicar via Instagram Graph API / YouTube Data API
```

### ConfiguraêÂ§êÂ£o

1. Criar um bucket chamado `campaign-uploads` marcado como **pêÂºblico**
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

### Publish MD ê¢â¬â O guia de publicaêÂ§êÂ£o

ApêÂ³s o upload no Supabase, o Agente de DistribuiêÂ§êÂ£o gera um arquivo `Publish <task_name> <date>.md` com tudo pronto para publicar. Esse arquivo funciona como um **guia completo da campanha** e contêÂ©m:

| SeêÂ§êÂ£o | O que tem |
|---|---|
| Status | Checklist por plataforma (Instagram, YouTube, Threads) |
| MêÂ­dias hospedadas | Tabela com todos os arquivos, tipo, plataforma e URL pêÂºblica |
| Instagram ê¢â¬â Carrossel | Slides na ordem exata + caption com hashtags |
| Instagram ê¢â¬â Stories | SequêÂªncia de stories com texto e instruêÂ§êÂµes de formato |
| YouTube | TêÂ­tulo, descriêÂ§êÂ£o, tags e URL do vêÂ­deo por vêÂ­deo |
| Threads | Texto pronto para colar (publicaêÂ§êÂ£o manual) |
| Agendamento | CalendêÂ¡rio da semana com datas, horêÂ¡rios e justificativa estratêÂ©gica |
| InstruêÂ§êÂµes de execuêÂ§êÂ£o | Como acionar a publicaêÂ§êÂ£o via API |

### Trava de publicaêÂ§êÂ£o

A publicaêÂ§êÂ£o real nas APIs (Instagram Graph API, YouTube Data API) **sêÂ³ acontece quando o usuêÂ¡rio referencia o Publish MD pelo nome**:

```
Executar Publish dia_das_maes 2026-05-10.md
```

Sem essa referêÂªncia explêÂ­cita, nenhuma chamada de API êÂ© feita. Isso garante que o usuêÂ¡rio sempre revisa o conteêÂºdo antes de publicar.

```
Pipeline completo
    ê¢â â
Publish MD gerado com todo o conteêÂºdo
    ê¢â â
UsuêÂ¡rio revisa captions, hashtags, agendamento
    ê¢â â
UsuêÂ¡rio referencia o arquivo pelo nome ê¢â â publicaêÂ§êÂ£o executa
    ê¢â â
Instagram: Graph API (container ê¢â â publish)
YouTube: Data API (upload com têÂ­tulo/descriêÂ§êÂ£o/tags)
Threads: manual (sem API pêÂºblica)
```

---

## Estrutura do Projeto

```
timesmkt/
ê¢âÅê¢ââ¬ê¢ââ¬ assets/                  # Imagens da marca (fotos de produto)
ê¢âÅê¢ââ¬ê¢ââ¬ knowledge/               # Identidade, guidelines, briefing
ê¢âÅê¢ââ¬ê¢ââ¬ skills/                  # DefiniêÂ§êÂµes dos 5 agentes + media-help
ê¢âÅê¢ââ¬ê¢ââ¬ media/                   # MêÂ³dulo multi-provider (imagem, voz, SFX, mêÂºsica)
ê¢âÅê¢ââ¬ê¢ââ¬ pipeline/                # Orquestrador, worker, render de vêÂ­deo
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ orchestrator.js      # Enfileira jobs no BullMQ
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ worker.js            # Executa agentes via Claude CLI
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ render-video.js      # Renderiza vêÂ­deo com Remotion
ê¢ââ   ê¢ââê¢ââ¬ê¢ââ¬ payloads/            # JSONs de campanha
ê¢âÅê¢ââ¬ê¢ââ¬ remotion-ad/             # Projeto Remotion (vêÂ­deo)
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ src/DynamicAd.tsx    # ComposiêÂ§êÂ£o dinêÂ¢mica
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ src/scenes/          # Cenas por tipo
ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ src/components/      # CameraMotion, TextOverlay, etc.
ê¢ââ   ê¢ââê¢ââ¬ê¢ââ¬ public/              # Assets e êÂ¡udios
ê¢âÅê¢ââ¬ê¢ââ¬ outputs/                 # Campanhas geradas
ê¢âÅê¢ââ¬ê¢ââ¬ .env.example             # Template de variêÂ¡veis
ê¢ââê¢ââ¬ê¢ââ¬ CLAUDE.md                # DocumentaêÂ§êÂ£o têÂ©cnica completa
```

---

## Exemplo de Campanha ê¢â¬â Dia das MêÂ£es

Campanha gerada com fotos, vêÂ­deos com narraêÂ§êÂ£o e mêÂºsica.

### VêÂ­deo 1 ê¢â¬â "Domingo com Ela" (21s)

| Cena | Tempo | Background | CêÂ¢mera | NarraêÂ§êÂ£o |
|---|---|---|---|---|
| Hook | 0-3s | GrêÂ£os de cafêÂ© | `push-in` | *"Domingos têÂªm um cheiro especial."* |
| ConexêÂ£o | 3-6.5s | MêÂ£e e filha | `drift` | *"Ela ensinou tudo. AtêÂ© o cafêÂ© perfeito."* |
| Produto | 6.5-9.5s | Cold brew | `ken-burns-in` | *"Suave. Gelado. Perfeito."* |
| Brinde | 9.5-12s | Cozinha | `parallax-zoom` | *"Um brinde pra ela."* |
| **CTA** | **12-21s** | **AbraêÂ§o** | `breathe` | *"Presente perfeito para quem te deu tudo."* + **hold** |

### VêÂ­deo 2 ê¢â¬â "Anos de CafêÂ©" (19s)

| Cena | Tempo | Background | CêÂ¢mera | NarraêÂ§êÂ£o |
|---|---|---|---|---|
| Hook | 0-2.7s | GrêÂ£os de cafêÂ© | `push-in` | *"Ela sempre teve um cafêÂ© pra te oferecer."* |
| InfêÂ¢ncia | 2.7-5.8s | Cozinha (sêÂ©pia) | `ken-burns-out` | *"Quando vocêÂª tinha cinco anos..."* |
| AdolescêÂªncia | 5.8-8.8s | MêÂ£e e filha (sêÂ©pia) | `pan-left` | *"...e quando vocêÂª tinha dezessete."* |
| Presente | 8.8-12s | Cold brew (flash) | `push-in` | *"Hoje êÂ© a sua vez de cuidar dela."* |
| **CTA** | **12-19s** | **AbraêÂ§o** | `breathe` | *"Cada gole êÂ© um abraêÂ§o."* + **hold** |

---

## Problemas Comuns

| Problema | SoluêÂ§êÂ£o |
|---|---|
| `npm install` falha | Instalar Node.js v18+ |
| Playwright nêÂ£o renderiza | `npx playwright install chromium` |
| Redis nêÂ£o conecta | `docker run -d --name redis -p 6379:6379 redis:alpine` |
| Remotion falha | `cd remotion-ad && npm install` |
| ElevenLabs 401 | Verificar key e permissêÂµes em elevenlabs.io |
| VêÂ­deo sai igual ao template fixo | Passar o scene_plan.json como segundo argumento no render-video.js |
| Texto sobre rosto nas imagens | Seguir regras de posicionamento (analisar imagem antes) |

---

## ComeêÂ§ando do Zero

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

### 5. Renderizar vêÂ­deo avulso

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