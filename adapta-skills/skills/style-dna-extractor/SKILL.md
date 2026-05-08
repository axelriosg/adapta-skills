---
name: style-dna-extractor
description: Extrai o Style DNA de qualquer pessoa a partir de transcripts ou amostras de texto. Análise forense quantitativa em PT-BR com métricas estruturais (comprimento de frase, distribuição de frases curtas, densidade de vírgulas, complexidade vocabular), marcadores de tom em 7 categorias (informal, entusiasmo, certeza, inclusivo, direto, hedging, analogias), verbal tics ranqueados por frequência, padrões estruturais (hooks, conectores, perguntas, demos, listas) e top palavras de conteúdo. Dispara quando o usuário cola um transcript, arquivo .txt ou .vtt, pede análise de voz, padrões de escrita, estilo de comunicação, Style DNA, voice DNA, mapeamento de tom, análise forense de estilo, ou pede pra reproduzir o jeito de falar/escrever de alguém. Também dispara em pedidos como "analisa esse transcript", "extrai o tom", "como essa pessoa fala", "destila o estilo dela".
---

# Style DNA Extractor

Análise forense quantitativa de padrões de fala e escrita em português brasileiro. Recebe transcripts ou amostras de texto, devolve um perfil estilístico com 30+ métricas que descrevem o DNA único da pessoa analisada.

Mesmo método aplicado por Axel ao Masterizando IA Generativa para extrair o Style DNA de Max Peters a partir de 49.780 palavras (42 aulas, 7 horas de transcript).

## Quando ativar

Use sempre que o usuário:
- Colar um transcript (`.txt`, `.vtt`, ou texto direto)
- Pedir análise de voz, estilo, tom de comunicação, padrões de escrita
- Mencionar "Style DNA", "Voice DNA", "mapeamento de tom"
- Querer entender como alguém fala ou escreve para reproduzir esse jeito
- Pedir comparação estilística entre dois autores ou textos

Em dúvida, ative. Se faltar amostra de texto, peça antes de prosseguir.

## Inputs aceitos

Qualquer texto em PT-BR. Limpa automaticamente:
- Headers `WEBVTT`
- Timestamps no formato `00:00:00.000`
- Marcadores `align:center`
- Numerações de cue
- Mantém headers de aula/módulo como contexto

**Mínimo recomendado**: 5.000 palavras. Abaixo disso, alerte o usuário que os números perdem força estatística. Para análise definitiva, 30.000 palavras ou mais.

## Workflow

Execute em ordem. Não pule etapas. Toda análise sai como markdown estruturado pronto para colar em qualquer documento.

### Etapa 1: Recepção e limpeza

Se o usuário enviou arquivo via `upload_local_file`, leia o conteúdo. Se colou texto direto, use o texto da mensagem.

Limpe o texto removendo:
- Linhas que começam com `WEBVTT`
- Linhas que casam com regex `^\d{2}:\d{2}`
- Linhas que contêm `align:`
- Linhas que são apenas um número (cues numerados)
- Linhas vazias

Preserve linhas que começam com `Módulo`, `Aula`, ou `Curso de` como contexto, marcando com `=== ... ===`.

Após limpeza, junte em texto contínuo, conte palavras totais. Se ficar abaixo de 1.000 palavras, avise o usuário e pergunte se quer prosseguir mesmo assim.

### Etapa 2: Métricas estruturais

Calcule e apresente em tabela markdown:

| Métrica | Como calcular |
|---------|---------------|
| Total de palavras | `len(text.split())` |
| Total de frases | Split por `[.!?]+`, filtra strings vazias |
| Comprimento médio de frase | Média de palavras por frase |
| Frases curtas (%) | Percentual de frases com menos de 8 palavras |
| Frases muito curtas (%) | Percentual de frases com menos de 5 palavras |
| Comprimento médio de palavra | Caracteres por palavra |
| Densidade de vírgulas / 100w | `(commas / total_words) * 100` |
| Complexidade vocabular | `(unique_words / total_words) * 100` |

### Etapa 3: Marcadores de tom

