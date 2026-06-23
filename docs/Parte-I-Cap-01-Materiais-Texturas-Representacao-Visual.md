# Capítulo 1 — Materiais, Texturas e a Representação Visual em Computação Gráfica

## Introdução

Quando um jogador olha para uma espada enferrujada caída no chão de uma masmorra, para o couro gasto de uma jaqueta ou para o reflexo úmido de uma rua após a chuva, ele raramente percebe que está observando, na realidade, uma elaborada ilusão construída a partir de números. Nada daquilo é metal, couro ou água. São apenas coordenadas, vetores e imagens bidimensionais interpretadas, em tempo real, por um conjunto de programas que decidem qual cor cada ponto da tela deve assumir. A disciplina de Texturização é, em essência, o estudo de como essa ilusão é planejada, construída e controlada.

Este primeiro capítulo estabelece o vocabulário e os conceitos sem os quais o restante da disciplina não pode ser compreendido. Três palavras, frequentemente confundidas entre si na linguagem cotidiana e mesmo entre estudantes iniciantes, serão cuidadosamente separadas: **material**, **textura** e **texturização**. Cada uma designa algo distinto, e a confusão entre elas é a origem de boa parte dos erros conceituais que acompanham quem está começando. Mais do que definir termos, o objetivo aqui é construir um modelo mental correto sobre o que significa "dar aparência" a uma superfície tridimensional, modelo esse que será aprofundado, refinado e questionado ao longo de toda a apostila.

Parte-se de um pressuposto simples, mas que merece ser tornado explícito: em computação gráfica, a aparência de um objeto não é uma propriedade intrínseca dele, e sim o resultado de uma simulação de como a luz interage com sua superfície. Compreender texturização significa, portanto, compreender que estamos sempre manipulando dados que alimentam um cálculo de iluminação, e não "pintando" objetos no sentido literal do termo.

## Desenvolvimento

### Do que é feito um modelo tridimensional

Antes de falar de aparência, é preciso entender sobre o que essa aparência será aplicada. Um modelo tridimensional, na quase totalidade das aplicações de jogos, é composto por uma malha poligonal. Essa malha é um conjunto de **vértices** — pontos definidos por coordenadas em um espaço tridimensional, descrito pelos eixos X, Y e Z — conectados por **arestas**, que por sua vez delimitam **faces**. Embora muitos softwares de modelagem permitam trabalhar com faces de quatro lados, chamadas de quads, o hardware gráfico em última instância converte tudo em **triângulos**, pois o triângulo é a única figura plana garantidamente convexa e não ambígua: três pontos sempre definem um único plano. Essa é uma primeira lição que se repetirá ao longo do curso: aquilo que o artista manipula nem sempre corresponde àquilo que a máquina efetivamente processa.

A malha, sozinha, é apenas uma estrutura geométrica invisível. Para que ela apareça na tela, o objeto precisa estar associado a algo que descreva como sua superfície reage à luz. Esse "algo" é o material.

### O material: a descrição do comportamento da superfície

O **material** é a camada de informação que define como a superfície de um objeto se comporta diante da luz. É o material que determina se uma superfície parecerá macia como um tecido, dura e fosca como concreto, ou lisa e reflexiva como um espelho. O material não é a imagem que vemos: é o conjunto de regras e propriedades que, combinadas com as luzes da cena, produzem a imagem que vemos.

Convém pensar no material como uma receita. Uma receita lista ingredientes e proporções, mas não é, ela própria, o prato pronto. Da mesma forma, o material reúne propriedades como cor base, rugosidade, grau de metalicidade e relevo aparente, e entrega esse conjunto ao mecanismo de renderização, que então "cozinha" a imagem final levando em conta a posição das luzes, da câmera e dos demais objetos da cena. Por isso o mesmo material pode produzir resultados visuais muito diferentes sob iluminações diferentes — um aspecto que se tornará central quando, no Capítulo 3, discutirmos renderização baseada em física.

É importante notar que um material pode ser inteiramente uniforme. Um material que define "vermelho fosco" sem nenhuma variação produzirá uma superfície de cor única e sem detalhe, semelhante a um objeto recém-pintado de fábrica. O mundo real, no entanto, raramente é assim: superfícies têm manchas, arranhões, variações de cor, sujeira acumulada nas frestas e desgaste nas quinas. É exatamente para introduzir essa riqueza de variação que entram as texturas.

