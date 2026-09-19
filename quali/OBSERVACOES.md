# Observações sobre a qualificação — rigor científico e qualidade das explicações

Estas observações tratam de **como você está fazendo e escrevendo pesquisa**, não de erros pontuais do texto (esses já foram corrigidos no fonte). São os pontos que, se você endereçar agora, mudam o patamar da dissertação.

---

## Parte 1 — Rigor científico

### 1.1 Você está confundindo "medida que consegui calcular" com "evidência"

O procedimento de numeração atribuiu identificador a **90,77% dos objetos**. Esse número está no resumo, nas considerações finais e na discussão integrada. Mas ele mede **completude**, não **correção** — e você mesmo escreve isso, três vezes, com clareza exemplar.

O problema não é a honestidade; é a **hierarquia**. Um número que não responde à pergunta de pesquisa não deveria ocupar o lugar mais nobre do texto. Se você não pode dizer se os identificadores estão certos, o resultado da seção é: *"o procedimento roda e produz saída; sua acurácia é desconhecida"*. Isso é uma frase, não um destaque do resumo.

**Regra prática:** antes de colocar um número no resumo, pergunte *"que afirmação eu posso defender na banca com este número?"*. Se a resposta tiver a forma "consegui calcular isso", o número não entra.

O mesmo vale para os 76,66% do classificador. Sem a linha de base de 56,74% (classe majoritária), o número não significa nada — e ficou solto no texto por meses. Adicionei a tabela por classe; adote o hábito: **todo classificador vem com baseline trivial, sempre.**

### 1.2 Circularidade na escolha da configuração da proposta

A proposta adota YOLO26s-OBB @1024 porque foi *"a configuração do detector de melhor desempenho na avaliação comparativa"*. Mas o Capítulo 3 afirma, corretamente e em quatro lugares diferentes, que aquela comparação **não é uma ablação** e que as diferenças **não podem ser atribuídas à arquitetura nem à representação**.

Não dá para ter as duas coisas. Ou a comparação sustenta uma conclusão sobre configuração — e aí é ablação — ou não sustenta, e aí a escolha precisa ser declarada pelo que é: **uma decisão pragmática** (é o modelo que está em uso, é o que temos pesos e pipeline prontos). Declare assim. Banca não penaliza escolha pragmática declarada; penaliza conclusão que o desenho não sustenta.

### 1.3 Você adia hipóteses alternativas que custam uma tarde

A explicação "treinado com 1.280 px, avaliado com 1.024" aparece **três vezes** no texto, sempre como *"explicação possível, ainda não testada"*, e foi empurrada para o cronograma.

Testá-la é rodar `val` com `imgsz=1280` nos pesos que já existem. Horas, não semanas.

Enquanto isso não for feito, o resultado central do Capítulo 4 — o contraste entre recall 0,930 e 0,523/0,546 — tem uma explicação alternativa trivial em aberto. **Isso é o primeiro lugar onde a banca vai bater.** Um dos pilares do rigor é: *quando existe explicação alternativa barata de testar, você testa antes de escrever o parágrafo que a menciona.* Levantar e não fechar é pior que não levantar, porque mostra que você viu o problema e escolheu não resolvê-lo.

### 1.4 Proveniência de dados não é "limitação" — é pré-condição

O texto registra que o conteúdo de `dataset_imgs_aug1` na época do treinamento **não é conhecido**, e que por isso os dados de treinamento do `260525_2231` não podem ser reconstituídos. Esse checkpoint é, simultaneamente:

- o modelo de inferência do sistema;
- o gerador dos rótulos do classificador auxiliar;
- o melhor resultado do trabalho (F1 0,941);
- a justificativa da configuração da proposta.

Você está declarando isso como limitação. **Não é.** É um risco de invalidação: se `dataset_imgs_aug1` contiver qualquer das 27 imagens de teste, o 0,941 não existe. Ele é 14 pontos acima do segundo colocado (0,891) e 25 acima dos outros dois OBB — exatamente o padrão que contaminação produz.

