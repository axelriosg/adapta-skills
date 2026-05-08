#vintage-video2
Gera vídeos estilo documentário vintage
---

name: vintage-video

description: Gera vídeos estilo documentário vintage com efeito de filme antigo (grain animado 24fps, scratches, dust, light leak, vignette, gate weave, aberração cromática, fade, split-tone) replicando o exemplo Anthropic Remotion. Slides de retrato, legenda, quote, título, e fullscreen (imagem cobrindo todo o frame). Aceita imagens locais uploadadas pelo user (converte automaticamente pra base64 inline, à prova de CORS). Use em vídeo vintage, documentário, filme antigo, biografia em vídeo, abertura cinematográfica retrô, fotos+legendas pra slideshow vintage, ou quando user sobe uma imagem pedindo pra aplicar efeito vintage.

---

# Vintage Video

Vídeos estilo documentário vintage idêntico ao exemplo Anthropic Remotion. Cream paper, Playfair Display serif, 5 tipos de slide. Shader: grain animado, scratches, dust, light leak, vignette, gate weave, aberração cromática, fade, split-tone.

Replicação 1:1 sem HTML-in-canvas. Canvas 2D + WebGL2 + MediaRecorder. Imagens viram base64 inline (zero dependência de CORS).

## Saída

HTML autocontido em /home/user/vintage.html, upload pra S3, URL pública. User abre, vê preview em loop, clica "Baixar vídeo (WebM)" e pega 1920x1080 VP9.

## Fluxo

### 1. Imagens

Se o user uploadou imagens (caminho local tipo /home/user/uploads/foto.png), o Python converte cada uma pra data URL antes de injetar na timeline. Helper img_to_data_url(path) está incluído.

URL externa também funciona, mas só se o servidor tiver CORS habilitado (Access-Control-Allow-Origin). S3 do Adapta com signed URL não tem CORS — sempre prefira converter localmente. Wikimedia Commons funciona.

### 2. Montar TIMELINE

Tipos de slide:

- {type: "title", title, subtitle, duration}. Card de abertura: título uppercase + subtitle italic.

- {type: "portrait", image_url, name, epithet, duration}. Foto quadrada 420 cover (rosto enquadrado em 55%/30%) + nome uppercase + epíteto italic.

- {type: "caption", image_url, caption, duration}. Imagem horizontal 1280x500 contain sobre fundo escuro + legenda italic.

- {type: "quote", text, attribution, duration}. Quote italic centralizado + atribuição menor.

- {type: "fullscreen", image_url, fit, duration}. Imagem ocupa todo o frame 1920x1080. fit: "contain" (padrão, mostra inteira com letterbox preto se aspect ratio diferente) ou "cover" (preenche tudo, crop centralizado se aspect ratio diferente). Sem texto sobreposto.

duration em frames a 30fps. Padrões: title=60, portrait=90, caption=120, quote=90, fullscreen=120. Transições entre slides são 25 frames slide-from-right easeOutQuint, automáticas.

Limite: 3 a 6 slides.

### 3. Executar no workbench

Cole o bloco Python abaixo, troque a TIMELINE pela sua.

### 4. Responder

```
[Cena curta] · 1920x1080 · 30fps

[URL]

Abra, deixe rodar, clique Baixar vídeo pra exportar WebM.
```

## Design system (inviolável)

Background #eaeaea. Texto #1a1410. Playfair Display. 1920x1080. Shader: grain 0.126, vignette 0.6, warmth 0.28, fade 0.385 (não exponha).

## Exemplo (imagem do user + texto)

```python
TIMELINE = [
    {"type": "title", "title": "Transplante de IA", "subtitle": "manifesto", "duration": 50},
    {"type": "fullscreen", "image_url": img_to_data_url('/home/user/uploads/arte.png'),
     "fit": "contain", "duration": 120},
    {"type": "quote", "text": "Você só vai abrir o ChatGPT uma vez por dia.",
     "attribution": "Transplante de IA", "duration": 90}
]
```

Ritmo: title curto (1.5-2s), fullscreen 4s pra absorver, quote 3s pra ler.

## Aprendizados de produção (adicionados após uso real)

