# Capítulo 3 — Como Motores de Jogo Interpretam Materiais

## Introdução

Os dois capítulos anteriores estabeleceram *o que* é um material e *como* a indústria chegou às formas atuais de descrevê-lo. Resta a pergunta que fecha os fundamentos da disciplina: o que, exatamente, um motor de jogo faz com tudo isso? Quando o artista entrega uma geometria, um conjunto de texturas e um material, o que acontece nos bastidores para que, milhões de vezes por segundo, o motor decida a cor de cada pixel da tela?

Este capítulo abre a "caixa-preta" da renderização em tempo real, mas o faz no nível conceitual, sem se prender a uma interface ou a um software específico. O objetivo é que o estudante entenda o motor como um intérprete: ele recebe dados — vértices, coordenadas UV, canais de textura e parâmetros de material — e produz uma imagem segundo regras bem definidas. A maior dessas regras, hoje, é o conjunto de princípios da renderização baseada em física, o PBR, que foi anunciado historicamente no capítulo anterior e que agora examinaremos por dentro. Compreender como o motor interpreta os materiais é o que transforma o artista de alguém que "tenta até parecer bom" em alguém que entende por que algo parece bom — e por que, quando não parece, sabe onde está o erro.

Recuperaremos aqui o princípio fundamental do Capítulo 1: aparência é o resultado de um cálculo que combina dados de superfície com luz. Este capítulo é, em essência, o estudo desse cálculo e dos dados que o alimentam.

## Desenvolvimento

### O sombreador: o programa que pinta cada ponto

> 📘 **Definição**
> **Sombreador** (*shader*) é um programa executado pelo processador gráfico (GPU) que define as regras pelas quais cada ponto de uma superfície é convertido em cor na tela. Ele recebe como entrada a geometria, os canais de textura, os parâmetros do material e as luzes da cena, e produz como saída a cor final de cada pixel.

No coração da interpretação de materiais está o **sombreador**. Um sombreador é um pequeno programa, executado pelo processador gráfico (a GPU), que define as regras pelas quais cada ponto de uma superfície será convertido em cor na tela. Ele recebe como entrada a geometria, as texturas, os parâmetros do material e as luzes da cena, e produz como saída a cor final de cada pixel. Tudo o que discutimos sob o nome de "material" é, em última análise, alimentado a um sombreador, que executa o cálculo de iluminação propriamente dito.

A enorme eficiência do hardware gráfico vem de sua capacidade de executar esse mesmo programa para um número imenso de pontos em paralelo. Enquanto o processador central, a CPU, é otimizado para executar instruções variadas em sequência, a GPU é construída para repetir o mesmo cálculo simples sobre milhões de elementos simultaneamente — exatamente o que a renderização exige. É essa arquitetura que torna possível avaliar, dezenas de vezes por segundo, a interação entre luz e superfície em cada ponto visível da cena. Quando o Capítulo 1 afirmou que aparência é cálculo, é a este cálculo, executado pelo sombreador na GPU, que nos referíamos.

Para tornar essa criação acessível a artistas sem formação em programação, motores como Unreal Engine e Unity expõem o sombreador como um **grafo de nós** (*node graph*): um editor visual em que cada nó representa uma operação matemática e as conexões descrevem o fluxo de dados. *Metal Gear Solid V: The Phantom Pain* (Kojima Productions, 2015) é um caso documentado de estúdio que foi além — a equipe construiu sobre o Fox Engine uma camada visual de nós customizados para os biomas afegão e africano do jogo, reduzindo a dependência de sombreadores escritos manualmente; essa abordagem, de ocultar a matemática atrás de blocos visuais intuitivos, tornou-se padrão na indústria e é o que os artistas manipulam diariamente sem escrever uma linha de HLSL ou GLSL.

### Os princípios da renderização baseada em física

O PBR fornece ao sombreador um modelo de como a luz interage com as superfícies que se aproxima do comportamento físico real. Em vez de tratar reflexos e brilhos como ajustes arbitrários definidos a olho pelo artista, o PBR modela a superfície a partir de propriedades com correspondência física: a rugosidade, a metalicidade, a cor base, a forma como a luz é absorvida ou refletida. A partir dessas propriedades, o cálculo de iluminação simula a luz como um fluxo que incide sobre a superfície e dela se reflete ou refrata de acordo com suas características.

