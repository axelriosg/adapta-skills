#adapta-aula
Gera micro-aulas 9:16 brand Adapta com voz humana edge-tts
---

name: adapta-aula

description: Gera micro-aulas 9:16 brand Adapta com voz humana edge-tts, captions estilo MrBeast word-level, beats hook/concept/step/cta com ícones Lucide. Pra Reels/TikTok/Shorts educacionais de IA generativa, trailer de curso, teaser, micro-aula conceitual. Use em "explica X em 60s", "trailer do curso Y", "teaser do Skip", "micro-aula sobre Z".

---

# Adapta Micro-aula

Saída: HTML em /home/user/aula.html → S3. User clica Baixar vídeo, pega 1080x1920 VP9+Opus. Pra MP4 (Reels/TikTok), cloudconvert.com.

**IMPORTANTE:** user NÃO pode trocar de aba durante gravação (Chrome pausa rAF). Template avisa.

## Fluxo

Você (Claude) é roteirista. Pra "explica X em 60s", monte TIMELINE de 4 a 7 beats, cada um com narration (1 a 3 frases) e campos visuais. Sem screenshots. Pacing alvo: 50 a 90 palavras totais (25 a 50s) a 2.2 palavras/s.

**Beats:**

- {type:"hook", text, narration} - pergunta gigante centralizada

- {type:"concept", icon, title, desc, narration} - ícone Lucide + título + desc

- {type:"step", n, text, narration} - número gigante verde + texto

- {type:"cta", text, url, narration} - card final URL pílula verde

**Ícones:** search, database, brain, check, sparkles, zap, arrow-right, lightbulb.

**Voz:** VOICE = "pt-BR-FranciscaNeural" (default). Outras: AntonioNeural (M), ThalitaNeural (F jovem), FabioNeural (M formal). Sempre prefixe com pt-BR-.

**Música:** BGM = audio_to_data_url(path) mixa em -15dB. Opcional.

**Resposta após executar:**

```

[Tema] · 1080x1920 · 30fps · voz [voz]

[URL]

Abra e clique Baixar vídeo (NÃO troque de aba).

## Design

BG #0a0a0a. Accent #28C98A. Ink #f5f5f5. Instrument Sans. 1080x1920.

## Workbench Python

```python

import asyncio, base64, json, mimetypes

import edge_tts

TITLE = "Aula"

SLUG = "aula"

def to_data_url(path, default='image/png'):

    mime = mimetypes.guess_type(path)[0] or default

    with open(path, 'rb') as f: b64 = base64.b64encode(f.read()).decode()

    return f'data:{mime};base64,{b64}'

img_to_data_url = to_data_url

audio_to_data_url = lambda p: to_data_url(p, 'audio/mpeg')

ICONS = json.loads(r'''{"search":"<path d=\"m21 21-4.34-4.34\"/><circle cx=\"11\" cy=\"11\" r=\"8\"/>","database":"<ellipse cx=\"12\" cy=\"5\" rx=\"9\" ry=\"3\"/><path d=\"M3 5V19A9 3 0 0 0 21 19V5\"/><path d=\"M3 12A9 3 0 0 0 21 12\"/>","brain":"<path d=\"M12 18V5\"/><path d=\"M15 13a4.17 4.17 0 0 1-3-4 4.17 4.17 0 0 1-3 4\"/><path d=\"M17.598 6.5A3 3 0 1 0 12 5a3 3 0 1 0-5.598 1.5\"/><path d=\"M17.997 5.125a4 4 0 0 1 2.526 5.77\"/><path d=\"M18 18a4 4 0 0 0 2-7.464\"/><path d=\"M19.967 17.483A4 4 0 1 1 12 18a4 4 0 1 1-7.967-.517\"/><path d=\"M6 18a4 4 0 0 1-2-7.464\"/><path d=\"M6.003 5.125a4 4 0 0 0-2.526 5.77\"/>","check":"<path d=\"M20 6 9 17l-5-5\"/>","sparkles":"<path d=\"M11.017 2.814a1 1 0 0 1 1.966 0l1.051 5.558a2 2 0 0 0 1.594 1.594l5.558 1.051a1 1 0 0 1 0 1.966l-5.558 1.051a2 2 0 0 0-1.594 1.594l-1.051 5.558a1 1 0 0 1-1.966 0l-1.051-5.558a2 2 0 0 0-1.594-1.594l-5.558-1.051a1 1 0 0 1 0-1.966l5.558-1.051a2 2 0 0 0 1.594-1.594z\"/><path d=\"M20 2v4\"/><path d=\"M22 4h-4\"/><circle cx=\"4\" cy=\"20\" r=\"2\"/>","zap":"<path d=\"M4 14a1 1 0 0 1-.78-1.63l9.9-10.2a.5.5 0 0 1 .86.46l-1.92 6.02A1 1 0 0 0 13 10h7a1 1 0 0 1 .78 1.63l-9.9 10.2a.5.5 0 0 1-.86-.46l1.92-6.02A1 1 0 0 0 11 14z\"/>","arrow-right":"<path d=\"M5 12h14\"/><path d=\"m12 5 7 7-7 7\"/>","lightbulb":"<path d=\"M15 14c.2-1 .7-1.7 1.5-2.5 1-.9 1.5-2.2 1.5-3.5A6 6 0 0 0 6 8c0 1 .2 2.2 1.5 3.5.7.7 1.3 1.5 1.5 2.5\"/><path d=\"M9 18h6\"/><path d=\"M10 22h4\"/>"}''')

