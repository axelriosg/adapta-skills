#design-clone-files
Extrai design system completo 
---

name: design-clone-files

description: Extrai design system (cores, fontes, espaçamentos, componentes) de imagens, PDFs, slides PPTX ou briefs uploadados. Entrega 2 downloads: skill portátil pra colar em outros chats como contexto, e preview HTML demonstrando tudo aplicado. Ative em design system, paleta, cores, fontes, brand, identidade visual, "extrai esse design", "cria sistema desse mockup", "pega o estilo desse PDF", mesmo sem pedir explicitamente.

---

# Design Clone Files

Extrai design system de arquivos privados (imagens, PDFs, PPTX, briefs). Entrega 2 downloads: skill portátil design-{slug}.skill.md) pra colar em outros chats como contexto, e preview HTML.

## Fluxo em 3 fases (obrigatório)

**Fase 1 — Inspect (Python):** lista arquivos, extrai texto (PDF/PPTX/MD), converte PDFs em imagens, identifica imagens.

**Fase 2 — Analyze (você, com vision):** ABRA cada imagem com view, lê texto extraído, PREENCHE DESIGN_CONTENT com valores reais. Não chute, não copie exemplo.

**Fase 3 — Execute (Python):** valida anti-placeholder + anti-hardcode, salva DESIGN.md, gera preview HTML, monta skill portátil, faz upload.

## Tipos de input

| Formato | Análise |

|---|---|

| PNG/JPG/WEBP/GIF | Você abre via view |

| PDF | Texto via pypdf + páginas convertidas em PNG (você abre com view) |

| PPTX | Texto + theme colors via python-pptx (até 30 slides) |

| TXT/MD | Direto |

| Múltiplos | Prioridade: doc escrito > screenshot > mockup > logo |

## Guidelines de extração

**Mínimos em DESIGN_CONTENT:**

- ≥10 cores: primary, secondary, tertiary, surface, surface-container, background, on-primary, on-surface, on-background, outline (+ error se houver vermelho)

- ≥6 typography: display-lg, headline-lg, headline-md, body-lg, body-md, label-sm

- ≥4 components: button-primary, button-secondary, card, input-field

- Prose ≥2-3 frases por seção

**Logo isolada:** sem mockup, extraia só paleta. Typography com nota "Não derivado dos uploads". Components, omita.

**Mapping cores:** CTA → primary + on-primary. Lum<0.2 → background/surface. Lum>0.8 sobre escuro → on-*. Saturadas baixa freq → tertiary.

**Mapping tipografia:** maior fontSize → display-lg, decrescente até label-*. Valores exatos.

**Anti-hardcode (validado):** TODO valor em components: deve referenciar token via {colors.xxx}, {rounded.xxx}. Skill quebra se encontrar hex em components.

**Não inventar:** valor não observável, omita.

## Código

### Fase 1 — Inspect

