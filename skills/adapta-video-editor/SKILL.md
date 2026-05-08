---
name: adapta-video-editor
description: Edita vídeos diretamente do Google Drive via Composio + ffmpeg. Corta, concatena, comprime, converte formato, extrai áudio, e devolve com link compartilhável na mesma pasta. Pipeline completo Drive in → ffmpeg → Drive out, sem upload manual e sem limite prático de tamanho. Dispara quando o usuário pede para "cortar vídeo do drive", "juntar vídeos", "concatenar clipes", "editar aula", "fazer trailer de aula", "primeiros N segundos de", "comprimir vídeo", "extrair áudio de vídeo", "converter mp4 pra webm", "trocar codec", ou descreve qualquer edição de vídeo que está em pasta do Google Drive.
---

# Adapta Video Editor

Edita vídeos do Google Drive sem sair do ONE. Composio Drive baixa pro sandbox, ffmpeg processa, Composio Drive devolve com link compartilhável.

## Saída

Vídeo editado salvo na mesma pasta do Drive (ou outra que o user pedir), com permissão "anyone with link can view" e webViewLink retornado pro user.

## Operações suportadas

| Operação | ffmpeg base | Quando usa |
|---|---|---|
| Cortar (trim) | `-ss START -i in -t DUR -c copy out` | Pegar trecho específico |
| Concatenar (mesmo codec) | `-f concat -safe 0 -i list.txt -c copy out` | Juntar clipes do mesmo source |
| Concatenar (codecs diferentes) | `-filter_complex concat=n=N:v=1:a=1` | Vídeos heterogêneos |
| Comprimir | `-vcodec libx264 -crf 28 out` | Reduzir tamanho |
| Converter MP4 → WebM | `-c:v libvpx-vp9 -c:a libopus out.webm` | Pra Slack/LinkedIn nativo |
| Converter WebM → MP4 | `-c:v libx264 -c:a aac out.mp4` | Pra Instagram/WhatsApp |
| Extrair áudio | `-vn -acodec mp3 out.mp3` | Só o áudio |
| Remover áudio | `-an -c:v copy out` | Vídeo silencioso |
| Acelerar 2x | `-filter:v "setpts=0.5*PTS" -filter:a "atempo=2"` | Speed up |
| Adicionar fade | `-vf "fade=in:0:30,fade=out:N:30"` | Transições suaves |
| Resize | `-vf scale=1080:-2 -c:a copy` | Reduzir resolução |

Regra de ouro: `-c copy` quando a operação permitir, porque copia bytes sem reencode e roda em segundos mesmo em vídeo de 2GB.

## Fluxo

### Passo 1. Entender o pedido

Identificar:
- **Operação**: cortar, concatenar, comprimir, converter, extrair, etc
- **Origem**: nome da pasta no Drive (ex: "desafio-max", "Aulas IA Vendas")
- **Arquivos**: quais vídeos da pasta (todos, primeiros N, por nome, intervalo)
- **Parâmetros**: tempos de corte, ordem de concat, CRF de compressão, formato alvo
- **Destino**: mesma pasta (default) ou outra

Se faltar info crítica (qual pasta, qual operação, qual ordem), pergunte antes de rodar. Não inventar nome de pasta nem ordem de concat.

### Passo 2. Localizar pasta no Drive

Composio Drive tools (nomes podem variar, use tool_search se precisar):

```python
# GOOGLEDRIVE_SEARCH_FOLDER_BY_NAME
folder = search_folder(name=PASTA_NOME)
folder_id = folder['id']

# GOOGLEDRIVE_LIST_FILES
files = list_files(folder_id=folder_id)
videos = [f for f in files if f['name'].lower().endswith(('.mp4', '.mov', '.webm', '.mkv', '.avi'))]
```

Se a pasta não existir ou tiver mais de uma com mesmo nome, pergunte ao user qual é antes de prosseguir.

### Passo 3. Baixar vídeos pro sandbox

```python
# GOOGLEDRIVE_DOWNLOAD_FILE
paths = []
for v in videos:
    local = f"/home/user/{v['name']}"
    download_file(file_id=v['id'], output_path=local)
    paths.append(local)
```

Para vídeos pesados, antes de baixar verificar tamanho total. Se passar de 1.5GB no agregado, avisar o user que pode estourar o sandbox e oferecer cortar antes de baixar.

### Passo 4. Executar ffmpeg

Sempre rodar via subprocess e checar exit code antes de prosseguir.