> ⚠️ **Atenção**
> No fluxo *metallic/roughness* — o que esta apostila adota como padrão —, o artista *não* define diretamente o índice de refração (IOR) dos materiais. A reflectância especular dos dielétricos é assumida como um valor praticamente constante (em torno de 4% da luz incidente, correspondendo a um IOR de aproximadamente 1,5), e o artista apenas declara, pelo mapa metálico, se a superfície é ou não metal. O índice de refração só se torna um parâmetro explícito em fluxos ou sombreadores especializados.

A consequência prática mais importante desse modelo é que ele **remove a adivinhação**. Em fluxos anteriores, o artista precisava estimar, manualmente, o quanto cada superfície deveria brilhar — um processo subjetivo e inconsistente. No PBR, esse comportamento decorre das propriedades físicas declaradas: uma vez que se diga que um material é metálico e moderadamente rugoso, o modelo sabe como ele deve refletir a luz. Disso derivam três benefícios que justificam a adoção do paradigma: os objetos parecem corretos sob qualquer condição de iluminação, pois o cálculo responde fisicamente a cada luz; a produção ganha consistência, pois artistas diferentes seguindo as mesmas regras chegam a resultados coerentes; e a criação de *assets* realistas torna-se mais previsível e menos dependente de tentativa e erro.

### Os mapas como entradas do cálculo

O modelo PBR é alimentado por um conjunto de canais de textura, cada um responsável por descrever, ponto a ponto, uma propriedade da superfície. Compreender o significado de cada canal é compreender a linguagem com que o artista conversa com o motor de jogo.

O canal de **cor base** (*albedo* ou *base color*) descreve a cor própria da superfície — o barro do tijolo, o pigmento da tinta, o tom do metal. É fundamental, como já se enfatizou desde o Capítulo 1, que esse canal não contenha informação de iluminação: nada de sombras, reflexos ou realces pintados. Ele descreve apenas a cor que a superfície tem, deixando todo o cálculo de luz a cargo do sombreador.

> ❌ **Erro Comum**
> Pintar sombras ou realces de iluminação diretamente no canal de cor base é um dos erros mais persistentes, herança dos fluxos de trabalho anteriores ao PBR. Esse erro anula a principal vantagem do paradigma: a superfície parecerá correta apenas sob a iluminação para a qual foi pintada, e estranha em qualquer outra condição.

O **mapa de normais** (*normal map*) armazena, em seus canais de cor, a direção para a qual cada ponto da superfície aponta. Como vimos no Capítulo 2, é esse mapa que permite ao cálculo de iluminação tratar uma superfície geometricamente plana como se fosse repleta de microdetalhe, fazendo a luz incidir corretamente sobre saliências e reentrâncias que não existem na malha. Seus valores não têm significado visual intuitivo: abrir um mapa de normais revela uma imagem predominantemente azulada e estranha, pois suas cores codificam direções, não tons.

O **mapa de rugosidade** (*roughness map*) descreve o quanto a superfície é áspera ou lisa, em valores que vão de zero a um. Uma superfície lisa reflete a luz de maneira nítida e concentrada, produzindo realces pequenos e intensos; uma superfície rugosa espalha a luz, produzindo realces amplos e difusos. É a rugosidade, mais do que qualquer outro mapa, que comunica ao olho a diferença táctil entre um espelho e uma parede de gesso.

> ⚠️ **Atenção**
> Embora os valores armazenados no mapa de rugosidade sejam lineares, a *resposta visual* à rugosidade não é proporcional: os modelos de microfaceta tipicamente elevam o valor ao quadrado antes de alimentar o cálculo. Isso significa que dobrar o valor no mapa não dobra o "tamanho" do realce — a relação é não linear, e o artista deve pintá-la observando o resultado no motor, não esperando uma correspondência direta entre o número e o efeito visual.

O **mapa metálico** (*metallic map*) define quais regiões da superfície são metal e quais não são, em geral como uma imagem aproximadamente preto-e-branca. Essa distinção é decisiva porque metais e não metais interagem com a luz de maneiras fundamentalmente diferentes, e o cálculo de iluminação trata cada caso de forma distinta.

Outros canais complementam esse núcleo. O **mapa de altura** (*height map*) registra a elevação relativa de cada ponto, com o preto representando os pontos mais baixos e o branco os mais altos, e pode ser usado para deslocar a superfície ou reforçar a percepção de profundidade. O **mapa especular** (*specular map*) e o **mapa de brilho** (*gloss map*) aparecem em determinados fluxos de trabalho para descrever, respectivamente, a intensidade do reflexo especular e a suavidade da superfície — sendo o brilho (*gloss*), conceitualmente, o inverso da rugosidade.