### A textura: variação ponto a ponto

Uma **textura** é, na maioria dos casos, uma imagem bidimensional cujo propósito é quebrar a uniformidade de uma superfície, fornecendo um valor diferente para cada ponto dela. Por isso a textura também é chamada de **mapa de textura**: ela funciona como um mapa que, para cada local da superfície, indica qual valor uma determinada propriedade do material deve assumir naquele ponto.

Aqui reside um dos conceitos mais importantes — e mais negligenciados — do início da disciplina: **uma textura não precisa conter cores no sentido visual**. Embora a textura mais intuitiva seja aquela que descreve a cor da superfície, uma imagem é, no fundo, apenas uma grade de números. Esses números podem representar cor, mas também podem representar rugosidade, altura, transparência, direção de relevo ou qualquer outra propriedade que o material saiba interpretar. Uma textura é, portanto, um meio genérico de armazenar variação espacial de dados. Essa generalidade é o que torna a texturização tão poderosa, e será explorada exaustivamente quando estudarmos os diferentes tipos de mapas no contexto da renderização baseada em física.

A menor unidade de uma textura é o **texel**, contração de *texture element*, em analogia ao pixel (*picture element*) das imagens comuns. Um texel corresponde a uma célula da grade da textura e armazena o valor — ou conjunto de valores — daquele ponto. A distinção terminológica entre pixel e texel não é mero preciosismo: ela ajuda a manter clara a diferença entre o que está armazenado na imagem da textura (texels) e o que aparece, ao final, na tela do jogador (pixels). Um único texel pode acabar cobrindo muitos pixels na tela, se o objeto estiver muito próximo da câmera, ou, ao contrário, vários texels podem ser comprimidos em um só pixel, se o objeto estiver distante. Essa relação entre texels e pixels é a semente de um conceito central da disciplina, a densidade de texels, que será tratada em profundidade na parte dedicada ao mapeamento UV.

### A texturização: o ato de conectar os dois mundos

Definidos o material (o comportamento da superfície) e a textura (a variação ponto a ponto), resta o elo que os une à geometria. Esse elo é a **texturização**, propriamente dita.

A texturização é a técnica de síntese de imagens que consiste em mapear cada ponto da superfície tridimensional, descrito por coordenadas (X, Y, Z), a um ponto correspondente no mapa de textura, descrito por coordenadas **(U, V)**. Usam-se as letras U e V justamente para não confundir as coordenadas da textura, que vivem em um espaço bidimensional, com as coordenadas X, Y e Z do espaço tridimensional onde está o modelo. Esse processo é também conhecido como **mapeamento UV**, e o conjunto de coordenadas UV associado à malha é o que permite que uma imagem plana seja "vestida" sobre uma forma tridimensional, da mesma maneira que um molde de costura, recortado em pano plano, dá origem a uma roupa que envolve um corpo.

Uma vez estabelecido esse mapeamento, os valores lidos da textura em cada ponto podem ser usados para modificar localmente as propriedades do material. É essa modificação local que permite enriquecer enormemente o detalhe visual de um objeto **sem aumentar a complexidade de sua geometria**. Trata-se de uma das ideias mais economicamente importantes de toda a computação gráfica em tempo real: em vez de modelar cada arranhão, cada poro e cada grão de madeira como geometria — o que exigiria um número proibitivo de vértices — descreve-se esse detalhe como variação registrada em uma imagem, que o material consulta a baixo custo. O detalhe deixa de ser forma e passa a ser informação pintada sobre a forma.

### A cadeia conceitual

Vale consolidar a relação entre os três termos em uma cadeia. A **geometria** fornece a forma. O **material** descreve como essa forma reage à luz. As **texturas** fornecem, ponto a ponto, os valores que alimentam as propriedades do material. E a **texturização**, por meio do mapeamento UV, é o que amarra cada ponto da forma a um ponto das texturas. Quando essa cadeia funciona em harmonia, o mecanismo de renderização consegue calcular, para cada pixel da tela, uma cor que convence o olho de estar diante de metal, madeira ou pele.

