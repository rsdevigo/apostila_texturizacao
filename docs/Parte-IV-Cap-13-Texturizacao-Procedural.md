# Capítulo 13 — Texturização Procedural

## Introdução

O capítulo anterior fechou com uma promessa: ao distinguir os dois caminhos para construir uma textura que se repete, anunciamos que o método procedural resolve, na própria geração, o problema que o método fotográfico precisa corrigir depois. Cumprimos agora essa promessa, dedicando um capítulo inteiro a um modo de produzir texturas que, nas últimas duas décadas, transformou a indústria: a **texturização procedural**, em que a imagem não é pintada nem fotografada, mas *gerada* por regras, parâmetros e operações matemáticas. Em vez de definir cor a cor, o artista define um *processo* — uma cadeia de instruções que, executada, produz a textura —, e ao ajustar os parâmetros desse processo obtém infinitas variações sem refazer o trabalho. É uma mudança profunda na natureza do que se autora: deixa-se de criar uma imagem e passa-se a criar uma *receita* de imagens.

Este capítulo apresenta o paradigma procedural em seus fundamentos e em sua lógica de produção. Veremos o que significa gerar uma textura por regras, quais são os blocos construtivos elementares dessa geração — ruídos, padrões, operações —, e como eles se combinam, em grafos de nós, para produzir desde superfícies orgânicas complexas até materiais arquitetônicos precisos. Discutiremos a grande virtude do método — a flexibilidade não-destrutiva e a resolução independente de pixels — e seus custos e limites, pois nenhum método é universal. E reencontraremos, sob nova forma, a tensão que percorre a Parte IV entre o genérico, que o procedural produz com excelência, e o singular, que ele captura com dificuldade. Como nos demais capítulos, o objetivo não é ensinar a operar um software específico, mas formar a compreensão dos conceitos que sustentam o paradigma — conceitos que valem para o Substance Designer, para os nós de textura do Blender, para os sombreadores procedurais e para qualquer ferramenta presente ou futura que adote essa lógica.

## Desenvolvimento

### O que significa gerar uma textura por regras

> 📘 **Definição**
> **Texturizar proceduralmente** é descrever uma superfície não por sua aparência final, mas pelo *procedimento* que a produz. Em vez de definir o valor de cada pixel, o artista define uma cadeia de operações matemáticas — geradores, transformações, combinações — e a textura emerge da execução dessa cadeia. O artista autora o *processo*, não a imagem.

A diferença com a pintura digital, tema do próximo capítulo, é radical e vale fixá-la desde já. Quando se pinta, define-se diretamente o valor de cada região da imagem: este pixel é marrom, aquele é mais escuro, aqui há uma rachadura. Quando se gera proceduralmente, define-se uma cadeia de operações — "tome um ruído de células, distorça-o com um segundo ruído, use o resultado para misturar duas cores, escureça as fendas segundo a curvatura" — e a imagem emerge da execução dessa cadeia. O artista não toca os pixels; ele toca os *parâmetros* e as *operações* que os produzem. Essa indireção é, ao mesmo tempo, a força e a peculiaridade do método: força, porque mudar um parâmetro reconstrói a textura inteira instantaneamente, abrindo um espaço de variações infinitas; peculiaridade, porque exige pensar a superfície como sistema de regras, não como imagem, o que demanda um modo de raciocínio mais próximo da programação do que do desenho.

Dessa natureza decorrem duas propriedades que definem o valor do procedural. A primeira é a **edição não-destrutiva**: como a textura é o resultado de um processo, qualquer etapa pode ser alterada a qualquer momento sem desfazer as demais — aumenta-se a escala do padrão, troca-se a cor, intensifica-se o desgaste, e tudo se recalcula coerentemente. Não há o acúmulo irreversível de operações que caracteriza a edição de pixels; há uma receita sempre aberta à revisão. A segunda é a **independência de resolução**: como muitas operações procedurais são definidas matematicamente sobre um domínio contínuo, e não sobre uma grade fixa de pixels, a mesma receita pode ser executada em qualquer resolução, produzindo um mapa de mil ou de quatro mil pixels conforme a necessidade, sem perda de nitidez.

> 💡 **Dica Profissional**
> A independência de resolução é preciosa para pipelines de múltiplas plataformas: de um mesmo grafo procedural geram-se mapas em diferentes resoluções para console e para celular, atendendo a orçamentos distintos a partir de uma única fonte de autoria.

### Os blocos construtivos: ruídos, padrões e formas