> **Figura 3.1** — Painel com os principais canais de textura PBR de um mesmo material lado a lado — cor base, mapa de normais, mapa de rugosidade, mapa metálico e mapa de altura — cada um rotulado com a propriedade que descreve, seguido da imagem renderizada resultante da combinação de todos.

**Screenshot sugerido:** Captura de um material PBR completo em um software de texturização, exibindo simultaneamente as miniaturas de cada canal e a pré-visualização do resultado final iluminado.

### Os dois fluxos de trabalho do PBR

A indústria consolidou dois modos principais de organizar essas propriedades, conhecidos como **fluxos de trabalho** (*workflows*). Ambos buscam resultados fisicamente plausíveis, mas partem de premissas diferentes sobre quais mapas usar.

O fluxo **metallic-roughness** define o material por meio de dois canais centrais: o mapa metálico e o mapa de rugosidade. Parte da premissa de que um material ou é metálico ou não é, e de que a rugosidade é uma propriedade universal, válida para todas as superfícies. Por concentrar a descrição nesses dois eixos, esse fluxo tende a ser mais simples de usar e a produzir resultados mais previsíveis, razão pela qual se tornou o mais difundido.

O fluxo **specular-glossiness** define o material por meio de um mapa especular e um mapa de brilho. Parte da premissa de que o reflexo especular é uma propriedade universal de todas as superfícies, metálicas ou não. Em troca de maior complexidade, esse fluxo oferece controle mais fino sobre o comportamento especular dos materiais.

> 💡 **Dica Profissional**
> A escolha entre os fluxos *metallic-roughness* e *specular-glossiness* depende das ferramentas envolvidas e do motor de jogo de destino. O profissional deve compreender ambos para transitar entre pipelines distintos — não há um vencedor absoluto, mas o *metallic-roughness* é hoje o padrão predominante nos principais motores de jogo.

### Espaço de cor: o detalhe que separa o correto do incorreto

Há um aspecto técnico, frequentemente ignorado por iniciantes, que separa um resultado fisicamente correto de um resultado sutilmente errado: o **espaço de cor** em que cada canal de textura é interpretado. Existem dois espaços relevantes. O espaço **sRGB** é não linear: ele comprime as cores escuras e expande as claras para se ajustar à forma como o olho humano percebe o brilho e à maneira como os monitores exibem imagens. Já o espaço **linear** representa os valores de forma uniforme, fiel às quantidades físicas envolvidas, e é nesse espaço que o cálculo de iluminação precisa operar para ser preciso.

> 📘 **Definição**
> **Espaço de cor** é a convenção matemática que define como os valores numéricos de uma imagem se relacionam com as quantidades de luz que representam. O espaço **sRGB** é não linear (comprimido para compensar a percepção humana e o comportamento dos monitores); o espaço **linear** representa as quantidades de forma uniforme e é necessário para cálculos de iluminação fisicamente corretos.

A regra prática que decorre disso é decisiva: o canal de cor base, por descrever cores que serão vistas, é normalmente interpretado em sRGB; mas **todos os demais canais** — mapa de rugosidade, mapa metálico, mapa de normais, mapa de altura — devem ser interpretados em espaço linear, pois seus valores são dados para cálculo, e não cores para exibição.

> ⚠️ **Atenção**
> Interpretar um mapa de rugosidade como se fosse uma imagem sRGB distorce silenciosamente seus valores e produz materiais que parecem "quase certos", mas nunca completamente convincentes. Como esse erro não gera um defeito óbvio — apenas uma imprecisão difusa —, ele é um dos mais difíceis de diagnosticar para quem não compreende a distinção entre espaços de cor.

## Aplicação em Jogos

A compreensão de como o motor de jogo interpreta materiais é o que permite ao artista trabalhar com intenção. Saber que o mapa de rugosidade controla o tamanho e a difusão dos realces faz com que o artista pinte esse mapa pensando em quais regiões devem parecer polidas e quais devem parecer gastas. Saber que o canal de cor base não deve conter sombras impede o erro de embutir iluminação, herança das eras antigas que o PBR justamente busca eliminar. Saber que canais de dados vivem em espaço linear evita uma classe inteira de defeitos sutis.

Há também uma dimensão de desempenho. Como o sombreador é executado para cada ponto visível, a complexidade do material tem custo direto sobre a taxa de quadros. Materiais com muitos canais e cálculos elaborados são mais caros que materiais simples, e o profissional precisa equilibrar fidelidade visual e orçamento de processamento, especialmente em plataformas modestas.