**Faça agora, antes de qualquer outra coisa:** liste os arquivos de `dataset_imgs_aug1` e cruze com a partição de teste, por nome de origem (removendo os sufixos de augmentation). É um `comm` de duas listas.

E adote o hábito que evita isso para sempre: **todo treinamento grava, junto dos pesos, um manifesto** com a lista de arquivos de cada partição e o hash de cada um. Sem isso, um experimento não é um experimento — é uma anedota.

### 1.5 Falta o braço de controle que torna o resultado interpretável

A proposta compara quatro condições: completo, F1 < 0,97, remover 30% menos instáveis, aleatório de mesmo tamanho.

Falta **remover os 30% MAIS instáveis**.

Sem esse braço, suponha que a seleção por instabilidade vença o aleatório. Você não consegue distinguir duas explicações:

1. o critério identifica imagens informativas (sua hipótese);
2. podar 30% de qualquer forma sistemática — em oposição a aleatória — ajuda.

O braço oposto separa as duas. Ele custa 3 treinamentos e é o que transforma o experimento de "testei minha ideia" em "testei minha ideia contra a alternativa".

### 1.6 Uma limitação conceitual do escore que você ainda não discutiu

Seu próprio Capítulo 2 diz que instabilidade baixa agrupa objetos *"detectados **ou não detectados** de forma consistente"*.

Portanto, remover os 30% de menor instabilidade descarta, no mesmo saco, **o trivial e o impossível**. São coisas opostas: a imagem que o detector sempre acerta não ensina nada; a imagem que ele nunca acerta pode ser justamente a que importa (baixa resolução, parcela morta, oclusão).

Isso precisa estar na proposta como limitação conhecida, com um diagnóstico previsto: **reportar a distribuição de F1 dentro do grupo removido.** Se o grupo removido for bimodal, você sabe que o escore está fundindo dois fenômenos, e isso é em si um resultado publicável.

### 1.7 Três limiares de correspondência sem justificativa unificada

No mesmo trabalho convivem:

| Limiar de IoU | Onde | Justificativa no texto |
|---|---|---|
| 0,3 | opção de detecção do sistema | nenhuma |
| 0,5 | avaliação comparativa | nenhuma |
| > 0 | cálculo do escore de instabilidade (proposta) | nenhuma |

Cada um deve ter uma frase de justificativa, e o da proposta — o `> 0` — merece uma análise de sensibilidade, porque ele define o sinal que você vai medir. Se a conclusão mudar entre `> 0` e `≥ 0,3`, você precisa saber disso antes da banca, não depois.

**Princípio:** parâmetro herdado por inércia é parâmetro indefensável. Ou você justifica, ou você mostra que o resultado não depende dele.

### 1.8 Defina o tamanho de efeito relevante ANTES de rodar

A proposta diz que a hipótese será avaliada "pelas diferenças estimadas e por seus intervalos de confiança". Correto, mas incompleto: **falta dizer qual diferença importa**.

Se a seleção por instabilidade der +0,004 de mAP@0,5:0,95 com IC que não cruza zero, isso é um resultado positivo ou irrelevante? Você precisa responder isso agora, no texto, e não depois de ver o número — senão a decisão vira racionalização.

Defina também como a variabilidade será decomposta: você tem variabilidade **entre sementes** (3 por condição) e **entre grupos espaciais** (o bootstrap). São fontes diferentes e o texto trata só da segunda.

---

## Parte 2 — Qualidade das explicações

### 2.1 O excesso de ressalvas está trabalhando contra você

Contei dezenas de ocorrências de *"não permite atribuir"*, *"não isola o efeito"*, *"não estima generalização"*, *"não deve ser interpretado como"*. Elas estão **todas corretas**. E é justamente por isso que vale o aviso: repetidas em toda seção, deixam de comunicar rigor e passam a comunicar insegurança. O leitor cansa e começa a ler o texto como uma defesa antecipada, não como um relato.