> Figura: Diagrama da cadeia conceitual mostrando, da esquerda para a direita, quatro blocos conectados por setas — "Geometria (malha de triângulos)", "Coordenadas UV (mapeamento)", "Texturas (mapas 2D)" e "Material (regras de iluminação)" — convergindo para um quadro final rotulado "Imagem renderizada na tela".
>
> **Screenshot sugerido:**
> Captura de um software 3D (por exemplo, Blender) mostrando, lado a lado, o mesmo objeto: à esquerda apenas a malha em wireframe; ao centro o objeto com material uniforme; à direita o objeto com texturas aplicadas. Anotar cada estágio com seu nome.

## Aplicação em Jogos

A distinção entre material, textura e texturização não é apenas teórica: ela estrutura o modo como o trabalho é dividido e otimizado na produção de jogos. Em um jogo, cada milissegundo de processamento e cada megabyte de memória de vídeo são recursos disputados. Compreender que o detalhe pode ser armazenado em texturas, e não em geometria, é o que viabiliza mundos visualmente densos rodando a taxas de quadros aceitáveis em hardware doméstico.

Considere um objeto comum de cenário, como um barril de madeira reforçado por aros de metal. Modelar geometricamente cada ripa, cada rebite e cada lasca seria inviável para um objeto que aparecerá dezenas de vezes em um nível. Em vez disso, modela-se uma forma simples — pouco mais que um cilindro — e descreve-se todo o detalhe por meio de texturas: a cor da madeira e do metal, o quanto cada região reflete a luz, e o relevo aparente das ripas e rebites. O resultado, observado em jogo, parece infinitamente mais complexo do que a geometria que o sustenta. Essa é a aplicação mais elementar e mais onipresente dos conceitos deste capítulo.

Há ainda uma consequência organizacional. Como o material concentra as regras de comportamento e as texturas concentram a variação, equipes podem reaproveitar um mesmo material para muitos objetos, trocando apenas as texturas, ou compartilhar texturas genéricas (como um padrão de sujeira) entre materiais distintos. Essa modularidade, que será desenvolvida nas partes seguintes da disciplina, nasce diretamente da separação conceitual estabelecida aqui.

## Estudo de Caso

### Half-Life 2 e o Sistema de Materiais do Source Engine — Valve, 2004

![Corredor da Estação de Ravenholm em Half-Life 2, com superfícies de concreto, metal e tijolo respondendo à iluminação dinâmica de uma tocha](imagens/cap01-half-life2-ravenholm.jpg)

<!-- Imagem: captura de tela da Valve, disponível em https://www.half-life.com/en/halflife2 ou via Steam (presskit oficial) -->

Em 2004, quando a maioria dos jogos ainda colava fotografias de superfícies diretamente sobre a geometria, a Valve lançou *Half-Life 2* com um sistema de materiais que tratava cada superfície como um conjunto de propriedades independentes, não como uma imagem. O mecanismo central era o arquivo `.vmt` (*Valve Material Type*) — um arquivo de texto legível que definia o comportamento da superfície: qual textura continha a cor, qual continha o relevo, qual descrevia a rugosidade especular, e como essas entradas deveriam ser combinadas pelo shader. Uma parede de concreto, por exemplo, podia ter sua textura de cor trocada por outra sem afetar em nada o comportamento de reflexo ou o relevo — porque cada propriedade vivia em um arquivo e em um canal separado.

O efeito desta separação tornou-se imediatamente visível nos cenários do jogo. Os corredores da estação de Ravenholm, os esgotos sob City 17 e as fachadas das torres da Cidadela compartilhavam uma característica que surpreendia o jogador de 2004 sem que ele soubesse nomear: as superfícies *respondiam* à luz. Quando Gordon Freeman acendia a lanterna num corredor escuro, o concreto úmido brilhava diferente do concreto seco, o metal reagia de modo diferente do tijolo, e as juntas entre as pedras mergulhavam em sombra enquanto as faces salientes captavam a luz — tudo calculado em tempo real, sem uma única sombra pintada. A Valve havia operacionalizado, anos antes da formalização do PBR, o princípio fundamental que este capítulo apresenta: **aparência é o produto de dados e luz, não uma imagem estática**.