Toda textura procedural, por mais complexa, é construída a partir de um repertório reduzido de geradores elementares. O mais fundamental deles é o **ruído** (*noise*): uma função que produz variação pseudo-aleatória mas controlada, a matéria-prima de quase toda irregularidade orgânica. Há famílias distintas de ruído, cada uma com um caráter visual próprio.

> 📘 **Definição**
> O **ruído de Perlin** e seus parentes produzem variações suaves e nebulosas, úteis para nuvens, manchas e variação tonal difusa. O **ruído celular** (ou de Voronoi) particiona o espaço em células, produzindo padrões que lembram escamas, pedras justapostas, craquelados e bolhas. O **ruído fractal** combina várias escalas de um mesmo ruído, somando versões cada vez menores e mais fracas — as chamadas "oitavas" — para produzir a riqueza de detalhe em múltiplas frequências que caracteriza superfícies naturais como terrenos, rochas e madeira.

*Minecraft* (Mojang, 2011) é o caso mais universalmente reconhecível desse princípio em um jogo: o relevo, as cavernas e a distribuição de biomas são gerados inteiramente por camadas de ruído Perlin e Simplex alimentadas por sementes numéricas, e cada semente produz um mundo único — demonstração prática de que o mesmo processo de ruído, com parâmetros distintos, jamais produz o mesmo resultado.

Ao lado dos ruídos estão os **padrões regulares** — grades, tijolos, ladrilhos, listras, padrões hexagonais — que fornecem a ordem geométrica das superfícies construídas pelo homem, e as **formas elementares** — gradientes, círculos, polígonos, formas de contorno — que servem de base para composições mais deliberadas. Um aspecto crucial, que conecta este capítulo ao anterior, é que esses geradores podem ser definidos para serem *seamless por construção*: como são funções sobre o domínio da textura, basta defini-los sobre o domínio toroidal para que a continuidade nas bordas seja automática. É por isso que o procedural nasce ladrilhável: o ruído e o padrão gerados corretamente já se repetem sem costura, sem necessidade da correção que o método fotográfico exige.

### Operações: combinar, transformar, mascarar

Os geradores produzem matéria-prima; são as **operações** que a esculpem em material. O repertório de operações é o segundo pilar do paradigma, e organiza-se em algumas grandes categorias. As **operações de combinação** misturam duas ou mais entradas: somam, multiplicam, subtraem, sobrepõem — as mesmas operações de mistura familiares à edição de imagem, agora aplicadas a sinais gerados. As **transformações** alteram a geometria do sinal: deslocam, rotacionam, repetem, espelham, ou — operação especialmente poderosa — *distorcem* um sinal usando outro como mapa de deslocamento (*warp*), técnica que quebra a regularidade mecânica do ruído e produz a irregularidade fluida de veios de madeira, fumaça, mármore. As operações de **ajuste** remapeiam valores: alteram contraste, níveis, curvas, transformando um ruído suave num padrão de manchas nítidas ou num gradiente delicado conforme a intenção.

> 📘 **Definição**
> O **mascaramento** procedural é a operação de usar um sinal — frequentemente um ruído ou um mapa derivado da geometria, como a curvatura — como máscara para aplicar um efeito apenas em certas regiões: escurecer só as reentrâncias, acumular ferrugem só nas quinas, depositar sujeira só nas áreas planas. É a operação transversal que dá distribuição plausível a todos os outros efeitos.

A potência do método vem da *composição* dessas operações em cadeias longas, cada etapa alimentando a seguinte. Uma textura de pedra desgastada pode nascer de um ruído celular para as pedras, distorcido por um ruído fino para irregularizar os contornos, com as juntas escurecidas por uma operação de aresta, manchas de musgo aplicadas por uma máscara de oclusão nas reentrâncias, e variação de cor introduzida por um ruído de grande escala — tudo encadeado, tudo reajustável.

> **Figura 13.1** — Uma cadeia procedural elementar representada como fluxo da esquerda para a direita — um ruído celular, sua distorção por um segundo ruído, o escurecimento das juntas, a aplicação de manchas por máscara — mostrando como cada operação transforma a saída da anterior até o material final.

**Screenshot sugerido:** Captura de um pequeno grafo de nós em uma ferramenta procedural, com cada nó rotulado por sua função, ilustrando a composição de geradores e operações.

### O grafo de nós: a textura como receita visível

> 📘 **Definição**
> Um **grafo de nós** é a representação visual da cadeia de geração procedural: cada gerador e cada operação é uma caixa (nó), e as conexões entre elas mostram como os sinais fluem da entrada à saída. Esse grafo *é* a textura — não existe imagem armazenada, apenas a receita que a produz quando executada.

