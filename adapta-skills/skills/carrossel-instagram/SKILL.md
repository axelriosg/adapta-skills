---
name: carrossel-instagram
description: Gera carrosseis estaticos pro Instagram em 4:5 (1080x1350) com qualidade pixel-perfect via HTML/CSS + foreignObject + Canvas. Output e um HTML auto-contido que carrega Instrument Sans via fetch CDN, renderiza cada slide com tipografia real do browser (kerning, letter-spacing, antialiasing), e gera ZIP de PNGs prontos pro feed. Estrutura narrativa viral (capa hero dark, problema com pills vermelhas, principios, sintese, CTA verde). Suporta hero images via URL ou path local. Acentos PT-BR direto via UTF-8, sem hack regex. Dispara em "carrossel", "carrossel pro Instagram", "post carrossel", "slides Insta", "infografico Instagram", "carrossel viral", ou pedido descritivo de tema querendo posts em formato carrossel.
---

# Carrossel Instagram

Carrossel viral pro feed da @adapta_org. Renderiza HTML/CSS pixel-perfect via SVG foreignObject -> Canvas -> ZIP de PNGs 1080x1350.

## Postura

Diretor criativo + redator brasileiro de Instagram. Decide com confianca, sem pedir permissao. Tema claro, images definidas, slides preenchidos, gera. Headlines de Instagram, nao de relatorio.

## Anatomia

1. Capa = scroll-stopper. Hero image fullbleed em fundo dark + headline em 2 cores (white + GREEN) <=8 palavras + dark sub card + parens muted.
2. Slide 2 standalone. Hero image rounded no topo + headline + 4 pills vermelhas X + body + divider + emphasis.
3. 1 ideia por slide. <=15 palavras.
4. Slide final = CTA. Sem "Arraste".
5. "Arraste" + mao SVG canto inferior direito de TODOS menos o ultimo.
6. Pagination dots centro inferior de TODOS.
7. ADAPTΔ + @adapta_org canto inferior esquerdo de TODOS.

## Tipos de slide (classes CSS prontas)

- `s1` capa dark: hero fullbleed + h1 2 cores + sub card + parens
- `s2` problema cream: hero topo rounded + h1 + 4 pills X + body + emphasis
- `princ` principio: bigNum bg + badge color + h1 + (quote+attr+body) ou cards ou timeline ou insight
- `s7` sintese dark: rings + h1 multilinha colorida + body
- `s8` cta verde: h1 grande + body + pill

## Imagens

`IMAGES` lista por slide:
- URL `"https://..."` Python faz fetch e embeda base64
- Path local `"/home/user/img.jpg"` Python le e embeda base64
- `None` slide sem imagem

Imagens precisam ser geradas SEM texto sobreposto. A skill adiciona o texto.

## Boilerplate

