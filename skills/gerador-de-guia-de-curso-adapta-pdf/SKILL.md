#gerador-de-guia-de-curso-adapta-pdf
Gerador de PDFs educacionais e materiais bonus para Adapta.org.
---

name: gerador-de-guia-de-curso-adapta-pdf

description: Gerador de PDFs educacionais e materiais bonus para Adapta.org. Cria PDFs profissionais light-theme (cream) com reportlab seguindo o design system da Adapta. Use sempre que o usuario pedir para criar PDFs, guias de curso, materiais bonus, eBooks, documentos de referencia, ou qualquer material educacional para a Adapta. Tambem dispara em mencoes a "PDF do curso", "guia do modulo", "material bonus", "eBook", "documento de referencia", "Site Perfeito", "Ideias Universalmente Uteis", ou qualquer pedido de material visual para distribuicao.

---

 

# Skill: Gerador de PDFs Adapta

 

Gera PDFs profissionais no padrao visual da Adapta.org usando Python + reportlab. Dois tipos:

 

| Tipo | Exemplo real | Paginas | Estrutura |

|---|---|---|---|

| **Guia de Curso** | "IA para Imagem" (Vini), "Como Criar Assistentes de IA" (Edson Muniz) | 3-12 | Capa dark + paginas cream com modulos, cards numerados, highlights, badges |

| **Material Bonus** | "Ideias Universalmente Uteis" (Skip) | 15-50 | Capa dark + intro + N sistemas (overview + prompt) + fechamento |

 

---

 

## 1. DESIGN SYSTEM

 

### 1.1 Modo padrao: LIGHT MODE (cream)

 

O padrao atual e light mode. Capa sempre dark, paginas de conteudo em cream.

 

```python

# CAPA (sempre dark)

BG_COVER    = "#0D1A2E"     # dark navy (default), trocar por curso

 

# PAGINAS DE CONTEUDO (light)

BG_PAGE     = "#F3EDE6"     # warm cream

BG_CARD     = "#FFFFFF"     # white cards

BG_HL       = "#2B3440"     # dark navy para highlight blocks

ACCENT      = "#3AAEE0"     # cor de destaque (trocar por curso)

ACCENT_LINE = "#4FC3F7"     # versao brighter para linhas/orbe

 

# TEXTO

TEXT_DARK   = "#2D2D2D"     # near-black para titulos

TEXT_BODY   = "#5A5A5A"     # gray para corpo

TEXT_MUTED  = "#999999"     # muted para footer

WHITE       = "#FFFFFF"     # texto em blocos escuros

BORDER      = "#E0D8CF"     # bordas e separadores (warm, matching cream)

```

 

### 1.2 Paleta de accent por curso

 

Trocar ACCENT + ACCENT_LINE + BG_COVER:

 

| Curso | ACCENT | ACCENT_LINE | BG_COVER |

|---|---|---|---|

| Padrao Adapta One | #28C98A | #28C98A | #071A10 |

| IA para Imagem (celeste) | #3AAEE0 | #4FC3F7 | #0D1A2E |

| Inteligencia Pragmatica (dourado) | #B8942E | #C9A84C | #0D1B1A |

| Experts avancado (roxo) | #6D33CC | #7C3AED | #1A0D2E |

| IA para Vendas (vermelho) | #D44638 | #E05545 | #1A0D0D |

 

### 1.3 Tipografia

 

**Hierarquia de fontes (REGRA CRITICA):**

 

| Elemento | Fonte | Peso | Tamanho |

|---|---|---|---|

| Titulo da pagina | Times-Bold (serif) | Bold | 28pt |

| Subtitulos de secao | Helvetica-Bold (sans) | Bold | 18pt |

| Titulo de card/item | Helvetica-Bold (sans) | Bold | 11pt |

| Titulo de arrow item | Helvetica-Bold (sans) | Bold | 10pt |

| Corpo principal | Times-Roman (serif) | Regular | 10pt |

| Corpo pequeno (cards) | Times-Roman (serif) | Regular | 9.5pt |

