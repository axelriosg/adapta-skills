#adapta-slides-preview
Preview HTML standalone de decks com Google Fonts nativo.
---
name: adapta-slides-preview
description: HTML preview hospedado em S3 via upload. Companion #adapta-slides.
---

# Preview

Companion de `#adapta-slides`. Gera HTML, sobe via `upload_local_file()`, entrega `preview_url`. **NUNCA** entregue `.html` via `present_files`.

```python
generate_html_preview(slides, br, "/tmp/preview.html")
result, err = upload_local_file("/tmp/preview.html")
preview_url = result["s3_url"]
```

Entrega `preview_url` ao user junto com `.pptx` via `present_files`. **NUNCA** entregue `.html` via `present_files`. Suporta 6 tipos core. Custom renderiza "Unsupported".

```python
from pptx.dml.color import RGBColor
CSS = """.slide-section_divider,.slide-close,.slide-manifesto-dark{background:var(--bg-dark)}.slide-manifesto-dark .mf-t{color:var(--on-dark)}.slide-manifesto-dark .mf-k,.slide-manifesto-dark .mf-a{color:var(--on-dark)}.slide-cover,.slide-evidence_grid,.slide-timeline_list,.slide-manifesto-light{background:var(--bg-primary)}.cover-kicker,.tc-kicker,.bs-kicker,.cl-kicker,.cover-meta-label,.bs-b-label{font-family:var(--mono);font-size:18px;letter-spacing:.4px}*{box-sizing:border-box;margin:0;padding:0}html,body{background:#1a1a1a;color:var(--text-primary);font-family:var(--sans);min-height:100vh;overflow:hidden}.deck{width:100vw;height:100vh;position:relative}.slide{position:absolute;inset:0;width:100%;height:100%;aspect-ratio:1440/810;max-width:calc(100vh*1440/810);max-height:calc(100vw*810/1440);margin:auto;display:none}.slide.active{display:block}.slide-body{width:1440px;height:810px;transform-origin:0 0;position:relative}.hdr,.ftr{position:absolute;left:90px;right:90px;display:flex;justify-content:space-between;font-family:var(--mono);font-size:18px;letter-spacing:.4px;color:var(--text-meta)}.hdr{top:33px}.ftr{bottom:33px}.hdr.dark,.ftr.dark{color:var(--on-dark)}.hdr-left{display:flex;align-items:center;gap:12px}.hdr-dot{width:13.5px;height:13.5px;border-radius:50%;background:var(--bg-dark)}.hdr.dark .hdr-dot{background:var(--bg-primary)}.cover-kicker{position:absolute;top:115px;left:90px;color:var(--text-meta)}.cover-prefix{position:absolute;top:153px;left:90px;font-family:var(--serif);font-size:72px;color:var(--text-soft);font-style:italic}.cover-title{position:absolute;top:245px;left:90px;right:90px;font-size:96px;color:var(--text-primary);line-height:1;font-weight:500;white-space:pre-line}.cover-meta{position:absolute;top:636px;left:90px;display:flex;gap:60px}.cover-meta-col{width:220px}.cover-meta-label{color:var(--text-meta);margin-bottom:8px}.cover-meta-value{font-size:19.5px;color:var(--text-meta)}.tc-kicker{position:absolute;top:73px;left:90px;color:var(--text-meta)}.tc-title{position:absolute;top:106px;left:90px;right:90px;font-size:48px;color:var(--text-primary);line-height:1.06;font-weight:500;white-space:pre-line}.tc-cols{position:absolute;top:267px;left:90px;display:grid;grid-template-columns:repeat(3,392px);gap:40px}.tc-col-kicker{font-family:var(--mono);font-size:18px;color:var(--accent);letter-spacing:.4px;margin-bottom:12px}.tc-col-stat{font-family:var(--serif);font-size:66px;color:var(--text-primary);line-height:1.05;margin-bottom:18px;white-space:pre-line}.tc-col-label{font-size:19.5px;color:var(--text-soft);line-height:1.35;margin-bottom:20px}.tc-col-rule{height:1px;background:var(--divider);margin-bottom:20px}.tc-col-note{font-size:19.5px;color:var(--text-meta);line-height:1.4}.slide-data_hero{background:var(--bg-secondary)}.bs-divider{position:absolute;top:72px;left:679.5px;width:1px;height:648px;background:var(--divider)}.bs-kicker{position:absolute;top:356px;left:90px;color:var(--text-meta)}.bs-title{position:absolute;top:393px;left:90px;width:530px;font-size:54px;color:var(--text-primary);line-height:1.04;font-weight:500;white-space:pre-line}.bs-lede{position:absolute;top:550px;left:90px;width:530px;font-size:25px;color:var(--text-soft);line-height:1.3}.bs-figure{position:absolute;top:100px;left:740px;width:610px;font-family:var(--serif);font-size:160px;color:var(--text-primary);font-style:italic;line-height:1;white-space:nowrap}.bs-caption{position:absolute;top:395px;left:740px;width:610px;font-size:22px;color:var(--text-meta);line-height:1.3;overflow-wrap:break-word}.bs-rule{position:absolute;top:437px;left:740px;width:610px;height:1px;background:var(--text-rule)}.bs-breakdown{position:absolute;top:462px;left:740px;display:grid;grid-template-columns:repeat(3,200px);gap:16px}.bs-b-label{color:var(--text-meta);margin-bottom:14px}.bs-b-value{font-size:36px;color:var(--text-primary);font-weight:700;margin-bottom:16px}.bs-b-desc{font-size:19.5px;color:var(--text-meta);line-height:1.35}.cl-kicker{position:absolute;top:73px;left:90px;color:var(--on-dark)}.cl-title{position:absolute;top:106px;left:90px;width:620px;font-size:108px;color:var(--on-dark);line-height:1.02;font-weight:500;white-space:pre-line}.cl-blurb{position:absolute;top:520px;left:90px;width:620px;font-family:var(--serif);font-size:33px;color:var(--on-dark);font-style:italic;line-height:1.18}.cl-partner{position:absolute;bottom:100px;left:90px;width:620px}.cl-p-label{font-family:var(--mono);font-size:18px;color:var(--on-dark);letter-spacing:.4px;margin-bottom:10px}.cl-p-name{font-size:25.5px;color:var(--on-dark);font-weight:700;margin-bottom:10px}.cl-p-contact{font-size:19.5px;color:var(--on-dark)}.cl-steps{position:absolute;top:140px;right:90px;width:620px;display:flex;flex-direction:column;gap:30px}.cl-step{display:grid;grid-template-columns:70px 1fr auto;gap:30px;padding-top:20px;border-top:1px solid var(--on-dark);align-items:start}.cl-step:first-child{border-top:none;padding-top:0}.cl-step-num{font-family:var(--serif);font-size:54px;color:var(--on-dark);font-style:italic}.cl-step-title{font-size:24px;color:var(--on-dark);font-weight:700;line-height:1.1;margin-bottom:12px}.cl-step-desc{font-size:17px;color:var(--on-dark);line-height:1.4}.cl-step-pill{display:inline-block;padding:8px 20px;border:1px solid var(--on-dark);border-radius:999px;font-family:var(--mono);font-size:18px;color:var(--on-dark);letter-spacing:.5px;align-self:start;margin-top:12px}.br-w{position:absolute;top:270px;left:90px;right:90px;display:flex;flex-direction:column;gap:22px}.br{display:grid;grid-template-columns:280px 500px 350px;gap:20px;align-items:center}.br-l{font-size:18px;color:var(--text-primary);font-weight:700}.br-t{height:26px;background:var(--divider);border-radius:3px;overflow:hidden}.br-f{height:100%;background:var(--accent)}.br-v{font-family:var(--serif);font-size:24px;color:var(--text-primary);font-style:italic;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}.br-n{position:absolute;bottom:60px;left:90px;font-size:12px;color:var(--text-meta);font-style:italic}.mf-k,.mf-a{position:absolute;left:0;right:0;text-align:center;font-family:var(--mono);font-size:18px;letter-spacing:.4px;color:var(--text-meta)}.mf-k{top:120px}.mf-a{bottom:120px}.mf-t{position:absolute;top:200px;left:90px;right:90px;bottom:200px;font-family:var(--serif);font-size:140px;color:var(--text-primary);font-style:italic;text-align:center;line-height:1;white-space:pre-line;display:flex;align-items:center;justify-content:center}.tl-w{position:absolute;top:200px;left:90px;right:90px}.tl-head{display:grid;grid-template-columns:160px 580px 520px;gap:20px;padding-bottom:16px;border-bottom:1px solid var(--divider);margin-bottom:16px}.tl-h{font-family:var(--mono);font-size:14px;letter-spacing:.4px;color:var(--text-meta);text-transform:uppercase}.tl-r{display:grid;grid-template-columns:160px 580px 520px;gap:20px;padding:16px 0;border-bottom:1px solid var(--divider)}.tl-r:last-child{border-bottom:0}.tl-r .tl-c0{font-family:var(--mono);font-size:18px;color:var(--text-meta)}.tl-r .tl-c1,.tl-r .tl-c2{font-family:var(--sans);font-size:20px;color:var(--text-primary)}"""
JS = """let c=0,S=document.querySelectorAll('.slide');function f(){const s=S[c];if(!s)return;const b=s.querySelector('.slide-body');if(!b)return;const r=s.getBoundingClientRect();b.style.transform=`scale(${Math.min(r.width/1440,r.height/810)})`}function show(i){c=Math.max(0,Math.min(S.length-1,i));S.forEach((s,j)=>s.classList.toggle('active',j===c));f()}addEventListener('keydown',e=>{if(e.key==='ArrowRight'||e.key===' ')show(c+1);if(e.key==='ArrowLeft')show(c-1)});addEventListener('resize',f);show(0)"""
FONT_LINK = 'https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=EB+Garamond:ital,wght@0,400;1,400;1,500&family=Roboto+Mono:wght@400;500&family=Instrument+Serif:ital@0;1&family=Playfair+Display:ital,wght@0,400;1,400;1,500&family=Source+Sans+Pro:wght@400;600&family=JetBrains+Mono:wght@400;500&display=swap'
def _e(s):
    return str(s).replace('&','&amp;').replace('<','&lt;').replace('>','&gt;').replace('"','&quot;')
def _to_hex(v):
    if isinstance(v, RGBColor): return f"#{str(v)}"
    if isinstance(v, str): return v if v.startswith("#") else f"#{v}"
    return "#000"
def _slide(stype, d, idx, meta, dark):
    name = (d.get("brand_name") or meta.get("name","BRAND")).upper()
    dc = " dark" if dark else ""
    right = d.get("confidential") or d.get("section_marker") or ""
    hdr = f'<div class="hdr{dc}"><div class="hdr-left"><div class="hdr-dot"></div><span>{_e(name)}</span></div><span>{_e(right)}</span></div>'
    ftr_l = d.get("sub_brand") or d.get("footer_note") or ""
    ftr = f'<div class="ftr{dc}"><span>{_e(ftr_l)}</span><span>{idx:02d}</span></div>'
    if stype == "cover":
        meta_html = ''.join(f'<div class="cover-meta-col"><div class="cover-meta-label">{_e(m["label"])}</div><div class="cover-meta-value">{_e(m["value"])}</div></div>' for m in d.get("meta_items",[]))
        body = f'<div class="cover-kicker">{_e(d.get("kicker",""))}</div><div class="cover-prefix">{_e(d.get("prefix",""))}</div><div class="cover-title">{_e(d["title"])}</div><div class="cover-meta">{meta_html}</div>'
    elif stype == "evidence_grid":
        cols = ''.join(f'<div><div class="tc-col-kicker">{_e(c["kicker"])}</div><div class="tc-col-stat">{_e(c["stat"])}</div><div class="tc-col-label">{_e(c["label"])}</div><div class="tc-col-rule"></div><div class="tc-col-note">{_e(c["note"])}</div></div>' for c in d.get("columns",[]))
        body = f'<div class="tc-kicker">{_e(d.get("kicker",""))}</div><div class="tc-title">{_e(d["title"])}</div><div class="tc-cols">{cols}</div>'
    elif stype == "data_hero":
        if d.get("chart"):
            bb=d["chart"].get("bars",[]); mv=max((x.get("numeric_value",0) for x in bb),default=1) or 1
            it=''.join(f'<div class="br"><div class="br-l">{_e(x.get("label",""))}</div><div class="br-t"><div class="br-f" style="width:{(x.get("numeric_value",0)/mv)*100}%"></div></div><div class="br-v">{_e(x.get("value",""))}</div></div>' for x in bb)
            fn=f'<div class="br-n">{_e(d.get("footer_note",""))}</div>' if d.get("footer_note") else ''
            body=f'<div class="tc-kicker">{_e(d.get("kicker",""))}</div><div class="tc-title">{_e(d["title"])}</div><div class="br-w">{it}</div>{fn}'
        else:
            bd=''.join(f'<div><div class="bs-b-label">{_e(b["label"])}</div><div class="bs-b-value">{_e(b["value"])}</div><div class="bs-b-desc">{_e(b["desc"])}</div></div>' for b in d.get("breakdown",[]))
            body=f'<div class="bs-divider"></div><div class="bs-kicker">{_e(d.get("kicker",""))}</div><div class="bs-title">{_e(d["title"])}</div><div class="bs-lede">{_e(d.get("lede",""))}</div><div class="bs-figure">{_e(d["figure"])}</div><div class="bs-caption">{_e(d.get("caption",""))}</div><div class="bs-rule"></div><div class="bs-breakdown">{bd}</div>'

    elif stype == "close":
        p = d.get("partner",{})
        steps = ''
        for i, st in enumerate(d.get("steps",[])):
            pill = f'<div class="cl-step-pill">{_e(st["date"])}</div>' if st.get("date") else ''
            steps += f'<div class="cl-step"><div class="cl-step-num">{i+1:02d}</div><div><div class="cl-step-title">{_e(st["title"])}</div><div class="cl-step-desc">{_e(st["desc"])}</div></div>{pill}</div>'
        contact = f'<div class="cl-p-contact">{_e(p.get("contact",""))}</div>' if p.get("contact") else ''
        body = f'<div class="cl-kicker">{_e(d.get("kicker","CLOSE"))}</div><div class="cl-title">{_e(d["title"])}</div><div class="cl-blurb">{_e(d.get("blurb",""))}</div><div class="cl-partner"><div class="cl-p-label">{_e(p.get("label",""))}</div><div class="cl-p-name">{_e(p.get("name",""))}</div>{contact}</div><div class="cl-steps">{steps}</div>'
    
    elif stype == "timeline_list":
        cols=d.get("columns",["time","event","response"])[:3]
        rows=d.get("rows",[])
        head=''.join(f'<div class="tl-h">{_e(c.upper())}</div>' for c in cols) if d.get("show_headers",True) else ''
        body_rows=''.join('<div class="tl-r">' + ''.join(f'<div class="tl-c{i}">{_e(str(r.get(cols[i],"")))}</div>' for i in range(len(cols))) + '</div>' for r in rows)
        body=f'<div class="tc-kicker">{_e(d.get("kicker",""))}</div><div class="tc-title">{_e(d["title"])}</div><div class="tl-w"><div class="tl-head">{head}</div>{body_rows}</div>'
    elif stype == "manifesto":
        cls="manifesto-dark" if d.get("dark") else "manifesto-light"
        k=f'<div class="mf-k">{_e(d.get("kicker",""))}</div>' if d.get("kicker") else ''
        a=f'<div class="mf-a">{_e(d.get("attribution",""))}</div>' if d.get("attribution") else ''
        body=f'{k}<div class="mf-t">{_e(d["title"])}</div>{a}'
        stype=cls  # override for CSS class
    else:
        body = f'<div style="padding:100px;color:red">Unsupported: {stype}</div>'
    active = " active" if idx == 1 else ""
    return f'<section class="slide slide-{stype}{active}"><div class="slide-body">{hdr}{body}{ftr}</div></section>'
def generate_html_preview(slides, brand, out_path):
    """Generate standalone HTML preview with Google Fonts. brand can be resolved or raw."""
    meta = brand.get("_meta", {})
    dark_types = {"section_divider", "close"}
    sections = '\n'.join(_slide(t, d, i+1, meta, t in dark_types) for i, (t, d) in enumerate(slides))
    css_vars = f""":root{{
  --bg-primary:{_to_hex(brand["bg_primary"])};
  --bg-secondary:{_to_hex(brand["bg_secondary"])};
  --bg-dark:{_to_hex(brand["bg_dark"])};
  --on-dark:{_to_hex(brand["on_dark"])};
  --accent:{_to_hex(brand["accent"])};
  --divider:{_to_hex(brand["divider"])};
  --text-primary:{_to_hex(brand["text_primary"])};
  --text-soft:{_to_hex(brand["text_soft"])};
  --text-rule:{_to_hex(brand["text_rule"])};
  --text-meta:{_to_hex(brand["text_meta"])};
  --sans:'{brand["sans"]}',sans-serif;
  --serif:'{brand["serif"]}',serif;
  --mono:'{brand["mono"]}',monospace
}}"""
    html = f'''<!DOCTYPE html><html><head><meta charset="UTF-8"><title>{_e(meta.get("name","Deck"))}</title>

<link href="{FONT_LINK}" rel="stylesheet">
<style>{css_vars}{CSS}</style></head>
<body><div class="deck">{sections}</div>

<script>{JS}</script></body></html>'''
    with open(out_path, 'w', encoding='utf-8') as f:
        f.write(html)
    return out_path```