TIMELINE = []

VOICE = "pt-BR-FranciscaNeural"

BGM = None

async def gen_narration(text, voice):

    sm = edge_tts.SubMaker()

    comm = edge_tts.Communicate(text, voice)

    audio = b''

    async for chunk in comm.stream():

        if chunk["type"] == "audio": audio += chunk["data"]

        elif chunk["type"] == "WordBoundary": sm.feed(chunk)

    words = []

    for c in sm.cues:

        t = c.text.strip()

        if not any(ch.isalnum() for ch in t): continue

        words.append({"w": t, "s": c.start.total_seconds(), "e": c.end.total_seconds()})

    return audio, words

# Concatenar narrações de cada beat

full_narr = " . ".join(b['narration'].rstrip('.!?') for b in TIMELINE) + "."

audio_bytes, words = asyncio.run(gen_narration(full_narr, VOICE))

# Mapear ranges por beat baseado em quantas palavras cada narração tem

import re

def count_words(s): return len([w for w in re.findall(r'\S+', s) if any(c.isalnum() for c in w)])

ranges = []; fw = 0

for b in TIMELINE:

    n = count_words(b['narration'])

    lw = min(fw + n - 1, len(words) - 1)

    ranges.append((fw, lw))

    fw = lw + 1

for i, b in enumerate(TIMELINE):

    b['_fw'] = ranges[i][0]

    b['_lw'] = ranges[i][1]

narr_url = f'data:audio/mpeg;base64,{base64.b64encode(audio_bytes).decode()}'

bgm_attr = f'src="{BGM}"' if BGM else ''

