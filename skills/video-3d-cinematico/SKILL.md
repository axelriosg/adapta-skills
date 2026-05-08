#video-3d-cinematico
Gera vídeos 3D cinematográficos
---

name: video-3d-cinematico

description: Gera vídeos 3D cinematográficos (não diagramas) com three.js — product shots, cenas abstratas, teasers de produto, storytelling visual. Lighting 3-point, PBR materials, post-processing (bloom/grain/vignette), ACES tone mapping, câmera cinemática. Dispara em pedidos de "vídeo 3D cinematográfico", "product shot 3D", "teaser 3D", "cena 3D de produto", "vídeo 3D estilo cinema", "landing page hero 3D", "abstract 3D mood".

---

# Vídeo 3D Cinematográfico

Skill three.js/WebGL pra cenas 3D com feel de cinema. Diferente da diagrama-em-video-3d (line art wireframe), essa usa PBR materials, lighting 3-point, shadows e post-processing.

**Depende de #cinematografia-base  — carrega BASE_HTML e render(scene_code, orientation). Sempre carrega antes de executar qualquer Tipo.

Arquitetura: **4 archetypes canônicos + Tipo 5 livre**.

## Fluxo

**Detectar archetype**:

- **Tipo 1 PRODUCT_TURNTABLE** — subject no centro, câmera orbita 360° em 10s. Para: lançamento de feature, produto hero.

- **Tipo 2 DOLLY_REVEAL** — pull-back nos primeiros 3s + dolly-in dramático nos 7s finais. Para: teaser, revelação.

- **Tipo 3 FLOATING_PARALLAX** — 3-5 objetos em profundidades diferentes com fog, câmera parallax lateral. Para: storytelling, landing hero.

- **Tipo 4 ABSTRACT_AMBIENT** — 6-10 geometrias pulsando/rotacionando, mood piece. Para: background loop.

- **Tipo 5 LIVRE** — LLM escreve buildScene customizado.

**Configurar parâmetros** (por Tipo): ORIENTATION · LIGHTING · BG · cores · materiais.

**Executar** + responder (sem preâmbulo, sem emoji):

```

[archetype] 3D · 1920x1080 (ou 1080x1920) · 10s

[URL]

Abra, clique Baixar vídeo pra WebM. Converta em cloudconvert.com se precisar MP4.

```

## Tipo 1 — Product Turntable

```python

ORIENTATION = "horizontal"

SUBJECT_KIND = "torus_knot"   # box|sphere|torus|torus_knot|cylinder|cone|octa|icosa|dodec

SUBJECT_MAT = "metal"         # matte|plastic|metal|glass|neon|clay|paper

SUBJECT_COLOR = 0xd4af37

LIGHTING = "studio"           # studio|moody|neon_night

BG = "radial_dark"            # solid|radial_dark|gradient

BG_COLOR_1 = 0x2a2a2a

BG_COLOR_2 = 0x050505

SCENE = f'''

function buildScene(scene, THREE, H) {{

  H.setupLighting("{LIGHTING}");

  H.setBackground("{BG}", 0x{BG_COLOR_1:06x}, 0x{BG_COLOR_2:06x});

  H.setupPost({{bloom:0.35, grain:0.12, vignette:0.35}});

  const subject = H.mesh("{SUBJECT_KIND}", H.mat["{SUBJECT_MAT}"](0x{SUBJECT_COLOR:06x}));

  scene.add(subject);

  const ground = H.mesh("plane", H.mat.clay(0x181818), {{w:2000, h:2000, receive:true, shadow:false}});

  ground.rotation.x = -Math.PI/2; ground.position.y = -160;

  scene.add(ground);

  return {{

    objects: [subject, ground],

    camera: {{archetype:'turntable', distance:520, elevation:80, lookAtY:0}},

    onFrame: (t)=>{{ subject.rotation.y = t*0.15; }}

  }};

}}

'''

render(SCENE, ORIENTATION)

```

## Tipo 2 — Dolly-In Reveal

```python

ORIENTATION = "horizontal"

SUBJECT_KIND = "icosa"

SUBJECT_MAT = "glass"

SUBJECT_COLOR = 0x88ccff

LIGHTING = "moody"

BG = "radial_dark"

BG_COLOR_1 = 0x1a2240

BG_COLOR_2 = 0x000000

SCENE = f'''

function buildScene(scene, THREE, H) {{

  H.setupLighting("{LIGHTING}");

  H.setBackground("{BG}", 0x{BG_COLOR_1:06x}, 0x{BG_COLOR_2:06x});

  H.setupPost({{bloom:0.6, grain:0.2, vignette:0.5}});

  const subject = H.mesh("{SUBJECT_KIND}", H.mat["{SUBJECT_MAT}"](0x{SUBJECT_COLOR:06x}), {{r:120}});

  scene.add(subject);

  return {{

    objects: [subject],

    camera: {{archetype:'dolly_reveal', distance:450, elevation:30}},

    onFrame: (t)=>{{ subject.rotation.x = t*0.2; subject.rotation.y = t*0.3; }}

  }};

}}

'''

render(SCENE, ORIENTATION)

```

## Tipo 3 — Floating Parallax