Ferramentas procedurais modernas — o Substance Designer como exemplo canônico, mas também os sistemas de nós do Blender e dos motores de jogo — adotam essa representação. Ela não é mero detalhe de interface; ela materializa as propriedades que discutimos. A edição não-destrutiva torna-se literal — alterar um nó recalcula tudo a jusante; a parametrização torna-se visível — expõem-se controles que ajustam o comportamento de cada nó; e a reutilização torna-se natural — um subgrafo que produz ferrugem pode ser empacotado e reaproveitado em muitos materiais.

> 💡 **Dica Profissional**
> Construa grafos *legíveis e organizados*, como se escreve um bom programa: nomeie os nós, agrupe subgrafos por função, exponha parâmetros significativos e documente a intenção. Um grafo desorganizado é tão custoso de manter quanto código ilegível — e desperdiça a maior vantagem do método, a edição não-destrutiva e a reutilização.

Dessa estrutura nasce a aplicação mais transformadora do paradigma: o **material procedural parametrizado**, ou *smart material*. Em vez de uma textura fixa, produz-se um *gerador de texturas* com controles expostos — a quantidade de desgaste, a intensidade da ferrugem, a cor dominante, a escala do padrão — que qualquer membro da equipe pode ajustar para obter variações sem mexer no grafo. Constrói-se uma vez a lógica de "metal pintado e desgastado" e dela se extraem dezenas de metais diferentes apenas movendo controles. Esse é o cerne do ganho de produção do procedural em escala: não se autora cada material, autora-se a *família* de materiais, e a biblioteca de geradores reutilizáveis de um estúdio torna-se um ativo tão valioso quanto as próprias texturas. Reencontramos, elevada a outro patamar, a lógica de reutilização das *trim sheets* do Capítulo 12 — só que aqui o que se reutiliza não é uma imagem, mas a inteligência que a gera.

### A grande virtude e os limites do procedural

A virtude central do procedural já está exposta: flexibilidade não-destrutiva, independência de resolução, parametrização e reutilização em escala, e a geração automática de variação que combate a repetição estudada no capítulo anterior — pode-se gerar quatro variantes de uma pedra apenas mudando a semente do ruído. Some-se a isso a coerência: um material procedural bem construído incorpora regras de plausibilidade física — a ferrugem se acumula onde a água escorre, o desgaste aparece nas quinas salientes — que se aplicam automaticamente, poupando o artista de decidir, ponto a ponto, onde cada efeito ocorre.

> ⚠️ **Atenção**
> O ponto fraco do procedural é justamente o oposto de sua força: o **controle singular, ponto a ponto**, da intenção artística específica. Definir por regras uma marca particular num lugar exato — a cicatriz nesta posição do rosto do herói, o logotipo nesta face da caixa — é antinatural e trabalhoso no paradigma procedural, que pensa em termos de distribuições e padrões, não de ocorrências únicas.

Há também uma curva de aprendizado e um modo de pensar exigente: construir grafos complexos requer raciocínio quase algorítmico, e um grafo mal estruturado torna-se tão difícil de manter quanto um programa mal escrito. A conclusão reencontra a tese da Parte IV: o procedural domina o genérico, o repetível, o variável; a pintura digital, próximo capítulo, domina o singular, o específico, o intencional; e o trabalho profissional combina os dois, usando o procedural para construir a base coerente e a pintura para acrescentar o particular que a regra não alcança.

## Aplicação em Jogos

Na produção contemporânea, a texturização procedural é o motor da escala. Estúdios que precisam povoar mundos vastos com materiais consistentes constroem bibliotecas de *smart materials* — metais, madeiras, concretos, tecidos, terrenos — que se aplicam e se ajustam rapidamente, garantindo coerência visual entre centenas de *assets* produzidos por muitas mãos. *Halo 5: Guardians* (343 Industries, 2015) é considerado um dos primeiros jogos AAA de grande orçamento a adotar o Substance Designer como ferramenta central de construção de materiais de ambiente; a equipe documentou como a biblioteca de *smart materials* — concreto, metal Forerunner, terraço rochoso — tornou possível vestir cenários inteiramente novos em tempo de produção significativamente inferior ao que a pintura manual exigiria, validando, em escala industrial, o argumento central do paradigma procedural.