Jason Mitchell, Moby Francke e Dhabih Eng documentaram o sistema em detalhes na GDC 2006, no talk *"Shading in Valve's Source Engine"*. O paper descreve como o Source Engine separou o modelo de material em entradas distintas — diffuse, normal, specular, envmask — e como essa separação permitiu que ambientes enormes fossem texturizados de modo coerente por equipes diferentes, pois as regras de comportamento estavam no `.vmt` e as texturas eram apenas dados. A documentação técnica completa do sistema está disponível até hoje na Valve Developer Community Wiki (`developer.valvesoftware.com`), tornando *Half-Life 2* um dos raros jogos da indústria cuja arquitetura interna de materiais pode ser estudada linha a linha.

> **Figura 1.1** — Corredor de Ravenholm em *Half-Life 2* com a lanterna de Gordon Freeman iluminando superfícies de tijolo, concreto e metal. Observe que cada material reage de modo diferente à mesma fonte de luz: o tijolo escatola difusamente, o metal nas dobradiças forma um pequeno realce brilhante, e as juntas entre os tijolos mergulham em sombra — comportamentos gerados pelos arquivos `.vmt` e suas texturas de propriedade, não por sombras pintadas.
>
> **Fonte da imagem:** Valve Corporation — material de press kit de *Half-Life 2* (2004). Disponível em: `https://store.steampowered.com/app/220/HalfLife_2/` (capturas de tela oficiais na página da Steam).

## Boas Práticas

A boa prática fundamental decorrente deste capítulo é manter rigor terminológico. Falar com precisão sobre material, textura e texturização não é formalismo acadêmico: é a base para se comunicar dentro de uma equipe, ler documentação técnica e diagnosticar problemas. Quando um membro da equipe diz "o material está errado", a equipe precisa saber se ele se refere às regras de comportamento da superfície ou à imagem aplicada sobre ela.

Recomenda-se também, desde o início, adotar o hábito mental de perguntar, diante de qualquer detalhe visual desejado, se aquele detalhe deve ser forma (geometria) ou informação (textura). Essa pergunta orienta decisões de orçamento de polígonos e de memória que acompanharão o profissional por toda a carreira. Outra prática saudável é evitar embutir informação de iluminação nas texturas de cor: como o estudo de caso evidenciou, sombras pintadas brigam com a iluminação dinâmica da cena. A cor base deve, idealmente, descrever apenas a coloração própria da superfície, deixando o cálculo de luz a cargo do material e do mecanismo de renderização.

Por fim, vale cultivar desde cedo a consciência da relação entre texels e pixels. Mesmo sem ainda dominar a densidade de texels, o estudante deve perceber que a quantidade de detalhe perceptível em jogo depende de quantos texels cobrem cada porção da superfície vista pela câmera, e não apenas da resolução nominal da imagem de textura.

## Erros Comuns

O erro conceitual mais frequente é tratar "textura" e "material" como sinônimos. Essa confusão leva o iniciante a acreditar que aplicar uma imagem a um objeto basta para que ele pareça realista, ignorando que é o material, em diálogo com a luz, quem produz a aparência final. Um erro aparentado é supor que toda textura contém cores; como vimos, muitas das texturas mais importantes armazenam dados não visuais, como rugosidade ou relevo, e abri-las esperando ver uma imagem "bonita" só gera confusão.

Outro equívoco recorrente é confundir textura com texturização. A textura é o dado; a texturização é o processo de mapear esse dado sobre a geometria. Dizer "vou texturizar" quando se quer dizer "vou criar uma imagem de textura" embaralha duas etapas distintas do trabalho. Igualmente comum é tentar resolver com geometria aquilo que deveria ser resolvido com textura — modelar detalhes minúsculos que poderiam ser pintados em um mapa de relevo —, desperdiçando orçamento de polígonos. O erro inverso também ocorre: tentar representar com textura aquilo que precisaria de geometria real, esperando que um mapa de relevo crie silhuetas e saliências que ele, por sua natureza, não é capaz de produzir. O equilíbrio entre forma e informação é uma das decisões mais delicadas do ofício, e voltaremos a ela repetidamente.