> ⚠️ **Atenção**
> A gestão inadequada da compilação de variantes de sombreador pode gerar gaguejadas perceptíveis durante o jogo — fenômeno amplamente documentado no lançamento de *Cyberpunk 2077* (CD Projekt RED, 2020), cujos *shader compilation stutters* forçaram a CDPR a lançar patches dedicados à pré-compilação antes de novas cenas. A complexidade do sistema de materiais tem consequências diretas sobre a experiência do jogador, e não apenas sobre a aparência.

A escolha entre os fluxos de trabalho, o número de canais empregados e a resolução das texturas são, todas, decisões que reverberam no desempenho final do jogo. Compreender o motor é, portanto, compreender também os limites dentro dos quais a criatividade artística deve operar.

## Estudo de Caso

### Hellblade: Senua's Sacrifice e a Transparência do Pipeline de Materiais — Ninja Theory, 2017

> **Figura 3.2** — Rede de material do couro de Senua no editor de materiais da Unreal Engine 4, conforme demonstrado na série *"The Making of Hellblade"* — mostrando as entradas Base Color, Roughness e Normal separadas, cada uma alimentada por seu canal respectivo em espaço de cor correto.

**Screenshot sugerido:** Captura de tela oficial de Hellblade: Senua's Sacrifice — disponível em https://www.ninjatheory.com/hellblade-senuas-sacrifice/ (presskit) ou via Steam em https://store.steampowered.com/app/414340/Hellblade_Senuas_Sacrifice/. A rede de material está documentada nos episódios da série "The Making of Hellblade" no canal YouTube da Ninja Theory.

Em 2017, a Ninja Theory — estúdio independente de Cambridge com cerca de vinte pessoas — fez algo incomum para a indústria: documentou publicamente, em tempo real, o processo de criação de um jogo AAA. A série *"The Making of Hellblade"*, publicada em trinta episódios no YouTube ao longo do desenvolvimento, incluiu capítulos inteiros dedicados ao sistema de materiais no Unreal Engine 4. O que diferencia essa documentação de um tutorial comum é que a equipe explicou não apenas *como* criar os materiais, mas *por que* o motor de jogo interpreta cada entrada da maneira que interpreta.

Em *Hellblade*, Senua atravessa cenários de pedra úmida celta, cavernas de basalto, campos de névoa e interiores de barcos vikings. Para cada ambiente, os artistas construíram materiais no editor de materiais da Unreal partindo das mesmas entradas do capítulo: Base Color, Metallic, Roughness, Normal. Em um dos episódios mais didáticos da série, o artista de personagem demonstra como criar o couro da roupa de Senua — e o raciocínio exposto é preciso: a cor base recebe o tom do couro sem sombras, pois as sombras virão do cálculo do motor; o mapa de rugosidade recebe um canal com gradiente entre as regiões novas e as gastas, porque é a rugosidade, não a cor, que informa ao sombreador a microgeometria da superfície; o mapa de normais recebe as costuras e o relevo do couro a partir de um arquivo assado de um modelo de alta resolução. Metalicidade permanece em zero porque couro é dielétrico.

O resultado mais revelador do experimento da Ninja Theory foi observar o que acontecia quando qualquer dessas entradas era marcada incorretamente no importador. Um mapa de rugosidade importado como cor sRGB em vez de linear produzia um couro que parecia "quase certo" mas nunca completamente plausível — exatamente o erro silencioso descrito na seção de espaço de cor deste capítulo. Os artistas do estúdio identificaram esse problema ao vivo, na câmera, e explicaram a causa ao público. Esse momento acidental tornou-se um dos registros mais honestos da indústria sobre como um motor de jogo interpreta dados de textura — e por que a distinção entre cor e dados não é formalismo técnico, mas a diferença entre um material que funciona e um que permanece inexplicavelmente insatisfatório.

## Boas Práticas

A boa prática essencial deste capítulo é tratar cada canal de textura segundo o seu papel no cálculo do motor, e não segundo sua aparência ao olho. Pintar um mapa de rugosidade significa decidir onde a superfície é lisa ou áspera; pintar um canal de cor base significa decidir a cor própria da superfície, sem iluminação. Manter essa disciplina mental garante que o material se comporte como esperado quando o sombreador o interpretar.

