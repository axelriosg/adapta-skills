# Adapta Skills

Skills oficiais para o [Adapta ONE](https://adapta.org) workbench, em português.

## O que é

Coleção de 19 skills produzidas e usadas pela Adapta.org no dia a dia, prontas para colar no Adapta ONE workbench. Cada pasta tem um `SKILL.md` que você copia e cola em `Configurações > Skills > Nova Skill` no Adapta ONE.

## Skills

### Meta (skills que criam outras skills)

- **[skill-creator](skills/skill-creator)** — Q&A guiado pra criar skills do zero, com 11 hacker patterns e 5 níveis de complexidade.
- **[cola-e-cria](skills/cola-e-cria)** — Cola URL ou texto, ganha SKILL.md pronto. Aceita YouTube, Instagram, blog, PDF, transcript.
- **[style-dna-extractor](skills/style-dna-extractor)** — Análise forense quantitativa de voz em PT-BR. Recebe transcript, devolve Style DNA com 30+ métricas.

### Slides

- **[adapta-slides](skills/adapta-slides)** — Gera PPTX editável + PDF preview. Companion: adapta-slides-preview.
- **[adapta-slides-preview](skills/adapta-slides-preview)** *(auxiliar)* — Gera HTML preview hospedado no S3. Usado pela adapta-slides.

### PDF

- **[pdf-desafio](skills/pdf-desafio)** — PDFs longos (20-60 páginas) estilo "Desafio 5 Dias" com terminal cards e checklists.
- **[gerador-de-guia-de-curso-adapta-pdf](skills/gerador-de-guia-de-curso-adapta-pdf)** — PDFs curtos (3-12 páginas) estilo guia de curso, capa dark + páginas cream.

### Visual estático

- **[carrossel-instagram](skills/carrossel-instagram)** — Carrosséis 4:5 (1080x1350) pixel-perfect, exporta ZIP de 8 PNGs prontos pro feed.

### Vídeo

- **[diagrama-em-video](skills/diagrama-em-video)** — Vídeos 1920x1080 line art animado com narração TTS pt-BR e música acústica de fundo. 11 layouts.
- **[vintage-video](skills/vintage-video)** — Vídeos estilo documentário antigo com WebGL2 shader (grain, scratches, light leak, vignette).
- **[crt-terminal-video](skills/crt-terminal-video)** — Vídeos animados de terminal com efeito CRT, replicando estilo Anthropic Claude Code.
- **[video-3d-cinematico](skills/video-3d-cinematico)** — Vídeos 3D cinematográficos com three.js, PBR materials e post-processing. Auxiliar: cinematografia-base.
- **[cinematografia-base](skills/cinematografia-base)** *(auxiliar)* — Infraestrutura three.js compartilhada pras skills de vídeo 3D.
- **[adapta-aula](skills/adapta-aula)** — Micro-aulas 9:16 com voz humana edge-tts, captions estilo MrBeast e ícones Lucide.

### Ingestão e análise

- **[reel-em-texto](skills/reel-em-texto)** — Cola link de reel do Instagram, recebe transcript polido em pt-BR.
- **[extrator-videos-youtube](skills/extrator-videos-youtube)** — Extrai 3 últimos vídeos de qualquer canal YouTube, transcreve e gera resumo + quotes + newsletter.
- **[anuncios-concorrentes](skills/anuncios-concorrentes)** — Mapeia ads ativos de marca na Facebook Ads Library, gera variações no Style DNA da Adapta.

### Design

- **[design-clone-files](skills/design-clone-files)** — Extrai design system completo de imagens, PDFs, slides ou briefs uploadados.

### Interativo

- **[stem-interactive-deployer](skills/stem-interactive-deployer)** — Gera páginas HTML interativas pra ensinar conceitos STEM (sliders, gráficos, animações).

## Como instalar uma skill

1. Abra o Adapta ONE
2. Vá em `Configurações > Skills > Nova Skill`
3. Copie o conteúdo do `SKILL.md` da skill que quer instalar
4. Cole na nova skill, salve e ative

Algumas skills marcadas como *(auxiliar)* são dependências de outras. Se você instalar `adapta-slides`, instale também `adapta-slides-preview`. Se instalar `video-3d-cinematico`, instale também `cinematografia-base`.

## Sobre

Mantido por [Axel Rios](https://axelriosg.com), Head de Educação na Adapta.org.

[Adapta.org](https://adapta.org) é a maior plataforma de educação em IA generativa do Brasil.

## Licença

[MIT](LICENSE)
