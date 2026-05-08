#diagrama-em-video-ultraaaa
 Gera ilustrações animadas em vídeo (Canvas) cinematográficas com narraçã
---

name: diagrama-em-video

description: Gera ilustrações animadas em vídeo (Canvas) cinematográficas com narração TTS pt-BR, sound design sintético e música acústica de fundo (Karplus-Strong). Geração 100% livre — qualquer layout (hub-spoke, fluxo, ciclo, comparação, antes/depois, grid, timeline, funil, arquitetura). Vídeo 20s 1920x1080. Dispara em "diagrama animado", "ilustração em vídeo", "ilustra como X funciona", "diagrama do X com Y", "vídeo explicativo estilo Anthropic Academy", "diagrama com narração", "diagrama cinematográfico", "modo slide", ou prompt descritivo de ilustração pedindo versão animada.

---

# Diagrama em Vídeo Cinematográfico

Geração 100% livre. Você escolhe o layout, escreve a animação. Música acústica de fundo (Karplus-Strong sintetizado), sound design (whoosh+ding), narração Francisca pt-BR — tudo pré-pronto, só preencher drawScene(ctx, t).

## Fluxo

1. **Decidir layout** (catálogo abaixo) baseado na natureza do conteúdo. Não force tudo em hub-spoke. Cada conceito pede uma forma.

2. **Narração:** VOICE_TEXT ≤35 palavras, ~13s. Silencioso = "". Voz: pt-BR-FranciscaNeural (default) ou pt-BR-AntonioNeural.

3. **Executar:** Pré-processamento + AUDIO_HELPERS + Boilerplate, preenchendo drawScene. Substitua placeholders. O upload acontece no final do template.

4. **Resposta:** após executar, diga: [layout] · 1920x1080 · 20s · narração [voz] · com música\n[URL]\nClique Baixar vídeo. Converta em cloudconvert.com pra MP4.

## Catálogo de Layouts

Escolha o que serve ao conteúdo, NÃO pegue sempre o mesmo:

- **Hub-and-spoke (radial):** 1 central + N satélites em órbita elíptica. Pra ecossistemas, "X conectado com A,B,C".

- **Fluxo horizontal:** N cards em linha, setas pontilhadas direcionais. Pra processos sequenciais (etapa 1 → 2 → 3 → 4). ICMS-SP é exemplo.

- **Fluxo vertical:** cards empilhados, setas verticais. Pra funis ou hierarquias.

- **Ciclo circular fechado:** N nós no perímetro de um círculo, setas curvas entre eles, fecha do último ao primeiro. Pra continual learning, ciclos de feedback.

- **Comparação N-up:** N painéis lado a lado, 1 destacado em verde. Pra X vs Y vs Z.

- **Antes/depois split:** divisória vertical no meio do canvas, lado esquerdo cinza com problema, lado direito verde com solução. Setas conectando.

- **Grid:** matriz NxM de cards iguais. Pra catálogos, taxonomias.

- **Timeline horizontal:** eixo temporal no meio, marcadores acima/abaixo com datas e eventos.

- **Funil:** trapézios empilhados de largo (topo) a estreito (base), com %.

- **Arquitetura em camadas:** retângulos horizontais empilhados, cada camada com sub-elementos. Setas verticais bidirecionais.

- **Tipografia dominante:** texto grande domina (60-120px), elementos secundários servem o texto. Pra frameworks (TAPE, OPERA), conceitos.

## Catálogo de Animações

fade-in (.6s easeOutCubic), scale-pop (.6s easeOutCubic), slide-from-edge (.7s, top/bottom/left/right), stroke-draw (.8s, pra linhas/setas), typewriter (per char ~50ms), bounce-in (.7s easeOutBack), cross-fade (entre estados), pulse (loop sutil em destaques).

**Regras de timing:** stagger mínimo 0.4s entre elementos. Setas só começam a desenhar 0.3s APÓS ambos os endpoints estarem visíveis. Texto entra DEPOIS do container que o contém. Animação principal 100% concluída até t=15s. Fade-out global automático t=18-20s.

## Pré-processamento (gera MP3 + boundaries, executar primeiro)