> 💡 **Dica Profissional**
> Ao avaliar um mapa, a pergunta correta não é "esta imagem parece boa?", mas "este canal fornece os dados corretos para o cálculo do motor?". Um mapa de normais com cores predominantemente azuladas e sem apelo visual pode estar perfeitamente correto; um canal de cor base "bonito" e cheio de sombras pintadas pode estar conceitualmente errado.

É igualmente importante respeitar a regra de espaço de cor, garantindo que apenas a cor base seja tratada como sRGB e que os demais canais permaneçam em espaço linear. Recomenda-se, ainda, escolher conscientemente o fluxo de trabalho adequado ao projeto e às ferramentas, e manter-se coerente com ele ao longo de toda a produção. Por fim, vale lembrar que o material tem custo de desempenho: preferir a solução mais simples que atenda à necessidade visual é uma prática que protege a taxa de quadros e, com ela, a experiência do jogador.

## Erros Comuns

> ❌ **Erro Comum**
> O erro mais difundido é embutir iluminação no canal de cor base, pintando sombras e realces que deveriam emergir do cálculo do sombreador. Esse erro anula a principal vantagem do PBR e produz materiais que só funcionam sob a luz para a qual foram pintados.

Igualmente comum é negligenciar a metalicidade, esquecendo-se de marcar como metal aquilo que deveria sê-lo, o que faz metais parecerem plásticos foscos.

> ❌ **Erro Comum**
> Interpretar canais de dados (mapa de rugosidade, mapa de normais, mapa de altura) no espaço de cor sRGB em vez de linear é um erro que não gera um defeito gritante, mas uma imprecisão difusa. Ele frequentemente passa despercebido e compromete sutilmente a qualidade de toda uma biblioteca de materiais.

Há ainda a confusão entre os fluxos de trabalho, misturando canais de metallic-roughness com canais de specular-glossiness sem compreender que partem de premissas distintas.

## Resumo

Este capítulo abriu a caixa-preta da renderização em tempo real e mostrou o motor de jogo como um intérprete de materiais. No centro do processo está o sombreador, programa executado em paralelo pela GPU que, recebendo geometria, canais de textura, parâmetros de material e luzes, calcula a cor de cada ponto da superfície. O modelo que rege esse cálculo na produção contemporânea é o da renderização baseada em física, que descreve as superfícies por propriedades com correspondência física — mapa de rugosidade, mapa metálico, reflectância —, e, com isso, remove a adivinhação, garante consistência entre artistas e cenas e torna previsível a criação de *assets* realistas. Esse modelo é alimentado por canais de textura, cada um com função definida: cor base para a coloração própria, mapa de normais para o microdetalhe de direção, mapa de rugosidade para a aspereza, mapa metálico para a distinção entre metal e não metal, além de canais complementares de mapa de altura, especular e brilho. A indústria organiza esses canais em dois fluxos principais, metallic-roughness e specular-glossiness, cada um com suas premissas e compromissos. Reforçou-se, ainda, a regra do espaço de cor: o canal de cor base é interpretado em sRGB, enquanto todos os demais canais, por serem dados de cálculo, devem permanecer em espaço linear. Acima de tudo, consolidou-se o princípio que atravessa toda a Parte I: aparência é cálculo, e cada canal é uma entrada desse cálculo, de modo que dados corretos produzem aparências corretas, e dados incorretos, por mais bem pintados que estejam, produzem ilusões que não convencem.

## Exercícios

**1.** Explique o papel do sombreador na renderização e justifique por que esse cálculo é executado na GPU, e não na CPU. Relacione sua resposta à arquitetura de cada processador.

**2.** Para cada um dos seguintes canais de textura — cor base, mapa de normais, mapa de rugosidade e mapa metálico —, descreva qual propriedade da superfície ele controla e que efeito visual sua alteração produz. Em seguida, explique por que o canal de cor base não deve conter sombras.

**3.** Compare os fluxos de trabalho metallic-roughness e specular-glossiness, apontando a premissa central de cada um. Em que situação você preferiria um ao outro, e por quê?

**4.** Diagnóstico: um material que deveria representar metal polido aparece, no jogo, como um plástico fosco e sem reflexo, mesmo com a cor correta. Levante pelo menos duas hipóteses, baseadas nos conceitos do capítulo, para a causa do problema, e proponha como verificá-las.

**5.** Explique a diferença entre os espaços de cor sRGB e linear e a regra de qual canal pertence a cada um. Por que o erro de interpretar um mapa de rugosidade como sRGB é particularmente difícil de detectar?

