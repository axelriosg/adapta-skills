---
name: cola-e-cria
description: Gera uma SKILL.md completa pronta pra colar no Adapta ONE workbench a partir de qualquer fonte externa de conhecimento. Aceita URL de YouTube, Vimeo, Loom, Instagram Reel, Spotify, SoundCloud, podcast direto (mp3/wav/m4a), blog, artigo, Substack, Medium, Notion público, doc Google, página de documentação, PDF online, PDF upload, transcript .vtt/.srt, ou texto colado direto. Detecta automaticamente o tipo de fonte via urlparse e Content-Type. Baixa o conteúdo (yt-dlp pra vídeo/áudio com iPhone user-agent pra Instagram, requests + readability pra HTML, pdfplumber pra PDF). Transcreve áudio com faster-whisper modelo base int8 em pt-BR (8x velocidade real). Destila o conhecimento em SKILL.md executável seguindo os 5 princípios de skill boa do Adapta ONE (escopo cirúrgico, exemplos reais, gatilhos precisos, formato de saída definido, validação antes de entregar). Dispara quando o usuário envia URL pedindo "transforma em skill", "vira skill", "cria skill desse vídeo", "skill desse artigo", "automatiza esse conhecimento", "destila essa aula", "cola e cria", "skill desse podcast", "skill dessa documentação", "transforma esse PDF em skill", ou cola texto longo pedindo empacotamento como skill.
---

# Cola e Cria

Pega qualquer fonte de conhecimento externa e devolve um SKILL.md completo, validado, pronto pra colar no Adapta ONE workbench. Não faz perguntas iterativas no fluxo padrão: link entra, skill sai. Faz perguntas só quando a entrada é genuinamente ambígua.

## Princípios (importados do skill-creator do Adapta ONE)

Toda skill gerada respeita os 5 critérios:

1. **Escopo cirúrgico**: tarefa específica, não área genérica.
2. **Exemplos embutidos**: extrai casos concretos da fonte, inclui exemplo bom e ruim quando possível.
3. **Gatilhos precisos**: 30+ frases-gatilho na description.
4. **Formato de saída definido**: especifica exatamente como o output aparece.
5. **Validação antes de entregar**: checagem automática nos 4 critérios.

## Quando ativar

Use sempre que o usuário enviar URL ou texto longo com intenção de transformar em skill. Em dúvida, ative.

## Reconhecimento de entrada

A skill reconhece 6 padrões de entrada e adapta o fluxo:

| Padrão | Resposta |
|---|---|
| URL sozinha sem instrução | Detecta intenção pela URL e roda fluxo completo |
| URL + "vira skill" / "cria skill" | Roda fluxo completo direto |
| URL + contexto adicional | Roda fluxo considerando o contexto como filtro |
| Texto colado (>500 chars) | Pula Etapa 2, vai direto pra análise |
| SKILL.md existente colado + "melhora isso" | Modo melhoria: preserva `name` original, identifica pontos fracos, regenera |
| Playlist ou múltiplas URLs | Pergunta antes: "Gero uma skill consolidada ou uma por fonte?" |

## Workflow

### Etapa 1: Detecção robusta de fonte

Use `urlparse` em vez de string matching frágil. Inclua suporte a upload local e detecção via Content-Type quando ambíguo.