É revelador que o procedural raramente atue sozinho. O fluxo de trabalho dominante na indústria — exemplificado pela dupla Substance Designer e Substance Painter, mas não restrito a ela — separa duas tarefas complementares: gerar materiais procedurais reutilizáveis e *aplicá-los* a um modelo específico com ajuste e pintura. Constrói-se o material genérico proceduralmente; aplica-se a um *asset* concreto guiando-o pelos mapas derivados da geometria daquele objeto (curvatura, mapa de oclusão ambiente, identificação de partes), de modo que o desgaste e a sujeira procedurais se distribuam segundo a forma real; e completa-se, quando necessário, com pintura manual do que é singular. Esse encadeamento — procedural para a base, geometria para a distribuição, pintura para o particular — é a síntese prática dos métodos de produção, e a pintura digital (Capítulo 14) e as máscaras (Capítulo 17) desenvolvem suas peças restantes.

## Estudo de Caso

### No Man's Sky — Hello Games, 2016

*No Man's Sky* contém 18 quintilhões de planetas. Nenhuma delas tem uma textura pintada. Cada superfície — solo, rocha, vegetação, gelo, lama — é gerada em tempo real a partir de um sistema de materiais procedurais parametrizados por bioma. Grant Duncan, da Hello Games, apresentou na GDC 2017 o talk *"Building Worlds Using Math(s)"*, descrevendo como o estúdio construiu esse sistema com uma equipe de menos de vinte pessoas. O princípio é a síntese do que este capítulo ensina: em vez de pintar texturas de planetas, a Hello Games construiu *grafos que descrevem as regras de qualquer planeta possível*, e a textura nasce dos parâmetros.

O grafo de material de planeta de *No Man's Sky* recebe como entradas os parâmetros de bioma da semente procedural — temperatura (gelo, temperado, tropical), tipo de solo (rocha, areia, lama), nível de radiação, cor dominante da fauna — e produz, a partir deles, os canais de textura da superfície: cor base, mapa de rugosidade e mapa de normais. O mecanismo central é o *blending* por altitude e normal: rochas emergentes têm material distinto do solo ao redor, e a transição entre eles é guiada pela inclinação da superfície e pela altitude, exatamente como a natureza distribui materiais em terrenos reais. Ruído fractal injeta a variação orgânica; padrões de Voronoi criam as juntas de crosta e as fissuras; operações de *warp* distorcem os padrões para que nunca pareçam repetidos.

O mais revelador no caso da Hello Games, do ponto de vista pedagógico, é a capacidade de revisão que o sistema oferece. Em várias das atualizações pós-lançamento — *Next*, *Origins*, *Frontiers* —, a equipe reformulou completamente a aparência visual dos planetas sem refazer um único ativo manualmente: ajustou os parâmetros e as curvas de *blending* nos grafos, e todos os planetas existentes receberam a nova aparência automaticamente. É a demonstração mais clara já publicada do princípio que define o procedural: não economiza apenas o tempo de criar, mas multiplica o alcance de qualquer revisão, porque uma mudança num nó do grafo-mãe se propaga instantaneamente por toda a saída.

**O que aprender com isso:** O ganho do procedural não está apenas na criação inicial, mas na revisão: uma alteração num parâmetro central propaga-se automaticamente por toda a produção dependente desse grafo, reduzindo o custo de mudanças de direção que seriam proibitivas num pipeline de pintura manual.

> **Figura 13.2** — Dois planetas de *No Man's Sky* gerados pelo mesmo sistema de grafos com sementes e parâmetros de bioma distintos: planeta árido com solo alaranjado e formações de cristal, e planeta gelado com névoa azulada e vegetação baixa. Toda a diferença visual nasce dos parâmetros do bioma alimentados ao mesmo grafo procedural.

**Screenshot sugerido:** Comparação lado a lado de dois planetas de No Man's Sky de biomas contrastantes — árido e gelado — mostrando que a diferença visual total nasce da mesma estrutura de grafo com parâmetros distintos.

## Boas Práticas

A boa prática central da texturização procedural é pensar em *sistemas*, não em imagens: ao abordar um material, pergunte que regras o governam e construa o gerador dessas regras, em vez de pintar um caso particular — assim o trabalho rende variação e reutilização. Construa grafos legíveis e organizados, com nós nomeados, subgrafos agrupados por função e parâmetros significativos expostos. Aproveite a natureza seamless por construção do procedural para gerar texturas tileables corretas na origem, e a independência de resolução para alimentar o pipeline de múltiplas plataformas a partir de uma única fonte.

