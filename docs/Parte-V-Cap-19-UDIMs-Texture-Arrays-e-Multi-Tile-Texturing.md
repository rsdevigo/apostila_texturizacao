# Capítulo 19 — UDIMs, Texture Arrays e Multi-Tile Texturing

## Introdução

O capítulo anterior tratou do problema de fazer **poucas texturas servirem a muitas superfícies**, reunindo imagens em atlas e trim sheets para economizar memória, trabalho e *draw calls*. Este capítulo enfrenta o problema oposto e complementar: o que fazer quando **uma única textura não basta** para uma só superfície — quando um objeto é tão importante, tão grande ou exige resolução tão alta que toda a sua superfície não cabe, com a densidade de texels desejada, numa textura de tamanho viável. Surge então a necessidade de distribuir a superfície por **múltiplas texturas coordenadas**, e é desse conjunto de técnicas que o capítulo trata: os **UDIMs**, os **texture arrays** e o **multi-tile texturing** em geral. São abordagens distintas para um mesmo desafio — gerir muitas texturas para um objeto ou um sistema —, cada uma adequada a um contexto, e compreender suas diferenças é o que permite escolher a certa.

Há aqui uma tensão produtiva que percorrerá o capítulo: a tensão entre o **fluxo de produção de cinema** e o **fluxo de jogo**. Os UDIMs nasceram na produção de efeitos visuais e animação, onde a resolução é praticamente ilimitada e a renderização não precisa ser em tempo real, e otimizam a **autoria** de objetos de altíssima resolução. Os *texture arrays*, em contrapartida, são uma estrutura voltada ao **tempo real**, que resolve problemas específicos de desempenho na GPU. O multi-tile, como conceito geral, atravessa os dois mundos. O estudante de jogos precisa entender os UDIMs porque encontrará o conceito nas ferramentas e nos *assets* vindos do fluxo de cinema, mas precisa entender por que e como esse conceito é adaptado — ou substituído — quando o destino é um motor de jogo em tempo real. Este capítulo, portanto, não apenas apresenta as técnicas, mas as situa no mapa das prioridades de cada contexto de produção, fechando o quadro das estratégias de organização de texturas iniciado no capítulo anterior.

## Desenvolvimento

### O problema: quando uma textura não basta

Para entender essas técnicas é preciso ver com clareza o problema que elas resolvem. Vimos na Parte II que a qualidade de uma superfície texturizada depende da **densidade de texels** — quantos pixels de textura cobrem cada unidade de superfície — e que essa densidade é finita: uma textura tem um tamanho máximo (limitado pela plataforma e pela memória), e quanto maior a superfície que ela cobre, menor a densidade resultante. Para a maioria dos *assets*, uma única textura de boa resolução, bem desdobrada, oferece densidade suficiente. Mas há casos em que não: um personagem principal exibido em close, cuja pele, roupas e acessórios precisam de detalhe extremo; uma criatura gigante que ocupa a tela inteira; um veículo complexo examinado de perto. Espremer toda a superfície desses objetos numa só textura forçaria uma densidade de texels baixa demais, e o detalhe se perderia.

A solução conceitual é simples: **dividir a superfície em várias partes e dar a cada uma a sua própria textura**, multiplicando a densidade de texels total disponível. Em vez de uma textura cobrindo o personagem inteiro, uma para o rosto, outra para o torso, outra para cada membro — cada uma com resolução plena, somando uma densidade de texels que uma textura única jamais alcançaria. Mas essa divisão cria um problema de **gestão**: agora há muitas texturas para um só objeto, que precisam ser nomeadas, organizadas, carregadas e aplicadas de modo coordenado, e o desdobramento UV precisa indicar a cada parte da superfície qual das texturas usar. As técnicas deste capítulo são, cada uma à sua maneira, **sistemas para gerir essa multiplicidade de texturas** de forma organizada — diferindo em como a organizam e em que contexto operam.

### UDIM: a grade de tiles do espaço UV

> 📘 **Definição**
> **UDIM** (de *U-Dimension*) é uma convenção de organização de texturas múltiplas que estende o espaço UV padrão (0–1) numa grade numerada de quadrados (*tiles*). Cada *tile* recebe sua própria textura de resolução plena, e as ilhas UV distribuídas por eles determinam automaticamente qual textura cobre cada parte da superfície.