```python
from urllib.parse import urlparse
import requests

def detect_source(input_str: str) -> str:
    s = input_str.strip()
    
    # Upload local (s3_url retornado por upload_local_file ou path /uploads/)
    if 's3.amazonaws.com' in s or '/uploads/' in s or s.startswith('s3://'):
        # Inspeciona extensão antes do query string
        path = urlparse(s).path.lower()
        if path.endswith('.pdf'): return 'pdf_upload'
        if path.endswith(('.vtt', '.srt')): return 'transcript_upload'
        if path.endswith(('.mp3', '.wav', '.m4a', '.ogg')): return 'audio_upload'
        if path.endswith('.txt'): return 'text_upload'
        return 'unknown_upload'
    
    # URLs externas
    if s.startswith('http'):
        url = s.lower()
        parsed = urlparse(s)
        host = parsed.netloc.lower()
        path = parsed.path.lower()
        
        if 'youtube.com' in host or 'youtu.be' in host: return 'youtube'
        if 'vimeo.com' in host: return 'vimeo'
        if 'loom.com' in host: return 'loom'
        if 'instagram.com' in host and '/reel' in path: return 'instagram_reel'
        if 'instagram.com' in host and '/p/' in path: return 'instagram_post'
        if 'tiktok.com' in host: return 'tiktok'
        if 'spotify.com' in host: return 'spotify'
        if 'soundcloud.com' in host: return 'soundcloud'
        if path.endswith('.pdf'): return 'pdf_url'
        if path.endswith(('.mp3', '.wav', '.m4a')): return 'audio_url'
        
        # Ambíguo: faz HEAD request pra inspecionar Content-Type
        try:
            head = requests.head(s, timeout=10, allow_redirects=True,
                                 headers={'User-Agent': 'Mozilla/5.0'})
            ct = head.headers.get('Content-Type', '').lower()
            if 'pdf' in ct: return 'pdf_url'
            if 'audio' in ct: return 'audio_url'
            if 'video' in ct: return 'video_url'
        except: pass
        return 'web'
    
    # Conteúdo colado direto
    if len(s) > 500: return 'pasted_text'
    return 'unknown'
```

Se retornar `unknown` ou `unknown_upload`, faça uma pergunta antes de continuar.

### Etapa 2: Aquisição com gestão de erro

Instalar dependências consolidadas de uma vez no início (cache permite skip se já instalado):

```bash
pip install -q yt-dlp faster-whisper readability-lxml beautifulsoup4 pdfplumber requests
```

Pra cada tipo, use a estratégia certa com `try/except` documentado:

**YouTube, Vimeo, Loom, TikTok, Spotify, SoundCloud, Instagram**: yt-dlp com fallback de erro.

```python
import subprocess
def baixar_audio(url, instagram=False):
    cmd = ['yt-dlp', '-x', '--audio-format', 'mp3',
           '-o', '/tmp/audio.%(ext)s', url]
    if instagram:
        cmd.extend(['--user-agent', 
                    'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15'])
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
        if result.returncode != 0:
            return None, f"yt-dlp falhou: {result.stderr[:200]}"
        return '/tmp/audio.mp3', None
    except subprocess.TimeoutExpired:
        return None, "Download passou de 5 minutos. Vídeo muito longo ou conexão lenta."
```

**Cenários de erro com mensagem específica**: HTTP 403 ou geo/age block ("vídeo restrito, cola transcrição"); Private/privado ("vídeo privado, sem acesso"); Instagram auth ("reel privado, cola legenda"); Spotify falha ("tenta link do host original do podcast").

**Transcrição** (faster-whisper `base int8`, 8x real-time pt-BR):

```python
from faster_whisper import WhisperModel
import os
os.environ['HF_HOME'] = '/tmp/hf_cache'  # cache persistente
model = WhisperModel("base", device="cpu", compute_type="int8")
segments, _ = model.transcribe('/tmp/audio.mp3', language="pt", beam_size=5)
text = " ".join(s.text for s in segments)
# Se len(text) < 100: avisar transcrição vazia ou ruído
```

**Blog/HTML**: requests + readability + BeautifulSoup. Tratar 403 ("site bloqueia scraping, cola texto"), 404 ("link inexistente"), conteúdo < 200 chars ("pode ser SPA, cola texto"), timeout.

**PDF (URL ou upload)**: pdfplumber. Tratar PDF escaneado sem texto ("precisa OCR") e PDF com senha ("remova senha antes").

**Transcript .vtt/.srt**: parsing com filtro robusto de cues alfanuméricos (Adapta ONE constraint conhecida).

```python
import re
def limpar_vtt(raw):
    lines = []
    for l in raw.split('\n'):
        l = l.strip()
        if not l or l.startswith('WEBVTT'): continue
        if re.match(r'^\d{2}:\d{2}', l): continue
        if 'align:' in l: continue
        if re.match(r'^\d+$', l): continue
        # Filter cues sem alfanumérico
        if not re.search(r'[a-záàâãéêíóôõúçA-Z]', l): continue
        lines.append(l)
    return ' '.join(lines)
```