**Como consertar:** declare a limitação **uma vez**, no lugar certo, com rótulo próprio (você já tem `§Limitações Metodológicas`), e nas demais seções referencie: *"com a restrição da Seção X"*. Uma ressalva bem colocada vale dez repetidas.

### 2.2 Você está descrevendo o código, não explicando o método

Trechos como:

- *"a gravação dos recortes e da composição em grade está desabilitada"*
- *"dois arquivos de pesos estão preservados: ... e cnn_filtro_qualidade.pth, de uma arquitetura com 10 tensores"*
- *"o módulo tem um segundo diretório de dados, dataset1"*
- *"a supressão de regiões da imagem está implementada, mas desativada"*

Isso é **registro de engenharia**, não metodologia. Um capítulo de método responde a: *o que foi feito, por quê, e como reproduzir*. Não responde a *o que mais existe no repositório*.

**Critério de corte:** se o leitor não puder, com aquela informação, nem reproduzir seu experimento nem decidir se confia nele — vai para apêndice ou sai. O Capítulo 3 tem 86 KB; encolhê-lo em um terço vai fortalecê-lo, não enfraquecê-lo.

### 2.3 Falta o "por quê" antes do "como"

Você descreve com precisão notável **como** as coisas funcionam, e quase nunca **por que** foram escolhidas assim:

- por que 500 objetos como alvo da subdivisão? (por que não 200, ou 1.000?)
- por que a penalidade na matriz de custo é 1000, e não infinito?
- por que σ decrescente entre estágios, e não fixo?
- por que 70/20/10 no classificador, quando o resto do projeto usa 70/20/10 diferente?

Cada número escolhido por você merece uma oração de justificativa — mesmo que seja *"valor adotado empiricamente, sem otimização"*. Você faz isso muito bem no caso do limiar 0,97; faça em todos.

**Por que isso é rigor, e não estilo:** uma escolha sem justificativa é indistinguível de um acaso, e a banca não tem como saber se você pensou no assunto ou não.

### 2.4 O capítulo de Resultados não mostra nada

Nove seções, sete tabelas, **zero figuras**. O leitor termina o capítulo sem nunca ter visto uma parcela detectada.

Isso não é só apresentação: é uma lacuna de argumento. Você afirma que caixas orientadas têm vantagem geométrica em parcelas rotacionadas — mostre uma imagem com HBB e OBB sobrepostas na mesma parcela inclinada, e o argumento se prova sozinho. Você relata que uma imagem com 23 referências teve F1 = 0,1622 — mostre a imagem, e o leitor entende o modo de falha.

Mínimo para a dissertação:
1. detecções sobre uma imagem orbital e uma de drone, com acertos/FP/FN em cores distintas;
2. HBB vs. OBB na mesma parcela rotacionada;
3. o caso de falha (a imagem de F1 0,16) — casos que destoam explicam mais que médias;
4. a saída da numeração no P11, com os 58–61 objetos sem par destacados;
5. F1 e recall por faixa de densidade, como gráfico.

### 2.5 Números soltos, sem interpretação

*"No checkpoint com maior F1 global, uma imagem com 23 referências teve 14 predições e três correspondências aceitas (F1 de 0,1622)."*

E depois disso, nada. Qual imagem? Orbital ou drone? O que ela tem de diferente? Foi erro de detecção ou de anotação?

Um outlier desses é **a melhor oportunidade de explicação do capítulo** e você o trata como nota de rodapé. Pesquisa boa persegue o caso que não encaixa; é lá que está o mecanismo.

O mesmo com os 8 arquivos do P11 que têm 275–278 objetos contra uma referência de 217. Você observa corretamente que o excedente **tem** que ficar sem par — mas não pergunta o óbvio: *por que esses arquivos têm 60 objetos a mais?* São falsos positivos? A referência está incompleta? São parcelas de bordadura? A resposta muda completamente a leitura dos 90,77%.

