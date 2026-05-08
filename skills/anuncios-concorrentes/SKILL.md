#Anúncios Concorrentes
Mapeia ads ativos de uma marca na Facebook Ads Library
# Anúncios Concorrentes

## O que esta Skill faz

Mapeia ads ativos de uma marca na Facebook Ads Library, classifica nível de escala, identifica top 3 anúncios mais escalados com transcript completo, e gera três variações do top 1 em voz Adapta. Delega a coleta bruta ao Web Scrape nativo do Adapta ONE e foca em estrutura, score e ativação.

## Quando ativar

Use sempre que mencionar anúncios de concorrente, paid social, biblioteca de anúncios, ad library, "está em escala", espionar paid, mesmo sem palavras exatas. Em dúvida, ative.

- "encontra os ads da [marca]"

- "[marca] está em escala?"

- "espiona o paid da Rocketseat"

- "deep research nos ads da [concorrente]"

- "ads do concorrente"

- URL contendo facebook.com/ads/library

- URL de site de uma marca pedindo análise de paid

## Como o usuário usa

Modo Mapa (rápido, default): cola o nome da marca ou URL do site. A skill faz o scraping nativo via Web Scrape do ONE e devolve o relatório com clusters, score e top 3 em alguns segundos. Sem transcript de áudio dos vídeos.

Modo Deep (completo): começa a mensagem com Executa no workbench: e a skill roda o pipeline completo, baixando os três vídeos top com yt-dlp, transcrevendo em pt-BR via faster-whisper, e gerando as três variações no Style DNA da Adapta. Demora alguns minutos.

## Max Peters Style DNA (contexto fixo)

Voz que a Adapta usa em vídeos, ads e aulas, extraída de análise forense do curso Masterizando IA Generativa. Toda variação gerada na Fase 3 segue esse padrão. Não é negociável.

### Regras mecânicas

Frase média de 18 palavras, com 41% das frases abaixo de 8 palavras (alterna ritmo curto e longo, nunca staccato puro). "A gente" no lugar de "nós". Sem travessão. Linguagem oral, não escrita formal. Números específicos no lugar de adjetivos genéricos ("mais de cinco mil reviews" e não "muita gente"). Analogia concreta sempre que introduzir conceito ("é tipo receita de bolo, você segue a ordem"). Fecha aulas com "eu te espero já na próxima aula".

### Estrutura de 7 partes

1. **Promessa ousada**, hook de uma frase com benefício específico mais bloqueio do leitor

2. **História pessoal**, primeira pessoa, vulnerabilidade controlada ("eu também errava nisso")

3. **Framework nomeado**, sigla ou nome próprio fácil de lembrar (TAPE, 4Is, "computador escondido")

4. **Analogia real**, paralelo do mundo físico ou cotidiano

5. **Demo ao vivo**, mostra na tela o que tá falando, nunca só descreve

6. **Recap**, três bullets que resumem o que aprendeu

7. **Bridge**, gancho pra próxima aula ou próximo passo

### Exemplo bom (hook de aula)

"A maioria das pessoas usa IA do jeito errado, e é por isso que recebe resposta genérica. A gente vai consertar isso hoje. Eu também usava errado, perdia tempo, ficava frustrado, achava que a IA era hype. Até descobrir o que chamo de TAPE, que é Tarefa, Audiência, Padrão, Exemplo. Funciona como receita de bolo, você segue a ordem e a IA entrega exatamente o que você quer. Vou mostrar agora, no chat real, como isso muda tudo."

### Exemplo bom (hook de ad)

"Você tá perdendo dinheiro porque ainda não sabe usar IA direito. A gente analisou cinco mil reviews dos nossos alunos e descobriu que 90% das pessoas usam IA como se fosse Google, e por isso recebe resposta de Google. Quem trata IA como assistente, recebe resultado de assistente. A diferença cabe num post-it. Vou te mostrar."

### Exemplo ruim (não gerar assim)

"A inteligência artificial revolucionou a forma como trabalhamos. Nesta aula, abordaremos as principais técnicas para otimizar seus prompts e maximizar resultados. Acompanhe os tópicos a seguir."

Por que é ruim: frases longas demais, "nós" implícito, sem história pessoal, sem framework nomeado, sem analogia, sem demo prometida, linguagem corporativa morta.

## Instruções de comportamento

### Fase 1, coleta delegada ao Web Scrape do ONE

Scrape em três alvos, em ordem:

1. URL de busca por palavra-chave: https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=BR&q=[marca]&search_type=keyword_unordered. Pega contagem total e primeiros ads.