## Glossário

**Canal de cor base (albedo / base color):** Canal de textura que armazena a cor própria da superfície, sem informação de iluminação. É interpretado em espaço de cor sRGB.

**Espaço linear:** Convenção de espaço de cor em que os valores numéricos representam quantidades de luz de forma uniforme e diretamente proporcional. Necessário para cálculos de iluminação fisicamente corretos. Todos os canais de textura que armazenam dados de cálculo (mapa de rugosidade, mapa metálico, mapa de normais, mapa de altura) devem permanecer neste espaço.

**Espaço sRGB:** Convenção de espaço de cor não linear, comprimida para compensar a percepção humana e o comportamento dos monitores. Usado para o canal de cor base.

**Fluxo de trabalho (workflow):** Conjunto de convenções que define quais canais de textura usar e como organizá-los para descrever os materiais. Os dois fluxos principais do PBR são *metallic-roughness* e *specular-glossiness*.

**GPU (Graphics Processing Unit):** Processador gráfico especializado em executar o mesmo cálculo sobre milhões de elementos em paralelo. É onde os sombreadores são executados durante a renderização em tempo real.

**Mapa de altura:** Canal de textura que armazena a elevação relativa de cada ponto da superfície (preto = mais baixo, branco = mais alto). Pode ser usado para deslocar a superfície geometricamente ou reforçar a percepção de profundidade.

**Mapa de brilho (gloss map):** Canal de textura que descreve a suavidade da superfície, conceitualmente o inverso do mapa de rugosidade. Presente no fluxo *specular-glossiness*.

**Mapa metálico (metallic map):** Canal de textura que define quais regiões da superfície são metal e quais não são, geralmente como uma imagem preto-e-branca. A distinção é decisiva porque metais e não metais interagem com a luz de formas fundamentalmente diferentes.

**Mapa de normais:** Canal de textura que armazena, em seus canais RGB, a direção para a qual cada ponto da superfície aparenta apontar. Permite simular microdetalhe geométrico sem alterar a malha.

**Mapa de rugosidade:** Canal de textura que descreve, em valores de 0 a 1, o quanto a superfície é áspera (valor alto) ou lisa (valor baixo) em cada ponto, controlando a dispersão dos reflexos especulares.

**Mapa especular (specular map):** Canal de textura que descreve a intensidade do reflexo especular de cada ponto da superfície. Presente no fluxo *specular-glossiness*.

**Sombreador (shader):** Programa executado pela GPU que calcula a cor de cada ponto de uma superfície a partir da geometria, dos canais de textura, dos parâmetros do material e das luzes da cena.

## Leituras Complementares

- **McDERMOTT, Wes. *The PBR Guide* (Allegorithmic/Adobe, ed. 2018)** — referência central sobre o modelo PBR, os tipos de canal de textura, os fluxos metallic-roughness e specular-glossiness e a questão do espaço de cor; leitura direta e obrigatória para este capítulo.
- **PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023** — disponível gratuitamente em pbr-book.org; referência definitiva sobre a teoria física por trás da renderização e sua implementação, aprofundando em nível avançado o cálculo que o sombreador executa.
- **AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018** — os capítulos sobre sombreamento físico e materiais detalham, com rigor e foco em renderização em tempo real, os modelos que sustentam o PBR descrito neste capítulo.
- **KARIS, Brian. *Real Shading in Unreal Engine 4*. SIGGRAPH 2013** — apresenta, do ponto de vista de quem implementou um motor comercial, as decisões práticas de modelo de sombreamento e fluxo de trabalho discutidas aqui. Disponível em blog.selfshadow.com.
- ***LearnOpenGL* (learnopengl.com), de Joey de Vries, seções "Lighting" e "PBR"** — aprofundam, com código comentado, o funcionamento do sombreador e do cálculo de iluminação por ponto.
- ***The Book of Shaders*, de Patricio Gonzalez Vivo e Jen Lowe (thebookofshaders.com)** — introdução acessível à lógica dos sombreadores executados na GPU.
- **Documentação oficial da Unreal Engine (dev.epicgames.com), da Unity (docs.unity3d.com), do Adobe Substance 3D (helpx.adobe.com/substance-3d.html) e da Godot (docs.godotengine.org)** — seções sobre materiais, para observar como cada motor de jogo e ferramenta expõe, em seus editores, os canais de textura, os fluxos de trabalho e o espaço de cor discutidos neste capítulo.