O sistema ***UDIM*** é uma convenção elegante para organizar múltiplas texturas de um objeto, originada no fluxo de produção de cinema e hoje difundida nas ferramentas de modelagem e texturização. A ideia parte de uma observação sobre o espaço UV: como vimos na Parte II, o espaço UV padrão é um quadrado de coordenadas de 0 a 1, e todas as ilhas de um desdobramento normalmente cabem dentro desse quadrado. O UDIM estende esse espaço para uma **grade de quadrados** — chamados *tiles* — dispostos lado a lado: o primeiro *tile* (o quadrado de 0 a 1) é o UDIM 1001, o seguinte à direita é o 1002, e assim por diante, formando uma grade numerada. Cada *tile* da grade pode receber a sua **própria textura**, de resolução plena.

A elegância do sistema está em que ele transforma a gestão de muitas texturas numa simples questão de **onde** se colocam as ilhas UV. Em vez de nomear e ligar manualmente cada textura a cada parte, o artista distribui as ilhas do desdobramento pelos *tiles* da grade — o rosto no *tile* 1001, o torso no 1002, os braços no 1003 — e nomeia as texturas segundo a numeração UDIM correspondente. As ferramentas então **automaticamente** associam cada textura ao seu *tile* e, portanto, à parte da superfície cujas ilhas ali estão. Pinta-se o objeto continuamente, como se fosse uma superfície só, e o sistema cuida de gravar cada região na textura do *tile* certo. O UDIM é, assim, uma maneira de obter densidade de texels arbitrariamente alta — bastam mais *tiles* — com gestão automática e fluxo de pintura contínuo.

> **Figura 19.1** — A grade UDIM no espaço UV: uma matriz de quadrados numerados (1001, 1002, 1003... na primeira fileira; 1011, 1012... na seguinte) com as ilhas de um personagem distribuídas pelos *tiles* (rosto em um, roupa em outro, acessórios em outro), e, ao lado, as texturas correspondentes nomeadas pela numeração de cada *tile*.

**Screenshot sugerido:** Captura de uma ferramenta de texturização exibindo o editor UV com a grade UDIM e as ilhas distribuídas pelos *tiles*, ao lado da lista de texturas nomeadas por UDIM.

### UDIM no cinema e no jogo: a adaptação necessária

Aqui entra a tensão anunciada na introdução. No fluxo de **cinema** e efeitos visuais, o UDIM brilha sem ressalvas: a renderização não é em tempo real, a memória disponível é vasta, e um objeto pode ter dezenas de *tiles* de altíssima resolução sem que isso seja problema — o renderizador carrega o que precisa quando precisa, e o objetivo é a qualidade máxima de imagem, não o desempenho instantâneo. Um personagem de longa-metragem pode tranquilamente usar vinte ou mais UDIMs, cada um em resolução altíssima. A produção de *The Mandalorian* (Lucasfilm/ILM, 2019) exemplifica a fusão entre esses mundos: os *assets* dos personagens, modelados e texturizados no fluxo de cinema com múltiplos UDIMs de altíssima resolução, eram importados simultaneamente na Unreal Engine para a tecnologia de LED volume StageCraft — o que exigiu pipelines de conversão que preservassem a qualidade visual dos UDIMs dentro do orçamento de tempo real do display.

No fluxo de **jogo**, em tempo real, a situação muda. Cada *tile* UDIM é, na prática, uma textura separada, e cada textura separada implica custo de gestão, de memória e — crucialmente — de *draw call*, pois trocar de textura durante a renderização é caro. Um objeto com muitos UDIMs ativos pode forçar muitas trocas de textura e pesar no desempenho, justamente o oposto da economia que o capítulo anterior buscava.

> ⚠️ **Atenção**
> Levar ingenuamente um *asset* de muitos UDIMs, pensado para cinema, direto a um motor de jogo é um dos erros mais comuns na transição entre os dois fluxos. Cada *tile* UDIM é uma textura separada com custo de *draw call*. Em jogos, o UDIM deve ser tratado como comodidade de **autoria**, a ser consolidada antes de entrar no motor.