**Imagens do Wikimedia Commons:** nunca use URL direta no browser — bloqueio de CORS. Use a API REST para resolver a URL real e baixe no sandbox como base64:
```python
def get_wm_url(filename, width=1280):
    params = {'action':'query','titles':f'File:{filename}','prop':'imageinfo','iiprop':'url','iiurlwidth':width,'format':'json'}
    r = requests.get("https://commons.wikimedia.org/w/api.php", params=params, headers={'User-Agent':'AdaptaBot/1.0'}, timeout=15)
    pages = r.json().get('query',{}).get('pages',{})
    for p in pages.values():
        info = p.get('imageinfo',[])
        if info: return info[0].get('thumburl') or info[0].get('url')
    return None
```
Para buscar arquivos por tema: `action=query&list=search&srnamespace=6&srsearch=filetype:image <tema>`.

**Áudio com download (WebM + Opus):** o MediaRecorder não captura o elemento `<audio>` por padrão. Para incluir áudio no WebM exportado:
1. Baixe o MP3 no sandbox e injete como `data:audio/mpeg;base64,...` inline no HTML (sem fetch, sem CORS)
2. Use Web Audio API: `AudioContext.decodeAudioData()` → `AudioBufferSourceNode(loop)` → `GainNode` → `MediaStreamDestination`
3. Combine: `new MediaStream([...scr.captureStream(FPS).getVideoTracks(), ...recDest.stream.getAudioTracks()])`
4. WebGL2 context precisa de `{preserveDrawingBuffer: true}` para captureStream funcionar
5. Codec: `video/webm;codecs=vp9,opus` → fallback `vp8,opus` → fallback `video/webm`
6. `recorder.start(500)` — chunks de 500ms mais estáveis
7. NÃO usar `createMediaElementSource(<audio>)` — perde sync ao reiniciar o loop

**Fonte de áudio confiável (CC BY):** incompetech.com (Kevin MacLeod). Baixar no sandbox com `requests.get()` e converter para base64.

## Workbench Python