## Resumo

Este capítulo estabeleceu o tripé conceitual da disciplina. A **geometria** define a forma de um objeto por meio de uma malha de vértices, arestas e faces, convertida em triângulos pelo hardware. O **material** descreve como a superfície reage à luz, funcionando como uma receita de propriedades que só produz uma imagem quando combinada com a iluminação da cena. A **textura** é, em geral, uma imagem bidimensional — uma grade de texels — usada para fornecer variação ponto a ponto a essas propriedades, podendo armazenar dados visuais ou não visuais. A **texturização**, ou mapeamento UV, é o processo que associa cada ponto (X, Y, Z) da superfície a um ponto (U, V) da textura, permitindo enriquecer o detalhe sem aumentar a complexidade da malha. Acima de tudo, fixou-se a ideia de que, em computação gráfica, aparência não é uma propriedade intrínseca do objeto, mas o resultado de um cálculo que combina dados de superfície com luz. Esse princípio sustentará toda a discussão sobre renderização baseada em física no Capítulo 3.

## Exercícios

1. Explique, com suas próprias palavras e sem recorrer a definições decoradas, por que material, textura e texturização não são sinônimos. Construa um exemplo concreto, diferente do barril e da parede de tijolos usados no capítulo, em que a distinção entre os três fique evidente.

2. Considere a afirmação: "toda textura é uma imagem colorida da superfície". Critique essa afirmação, apontando em que medida ela é verdadeira e em que medida é equivocada. Dê pelo menos dois exemplos de informações não relacionadas a cor que uma textura pode armazenar.

3. Um colega propõe modelar geometricamente cada poro da pele de um personagem para aumentar o realismo. Avalie criticamente essa decisão à luz dos conceitos do capítulo. Em que situações ela poderia se justificar e em que situações seria um erro de planejamento?

4. Diferencie texel e pixel e explique por que essa distinção é relevante, mesmo que ambos pareçam, à primeira vista, a mesma coisa. Descreva uma situação de jogo em que muitos texels são comprimidos em poucos pixels e outra em que poucos texels cobrem muitos pixels.

5. Diagnóstico: uma parede de tijolos em um jogo parece convincente quando vista de frente, mas "achata" e perde realismo assim que a câmera se desloca para o lado e a luz incide obliquamente. Com base no capítulo, levante hipóteses sobre a causa do problema e proponha uma abordagem conceitual para resolvê-lo, sem recorrer a passos de software.

## Leituras Complementares

### Material de referência do projeto

- McDERMOTT, Wes. *The PBR Guide* (Allegorithmic/Adobe, ed. 2018) — fundamenta o entendimento de materiais como descrições do comportamento da superfície diante da luz, base para o Capítulo 3.
- Documentação do Blender (docs.blender.org) — para observar, na prática, como malha, material e texturas se organizam em um software de modelagem.

### Fontes complementares externas

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. O Capítulo 6, "Texturing", oferece o tratamento de referência sobre o que é uma textura, como o mapeamento associa superfície e imagem, e por que o detalhe é representado como informação e não como geometria — aprofundamento direto dos conceitos centrais deste capítulo.
- CATMULL, Edwin. *A Subdivision Algorithm for Computer Display of Curved Surfaces*. Tese de doutorado, University of Utah, 1974. Trabalho seminal frequentemente apontado como a origem do mapeamento de texturas; útil para compreender, na raiz histórica, a ideia de projetar uma imagem 2D sobre uma superfície curva.
- *Scratchapixel — Introduction to Texturing* (scratchapixel.com) — tutorial conceitual e gratuito que reconstrói, do zero, a relação entre coordenadas de superfície e coordenadas de textura, com excelente discussão sobre texel e amostragem.
- *Texture mapping* — verbete da Wikipédia (en.wikipedia.org/wiki/Texture_mapping) — visão panorâmica acessível, útil como porta de entrada antes das referências técnicas mais densas.
- Documentação da Unreal Engine (dev.epicgames.com) e da Unity (docs.unity3d.com), seções sobre materiais e texturas — para observar como dois motores distintos expõem, em seus editores, a mesma separação conceitual entre forma, material e textura.
