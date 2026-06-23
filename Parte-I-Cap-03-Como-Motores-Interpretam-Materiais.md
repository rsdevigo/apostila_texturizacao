# Capítulo 3 — Como Motores de Jogo Interpretam Materiais

## Introdução

Os dois capítulos anteriores estabeleceram *o que* é um material e *como* a indústria chegou às formas atuais de descrevê-lo. Resta a pergunta que fecha os fundamentos da disciplina: o que, exatamente, um motor de jogo faz com tudo isso? Quando o artista entrega uma geometria, um conjunto de texturas e um material, o que acontece nos bastidores para que, milhões de vezes por segundo, o motor decida a cor de cada pixel da tela?

Este capítulo abre a "caixa-preta" da renderização em tempo real, mas o faz no nível conceitual, sem se prender a uma interface ou a um software específico. O objetivo é que o estudante entenda o motor como um intérprete: ele recebe dados — vértices, coordenadas UV, mapas de textura e parâmetros de material — e produz uma imagem segundo regras bem definidas. A maior dessas regras, hoje, é o conjunto de princípios da renderização baseada em física, o PBR, que foi anunciado historicamente no capítulo anterior e que agora examinaremos por dentro. Compreender como o motor interpreta os materiais é o que transforma o artista de alguém que "tenta até parecer bom" em alguém que entende por que algo parece bom — e por que, quando não parece, sabe onde está o erro.

Recuperaremos aqui o princípio fundamental do Capítulo 1: aparência é o resultado de um cálculo que combina dados de superfície com luz. Este capítulo é, em essência, o estudo desse cálculo e dos dados que o alimentam.

## Desenvolvimento

### O shader: o programa que pinta cada ponto

No coração da interpretação de materiais está o **shader**. Um shader é um pequeno programa, executado pelo processador gráfico (a GPU), que define as regras pelas quais cada ponto de uma superfície será convertido em cor na tela. Ele recebe como entrada a geometria, as texturas, os parâmetros do material e as luzes da cena, e produz como saída a cor final de cada pixel. Tudo o que discutimos sob o nome de "material" é, em última análise, alimentado a um shader, que executa o cálculo de iluminação propriamente dito.

A enorme eficiência do hardware gráfico vem de sua capacidade de executar esse mesmo programa para um número imenso de pontos em paralelo. Enquanto o processador central, a CPU, é otimizado para executar instruções variadas em sequência, a GPU é construída para repetir o mesmo cálculo simples sobre milhões de elementos simultaneamente — exatamente o que a renderização exige. É essa arquitetura que torna possível avaliar, dezenas de vezes por segundo, a interação entre luz e superfície em cada ponto visível da cena. Quando o Capítulo 1 afirmou que aparência é cálculo, é a este cálculo, executado pelo shader na GPU, que nos referíamos.

### Os princípios da renderização baseada em física

O PBR fornece ao shader um modelo de como a luz interage com as superfícies que se aproxima do comportamento físico real. Em vez de tratar reflexos e brilhos como ajustes arbitrários definidos a olho pelo artista, o PBR modela a superfície a partir de propriedades com correspondência física: o índice de refração, a rugosidade, a metalicidade, a forma como a luz é absorvida ou refletida. A partir dessas propriedades, o cálculo de iluminação simula a luz como um fluxo que incide sobre a superfície e dela se reflete ou refrata de acordo com suas características.

A consequência prática mais importante desse modelo é que ele **remove a adivinhação**. Em fluxos anteriores, o artista precisava estimar, manualmente, o quanto cada superfície deveria brilhar — um processo subjetivo e inconsistente. No PBR, esse comportamento decorre das propriedades físicas declaradas: uma vez que se diga que um material é metálico e moderadamente rugoso, o modelo sabe como ele deve refletir a luz. Disso derivam três benefícios que justificam a adoção do paradigma: os objetos parecem corretos sob qualquer condição de iluminação, pois o cálculo responde fisicamente a cada luz; a produção ganha consistência, pois artistas diferentes seguindo as mesmas regras chegam a resultados coerentes; e a criação de *assets* realistas torna-se mais previsível e menos dependente de tentativa e erro.

### Os mapas como entradas do cálculo

O modelo PBR é alimentado por um conjunto de mapas de textura, cada um responsável por descrever, ponto a ponto, uma propriedade da superfície. Compreender o significado de cada mapa é compreender a linguagem com que o artista conversa com o motor.

O mapa de **cor base**, também chamado de *albedo* ou *base color*, descreve a cor própria da superfície — o barro do tijolo, o pigmento da tinta, o tom do metal. É fundamental, como já se enfatizou desde o Capítulo 1, que esse mapa não contenha informação de iluminação: nada de sombras, reflexos ou realces pintados. Ele descreve apenas a cor que a superfície tem, deixando todo o cálculo de luz a cargo do shader.