| Label de modulo | Helvetica-Bold (sans) | Bold | 8pt (UPPERCASE) |

| Badges | Helvetica-Bold (sans) | Bold | 7.5pt |

| Footer | Times-Roman (serif) | Regular | 8pt |

| Numeros grandes | Times-Bold (serif) | Bold | 24-26pt |

| Capa titulo | Times-Bold (serif) | Bold | 42pt |

 

**REGRA:** Sans-serif (Helvetica-Bold) para titulos de itens, labels, badges, subtitulos de secao. Serif (Times) para titulos de pagina, corpo, footer, numeros grandes. Isso cria hierarquia visual clara entre niveis.

 

```python

FS = FONT_SERIF      = "Times-Roman"

FB = FONT_SERIF_BOLD = "Times-Bold"

FN = FONT_SANS       = "Helvetica-Bold"

```

 

### 1.4 Layout

 

```python

from reportlab.lib.pagesizes import A4

W, H = A4  # 595.3 x 841.9 pt

 

ML, MR = 50, 50        # Margens laterais

CW     = W - ML - MR   # Content width = 495.3 pt

MB     = 44             # Margem inferior

 

# Posicoes Y fixas (y=0 no bottom)

FOOTER_Y   = 26

MOD_Y      = H - 44     # Label do modulo (topo)

TITLE_Y    = H - 78     # Titulo da pagina (36pt abaixo do modulo label)

BODY_START = H - 114    # Onde o corpo comeca

 

# Leading

BL = 15    # body leading

SL = 13    # small leading (cards)

```

 

### 1.5 Espacamento entre elementos

 

| Entre | Gap |

|---|---|

| Cards numerados | 10-14pt |

| Secoes maiores | 16-20pt |

| Apos badge | 22-24pt |

| Apos highlight block | 14-16pt |

| Arrow items | 6pt |

 

---

 

## 2. COMPONENTES REUTILIZAVEIS

 

### 2.1 Funcoes base

 

```python

def wrap(c, t, f, s, mw):

    ws, ls, cur = t.split(), [], ""

    for w in ws:

        x = cur+" "+w if cur else w

        if c.stringWidth(x, f, s) < mw: cur = x

        else:

            if cur: ls.append(cur)

            cur = w

    if cur: ls.append(cur)

    return ls

 

def bg(c, col=BG_PAGE):

    c.setFillColor(HexColor(col)); c.rect(0,0,W,H,fill=True,stroke=False)

 

def footer(c, n):

    c.setStrokeColor(HexColor(BORDER)); c.setLineWidth(0.4)

    c.line(ML, FOOTER_Y+12, W-MR, FOOTER_Y+12)

    c.setFont(FS, 8); c.setFillColor(HexColor(TEXT_MUTED))

    c.drawString(ML, FOOTER_Y, "Adapta.org")

    c.drawRightString(W-MR, FOOTER_Y, str(n))

 

def mod_hdr(c, t):

    c.setFont(FN, 8); c.setFillColor(HexColor(ACCENT))

    c.drawString(ML, MOD_Y, t.upper())

 

def new_pg(c, n, m):

    c.showPage(); n+=1; bg(c); footer(c,n); mod_hdr(c,m); return n

 

def title(c, t, sz=28):

    c.setFont(FB, sz); c.setFillColor(HexColor(TEXT_DARK))

    ls = wrap(c, t, FB, sz, CW)

    y = TITLE_Y

    for l in ls: c.drawString(ML, y, l); y -= sz+8

    # accent underline (~50% da largura do texto)

    uw = min(c.stringWidth(ls[0], FB, sz), CW*0.50)

    c.setStrokeColor(HexColor(ACCENT_LINE)); c.setLineWidth(2)

    c.line(ML, y+4, ML+uw, y+4)

    return y - 12

 

def body(c, y, t):

    c.setFont(FS, 10); c.setFillColor(HexColor(TEXT_BODY))

    for l in wrap(c, t, FS, 10, CW): c.drawString(ML, y, l); y -= BL

    return y - 6

 

def subtitle(c, y, t):

    """Subtitulo de secao com underline accent"""

    c.setFont(FN, 18); c.setFillColor(HexColor(TEXT_DARK))

    c.drawString(ML, y, t)

    tw = c.stringWidth(t, FN, 18)

    uw = min(tw, CW*0.4)

    c.setStrokeColor(HexColor(ACCENT_LINE)); c.setLineWidth(1.5)

    c.line(ML, y-6, ML+uw, y-6)

    return y - 28

 

def chk(c, y, need, n, m):

    """Page break check: se nao cabe, cria nova pagina"""

    if y-need < MB+30: n = new_pg(c, n, m); return BODY_START, n

    return y, n

```

 