Conte ocorrências de cada marcador. Normalize por 1000 palavras. Use exatamente estas listas:

**informal**: né, tá, legal, beleza, galera, cara, maneiro, massa
**enthusiasm**: muito, interessante, incrível, demais, maravilha, top, show, sensacional
**certainty**: claro, de fato, realmente, com certeza, sem dúvida, óbvio, evidente
**inclusive_a_gente**: a gente
**direct_voce**: você
**hedging**: talvez, de repente, provavelmente, pode ser, possível, digamos, mais ou menos, basicamente, querendo ou não
**analogies**: é como, que nem, é a mesma coisa, imagina que, digamos que, pensa nisso, tipo assim

Apresente em tabela com colunas: Categoria, Total, Por 1000 palavras. Em seguida, mostre breakdown dos 5 marcadores mais frequentes de cada categoria.

### Etapa 4: Verbal tics

Conte ocorrências dos seguintes tics. Inclua apenas os que aparecem 5 ou mais vezes. Ranqueie por frequência decrescente.

Lista padrão: certo, então, por exemplo, de novo, de repente, aqui, dentro, é claro, de fato, é interessante, bastante, principalmente, justamente, simplesmente, basicamente, e é claro, que nem, eu te espero, na próxima aula, ao longo do curso, que é, e por isso, lembra, a gente, olha só, na real, pra você, agora, aí, tá bom, sacou, beleza.

Se durante a leitura você notar tics adicionais não listados que aparecem com alta frequência, adicione à análise. Apresente como tabela: Tic, Frequência, Por 1000 palavras.

### Etapa 5: Padrões estruturais

Calcule:
- **Hook phrases**: contagem total de "nunca mais", "depois dessa aula", "eu tenho certeza", "você vai aprender", "você vai descobrir", "no final"
- **"Então" como conector**: contagem total
- **"Então" iniciando frase**: regex `(?:^|\. )então`
- **"E" iniciando frase**: regex `(?:^|\. )e `
- **Marcadores de demo ao vivo**: total de "aqui no", "dentro do", "vou abrir", "vou mostrar", "olha só", "presta atenção", "vamos ver"
- **Perguntas**: total de "?" e por 1000 palavras
- **Marcadores de lista**: contagem de "primeiro", "segundo", "terceiro", "por último"

Apresente em formato de bullets.

### Etapa 6: Top palavras de conteúdo

Tokenize com regex `\b[a-záàâãéêíóôõúç]+\b`, converta para lowercase. Filtre stopwords (lista abaixo) e palavras com 3 caracteres ou menos. Retorne as 20 mais frequentes.

Stopwords PT-BR: a, o, as, os, um, uma, uns, umas, de, do, da, dos, das, em, no, na, nos, nas, por, para, com, sem, sob, sobre, que, se, e, ou, mas, porém, então, quando, onde, como, eu, tu, ele, ela, nós, vós, eles, elas, você, vocês, meu, minha, seu, sua, nosso, nossa, dele, dela, é, são, foi, era, ser, estar, ter, tem, tinha, há, isso, isto, aquilo, esse, essa, esta, este, aquele, aquela, mais, muito, pouco, tanto, todo, toda, todos, todas, já, ainda, agora, aqui, aí, lá, ali, também, só, apenas, não, sim, talvez, sempre, nunca, às, ao, pela, pelo, me, te, lhe, vos, lhes, mim, ti, si, vai, vou, vão, pode, posso, podem, fazer, faz, está, estão, estava, estamos, fica, ficar.

Apresente em tabela: Palavra, Ocorrências.

### Etapa 7: Interpretação automática

Aplique estas regras e mostre apenas as que se aplicarem ao caso, em formato de bullets:

- Comprimento médio de frase abaixo de 16: "Frases curtas e diretas. Ritmo de conversa, não de palestra."
- Comprimento médio acima de 22: "Frases longas. Estilo mais elaborado ou acadêmico."
- Frases curtas acima de 35%: "Alta proporção de frases curtas indica ritmo de impacto."
- Complexidade vocabular abaixo de 15%: "Vocabulário simples e repetitivo. Acessibilidade alta."
- Complexidade acima de 30%: "Vocabulário variado. Estilo mais literário."
- "a gente" por 1000 palavras acima de 5: "Uso massivo de 'a gente' indica registro abrasileirado e inclusivo."
- "você" por 1000 palavras acima de 50: "Endereçamento direto ao leitor/ouvinte. Conversa, não monólogo."
- Hedging por 1000 palavras abaixo de 5: "Pouco hedging. Fala com convicção."

## Implementação técnica no Adapta ONE

Para cada análise, gere um script Python único no sandbox usando apenas a biblioteca padrão (`re`, `collections.Counter`, `pathlib`). Não precisa instalar dependências externas.

Estrutura sugerida do script:

```python
import re
from collections import Counter

# 1. Carrega texto, limpa headers VTT e timestamps
# 2. Calcula métricas estruturais
# 3. Conta marcadores de tom (loop sobre dicionário de categorias)
# 4. Conta verbal tics (filtro >= 5 ocorrências)
# 5. Calcula padrões estruturais
# 6. Top 20 palavras de conteúdo (após filtro de stopwords)
# 7. Aplica regras de interpretação
# 8. Monta relatório em markdown
```

Salve o script em `/tmp/style_dna.py`, execute via `bash`, capture o stdout e apresente para o usuário.

## Output esperado

Markdown estruturado em 6 seções na ordem:

1. **Métricas Estruturais** (tabela)
2. **Tom e Personalidade** (tabela + breakdown)
3. **Verbal Tics** (tabela)
4. **Padrões Estruturais** (bullets)
5. **Top Palavras de Conteúdo** (tabela)
6. **Interpretação Sugerida** (bullets)

Sempre comece com o nome da pessoa analisada e total de palavras processadas. Termine perguntando se o usuário quer aprofundar alguma seção, comparar com outro autor, ou usar o DNA extraído para escrever em estilo similar.

## O que evitar

- Não invente métricas além das listadas. A força do método é a consistência.
- Não tire conclusões de amostras menores que 1.000 palavras sem alertar.
- Não confunda análise quantitativa com análise de sentimento. Isso aqui mede estrutura, não emoção.
- Não traduza marcadores PT-BR para outras línguas sem alerta. Pra inglês ou espanhol, use listas adaptadas.
- Não rode duas vezes a mesma análise sem motivo. Se o usuário quer aprofundar, vá pra análise específica em vez de repetir tudo.

## Exemplo aplicado: Max Peters

Style DNA extraído de 49.780 palavras do curso Masterizando IA Generativa.

**Métricas estruturais**
- Comprimento médio de frase: 18.2 palavras
- Frases curtas (<8 palavras): 41.1%
- Frases muito curtas (<5 palavras): 20.3%
- Densidade de vírgulas / 100w: 6.58
- Complexidade vocabular: 12.2%
- Comprimento médio de palavra: 4.7 caracteres

**Tom dominante**
- "você" 109.7 por 1000 palavras (massivamente direto)
- "a gente" 8.2 por 1000 (sempre, nunca "nós")
- Informal 14.1 por 1000 ("tá" 250 vezes)
- Hedging baixo 3.8 por 1000 (fala com convicção)

**Verbal tics top 5**
- "aqui" 15.2 por 1000
- "então" 12.8 por 1000
- "a gente" 5.1 por 1000
- "por exemplo" 4.5 por 1000
- "dentro" 2.7 por 1000

**Palavra favorita**: "interessante" (181 ocorrências)

**Fórmula pedagógica destilada**: promessa bold > contexto pessoal > framework com nome > analogia do mundo real > demo ao vivo > recapitulação > ponte para próxima aula. Fechamento padrão: "Eu te espero já na próxima aula. Tchau."

## Limitações

- Funciona em PT-BR por default. Para outras línguas, edite as listas de marcadores e stopwords.
- Mede estrutura linguística, não conteúdo nem sentimento.
- Não substitui leitura humana, complementa com base quantitativa.
- Para áudio bruto, transcreva primeiro com whisper antes de rodar a análise.