Por isso, embora o UDIM seja amplamente usado como **ferramenta de autoria** em jogos — pinta-se o personagem em UDIMs pela comodidade e pela densidade de texels —, o resultado costuma ser **adaptado** antes de entrar no motor: reúnem-se os *tiles* num atlas, reduz-se o número de texturas, ou usam-se estruturas próprias do tempo real como os *texture arrays*. O UDIM, em jogos, é mais uma conveniência de produção do que uma estrutura de renderização: serve para autorar com densidade e fluxo contínuo, mas o que o motor recebe é frequentemente uma forma consolidada.

### Texture arrays: a estrutura do tempo real

> 📘 **Definição**
> **Texture array** (arranjo de texturas) é uma estrutura nativa do hardware gráfico moderno que empilha num único objeto várias texturas do mesmo tamanho e formato, acessadas pelo sombreador por um índice numérico, sem custo de troca de textura entre elas. É a solução do tempo real para o problema de ter muitas texturas uniformes disponíveis simultaneamente.

O ***texture array*** é uma estrutura nativa do *hardware* gráfico moderno, voltada ao tempo real, que responde a um problema diferente do UDIM. Um *texture array* é um **conjunto de texturas do mesmo tamanho e formato empilhadas numa única estrutura**, que a GPU trata como uma unidade e à qual o sombreador acessa por um **índice** — "use a textura número 3 deste arranjo". A diferença decisiva em relação a ter muitas texturas soltas é que o *texture array* é **uma única ligação** (*binding*) para a placa de vídeo: o sombreador pode escolher, por índice e ponto a ponto, qual das texturas empilhadas usar, **sem trocar de textura** e sem forçar nova *draw call*. É a maneira do tempo real de ter muitas texturas disponíveis ao mesmo tempo sem pagar o custo de alternar entre elas.

Os usos do *texture array* esclarecem seu valor. O caso clássico é a **mistura de materiais em terreno**: um terreno pode precisar de muitos materiais (grama, terra, rocha, areia, neve), e misturá-los por máscaras (Capítulo 17) exigiria, com texturas soltas, ligar e alternar muitas delas; com um *texture array*, todos os materiais ficam empilhados numa estrutura, e o sombreador escolhe e mistura por índice, ponto a ponto, numa só passagem eficiente. *Far Cry 5* (Ubisoft Montreal, 2018) documentou em análises técnicas o uso de *texture arrays* com até 16 camadas de materiais de terreno — solo, rocha, neve, lama, asfalto, grama seca e outras superfícies do estado de Montana —, todas misturadas em tempo real por mapas de peso ponto a ponto numa única ligação de arranjo, sem custo de troca de textura. Outro uso é dar **variação a objetos repetidos** — uma multidão em que cada personagem lê, por índice, uma variante de textura do mesmo arranjo, ganhando diversidade sem multiplicar ligações.

> 💡 **Dica Profissional**
> O *texture array* exige que todas as texturas empilhadas tenham o mesmo tamanho e formato. Isso é simultaneamente sua força (garante eficiência na GPU) e sua limitação (não serve para reunir imagens heterogêneas). Planejar os materiais de terreno já em tamanhos e formatos uniformes, desde a autoria, elimina retrabalho na hora de montar o arranjo.

> **Figura 19.2** — Esquema de um *texture array*: várias texturas de terreno (grama, terra, rocha, areia) empilhadas numa única estrutura acessada por índice, contrastadas com o mesmo conjunto como texturas soltas exigindo múltiplas ligações; ao lado, um terreno em que o sombreador mistura, ponto a ponto, os materiais do arranjo por máscara.

**Screenshot sugerido:** Captura de um motor de jogo mostrando a configuração de um terreno com *texture array* de camadas e a mistura resultante, evidenciando a única ligação para muitos materiais.

### Multi-tile como conceito: escolher a estratégia certa

