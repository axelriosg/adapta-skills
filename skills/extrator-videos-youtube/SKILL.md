#Extrator de Vídeos YouTube
Extrai vídeos, transcreve, resume e gera conteúdo derivado
## Skill: Extração + Transcrição + Conteúdo Derivado de Vídeos do YouTube

### Objetivo

Extrair os 3 últimos vídeos de QUALQUER canal do YouTube, apresentar em tabela, e automaticamente:

1. Transcrever e resumir o vídeo mais recente (Início/Meio/Fim)

2. Destacar 3-5 Key Quotes marcantes

3. Gerar bullet points prontos para newsletter

4. TUDO sempre em português, mesmo que o vídeo seja em inglês

---

### PASSO 1: Identificar o Canal

- Aceitar qualquer formato: @NomeDoCanal, URL completa, ou nome solto (ex: "MrBeast", "Lex Fridman")

- Construir URL: https://www.youtube.com/@NomeSemEspacos/videos

- Quantidade padrão: 3 vídeos. Ajustar .slice(0, N) se pedido.

---

### PASSO 2: Criar Task no Browser

Usar COMPOSIO_SEARCH_TOOLS para buscar browser tools, depois BROWSER_TOOL_CREATE_TASK:

```

task: "Navigate to [URL]/videos. Wait for video elements to load. Execute JavaScript: Array.from(document.querySelectorAll('a#video-title-link')).slice(0, [N]).map(a => ({title: a.innerText.trim(), url: a.href})). Return the full JSON array."

startUrl: "[URL]/videos"

```

---

### PASSO 3: Monitorar Task

Usar BROWSER_TOOL_WATCH_TASK com o taskId.

- started → polling (até 3x)

- finished + isSuccess: true → extrair do output

- finished + isSuccess: false → reportar erro

---

### PASSO 4: Apresentar Vídeos em Tabela

tableFormatter com colunas: #, Título, URL.

Título: "🎬 Últimos [N] vídeos — [Nome do Canal]"

---

### PASSO 5: Transcrever o Vídeo Mais Recente

IMEDIATAMENTE após a tabela, sem esperar o usuário:

1. Extrair videoId da URL do 1º vídeo (valor após v=, ignorar &pp= etc.)

2. Chamar youtubeTranscript com esse videoId

3. Gerar os 3 blocos abaixo com base na transcrição

---

### PASSO 6: Bloco 1 — Resumo Início / Meio / Fim

```

## 🎬 Resumo: "[Título do Vídeo]"

### 🟢 INÍCIO — [Subtítulo contextual]

[2-3 parágrafos curtos ~150 palavras]

### 🟡 MEIO — [Subtítulo contextual]

[2-3 parágrafos curtos ~150 palavras]

### 🔴 FIM — [Subtítulo contextual]

[2-3 parágrafos curtos ~150 palavras]

```

Regras: ~450 palavras total. Tom narrativo mas direto. Nomes em destaque. Momentos emocionais em **negrito**. Fechar com 1 frase de contexto geral.

---

### PASSO 7: Bloco 2 — Key Quotes

Extrair da transcrição as **3-5 frases mais impactantes** — aquelas que funcionariam como:

- Cortes virais para Reels/TikTok

- Destaques para thumbnails

- Citações para posts em redes sociais

Formato:

```

## 💬 Key Quotes

> "Frase marcante aqui."

> — [Quem disse], [contexto breve]

> "Outra frase impactante."

> — [Quem disse], [contexto breve]

```

Critérios de seleção:

- Carga emocional forte (motivação, vulnerabilidade, humor)

- Frases curtas e autossuficientes (fazem sentido fora de contexto)

- Potencial viral ou de engajamento

- Se o vídeo for em inglês, traduzir as quotes para português

---

### PASSO 8: Bloco 3 — Bullet Points para Newsletter

Gerar uma seção pronta para copiar e colar numa newsletter, com:

- **Título sugerido** para o bloco da newsletter (chamativo, curto)

- **1 frase de contexto** sobre o vídeo (gancho)

- **5-7 bullet points** com os principais insights/takeaways do vídeo

- **1 CTA final** direcionando para o vídeo original

Formato:

```

## 📋 Newsletter Ready

**[Título sugerido para newsletter]**

[Frase de contexto/gancho]

- [Insight/takeaway 1]

- [Insight/takeaway 2]

- [Insight/takeaway 3]

- [Insight/takeaway 4]

- [Insight/takeaway 5]

👉 [CTA com link para o vídeo]

```

Regras: Linguagem direta e profissional. Cada bullet é autoexplicativo. Evitar jargão. Tudo em português.

---

### PASSO 9 (Opcional): Follow-up

Oferecer:

- Transcrever outro vídeo da lista

- Extrair vídeos de outro canal

- Aprofundar alguma parte do resumo

- Gerar mais formatos de conteúdo derivado

---

### IDIOMA — REGRA ABSOLUTA

TODO output (resumo, quotes, newsletter) DEVE ser em **português brasileiro**, independente do idioma original do vídeo. Se a transcrição vier em inglês ou outro idioma, traduzir todo o conteúdo.

---

### Tratamento de Erros

1. **Seletor falha**: Tentar ytd-rich-item-renderer a#video-title-link

2. **Transcrição indisponível**: Informar e oferecer resumir outro da lista

3. **Task trava**: BROWSER_TOOL_STOP_TASK e recriar

4. **Canal inexistente**: Informar o usuário

---

### Notas

- Browser real (headless) para DOM dinâmico do YouTube

- NUNCA inventar dados, vídeos ou conteúdo

- Session_id do COMPOSIO_SEARCH_TOOLS em todas as chamadas

- Resumo CONCISO — não reproduzir transcrição
