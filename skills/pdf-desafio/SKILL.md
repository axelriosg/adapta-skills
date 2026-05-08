#pdf-desafio
Gerador de PDFs estilo "Desafio de 5 Dias" da Adapta.org.


name: pdf-desafio description: Gerador de PDFs estilo "Desafio de 5 Dias" da Adapta.org. Cria PDFs profissionais com design system completo replicando o estilo visual dos Desafios (capa impactante, terminal cards, grids de IAs, checklists, paginas de transicao, antes/depois cards, prompts em monospace). Use sempre que o usuario pedir para criar um PDF de Desafio, material de challenge, guia de 5 dias, ou qualquer PDF longo estilo workbook/challenge da Adapta. Tambem dispara em mencoes a "desafio de 5 dias", "challenge PDF", "workbook", "guia do desafio", "PDF para advogados", "PDF para [vertical]", ou qualquer pedido de material educacional longo (20+ paginas) com design premium no estilo Adapta. Dispara tambem quando o usuario cola conteudo de um desafio e pede para transformar em PDF.

Skill: PDF Desafio Adapta

Gera PDFs profissionais no estilo "Desafio de 5 Dias" da Adapta usando Python + reportlab + Instrument Sans. Para materiais educacionais longos (20-60 paginas) com design premium.

Diferenca do skill gerador-de-guia-de-curso-adapta-pdf: Aquele gera PDFs curtos (3-12 paginas, capa dark, conteudo cream, serif+sans). Este gera PDFs longos estilo workbook/challenge (30-60 paginas, fundo off-white, terminal cards, tipografia all-sans, proporcion mobile-first).



1. DESIGN SYSTEM

1.1 Paleta de Cores

# Fundo
BG_PAGE       = "#EDEEEA"   # off-white paper gray (NUNCA branco puro)
BG_WHITE      = "#FFFFFF"   # card body
BG_TRANSITION = "#2D4A40"   # dark forest green (paginas de transicao)
BG_TOC_DARK   = "#3C3C3C"   # charcoal (TOC metade direita)
BG_HEADER_BAR = "#E4E5E1"   # header bar sutil

# Accent
TEAL          = "#4AAFA0"   # cor primaria (card headers, links, section labels)
# NOTA: Este NAO e o #28C98A do brand Adapta. E um teal/seafoam mais suave.

# Texto
TEXT_MAIN     = "#2D2D2D"   # near-black para headings
TEXT_BODY     = "#3A3A3A"   # dark gray para body
TEXT_MUTED    = "#7A7A7A"   # muted gray (header, footer, labels)
TEXT_LIGHT    = "#C8D8D0"   # texto claro sobre fundo escuro (transicoes)

# Cards
CARD_BORDER   = "#D0D0D0"   # borda de cards

# Alertas
RED_ALERT     = "#C94040"   # erro/warning (armadilhas, cards "antes")
GOLD          = "#C4A030"   # aviso especial (dourado)


1.2 Tipografia

Font primaria: Instrument Sans (Google Fonts, convertida de woff2 via fontTools)

# Instalacao das fontes (OBRIGATORIO antes de gerar)
# 1. npm install @fontsource/instrument-sans
# 2. Converter woff2 -> ttf com fontTools:
from fontTools.ttLib import TTFont
for w, name in {'400': 'Regular', '600': 'SemiBold', '700': 'Bold'}.items():
    src = f'node_modules/@fontsource/instrument-sans/files/instrument-sans-latin-{w}-normal.woff2'
    font = TTFont(src); font.flavor = None
    font.save(f'InstrumentSans-{name}.ttf')

# Registro no reportlab
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont as RLFont
pdfmetrics.registerFont(RLFont('IS', 'InstrumentSans-Regular.ttf'))
pdfmetrics.registerFont(RLFont('IS-Bold', 'InstrumentSans-Bold.ttf'))
pdfmetrics.registerFont(RLFont('IS-SemiBold', 'InstrumentSans-SemiBold.ttf'))


Hierarquia tipografica:

Elemento Fonte Peso Tamanho Titulo capa (hero) IS-Bold Bold 82pt H1 titulo de pagina IS-Bold Bold 28pt Section header (monospace) Courier-Bold Bold 16pt ALL CAPS Body text IS (Regular) Regular 11.5pt Body bold IS-Bold Bold 11.5pt Card header Courier-Bold Bold 10.5pt ALL CAPS Card body Courier Regular 10pt Grid card header Courier-Bold Bold 8pt ALL CAPS Grid card body Courier Regular 8.5pt Header/Footer Courier / Courier-Bold Regular/Bold 8.5pt Bullet text IS (Regular) Regular 11.5pt Capa subtitulo IS-SemiBold SemiBold 10pt Capa "Desafio de 5 dias" IS-Bold Bold 52pt Textos verticais capa Helvetica / Helvetica-Bold - 7-9pt

REGRA CRITICA DE GLYPHS: Instrument Sans Latin NAO tem: checkmark (U+2713), triangle (U+25B6), delta (U+0394). Para esses caracteres, usar Helvetica/Helvetica-Bold como fallback.

1.3 Layout

W, H = 612, 979.2  # ~8.5" x 13.6" (proporcao ~5:8, mobile-first)

ML = 48            # margem esquerda
MR = 48            # margem direita
CW = W - ML - MR  # largura do conteudo (516pt)
MT = 56            # margem superior
MB = 50            # margem inferior

HEADER_Y = H - 30  # posicao y do header
FOOTER_Y = 22      # posicao y do footer


1.4 Leading (espacamento entre linhas)

LEAD_BODY = 16     # corpo de texto
LEAD_CARD = 14     # dentro de cards
LEAD_SMALL = 12    # texto pequeno


1.5 Espacamento entre elementos

Entre Gap Apos body text 4pt Apos body bold 2pt Antes de section header 10pt (breathing room) Apos section header 30pt Apos terminal card 14pt Apos H1 4pt Apos bullet LEAD_BODY Gap generico 8pt



2. TIPOS DE PAGINA

2.1 Capa (pagina 1)

Elementos:





Background: BG_PAGE (#EDEEEA)



Blob teal: 3 camadas de ellipses com alpha progressiva no canto superior esquerdo





Camada 1: 40 ellipses, TEAL, alpha 0.035 decrescente (diffuse outer)



Camada 2: 20 ellipses, #3DBFAB, alpha 0.06 decrescente (core saturado)



Camada 3: 10 ellipses, #7DD8C8, alpha 0.04 decrescente (bright highlight)



Titulo hero: 82pt IS-Bold, 4 linhas empilhadas (ex: "Inteligencia. / Artificial. / Para / Advogados."), comecando em y = H - 72



Label: "AI CHALLENGE" em IS-SemiBold 10pt, TEXT_MUTED



Textos verticais: "ADAPTA" na lateral esquerda (Helvetica-Bold 9pt rotated 90), tagline embaixo



"ARTIFICIAL INTELLIGENCE": IS-SemiBold 10pt, centralizado



"Desafio de 5 dias 2026": IS-Bold 52pt, canto inferior esquerdo



Cruzes decorativas (+): 6 posicoes, IS 18pt, CARD_BORDER



Texto vertical direita: descricao curta (Helvetica-Bold 6.5pt rotated -90)

NAO incluir: Badge "Certified Challenge" (removido por ficar estranho em reportlab)

2.2 Introducao (pagina 2)





Texto centrado, sem header/footer



"INTRODUCAO" em IS-Bold 18pt, TEAL



Subtitulo em IS-Bold 14pt



Paragrafos em IS Regular 11.5pt, centrados



"VAMOS COMECAR" em IS-Bold 16pt



Contato no rodape em IS 9pt muted

2.3 Sumario / TOC (pagina 3)





Split layout: ~42% esquerda (BG_PAGE), ~58% direita (BG_TOC_DARK #3C3C3C)



Blob teal no canto superior esquerdo (menor que a capa)



Numeros de pagina em IS-Bold 56pt, alinhados a direita na metade esquerda



Seta triangular (Helvetica 20pt, U+25B6) entre numero e titulo



Titulos em IS-Bold 16pt, branco, na metade escura



Gap vertical entre entries: 140pt

2.4 Pagina de transicao/intersticial





Background solid BG_TRANSITION (#2D4A40)



Texto em IS-Bold 24pt, TEXT_LIGHT (#C8D8D0)



Centralizado verticalmente (calculo automatico baseado no numero de linhas)



Sem header/footer



Usada entre dias e como "vamos para sua primeira vitoria"

2.5 Pagina de conteudo (todas as demais)

Header fixo:





Barra BG_HEADER_BAR (#E4E5E1), 42pt de altura



Linha teal 1.5pt na base



Esquerda: "DESAFIO DE 5 DIAS DE IA" em Courier 8.5pt, TEXT_MUTED



Direita: "■ DIA X" em Courier-Bold 8.5pt, TEXT_MAIN (com quadrado U+25B6)

Footer fixo:





Linha CARD_BORDER 0.5pt



Esquerda: "MATERIAL DISPONIBILIZADO PELA ADAPTA.ORG" (Courier, ADAPTA.ORG em TEAL)



Direita: numero da pagina em Courier-Bold

Corpo: comeca em y = H - 78 (18pt abaixo do header)



3. COMPONENTES

3.1 Terminal Card (componente principal)

Caixa com header colorido e body monospace. Imita visual de terminal/IDE.

+--[HEADER TEAL ALL CAPS]--[...]--+
|                                  |
|  Texto monospace do body...      |
|  Mais texto aqui...              |
|                                  |
+----------------------------------+


Specs:





Borda: CARD_BORDER 0.8pt



Header: 28pt altura, fundo TEAL (ou RED_ALERT para erros, GOLD para avisos)



Header texto: Courier-Bold 10.5pt, branco, ALL CAPS



Tres pontos (•••) no canto direito do header (#FFFFFF80)



Body: fundo branco, padding 12pt



Body texto: Courier 10pt, TEXT_BODY



Suporta markup:





[V] = checkbox verde com checkmark (Helvetica-Bold para glyph)



[X] = checkbox vermelho com X



**texto** = bold inline (Courier-Bold)

3.2 Grid Card (para secao "Escolhendo a IA Certa")

Card menor para layout em grid 3x3.

Specs:





3 colunas, gap 8pt, largura = (CW - 16) / 3



Altura fixa: 95pt



Header: 22pt, Courier-Bold 8pt ALL CAPS



Body: Courier 8.5pt, wrap automatico



Mesma estrutura de header colorido + body branco

3.3 Section Header

Bloco teal + texto monospace ALL CAPS + underline teal.

[■] TEXTO DO HEADER EM ALL CAPS
____________________________________


Specs:





Quadrado teal 14x14pt como icone



Texto em Courier-Bold 16pt, TEXT_MAIN, ALL CAPS



Underline teal 1.5pt, largura = texto + 26pt



Retorna y - 30

3.4 H1 (titulo de pagina)

IS-Bold 28pt, TEXT_MAIN. Wrap automatico em CW. Sem underline (diferente do skill de guia).

3.5 Checklist (dentro de terminal card)

Linhas [V] texto renderizam como:





Quadrado 12x12 preenchido TEAL



Checkmark "✓" em Helvetica-Bold 8pt branco DENTRO do quadrado



Texto em Courier 10pt apos o quadrado

3.6 Bullet points





Caractere "•" (U+2022) em IS 11.5pt



Texto em IS 11.5pt, indent 12pt apos o bullet



Wrap automatico

3.7 Bullet bold+value

Label em IS-Bold + valor em IS Regular na mesma linha. Usado para "Erro: ...", "Solucao: ..."



4. REGRAS DE COPY

Proibido





Travessao (em dash) NUNCA



Emoji como texto (nao renderizam na font)



Delta grego (U+0394) em Instrument Sans (nao existe no subset Latin)



Caractere U+2713 (checkmark) em Instrument Sans (usar Helvetica)



Caractere U+25B6 (triangle) em Instrument Sans (usar Helvetica)

Acentos





Todos os caracteres acentuados em portugues funcionam em Instrument Sans Latin



Usar unicode escapes no Python: \u00e7 (c cedilha), \u00e3 (a til), \u00e9 (e agudo), etc.

Tom





Direto, pratico, sem enrolacao



Informal mas profissional



Max 500 palavras/pagina



Prompts dentro de terminal cards (monospace)



5. ESTRUTURA DO DOCUMENTO

Estrutura padrao de um Desafio de 5 Dias:





Capa (1 pag)



Introducao (1 pag, concisa)



Sumario/TOC (1 pag, split layout)



Transicao "Mas antes de comecar..."



Manual Rapido (1-2 pags: 4Is, IA Certa, 3 Erros)



Pagina "Escolhendo a IA Certa" (grid 3x3 texto + grid 3x2 imagens)



3 Erros (cards vermelhos)



Aviso especial (se necessario, cards dourados)



Transicao "Vamos para sua primeira vitoria"



Dia 1 (6-8 pags cada dia)



Transicao "Ate amanha no Dia 2!"



Dia 2-5 (mesmo pattern)



Pagina final "Parabens! Voce completou o desafio!"

Pattern de cada Dia:

[H1] DIA X: Titulo do Dia
[Section] SUA VITORIA DE HOJE
[Body] Descricao + Resultado esperado
[Section] O CONCEITO-CHAVE
[Body] Explicacao
[Card] "Por que isso funciona" (checkmarks)
[Section] A VANTAGEM: ANTES VS DEPOIS
[Card vermelho] "Antes" (X checkmarks)
[Card teal] "Depois" (V checkmarks)
[Section] ROTA A: [Ferramenta gratuita]
[Body] Explicacao + bullets com passos
[Section] ROTA B: ADAPTA ONE [VERTICAL]
[Body] Passos
[Section] OS PROMPTS
[Card] Prompt 1, 2, 3, 4 (monospace)
[Section] EXEMPLO DE OUTPUT
[Card] Exemplo formatado
[Section] ARMADILHAS COMUNS
[Bold + bullets] Erro/Consequencia/Solucao x3
[Section] DICAS DE MESTRE
[Cards] Dica 1, 2, 3
[Section] CHECKLIST DA MISSAO DO DIA
[Card] Checklist com [V] items
[Section] PARABENS! VOCE COMPLETOU O DIA X
[Body] Resumo + proximo passo
[Card] "Prepare-se para o Dia X+1"




6. WORKFLOW





Receber input: vertical (advogados, vendas, marketing...) + conteudo dos 5 dias



Se faltar info, perguntar ANTES de gerar



Copiar as fontes InstrumentSans-*.ttf para o diretorio de trabalho



Usar o template base em assets/gen_desafio_template.py como ponto de partida



Customizar: vertical, IAs, prompts, exemplos, rotas A/B



Gerar o PDF com python gen_desafio_[vertical].py



Rasterizar com pdftoppm e checar visualmente (capa + 3-4 paginas de conteudo)



Corrigir problemas de overlap, glyphs quebrados, spacing



Entregar em /mnt/user-data/outputs/



7. ERROS COMUNS A EVITAR

Erro Solucao Emoji no texto (nao renderiza) Usar quadrado teal como icone de secao Delta/checkmark em Instrument Sans Usar Helvetica como fallback para glyphs especiais Titulo da capa cortado no topo y = H - 72 para 82pt (nao H - 50) Header bar sobrepondo H1 Conteudo comeca em y = H - 78, nunca H - 60 Texto sobreposto em transicoes Usar lista de linhas com calculo automatico de centralizacao Bold markers texto aparecendo literal Usar regex split no renderer de cards TOC com numeros errados Gerar PDF, usar PyMuPDF para encontrar paginas reais, atualizar Cards cortados no fim da pagina Checar y < MB + 140 antes de cada card, y < MB + 80 antes de texto Muito espaco vazio no rodape Ajustar thresholds de page break (80pt texto, 140pt cards) Fonte woff2 baixada como HTML Usar npm + fontTools para converter, NAO curl direto do GitHub



8. CHECKLIST FINAL





[ ] Fontes InstrumentSans-Regular/Bold/SemiBold.ttf presentes e registradas



[ ] Capa: titulo 82pt sem clipping, blob 3 camadas, sem badge, "ADAPTA" sem delta



[ ] TOC: numeros batem com paginas reais



[ ] Transicoes: texto centralizado sem overlap



[ ] Cards: checkmarks renderizam (Helvetica fallback)



[ ] Cards: bold inline renderiza sem asteriscos



[ ] Section headers: quadrado teal + ALL CAPS + underline



[ ] Header/footer em todas as paginas de conteudo



[ ] Sem travessao, sem emoji no texto



[ ] Acentos renderizando corretamente



[ ] Sem overlap em nenhuma pagina



[ ] Conteudo completo: intro + manual + 5 dias + final



[ ] Exportado tambem como PNGs 300dpi para Canva (opcional)



9. ASSETS

Os arquivos de template e fontes estao em assets/:





assets/gen_desafio_template.py - Script base completo com todo o design system



assets/days_content_template.py - Template para renderizar dias 2-5 usando sistema de sections



assets/InstrumentSans-Regular.ttf - Fonte Regular



assets/InstrumentSans-Bold.ttf - Fonte Bold



assets/InstrumentSans-SemiBold.ttf - Fonte SemiBold

Para usar: copiar assets para /home/claude, customizar conteudo, executar.