Reunindo as técnicas, o ***multi-tile texturing*** — texturizar com múltiplos *tiles* ou texturas coordenadas — revela-se menos uma técnica única e mais uma **família de estratégias** para o mesmo fim: dar a um objeto ou sistema mais densidade de texels e mais variedade do que uma textura caberia. O UDIM é a estratégia de **autoria**, organizando a pintura de altíssima resolução numa grade automática, ideal no fluxo de cinema e na criação de *assets* de alto orçamento. O *texture array* é a estratégia de **renderização em tempo real**, empilhando texturas uniformes numa estrutura indexada e eficiente, ideal para mistura de materiais em terreno e variação. E o **atlas** do capítulo anterior é a estratégia de **consolidação**, reunindo muitas imagens numa só textura para economizar *draw calls*. As três não competem: combinam-se ao longo do *pipeline* — pinta-se em UDIM, consolida-se em atlas ou converte-se em *texture array* para o motor — conforme o estágio e o objetivo.

A competência que o capítulo busca formar é, portanto, a de **escolher e converter** entre essas estratégias conforme o contexto. Diante de um *asset*, o profissional pergunta: qual densidade de texels ele exige, e por quê (proximidade, importância, tamanho)? Qual o destino — cinema, sem restrição de tempo real, ou jogo, com orçamento de *draw calls*? O sistema precisa de variação por índice em tempo de renderização, como um terreno ou uma multidão? As respostas indicam a estratégia: UDIM para autorar com densidade, atlas para consolidar e economizar chamadas, *texture array* para muitas texturas uniformes em tempo real. E, muitas vezes, a resposta é uma combinação encadeada, em que o *asset* migra de uma forma a outra ao longo da produção.

## Aplicação em Jogos

Na prática de jogos, essas técnicas aparecem em contextos bem definidos. O UDIM é usado sobretudo na **autoria de personagens e criaturas de alto orçamento**, onde a densidade de texels precisa ser alta e a pintura contínua é uma comodidade valiosa; o artista pinta o personagem em vários *tiles* no Substance Painter ou no Mari, e a equipe decide depois como consolidar para o motor — frequentemente reduzindo o número de *tiles* ou reunindo-os, pois muitos UDIMs ativos pesam no tempo real. Os *texture arrays* aparecem onde há **muitas texturas uniformes a misturar ou variar em tempo real**: são a base dos sistemas de terreno dos motores modernos, que misturam dezenas de camadas de material por arranjo indexado, e servem também à variação de vegetação, multidões e detritos. E o multi-tile, como decisão de projeto, está presente sempre que um *asset* importante exige mais do que uma textura comportaria.

A escolha entre essas técnicas é governada, como tudo nesta parte, pelo orçamento da plataforma e pelo destino do *asset*. Um jogo de alto orçamento para console e PC pode permitir-se personagens com vários UDIMs consolidados em poucas texturas de alta resolução; um jogo para celular evitará a multiplicação de texturas, preferindo atlas e densidades mais modestas. Os terrenos de mundo aberto dependem de *texture arrays* para misturar muitos materiais sem afogar o desempenho em *draw calls*. E os motores de jogo oferecem ferramentas para a conversão entre formas — empacotadores de atlas, importadores que montam *texture arrays*, suporte a UDIM na importação — que o texturizador precisa conhecer para mover seus *assets* entre o fluxo de autoria e o de renderização.

## Estudo de Caso

### *The Last of Us Part II* (Naughty Dog, 2020) — Personagens em Alta Resolução e a Gestão de Multi-Tile

*The Last of Us Part II* elevou o padrão técnico de personagens em jogos de console ao ponto em que o processo de produção de Ellie, Joel e os demais protagonistas é rotineiramente citado em discussões sobre texturização de altíssima resolução. A Naughty Dog documentou aspectos do pipeline em apresentações técnicas e no evento SIGGRAPH 2020, revelando que os personagens principais são autorizados com densidades de texels muito acima do que uma única textura comportaria — necessidade imposta pelo peso dramático das sequências em close que compõem boa parte do jogo. Pele com poros visíveis, cicatrizes com microestrutura legível, roupas com fios e desgaste de tecido detectáveis ao espectador: cada detalhe exige que a superfície seja dividida em regiões, cada uma com sua própria textura em alta resolução, para atingir a densidade necessária.