```python
import json, base64, mimetypes

TITLE = "Vintage"
SLUG = "vintage"

def img_to_data_url(path):
    mime = mimetypes.guess_type(path)[0] or 'image/png'
    with open(path, 'rb') as f:
        b64 = base64.b64encode(f.read()).decode()
    return f'data:{mime};base64,{b64}'

TIMELINE = []  # <- preenche aqui, usando img_to_data_url() pra imagens locais

HTML = r'''<!doctype html><html lang="pt-BR"><head>
<meta charset="utf-8"><title>__TITLE__</title>
<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&display=swap" rel="stylesheet">
<style>
body{margin:0;background:#1a1410;min-height:100vh;font-family:system-ui;color:#eaeaea;display:flex;flex-direction:column;align-items:center;padding:32px;gap:18px}
#s{width:min(94vw,1280px);aspect-ratio:16/9;display:block;background:#000;border-radius:14px;box-shadow:0 24px 80px rgba(0,0,0,.5)}
#t{display:none}
.r{display:flex;gap:14px;align-items:center}
button{background:#eaeaea;color:#1a1410;border:0;padding:12px 24px;border-radius:999px;font-weight:600;cursor:pointer;font-size:14px;font-family:'Playfair Display',serif}
button:disabled{opacity:.5;cursor:wait}
.m{font-size:13px;color:#9a9692;font-family:'Playfair Display',serif;font-style:italic}
</style></head><body>
<canvas id="s" width="1920" height="1080"></canvas><canvas id="t" width="1920" height="1080"></canvas>
<div class="r"><button id="rec">Baixar vídeo (WebM)</button><span class="m" id="st">Vintage · 1920x1080 · 30fps</span></div>
<script>
const TL=__TIMELINE_JSON__;
const FPS=30,W=1920,H=1080,TR=25;
const BG='#eaeaea',INK='#1a1410';
const SF='"Playfair Display","Didot","Bodoni 72","Times New Roman",serif';
let cur=0;
const SL=TL.map((s,i)=>{const d=s.duration||90,st=cur,en=cur+d;cur=en+(i<TL.length-1?TR:0);return{...s,st,en,d}});
const TS=SL.slice(0,-1).map((s,i)=>({st:s.en,en:s.en+TR,fr:i,to:i+1}));
const TF=cur;
const IMG={};
async function pre(){const us=[...new Set(TL.filter(s=>s.image_url).map(s=>s.image_url))];
await Promise.all(us.map(u=>new Promise(r=>{const i=new Image();i.crossOrigin='anonymous';
i.onload=()=>{IMG[u]=i;r()};i.onerror=()=>{IMG[u]=null;r()};i.src=u})))}
const term=document.getElementById('t'),tx=term.getContext('2d');
const ease=t=>1-Math.pow(1-t,5);
function dimg(img,cx,cy,w,h,opx,opy){if(!img){tx.fillStyle='#999';tx.fillRect(cx-w/2,cy-h/2,w,h);return}
const ir=img.width/img.height,tr=w/h;let sx,sy,sw,sh;
if(ir>tr){sh=img.height;sw=img.height*tr;sy=0;sx=(img.width-sw)*opx}
else{sw=img.width;sh=img.width/tr;sx=0;sy=(img.height-sh)*opy}
tx.drawImage(img,sx,sy,sw,sh,cx-w/2,cy-h/2,w,h)}
function dimgC(img,cx,cy,mw,mh){tx.fillStyle='#0a0806';tx.fillRect(cx-mw/2,cy-mh/2,mw,mh);
if(!img)return;const r=Math.min(mw/img.width,mh/img.height);const w=img.width*r,h=img.height*r;
tx.drawImage(img,cx-w/2,cy-h/2,w,h)}
function dimgF(img,fit){if(!img){tx.fillStyle='#0a0806';tx.fillRect(0,0,W,H);return}
const ir=img.width/img.height,tr=W/H;let sx,sy,sw,sh,dx,dy,dw,dh;
if(fit==='cover'){if(ir>tr){sh=img.height;sw=img.height*tr;sy=0;sx=(img.width-sw)/2}
else{sw=img.width;sh=img.width/tr;sx=0;sy=(img.height-sh)/2}
dx=0;dy=0;dw=W;dh=H;tx.drawImage(img,sx,sy,sw,sh,dx,dy,dw,dh)}
else{tx.fillStyle='#0a0806';tx.fillRect(0,0,W,H);
const r=Math.min(W/img.width,H/img.height);dw=img.width*r;dh=img.height*r;
dx=(W-dw)/2;dy=(H-dh)/2;tx.drawImage(img,dx,dy,dw,dh)}}
function dtxt(t,cx,y,sz,wt,it,op,ls){tx.fillStyle=INK;tx.globalAlpha=op;
tx.font=(it?'italic ':'')+wt+' '+sz+'px '+SF;tx.textBaseline='middle';
if(ls){let tw=0;for(const c of t)tw+=tx.measureText(c).width+ls;tw-=ls;
let x=cx-tw/2;tx.textAlign='left';for(const c of t){tx.fillText(c,x,y);x+=tx.measureText(c).width+ls}}
else{tx.textAlign='center';tx.fillText(t,cx,y)}tx.globalAlpha=1}
function dslide(s,ox){const cx=W/2+ox;
if(s.type==='fullscreen'){tx.save();if(ox)tx.translate(ox,0);
dimgF(IMG[s.image_url],s.fit||'contain');tx.restore();return}
if(s.type==='portrait'){const iy=H/2-100;
dimg(IMG[s.image_url],cx,iy,420,420,.55,.30);
dtxt((s.name||'').toUpperCase(),cx,iy+290,64,'700',false,1,6);
dtxt(s.epithet||'',cx,iy+360,36,'400',true,.75,1)}
else if(s.type==='caption'){const iy=H/2-80;
dimgC(IMG[s.image_url],cx,iy,1280,500);
dtxt(s.caption||'',cx,iy+330,48,'400',true,.78,1.5)}
else if(s.type==='quote'){
dtxt(s.text||'',cx,H/2-30,56,'400',true,1,1);
if(s.attribution)dtxt(s.attribution,cx,H/2+50,28,'400',false,.6,1)}
else if(s.type==='title'){
dtxt((s.title||'').toUpperCase(),cx,H/2-20,84,'700',false,1,8);
if(s.subtitle)dtxt(s.subtitle,cx,H/2+70,38,'400',true,.7,2)}}
function dtl(f){tx.fillStyle=BG;tx.fillRect(0,0,W,H);
for(const s of SL)if(f>=s.st&&f<s.en){dslide(s,0);return}
for(const t of TS)if(f>=t.st&&f<t.en){const p=(f-t.st)/(t.en-t.st),e=ease(p);
dslide(SL[t.fr],-W*e);dslide(SL[t.to],W*(1-e));return}
dslide(SL[SL.length-1],0)}
const scr=document.getElementById('s');
const gl=scr.getContext('webgl2',{alpha:false,antialias:false,premultipliedAlpha:false});
if(!gl)throw Error('no webgl2');
const VS=`#version 300 es
in vec2 a;in vec2 b;out vec2 v;void main(){gl_Position=vec4(a,0,1);v=b;}`;
const FSS=`#version 300 es
precision highp float;
uniform sampler2D u;uniform float t;uniform vec2 r;
uniform float gr,vg,wm,fd;
in vec2 v;out vec4 o;
float h(vec2 p){p=fract(p*vec2(123.34,456.21));p+=dot(p,p+45.32);return fract(p.x*p.y);}
void main(){
vec2 uv=v;
vec2 wv=vec2(sin(t*11.)*.0015+sin(t*3.7)*.0009,cos(t*8.3)*.0013+cos(t*2.4)*.0007);
uv+=wv;
vec2 fc=uv-.5;float l=length(fc);
vec2 d=l>1e-4?normalize(fc):vec2(0);
float ca=.0024*pow(l*1.4,1.6);
vec3 c;
c.r=texture(u,uv+d*ca).r;c.g=texture(u,uv).g;c.b=texture(u,uv-d*ca).b;
float lm=dot(c,vec3(.2126,.7152,.0722));
c=mix(c,vec3(lm),.30);
c=mix(vec3(.5),c,mix(1.,.78,fd));
vec3 sh=vec3(.22,.21,.20),hi=vec3(.98,.97,.96);
c=mix(c,mix(sh,hi,c),wm);
float fi=floor(t*24.);
float gn=h(gl_FragCoord.xy+vec2(fi*17.31,fi*7.91));
c+=(gn-.5)*gr;
float ss=floor(t*8.);
for(int i=0;i<3;i++){float fI=float(i);
float on=step(.78,h(vec2(ss,fI*19.7+3.1)));
float sx=h(vec2(ss,fI*7.13+11.4))*r.x;
float yj=.4*sin(v.y*80.+fI*9.);
float dd=abs(gl_FragCoord.x-sx+yj);
float s=on*smoothstep(2.4,0.,dd)*(.45+.25*h(vec2(ss,fI)));
c+=vec3(s)*.9;}
float ds=floor(t*24.);
for(int i=0;i<4;i++){float fI=float(i);
vec2 dp=vec2(h(vec2(ds,fI*2.13+1.7)),h(vec2(ds,fI*5.71+9.3)));
float on=step(.55,h(vec2(ds,fI*3.33+21.)));
float dr=.004+.006*h(vec2(ds,fI*4.4));
float sp=smoothstep(dr,0.,distance(v,dp));
c+=vec3(.52,.51,.50)*sp*on;}
float lb=.55+.45*sin(t*.7);
float lk=smoothstep(.85,.05,distance(v,vec2(.86,.18)));
c+=vec3(.96,.90,.84)*lk*.14*lb;
float lk2=smoothstep(.7,0.,distance(v,vec2(.12,.85)));
c+=vec3(.88,.86,.84)*lk2*.07*(.5+.5*sin(t*.4+1.7));
float vi=smoothstep(1.05,.30,l*1.25);
c*=mix(1.,vi,vg);
float fl=1.+.035*sin(t*7.)+.018*sin(t*19.+1.3);
c*=fl;
c*=vec3(1.01,1.0,.99);
o=vec4(c,1);}`;
function cmp(s,ty){const sh=gl.createShader(ty);gl.shaderSource(sh,s);gl.compileShader(sh);
if(!gl.getShaderParameter(sh,gl.COMPILE_STATUS))throw Error(gl.getShaderInfoLog(sh));return sh}
const pg=gl.createProgram();
gl.attachShader(pg,cmp(VS,gl.VERTEX_SHADER));gl.attachShader(pg,cmp(FSS,gl.FRAGMENT_SHADER));gl.linkProgram(pg);
const Q=new Float32Array([-1,-1,0,1,1,-1,1,1,-1,1,0,0,1,-1,1,1,-1,1,0,0,1,1,1,0]);
gl.bindBuffer(gl.ARRAY_BUFFER,gl.createBuffer());gl.bufferData(gl.ARRAY_BUFFER,Q,gl.STATIC_DRAW);
const va=gl.createVertexArray();gl.bindVertexArray(va);
const la=gl.getAttribLocation(pg,'a'),lb=gl.getAttribLocation(pg,'b');
gl.enableVertexAttribArray(la);gl.vertexAttribPointer(la,2,gl.FLOAT,false,16,0);
gl.enableVertexAttribArray(lb);gl.vertexAttribPointer(lb,2,gl.FLOAT,false,16,8);
const tex=gl.createTexture();gl.bindTexture(gl.TEXTURE_2D,tex);
[[gl.TEXTURE_MIN_FILTER,gl.LINEAR],[gl.TEXTURE_MAG_FILTER,gl.LINEAR],[gl.TEXTURE_WRAP_S,gl.CLAMP_TO_EDGE],[gl.TEXTURE_WRAP_T,gl.CLAMP_TO_EDGE]].forEach(([k,v])=>gl.texParameteri(gl.TEXTURE_2D,k,v));
const U={};['u','t','r','gr','vg','wm','fd'].forEach(n=>U[n]=gl.getUniformLocation(pg,n));
function paint(f){try{dtl(f)}catch(e){console.error(e);return}
gl.viewport(0,0,W,H);gl.clearColor(0,0,0,1);gl.clear(gl.COLOR_BUFFER_BIT);gl.useProgram(pg);
gl.activeTexture(gl.TEXTURE0);gl.bindTexture(gl.TEXTURE_2D,tex);
gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,gl.RGBA,gl.UNSIGNED_BYTE,term);
gl.uniform1i(U.u,0);gl.uniform1f(U.t,f/FPS);gl.uniform2f(U.r,W,H);
gl.uniform1f(U.gr,__GRAIN__);gl.uniform1f(U.vg,__VIG__);
gl.uniform1f(U.wm,__WARM__);gl.uniform1f(U.fd,__FADE__);
gl.bindVertexArray(va);gl.drawArrays(gl.TRIANGLES,0,6)}
let frm=0,st0=null,rec=false,rcd=null,chk=[],pa=false;
function lp(t){if(!pa){if(st0===null)st0=t;const e=(t-st0)/1000;frm=Math.floor(e*FPS);
if(frm>=TF){if(rec){rcd.stop();rec=false;pa=true}else{st0=t;frm=0}}paint(frm)}requestAnimationFrame(lp)}
const wf=document.fonts&&document.fonts.ready?document.fonts.ready:new Promise(r=>setTimeout(r,600));
Promise.all([pre(),wf]).then(()=>requestAnimationFrame(lp));
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
    .replace('__TITLE__', TITLE).replace('__SLUG__', SLUG)
    .replace('__TIMELINE_JSON__', json.dumps(TIMELINE, ensure_ascii=False))
    .replace('__GRAIN__', '0.126').replace('__VIG__', '0.6')
    .replace('__WARM__', '0.28').replace('__FADE__', '0.385'))

with open('/home/user/vintage.html', 'w', encoding='utf-8') as f:
    f.write(out)

result, error = upload_local_file('/home/user/vintage.html')
if error:
    print(f"STATUS:ERRO|{error}")
else:
    print(f"STATUS:OK")
    print(f"URL:{result['s3_url']}")
```

## Regras

- Imagens locais sempre via img_to_data_url(path). Cada imagem adiciona ~30% do tamanho do arquivo no HTML, então prefira PNGs <500KB ou JPEGs.
- 3 a 6 slides. Mais que isso vira cansativo.
- Nomes em portrait viram uppercase automaticamente.
- Quote/title sem attr/subtitle é válido (omita o campo).
- Caption usa fundo escuro #0a0806 atrás da imagem pra letterboxing.
- Fullscreen fit: "contain" (default) preserva a imagem inteira com letterbox preto. fit: "cover" preenche tudo cortando o que não couber.
- Transição é fixa: slide-from-right de 25 frames easeOutQuint.
- WebM é VP9 8Mbps. Pra MP4, cloudconvert.com.

## Limites

- Sem áudio. Som de projetor 16mm precisa de outra skill.
- 16:9 only (1920x1080). Vertical/quadrado precisa de outra skill.
- Imagens >2MB deixam HTML pesado. Reescale com PIL antes.
- Não muda design system (paper, serif, parâmetros do shader).
