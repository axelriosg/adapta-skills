#crt-terminal-video
Gera vídeos animados de terminal com efeito CRT 
---
name: crt-terminal-video
description: Gera vídeos animados de terminal com efeito CRT (background paper claro, scanlines, barrel distortion, aberração cromática, vinheta) replicando o estilo do vídeo de demo do Claude Code feito pela Anthropic com Remotion. Use quando Axel pedir vídeo de terminal, terminal animado, demo de comandos com CRT, vídeo de instalação de pacote, video estilo Anthropic Claude Code, terminal demo retro, ou enviar uma sequência de comandos/outputs querendo virar vídeo. Também dispara em onboarding técnico em vídeo, cena de instalação, ou workflow de CLI demonstrado animado.
---
# CRT Terminal Video
Gera vídeo animado de terminal com efeito CRT idêntico ao da demo Anthropic Claude Code. Background paper, prompts coloridos, banner Claude Code, e shader CRT por cima (curvatura de tubo, scanlines, aberração cromática, máscara de fósforo, vinheta, flicker).
Replicação 1:1 do visual sem depender de HTML-in-canvas (Chrome Canary 149+ exclusivo). Usa Canvas 2D pra renderizar o terminal, WebGL2 pro shader, MediaRecorder pra exportar. Roda em qualquer Chrome estável.
## Saída
HTML autocontido em /home/user/crt-terminal.html, upload pra S3, URL pública. Usuário abre, vê preview animado em loop, clica "Baixar vídeo (WebM)" e pega 1920x1080 VP9. Pra Instagram/WhatsApp, converte em cloudconvert.com.
## Fluxo
### 1. Entender o pedido
**Modo A. Sequência literal**: Axel cola comandos+outputs já formatados. Mapeie cada linha pra um step.
**Modo B. Pedido conceitual**: Axel diz "video de instalação do nosso SDK" ou "demo da CLI X". Invente sequência plausível com 3 a 6 comandos, mantendo o ritmo do exemplo.
Ambíguo demais (ex "faz um vídeo de terminal aí")? Use ask_user_input_v0 perguntando "Qual a sequência de comandos?".
### 2. Montar TIMELINE
Cada item é um step. Tipos:
- {type: "cmd", cwd, text, cps, holdAfter}. Prompt + comando digitado char por char. cwd: "home" vira ~, "project" vira ~/my-video, ou string literal. cps 1.5 a 3.0 fica natural. holdAfter em frames a 30fps.
- {type: "output", prefix, symbol, segments, indent, holdAfter}. Output do comando. prefix=success põe ✔ verde, error põe ✗ laranja, tag põe palavra em verde tipo "added"/"found". segments=[["cor","texto"], ...] mistura cores. Cores canônicas: #262626 ink, #5a5a56 dim, #2d7a3d green, #c2541a orange.
- {type: "claude_prompt", text, cps}. Prompt Claude Code com >  laranja seguido de texto digitado. Use depois de banner.
- {type: "banner", title, lines, holdAfter}. Caixa laranja estilo banner Claude Code. lines aceita strings ou arrays de segments.
- {type: "gap", h}. Espaço vertical extra.
Limite: 8 a 14 steps. Mais que isso estoura altura.
### 3. Executar no workbench
Cole o bloco Python abaixo, troque a TIMELINE pela sua, execute.
### 4. Responder
```
[Cena curta] · 1920x1080 · 30fps
[URL]
Abra, deixe rodar uma vez, clique Baixar vídeo pra exportar WebM.
Pra Instagram/WhatsApp converta em cloudconvert.com pra MP4.
```
Sem preâmbulo, sem mencionar Canvas/WebGL/shader/MediaRecorder.
## Design system (inviolável)
Background #ecebe6. Ink #262626. Dim #5a5a56. Prompt blue #2a5a8f. Success #2d7a3d. Orange #c2541a. 1920x1080. SF Mono / Menlo / Consolas. Curvatura 6.0/5.5, scanlines 0.08, vinheta 0.4 (fixos no Python, não exponha pro user).
## Exemplo de timeline (referência da demo Anthropic original)
```python
TIMELINE = [
    {"type":"cmd","cwd":"home","text":"npx create-video@latest --yes --blank my-video","cps":1.5,"holdAfter":6},
    {"type":"output","prefix":"success","segments":[["#5a5a56","Copied template to ./my-video"]],"holdAfter":12},
    {"type":"output","prefix":"success","segments":[["#5a5a56","Installed dependencies (47 packages)"]],"holdAfter":16},
    {"type":"output","indent":True,"segments":[["#5a5a56","Run "],["#262626","cd my-video"],["#5a5a56"," && "],["#262626","npx remotion studio"]],"holdAfter":6},
    {"type":"cmd","cwd":"home","text":"cd my-video","cps":2.4,"holdAfter":4},
    {"type":"cmd","cwd":"project","text":"npm i","cps":2.8,"holdAfter":6},
    {"type":"output","prefix":"tag","symbol":"added","segments":[["#5a5a56","623 packages, and audited 624 packages in 27s"]],"holdAfter":8},
    {"type":"output","indent":True,"segments":[["#5a5a56","214 packages are looking for funding · run "],["#262626","npm fund"],["#5a5a56"," for details"]],"holdAfter":8},
    {"type":"output","prefix":"tag","symbol":"found","segments":[["#5a5a56","0 vulnerabilities"]],"holdAfter":12},
    {"type":"cmd","cwd":"project","text":"claude","cps":0.5,"holdAfter":18},
    {"type":"banner","title":"Welcome to Claude Code","lines":[
        "/help for help, /status for your current setup",
        [["#5a5a56","cwd: "],["#262626","~/my-video"]]
    ],"holdAfter":10},
    {"type":"claude_prompt","text":"Add a CRT effect using HTML-in-canvas","cps":0.85}
]
```
Use como ancoragem do ritmo: comandos curtos passam mais rápido (cps 2-3), comandos com ênfase passam devagar (0.5-0.85).
## Workbench Python (HTML embutido)
```python
import json
TITLE = "CRT Terminal"
SLUG = "crt-terminal"
TIMELINE = []  # <- preenche com a timeline montada
HTML = r'''<!doctype html><html lang="pt-BR"><head>
<meta charset="utf-8"><title>__TITLE__</title>
<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Instrument+Sans:wght@400;500;600&display=swap" rel="stylesheet">
<style>
body{margin:0;background:#ecebe6;min-height:100vh;font-family:'Instrument Sans',system-ui;color:#262626;display:flex;flex-direction:column;align-items:center;padding:32px;gap:18px}
#s{width:min(94vw,1280px);aspect-ratio:16/9;display:block;background:#000;border-radius:14px;box-shadow:0 24px 80px rgba(0,0,0,.18)}
#t{display:none}
.r{display:flex;gap:14px;align-items:center}
button{background:#262626;color:#ecebe6;border:0;padding:12px 24px;border-radius:999px;font-weight:600;cursor:pointer;font-family:inherit;font-size:14px}
button:disabled{opacity:.5;cursor:wait}
.m{font-size:13px;color:#5a5a56}
</style></head><body>
<canvas id="s" width="1920" height="1080"></canvas><canvas id="t" width="1920" height="1080"></canvas>
<div class="r"><button id="rec">Baixar vídeo (WebM)</button><span class="m" id="st">CRT terminal · 1920x1080 · 30fps</span></div>
<script>
const TL=__TIMELINE_JSON__;
const FPS=30,W=1920,H=1080;
const C={paper:'#ecebe6',ink:'#262626',dim:'#5a5a56',blue:'#2a5a8f',green:'#2d7a3d',orange:'#c2541a'};
const FS_=36,LH=FS_*1.55,PX=130,PY=130;
const FNT${FS_}px "SF Mono","Menlo","Consolas",monospace;
let cur=0;
const ST=TL.map(s=>{const sf=cur;let d=0,h=LH;
if(s.type==='cmd'||s.type==='claude_prompt')d=Math.ceil(s.text.length/(s.cps||1.5));
else if(s.type==='banner'){h=240}
else if(s.type==='gap'){h=s.h||28;d=0}
cur+=d+(s.holdAfter||0);return{...s,sf,h}});
const TF=cur+60;
const term=document.getElementById('t'),tx=term.getContext('2d');
const tp=(t,sf,cps,f)=>f<sf?'':t.slice(0,Math.min(Math.floor((f-sf)*cps),t.length));
const td=(t,sf,cps,f)=>(f-sf)*cps>=t.length;
function dr(runs,x,y){let cx=x;for(const[c,t,w]of runs){tx.fillStyle=c;tx.font=(w?w+' ':'')+FNT;tx.fillText(t,cx,y);cx+=tx.measureText(t).width}return cx}
function dcur(x,y,c,on){if(!on)return;tx.fillStyle=c;tx.fillRect(x+4,y-FS_*.85,FS_*.55,FS_*1.05)}
function dpr(cwd,x,y){const p=cwd==='project'?'~/my-video':cwd==='home'?'~':(cwd||'~');return dr([[C.blue,p,'600'],[C.ink,' $ ']],x,y)}
function dban(x,y,title,lines){const bw=950,bh=200,r=12;tx.strokeStyle=C.orange;tx.lineWidth=2;tx.fillStyle='rgba(250,249,246,.6)';
tx.beginPath();tx.moveTo(x+r,y);tx.arcTo(x+bw,y,x+bw,y+bh,r);tx.arcTo(x+bw,y+bh,x,y+bh,r);tx.arcTo(x,y+bh,x,y,r);tx.arcTo(x,y,x+bw,y,r);tx.closePath();tx.fill();tx.stroke();
dr([[C.orange,'✻ '+(title||'Welcome to Claude Code'),'600']],x+24,y+50);
let ly=y+100;for(const ln of(lines||[])){if(Array.isArray(ln))dr(ln,x+24,ly);else dr([[C.dim,ln]],x+24,ly);ly+=50}}
function drawTerm(f){tx.fillStyle=C.paper;tx.fillRect(0,0,W,H);tx.textBaseline='alphabetic';
const blink=Math.floor(f/15)%2===0;let y=PY+FS_;
for(let i=0;i<ST.length;i++){const s=ST[i];if(f<s.sf)break;
const last=i===ST.length-1||f<ST[i+1].sf;
if(s.type==='cmd'){let cx=dpr(s.cwd,PX,y);const t=tp(s.text,s.sf,s.cps||1.5,f);cx=dr([[C.ink,t]],cx,y);
if(last)dcur(cx,y,C.ink,td(s.text,s.sf,s.cps||1.5,f)?blink:true);y+=LH}
else if(s.type==='claude_prompt'){let cx=dr([[C.orange,'> ','700']],PX,y);const t=tp(s.text,s.sf,s.cps||1.5,f);cx=dr([[C.ink,t]],cx,y);
if(last)dcur(cx,y,C.orange,td(s.text,s.sf,s.cps||1.5,f)?blink:true);y+=LH}
else if(s.type==='output'){const runs=[];
if(s.prefix==='success')runs.push([C.green,(s.symbol||'✔')+' ','700']);
else if(s.prefix==='error')runs.push([C.orange,(s.symbol||'✗')+' ','700']);
else if(s.prefix==='tag')runs.push([C.green,(s.symbol||'added')+' ','700']);
const segs=s.segments||[[C.dim,s.text||'']];for(const sg of segs)runs.push(sg);
dr(runs,PX+(s.indent?28:0),y);y+=LH+6}
else if(s.type==='banner'){dban(PX,y+8,s.title,s.lines);y+=s.h}
else if(s.type==='gap'){y+=s.h}}}
const scr=document.getElementById('s');
const gl=scr.getContext('webgl2',{alpha:false,antialias:false,premultipliedAlpha:false});
if(!gl)throw Error('no webgl2');
const VS=`#version 300 es
in vec2 a;in vec2 b;out vec2 v;void main(){gl_Position=vec4(a,0,1);v=b;}`;
const FSS=`#version 300 es
precision highp float;uniform sampler2D u;uniform float t;uniform vec2 r,k;uniform float n,g;in vec2 v;out vec4 o;
vec2 cv(vec2 q){vec2 c=q*2.-1.;vec2 f=abs(c.yx)/k;c=c+c*f*f;return c*.5+.5;}
void main(){vec2 q=cv(v);if(q.x<0.||q.x>1.||q.y<0.||q.y>1.){o=vec4(0,0,0,1);return;}
vec2 fc=q-.5;float l=length(fc);
float ca=.0018*pow(l*2.,2.);vec2 d=l>1e-4?normalize(fc):vec2(0);
vec3 c;c.r=texture(u,q+d*ca).r;c.g=texture(u,q).g;c.b=texture(u,q-d*ca).b;
c*=mix(1.,.5+.5*sin(q.y*r.y),n);
float p=mod(gl_FragCoord.x,3.);vec3 m=p<1.?vec3(1.04,.97,.97):p<2.?vec3(.97,1.04,.97):vec3(.97,.97,1.04);
c*=mix(vec3(1),m,.18);c*=mix(1.,smoothstep(.95,.45,l),g);
c*=1.+.012*sin(t*60.);c-=vec3(.04)*smoothstep(.62,.78,l);o=vec4(c,1);}`;
function cmp(s,ty){const sh=gl.createShader(ty);gl.shaderSource(sh,s);gl.compileShader(sh);
if(!gl.getShaderParameter(sh,gl.COMPILE_STATUS))throw Error(gl.getShaderInfoLog(sh));return sh}
const pg=gl.createProgram();gl.attachShader(pg,cmp(VS,gl.VERTEX_SHADER));gl.attachShader(pg,cmp(FSS,gl.FRAGMENT_SHADER));gl.linkProgram(pg);
const Q=new Float32Array([-1,-1,0,1,1,-1,1,1,-1,1,0,0,1,-1,1,1,-1,1,0,0,1,1,1,0]);
gl.bindBuffer(gl.ARRAY_BUFFER,gl.createBuffer());gl.bufferData(gl.ARRAY_BUFFER,Q,gl.STATIC_DRAW);
const va=gl.createVertexArray();gl.bindVertexArray(va);
const la=gl.getAttribLocation(pg,'a'),lb=gl.getAttribLocation(pg,'b');
gl.enableVertexAttribArray(la);gl.vertexAttribPointer(la,2,gl.FLOAT,false,16,0);
gl.enableVertexAttribArray(lb);gl.vertexAttribPointer(lb,2,gl.FLOAT,false,16,8);
const tex=gl.createTexture();gl.bindTexture(gl.TEXTURE_2D,tex);
[[gl.TEXTURE_MIN_FILTER,gl.LINEAR],[gl.TEXTURE_MAG_FILTER,gl.LINEAR],[gl.TEXTURE_WRAP_S,gl.CLAMP_TO_EDGE],[gl.TEXTURE_WRAP_T,gl.CLAMP_TO_EDGE]].forEach(([k,v])=>gl.texParameteri(gl.TEXTURE_2D,k,v));
const U={};['u','t','r','k','n','g'].forEach(n=>U[n]=gl.getUniformLocation(pg,n));
function paint(f){try{drawTerm(f)}catch(e){console.error(e);return}
gl.viewport(0,0,W,H);gl.clearColor(0,0,0,1);gl.clear(gl.COLOR_BUFFER_BIT);gl.useProgram(pg);
gl.activeTexture(gl.TEXTURE0);gl.bindTexture(gl.TEXTURE_2D,tex);
gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,gl.RGBA,gl.UNSIGNED_BYTE,term);
gl.uniform1i(U.u,0);gl.uniform1f(U.t,f/FPS);gl.uniform2f(U.r,W,H);
gl.uniform2f(U.k,__CURV_X__,__CURV_Y__);gl.uniform1f(U.n,__SCAN__);gl.uniform1f(U.g,__VIG__);
gl.bindVertexArray(va);gl.drawArrays(gl.TRIANGLES,0,6)}
let frm=0,st0=null,rec=false,rcd=null,chk=[],pa=false;
function lp(t){if(!pa){if(st0===null)st0=t;const e=(t-st0)/1000;frm=Math.floor(e*FPS);
if(frm>=TF){if(rec){rcd.stop();rec=false;pa=true}else{st0=t;frm=0}}paint(frm)}requestAnimationFrame(lp)}
const wf=document.fonts&&document.fonts.ready?document.fonts.ready:new Promise(r=>setTimeout(r,600));
wf.then(()=>requestAnimationFrame(lp));
document.getElementById('rec').addEventListener('click',()=>{const b=document.getElementById('rec'),s=document.getElementById('st');
b.disabled=true;s.textContent='Gravando...';pa=false;st0=null;frm=0;
const sm=scr.captureStream(FPS);
const mt=MediaRecorder.isTypeSupported('video/webm;codecs=vp9')?'video/webm;codecs=vp9':'video/webm;codecs=vp8';
rcd=new MediaRecorder(sm,{mimeType:mt,videoBitsPerSecond:8e6});chk=[];
rcd.ondataavailable=e=>e.data.size&&chk.push(e.data);
rcd.onstop=()=>{const bl=new Blob(chk,{type:'video/webm'});const u=URL.createObjectURL(bl);
const a=document.createElement('a');a.href=u;a.download='__SLUG__.webm';a.click();URL.revokeObjectURL(u);
b.disabled=false;s.textContent='Pronto.'};
rec=true;rcd.start(100);setTimeout(()=>rcd.stop(),TF/FPS*1000+400)});
</script></body></html>'''
out = (HTML
    .replace('__TITLE__', TITLE)
    .replace('__SLUG__', SLUG)
    .replace('__TIMELINE_JSON__', json.dumps(TIMELINE, ensure_ascii=False))
    .replace('__CURV_X__', '6.0')
    .replace('__CURV_Y__', '5.5')
    .replace('__SCAN__', '0.08')
    .replace('__VIG__', '0.4'))
with open('/home/user/crt-terminal.html', 'w', encoding='utf-8') as f:
    f.write(out)
result, error = upload_local_file('/home/user/crt-terminal.html')
if error:
    print(f"STATUS:ERRO|{error}")
else:
    print(f"STATUS:OK")
    print(f"URL:{result['s3_url']}")
```
## Regras gerais
- Cores canônicas sempre. Não invente hex.
- 14 steps no máximo. Mais que isso o terminal sai do quadro.
- Banner Claude Code é opcional. Use só quando a cena envolve Claude Code.
- Canvas tem altura útil de ~870px. Cada cmd/output ocupa ~62px. Banner ocupa ~240px.
- Outputs longos podem ser quebrados em segments com cores misturadas pra não virar parede de texto cinza.
- Template já lida com cursor piscando, fontes carregando, gravação WebM. Não customize.
- WebM é VP9 8Mbps 1920x1080. Pra MP4, cloudconvert.com.
## O que NÃO faz
- Não gera áudio nem narração.
- Não gera vídeo vertical ou quadrado (16:9 only).
- Não anima cursor de mouse, scroll, ou janelas. Terminal estático com CRT por cima.
- Não muda design system. Modo escuro ou paleta custom precisa de outra skill.
- Não embute player customizado. Página tem só canvas e botão de download.