O mapa de **normais** armazena, em seus canais de cor, a direção para a qual cada ponto da superfície aponta. Como vimos no capítulo histórico, é esse mapa que permite ao cálculo de iluminação tratar uma superfície geometricamente plana como se fosse repleta de microdetalhe, fazendo a luz incidir corretamente sobre saliências e reentrâncias que não existem na malha. Seus valores não têm significado visual intuitivo: abrir um mapa de normais revela uma imagem predominantemente azulada e estranha, pois suas cores codificam direções, não tons.

O mapa de **rugosidade** (*roughness*) descreve o quanto a superfície é áspera ou lisa, em valores que variam de modo linear de zero a um. Uma superfície lisa reflete a luz de maneira nítida e concentrada, produzindo realces pequenos e intensos; uma superfície rugosa espalha a luz, produzindo realces amplos e difusos. É a rugosidade, mais do que qualquer outro mapa, que comunica ao olho a diferença táctil entre um espelho e uma parede de gesso.

O mapa **metálico** (*metallic*) define quais regiões da superfície são metal e quais não são, em geral como uma imagem aproximadamente preto-e-branca. Essa distinção é decisiva porque metais e não metais interagem com a luz de maneiras fundamentalmente diferentes, e o cálculo de iluminação trata cada caso de forma distinta.

Outros mapas complementam esse núcleo. O mapa de **altura** (*height*) registra a elevação relativa de cada ponto, com o preto representando os pontos mais baixos e o branco os mais altos, e pode ser usado para deslocar a superfície ou reforçar a percepção de profundidade. O mapa **especular** (*specular*) e o mapa de **brilho** (*gloss*) aparecem em determinados fluxos de trabalho para descrever, respectivamente, a intensidade do reflexo especular e a suavidade da superfície — sendo o gloss, conceitualmente, o inverso da rugosidade.

> Figura: Painel com os principais mapas PBR de um mesmo material lado a lado — cor base, normais, rugosidade, metálico e altura — cada um rotulado com a propriedade que descreve, seguido da imagem renderizada resultante da combinação de todos.
>
> **Screenshot sugerido:**
> Captura de um material PBR completo em um software de texturização, exibindo simultaneamente as miniaturas de cada mapa e a pré-visualização do resultado final iluminado.

### Os dois fluxos de trabalho do PBR

A indústria consolidou dois modos principais de organizar essas propriedades, conhecidos como **fluxos de trabalho** (*workflows*). Ambos buscam resultados fisicamente plausíveis, mas partem de premissas diferentes sobre quais mapas usar.

O fluxo **metallic-roughness** define o material por meio de dois mapas centrais: o metálico e o de rugosidade. Parte da premissa de que um material ou é metálico ou não é, e de que a rugosidade é uma propriedade universal, válida para todas as superfícies. Por concentrar a descrição nesses dois eixos, esse fluxo tende a ser mais simples de usar e a produzir resultados mais previsíveis, razão pela qual se tornou o mais difundido.

O fluxo **specular-glossiness** define o material por meio de um mapa especular e um mapa de brilho. Parte da premissa de que o reflexo especular é uma propriedade universal de todas as superfícies, metálicas ou não. Em troca de maior complexidade, esse fluxo oferece controle mais fino sobre o comportamento especular dos materiais. A escolha entre um e outro depende das necessidades do projeto e das ferramentas envolvidas; não há um vencedor absoluto, e o profissional deve compreender ambos para transitar entre pipelines distintos.

### Espaço de cor: o detalhe que separa o correto do incorreto

Há um aspecto técnico, frequentemente ignorado por iniciantes, que separa um resultado fisicamente correto de um resultado sutilmente errado: o **espaço de cor** em que cada mapa é interpretado. Existem dois espaços relevantes. O espaço **sRGB** é não linear: ele comprime as cores escuras e expande as claras para se ajustar à forma como o olho humano percebe o brilho e à maneira como os monitores exibem imagens. Já o espaço **linear** representa os valores de forma uniforme, fiel às quantidades físicas envolvidas, e é nesse espaço que o cálculo de iluminação precisa operar para ser preciso.

A regra prática que decorre disso é decisiva: o mapa de cor base, por descrever cores que serão vistas, é normalmente interpretado em sRGB; mas **todos os demais mapas** — rugosidade, metálico, normais, altura — devem ser interpretados em espaço linear, pois seus valores são dados para cálculo, e não cores para exibição. Interpretar um mapa de rugosidade como se fosse uma imagem sRGB, por exemplo, distorce silenciosamente seus valores e produz materiais que parecem "quase certos", mas nunca convincentes. Esse é um dos erros mais difíceis de diagnosticar para quem não compreende a distinção, justamente porque não gera um defeito óbvio, e sim uma imprecisão difusa.

## Aplicação em Jogos