2. Para cada page_id de veiculador que aparecer (institucional, afiliados, contas relacionadas), nova URL: https://www.facebook.com/ads/library/?view_all_page_id=[id]&country=BR&active_status=active. Pega o catálogo completo de cada veiculador.

3. Cruza com agregadores externos adlibrary.com, leadenforce.com) quando o scrape direto perder dados ou pra triangular.

Extrai por anúncio: library_id (16 dígitos), start_date, page_name do veiculador, copy completo do post, headline, CTA, formato (vídeo, imagem, carrossel), URL do vídeo ou imagem, link de destino com slug e UTM, plataformas ativas.

Marcas brasileiras de info-produto frequentemente rodam tráfego via afiliados e não pela página institucional. Sempre busca por nome da marca mais variantes (ex: "adapta", "adapta.org", "adapta one") e investiga as páginas de veiculadores que aparecerem nos resultados.

### Fase 2, análise estrutural

Classifica escala da marca pela tabela:

| Ads ativos | Status |

|---|---|

| 0 | Não anuncia |

| 1 a 10 | Testando |

| 11 a 50 | Escala inicial |

| 51 a 200 | Em escala |

| 201 a 500 | Escala forte |

| 500+ | Líder de categoria |

Para cursos e info-produto BR, ajusta: 50+ é validação, 500+ é líder. Para SaaS B2B, divide por 2,5. Para e-commerce, multiplica por 2.

Agrupa anúncios em clusters de campanha pelo slug de destino. Por exemplo, clube-das-ias-ab-2026, pacote-atualizacoes-meta e lead-cariani são três clusters distintos. Cada cluster vira uma seção do relatório com angle, copy padrão, gatilho psicológico identificado, e variações de criativo rodando.

Identifica os três anúncios mais escalados via score combinando idade rodando (mais antigo é vencedor que sobreviveu à fadiga, peso maior), número de plataformas ativas, e formato (vídeo pesa mais que imagem). Score numérico explícito para tornar o ranking auditável:

```

score = min(dias_rodando / 30, 6) * 10 + plataformas * 5 + (vídeo ? 10 : 5)

```

Mapeia estrutura de funil. Procura padrão de subdomínio mais slug por afiliado (ex: go.adapta.org/campaigns/[slug]/[afiliado]) que indica tracking granular. Identifica se o funil leva pra checkout direto, webinar, lead magnet, app, evento, ou retargeting.

Para cada top 3, mapeia cinco campos: angle (a promessa central), hook (primeiros segundos), prova (números, depoimento, autoridade, antes-depois), oferta (preço, desconto, garantia, bônus, urgência), destino (categoria de campanha mais URL).

### Fase 3, transcript e variações (modo Deep, workbench)

Quando o usuário ativa com Executa no workbench:, roda Python para baixar e transcrever os top 3:

```python

import yt_dlp

from faster_whisper import WhisperModel

def transcrever(video_url, idx):

    out = f'/home/user/ad_{idx}.mp3'

    with yt_dlp.YoutubeDL({

        "format": "bestaudio/best",

        "outtmpl": out.replace('.mp3', '.%(ext)s'),

        "postprocessors": [{"key": "FFmpegExtractAudio", "preferredcodec": "mp3"}],

        "quiet": True,

    }) as ydl:

        ydl.download([video_url])

    model = WhisperModel("medium", device="cpu", compute_type="int8")

    segments, _ = model.transcribe(out, language="pt", vad_filter=True)

    return "\n".join(s.text.strip() for s in segments if s.text.strip())

for i, url in enumerate(videos_top3, 1):

    print(f"--- TOP {i} ---\n{transcrever(url, i)}\n")

```

Se yt-dlp não conseguir baixar (vídeo restrito a plataforma), tenta o link direto mp4 que aparece no scraping. Falha visível e nunca silenciosa: se um dos três falhar, segue com os dois restantes e informa qual perdeu.

Com transcripts em mãos, gera três variações do top 1 aplicando o Max Peters Style DNA descrito no contexto fixo acima (regras mecânicas, estrutura de 7 partes, exemplos bons como referência, exemplo ruim como contraste).

Variação A, novo hook na mesma estrutura. Mantém angle, prova e oferta. Troca os três segundos iniciais. Se o original abre com problema, abre com prova. Se abre com prova, abre com polêmica.

Variação B, formato estático. Pega a promessa central e a prova mais forte do vídeo, comprime em uma imagem com headline mais sub mais CTA. Design Adapta com verde #28C98A sobre fundo #0A0A0A, fonte Instrument Sans, logo ADAPTΔ verde.

