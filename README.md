# Adapta Skills

Coleção pública de Skills oficiais para a plataforma Adapta ONE, em português brasileiro.

## Sobre o repositório

Este repositório reúne dezenove Skills produzidas e utilizadas pela equipe da Adapta.org no contexto profissional, abrangendo desde criação de slides e documentos até produção de vídeo, ingestão de conteúdo e análise estilística. Cada Skill foi desenhada para resolver um caso de uso específico de forma reproduzível, e está disponível aqui em formato pronto para instalação no Adapta ONE.

A documentação oficial sobre o que são Skills e como funcionam dentro da plataforma está disponível em [docs.adapta.org/inicio-rapido/skills](https://docs.adapta.org/inicio-rapido/skills).

## Catálogo

### Meta

Skills que auxiliam na criação e refinamento de outras Skills.

- [skill-creator](skills/skill-creator), criação interativa de Skills via Q&A guiado, com suporte a cinco níveis de complexidade e onze padrões técnicos avançados.
- [cola-e-cria](skills/cola-e-cria), geração automática de Skills a partir de URLs ou textos colados, com suporte a YouTube, Instagram, blogs, PDFs e transcripts.
- [style-dna-extractor](skills/style-dna-extractor), análise quantitativa do estilo de comunicação a partir de transcripts ou amostras de texto.

### Apresentações

- [adapta-slides](skills/adapta-slides), geração de apresentações em PPTX editável com PDF de preview integrado.
- [adapta-slides-preview](skills/adapta-slides-preview) *(auxiliar)*, geração de preview HTML hospedado, utilizada em conjunto com adapta-slides.

### Documentos PDF

- [pdf-desafio](skills/pdf-desafio), produção de PDFs longos no formato editorial, com terminal cards e estrutura de checklist.
- [gerador-de-guia-de-curso-adapta-pdf](skills/gerador-de-guia-de-curso-adapta-pdf), produção de PDFs curtos no formato de guia de curso, com capa em tema escuro e páginas em creme.

### Visual estático

- [carrossel-instagram](skills/carrossel-instagram), produção de carrosséis em proporção 4:5 no formato 1080 por 1350 pixels, com exportação em ZIP de oito imagens prontas para o feed.

### Vídeo

- [diagrama-em-video](skills/diagrama-em-video), vídeos animados em estilo line art com narração em português brasileiro e onze layouts disponíveis.
- [vintage-video](skills/vintage-video), vídeos com estética de documentário antigo, utilizando shaders WebGL2 para grão, riscos, light leak e vinheta.
- [crt-terminal-video](skills/crt-terminal-video), vídeos animados de terminal com efeito CRT, no estilo da documentação técnica contemporânea.
- [video-3d-cinematico](skills/video-3d-cinematico), vídeos tridimensionais cinematográficos construídos com three.js, materiais PBR e pós-processamento.
- [cinematografia-base](skills/cinematografia-base) *(auxiliar)*, infraestrutura compartilhada para as Skills de vídeo 3D.
- [adapta-aula](skills/adapta-aula), micro-aulas em formato vertical 9:16 com voz humana sintetizada, legendas dinâmicas e ícones Lucide.

### Ingestão e análise

- [reel-em-texto](skills/reel-em-texto), transcrição de reels do Instagram em português brasileiro com refinamento estilístico.
- [extrator-videos-youtube](skills/extrator-videos-youtube), extração e transcrição dos três últimos vídeos de qualquer canal do YouTube, com geração automática de resumos, citações e newsletter.
- [anuncios-concorrentes](skills/anuncios-concorrentes), mapeamento de anúncios ativos de marcas na Facebook Ads Library e geração de variações alinhadas ao estilo Adapta.

### Design

- [design-clone-files](skills/design-clone-files), extração de design system completo a partir de imagens, PDFs, slides ou briefs enviados pelo usuário.

### Interativo

- [stem-interactive-deployer](skills/stem-interactive-deployer), geração de páginas HTML interativas para ensino de conceitos STEM, incluindo sliders, gráficos e animações.

## Como instalar uma Skill

A instalação de qualquer Skill deste repositório segue o procedimento padrão da plataforma, descrito em detalhe na documentação oficial em [docs.adapta.org/inicio-rapido/skills](https://docs.adapta.org/inicio-rapido/skills). Em resumo, basta abrir o Adapta ONE, acessar Configurações, ir em Skills, criar uma nova Skill, copiar o conteúdo do arquivo SKILL.md correspondente e colá-lo no editor. Após salvar e ativar, a Skill estará disponível para uso.

Algumas Skills marcadas como *(auxiliar)* funcionam como dependências de outras. Recomenda-se instalar adapta-slides em conjunto com adapta-slides-preview, e video-3d-cinematico em conjunto com cinematografia-base, para que o fluxo completo opere conforme projetado.

## Sobre a Adapta

[Adapta.org](https://adapta.org) é a maior plataforma brasileira de educação em inteligência artificial generativa. As Skills publicadas aqui refletem práticas e ferramentas desenvolvidas no contexto da operação da empresa, e são compartilhadas com a comunidade como referência aberta.

A manutenção deste repositório está a cargo de [Axel Rios](https://axelriosg.com), Head de Educação na Adapta.org.

## Licença

Este repositório é distribuído sob a licença [MIT](LICENSE).

## Isenção de responsabilidade

As Skills disponibilizadas neste repositório são fornecidas no estado em que se encontram, sem garantias expressas ou implícitas de qualquer natureza, incluindo, mas não se limitando a, garantias de adequação a um propósito específico, precisão de resultados ou compatibilidade com versões futuras da plataforma Adapta ONE.

O uso destas Skills é de responsabilidade exclusiva do usuário, que deve avaliar sua adequação ao caso concreto antes de qualquer aplicação em produção, especialmente em contextos profissionais, comerciais, jurídicos ou educacionais. A Adapta.org, seus mantenedores e colaboradores não se responsabilizam por quaisquer danos diretos, indiretos, incidentais ou consequenciais decorrentes do uso ou da impossibilidade de uso destas Skills, incluindo perda de dados, prejuízos financeiros ou interrupção de serviços.

Algumas Skills realizam chamadas a serviços de terceiros, como APIs públicas, bibliotecas de código aberto e plataformas de mídia. O usuário é responsável por verificar e cumprir os termos de uso aplicáveis a cada um desses serviços, bem como por garantir o tratamento adequado de dados pessoais conforme a legislação vigente, em particular a Lei Geral de Proteção de Dados Pessoais.

Resultados gerados por Skills, especialmente em fluxos que envolvem modelos de linguagem, podem conter imprecisões, omissões ou erros factuais. Recomenda-se revisão humana antes de qualquer publicação ou utilização em decisões com impacto material.
