#documento-branded
Gera um documento branded com qualquer logo, como link web para imprimir em PDF
---

name: documento-branded

description: Gera um documento branded com o logo de qualquer cliente, como um LINK que o usuário abre e imprime em PDF. Usa o template genérico já hospedado e o LOGO que o usuário subiu no chat do ONE. NÃO usa workbench. Dispara em "cria um documento com meu logo", "gera um PDF branded", "documento com o logo que subi", "faz um documento sobre X com meu logo".

---

# Método

O documento é uma página web já hospedada (o molde genérico). ONE só monta um LINK com a marca e o conteúdo embutidos na URL. O usuário abre o link e imprime em PDF com Ctrl+P. O entregável é o LINK, nunca um arquivo. ONE não usa workbench.

BASE_URL = https://raw.githack.com/axelriosg/pilula-assets/main/documento.html

# Passos

1. LOGO: pegar a URL da imagem que o usuário subiu no chat. Se não houver, pedir que suba o logo primeiro.

2. MARCA: cor em hex (se não informar, usar #333333), e nome e contatos se o usuário der.

3. CONTEÚDO: escrever o título e os blocos do documento conforme o pedido.

4. Montar o JSON:

   {"logo":"URL_DO_LOGO_SUBIDO","color":"#HEX","name":"NOME_OPCIONAL","contacts":["c1","c2"],"title":"TÍTULO","blocks":[{"type":"h","text":"subtítulo"},{"type":"p","text":"parágrafo com <b>negrito</b>"},{"type":"list","items":["ponto um","ponto dois"]}]}

5. Fazer o percent-encode do JSON inteiro com encodeURIComponent.

6. Link = BASE_URL + "#" + json_codificado.

7. Entregar:

   Documento gerado

   Abrir e imprimir: LINK

   E avisar: abra o link e faça Ctrl+P para salvar em PDF.

# Blocos

- "h" = subtítulo. "p" = parágrafo, aceita <b></b>. "list" = lista, usa items.

# Regras

- O logo vem do upload do usuário no ONE, nunca inventar a URL.

- Nunca workbench, documentGenerate ou integrações.

- ZERO travessão (—). Frases conectadas, sem staccato.

- Entregar SEMPRE o link clicável.