### 2.6 Terminologia flutuante

Você escreve, com lucidez: *"o termo centroide, empregado de forma genérica, encobre procedimentos que não são idênticos"*. Perceber isso é ótimo. Mas então o texto **continua usando os três termos de forma intercambiável** por mais 40 páginas.

Diagnosticar uma ambiguidade e conviver com ela é meio caminho. Fixe **um termo por conceito**, registre num glossário, e use sempre o mesmo. O pacote `acronym` já está carregado no preâmbulo e não é usado — VANT, GSD, HBB, OBB, CNN, IoU, AP, mAP, CPD, FPN estão todos definidos ad-hoc no meio do texto. Monte a lista de siglas.

### 2.7 Frases que não fecham

Havia (corrigi) uma frase sem predicado, várias vírgulas separando sujeito de verbo, e erros de concordância com `\citet` (*"mardanisamani2021 utiliza"* → *utilizam*). Isoladamente são deslizes; juntos são sintoma de **texto que não foi relido**.

**Método simples e eficaz:** leia em voz alta. Se você precisar respirar no meio da frase, ela está longa demais. Se você tropeçar, o leitor também tropeça. Faça isso antes de entregar qualquer versão ao orientador — releitura é sua responsabilidade, não dele.

---

## Parte 3 — Hábitos a adotar a partir de agora

1. **Manifesto de dados por experimento.** Lista de arquivos + hash por partição, gravada junto dos pesos. Sem exceção.
2. **Protocolo escrito antes de rodar.** Condições, desfecho primário, tamanho de efeito relevante, critério de parada. Depois de ver os resultados é tarde.
3. **Toda tabela e figura citada no texto.** (Corrigido — 13 das 14 figuras estavam órfãs.)
4. **Baseline sempre.** Classificador sem classe majoritária, detector sem modelo não ajustado, seleção sem braço oposto — nenhum resultado é interpretável sozinho.
5. **Hipótese alternativa barata se testa, não se declara.**
6. **Separe "o que eu fiz" de "o que existe no repositório".** O segundo vai para apêndice.
7. **Um termo por conceito.** Glossário no início, usado até o fim.

---

## O que está bom e você deve preservar

Não é elogio protocolar — três coisas aqui são acima da média para uma qualificação:

**Consistência numérica impecável.** Conferi todas as somas: partições (277 = 195+55+27; 384 = 258+80+46; 3.532 = 3.406+80+46), TP/FP/FN por faixa de densidade fechando exatamente com os totais globais (7.301 / 370 / 551), 4.691/5.168 = 90,77%, 3.480+994+497 = 4.971. **Nada está errado.** Isso é raro e é a base de tudo o mais.

**Honestidade intelectual.** Você declara que não sabe os dados de treinamento de um checkpoint, que a divisão vazou espacialmente, que o mAP vem de outra passagem com outros parâmetros. Muita gente esconde isso. Não perca esse traço — o ajuste que peço é de **dosagem e de posicionamento**, nunca de menos honestidade.

**Distinções conceituais que a maioria não faz:** global vs. média por imagem; IoU ponderado vs. por imagem; mAP@0,5 vs. @0,5:0,95 (e a separação entre variar confiança e variar limiar de sobreposição); ausência visual ≠ posição inexistente ≠ parcela morta; identificador atribuído ≠ identificador correto. Essa cadeia, na Seção sobre informação visual incompleta, é o melhor texto do documento.

---

## Prioridade, em ordem

1. Cruzar `dataset_imgs_aug1` com a partição de teste — **antes de tudo**
2. Reavaliar os checkpoints de 1.280 px na própria dimensão de treino
3. Acrescentar o braço "remover 30% mais instáveis" à proposta
4. Definir o tamanho de efeito relevante e escrevê-lo na proposta
5. Produzir as figuras do Capítulo 4
6. Enxugar o Capítulo 3 (tirar o que é registro de código)
7. Consolidar as ressalvas repetidas em um único lugar