### 2.2 Capa (dark + orbe)

 

Capa sempre dark. Orbe usa ACCENT_LINE com 25 camadas.

 

```python

def cover(c, title_lines, subtitle_text, author):

    bg(c, BG_COVER)

    cx, cy = W/2, H/2+30

    for i in range(25, 0, -1):

        c.setFillColor(HexColor(ACCENT_LINE))

        c.setFillAlpha(0.015+(25-i)*0.005)

        c.circle(cx, cy, i*9, fill=True, stroke=False)

    c.setFillAlpha(1)

    # logo

    c.setFont(FN, 13); c.setFillColor(HexColor(WHITE))

    c.drawCentredString(W/2, H-58, "ADAPTA")

    # accent line

    c.setStrokeColor(HexColor(ACCENT_LINE)); c.setLineWidth(1.2)

    c.line(W/2-16, H-67, W/2+16, H-67)

    # titulo (max 2 linhas)

    for i, line in enumerate(title_lines):

        c.setFont(FB, 42); c.setFillColor(HexColor(WHITE))

        c.drawCentredString(W/2, 250-i*54, line)

    # subtitulo

    c.setFont(FS, 12); c.setFillColor(HexColor("#B0C4D8"))

    c.drawCentredString(W/2, 120, subtitle_text)

    # separador sutil

    c.saveState()

    c.setStrokeColor(HexColor(ACCENT_LINE)); c.setStrokeAlpha(0.3); c.setLineWidth(0.4)

    c.line(W/2-40, 108, W/2+40, 108)

    c.restoreState()

    # autor

    c.setFont(FS, 11); c.setFillColor(HexColor("#8899AA"))

    c.drawCentredString(W/2, 92, author)

```

 

### 2.3 Card numerado (componente principal)

 

Card branco, numero grande accent, titulo SANS-SERIF BOLD, separador fino, descricao serif.

 

```python

def num_card(c, y, num, ttl, desc):

    dl = wrap(c, desc, FS, 9.5, CW-70)

    ch = max(54, 28 + len(dl)*SL + 14)

    cy = y - ch

    # card branco

    c.setFillColor(HexColor(BG_CARD))

    c.roundRect(ML, cy, CW, ch, 8, fill=True, stroke=False)

    # numero grande (accent, serif bold)

    c.setFont(FB, 26); c.setFillColor(HexColor(ACCENT))

    c.drawString(ML+12, cy+ch-32, str(num))

    # titulo (SANS-SERIF BOLD)

    tx = ML+56

    c.setFont(FN, 11); c.setFillColor(HexColor(TEXT_DARK))

    c.drawString(tx, cy+ch-20, ttl)

    # separador fino entre titulo e desc

    c.setStrokeColor(HexColor(BORDER)); c.setLineWidth(0.4)

    c.line(tx, cy+ch-26, ML+CW-16, cy+ch-26)

    # descricao (serif regular)

    c.setFont(FS, 9.5); c.setFillColor(HexColor(TEXT_BODY))

    ty = cy+ch-38

    for l in dl: c.drawString(tx, ty, l); ty -= SL

    return cy - 10

```

 

### 2.4 Highlight block (INSIGHT CHAVE / DICA PRO)

 

Bloco dark navy, barra lateral accent, label accent, texto branco.

 