A abordagem da equipe ilustra com precisão a tensão entre autoria e renderização descrita neste capítulo. No fluxo de criação, os personagens são desenvolvidos com ferramentas que permitem trabalhar em múltiplas regiões de textura simultaneamente — equivalente funcional do UDIM —, distribuindo rosto, pescoço, tronco, membros e acessórios em áreas distintas, cada uma com resolução independente. Isso libera os artistas para pintar o personagem de forma contínua e intuitiva, sem se preocupar em comprimir todo o detalhe num único espaço UV. A comodidade de autoria é decisiva na produção de um jogo com a densidade de personagens de *The Last of Us Part II*, onde dezenas de NPCs também exigem tratamento de alta qualidade.

O que o estudo de caso demonstra com clareza é a etapa de adaptação para o tempo real: a Naughty Dog consolidou e priorizou texturas conforme a importância de cada região e sua frequência de aparição em close. O rosto de Ellie, centro das sequências dramáticas e constantemente visto de perto, recebe densidade muito superior à de zonas raramente vistas de perto, como a parte posterior das pernas ou as solas dos sapatos. Essa distribuição deliberada de resolução — mais textura onde o olhar do jogador pousa, menos onde não pousa — é a mesma lógica da densidade de texels da Parte II aplicada ao nível do multi-tile: a densidade não é uniforme, é proporcional à importância.

**O que aprender com isso:** A gestão de multi-tile madura não é apenas técnica, mas editorial. A densidade de texels é uma decisão de projeto, distribuída de forma proporcional à importância de cada região, e não um valor uniforme aplicado cegamente a toda a superfície.

> **Figura 19.3** — Diagrama do personagem de *The Last of Us Part II* com suas regiões de textura anotadas e suas densidades de texels relativas, mostrando a concentração de resolução no rosto e nas mãos e a redução progressiva nas zonas periféricas — a distribuição proporcional à importância que este capítulo defende.

**Screenshot sugerido:** Captura de close do rosto de Ellie em *The Last of Us Part II* evidenciando a microestrutura de pele, cicatrizes e cabelo — qualidade obtida pela alta densidade de texels possibilitada pelo multi-tile. Fonte: modo foto do jogo (PS4/PS5) ou galeria oficial da PlayStation.

## Boas Práticas

A boa prática que organiza o capítulo é **diagnosticar o problema antes de escolher a técnica**: perguntar qual densidade de texels o *asset* exige e por quê, qual seu destino (cinema sem restrição ou jogo em tempo real) e se há necessidade de variação ou mistura por índice em tempo de renderização.

> 💡 **Dica Profissional**
> Use UDIMs para autorar *assets* de altíssima resolução com pintura contínua e densidade de texels arbitrária, aproveitando a gestão automática da grade. Mas trate-os como comodidade de **autoria**, não como forma final de renderização. Ao levar o *asset* a um motor de jogo, consolide conscientemente — reduzindo *tiles*, reunindo em atlas ou mantendo alta resolução apenas onde o jogador olha.

Use **texture arrays** quando precisar de muitas texturas **uniformes** disponíveis simultaneamente em tempo real sem custo de troca — mistura de materiais em terreno, variação de objetos repetidos —, aproveitando a ligação única e o acesso por índice; lembre que o arranjo exige texturas do mesmo tamanho e formato e não serve a imagens heterogêneas. Pense o multi-tile como uma **família encadeada** de estratégias, e domine a conversão entre elas — autorar em UDIM, consolidar em atlas, montar em *texture array* — em vez de tratar qualquer uma como solução única. E, em tudo, deixe o **orçamento da plataforma e a proximidade do *asset*** governarem a decisão.

## Erros Comuns

> ❌ **Erro Comum**
> Levar um *asset* de muitos UDIMs, pensado para cinema, direto a um motor de jogo sem consolidação multiplica texturas e *draw calls*, afundando o desempenho. O UDIM é comodidade de autoria, não estrutura de renderização para tempo real.

O erro mais característico deste capítulo é levar **ingenuamente um *asset* de muitos UDIMs, pensado para cinema, direto a um motor de jogo**, multiplicando texturas e trocas e afundando o desempenho — por não distinguir o UDIM como comodidade de autoria da estrutura de renderização que o tempo real exige. Seu oposto é evitar o UDIM por completo na autoria de *assets* que de fato precisam de altíssima densidade, espremendo um personagem de close numa única textura e perdendo detalhe — recusar a ferramenta certa para a autoria por receio de seu custo na renderização, quando a solução é autorar em UDIM e consolidar depois.