### Etapa 3: Análise com critério binário

Antes de gerar a skill, identifique no conteúdo extraído:

1. **Tópico central**: do que trata? Em uma frase de no máximo 15 palavras.
2. **Método replicável**: liste 3 ações concretas que alguém poderia executar amanhã com base na fonte. **Se não conseguir listar 3 ações**, declare que a fonte não tem método e pare. Skill sem processo executável é skill morta.
3. **Frameworks nomeados**: tem siglas, acrônimos, ou nomes próprios de método? (ex: SCANT, OPERA, 4Is, TAPE)
4. **Exemplos concretos**: a fonte mostrou 1+ caso aplicado? Capture o exemplo verbatim ou destilado.
5. **Pré-requisitos**: o método exige conhecimento prévio ou ferramenta específica?
6. **Língua da fonte**: se for inglês ou espanhol, marque pra adaptação na destilação.

Exemplo de aplicação binária:

- Fonte: vídeo de 18 min sobre cold email com 5 etapas (research, hook, body, CTA, follow-up). **Passa**: tem 3+ ações concretas.
- Fonte: TED talk sobre "ser autêntico". **Não passa**: filosofia sem ações executáveis. Pare e avise.

### Etapa 4: Destilação em SKILL.md

Gere o arquivo seguindo as regras abaixo. Mire em **10K-13K caracteres**, nunca passar de 15K (cap operacional do Adapta ONE workbench).

**Regra de slug**: lowercase, hífens, máximo 4 palavras significativas, remove stopwords PT-BR (a, o, de, da, do, em, para, com, e, ou). Exemplos válidos: `cold-email-5-camadas`, `headline-direto`, `roteiro-lancamento`. Inválidos: `como-fazer-um-cold-email`, `skill-de-cold-email`.

**Regra de description** (a parte mais crítica, é o que dispara ativação):

| Description ruim | Description boa |
|---|---|
| "Ajuda com cold email." | "Reescreve cold emails seguindo o framework de 5 camadas (research, hook, body, CTA, follow-up). Dispara quando o usuário cola um briefing de prospecção, pede 'cold email pra X', menciona 'email frio', 'outreach', 'prospect', 'reach out', cola um email mediano pedindo melhoria, ou pergunta como abordar lead específico. Inclui exemplos de hook bom e ruim baseados em SaaS founders." |

A boa tem 30+ palavras-chave de gatilho, descreve quando ativar com bullets implícitos, e ancora num exemplo. A ruim tem 4 palavras genéricas que não disparam nada.

**Estrutura obrigatória do SKILL.md gerado**:

```
---
name: <slug>
description: <2-4 frases ricas em gatilhos>
---

# <Título>

<1 parágrafo sobre o que faz>

## Quando ativar
<gatilhos linguísticos PT-BR>

## Inputs aceitos
<o que o usuário precisa fornecer>

## Workflow
### Etapa 1: ...
### Etapa N: ...

## Output esperado
<formato e estrutura>

## Exemplo
<caso real da fonte, ou aviso de que fonte não forneceu>

## Limitações
<o que está fora de escopo>
```

**Tradução para fontes em outras línguas**: se a fonte foi inglês ou espanhol, traduza tópico, exemplos e gatilhos pra PT-BR. Mantenha siglas e nomes de framework no original (ex: "TAPE framework" vira "framework TAPE", não "fita adesiva"). Adicione uma linha em "Limitações": "Fonte original em [língua]; gatilhos PT-BR podem precisar ajuste."

### Etapa 5: Validação automática com auto-correção

Antes de entregar, rode os 4 checks. Se falhar, ajuste em até 3 tentativas. Se ainda falhar, entregue com aviso explícito.

