#cinematografia-base
Skill auxiliar que fornece a base three.js compartilhada pras skills de vídeo 3D
---

name: cinematografia-base

description: Skill auxiliar que fornece a base three.js compartilhada pras skills de vídeo 3D cinematográfico Adapta. Define BASE_HTML (three.js + post-processing + lighting schemes + PBR materials + mesh helpers + camera choreography + loop + recording) e função render(scene_code, orientation) que monta HTML, faz upload e imprime URL. Invocada via #cinematografia-base de outras skills.

---

# Cinematografia Base

Infraestrutura three.js compartilhada pras skills de vídeo 3D cinematográfico. Expõe 2 coisas na global Python:

1. BASE_HTML — string com HTML completo (three.js 0.160 + addons + imports + renderer setup + helpers + animation loop + recording), com placeholders ___SCENE_BUILDER___, ___W___, ___H___.

2. render(scene_code, orientation) — recebe string JS do scene builder e orientação "horizontal" ou "vertical"), monta HTML, salva, faz upload, imprime URL.

Skills que consomem essa base definem uma string SCENE = r'''function buildScene(scene, THREE, H) {...}''' e chamam render(SCENE, ORIENTATION).

## Helpers expostos no scene builder H)

- H.mat[kind](color) — materials: matte, plastic, metal, glass, neon, clay, paper

- H.mesh(geomKind, material, opts) — geometrias: box, sphere, torus, torus_knot, cylinder, cone, octa, icosa, dodec, plane. opts: {w,h,d,r,tube,rt,rb,shadow,receive}

- H.setupLighting(scheme) — studio | moody | neon_night (ambient + 3-point ou 3 point lights coloridas)

- H.setBackground(kind, col1, col2) — solid | radial_dark | gradient

- H.setupPost({bloom, grain, vignette}) — cada 0 a 1 aprox, 0 desliga

Scene builder retorna:

```js

{ objects: [...], camera: {archetype, distance, elevation, lookAtY, lookAt}, onFrame: (t, gf)=>{} }

```

Camera archetypes: turntable · dolly_reveal · parallax · orbit_drift · static.

## Código