> ❌ **Erro Comum**
> Tentar empilhar texturas de tamanhos ou formatos diferentes num *texture array* — ignorando que o arranjo exige uniformidade — resulta em erros de criação do arranjo ou em texturas incompatíveis. Planejar o tamanho e formato de todos os materiais do sistema desde o início evita esse problema.

Há o erro de confundir os papéis das técnicas: usar *texture array* onde caberia um atlas (forçando uniformidade desnecessária a imagens heterogêneas) ou tentar misturar terreno com texturas soltas onde um *texture array* resolveria o custo de troca. E persiste o erro de fundo, comum a toda esta parte: tratar a densidade de texels como gratuita, multiplicando *tiles* e resolução sem consciência do custo de memória e de *draw call*, esgotando o orçamento da plataforma com detalhe que o jogador, à distância em que vê o objeto, sequer percebe.

## Resumo

Este capítulo tratou do problema oposto ao do anterior: o que fazer quando uma única textura não basta para uma só superfície, por exigência de densidade de texels alta demais para um tamanho de textura viável. A solução conceitual — dividir a superfície em partes, cada uma com sua textura, multiplicando a densidade total — cria um problema de gestão de muitas texturas, e as técnicas do capítulo são sistemas para geri-la. Apresentamos o **UDIM** como a convenção que estende o espaço UV numa grade numerada de *tiles*, cada um com sua textura, permitindo autorar com densidade de texels arbitrária e pintura contínua e gestão automática — padrão no fluxo de cinema e na autoria de *assets* de alto orçamento. Enfatizamos a tensão entre cinema e jogo: o UDIM brilha onde não há restrição de tempo real, mas, em jogos, cada *tile* é uma textura com custo de *draw call*, de modo que o UDIM é comodidade de **autoria** a ser **consolidada** antes de entrar no motor. Apresentamos o **texture array** como a estrutura do tempo real — texturas uniformes empilhadas numa ligação única, acessadas por índice sem custo de troca —, ideal para mistura de materiais em terreno e variação de objetos repetidos, com a contrapartida de exigir uniformidade. E reunimos tudo sob o **multi-tile** como família de estratégias — UDIM para autorar, atlas para consolidar, *texture array* para renderizar — que se encadeiam ao longo do *pipeline*, cabendo ao profissional diagnosticar o problema e escolher e converter entre elas conforme densidade exigida, destino e necessidade de variação.

## Exercícios

**1.** Explique o problema que UDIMs, *texture arrays* e multi-tile vêm resolver, relacionando-o ao conceito de densidade de texels da Parte II. Por que, para certos *assets*, dividir a superfície em várias texturas é preferível a usar uma única textura grande?

**2.** Descreva o sistema UDIM: como ele estende o espaço UV, como as ilhas são distribuídas pelos *tiles* e como a numeração automatiza a associação entre texturas e partes da superfície. Explique por que ele é especialmente valioso para a autoria de *assets* de altíssima resolução.

**3.** Análise crítica: o capítulo afirma que, em jogos, "o UDIM é mais uma comodidade de produção do que uma estrutura de renderização". Explique essa afirmação, contrastando o fluxo de cinema com o de jogo, e descreva o que significa "consolidar" um *asset* de UDIMs antes de levá-lo a um motor em tempo real.

**4.** Comparação: distinga o *texture array* do *texture atlas* (Capítulo 18) quanto à estrutura, ao requisito de uniformidade das imagens, à forma de acesso pelo sombreador e ao tipo de problema que cada um resolve melhor. Dê um exemplo de uso adequado para cada.

**5.** Planejamento de pipeline: você precisa texturizar, para um jogo em tempo real, (a) um chefe gigante visto sempre de muito perto e (b) um terreno de mundo aberto que mistura oito materiais de solo. Descreva, em texto contínuo, qual estratégia de multi-tile usaria para cada caso e por quê, indicando como autoraria e como adaptaria cada *asset* para o motor, e justificando as escolhas pelo orçamento de *draw calls* e de memória.