```python

import os, json, subprocess, re, time, yaml, html as _h

from pathlib import Path

from itertools import islice

# === CONFIG ===

BRAND_SLUG = "cliente"  # OBRIGATÓRIO substituir baseado nos uploads

# === LOCALIZAR UPLOADS ===

UPLOADS_DIR = Path("/mnt/user-data/uploads")

if not UPLOADS_DIR.exists():

    for alt in ["/mnt/uploads", "/tmp/uploads", os.path.expanduser("~/uploads")]:

        if Path(alt).exists():

            UPLOADS_DIR = Path(alt); break

if not UPLOADS_DIR.exists():

    raise RuntimeError("Pasta de uploads não encontrada. Esperado /mnt/user-data/uploads. Peça pro usuário fazer upload primeiro.")

# === LISTAR E EXTRAIR ===

print(f"[Fase 1] Inspecionando {UPLOADS_DIR}...")

files = sorted([p for p in UPLOADS_DIR.iterdir() if p.is_file()])

if not files:

    raise RuntimeError("Nenhum arquivo uploadado. Peça pro usuário adicionar imagens, PDFs, PPTX ou briefs.")

IMG = {'.png','.jpg','.jpeg','.webp','.gif'}

TXT = {'.md','.txt'}

extracted_text = {}

image_files = []

for p in files:

    ext = p.suffix.lower()

    if ext in IMG: kind, _ = "image", image_files.append(p)

    elif ext == '.pdf': kind = "pdf"

    elif ext in ('.pptx','.ppt'): kind = "slides"

    elif ext in TXT: kind = "text"

    else: kind = "other"

    print(f"  [{kind}] {p.name} ({p.stat().st_size/1024:.1f} KB)")

    try:

        if kind == "text":

            extracted_text[p.name] = p.read_text(encoding='utf-8', errors='replace')

        elif kind == "pdf":

            from pypdf import PdfReader

            reader = PdfReader(str(p))

            extracted_text[p.name] = "\n\n".join(pg.extract_text() or "" for pg in reader.pages[:20])

            try:

                from pdf2image import convert_from_path

                pages = convert_from_path(str(p), dpi=120, first_page=1, last_page=8)

                for i, img in enumerate(pages):

                    out = f"/tmp/pdf_{p.stem}_pg{i+1}.png"

                    img.save(out, "PNG")

                    image_files.append(Path(out))

                print(f"    {len(pages)} imagens (abra com view)")

            except Exception as e:

                print(f"    PDF→imagem falhou: {e}")

        elif kind == "slides":

            from pptx import Presentation

            prs = Presentation(str(p))

            slides_txt, theme_colors = [], set()

            for i, slide in enumerate(islice(prs.slides, 30)):

                content = [f"--- Slide {i+1} ---"]

                try: shapes_iter = list(slide.shapes)

                except Exception: shapes_iter = []

                for shape in shapes_iter:

                    try:

                        if getattr(shape, 'has_text_frame', False):

                            for para in shape.text_frame.paragraphs:

                                txt = para.text.strip()

                                if txt: content.append(txt)

                                for run in para.runs:

                                    try:

                                        c = run.font.color

                                        if c and c.type is not None and c.rgb: theme_colors.add(f"#{c.rgb}")

                                    except Exception: pass

                        try:

                            f = getattr(shape, 'fill', None)

                            if f and f.type == 1:

                                rgb = f.fore_color.rgb

                                if rgb: theme_colors.add(f"#{rgb}")

                        except Exception: pass

                    except Exception: pass

                slides_txt.append("\n".join(content))

            extracted_text[p.name] = "\n".join(slides_txt)

            if theme_colors:

                extracted_text[p.name] += "\n\n=== THEME COLORS ===\n" + "\n".join(sorted(theme_colors))

    except Exception as e:

        print(f"    extração de {p.name} falhou: {e}")

if extracted_text:

    print(f"\n[Texto extraído de {len(extracted_text)} arquivo(s)]")

    for name, txt in extracted_text.items():

        print(f"=== {name} ===")

        print(txt[:2500] + ("..." if len(txt) > 2500 else ""))

        print()

if image_files:

    print(f"\n[ATENÇÃO] {len(image_files)} imagem(ns) — ABRA cada uma com view ANTES de preencher:")

    for p in image_files: print(f"  - {p}")

print(f"\n[Próximo: Fase 2] Olhe imagens, depois preencha DESIGN_CONTENT. Substitua BRAND_SLUG ('{BRAND_SLUG}').")

```

### Fase 2 — Você analisa e preenche

**ATENÇÃO:** Não copie nenhum exemplo. Abra cada imagem com view PRIMEIRO. Depois preencha.

1. Abra cada imagem em image_files com view (incluindo PDFs convertidos)

2. Identifique paleta real, tipografia, radii, spacings

3. Substitua TODOS os [OBSERVE] com valores observados

4. Substitua BRAND_SLUG

```python

BRAND_SLUG = "[OBSERVE]"  # nome curto descritivo dos uploads

DESIGN_CONTENT = """---

name: [OBSERVE]

description: [OBSERVE - 1 linha sobre vibe/identidade]

colors:

  primary: "[OBSERVE]"

  secondary: "[OBSERVE]"

  tertiary: "[OBSERVE]"

  surface: "[OBSERVE]"

  surface-container: "[OBSERVE]"

  background: "[OBSERVE]"

  on-primary: "[OBSERVE]"

  on-surface: "[OBSERVE]"

  on-background: "[OBSERVE]"

  outline: "[OBSERVE]"

typography:

  display-lg: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]", lineHeight: "[OBSERVE]"}

  headline-lg: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]"}

  headline-md: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]"}

  body-lg: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]"}

  body-md: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]"}

  label-sm: {fontFamily: "[OBSERVE]", fontSize: "[OBSERVE]", fontWeight: "[OBSERVE]"}

rounded: {sm: [OBSERVE], md: [OBSERVE], lg: [OBSERVE], xl: [OBSERVE], full: 9999px}

spacing: {sm: [OBSERVE], md: [OBSERVE], lg: [OBSERVE], xl: [OBSERVE]}

components:

  button-primary: {backgroundColor: "{colors.primary}", textColor: "{colors.on-primary}", rounded: "{rounded.full}"}

  button-secondary: {backgroundColor: "transparent", borderColor: "{colors.outline}", textColor: "{colors.on-surface}"}

  card: {backgroundColor: "{colors.surface-container}", rounded: "{rounded.lg}"}

  input-field: {backgroundColor: "{colors.surface}", borderColor: "{colors.outline}"}

---

## Overview

[OBSERVE - 2-3 frases sobre vibe, baseadas no que você VIU nas imagens]

## Colors

[OBSERVE - rationale por cor-chave: por que essa cor pra esse role]

## Typography

[OBSERVE - famílias e hierarquia observadas]

## Components

[OBSERVE - comportamento de cada componente]

"""

```