```python

import asyncio, base64, json, subprocess, time, os

# Snapshot locals immediately to avoid mutation by parallel runs in shared namespace

_VT, _V = VOICE_TEXT, VOICE

boundaries=[]; audio=b''

if _VT:

    subprocess.run(['pip','install','edge-tts','-q'],check=True)

    from edge_tts import Communicate

    chunks=[]

    async def _g(_vt=_VT, _v=_V):

        c=Communicate(_vt,_v,boundary="WordBoundary")

        async for ck in c.stream():

            if ck['type']=='audio': chunks.append(ck['data'])

            elif ck['type']=='WordBoundary': boundaries.append({'w':ck['text'],'t':ck['offset']/1e7})

    asyncio.run(_g())

    audio=b''.join(chunks)

AUDIO_B64=base64.b64encode(audio).decode()

BOUND_JSON=json.dumps(boundaries)

RUN_ID=f"{int(time.time()*1000)}_{os.getpid()}"

```

## Audio Helpers

```python

AUDIO_HELPERS=r'''let AC=null,AC_DST=null,AUD_SRC=null;

function whoosh(){if(!AC)return;try{const dur=.4,sr=AC.sampleRate,buf=AC.createBuffer(1,Math.floor(sr*dur),sr),d=buf.getChannelData(0);for(let i=0;i<d.length;i++)d[i]=(Math.random()*2-1)*Math.pow(1-i/d.length,2);const s=AC.createBufferSource();s.buffer=buf;const f=AC.createBiquadFilter();f.type='lowpass';f.frequency.setValueAtTime(2200,AC.currentTime);f.frequency.exponentialRampToValueAtTime(180,AC.currentTime+dur);const g=AC.createGain();g.gain.value=.10;s.connect(f).connect(g);g.connect(AC_DST);g.connect(AC.destination);s.start()}catch(e){console.warn('whoosh:',e.message)}}

function ding(f){if(!AC)return;try{const o=AC.createOscillator();o.type='sine';o.frequency.value=f||880;const g=AC.createGain(),t=AC.currentTime;g.gain.setValueAtTime(.16,t);g.gain.exponentialRampToValueAtTime(.001,t+.6);o.connect(g);g.connect(AC_DST);g.connect(AC.destination);o.start(t);o.stop(t+.7)}catch(e){console.warn('ding:',e.message)}}

function note(freq,when,dur,gain){if(!AC)return;try{const o=AC.createOscillator();o.type='triangle';o.frequency.value=freq;const g=AC.createGain(),t=AC.currentTime+when;g.gain.setValueAtTime(0,t);g.gain.linearRampToValueAtTime(gain,t+.05);g.gain.exponentialRampToValueAtTime(.001,t+dur);o.connect(g);g.connect(AC_DST);g.connect(AC.destination);o.start(t);o.stop(t+dur+.1)}catch(e){console.warn('note:',e.message)}}

function playMusic(){if(!AC)return;const prog=[[220,261,330,392],[196,247,294,370],[174,220,261,329],[164,196,247,294]];for(let bar=0;bar<4;bar++){const ch=prog[bar%4],when=bar*4;ch.forEach((f,i)=>note(f,when+i*.4,3,.04))}}'''

```

## Boilerplate (preencha drawScene)