## Glossário

**Atlas (de texturas):** Técnica de consolidação que reúne muitas imagens de tamanhos variados numa única textura, reduzindo o número de ligações e de *draw calls*. Discutido em detalhe no Capítulo 18.

**Densidade de texels:** Relação entre o tamanho físico de uma superfície no espaço 3D e a quantidade de pixels de textura que a recobre. Determina o nível de detalhe visível e governa a decisão de usar multi-tile quando uma textura única seria insuficiente.

**Draw call:** Comando enviado pela CPU à GPU para renderizar um objeto. Cada troca de textura durante a renderização pode forçar uma nova *draw call*, tornando custosa a multiplicação descontrolada de texturas.

**Espaço UV:** Plano bidimensional de coordenadas (U horizontal, V vertical) que indexa a posição de cada ponto da superfície de uma malha numa textura. No UDIM, esse espaço é estendido para uma grade de *tiles* além do quadrado padrão de 0 a 1.

**Ilhas UV:** Regiões contíguas da superfície de uma malha, separadas por costuras no desdobramento UV, que aparecem como áreas planas e delimitadas no espaço UV.

**Mapeamento UV:** Processo de associar pontos da superfície tridimensional de uma malha a coordenadas bidimensionais (U e V) que indexam sua posição numa textura.

**Motor de jogo:** Sistema de software que gerencia a renderização, a física, a lógica e os demais aspectos de um jogo em tempo real. A estrutura de importação de texturas e os sistemas de terreno dos motores de jogo determinam como UDIMs e *texture arrays* são suportados.

**Multi-tile texturing:** Família de técnicas que distribuem a superfície de um objeto ou sistema por múltiplas texturas coordenadas, para atingir densidade de texels ou variedade superior ao que uma única textura ofereceria.

**Texture array (arranjo de texturas):** Estrutura nativa do hardware gráfico que empilha texturas de mesmo tamanho e formato numa única ligação, permitindo ao sombreador escolher entre elas por índice sem custo de troca de textura.

**Tile:** Cada quadrado da grade UDIM no espaço UV, identificado por um número (1001, 1002...), que pode receber sua própria textura de resolução plena.

**UDIM (U-Dimension):** Convenção de organização de múltiplas texturas que estende o espaço UV padrão numa grade de *tiles* numerados, automatizando a associação entre cada textura e a parte da superfície cujas ilhas UV estão naquele *tile*.

## Leituras Complementares

- **[AOD — Texel Density]** — Fundamenta a relação entre densidade de texels, tamanho de textura e proximidade que justifica recorrer a múltiplas texturas (UDIM, multi-tile) quando uma só não basta. Base conceitual indispensável antes de escolher entre as estratégias deste capítulo.
- **[The PBR Guide]** (Wes McDermott; Allegorithmic / Adobe), versão 2018 — Contextualiza a autoria de materiais de alta qualidade no fluxo de ferramentas que adotam UDIM, útil para situar a pintura em múltiplos *tiles* dentro de um fluxo PBR completo.
- **[Adobe Substance 3D]** — https://helpx.adobe.com/substance-3d.html. Documentação do Substance Painter sobre o fluxo de trabalho com UDIM (*tiles*, pintura contínua, exportação por *tile*), referência direta da seção sobre autoria em UDIM e ponto de partida para praticar a técnica.
- **[Unreal Engine Documentation]** — https://dev.epicgames.com/documentation/unreal-engine/. Seções sobre suporte a *UDIM textures*, *texture arrays* e os sistemas de *Landscape*/terreno com mistura de camadas; para observar como o motor importa e consolida *assets* multi-tile e implementa a mistura por arranjo.
- **[Unity Manual]** — https://docs.unity3d.com/. Seções sobre *Texture2DArray*, o sistema de *Terrain* e a mistura de camadas, e o suporte a UDIM, para comparar a implementação do tempo real em outro motor.
- **[Real-Time Rendering]** — AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. 4. ed. Boca Raton: CRC Press, 2018. Os capítulos sobre *texturing*, arranjos de textura e arquitetura de GPU fundamentam tecnicamente os *texture arrays* e a economia de ligações discutida neste capítulo.