HTML = r'''<!doctype html><html lang="pt-BR"><head>

<meta charset="utf-8"><title>__TITLE__</title>

<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<link href="https://fonts.googleapis.com/css2?family=Instrument+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">

<style>

body{margin:0;background:#000;min-height:100vh;font-family:'Instrument Sans',system-ui;color:#eee;display:flex;flex-direction:column;align-items:center;padding:24px;gap:14px}

#s{width:min(50vh,360px);aspect-ratio:9/16;display:block;background:#0a0a0a;border-radius:18px;box-shadow:0 24px 80px rgba(0,0,0,.6)}

.r{display:flex;gap:14px;align-items:center}

button{background:#28C98A;color:#071A10;border:0;padding:12px 24px;border-radius:999px;font-weight:600;cursor:pointer;font-size:14px;font-family:'Instrument Sans',sans-serif}

button:disabled{opacity:.5;cursor:wait}

.m{font-size:13px;color:#7a7a7a}

audio{display:none}

</style></head><body>

<canvas id="s" width="1080" height="1920"></canvas>

<audio id="narr" src="__NARR_URL__"></audio>

<audio id="bgm" loop crossorigin="anonymous" __BGM_ATTR__></audio>

<div class="r"><button id="rec">Baixar vídeo (WebM)</button><span class="m" id="st">Adapta · 1080x1920 · 30fps</span></div>

<script>

const TL=__TIMELINE_JSON__;

const WD=__WORDS_JSON__;

const ICN=__ICONS_JSON__;

const FPS=30,W=1080,H=1920,FA='#28C98A',INK='#f5f5f5',DIM='#8a8a8a';

const F='Instrument Sans, system-ui';

const SL=TL.map(b=>{const fw=b._fw,lw=b._lw;const st=WD[fw]?WD[fw].s:0;const en=WD[lw]?WD[lw].e+.4:st+3;return{...b,st,en}});

const TF=Math.ceil((SL[SL.length-1]?SL[SL.length-1].en:5)*FPS);

const c=document.getElementById('s'),x=c.getContext('2d');

const ease=t=>1-Math.pow(1-t,4);

function rr(rx,ry,rw,rh,r){x.beginPath();x.moveTo(rx+r,ry);x.arcTo(rx+rw,ry,rx+rw,ry+rh,r);x.arcTo(rx+rw,ry+rh,rx,ry+rh,r);x.arcTo(rx,ry+rh,rx,ry,r);x.arcTo(rx,ry,rx+rw,ry,r);x.closePath()}

function gr(){const g=x.createLinearGradient(0,0,0,H);g.addColorStop(0,'#0a0a0a');g.addColorStop(.5,'#0d1410');g.addColorStop(1,'#0a0a0a');x.fillStyle=g;x.fillRect(0,0,W,H)}

function badge(){x.fillStyle=DIM;x.font='500 24px '+F;x.textAlign='center';x.textBaseline='middle';x.fillText('ΔDAPTA',W/2,75);x.fillStyle=FA;x.fillRect(W/2-12,90,24,2)}

function progress(t){const p=t/(TF/FPS);x.fillStyle='#1a1a1a';x.fillRect(60,H-40,W-120,4);x.fillStyle=FA;x.fillRect(60,H-40,(W-120)*p,4)}

function icon(name,cx,cy,sz,col){const sv=ICN[name];if(!sv)return;

x.save();x.translate(cx-sz/2,cy-sz/2);x.scale(sz/24,sz/24);

x.strokeStyle=col;x.lineWidth=2;x.lineCap='round';x.lineJoin='round';

const m=sv.matchAll(/<(\w+)\s+([^/]*)\/>/g);

for(const r of m){const tag=r[1],a={};

const ax=r[2].matchAll(/(\w+(?:-\w+)?)="([^"]*)"/g);for(const n of ax)a[n[1]]=n[2];

x.beginPath();

if(tag==='path')drawPath(a.d);

else if(tag==='circle')x.arc(+a.cx,+a.cy,+a.r,0,Math.PI*2);

else if(tag==='ellipse')x.ellipse(+a.cx,+a.cy,+a.rx,+a.ry,0,0,Math.PI*2);

else if(tag==='line'){x.moveTo(+a.x1,+a.y1);x.lineTo(+a.x2,+a.y2)}

x.stroke()}

x.restore()}

function drawPath(d){const cmds=d.match(/[a-zA-Z][^a-zA-Z]*/g)||[];let cx=0,cy=0;

for(const cm of cmds){const op=cm[0];const nums=(cm.slice(1).match(/-?\d*\.?\d+/g)||[]).map(Number);

if(op==='M'){x.moveTo(nums[0],nums[1]);cx=nums[0];cy=nums[1];for(let i=2;i<nums.length;i+=2){x.lineTo(nums[i],nums[i+1]);cx=nums[i];cy=nums[i+1]}}

else if(op==='m'){x.moveTo(cx+nums[0],cy+nums[1]);cx+=nums[0];cy+=nums[1];for(let i=2;i<nums.length;i+=2){x.lineTo(cx+nums[i],cy+nums[i+1]);cx+=nums[i];cy+=nums[i+1]}}

else if(op==='L'){for(let i=0;i<nums.length;i+=2){x.lineTo(nums[i],nums[i+1]);cx=nums[i];cy=nums[i+1]}}

else if(op==='l'){for(let i=0;i<nums.length;i+=2){x.lineTo(cx+nums[i],cy+nums[i+1]);cx+=nums[i];cy+=nums[i+1]}}

else if(op==='H'){for(const n of nums){x.lineTo(n,cy);cx=n}}

else if(op==='h'){for(const n of nums){x.lineTo(cx+n,cy);cx+=n}}

else if(op==='V'){for(const n of nums){x.lineTo(cx,n);cy=n}}

else if(op==='v'){for(const n of nums){x.lineTo(cx,cy+n);cy+=n}}

else if(op==='Z'||op==='z')x.closePath()

else if(op==='A'||op==='a'){if(nums.length>=7){const ex=op==='a'?cx+nums[5]:nums[5],ey=op==='a'?cy+nums[6]:nums[6];x.lineTo(ex,ey);cx=ex;cy=ey}}}}

function wrap(t,mw,sz,wt){x.font=(wt||'400')+' '+sz+'px '+F;

const ws=t.split(' '),lns=[];let l='';

for(const w of ws){const tst=l?l+' '+w:w;if(x.measureText(tst).width>mw&&l){lns.push(l);l=w}else l=tst}

if(l)lns.push(l);return lns}

function dline(t,xc,yc,sz,wt,col){x.fillStyle=col||INK;x.font=(wt||'400')+' '+sz+'px '+F;x.textAlign='center';x.textBaseline='middle';x.fillText(t,xc,yc)}

function multi(lines,xc,y0,sz,wt,col,lh){for(let i=0;i<lines.length;i++)dline(lines[i],xc,y0+i*sz*(lh||1.15),sz,wt,col)}

function bHook(b,p){const sz=b.text&&b.text.length>20?72:88;const lns=wrap(b.text||'',W-160,sz,'700');

multi(lns,W/2,H/2-200,sz,'700',INK,1.1)}

function bConcept(b,p){const ic=b.icon||'sparkles';const iy=H/2-280;const isz=240;

x.save();x.shadowColor=FA;x.shadowBlur=40;icon(ic,W/2,iy,isz,FA);x.restore();

const ttl=(b.title||b.text||'').toUpperCase();const lns=wrap(ttl,W-120,52,'700');

multi(lns,W/2,iy+200,52,'700',INK,1.15);

if(b.desc){const lns2=wrap(b.desc,W-180,32,'400');multi(lns2,W/2,iy+200+lns.length*60+40,32,'400',DIM,1.3)}}

function bStep(b,p){const n=String(b.n||'?');

const numY=H/2-200;

x.fillStyle=FA;x.font='700 320px '+F;x.textAlign='center';x.textBaseline='middle';

x.fillText(n,W/2,numY);

x.fillStyle=FA;x.fillRect(W/2-100,numY+170,200,3);

const lns=wrap(b.text||'',W-160,42,'500');multi(lns,W/2,numY+260,42,'500',INK,1.25)}

function bCTA(b,p){const lns=wrap(b.text||'Aprenda na Adapta',W-120,56,'700');

multi(lns,W/2,H/2-220,56,'700',INK,1.15);

icon('arrow-right',W/2,H/2-220+lns.length*65+80,80,FA);

const url=b.url||'adapta.org';

x.font='600 32px '+F;const uw=x.measureText(url).width;

rr(W/2-(uw/2+50),H/2+80,uw+100,80,40);x.fillStyle=FA;x.fill();

dline(url,W/2,H/2+120,32,'600','#071A10')}

function captions(t){if(!WD.length)return;

let active=-1;for(let i=0;i<WD.length;i++)if(t>=WD[i].s&&t<=WD[i].e+.05){active=i;break}

if(active<0)for(let i=0;i<WD.length;i++)if(WD[i].s<=t)active=i;

if(active<0)return;

const win=2,lo=Math.max(0,active-win),hi=Math.min(WD.length-1,active+win),cy=H-220;

x.font='700 42px '+F;x.textBaseline='middle';

let totalW=0;for(let i=lo;i<=hi;i++)totalW+=x.measureText(WD[i].w).width+22;totalW-=22;

let xC=W/2-totalW/2;

for(let i=lo;i<=hi;i++){const w=WD[i].w,ww=x.measureText(w).width;

if(i===active){x.fillStyle=FA;rr(xC-12,cy-32,ww+24,64,10);x.fill();x.fillStyle='#071A10'}

else if(i<active){x.fillStyle='rgba(245,245,245,.3)'}

else{x.fillStyle='rgba(245,245,245,.85)'}

x.fillText(w,xC,cy);xC+=ww+22}}

function findBeat(t){for(const b of SL)if(t>=b.st&&t<b.en)return b;return SL[SL.length-1]}

function paint(){const a=document.getElementById('narr'),t=a.currentTime;

gr();badge();

const b=findBeat(t),p=Math.max(0,Math.min(1,(t-b.st)/(b.en-b.st||1)));

if(b.type==='hook')bHook(b,p);

else if(b.type==='concept')bConcept(b,p);

else if(b.type==='step')bStep(b,p);

else if(b.type==='cta')bCTA(b,p);

captions(t);progress(t)}

let rec=false,rcd=null,chk=[],aC=null,aSrc=null,aD=null,bSrc=null;

function loop(){paint();requestAnimationFrame(loop)}

window.addEventListener('load',()=>loop());

document.getElementById('rec').addEventListener('click',async()=>{

const b=document.getElementById('rec'),s=document.getElementById('st');

b.disabled=true;s.textContent='Gravando... NÃO troque de aba';

const narr=document.getElementById('narr'),bgm=document.getElementById('bgm');

narr.currentTime=0;

const sm=c.captureStream(FPS);

let tracks=[...sm.getVideoTracks()];

try{

if(!aC){aC=new(window.AudioContext||window.webkitAudioContext)();aD=aC.createMediaStreamDestination();

aSrc=aC.createMediaElementSource(narr);aSrc.connect(aD);aSrc.connect(aC.destination);

if(bgm&&bgm.src){bSrc=aC.createMediaElementSource(bgm);const g=aC.createGain();g.gain.value=.18;bSrc.connect(g);g.connect(aD);g.connect(aC.destination)}}

if(aC.state==='suspended')await aC.resume();

if(bgm&&bgm.src){bgm.currentTime=0;bgm.play().catch(()=>{})}

narr.play();

tracks=[...tracks,...aD.stream.getAudioTracks()]

}catch(e){console.warn(e)}

const sts=new MediaStream(tracks);

const mt=MediaRecorder.isTypeSupported('video/webm;codecs=vp9,opus')?'video/webm;codecs=vp9,opus':'video/webm';

rcd=new MediaRecorder(sts,{mimeType:mt,videoBitsPerSecond:8e6,audioBitsPerSecond:128e3});chk=[];

rcd.ondataavailable=e=>e.data.size&&chk.push(e.data);

rcd.onstop=()=>{const bl=new Blob(chk,{type:'video/webm'});const u=URL.createObjectURL(bl);

const a=document.createElement('a');a.href=u;a.download='__SLUG__.webm';a.click();URL.revokeObjectURL(u);

b.disabled=false;s.textContent='Pronto.';if(bgm)bgm.pause();narr.pause()};

rec=true;rcd.start(250);

narr.addEventListener('ended',()=>{if(rec){rcd.stop();rec=false}},{once:true});

setTimeout(()=>{if(rec){rcd.stop();rec=false}},(TF/FPS+5)*1000)});

document.addEventListener('visibilitychange',()=>{if(rec&&document.hidden){document.getElementById('st').textContent='⚠️ ABA OCULTA — vídeo vai cortar! Volte agora'}});

</script></body></html>'''

out = (HTML

    .replace('__TITLE__', TITLE).replace('__SLUG__', SLUG)

    .replace('__NARR_URL__', narr_url)

    .replace('__BGM_ATTR__', bgm_attr)

    .replace('__TIMELINE_JSON__', json.dumps(TIMELINE, ensure_ascii=False))

    .replace('__WORDS_JSON__', json.dumps(words, ensure_ascii=False))

    .replace('__ICONS_JSON__', json.dumps(ICONS, ensure_ascii=False)))

with open('/home/user/aula.html', 'w', encoding='utf-8') as f: f.write(out)

result, error = upload_local_file('/home/user/aula.html')

if error: print(f"STATUS:ERRO|{error}")

else: print(f"STATUS:OK\nURL:{result['s3_url']}")

```

## Regras

- 4 a 7 beats. Hook sempre primeiro, CTA sempre último.

- Cada beat tem narration mandatório.

- Use só os 8 ícones disponíveis (escolha o mais próximo se não bater).

- Áudio entra no WebM SE user não trocar de aba.