```python

def hl(c, y, label, text):

    ls = wrap(c, text, FS, 10, CW-42)

    bh = len(ls)*BL + 42; by = y - bh

    c.setFillColor(HexColor(BG_HL))

    c.roundRect(ML, by, CW, bh, 8, fill=True, stroke=False)

    c.setFillColor(HexColor(ACCENT_LINE))

    c.rect(ML, by, 3.5, bh, fill=True, stroke=False)

    c.setFont(FN, 7.5); c.setFillColor(HexColor(ACCENT_LINE))

    c.drawString(ML+16, y-18, label.upper())

    c.setFont(FS, 10); c.setFillColor(HexColor(WHITE))

    ty = y - 34

    for l in ls: c.drawString(ML+16, ty, l); ty -= BL

    return by - 14

```

 

### 2.5 Badge

 

```python

def badge(c, x, y, t):

    tw = c.stringWidth(t, FN, 7.5)

    c.setFillColor(HexColor(ACCENT))

    c.roundRect(x, y-4, tw+16, 17, 9, fill=True, stroke=False)

    c.setFont(FN, 7.5); c.setFillColor(HexColor(WHITE))

    c.drawString(x+8, y+2, t)

```

 

### 2.6 Arrow item

 

Dot accent + titulo sans-serif bold + descricao serif.

 

```python

def arrow(c, y, ttl, desc):

    c.setFillColor(HexColor(ACCENT))

    c.circle(ML+7, y+3, 3, fill=True, stroke=False)

    tx = ML+20

    c.setFont(FN, 10); c.setFillColor(HexColor(TEXT_DARK))

    c.drawString(tx, y, ttl)

    if desc:

        c.setFont(FS, 9); c.setFillColor(HexColor(TEXT_BODY))

        ty = y-14

        for l in wrap(c, desc, FS, 9, CW-28):

            c.drawString(tx, ty, l); ty -= SL

        return ty - 6

    return y - 20

```

 

### 2.7 Checklist duas colunas

 

```python

def checks(c, y, items):

    mid = (len(items)+1)//2

    for i, it in enumerate(items[:mid]):

        c.setFillColor(HexColor(ACCENT))

        c.rect(ML+4, y-i*20, 5, 5, fill=True, stroke=False)

        c.setFont(FN, 9.5); c.setFillColor(HexColor(TEXT_DARK))

        c.drawString(ML+16, y-i*20, it)

    x2 = ML+CW/2

    for i, it in enumerate(items[mid:]):

        c.setFillColor(HexColor(ACCENT))

        c.rect(x2, y-i*20, 5, 5, fill=True, stroke=False)

        c.setFont(FN, 9.5); c.setFillColor(HexColor(TEXT_DARK))

        c.drawString(x2+12, y-i*20, it)

    return y - max(mid, len(items)-mid)*20 - 10

```

 

### 2.8 Framework vertical (PERFEITO, etc.)

 

```python

def framework_item(c, y, letter, name, desc):

    dl = wrap(c, desc, FS, 9.5, CW-70)

    ch = max(48, 20 + len(dl)*SL + 14)

    cy = y - ch

    # card branco

    c.setFillColor(HexColor(BG_CARD))

    c.roundRect(ML, cy, CW, ch, 8, fill=True, stroke=False)

    # circulo escuro com letra

    cx_c, cy_c = ML+24, cy+ch/2

    c.setFillColor(HexColor(BG_HL))

    c.circle(cx_c, cy_c, 16, fill=True, stroke=False)

    c.setFont(FB, 14); c.setFillColor(HexColor(ACCENT_LINE))

    tw = c.stringWidth(letter, FB, 14)

    c.drawString(cx_c-tw/2, cy_c-5, letter)

    # titulo sans bold

    tx = ML+52

    c.setFont(FN, 11); c.setFillColor(HexColor(TEXT_DARK))

    c.drawString(tx, cy+ch-18, name)

    # descricao serif

    c.setFont(FS, 9.5); c.setFillColor(HexColor(TEXT_BODY))

    ty = cy+ch-34

    for l in dl: c.drawString(tx, ty, l); ty -= SL

    return cy - 8

```

 