### Fase 3 — Validação + saída

```python

# === ANTI-PLACEHOLDER ===

forbidden = ['[OBSERVE]', '"#..."', '[brand]', '[vibe', 'fontFamily: "..."']

for token in forbidden:

    if token in DESIGN_CONTENT:

        raise RuntimeError(f"Placeholder '{token}' presente. Volte pra Fase 2: ABRA imagens com view e PREENCHA. Não copie exemplo.")

if BRAND_SLUG in ("cliente", "", None):

    raise RuntimeError(f"BRAND_SLUG ainda é '{BRAND_SLUG}'. Defina algo descritivo (ex: 'acme', 'natura').")

# === PARSE ===

m_fm = re.match(r'^---\s*\n(.*?)\n---\s*\n?(.*)$', DESIGN_CONTENT, re.DOTALL)

if not m_fm:

    raise RuntimeError("DESIGN_CONTENT mal formatado: faltam delimitadores '---'.")

fm = yaml.safe_load(m_fm.group(1)) or {}

prose_text = m_fm.group(2)

# === ANTI-HARDCODE ===

hardcoded = []

for cname, cdef in (fm.get("components") or {}).items():

    if not isinstance(cdef, dict): continue

    for prop, val in cdef.items():

        if isinstance(val, str) and val.startswith("#") and len(val) in (4, 7, 9):

            hardcoded.append(f"{cname}.{prop}={val}")

if hardcoded:

    raise RuntimeError("Components com cores hardcoded em vez de tokens: " + ", ".join(hardcoded) + ". Crie tokens em colors: e referencie via {colors.xxx}.")

# === SALVAR DESIGN.md ===

print("[Fase 3] Validações OK. Salvando DESIGN.md...")

design_path = "/home/user/DESIGN.md"

with open(design_path, "w", encoding="utf-8") as f: f.write(DESIGN_CONTENT)

yaml_str = yaml.dump(fm, allow_unicode=True, sort_keys=False, default_flow_style=False)

name = fm.get("name", BRAND_SLUG.title())

desc = fm.get("description", "")

# === LINT + EXPORTS (fail visible) ===

def cli(args, timeout=180):

    try:

        r = subprocess.run(["npx","-y","@google/design.md"]+args, capture_output=True, text=True, encoding='utf-8', errors='replace', timeout=timeout)

        if r.returncode != 0: print(f"  CLI exit {r.returncode}: {r.stderr[:200]}")

        return r

    except (subprocess.TimeoutExpired, FileNotFoundError) as e:

        print(f"  CLI indisponível: {e}")

        class F: stdout=""; stderr=str(e); returncode=1

        return F()

lint_r = cli(["lint", design_path, "--format", "json"])

try: lint_report = json.loads(lint_r.stdout) if lint_r.stdout else {}

except: lint_report = {}

lint_sum = lint_report.get("summary", {"errors":"?","warnings":"?"})

tw_json = (cli(["export","--format","tailwind", design_path]).stdout or "").strip()

dtcg_json = (cli(["export","--format","dtcg", design_path]).stdout or "").strip()

cli_available = bool(tw_json) or bool(dtcg_json)

print(f"  lint: {lint_sum.get('errors','?')} err, {lint_sum.get('warnings','?')} warn")

if not cli_available: print(f"  Tailwind/DTCG indisponíveis — sinalizado no skill.md")

# === CSS VARS ===

css_lines = []

for k, v in (fm.get("colors") or {}).items():

    if isinstance(v, str): css_lines.append(f"  --color-{k}: {v};")

for k, v in (fm.get("rounded") or {}).items():

    css_lines.append(f"  --rounded-{'base' if k=='DEFAULT' else k}: {v};")

for k, v in (fm.get("spacing") or {}).items():

    css_lines.append(f"  --spacing-{k}: {v};")

css_vars_block = "\n".join(css_lines)

# === PREVIEW HTML ===

print("[Fase 3] Gerando preview HTML...")

src_label = f"Files: {', '.join(p.name for p in files[:4])}" + (f" (+{len(files)-4})" if len(files)>4 else "")

cl, ty, rd, sp = fm.get('colors') or {}, fm.get('typography') or {}, fm.get('rounded') or {}, fm.get('spacing') or {}

bg = cl.get('background') or '#0A0A0A'

srf = cl.get('surface') or bg

srfc = cl.get('surface-container') or srf

onS = cl.get('on-surface') or '#F5F0EB'

prim = cl.get('primary') or '#2DA562'

onPrim = cl.get('on-primary') or '#FFFFFF'

outline = cl.get('outline') or '#3A3A3A'

families = set()

for s in ty.values():

    if isinstance(s, dict) and s.get('fontFamily'):

        families.add(s['fontFamily'].split(',')[0].strip().strip("'\""))

fonts_import = ""

if families:

    fams_url = "&family=".join(f.replace(' ', '+') + ":wght@400;500;600;700" for f in families if f != 'system-ui')

    if fams_url: fonts_import = f"@import url('https://fonts.googleapis.com/css2?family={fams_url}&display=swap');"

ff_main = (list(families)[0] if families else 'Inter')

sws = "".join(f'<div style="border:1px solid {outline};border-radius:8px;overflow:hidden"><div style="height:80px;background:{_h.escape(v)}"></div><div style="padding:10px;font-size:11px;font-family:monospace"><b>{_h.escape(k)}</b><br>{_h.escape(v)}</div></div>' for k,v in cl.items() if isinstance(v,str))

tys = "".join(f'<div style="margin-bottom:24px;padding-bottom:24px;border-bottom:1px solid {outline}"><div style="font-size:11px;opacity:.5;font-family:monospace;margin-bottom:8px">{_h.escape(k)} · {s.get("fontSize","?")} · {s.get("fontWeight","?")}</div><div style="font-family:\'{s.get("fontFamily","Inter")}\',sans-serif;font-size:{s.get("fontSize","16px")};font-weight:{s.get("fontWeight","400")};line-height:{s.get("lineHeight","1.4")};letter-spacing:{s.get("letterSpacing","normal")}">The quick brown fox jumps over the lazy dog</div></div>' for k,s in ty.items() if isinstance(s, dict))

comp_demo = f'''<div style="background:{bg};color:{onS};padding:32px;border-radius:16px;border:1px solid #E5E5E5;font-family:'{ff_main}',system-ui,sans-serif"><div style="display:flex;gap:12px;flex-wrap:wrap;margin-bottom:24px"><button style="background:{prim};color:{onPrim};border:none;padding:12px 24px;border-radius:{rd.get('full','9999px')};font-family:inherit;font-size:14px;font-weight:500;cursor:pointer">Primary CTA</button><button style="background:transparent;color:{onS};border:1px solid {outline};padding:12px 24px;border-radius:{rd.get('full','9999px')};font-family:inherit;font-size:14px;font-weight:500;cursor:pointer">Secondary</button></div><div style="background:{srfc};padding:24px;border-radius:{rd.get('lg','16px')};max-width:480px;margin-bottom:24px"><div style="font-size:11px;text-transform:uppercase;letter-spacing:.12em;opacity:.5;margin-bottom:8px">CARD EXAMPLE</div><h3 style="font-family:inherit;font-size:24px;font-weight:600;margin:0 0 8px 0">Componente card aplicado</h3><p style="margin:0;opacity:.7;line-height:1.5">Tokens em uso: surface-container, padding, radius, tipografia.</p></div><input type="text" placeholder="Input field" style="background:{srf};border:1px solid {outline};color:{onS};padding:12px 16px;border-radius:{rd.get('md','8px')};font-family:inherit;font-size:14px;width:100%;max-width:480px;box-sizing:border-box" /></div>'''

cli_warn = f'<div style="background:#3a2a00;border:1px solid #6a5a00;color:#ffe79e;padding:12px 16px;border-radius:8px;margin-bottom:24px;font-size:13px">Tailwind/DTCG não disponíveis nesta sessão. Re-execute com @google/design.md acessível.</div>' if not cli_available else ""

preview_html = f'''<!DOCTYPE html><html><head><meta charset="UTF-8"><title>{_h.escape(name)} · Design System</title><style>{fonts_import}*{{margin:0;padding:0;box-sizing:border-box}}body{{background:#FAFAFA;color:#1A1A1A;font-family:'{ff_main}',system-ui,sans-serif;padding:48px;min-height:100vh;line-height:1.5}}h1{{font-size:56px;font-weight:600;margin:0 0 12px;letter-spacing:-0.02em}}h2{{opacity:.5;font-size:11px;text-transform:uppercase;letter-spacing:.14em;margin:48px 0 16px;font-weight:500}}.meta{{opacity:.6;margin-bottom:24px;font-size:14px}}.grid{{display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:12px}}.prose{{max-width:680px;opacity:.8;margin-bottom:24px}}code{{font-family:monospace;background:#F0F0F0;padding:2px 6px;border-radius:4px;font-size:13px}}</style></head><body><h1>{_h.escape(name)}</h1><p class="meta">{_h.escape(src_label)} · Lint {lint_sum.get("errors","?")}/{lint_sum.get("warnings","?")} err/warn</p><p class="prose">{_h.escape(desc)}</p>{cli_warn}<h2>Palette · {len(cl)} colors</h2><div class="grid">{sws}</div><h2>Typography · {len(ty)} scales</h2>{tys}<h2>Components in action</h2>{comp_demo}<h2>Como usar</h2><div class="prose">Baixe <code>design-{BRAND_SLUG}.skill.md</code> e cole em outros chats como contexto. A IA gera PDFs, slides, imagens ou UI seguindo essa identidade.</div></body></html>'''

preview_path = f"/home/user/design-{BRAND_SLUG}.preview.html"

with open(preview_path,"w",encoding="utf-8") as f: f.write(preview_html)

# === SKILL.MD PORTÁTIL ===

print("[Fase 3] Empacotando skill portátil...")

today = time.strftime("%Y-%m-%d")

TB = chr(96) * 3

src_desc = f"{len(files)} arquivo(s) ({', '.join(p.name for p in files[:3])}{'...' if len(files)>3 else ''})"

parts = [

    "---",

    f"name: design-{BRAND_SLUG}",

    f'description: Design system "{name}" extraído de {src_desc}. Use como fonte de verdade pra UI, slides, PDFs, imagens. Ative em "use design {BRAND_SLUG}", "estilo {name}", "no padrão {BRAND_SLUG}".',

    f"source: files · {len(files)} arquivo(s)",

    f"extracted_at: {today}",

    "---", "",

    f"# Design System · {name}", "",

    desc or f"Extraído de {src_desc} via Adapta ONE.", "",

    "## Como usar", "",

    "Você é uma IA gerando conteúdo visual. Use os tokens abaixo como fonte de verdade. Não invente cor, tipografia ou espaçamento. Aplique literalmente em PDFs, slides, imagens, código UI ou qualquer artefato visual que gerar.", "",

    "## Tokens", "",

    TB+"yaml", yaml_str.strip(), TB, "",

    "## Rationale", "", prose_text.strip(), ""

]

if tw_json: parts += ["## Tailwind config", "", TB+"json", tw_json, TB, ""]

if dtcg_json: parts += ["## DTCG (W3C standard)", "", TB+"json", dtcg_json, TB, ""]

parts += [

    "## CSS Variables", "",

    TB+"css", ":root {", css_vars_block, "}", TB, "",

    "## Lint", "",

    f"- Errors: {lint_sum.get('errors','?')}",

    f"- Warnings: {lint_sum.get('warnings','?')}"

]

if not cli_available:

    parts += ["", "## Aviso", "", "Tailwind/DTCG não disponíveis na extração (CLI offline). Re-execute pra gerar."]

skill_md = "\n".join(parts)

skill_path = f"/home/user/design-{BRAND_SLUG}.skill.md"

with open(skill_path,"w",encoding="utf-8") as f: f.write(skill_md)

# === UPLOAD ===

print("[Fase 3] Upload final...")

skill_r, skill_e = upload_local_file(skill_path)

preview_r, preview_e = upload_local_file(preview_path)

print()

print(f"Skill portátil: {skill_r['s3_url']}" if not skill_e else f"ERRO: {skill_e}")

print(f"Preview visual: {preview_r['s3_url']}" if not preview_e else f"ERRO: {preview_e}")

print(f"\nDesign system extraído. Cole o .skill.md em outros chats pra usar como contexto.")

```

## Output e limites

**Output:** design-{slug}.skill.md + design-{slug}.preview.html (catálogo neutro com seção branded).

**Limites:** PDF 20 págs (8 viram imagens via pdf2image) · 