**Cortar:**
```python
import subprocess
r = subprocess.run(
    ['ffmpeg', '-y', '-ss', str(START), '-i', input_path, '-t', str(DUR), '-c', 'copy', output_path],
    capture_output=True, text=True
)
assert r.returncode == 0, f"ffmpeg falhou: {r.stderr[-500:]}"
```

**Concatenar (mesmo codec, instantâneo):**
```python
with open('/home/user/concat.txt', 'w') as f:
    for p in paths_to_concat:
        f.write(f"file '{p}'\n")

r = subprocess.run(
    ['ffmpeg', '-y', '-f', 'concat', '-safe', '0', '-i', '/home/user/concat.txt', '-c', 'copy', '/home/user/final.mp4'],
    capture_output=True, text=True
)
```

**Concatenar (codec mismatch, reencode):**
```python
inputs = []
for p in paths_to_concat:
    inputs += ['-i', p]
n = len(paths_to_concat)
mapping = ''.join(f'[{i}:v][{i}:a]' for i in range(n))
filter_str = f'{mapping}concat=n={n}:v=1:a=1[outv][outa]'
r = subprocess.run(
    ['ffmpeg', '-y', *inputs, '-filter_complex', filter_str,
     '-map', '[outv]', '-map', '[outa]', '/home/user/final.mp4'],
    capture_output=True, text=True
)
```

**Comprimir:**
```python
r = subprocess.run(
    ['ffmpeg', '-y', '-i', input_path, '-vcodec', 'libx264', '-crf', '28', '-preset', 'fast',
     '-c:a', 'aac', '-b:a', '128k', output_path],
    capture_output=True, text=True
)
```

CRF 23 = qualidade alta, 28 = balanceado, 32-35 = compressão agressiva.

**Converter pra WebM (Slack/LinkedIn):**
```python
r = subprocess.run(
    ['ffmpeg', '-y', '-i', input_path, '-c:v', 'libvpx-vp9', '-b:v', '2M', '-c:a', 'libopus', output_path],
    capture_output=True, text=True
)
```

**Extrair áudio:**
```python
r = subprocess.run(
    ['ffmpeg', '-y', '-i', input_path, '-vn', '-acodec', 'libmp3lame', '-b:a', '192k', output_path],
    capture_output=True, text=True
)
```

### Passo 5. Upload de volta pro Drive

**Sempre usar resumable upload** porque vídeo edited geralmente passa de 5MB. O fluxo de duas etapas é:

```python
# 1. Sobe pro storage Composio (gera s3key)
result, error = upload_local_file('/home/user/final.mp4')
assert not error, f"Upload local falhou: {error}"
s3_url = result['s3_url']
s3_key = result.get('s3_key') or s3_url.split('/')[-1]

# 2. GOOGLEDRIVE_RESUMABLE_UPLOAD usa o s3key
upload = resumable_upload(
    folder_id=folder_id,
    s3key=s3_key,
    name='final.mp4',
    mime_type='video/mp4'
)
file_id = upload['id']
```

Se o arquivo for < 5MB raro mas possível para clipes curtos comprimidos use GOOGLEDRIVE_UPLOAD_FILE direto.

### Passo 6. Criar link compartilhável

```python
# GOOGLEDRIVE_CREATE_PERMISSION
create_permission(file_id=file_id, role='reader', type='anyone')

# GOOGLEDRIVE_GET_FILE_METADATA
meta = get_file_metadata(file_id=file_id)
link = meta['webViewLink']
```

### Passo 7. Responder

Formato fixo, sem preâmbulo, sem emoji:

```
Edição concluída · [operação resumida] · [tamanho final]

[webViewLink]

Permissão: anyone with link can view.
```

Se rodou múltiplas operações em sequência (ex: trim de 2 vídeos + concat), resumir em uma linha:

```
Edição concluída · trim 30s + concat 2 vídeos · 83.5 MB

https://drive.google.com/file/d/.../view

Permissão: anyone with link can view.
```

## Constraints técnicos

- **Composio Drive upload normal cap em 5MB.** Use resumable acima disso, que é praticamente sempre.
- **Memória do sandbox ~2GB.** Vídeos > 1GB com reencode podem dar OOM. Prefira `-c copy` ou corte primeiro.
- **Timeout ~120s por execução do workbench.** Reencode de vídeo longo estoura. Se precisar comprimir vídeo de 30min, cortar em chunks ou avisar o user.
- **Codec mismatch em concat:** `-c copy` não funciona se vídeos têm codecs diferentes. Cair automaticamente pro `filter_complex` se o concat simples falhar.
- **ffmpeg já vem instalado no sandbox**, não precisa apt-get nem pip install.
- **Nomes de tools Composio são case-sensitive e podem variar** entre revisões. Se uma chamada falhar com "tool not found", rodar `tool_search` com keyword pra achar o nome atual.