```python

ORIENTATION = "horizontal"

OBJECTS = [

  {"kind":"octa",       "mat":"metal",   "color":0xffaa44, "pos":(-280,40,-200), "r":85},

  {"kind":"torus_knot", "mat":"plastic", "color":0xff3377, "pos":(60,-20,80),    "r":70, "tube":18},

  {"kind":"sphere",     "mat":"glass",   "color":0x66eeff, "pos":(280,60,-60),   "r":95},

  {"kind":"dodec",      "mat":"clay",    "color":0xeeeeee, "pos":(-100,-80,240), "r":60},

  {"kind":"icosa",      "mat":"neon",    "color":0xff00aa, "pos":(200,-50,380), "r":40},

]

LIGHTING = "moody"

BG = "gradient"

BG_COLOR_1 = 0x2a1040

BG_COLOR_2 = 0x050510

import json

objs_json = json.dumps(OBJECTS, default=lambda o: list(o) if isinstance(o, tuple) else o)

SCENE = f'''

function buildScene(scene, THREE, H) {{

  H.setupLighting("{LIGHTING}");

  H.setBackground("{BG}", 0x{BG_COLOR_1:06x}, 0x{BG_COLOR_2:06x});

  H.setupPost({{bloom:0.45, grain:0.1, vignette:0.4}});

  scene.fog = new THREE.Fog(0x{BG_COLOR_2:06x}, 400, 1400);

  const defs = {objs_json};

  const objs = defs.map((d,i) => {{

    const m = H.mesh(d.kind, H.mat[d.mat](d.color), {{r:d.r, tube:d.tube||20}});

    m.position.set(d.pos[0], d.pos[1], d.pos[2]);

    m.userData.base = m.position.clone();

    m.userData.phase = i * 0.8;

    return m;

  }});

  objs.forEach(o => scene.add(o));

  return {{

    objects: objs,

    camera: {{archetype:'parallax', distance:700, elevation:20}},

    onFrame: (t)=> objs.forEach((o,i) => {{

      o.position.y = o.userData.base.y + Math.sin(t*0.6 + o.userData.phase) * 30;

      o.rotation.y = t*0.15 + i*0.3;

      o.rotation.x = Math.sin(t*0.3 + i) * 0.4;

    }})

  }};

}}

'''

render(SCENE, ORIENTATION)

```

## Tipo 4 — Abstract Ambient

```python

ORIENTATION = "horizontal"

N_OBJECTS = 8

PALETTE = [0xff00aa, 0x00ddff, 0xffdd00, 0xaa00ff]

LIGHTING = "neon_night"

BG = "solid"

BG_COLOR_1 = 0x050008

SCENE = f'''

function buildScene(scene, THREE, H) {{

  H.setupLighting("{LIGHTING}");

  H.setBackground("{BG}", 0x{BG_COLOR_1:06x});

  H.setupPost({{bloom:1.0, grain:0.25, vignette:0.45}});

  const KINDS=['octa','icosa','dodec','torus_knot','torus'];

  const PAL={PALETTE};

  const N={N_OBJECTS};

  const objs=[];

  for(let i=0;i<N;i++){{

    const k=KINDS[i%KINDS.length], c=PAL[i%PAL.length], useNeon=i%3===0;

    const m=H.mesh(k, useNeon?H.mat.neon(c):H.mat.metal(c), {{r:40+Math.random()*30}});

    const a=(i/N)*Math.PI*2, R=320+Math.random()*120;

    m.position.set(Math.cos(a)*R, (Math.random()-.5)*240, Math.sin(a)*R*0.7 - 80);

    m.userData.base=m.position.clone();

    m.userData.axis=new THREE.Vector3(Math.random(),Math.random(),Math.random()).normalize();

    m.userData.speed=0.3+Math.random()*0.5;

    m.userData.phase=i*0.7;

    objs.push(m); scene.add(m);

  }}

  return {{

    objects: objs,

    camera: {{archetype:'orbit_drift', distance:650, elevation:50}},

    onFrame: (t)=> objs.forEach(o=>{{

      o.rotateOnAxis(o.userData.axis, o.userData.speed*0.03);

      const pulse=1+Math.sin(t*0.8+o.userData.phase)*0.12;

      o.scale.setScalar(pulse);

      o.position.y=o.userData.base.y+Math.sin(t*0.5+o.userData.phase)*25;

    }})

  }};

}}

'''

render(SCENE, ORIENTATION)

```

## Tipo 5 — Geração Livre

LLM escreve buildScene(scene, THREE, H) retornando {objects, camera, onFrame}.

**Geometrias** (via H.mesh(kind, mat, opts)): box, sphere, torus, torus_knot, cylinder, cone, octa, icosa, dodec, plane.

**Materiais** (via H.mat[kind](color)): matte, plastic, metal, glass, neon, clay, paper.

**Lighting** (via H.setupLighting(scheme)): studio, moody, neon_night.

**Background** (via H.setBackground(kind, c1, c2)): solid, radial_dark, gradient.

**Post** (via H.setupPost({bloom, grain, vignette})): cada 0-1 aprox.

**Camera archetypes**: turntable | dolly_reveal | parallax | orbit_drift | static. Params: distance, elevation, lookAtY, lookAt.

```python

SCENE = r'''

function buildScene(scene, THREE, H) {

  H.setupLighting("studio");

  H.setBackground("radial_dark", 0x1a1a1a, 0x000000);

  H.setupPost({bloom:0.4, grain:0.1, vignette:0.3});

  // LLM compõe cena aqui

  return { objects:[], camera:{archetype:'turntable', distance:500}, onFrame:(t)=>{} };

}

'''

render(SCENE, "horizontal")

```

## Regras e Limites

- Priorizar Tipos 1-4 (canônicos). Tipo 5 é fallback.

- WebM only → cloudconvert pra MP4. Nunca mencionar "three.js", "post-processing", "ACES", "EffectComposer" ao user.

- 10s fixo, 1920×1080 (horizontal) ou 1080×1920 (vertical), sem áudio.

- neon exige bloom>0, senão vira flat color.

- glass é caro — evitar em cenas com 5+ objetos transmissivos.

- Regenerar é OK.

- Não faz: logos, UI mockups, charts de dados, fotorrealismo full (sem HDRI custom), vídeos >10s, áudio.