A compreensão de como o motor interpreta materiais é o que permite ao artista trabalhar com intenção. Saber que a rugosidade controla o tamanho e a difusão dos realces faz com que o artista pinte esse mapa pensando em quais regiões devem parecer polidas e quais devem parecer gastas. Saber que o mapa de cor base não deve conter sombras impede o erro de embutir iluminação, herança das eras antigas que o PBR justamente busca eliminar. Saber que mapas de dados vivem em espaço linear evita uma classe inteira de defeitos sutis.

Há também uma dimensão de desempenho. Como o shader é executado para cada ponto visível, a complexidade do material tem custo direto sobre a taxa de quadros. Materiais com muitos mapas e cálculos elaborados são mais caros que materiais simples, e o profissional precisa equilibrar fidelidade visual e orçamento de processamento, especialmente em plataformas modestas. A escolha entre os fluxos de trabalho, o número de mapas empregados e a resolução das texturas são, todas, decisões que reverberam no desempenho final do jogo. Compreender o motor é, portanto, compreender também os limites dentro dos quais a criatividade artística deve operar.

## Estudo de Caso

Considere uma equipe que produz uma única esfera de teste — recurso clássico para validar materiais — com a intenção de criar ouro polido. No primeiro ensaio, o artista, ainda sem dominar o modelo do motor, pinta na cor base um amarelo com realces claros já desenhados, define a rugosidade num valor intermediário e esquece de marcar o material como metálico. O resultado parece um plástico amarelo sujo: os realces pintados brigam com a luz da cena, a ausência de metalicidade impede o reflexo característico do metal, e a rugosidade intermediária espalha a luz de forma incompatível com um material polido.

No segundo ensaio, compreendendo o modelo, o artista corrige cada entrada à luz de seu significado. A cor base passa a conter apenas o tom puro do ouro, sem qualquer realce. O mapa metálico é levado a valor alto, informando ao shader que se trata de metal. A rugosidade é reduzida, indicando uma superfície polida que concentra os reflexos. E o artista verifica que os mapas de dados estão sendo interpretados em espaço linear. O resultado, agora, reage corretamente à iluminação: a esfera reflete o ambiente como ouro, com realces nítidos que se deslocam conforme a câmera se move. A diferença entre os dois ensaios não está em talento artístico, mas em compreender que cada mapa é uma entrada de um cálculo físico, e que alimentar o cálculo com dados incorretos produz, inevitavelmente, uma aparência incorreta.

> Figura: Duas esferas de material lado a lado representando ouro — uma resultante de entradas incorretas (cor base com realce pintado, sem metalicidade, rugosidade intermediária) e outra de entradas corretas (cor base pura, metálico alto, baixa rugosidade) — sob a mesma iluminação.
>
> **Screenshot sugerido:**
> Captura de duas esferas em um visualizador de materiais sob iluminação ambiental idêntica, com os valores de cada parâmetro anotados ao lado de cada esfera.

## Boas Práticas

A boa prática essencial deste capítulo é tratar cada mapa de textura segundo o seu papel no cálculo do motor, e não segundo sua aparência ao olho. Pintar um mapa de rugosidade significa decidir onde a superfície é lisa ou áspera; pintar um mapa de cor base significa decidir a cor própria da superfície, sem iluminação. Manter essa disciplina mental garante que o material se comporte como esperado quando o shader o interpretar.

É igualmente importante respeitar a regra de espaço de cor, garantindo que apenas a cor base seja tratada como sRGB e que os demais mapas permaneçam em espaço linear. Recomenda-se, ainda, escolher conscientemente o fluxo de trabalho adequado ao projeto e às ferramentas, e manter-se coerente com ele ao longo de toda a produção. Por fim, vale lembrar que o material tem custo de desempenho: preferir a solução mais simples que atenda à necessidade visual é uma prática que protege a taxa de quadros e, com ela, a experiência do jogador.

## Erros Comuns

O erro mais difundido, e já recorrente nesta apostila, é embutir iluminação na cor base, pintando sombras e realces que deveriam emergir do cálculo do shader. Esse erro anula a principal vantagem do PBR e produz materiais que só funcionam sob a luz para a qual foram pintados. Igualmente comum é negligenciar a metalicidade, esquecendo-se de marcar como metal aquilo que deveria sê-lo, o que faz metais parecerem plásticos foscos.

Outro equívoco frequente é interpretar mapas de dados no espaço de cor errado, tratando rugosidade, normais ou altura como se fossem imagens sRGB. Como esse erro não gera um defeito gritante, mas uma imprecisão difusa, ele frequentemente passa despercebido e compromete sutilmente a qualidade de toda uma biblioteca de materiais. Há ainda a confusão entre os fluxos de trabalho, misturando mapas de metallic-roughness com mapas de specular-glossiness sem compreender que partem de premissas distintas. Por fim, um erro de mentalidade: avaliar um mapa pela sua beleza isolada, em vez de avaliá-lo pelo efeito que produz quando interpretado pelo motor — afinal, um mapa de normais "feio" e azulado pode estar perfeitamente correto, enquanto uma cor base "bonita" e cheia de sombras pode estar conceitualmente errada.