```python

EXPLANATION = "Sua frase final aqui."

VOICE_TEXT  = "Sua narração aqui, mencionando os elementos do diagrama."

VOICE       = "pt-BR-FranciscaNeural"

# [executar Pré-processamento + definir AUDIO_HELPERS antes]

html = r'''<!DOCTYPE html><html><head><meta charset="UTF-8"><style>@import url('https://fonts.googleapis.com/css2?family=Instrument+Sans:wght@400;500;600&display=swap');*{margin:0;padding:0;box-sizing:border-box}body{background:#F5F0EB;display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;font-family:'Instrument Sans',system-ui}canvas{display:block;max-width:100%;height:auto}#dl{position:fixed;bottom:24px;right:24px;background:#1A5E3F;color:#F5F0EB;border:none;border-radius:8px;padding:12px 24px;font-family:'Instrument Sans';font-size:15px;font-weight:500;cursor:pointer;z-index:100}#dl:disabled{opacity:.5}</style></head><body><canvas id="c" width="1920" height="1080"></canvas><audio id="a" preload="auto" src="data:audio/mp3;base64,___AUDIO___"></audio><button id="dl">Baixar vídeo</button><script>

const W=1920,H=1080,CX=W/2,CY=H/2,GREEN='#2DA562',DARK='#1A5E3F',NEUT='#2C2C2C',MUTED='#7A7A6E',BG='#F5F0EB',FPS=30,DUR=20,VST=1000,HA=___HASA___,EXP=___EXP___,BOUND=___BOUND___;

function ec(t){return 1-Math.pow(1-t,3)}function ei(t){return t*t*t}function eb(t){const c=1.70158;return 1+(c+1)*Math.pow(t-1,3)+c*Math.pow(t-1,2)}function cl(t){return Math.max(0,Math.min(1,t))}function pr(t,s,e){return cl((t-s)/(e-s))}

function rgba(h,a){const r=parseInt(h.slice(1,3),16),g=parseInt(h.slice(3,5),16),b=parseInt(h.slice(5,7),16);return rgba(${r},${g},${b},${a})}

function rr(c,x,y,w,h,r){c.beginPath();if(c.roundRect)c.roundRect(x,y,w,h,r);else{c.moveTo(x+r,y);c.lineTo(x+w-r,y);c.quadraticCurveTo(x+w,y,x+w,y+r);c.lineTo(x+w,y+h-r);c.quadraticCurveTo(x+w,y+h,x+w-r,y+h);c.lineTo(x+r,y+h);c.quadraticCurveTo(x,y+h,x,y+h-r);c.lineTo(x,y+r);c.quadraticCurveTo(x,y,x+r,y);c.closePath()}}

function arrow(c,x1,y1,x2,y2,prog,curve,dashed){if(prog<=0)return;const mx=(x1+x2)/2+(curve||0)*-(y2-y1)/Math.hypot(x2-x1,y2-y1)*60,my=(y1+y2)/2+(curve||0)*(x2-x1)/Math.hypot(x2-x1,y2-y1)*60;let tl=0;const pts=[];for(let k=0;k<=30;k++){const tt=k/30,bx=(1-tt)*(1-tt)*x1+2*(1-tt)*tt*mx+tt*tt*x2,by=(1-tt)*(1-tt)*y1+2*(1-tt)*tt*my+tt*tt*y2;pts.push({x:bx,y:by});if(k>0)tl+=Math.hypot(pts[k].x-pts[k-1].x,pts[k].y-pts[k-1].y)}c.save();c.strokeStyle=GREEN;c.lineWidth=1.5;c.globalAlpha=.6;if(dashed){c.setLineDash([8,5]);c.lineDashOffset=tl-tl*prog}c.beginPath();c.moveTo(x1,y1);for(let k=1;k<=Math.floor(pts.length*prog);k++)c.lineTo(pts[k].x,pts[k].y);c.stroke();c.setLineDash([]);if(prog>.95){const dx=x2-pts[Math.max(0,pts.length-2)].x,dy=y2-pts[Math.max(0,pts.length-2)].y,n=Math.hypot(dx,dy),tx=dx/n,ty=dy/n,al=10,ah=6;c.fillStyle=GREEN;c.beginPath();c.moveTo(x2,y2);c.lineTo(x2-tx*al+ty*ah,y2-ty*al-tx*ah);c.lineTo(x2-tx*al-ty*ah,y2-ty*al+tx*ah);c.closePath();c.fill()}c.restore()}

function card(c,x,y,w,h,fade,scale,green){c.save();const cx=x+w/2,cy=y+h/2;c.globalAlpha=fade;c.translate(cx,cy);c.scale(scale,scale);c.translate(-cx,-cy);c.strokeStyle=green?GREEN:NEUT;c.lineWidth=2;rr(c,x,y,w,h,12);c.stroke();if(green){c.save();rr(c,x,y,w,h,12);c.clip();c.fillStyle=GREEN;c.fillRect(x,y,w,8);c.restore()}c.restore()}

function txt(c,t,x,y,fs,fw,col,fade,align,maxW){c.save();c.globalAlpha=fade;c.fillStyle=col||DARK;c.font${fw||500} ${fs}px 'Instrument Sans',system-ui;c.textAlign=align||'center';c.textBaseline='middle';if(maxW){const ws=t.split(' '),lines=[];let cur='';for(const w of ws){const tst=cur?cur+' '+w:w;if(c.measureText(tst).width>maxW&&cur){lines.push(cur);cur=w}else cur=tst}if(cur)lines.push(cur);const lh=fs*1.3,total=lines.length*lh;lines.forEach((l,i)=>c.fillText(l,x,y-total/2+lh/2+i*lh))}else c.fillText(t,x,y);c.restore()}

function chip(c,t,x,y,fade){c.save();c.globalAlpha=fade;c.font="500 28px 'Instrument Sans'";const w=c.measureText(t).width+60,h=56;c.fillStyle=rgba(GREEN,.18);rr(c,x-w/2,y-h/2,w,h,28);c.fill();c.fillStyle=DARK;c.textAlign='center';c.textBaseline='middle';c.fillText(t,x,y);c.restore()}

function nrm(s){return s.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'').replace(/[^a-z0-9]/g,'')}

function findWord(label){if(!BOUND.length||!HA||!label)return -1;const ws=label.split(/\s+/).filter(w=>w.length>=2);for(const w of ws){const n=nrm(w);for(const b of BOUND){if(nrm(b.w)===n)return b.t+VST/1000}}return -1}

function pulseFx(t,pt){if(pt<0)return{s:1,g:0};const e=t-pt;if(e<-.1||e>.6)return{s:1,g:0};const p=(e+.1)/.7,w=Math.sin(p*Math.PI);return{s:1+.13*w,g:w*.55}}

function dglow(c,cx,cy,r,a){c.save();c.globalAlpha=a*.8;c.strokeStyle=GREEN;c.lineWidth=4;c.beginPath();c.arc(cx,cy,r,0,2*Math.PI);c.stroke();c.globalAlpha=a*.4;c.lineWidth=12;c.beginPath();c.arc(cx,cy,r+10,0,2*Math.PI);c.stroke();c.restore()}

___HELPERS___

// =============== SUA ANIMAÇÃO AQUI ===============

// Helpers disponíveis: ec ei eb cl pr (easings), rgba, rr (roundRect), arrow, card, txt, chip, dglow, findWord(label), pulseFx, whoosh(), ding(freq), playMusic()

// Cores: GREEN DARK NEUT MUTED BG. Constantes: W H CX CY DUR VST.

// Stagger mínimo 0.4s. Setas só desenham 0.3s depois de ambos endpoints visíveis.

// Animação principal acabar até t=15s. EXP fade global são automáticos.

function drawScene(ctx,t){

  // EXEMPLO removido — gere conforme o conceito pedido

}

// =================================================

const cnv=document.getElementById('c'),ctx=cnv.getContext('2d'),aud=document.getElementById('a');let st=null,played={};

function draw(t){ctx.fillStyle=BG;ctx.fillRect(0,0,W,H);const gf=t<18?1:1-ei(pr(t,18,20));ctx.save();ctx.globalAlpha=gf;drawScene(ctx,t);if(EXP){const ea=ec(pr(t,1.5,2.3)),ed=t<18?0:ei(pr(t,18,19)),eo=ea*(1-ed);if(eo>0){ctx.save();ctx.globalAlpha=eo*.3;ctx.strokeStyle=GREEN;ctx.lineWidth=1;ctx.beginPath();ctx.moveTo(CX-260,950);ctx.lineTo(CX+260,950);ctx.stroke();ctx.restore();txt(ctx,EXP,CX,990,24,400,DARK,eo,'center')}}ctx.restore()}

function loop(ts){if(!st)st=ts;const t=Math.min((ts-st)/1000,DUR);try{draw(t)}catch(e){console.error('draw err:',e.message)}if(t<DUR)requestAnimationFrame(loop);else setTimeout(()=>{st=null;played={};requestAnimationFrame(loop)},400)}

(document.fonts?.ready||Promise.resolve()).then(()=>requestAnimationFrame(loop));

document.getElementById('dl').addEventListener('click',function(){const b=this;if(b.disabled)return;b.disabled=true;b.textContent='Gravando...';st=null;played={};

try{

if(!AC){AC=new (window.AudioContext||window.webkitAudioContext)()}

if(AC.state==='suspended')AC.resume();

AC_DST=AC.createMediaStreamDestination();

const vs=cnv.captureStream(FPS);

if(HA){if(!AUD_SRC){try{AUD_SRC=AC.createMediaElementSource(aud)}catch(e){console.warn('aud src already exists:',e.message)}}if(AUD_SRC){AUD_SRC.connect(AC_DST);AUD_SRC.connect(AC.destination)}}

playMusic();

const ms=new MediaStream([...vs.getVideoTracks(),...AC_DST.stream.getAudioTracks()]);

const mt=MediaRecorder.isTypeSupported('video/webm;codecs=vp8,opus')?'video/webm;codecs=vp8,opus':'video/webm';

const rec=new MediaRecorder(ms,{mimeType:mt,videoBitsPerSecond:3e6,audioBitsPerSecond:128e3}),ch=[];

rec.ondataavailable=e=>{if(e.data.size>0)ch.push(e.data)};

rec.onstop=()=>{try{const bl=new Blob(ch,{type:'video/webm'}),u=URL.createObjectURL(bl),a=document.createElement('a');a.href=u;a.download='adapta-diagrama.webm';a.click();URL.revokeObjectURL(u)}catch(e){console.error('save err:',e.message)}b.disabled=false;b.textContent='Baixar vídeo';if(HA){try{aud.pause();aud.currentTime=0}catch{}}};

if(HA){try{aud.currentTime=0}catch{}}rec.start(100);

if(HA){setTimeout(()=>{aud.play().catch(e=>console.warn('aud play:',e.message))},VST)}

setTimeout(()=>{try{rec.stop()}catch(e){console.warn('rec stop:',e.message)}},20400)

}catch(e){console.error('record err:',e.message);b.disabled=false;b.textContent='Erro: '+e.message.slice(0,30)}});

</script></body></html>'''

html = html.replace('___HELPERS___', AUDIO_HELPERS)

html = html.replace('___EXP___', json.dumps(EXPLANATION) if EXPLANATION else 'null')

html = html.replace('___BOUND___', BOUND_JSON)

html = html.replace('___AUDIO___', AUDIO_B64)

html = html.replace('___HASA___', 'true' if AUDIO_B64 else 'false')

_html_path = f'/home/user/diagrama_{RUN_ID}.html'

with open(_html_path, 'w', encoding='utf-8') as f:

    f.write(html)

result, error = upload_local_file(_html_path)

print(f"URL: {result['s3_url']}" if not error else f"ERRO: {error}")

```