### 2.9 Card de prompt (Material Bonus)

 

```python

def prompt_card(c, y, prompt_text):

    ls = wrap(c, prompt_text, FS, 9.5, CW-32)

    ch = len(ls)*SL + 32; cy = y - ch

    c.setFillColor(HexColor(BG_CARD))

    c.setStrokeColor(HexColor(BORDER))

    c.roundRect(ML, cy, CW, ch, 8, fill=True, stroke=True)

    c.setFont(FS, 9.5); c.setFillColor(HexColor(TEXT_BODY))

    ty = y - 16

    for l in ls: c.drawString(ML+16, ty, l); ty -= SL

    return cy - 10

```

 

---

 

## 3. REGRAS DE COPY

 

### Proibido

 

- Nomes proprios de colaboradores (usar cargos)

- Mencoes a VSL, anuncio, venda, funil de marketing

- Nomes especificos de APIs externas

- Nomes de templates que podem mudar

- Travessao NUNCA

 

### Portugues

 

- "Qualquer" (uma palavra)

- "funcionalidade" (nao "feature")

- Sem anglicismos desnecessarios

- Acentos via unicode escapes (\u00e7, \u00e3o, \u00e9, etc.)

 

### Tom

 

- Direto, frases curtas

- Informal mas profissional

- Max 500 palavras/pagina

- 2-5 linhas por topico

 

---

 

## 4. WORKFLOW

 

1. Receber input (curso+autor+modulos OU tema+lista de sistemas)

2. Se faltar info, perguntar ANTES de gerar

3. Classificar: Tipo A (Guia) ou Tipo B (Bonus)

4. Identificar o ACCENT do curso na tabela 1.2

5. Montar estrutura de dados

6. Gerar script Python + executar

7. Renderizar com pymupdf e checar visualmente (pelo menos capa + 2 paginas de conteudo)

8. Corrigir erros de posicionamento/texto

9. Entregar em /mnt/user-data/outputs/

 

---

 

## 5. CHECKLIST FINAL

 

- [ ] Capa dark, conteudo cream (#F3EDE6)

- [ ] Cards brancos (#FFFFFF), cantos 8pt

- [ ] Highlights escuros (#2B3440) com barra accent + texto branco

- [ ] Titulos de pagina: serif bold 28pt, TEXT_DARK, com underline accent

- [ ] Titulos de itens: SANS-SERIF bold 11pt (Helvetica-Bold)

- [ ] Corpo: serif regular 9.5-10pt (Times-Roman)

- [ ] Numeros grandes: serif bold 24-26pt, cor ACCENT

- [ ] Separador fino BORDER entre titulo e desc dentro de cards

- [ ] Footer "Adapta.org" + numero com linha BORDER

- [ ] Module label: sans bold 8pt UPPERCASE, cor ACCENT

- [ ] Badges: pill accent com texto branco

- [ ] Sem travessao, sem nomes proprios

- [ ] Acentos renderizando

- [ ] Max 2 highlights por pagina

- [ ] MOD_Y e TITLE_Y com 36pt de gap (nunca colam)

 

---

 

## 6. ERROS COMUNS A EVITAR

 

| Erro | Solucao |

|---|---|

| Module label colando no titulo | MOD_Y = H-44, TITLE_Y = H-78 (36pt gap) |

| Glow/efeitos exagerados em paginas | Sem glows no conteudo. Orbe so na capa |

| Sem hierarquia visual (tudo parece igual) | Sans-serif bold para titulos de item, serif para corpo |

| Dark mode cansa para leitura | Cream + white cards. Dark so na capa e highlights |

| Cards sem separacao titulo/desc | Linha fina BORDER entre titulo e desc |

| Paginas com muito espaco vazio | Consolidar conteudo relacionado |

| Titulo sem destaque | Underline accent ~50% da largura |