```python

BASE_HTML = r'''<!DOCTYPE html><html><head><meta charset="UTF-8"><style>*{margin:0;padding:0;box-sizing:border-box}body{background:#000;display:flex;align-items:center;justify-content:center;min-height:100vh;font-family:system-ui}canvas{display:block;max-width:100%;height:auto}#dl{position:fixed;bottom:24px;right:24px;background:#fff;color:#000;border:none;border-radius:8px;padding:12px 24px;font-size:15px;font-weight:500;cursor:pointer;z-index:100}#dl:disabled{opacity:.5}</style>

<script type="importmap">{"imports":{"three":"https://unpkg.com/three@0.160.0/build/three.module.js","three/addons/":"https://unpkg.com/three@0.160.0/examples/jsm/"}}</script>

</head><body><canvas id="c" width="___W___" height="___H___"></canvas><button id="dl">Baixar vídeo</button><script type="module">

import * as THREE from 'three';

import {EffectComposer} from 'three/addons/postprocessing/EffectComposer.js';

import {RenderPass} from 'three/addons/postprocessing/RenderPass.js';

import {UnrealBloomPass} from 'three/addons/postprocessing/UnrealBloomPass.js';

import {FilmPass} from 'three/addons/postprocessing/FilmPass.js';

import {OutputPass} from 'three/addons/postprocessing/OutputPass.js';

import {ShaderPass} from 'three/addons/postprocessing/ShaderPass.js';

import {VignetteShader} from 'three/addons/shaders/VignetteShader.js';

import {RoomEnvironment} from 'three/addons/environments/RoomEnvironment.js';

const W=___W___,H=___H___,FPS=30,DUR=10;

const canvas=document.getElementById('c');

const renderer=new THREE.WebGLRenderer({canvas,antialias:true,preserveDrawingBuffer:true,powerPreference:'high-performance'});

renderer.setSize(W,H,false);renderer.setPixelRatio(1);

renderer.outputColorSpace=THREE.SRGBColorSpace;

renderer.toneMapping=THREE.ACESFilmicToneMapping;renderer.toneMappingExposure=1.0;

renderer.shadowMap.enabled=true;renderer.shadowMap.type=THREE.PCFSoftShadowMap;

renderer.physicallyCorrectLights=true;

const scene=new THREE.Scene();

const camera=new THREE.PerspectiveCamera(35,W/H,0.5,5000);camera.position.set(0,0,600);camera.lookAt(0,0,0);

const pmrem=new THREE.PMREMGenerator(renderer);

scene.environment=pmrem.fromScene(new RoomEnvironment(),0.04).texture;

const C=hex=>new THREE.Color(hex);

const mat={

  matte:(c)=>new THREE.MeshLambertMaterial({color:C(c)}),

  plastic:(c)=>new THREE.MeshStandardMaterial({color:C(c),roughness:.55,metalness:.1}),

  metal:(c)=>new THREE.MeshStandardMaterial({color:C(c),roughness:.22,metalness:1}),

  glass:(c)=>new THREE.MeshPhysicalMaterial({color:C(c),roughness:.05,metalness:0,transmission:.95,thickness:.5,ior:1.5,transparent:true}),

  neon:(c)=>{const m=new THREE.MeshBasicMaterial({color:new THREE.Color(c).multiplyScalar(3)});m.toneMapped=false;return m},

  clay:(c)=>new THREE.MeshStandardMaterial({color:C(c),roughness:.85,metalness:0}),

  paper:(c)=>new THREE.MeshLambertMaterial({color:C(c),flatShading:true}),

};

function mesh(kind,material,opts={}){

  const G={box:()=>new THREE.BoxGeometry(opts.w||200,opts.h||200,opts.d||200),sphere:()=>new THREE.SphereGeometry(opts.r||100,48,32),torus:()=>new THREE.TorusGeometry(opts.r||100,opts.tube||30,24,48),torus_knot:()=>new THREE.TorusKnotGeometry(opts.r||80,opts.tube||22,128,16),cylinder:()=>new THREE.CylinderGeometry(opts.rt||80,opts.rb||80,opts.h||200,32),cone:()=>new THREE.ConeGeometry(opts.r||90,opts.h||200,32),octa:()=>new THREE.OctahedronGeometry(opts.r||90,0),icosa:()=>new THREE.IcosahedronGeometry(opts.r||90,0),dodec:()=>new THREE.DodecahedronGeometry(opts.r||90,0),plane:()=>new THREE.PlaneGeometry(opts.w||1000,opts.h||1000)};

  const m=new THREE.Mesh(G[kind](),material);m.castShadow=opts.shadow!==false;m.receiveShadow=!!opts.receive;return m;

}

function mkDir(col,int,x,y,z,shadow=false){const l=new THREE.DirectionalLight(col,int);l.position.set(x,y,z);if(shadow){l.castShadow=true;l.shadow.mapSize.set(1024,1024);l.shadow.camera.near=1;l.shadow.camera.far=1200;l.shadow.camera.left=-400;l.shadow.camera.right=400;l.shadow.camera.top=400;l.shadow.camera.bottom=-400;l.shadow.bias=-0.0005}return l}

function setupLighting(scheme){

  scene.add(new THREE.AmbientLight(0xffffff,scheme==='neon_night'?.05:.25));

  if(scheme==='studio'){scene.add(mkDir(0xffffff,3,200,300,250,true));scene.add(mkDir(0xffffff,.7,-250,120,180));scene.add(mkDir(0xffffff,1.2,0,180,-300))}

  else if(scheme==='moody'){scene.add(mkDir(0xffb877,3.2,260,240,180,true));scene.add(mkDir(0x4a78ff,.9,-280,100,200));scene.add(mkDir(0xffffff,1.4,0,200,-320))}

  else if(scheme==='neon_night'){[[0xff00aa,220,120,180],[0x00ddff,-240,80,160],[0xffdd00,40,200,-260]].forEach(([c,x,y,z])=>{const p=new THREE.PointLight(c,600,1500,2);p.position.set(x,y,z);scene.add(p)})}

}

function gradCanvas(stops,vertical=true){const cvs=document.createElement('canvas');cvs.width=vertical?2:512;cvs.height=vertical?512:2;const ctx=cvs.getContext('2d');const g=vertical?ctx.createLinearGradient(0,0,0,512):ctx.createLinearGradient(0,0,512,0);stops.forEach(([at,col])=>g.addColorStop(at,'#'+new THREE.Color(col).getHexString()));ctx.fillStyle=g;ctx.fillRect(0,0,cvs.width,cvs.height);const t=new THREE.CanvasTexture(cvs);t.colorSpace=THREE.SRGBColorSpace;return t}

function setBackground(kind,col1=0x1a1a1a,col2=0x000000){

  if(kind==='solid'){scene.background=new THREE.Color(col1)}

  else if(kind==='radial_dark'){const cvs=document.createElement('canvas');cvs.width=512;cvs.height=512;const ctx=cvs.getContext('2d');const g=ctx.createRadialGradient(256,256,60,256,256,380);g.addColorStop(0,'#'+new THREE.Color(col1).getHexString());g.addColorStop(1,'#'+new THREE.Color(col2).getHexString());ctx.fillStyle=g;ctx.fillRect(0,0,512,512);const tex=new THREE.CanvasTexture(cvs);tex.colorSpace=THREE.SRGBColorSpace;scene.background=tex}

  else if(kind==='gradient'){scene.background=gradCanvas([[0,col1],[1,col2]],true)}

}

let composer=null;

function setupPost(opts){

  composer=new EffectComposer(renderer);composer.setSize(W,H);

  composer.addPass(new RenderPass(scene,camera));

  if(opts.bloom>0)composer.addPass(new UnrealBloomPass(new THREE.Vector2(W,H),opts.bloom,0.4,0.85));

  if(opts.grain>0)composer.addPass(new FilmPass(opts.grain,false));

  if(opts.vignette>0){const v=new ShaderPass(VignetteShader);v.uniforms.offset.value=.95;v.uniforms.darkness.value=1+opts.vignette*2;composer.addPass(v)}

  composer.addPass(new OutputPass());

}

___SCENE_BUILDER___

const built=buildScene(scene,THREE,{mat,mesh,setupLighting,setBackground,setupPost});

const camPlan=built.camera||{archetype:'static',distance:600};

const ec=t=>1-Math.pow(1-t,3),eio=t=>t<.5?4*t*t*t:1-Math.pow(-2*t+2,3)/2,ei=t=>t*t*t,cl=t=>Math.max(0,Math.min(1,t));

function animateCamera(t){

  const u=t/DUR,d=camPlan.distance||600,el=camPlan.elevation||0;

  if(camPlan.archetype==='turntable'){const a=u*Math.PI*2;camera.position.set(Math.sin(a)*d,el||80,Math.cos(a)*d)}

  else if(camPlan.archetype==='dolly_reveal'){const farD=d*1.8,near=d*.55;const p1=cl(u/.3),p2=cl((u-.3)/.7);const dist=p1<1?THREE.MathUtils.lerp(d,farD,ec(p1)):THREE.MathUtils.lerp(farD,near,eio(p2));camera.position.set(Math.sin(u*0.4)*dist*.15,el||60,dist)}

  else if(camPlan.archetype==='parallax'){camera.position.x=Math.sin(u*Math.PI*2)*d*.25;camera.position.y=Math.sin(u*Math.PI)*d*.08+(el||40);camera.position.z=d}

  else if(camPlan.archetype==='orbit_drift'){const a=u*Math.PI*1.4;camera.position.set(Math.sin(a)*d,(el||60)+Math.sin(u*Math.PI*2)*50,Math.cos(a)*d)}

  else{camera.position.set(0,el,d)}

  camera.lookAt(camPlan.lookAt||new THREE.Vector3(0,camPlan.lookAtY||0,0));

}

let st=null;

function loop(ts){if(st===null)st=ts;let t=(ts-st)/1000;if(t>DUR+.4)st=ts,t=0;t=Math.min(t,DUR);const gf=t<8?1:1-ei(cl((t-8)/2));animateCamera(t);if(built.onFrame)built.onFrame(t,gf);if(composer)composer.render();else renderer.render(scene,camera);requestAnimationFrame(loop)}

requestAnimationFrame(loop);

document.getElementById('dl').addEventListener('click',()=>{

  const b=document.getElementById('dl');b.disabled=true;b.textContent='Gravando...';st=null;

  const s=canvas.captureStream(FPS);

  const mm=MediaRecorder.isTypeSupported('video/webm;codecs=vp9')?'video/webm;codecs=vp9':'video/webm';

  const r=new MediaRecorder(s,{mimeType:mm,videoBitsPerSecond:10e6}),ch=[];

  r.ondataavailable=e=>{if(e.data.size>0)ch.push(e.data)};

  r.onstop=()=>{const bl=new Blob(ch,{type:'video/webm'}),u=URL.createObjectURL(bl),a=document.createElement('a');a.href=u;a.download='adapta-cinematico.webm';a.click();URL.revokeObjectURL(u);b.disabled=false;b.textContent='Baixar vídeo'};

  r.start(100);setTimeout(()=>r.stop(),DUR*1000+400);

});

</script></body></html>'''

def render(scene_code, orientation='horizontal'):

    W,H = (1920,1080) if orientation=='horizontal' else (1080,1920)

    html = BASE_HTML.replace('___SCENE_BUILDER___', scene_code).replace('___W___', str(W)).replace('___H___', str(H))

    with open('/home/user/cinematico.html','w',encoding='utf-8') as f:

        f.write(html)

    result, error = upload_local_file('/home/user/cinematico.html')

    print(f"URL: {result['s3_url']}" if not error else f"ERRO: {error}")

```