Variação C, audiência adjacente. Mantém estrutura e oferta, traduz linguagem para outra vertical do ICP Adapta. Se o original fala com criador de conteúdo, a variação fala com gestor PME. Se fala com profissional liberal, a variação fala com vendedor.

## O que evitar

Não inventar contagens quando o scrape falhar. Diga "não consegui dados de [marca], possíveis razões: marca não anuncia, anuncia só via afiliados que precisam ser identificados antes, ou bloqueio temporário do scrape, tenta de novo em alguns minutos". Falha visível.

Não copiar copy verbatim do concorrente nas variações. Sempre reescrever no Style DNA Adapta. Variação A reaproveita só a estrutura, nunca as palavras exatas.

Não pular a Fase 2 mesmo quando os dados forem ricos. Score e clusters são centro do output, sem eles a entrega vira dump.

Não usar travessão em nenhum output. Não escrever em staccato (frases curtas em sequência sem conexão), sempre prosa natural com vírgulas e conjunções.

Não gerar variação sem ter transcript no modo Deep. Sem transcript do top 1, a variação vira chute. Pede transcript primeiro, gera depois.

Não confundir Style DNA com tom genérico de info-produto. As variações precisam passar pelos sete itens da estrutura ou não estão prontas.

## Exemplos de referência

**Bom**: usuário cola "deep research nos ads da Adapta no facebook". Web Scrape identifica que tráfego principal vai por afiliados (Peter Jordan, Daniel Zukerman, Adapta Experience), mapeia três clusters principais (9 IAs Premium, Pacote Atualizações 2 Anos, Lead Cariani), calcula score por afiliado, identifica funil go.adapta.org/campaigns/[slug]/[afiliado], seleciona top 3 pelo combinador idade-plataformas-formato. Output completo com mapa de funil inferido e implicações para o Education team.

**Bom**: usuário cola "encontra ads da Rocketseat" com Executa no workbench:. Scrape acha 312 ads ativos em 6 veiculadores. Análise classifica Escala Forte para vertical cursos, identifica três clusters ("imersão grátis", "conta sem code", "aula 0 programador"). Top 3 são todos vídeos com 90 dias rodando em 4 plataformas. Workbench baixa os três, transcreve em pt-BR, e devolve variações Adapta com hook reescrito mantendo o angle vencedor, todas passando pelos 7 itens do Style DNA.

**A evitar**: usuário pede análise da Hashtag Treinamentos, output devolve só "134 ads ativos, em escala". Falhou em mapear clusters, selecionar top 3 e gerar variações. O relatório completo é obrigatório, profundidade adapta ao volume de dados mas estrutura nunca encurta.

## Formato de saída

```

# [Marca] | Facebook Ads | [data]

## Resumo executivo

Total ativos: X | Status: [classificação] | Vencedores 60d+: Y

Veiculadores: [marca + afiliados identificados]

Adapta hoje: [contexto comparativo se relevante]

## Estrutura de campanha

A marca distribui tráfego via [N] páginas: [lista].

[Inferência: institucional, network de afiliados, mix, etc]

## Clusters de campanha

### Cluster A, [nome]

Slug: [slug]

Copy padrão: [resumo curto]

Angle: [análise psicológica do gatilho]

Veiculadores rodando: [quem]

Variações de criativo: [N]

### Clusters B, C, etc, mesmo formato

## Top 3 escalados

### #1, [formato, dias rodando, N plataformas, score X]

Destino: [URL ou categoria]

Estrutura: angle=[...], hook=[...], prova=[...], oferta=[...], destino=[...]

Copy do post: [verbatim]

Transcript completo (modo Deep): [texto via whisper]

### #2 e #3 no mesmo formato

## Mapa de funil inferido

ad → [LP slug] → [conversão] → [upsell]

## O que está funcionando, resumo executivo

| Padrão | Evidência | Por que funciona |

|---|---|---|

| ... | ... | ... |

## Variações geradas pro Top 1 (modo Deep)

### A, novo hook na mesma estrutura

[roteiro completo, voz Max Peters Style DNA, todos 7 itens]

### B, formato estático

Headline: [...]

Sub: [...]

Body: [...]

CTA: [...]

Cena visual: [...]

### C, audiência adjacente

[roteiro completo, novo público, mesmo Style DNA]

## Próximo passo sugerido

[teste concreto, definido em uma frase]

## Implicações para Adapta

[2 ou 3 insights estratégicos diretos do que o concorrente faz, na voz do Education team]

```

Título: Anúncios Concorrentes

Descrição: Analisa ads concorrente e gera variações. Ative: ads, paid, ad library, escala