```python
import json, time, os, base64, urllib.request, zlib

TEMA = "O metodo de US$ 3 trilhoes da Amazon"

# URL, path local, ou None - uma entrada por slide
IMAGES = [
    "https://...slide1_hero.jpg",
    "https://...slide2_hero.jpg",
    None, None, None, None, None, None,
]

# Os 8 slides como strings HTML. Edite SO o texto. NAO mude classes.
SLIDES = [
    '''<div class="slide s1"><div class="hero" data-img="0"></div><div class="grad"></div><div class="ct"><h1>US$ 3 <span class="g">TRILHÕES</span></h1><div class="sub">Foi quanto a Amazon valorizou aplicando um método simples que 90% das empresas brasileiras ainda erram com IA.</div><div class="par">(E provavelmente a sua é uma delas)</div></div>__FOOTER__</div>''',
    '''<div class="slide s2"><div class="hi" data-img="1"></div><h1>Antes de continuar, olhe pra<br/>sua empresa hoje:</h1><div class="pills"><div class="p"><span class="x">✕</span>Reuniões de 2h pra decidir o que IA decidiria em 4 min</div><div class="p"><span class="x">✕</span>Processos que sumiram quando o João pediu demissão</div><div class="p"><span class="x">✕</span>Planilha que alguém atualiza toda sexta às 18h</div><div class="p"><span class="x">✕</span>"Vou perguntar pro Pedro como faz isso"</div></div><div class="body">Se você marcou pelo menos 2, sua empresa está com o trabalho na caixa errada.</div><div class="div"></div><div class="em">E é exatamente por isso que IA ainda não funcionou aí dentro.</div>__FOOTER__</div>''',
    '''<div class="slide princ"><div class="bg">01</div><div class="badge green">PRINCÍPIO 1</div><h1>Trabalho na caixa certa</h1><div class="quote">"Se o trabalho mora em planilha, mora na cabeça do Pedro. E IA não acessa nem um nem outro."</div><div class="attr">— Andy Jassy, CEO Amazon</div><div class="body">Antes de pensar em IA, mude onde o trabalho vive. Sistema, não memória.</div>__FOOTER__</div>''',
    '''<div class="slide princ"><div class="bg">02</div><div class="badge gold">PRINCÍPIO 2</div><h1>Três lugares pra olhar</h1><div class="cards"><div class="card"><div class="l">DADOS</div><div class="t">Onde estão os dados que sua decisão precisa? Se não estão em sistema, IA não usa.</div></div><div class="card"><div class="l">DECISÃO</div><div class="t">Quem decide? Se a resposta é o Pedro, IA decide junto com ele ou no lugar dele.</div></div><div class="card"><div class="l">EXECUÇÃO</div><div class="t">Quem faz acontecer? Se é a Maria, IA cria a primeira versão e ela revisa.</div></div></div>__FOOTER__</div>''',
    '''<div class="slide princ"><div class="bg">03</div><div class="badge green">PRINCÍPIO 3</div><h1>Quatro frentes em paralelo</h1><div class="timeline"><div class="t-item"><div class="ct"><div class="l">MAPEIA</div><div class="t">Lista os 5 processos mais caros que ainda rodam na cabeça de alguém.</div></div></div><div class="t-item"><div class="ct"><div class="l">MOVE</div><div class="t">Move tudo pra sistema. Notion, planilha estruturada, banco. Tanto faz.</div></div></div><div class="t-item"><div class="ct"><div class="l">AUTOMATIZA</div><div class="t">Pluga IA na entrada e na saída. Não no meio. O meio você ainda controla.</div></div></div><div class="t-item"><div class="ct"><div class="l">MEDE</div><div class="t">Tempo gasto antes vs depois. Se não caiu 50%, você automatizou ruído.</div></div></div></div>__FOOTER__</div>''',
    '''<div class="slide princ"><div class="bg">04</div><div class="badge black">PRINCÍPIO 4</div><h1>O método é velho</h1><div class="quote">"Bezos escreveu isso em 2002. Antes de IA existir."</div><div class="attr">— API Mandate, Amazon</div><div class="insight"><div class="t">Quem já tinha o trabalho em sistema em 2023 colocou IA em três meses. Quem não tinha vai gastar dois anos só arrumando a casa.</div></div>__FOOTER__</div>''',
    '''<div class="slide s7"><div class="rings"></div><h1><span class="mute">Resumo:</span><br/>trabalho na caixa certa,<br/><span class="g">IA depois.</span></h1><div class="div"></div><div class="body">Se você inverte a ordem, IA só acelera o caos.</div>__FOOTER__</div>''',
    '''<div class="slide s8"><h1>Salve esse post</h1><div class="body">Volta nele toda vez que alguém na sua empresa disser "IA não funcionou aqui".</div><div class="pill">Segue @adapta_org pra mais</div>__FOOTER__</div>'''
]

# ===== MOTOR (NAO EDITAR) =====
N = len(SLIDES)
RUN_ID = f"{int(time.time()*1000)}_{os.getpid()}"
DARK_SLIDES = {0, 6, 7}

def load_image_b64(src):
    if not src: return None
    if src.startswith("http"):
        try:
            with urllib.request.urlopen(src, timeout=15) as r:
                return base64.b64encode(r.read()).decode()
        except Exception as e:
            print(f"img fail {src}: {e}"); return None
    if os.path.exists(src):
        with open(src, "rb") as f: return base64.b64encode(f.read()).decode()
    return None

IMG_B64 = [load_image_b64(s) for s in IMAGES]

def footer(idx, dark, last):
    cls = "dark" if dark else "light"
    dots = "".join('<span class="on"></span>' if i==idx else '<span></span>' for i in range(N))
    arr = '' if last else '<div class="arr"><span>Arraste</span><span class="hd"></span></div>'
    return f'<div class="f {cls}"><div class="brand"><span class="m">ADAPTΔ</span><span class="h">@adapta_org</span></div><div class="dots">{dots}</div>{arr}</div>'

for i in range(N):
    SLIDES[i] = SLIDES[i].replace('__FOOTER__', footer(i, i in DARK_SLIDES, i == N-1))
    for j, b64 in enumerate(IMG_B64):
        if b64:
            SLIDES[i] = SLIDES[i].replace(f'data-img="{j}"', f'style="background-image:url(data:image/jpeg;base64,{b64})"')

HAND_SVG = '''<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 28" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M19 16V7c0-1.1-.9-2-2-2s-2 .9-2 2v8.5"/><path d="M15 12V5c0-1.1-.9-2-2-2s-2 .9-2 2v10"/><path d="M11 13V8c0-1.1-.9-2-2-2s-2 .9-2 2v9"/><path d="M23 16V11c0-1.1-.9-2-2-2s-2 .9-2 2v7"/><path d="M7 17l-1.5 1.5C4.5 19.5 5 21 6 21l4 4c1 1 3 1 4 0l8.5-3c1.5-.5 2.5-2 2.5-3.5V14"/></svg>'''
HAND = base64.b64encode(HAND_SVG.encode()).decode()

CSS_B64 = "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"
JS_B64 = "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"

CSS = zlib.decompress(base64.b64decode(CSS_B64)).decode().replace('__HAND__', HAND)
JS = zlib.decompress(base64.b64decode(JS_B64)).decode()

FONT_URLS = {
    500: "https://api.fontsource.org/v1/fonts/instrument-sans/latin-ext-500-normal.woff2",
    600: "https://api.fontsource.org/v1/fonts/instrument-sans/latin-ext-600-normal.woff2",
    700: "https://api.fontsource.org/v1/fonts/instrument-sans/latin-ext-700-normal.woff2",
}

JS_FINAL = (JS
    .replace('__N__', str(N))
    .replace('__SLIDES__', json.dumps(SLIDES))
    .replace('__FONT_URLS__', json.dumps(FONT_URLS))
    .replace('__CSS_VAR__', '`' + CSS.replace('`', '\\`').replace('${', '\\${') + '`'))

HTML = f'''<!DOCTYPE html><html lang="pt-BR"><head><meta charset="UTF-8"><title>{TEMA}</title>
<style>{CSS}
html,body{{background:#0f0f0f;font-family:'IS',sans-serif;color:#aaa;padding:32px;display:flex;flex-direction:column;align-items:center;gap:20px;min-height:100vh}}
#hdr{{font-size:13px;font-weight:600;letter-spacing:.18em;text-transform:uppercase;color:#888}}#hdr b{{color:#ddd}}
#status{{font-weight:600}}
#preview-wrap{{width:540px;height:675px;overflow:hidden}}
#preview{{transform:scale(0.5);transform-origin:top left;box-shadow:0 14px 50px rgba(0,0,0,.55)}}
#nav{{display:flex;align-items:center;gap:14px;background:rgba(255,255,255,.04);border:1px solid rgba(255,255,255,.08);border-radius:999px;padding:6px 14px}}
#nav button{{width:34px;height:34px;border-radius:50%;border:1px solid rgba(255,255,255,.14);background:transparent;color:#ddd;cursor:pointer;font:inherit}}
#nav button:disabled{{opacity:.4}}#count{{font-size:13px;color:#888}}
#dl{{position:fixed;bottom:24px;right:24px;background:#2DA562;color:#0f0f0f;border:none;border-radius:10px;padding:14px 22px;font:700 14px 'IS';cursor:pointer;z-index:100;box-shadow:0 6px 20px rgba(45,165,98,.4)}}
#dl:disabled{{opacity:.6;cursor:wait}}</style></head><body>
<div id="hdr">Carrossel · <b>{TEMA}</b></div>
<div id="status">Carregando fontes...</div>
<div id="preview-wrap"><div id="preview"></div></div>
<div id="nav"><button id="prev">←</button><span id="count">1 / {N}</span><button id="next">→</button></div>
<button id="dl" disabled>Baixar ZIP (PNGs)</button>
<script>{JS_FINAL}</script></body></html>'''

_path = f'/home/user/carrossel_{RUN_ID}.html'
with open(_path, 'w', encoding='utf-8') as f: f.write(HTML)
result, error = upload_local_file(_path)
print(f"URL: {result['s3_url']}" if not error else f"ERRO: {error}")
```

## Anti-bug

1. Hero images via URL precisam de CORS aberto. S3 do Adapta ONE funciona. Stock externos podem falhar.
2. Acentos PT-BR vao DIRETO via UTF-8. NUNCA usar regex hack.
3. Edite SO o texto dentro dos slides. NAO mude classes CSS (s1, s2, princ, hero, hi, pills, etc) ou data-img.
4. Cada `data-img="N"` referencia o indice N de IMAGES. Se IMAGES[N] for None, slide renderiza sem imagem.
5. Imagens precisam ser geradas SEM texto sobreposto. A skill adiciona o texto.
6. Slide 8 (CTA) NAO leva "Arraste". E o ultimo slide.
7. Variar badge color entre principios: green, gold, green, black.
8. Tamanho fixo 4:5 (1080x1350).
9. Foreign object renderiza HTML como SVG embedded. SVGs inline DENTRO do HTML podem nao renderizar; use background-image data URL ou Unicode glyph.
10. Fontes carregam via fetch CDN (api.fontsource.org). Se network falhar, cai pra system-ui.

## Limites

ZIP final 1-3MB (PNGs ~300KB cada x 8). Render leva 8-15s no browser do user. CSS e JS comprimidos via zlib+base64 pra caber no skill cap.