## Design System (rigoroso)

bg #F5F0EB · accent #2DA562 (minoria <25%) · dark #1A5E3F (texto) · neutral #2C2C2C (elementos não destacados) · muted #7A7A6E (subtítulos). Stroke 2px round (1.5px op .6 secundárias, 1px op .3 dividers). Dashed 8,5. Instrument Sans 22px/500 (label médio), 18px/500 (label pequeno), 24-28px/400 (frase), 60-120px/600 (tipografia dominante). Negative space ≥100px borda do canvas, ≥60px entre elementos. **Verde é minoria, só destaca.**

## Regras Anti-Bug (críticas, aprendidas)

1. **Stagger:** mínimo 0.4s entre elementos. Pra fluxos sequenciais (timeline, fluxo H/V, ciclo) use **1-2s** entre elementos pra dar respiro estilo slide. Anima principal acaba até t=15s.

2. **Setas começam SÓ 0.3s depois** de ambos os endpoints estarem 100% visíveis. Senão setas chegam antes de cards = visualmente quebrado.

3. **Elementos NUNCA na mesma posição inicial.** Cada card/nó tem coordenada única desde o frame 1. Fade-in é só opacity, não translação que sobrepõe.

4. **Texto entra DEPOIS do container** que o contém, pelo menos 0.2s atrás.