## Design System exposto

- **Tone mapping**: ACESFilmic, exposure 1.0. outputColorSpace=SRGBColorSpace, physicallyCorrectLights=true. Tudo pronto no renderer.

- **Materiais** (custo crescente): matte (Lambert) · paper (Lambert flat) · plastic/metal/clay (Standard PBR) · glass (Physical transmission) · neon (Basic sem tone-mapping, multiplica cor x3 pra bloom picar).

- **Lighting** (todas com AmbientLight de base):

  - studio: key branca 3.0 + fill .7 + rim 1.2 — neutro, produto

  - moody: key âmbar 3.2 + fill azul .9 + rim branca 1.4 — dramático, cinema

  - neon_night: só 3 PointLight coloridas (magenta/ciano/amarelo) intensidade 600, distance 1500, decay 2 — sintético, tech

- **Backgrounds**: solid (1 cor) · radial_dark (radial gradient col1→col2) · gradient (vertical 2 cores)

- **Post**: bloom (UnrealBloom, strength 0-1, radius .4, threshold .85) · grain (FilmPass intensidade 0-.3) · vignette (ShaderPass, offset .95, darkness 1+v*2)

- **Shadows**: PCFSoftShadowMap, mapSize 1024², bias -0.0005. Só key light casta. Subject com shadow:true (default), ground com receive:true.

- **Environment**: RoomEnvironment via PMREMGenerator — PBR reflections sem HDRI externa.

## Notas técnicas

- Só expor ao user as skills que consomem essa base. Essa skill é só infraestrutura.

- neon material só brilha se bloom>0. Sem bloom vira flat color oversaturated.

- glass é caro — evitar em cenas com 5+ objetos transmissivos.

- preserveDrawingBuffer: true obrigatório pra recording funcionar consistentemente em WebGL.

- three.js 0.160.0 + addons via unpkg importmap. Versão travada — não atualizar sem testar composer passes.