É boa prática guiar a distribuição de desgaste, sujeira e variação pelos mapas derivados da geometria — curvatura, mapa de oclusão ambiente, identificação de partes —, de modo que os efeitos procedurais respeitem a forma real do objeto e ganhem plausibilidade física. Mantenha, como sempre, a disciplina do PBR da Parte III: a saída procedural deve produzir canais de textura calibrados e fisicamente corretos, com cor base livre de luz e valores dentro das faixas de referência. E reconheça com lucidez os limites do método: quando a superfície exige uma marca singular, intencional, num ponto específico, combine o procedural com a pintura digital do próximo capítulo, usando cada paradigma para aquilo em que é forte.

## Erros Comuns

O erro conceitual mais comum entre iniciantes é abordar o procedural com a mentalidade da pintura, tentando controlar a textura ponto a ponto e frustrando-se com a indireção do método — quando o ganho do procedural está justamente em pensar por regras e parâmetros, não por pixels. O erro oposto também ocorre: usar o procedural onde ele não cabe, insistindo em produzir por regras uma marca singular e específica — um logotipo, uma cicatriz numa posição exata — que a pintura resolveria em segundos, gastando horas a construir um grafo tortuoso para imitar o que é, por natureza, um ato pontual.

> ❌ **Erro Comum**
> Produzir um material procedural fixo, sem controles expostos, que poderia ter gerado uma família inteira de variantes mas gera apenas um caso. Não parametrizar é desperdiçar a maior vantagem do método: qualquer *smart material* deve ter pelo menos os parâmetros de escala, intensidade de desgaste e variação de cor expostos como controles ajustáveis.

Há o erro de ignorar os mapas derivados da geometria, distribuindo desgaste e sujeira por ruído puro, sem relação com a forma do objeto, o que produz resultados genéricos e fisicamente implausíveis — ferrugem onde a água nunca escorreria, desgaste em superfícies que nada toca. E persiste o erro de descuidar da disciplina PBR na empolgação com o grafo, gerando canais de textura com cor base "suja" de iluminação ou valores fora das faixas de referência, repetindo no contexto procedural os erros que a Parte III ensinou a evitar.

## Resumo

Este capítulo apresentou a texturização procedural, o paradigma em que a imagem não é pintada nem fotografada, mas gerada por regras, parâmetros e operações. Vimos que sua diferença essencial em relação à pintura é a indireção — autora-se o *processo* que produz a textura, não a textura —, e que dela decorrem suas duas propriedades definidoras: a edição não-destrutiva, em que qualquer etapa é revisável sem desfazer as demais, e a independência de resolução, em que a mesma receita se amostra em qualquer densidade. Estudamos os blocos construtivos — os ruídos (Perlin, celular, fractal) que injetam o acaso plausível da natureza, os padrões regulares da ordem construída e as formas elementares — e as operações que os esculpem — combinação, transformação, distorção (*warp*), ajuste e mascaramento —, compostas em cadeias longas representadas pelo grafo de nós, que materializa a textura como receita visível. Vimos nascer dessa estrutura o *smart material* parametrizado, gerador de famílias inteiras de variantes e ápice da reutilização que herda e eleva a lógica das *trim sheets*. Reconhecemos a grande virtude do procedural — flexibilidade, escala, variação automática contra a repetição, coerência física — e seus limites — a dificuldade com o controle singular e intencional, a curva de raciocínio algorítmico, o custo de complexidade. E concluímos que o procedural domina o genérico e o repetível, enquanto o singular pertence à pintura digital do próximo capítulo.

## Exercícios

**1.** Explique, com suas palavras, a diferença fundamental entre texturizar por pintura e texturizar proceduralmente. A partir dessa diferença, deduza por que o procedural oferece edição não-destrutiva e independência de resolução, e por que essas duas propriedades são valiosas para um pipeline que atende a múltiplas plataformas.

**2.** Descreva o papel dos ruídos na geração procedural e diferencie pelo menos duas famílias (por exemplo, Perlin e celular) quanto ao caráter visual e ao tipo de superfície que ajudam a produzir. Explique por que combinar várias escalas de ruído (ruído fractal) aproxima o resultado da aparência de superfícies naturais.

**3.** Comparação crítica: discuta as forças e os limites do paradigma procedural. Dê um exemplo de superfície para a qual o procedural é claramente a melhor escolha e um exemplo para o qual ele é a pior, justificando ambos a partir da oposição entre o genérico/repetível e o singular/intencional que estrutura esta parte da apostila.