5. **Containers de mesmo tipo têm tamanho idêntico.** 4 cards no fluxo? Largura igual entre os 4.

6. **Padding interno dos cards ≥20px.**

7. **Texto longo (>2 palavras ou frase) usa wrap:** txt(ctx, "Texto longo aqui", x, y, 18, 500, DARK, 1, 'center', maxWidth). O 10º arg maxWidth faz wrap em múltiplas linhas automaticamente.

8. **Sons (whoosh, ding) só tocam durante gravação**, não no preview do loop. Isso é esperado — AC só inicializa no clique.

9. **VOICE_TEXT mencionando elementos** ativa highlight sync via findWord + pulseFx. Use dglow em torno do CARD (não do marker) com raio = max(card width, height)/2 + 20.

10. **Card deve estar 100% visível ANTES da voz mencioná-lo.** Voz começa em t=1s. Se VOICE_TEXT diz "X" como 2ª palavra (~t=1.5s), o card de X deve entrar em ≤t=1.0s. Validação rápida: card.startT + 0.7 < (palavra_position_em_segundos + VST/1000). Se desalinhado, pulse não dispara porque o card nem existe quando a voz fala.

11. **NUNCA use emojis (💡📚🧠✍ etc) como ícones.** Glyphs de emoji color font são re-rasterizados a cada frame e travam o VP8 encoder, derrubando fps de 26 pra 2-6 nos primeiros segundos da gravação. Use letras/números (01, 02, 03), iniciais (T, A, P, E), símbolos (▸ ●), ou desenhe primitivos SVG-style com paths/circles.

12. **Iterações em drawScene (forEach/for) DEVEM envolver o corpo em try/catch local.** Se o item N quebra, o try/catch do loop principal protege a animação geral, mas dentro de um forEach o erro PARA a iteração — itens N+1, N+2 nunca renderizam. Padrão obrigatório: items.forEach((it,i)=>{try{ /* desenhar item */ }catch(e){console.warn('item',i,e.message)}}).

## Limites

20s, 1920x1080, WebM VP9+Opus. Áudio pt-BR ≤35 palavras. Só WebM (cloudconvert pra MP4). NUNCA mencione "Canvas/MediaRecorder/edge-tts/Karplus" ao user. Regenerar é OK.