## Lógica de fallback automático

Se uma operação falhar, tentar a próxima:

1. **Concat falha com "codec params mismatch"** → cair pro filter_complex reencode
2. **Compressão produz arquivo maior que original** → aumentar CRF (28 → 32 → 35)
3. **Resumable upload falha com timeout** → tentar split em chunks menores ou avisar user
4. **Download falha por permissão** → pedir ao user que verifique se o arquivo está acessível pela conta conectada

## Exemplos de uso

### Exemplo 1: desafio Max (cortar e concatenar)

> "Acessa a pasta `desafio-max` no Drive, corta os primeiros 30 segundos de cada vídeo, concatena na ordem em que apareceram, e sobe `final.mp4` na mesma pasta."

Skill executa: search folder → list files → download N → ffmpeg trim em cada → ffmpeg concat (`-c copy` primeiro, fallback pra filter_complex se mismatch) → resumable upload → create permission → retorna link.

### Exemplo 2: comprimir aula pesada

> "Pega `aula01.mp4` da pasta `IA Vendas`, comprime pra ficar abaixo de 100MB, sobe como `aula01-light.mp4`."

Skill: download → ffmpeg crf 28 → check size → se > 100MB rerun com crf 32 → resumable upload → link.

### Exemplo 3: trailer de curso

> "Da pasta `Masterizando IA`, pega as 3 primeiras aulas, corta 10 segundos do início de cada, junta com fade in/out de 1s entre elas, sobe como `trailer.mp4`."

Skill: list 3 → download → trim 10s cada → reencode com fade filter → concat com filter_complex → upload → link.

### Exemplo 4: converter pra WebM pro Slack

> "Pega `demo-skip.mp4` da pasta `Marketing Skip`, converte pra WebM 1080p pra mandar no Slack."

Skill: download → ffmpeg libvpx-vp9 + libopus + scale → upload `demo-skip.webm` → link.

### Exemplo 5: extrair áudio pra transcrever

> "Pega `entrevista-carlos.mp4` da pasta `IA Vendas`, extrai o áudio em mp3, sobe na mesma pasta."

Skill: download → ffmpeg vn acodec mp3 → upload mp3 → link. Daí o user passa pra Reel em Texto ou outro skill de transcrição.

## Regras

- NUNCA invente nome de pasta. Se user não falar, pergunte.
- NUNCA invente file_id. Sempre search → list pra pegar IDs reais.
- SEMPRE confirme com o user antes do upload final se o arquivo de saída > 200MB.
- SEMPRE use `-y` no ffmpeg pra evitar prompt de overwrite trava.
- SEMPRE use `-c copy` quando a operação permitir.
- SEMPRE valide `r.returncode == 0` antes de prosseguir pro próximo passo.
- SEMPRE crie permissão `anyone with link reader` no arquivo final, a menos que o user peça privado.
- NUNCA tente baixar agregado > 1.5GB de vídeos sem avisar primeiro.
- NUNCA processe vídeo > 30min com reencode sem avisar do risco de timeout.

## Stack mínima validada

- Conexão Composio Google Drive ativa
- ffmpeg no sandbox (instalado por padrão)
- `upload_local_file` disponível (retorna s3_url + s3_key)
- Tools: GOOGLEDRIVE_SEARCH_FOLDER_BY_NAME, GOOGLEDRIVE_LIST_FILES, GOOGLEDRIVE_DOWNLOAD_FILE, GOOGLEDRIVE_RESUMABLE_UPLOAD, GOOGLEDRIVE_CREATE_PERMISSION, GOOGLEDRIVE_GET_FILE_METADATA

## O que esta skill NÃO faz

- Edição de áudio multi-track ou mixing complexo, use DaVinci ou Premiere
- Geração de vídeo do zero, use Diagrama em Vídeo, Vídeo em Texto, Link em Vídeo
- Transcrição embutida, use Reel em Texto antes ou depois
- Vídeos > 1.5GB no agregado, limite do sandbox
- Edição de vídeo fora do Drive, baixe primeiro com yt-dlp ou outro download
- Real-time streaming ou live editing
- Geração de legenda burned-in com IA, use skill separada de transcrição + ffmpeg subtitle filter