**4.** O *smart material* parametrizado é descrito como o ápice da reutilização procedural. Explique o que é, relacione-o à lógica de reutilização das *trim sheets* do Capítulo 12 e descreva, com um exemplo, por que ele economiza esforço sobretudo na fase de *revisão* de um projeto, e não apenas na de criação.

**5.** Planejamento integrado: você precisa texturizar uma floresta de pedras e rochas para um jogo de mundo aberto — centenas de rochas, todas coerentes mas variadas, vistas a distâncias diferentes. Descreva como conduziria a tarefa proceduralmente: que geradores e operações usaria para a base, como distribuiria musgo e desgaste de modo fisicamente plausível usando mapas derivados da geometria, como produziria variação para combater a repetição, e como a independência de resolução atenderia às diferentes distâncias de visualização. Indique, por fim, onde a pintura digital ainda poderia ser necessária.

## Glossário

**Canal de textura:** Cada um dos mapas de informação que compõem um material PBR — cor base, mapa de rugosidade, mapa metálico, mapa de normais etc. No contexto procedural, cada canal é gerado por uma saída do grafo de nós.

**Edição não-destrutiva:** Propriedade de um sistema de autoria em que qualquer etapa pode ser modificada sem desfazer as demais, porque o resultado é calculado a partir da receita, não gravado diretamente.

**Grafo de nós:** Representação visual de uma cadeia procedural em que cada gerador ou operação é um nó e as conexões mostram o fluxo de dados; materializa a textura como receita visível e editável.

**Independência de resolução:** Propriedade de operações procedurais definidas sobre domínio contínuo, que permite executar a mesma receita em qualquer resolução de saída sem perda de nitidez.

**Mapa de oclusão ambiente:** Canal de textura que registra o quanto cada ponto da superfície está exposto à luz ambiente — escuro nas reentrâncias, claro nas protuberâncias. Usado como máscara para distribuir sujeira e desgaste.

**Mascaramento:** Operação que usa um sinal como controle de onde outro efeito é aplicado; no procedural, permite concentrar desgaste, ferrugem ou sujeira apenas nas regiões fisicamente plausíveis.

**Ruído celular (Voronoi):** Gerador procedural que particiona o espaço em células de Voronoi, produzindo padrões semelhantes a escamas, pedras justapostas e craquelados.

**Ruído de Perlin:** Gerador procedural de variação suave e nebulosa, base de nuvens, manchas orgânicas e variação tonal difusa; combinado em oitavas produz ruído fractal.

**Ruído fractal:** Combinação de várias escalas de um mesmo ruído (oitavas), somando versões progressivamente menores e mais fracas para aproximar a riqueza de detalhe de superfícies naturais.

**Smart material:** Material procedural parametrizado com controles expostos — escala, intensidade de desgaste, cor dominante etc. — que gera uma família de variantes a partir de um único grafo, sem reautoria.

**Warp (distorção):** Operação procedural que usa um sinal como mapa de deslocamento para deformar outro, quebrando a regularidade mecânica e produzindo a irregularidade fluida de veios, fumaça e mármore.

## Leituras Complementares

- **[*The PBR Guide* — Wes McDermott, Allegorithmic / Adobe, 2018]** — Embora centrado no PBR, fundamenta a exigência de que a saída procedural produza canais de textura calibrados e fisicamente corretos. Leitura necessária para manter a disciplina de autoria no contexto do grafo de nós.

- **[*The Book of Shaders* — https://thebookofshaders.com/?lan=pt]** — Introdução acessível e progressiva à geração procedural de padrões e ruídos a partir de funções matemáticas. Leitura ideal para compreender, por dentro, os blocos construtivos — ruído, padrões, distorção — tratados neste capítulo.

- **[*Adobe Substance 3D* — https://helpx.adobe.com/substance-3d.html]** — Documentação do Substance Designer (construção procedural por grafos de nós) e do Substance Painter (aplicação de *smart materials* a modelos). Referência da indústria para o fluxo descrito neste capítulo.

- **[*Blender Manual* — https://docs.blender.org/]** — Seções sobre o sistema de nós de textura e sombreamento, geradores procedurais e ruídos. Recomendado para praticar os conceitos do grafo de nós em uma ferramenta de código aberto.

- **[AKENINE-MÖLLER; HAINES; HOFFMAN. *Real-Time Rendering*, 4. ed. CRC Press, 2018]** — Os capítulos sobre texturas procedurais e funções de ruído fundamentam teoricamente a geração tratada aqui; leitura recomendada para quem deseja aprofundamento matemático.