| Check | Critério | Se falhar |
|---|---|---|
| Tamanho | 5K < `len(skill_md)` < 15K | < 5K: expandir exemplos. > 15K: cortar seção menos crítica. |
| Description | 30+ palavras-chave | Adicionar variações PT-BR coloquial e formal |
| Workflow | 3+ etapas executáveis | Voltar Etapa 3 e refinar |
| Exemplo | 1+ concreto OU declaração | Declarar: "Fonte não forneceu; testar com caso próprio." |

### Etapa 6: Pacote completo de entrega

Devolva ao usuário tudo junto, sem fragmentar:

1. **Resumo em 2 frases**: o que a skill faz e quando ativar.
2. **Nome sugerido** (slug): pronto pro campo `name` no Adapta ONE.
3. **Description curta** (até 80 caracteres): pra UI do Adapta ONE que trunca preview.
4. **SKILL.md completo**: em bloco de código markdown pra copiar.
5. **Instruções de instalação**: "Vá em Configurações > Skills > Nova Skill > cole o conteúdo > salva > ativa."
6. **`s3_url`** (se gerou arquivo): retorne pra usuário poder baixar.
7. **Aviso de validação** (se algum check falhou): explicite o que não passou.

## Constraints técnicas do Adapta ONE

SKILL.md cap operacional 15K chars (hard ~18-19K). Description preview UI ~80 chars. yt-dlp Instagram exige iPhone user-agent. faster-whisper `base int8` em pt-BR a 8x velocidade real, cache em `/tmp/hf_cache` reaproveitado entre execuções. `upload_local_file('/tmp/skill.md')` retorna `{s3_url}` pra entrega. WebM nativo Slack/LinkedIn/Discord; MP4 via cloudconvert pra Instagram/WhatsApp. Markdown gerado vai inline no chat, não em bloco que precisa "Copiar com formatação".

## Casos de borda tratados

Código técnico: preserve verbatim em blocos de código. Playlist ou múltiplas URLs: pergunta antes se gera 1 ou N skills. SKILL.md colado + intenção de melhoria: modo update, preserva `name` original. Multimodal (vídeo com slides): tenta frame sampling pra slides, se falhar declara perda de sinal visual. Deduplicação: avisa antes se tópico já tem skill no workspace. Fonte de baixa qualidade: corte na Etapa 3 evita gerar skill vazia.

## O que evitar

Não invente método se a fonte não tem 3+ ações. Não copie verbatim no corpo da skill. Description curta = under-trigger. Não duplique skill existente sem checar. Não envie sem rodar a Etapa 5. Use whisper `base int8`, não `small` ou `medium`. Inclua iPhone user-agent pro Instagram.

## Exemplo aplicado completo

**Input**: `https://www.youtube.com/watch?v=ABCD1234` (vídeo 18 min sobre cold email).

**Pipeline**: detecção `youtube` → yt-dlp baixa mp3 (45s) → faster-whisper pt-BR (~2.800 palavras, 90s) → análise extrai 3+ ações + framework "5-Layer" + 2 exemplos verbatim → destila em 11.400 chars → validação passa nos 4 checks.

**Output ao usuário**:

> A skill **cold-email-5-camadas** reescreve cold emails seguindo um framework de research, hook, body, CTA e follow-up. Ativa quando você cola um briefing de prospecção ou pede ajuda com email frio.
>
> **Nome**: `cold-email-5-camadas` | **Description curta**: "Cold emails em 5 camadas: research, hook, body, CTA, follow-up"
>
> ```markdown
> [SKILL.md completo de 11.400 chars]
> ```
>
> Pra usar: Configurações > Skills > Nova Skill > cole > salva > ativa.

## O que evitar

- Não invente método. Se a fonte não tem 3+ ações concretas, pare.
- Não copie a fonte verbatim no corpo da skill. Destile em instruções próprias.
- Não gere description curta. Menos de 100 palavras = under-trigger.
- Não duplique skill existente sem checar primeiro.
- Não envie sem rodar a Etapa 5 de validação.
- Não use modelo `small` ou `medium` do whisper. Use `base int8` (mais rápido, qualidade suficiente).
- Não esqueça do iPhone user-agent pro Instagram.
