#adapta-slidesv5-org
Gera PPTX editável + PDF preview
---
name: adapta-slides
description: Gera PPTX editável + PDF preview + HTML preview hospedado S3 (companion #adapta-slidesv5 ). Dispara em slides, apresentação, deck, pitch deck, sales proposal, investor deck, all-hands, keynote, "faz um deck", "cria slides", pedido multi-slide editável. Substitui Gamma. Brand system via dict.
---

# Adapta Slides

.pptx editável + .pdf preview via python-pptx 1.0.2 e LibreOffice (pré-instalados).

## COMO USAR

1. Copiar TODO código desta skill (helpers+brand+responsive+renderers+orchestrator).
2. Montar `slides=[(type,data),...]` usando os 6 tipos: `cover`, `section_divider`, `three_column`, `big_stat`, `bullet_list`, `close`.
3. Chamar `generate(slides, pptx, pdf, brand=brand_dict)`.

**NUNCA** construir BRAND com RGBColor manual; use `brand={"theme":"consulting-light","accent":"#FF6B35"}` e deixe `resolve_brand()` converter. NUNCA reescrever renderers do zero. NUNCA hardcodar hex ou nome de fonte — sempre ler do brand resolvido.

**Exemplo**: `slides=[("cover",{"title":"X","prefix":"Introducing","meta_items":[{"label":"R","value":"V"}]}),("three_column",{"title":"Y","columns":[{"kicker":"A","stat":"12","label":"L","note":"N"}]*3}),("close",{"title":"Z","blurb":"...","partner":{"label":"CEO","name":"L","contact":"..."},"steps":[{"title":"X","desc":"Y","date":"WK 1"}]})]` → `generate(slides, "/mnt/user-data/outputs/deck.pptx", "/mnt/user-data/outputs/deck.pdf", brand={"name":"X","theme":"consulting-light"})`.

## CONTRATO

Renderer custom: `render_X(prs,b,d,idx,total)`:

1. `brand` vem resolvido — APENAS ler `brand["bg_primary"]` etc.
2. `header()` no topo + `footer()` embaixo. Obrigatórios.
3. Safe area: x[90,1350] y[73,730].
4. Títulos ≥40pt: via `fit_title_size()`. Hero figures ≥100pt: via `fit_text_size()`.
5. Stats horizontais: two-pass alignment (altura máx → alinhar labels).
6. Container: width ≥ `font_size × max_chars × 0.65`. Números curtos (<5 chars): width ≥ `font_size × 2`.
7. Registrar em `RENDERERS[type] = render_X`.

## PREVIEW HTML (companion)

Após `generate()`, chame `#adapta-slides-preview` (companion). Ele cria HTML standalone com Google Fonts nativas e hospeda via `upload_local_file()`, retornando URL pública. Entrega URL, não arquivo HTML.

```python
generate(slides, pptx, pdf, brand=brand)
br = resolve_brand(brand)
generate_html_preview(slides, br, "/tmp/preview.html")
result, err = upload_local_file("/tmp/preview.html")
preview_url = result["s3_url"] if not err else None

```

Entrega final ao user: `.pptx` via `present_files` + `preview_url` em texto clicável. **NUNCA** entregue `.html` via `present_files`.

## CANVAS & BRAND

1440×810pt. Safe x[90,1350] y[73,730]. Header y=37, footer y=751. Default theme: `consulting-light` (cream+dark-green). Override via brand dict.

```python
from pptx.dml.color import RGBColor
def hex_to_rgb(h):
    h=h.lstrip("#"); return RGBColor(int(h[0:2],16),int(h[2:4],16),int(h[4:6],16))
THEMES={"consulting-light": {"bg_primary":"#F2ECDF","bg_secondary":"#EBE3D3","bg_dark":"#1F3C2E",
        "on_dark":"#F3EDE0","accent":"#1F3D2E","accent_alt":"#B58A3D",
        "divider":"#D8D1C4","text_primary":"#141414","text_soft":"#2B2B28",
        "text_rule":"#131313","text_meta":"#6B6A64","parallel":"#131313",
        "sans":"Inter","serif":"EB Garamond","mono":"Roboto Mono",},}
def resolve_brand(b=None):
    if b is None: b="consulting-light"
    if isinstance(b,str): t=THEMES[b].copy()
    else:
        t=THEMES[b.get("theme","consulting-light")].copy()
        for k,v in b.items():
            if k in t: t[k]=v
        t["_meta"]={k:v for k,v in b.items() if k not in t}
    for k,v in list(t.items()):
        if isinstance(v,str) and v.startswith("#"): t[k]=hex_to_rgb(v)
    return t

```

**Presets**: `consulting-light` (default). Outros (startup-bold dark, monocle-editorial magazine, swiss-technical minimal): cliente passa override completo no brand dict com `{"bg_primary":"#...","accent":"#...","sans":"Inter",...}`.

## HELPERS

```python
from pptx import Presentation
from pptx.util import Pt
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN,MSO_ANCHOR
def _B(s,c):
    f=s.background.fill; f.solid(); f.fore_color.rgb=c
def _R(s,x,y,w,h,fill,line=None):
    r=s.shapes.add_shape(MSO_SHAPE.RECTANGLE,Pt(x),Pt(y),Pt(w),Pt(h))
    r.fill.solid(); r.fill.fore_color.rgb=fill
    if line is None: r.line.fill.background()
    else: r.line.color.rgb=line; r.line.width=Pt(0.75)
    r.shadow.inherit=False; return r
def _C(s,x,y,d,fill):
    c=s.shapes.add_shape(MSO_SHAPE.OVAL,Pt(x),Pt(y),Pt(d),Pt(d))
    c.fill.solid(); c.fill.fore_color.rgb=fill
    c.line.fill.background(); c.shadow.inherit=False; return c
def _T(s,x,y,w,h,text,font,size,color,bold=False,italic=False,
             align=PP_ALIGN.LEFT,anchor=MSO_ANCHOR.TOP,ls=0,lh=1.0):
    tb=s.shapes.add_textbox(Pt(x),Pt(y),Pt(w),Pt(h))
    tf=tb.text_frame
    tf.margin_left=tf.margin_right=tf.margin_top=tf.margin_bottom=0
    tf.word_wrap=True; tf.vertical_anchor=anchor
    for i,ln in enumerate(str(text).split("\n")):
        p=tf.paragraphs[0] if i==0 else tf.add_paragraph()
        p.alignment=align; p.lh=lh
        r=p.add_run(); r.text=ln
        r.font.name=font; r.font.size=Pt(size); r.font.color.rgb=color
        r.font.bold=bold; r.font.italic=italic
        if ls:
            r._r.get_or_add_rPr().set("spc",str(int(ls*100)))
    return tb
def _H(sl,b,name,right="",dark=False):
    c=b["on_dark"] if dark else b["text_meta"]
    dot=b["bg_primary"] if dark else b["bg_dark"]
    _C(sl,90,41.2,13.5,dot)
    _T(sl,114,33,800,24,name,b["mono"],18,c,ls=0.4)
    if right: _T(sl,800,33,550,24,right,b["mono"],18,c,align=PP_ALIGN.RIGHT,ls=0.4)
def _F(sl,b,left="",right="",dark=False):
    c=b["on_dark"] if dark else b["text_meta"]
    if left: _T(sl,90,747,900,24,left,b["mono"],18,c,ls=0.4)
    if right: _T(sl,1100,747,250,24,right,b["mono"],18,c,align=PP_ALIGN.RIGHT,ls=0.4)

```

## RESPONSIVE SIZING

```python
def fit_text_size(text,max_w,max_font=66,min_font=28):
    lines=text.split("\n") if text else [""]
    n=len(lines); longest=max((len(l) for l in lines),default=0)
    if longest==0: return max_font,n
    for sz in range(max_font,min_font-1,-2):
        if longest*sz*0.58<=max_w: return sz,n
    return min_font,n
def fit_title_size(text,max_w,max_font=108,min_font=56,max_lines=3):
    explicit=text.split("\n") if text else [""]
    for sz in range(max_font,min_font-1,-4):
        cw=sz*0.58; tot=0
        for ln in explicit:
            if not ln: tot+=1; continue
            lw=len(ln)*cw
            tot+=max(1,int(lw/max_w)+(1 if lw%max_w else 0))
        if tot<=max_lines: return sz,tot
    return min_font,max_lines+1

```

## ORCHESTRATOR

```python
def generate(slides,out_pptx,out_pdf=None,brand=None):
    br=resolve_brand(brand)
    prs=Presentation(); prs.slide_width=Pt(1440); prs.slide_height=Pt(810)
    total=len(slides)
    for idx,(stype,data) in enumerate(slides,1):
        RENDERERS[stype](prs,br,data,idx,total)
    prs.save(out_pptx)
    if out_pdf:
        import subprocess,os
        subprocess.run(["soffice","--headless","--convert-to","pdf",
                        "--outdir",os.path.dirname(out_pdf) or ".",out_pptx],check=True)

```

## RENDERERS

`render_X(prs,b,d,idx,total)`. Tudo via `brand`.

```python
def render_cover(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_primary"]); _H(s,b,name.upper(),d.get("confidential",""))
    _T(s,90,115,1260,24,d.get("kicker",""),b["mono"],18,b["text_meta"],ls=0.4)
    _T(s,90,153,1260,100,d.get("prefix",""),b["serif"],72,b["text_soft"],italic=True)
    sz,nl=fit_title_size(d["title"],1260,max_font=96,min_font=56,max_lines=3)
    h=int(sz*1.02*nl)+10
    _T(s,90,245,1260,h,d["title"],b["sans"],sz,b["text_primary"],lh=1.0)
    cx=[90,327,576]
    for i,m in enumerate(d.get("meta_items",[])[:3]):
        _T(s,cx[i],636,230,24,m["label"],b["mono"],18,b["text_meta"],ls=0.4)
        _T(s,cx[i],665,230,28,m["value"],b["sans"],19.5,b["text_meta"])
    _F(s,b,d.get("sub_brand",""),f"{idx:02d} / {total}")
def render_section_divider(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_dark"]); _H(s,b,name.upper(),d.get("section_marker",""),dark=True)
    _T(s,90,161,500,24,d.get("part_label",""),b["mono"],18,b["on_dark"],ls=0.4)
    sz,nl=fit_title_size(d["title"],1260,max_font=150,min_font=80,max_lines=3)
    h=int(sz*0.95*nl)+10
    _T(s,90,196,1260,h,d["title"],b["sans"],sz,b["on_dark"],lh=0.95)
    _T(s,90,196+h+20,1050,110,d.get("subtitle",""),b["serif"],36,b["on_dark"],italic=True,lh=1.25)
    _F(s,b,d.get("footer_note",""),f"{idx:02d} / {total}",dark=True)
def render_three_column(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_primary"]); _H(s,b,name.upper(),d.get("section_marker",""))
    _T(s,90,73,1260,24,d.get("kicker",""),b["mono"],18,b["text_meta"],ls=0.4)
    _T(s,90,106,1260,120,d["title"],b["sans"],48,b["text_primary"],lh=1.06)
    cx=[90,524,958]; cw=392
    cols=d.get("columns",[])[:3]
    sizes=[]
    for c in cols:
        sz,nl=fit_text_size(c["stat"],cw,max_font=66,min_font=36)
        sizes.append((sz,nl,int(sz*1.15*nl)))
    mh=max((h for _,_,h in sizes),default=90)
    ly=298+mh+10; ry=max(469,ly+80)
    for i,c in enumerate(cols):
        x=cx[i]; sz,nl,h=sizes[i]
        _T(s,x,267,cw,24,c["kicker"],b["mono"],18,b["accent"],ls=0.4)
        _T(s,x,298,cw,h,c["stat"],b["serif"],sz,b["text_primary"],lh=1.05)
        _T(s,x,ly,cw,80,c["label"],b["sans"],19.5,b["text_soft"],lh=1.35)
        _R(s,x,ry,cw,0.75,b["divider"])
        _T(s,x,ry+20,cw,150,c["note"],b["sans"],19.5,b["text_meta"],lh=1.4)
    _F(s,b,d.get("footer_note",""),f"{idx:02d} / {total}")
def render_big_stat(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_secondary"]); _H(s,b,name.upper(),d.get("section_marker",""))
    _R(s,679.5,72,0.75,648,b["divider"])
    _T(s,90,356,530,24,d.get("kicker",""),b["mono"],18,b["text_meta"],ls=0.4)
    tsz,tnl=fit_title_size(d["title"],530,max_font=54,min_font=34,max_lines=3)
    th=int(tsz*1.04*tnl)+6
    _T(s,90,393,530,th,d["title"],b["sans"],tsz,b["text_primary"],lh=1.04)
    _T(s,90,393+th+12,530,200,d.get("lede",""),b["sans"],25,b["text_soft"],lh=1.3)
    fsz,_=fit_text_size(d["figure"],610,max_font=200,min_font=120)
    _T(s,740,100,610,290,d["figure"],b["serif"],fsz,b["text_primary"],italic=True)
    cap=d.get("caption","")
    _T(s,740,395,610,70,cap,b["sans"],22,b["text_meta"],lh=1.3)
    dy=437 if len(cap)<55 else 470
    _R(s,740,dy,610,0.75,b["text_rule"])
    bx=[740,955,1171]; by=dy+25
    for i,br in enumerate(d.get("breakdown",[])[:3]):
        _T(s,bx[i],by,200,24,br["label"],b["mono"],18,b["text_meta"],ls=0.4)
        _T(s,bx[i],by+36,200,45,br["value"],b["sans"],36,b["text_primary"],bold=True)
        _T(s,bx[i],by+95,200,120,br["desc"],b["sans"],19.5,b["text_meta"],lh=1.35)
    _F(s,b,d.get("footer_note",""),f"{idx:02d} / {total}")
def render_bullet_list(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_primary"]); _H(s,b,name.upper(),d.get("section_marker",""))
    _T(s,90,73,1260,24,d.get("kicker",""),b["mono"],18,b["text_meta"],ls=0.4)
    sz,nl=fit_title_size(d["title"],1260,max_font=48,min_font=32,max_lines=2)
    h=int(sz*1.06*nl)+8
    _T(s,90,106,1260,h,d["title"],b["sans"],sz,b["text_primary"],lh=1.06)
    bullets=d.get("bullets",[])[:6]
    bs=106+h+40; avail=720-bs; sp=avail/max(len(bullets),1)
    for i,bl in enumerate(bullets):
        y=bs+i*sp
        _T(s,90,y,30,40,"—",b["sans"],24,b["accent"],bold=True)
        _T(s,130,y,1220,60,bl,b["sans"],22,b["text_primary"],lh=1.35)
    _F(s,b,d.get("footer_note",""),f"{idx:02d} / {total}")
def render_close(prs,b,d,idx,total):
    name=d.get("brand_name") or b.get("_meta",{}).get("name","BRAND")
    s=prs.slides.add_slide(prs.slide_layouts[6])
    _B(s,b["bg_dark"]); _H(s,b,name.upper(),d.get("section_marker","CLOSE"),dark=True)
    _T(s,90,73,500,24,d.get("kicker","CLOSE"),b["mono"],18,b["on_dark"],ls=0.4)
    tsz,tnl=fit_title_size(d["title"],620,max_font=108,min_font=56,max_lines=3)
    th=int(tsz*1.02*tnl)+10
    _T(s,90,106,620,th,d["title"],b["sans"],tsz,b["on_dark"],lh=1.02)
    _T(s,90,106+th+20,620,120,d.get("blurb",""),b["serif"],33,b["on_dark"],italic=True,lh=1.18)
    p=d.get("partner",{})
    _T(s,90,600,620,22,p.get("label",""),b["mono"],18,b["on_dark"],ls=0.4)
    _T(s,90,632,620,32,p.get("name",""),b["sans"],25.5,b["on_dark"],bold=True)
    if p.get("contact"): _T(s,90,670,620,24,p["contact"],b["sans"],19.5,b["on_dark"])
    steps=d.get("steps",[])[:3]
    sp=580/max(len(steps),1)
    for i,st in enumerate(steps):
        y=int(140+i*sp)
        if i>0: _R(s,730,y-20,620,0.75,b["on_dark"])
        _T(s,730,y,70,70,f"{i+1:02d}",b["serif"],54,b["on_dark"],italic=True)
        tw=280 if st.get("date") else 400
        _T(s,830,y+10,tw,70,st["title"],b["sans"],24,b["on_dark"],bold=True,lh=1.1)
        _T(s,830,y+70,400,100,st["desc"],b["sans"],17,b["on_dark"],lh=1.4)
        if st.get("date"):
            pl=s.shapes.add_shape(MSO_SHAPE.ROUNDED_RECTANGLE,Pt(1230),Pt(y+18),Pt(120),Pt(36))
            pl.fill.background(); pl.line.color.rgb=b["on_dark"]; pl.line.width=Pt(1)
            pl.adjustments[0]=0.5; pl.shadow.inherit=False
            _T(s,1230,y+24,120,24,st["date"],b["mono"],18,b["on_dark"],
                     align=PP_ALIGN.CENTER,ls=0.5)
    _F(s,b,d.get("footer_note",""),f"{idx:02d} / {total}",dark=True)
RENDERERS={"cover":render_cover,"section_divider":render_section_divider,
           "three_column":render_three_column,"big_stat":render_big_stat,
           "bullet_list":render_bullet_list,"close":render_close}

```

## WORKFLOW

1. Perguntar: tópico, audiência, num slides (default 8-12), brand.
2. Brand onboarding: nome empresa, cores (se tiver), tema (consulting-light default, ou startup-bold/editorial). Sem info = consulting-light.
3. NUNCA inventar números/nomes/quotes.
4. Montar `slides = [(tipo, data), ...]` usando APENAS os 6 tipos prontos.
5. Chamar `generate(slides, pptx, pdf, brand=brand)`. Entregar em `/mnt/user-data/outputs/`.

## RULES

1. Output em `/mnt/user-data/outputs/`.
2. Usar `generate()` + renderers prontos. Não reescrever.
3. Cores/fontes sempre do `brand` dict resolvido.
4. Em-dash `—` OK em PDF, no chat nunca.
5. Title aceita `\n` pra line breaks.

## CHECK

PPTX abre. PDF OK. 1440×810pt. Page numbers no footer. Zero overflow. Zero dados inventados. Cores do brand dict (não hardcoded).