## Resumo

Este capítulo abriu a caixa-preta da renderização em tempo real e mostrou o motor como um intérprete de materiais. No centro do processo está o shader, programa executado em paralelo pela GPU que, recebendo geometria, texturas, parâmetros de material e luzes, calcula a cor de cada ponto da superfície. O modelo que rege esse cálculo na produção contemporânea é o da renderização baseada em física, que descreve as superfícies por propriedades com correspondência física — rugosidade, metalicidade, índice de refração — e, com isso, remove a adivinhação, garante consistência entre artistas e cenas e torna previsível a criação de *assets* realistas. Esse modelo é alimentado por mapas de textura, cada um com função definida: cor base para a coloração própria, normais para o microdetalhe de direção, rugosidade para a aspereza, metálico para a distinção entre metal e não metal, além de mapas complementares de altura, especular e brilho. A indústria organiza esses mapas em dois fluxos principais, metallic-roughness e specular-glossiness, cada um com suas premissas e compromissos. Reforçou-se, ainda, a regra do espaço de cor: a cor base é interpretada em sRGB, enquanto todos os demais mapas, por serem dados de cálculo, devem permanecer em espaço linear. Acima de tudo, consolidou-se o princípio que atravessa toda a Parte I: aparência é cálculo, e cada mapa é uma entrada desse cálculo, de modo que dados corretos produzem aparências corretas, e dados incorretos, por mais bem pintados que estejam, produzem ilusões que não convencem.

## Exercícios

1. Explique o papel do shader na renderização e justifique por que esse cálculo é executado na GPU, e não na CPU. Relacione sua resposta à arquitetura de cada processador.

2. Para cada um dos seguintes mapas — cor base, normais, rugosidade e metálico —, descreva qual propriedade da superfície ele controla e que efeito visual sua alteração produz. Em seguida, explique por que o mapa de cor base não deve conter sombras.

3. Compare os fluxos de trabalho metallic-roughness e specular-glossiness, apontando a premissa central de cada um. Em que situação você preferiria um ao outro, e por quê?

4. Diagnóstico: um material que deveria representar metal polido aparece, no jogo, como um plástico fosco e sem reflexo, mesmo com a cor correta. Levante pelo menos duas hipóteses, baseadas nos conceitos do capítulo, para a causa do problema, e proponha como verificá-las.

5. Explique a diferença entre os espaços de cor sRGB e linear e a regra de qual mapa pertence a cada um. Por que o erro de interpretar um mapa de rugosidade como sRGB é particularmente difícil de detectar?

## Leituras Complementares

### Material de referência do projeto

- McDERMOTT, Wes. *The PBR Guide* (Allegorithmic/Adobe, ed. 2018) — referência central sobre o modelo PBR, os tipos de mapa, os fluxos metallic-roughness e specular-glossiness e a questão do espaço de cor.

### Fontes complementares externas

- PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023. Disponível gratuitamente em pbr-book.org. Referência definitiva sobre a teoria física por trás da renderização e sua implementação; aprofunda, em nível avançado, o cálculo que o shader executa.
- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Os capítulos sobre sombreamento físico e materiais detalham, com rigor e foco em tempo real, os modelos que sustentam o PBR descrito neste capítulo.
- KARIS, Brian. *Real Shading in Unreal Engine 4*. SIGGRAPH 2013. Apresenta, do ponto de vista de quem implementou um motor comercial, as decisões práticas de modelo de sombreamento e fluxo de trabalho discutidas aqui. Disponível em blog.selfshadow.com.
- *LearnOpenGL* (learnopengl.com), de Joey de Vries, seções "Lighting" e "PBR" — aprofundam, com código comentado, o funcionamento do shader e do cálculo de iluminação por ponto.
- *The Book of Shaders*, de Patricio Gonzalez Vivo e Jen Lowe (thebookofshaders.com) — introdução acessível à lógica dos shaders executados na GPU.
- *Graphics Compendium — Cook-Torrance Reflectance Model* (graphicscompendium.com) — explica, de forma didática, o modelo de reflectância que está na base do cálculo especular usado pelos materiais PBR.
- Documentação oficial da Unreal Engine (dev.epicgames.com), da Unity (docs.unity3d.com), do Adobe Substance 3D (helpx.adobe.com/substance-3d.html) e da Godot (docs.godotengine.org), seções sobre materiais — para observar como cada motor e ferramenta expõe, em seus editores, os mapas, fluxos de trabalho e o espaço de cor discutidos neste capítulo.
