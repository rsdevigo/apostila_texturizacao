# Capítulo 12 — Texturas Seamless e Tileables

## Introdução

As três primeiras partes desta apostila construíram, passo a passo, o pipeline que leva da geometria nua à imagem iluminada. Aprendemos o que é um material e por que a aparência de uma superfície é um cálculo (Parte I); aprendemos a desdobrar e a preparar a malha para receber esse material (Parte II); aprendemos a autorar os mapas físicos que o sombreador consome e a julgá-los sob a luz (Parte III). Em todo esse percurso, porém, deixamos implícita uma pergunta de ordem prática que agora precisa ser enfrentada de frente: *como, concretamente, se produz uma textura?* A Parte IV se dedica a essa pergunta, apresentando os principais fluxos de trabalho pelos quais a indústria efetivamente cria as imagens que se tornam materiais. Não há um único caminho. Há um repertório de métodos, cada um adequado a um tipo de problema, e a competência profissional está menos em dominar um deles do que em saber qual escolher e como combiná-los.

Este primeiro capítulo da parte trata do método mais antigo, mais econômico e ainda hoje mais onipresente de cobrir grandes superfícies: a textura que se repete. Paredes, pisos, terrenos, fachadas, estradas, panos, cascos de navio — sempre que uma superfície é grande demais para receber uma textura única e detalhada, a solução é construir um pedaço pequeno que possa ser ladrilhado, repetido lado a lado, cobrindo a área inteira sem que a emenda apareça. É o que se chama de textura **tileable** (ladrilhável) e, quando suas bordas se encaixam sem costura perceptível, **seamless** (sem emenda). Compreender por que essa técnica existe, como se constrói uma textura que se repete bem, como se combate a repetição visível que ela inevitavelmente gera, e como ela se desdobra em técnicas mais sofisticadas como as *trim sheets* e o *hotspot texturing* é o objeto deste capítulo — e o ponto de partida natural para uma parte dedicada à produção.

## Desenvolvimento

### Por que repetir: a economia da memória de textura

A razão de existir das texturas tileables é, antes de tudo, econômica, e reencontra o fio condutor de toda a apostila: o equilíbrio entre fidelidade visual e custo. Vimos na Parte II, ao tratar da densidade de texels, que a nitidez de uma superfície depende de quantos pixels de textura cobrem cada unidade de seu espaço físico. Cobrir o chão inteiro de um galpão, ou a fachada de um arranha-céu, com uma densidade de texels adequada exigiria texturas de dimensões colossais — dezenas de milhares de pixels de lado —, muito além do que a memória de vídeo de qualquer plataforma comporta. A textura tileable resolve o impasse invertendo a relação: em vez de uma imagem gigante usada uma vez, usa-se uma imagem pequena, de alta densidade, muitas vezes. Um ladrilho de tijolos de mil pixels de lado, repetido vinte vezes em cada direção, cobre uma parede inteira com a nitidez de uma textura de vinte mil pixels, mas ocupando a memória de apenas mil. Essa multiplicação de área a custo de memória constante é o que torna possível texturizar mundos inteiros.

> 📘 **Definição**
> Uma textura **tileable** (ladrilhável) é uma imagem projetada para ser repetida lado a lado cobrindo uma superfície. Quando suas bordas se encaixam sem descontinuidade perceptível, diz-se que é **seamless** (sem costura). Juntas, essas propriedades permitem cobrir grandes áreas com alta densidade de texels a custo de memória constante.

Essa economia tem como contrapartida uma limitação fundadora que organiza todo o resto do capítulo: como o mesmo pedaço aparece muitas vezes, qualquer característica única, qualquer detalhe singular, qualquer variação que conte uma história específica naquele ponto da superfície será repetida junto, denunciando o truque. A textura tileable é, por natureza, **genérica**: ela representa bem aquilo que numa superfície real também se repete — a trama de um tecido, o padrão de uma alvenaria, o grão de uma madeira — e representa mal aquilo que é singular — uma mancha específica, uma rachadura particular, o desgaste concentrado de um canto. Toda a arte de trabalhar com tileables consiste em explorar a economia da repetição sem deixar que a repetição se torne visível.

### A condição seamless: continuidade nas quatro bordas

Para que um ladrilho se repita sem costura, ele precisa satisfazer uma condição geométrica precisa: a borda direita deve continuar perfeitamente na borda esquerda, e a borda superior na inferior, de modo que, ao colocar duas cópias lado a lado, nada na junção revele onde uma termina e a outra começa. Pensar nessa propriedade é pensar a textura não como um retângulo plano, mas como a superfície de um cilindro — e, considerando as duas direções simultaneamente, de um toro: as bordas opostas são, na verdade, *a mesma linha*, vista de dois lados. Tudo o que atravessa a borda direita reentra pela esquerda; tudo o que sai por cima reentra por baixo.

A consequência prática dessa condição é dupla, e ambas as faces precisam ser respeitadas para que o resultado convença. A primeira é a **continuidade**: as formas que cruzam uma borda devem ter sua continuação exata na borda oposta — uma veia de mármore que sai pela direita precisa reentrar pela esquerda na mesma altura e com a mesma inclinação, sob pena de produzir uma quebra visível, uma costura reta que o olho capta imediatamente por ser antinatural. A segunda, mais sutil e mais frequentemente negligenciada, é a **ausência de feições marcantes próximas às bordas e a homogeneidade geral do ladrilho**: mesmo que as bordas se encaixem perfeitamente, qualquer elemento visualmente forte — uma mancha escura, um objeto destacado, uma variação de brilho — se transformará, ao se repetir, num padrão regular que o olho reconhece como artificial.

> ❌ **Erro Comum**
> Uma textura pode ser tecnicamente seamless — com bordas que se encaixam — e ainda assim "tilear mal", porque possui um elemento dominante que, repetido em grade, denuncia a repetição. A boa textura tileable é simultaneamente contínua nas bordas *e* homogênea no interior, sem pontos que chamem a atenção para si.

> **Figura 12.1** — Um ladrilho seamless representado três vezes — isolado, repetido em uma grade 3×3 com encaixe perfeito e homogêneo, e um contraexemplo tecnicamente seamless mas com uma mancha escura proeminente que, ao se repetir, forma um padrão regular denunciador.

**Screenshot sugerido:** Visualização de uma textura em modo de repetição (tiling preview) numa ferramenta de autoria, mostrando lado a lado um caso bem resolvido e um caso com elemento dominante repetido.

### Como se constrói um ladrilho que se repete

Há, historicamente, dois grandes caminhos para produzir uma textura seamless, e ambos seguem vivos na prática contemporânea. O primeiro, mais antigo, parte de uma imagem real — uma fotografia de uma superfície — e a torna ladrilhável por manipulação. A técnica clássica consiste em deslocar a imagem pela metade, de modo que as antigas bordas venham para o centro, expondo as descontinuidades como uma cruz de costuras no meio do quadro; essas costuras são então removidas por clonagem e pintura, trabalhando-se no interior da imagem, onde é seguro, em vez de nas bordas, que devem permanecer intactas para o encaixe. A esse trabalho soma-se a remoção de iluminação assada — sombras e realces capturados pela fotografia que, como vimos na Parte III, não devem habitar a cor base —, para que a textura possa receber a luz do motor de jogo.

O segundo caminho, hoje dominante, é a construção **procedural**, em que a textura nasce já ladrilhável por princípio: padrões e ruídos gerados matematicamente podem ser definidos sobre o domínio toroidal, de modo que a continuidade nas bordas seja uma garantia da geração, não uma correção posterior. Esse caminho é o tema do próximo capítulo, mas convém antecipar que ele resolve na origem o problema que o método fotográfico precisa corrigir no fim.

> 💡 **Dica Profissional**
> Independentemente do método de construção, todos os canais de textura do material — cor base, mapa de rugosidade, mapa de normais — precisam ser seamless em conjunto. Uma costura invisível na cor base pode revelar-se como uma linha de relevo no mapa de normais sob iluminação rasante.

Independentemente do caminho, alguns princípios de composição governam a qualidade de um ladrilho. Deve-se evitar concentrar variação tonal — regiões mais claras ou mais escuras de grande extensão —, pois elas, repetidas, formam manchas em grade; busca-se uma distribuição tonal relativamente uniforme, com a variação ocorrendo em pequena escala. Deve-se cuidar para que a iluminação implícita seja neutra e direcional-neutra, sem uma direção de luz embutida que se repetiria de modo impossível. E deve-se pensar na escala de uso: um ladrilho de tijolos projetado para ser visto de perto numa parede próxima tem necessidades diferentes de um projetado para um terreno visto de longe, e a densidade de texels planejada determina quanto detalhe faz sentido incluir.

### O inimigo da repetição: o padrão visível e como combatê-lo

Mesmo um ladrilho impecavelmente seamless e homogêneo trai sua natureza quando repetido sobre uma área extensa: o olho humano é extraordinariamente sensível a padrões e, ao ver a mesma configuração reaparecer em intervalos regulares, percebe a grade, e a ilusão de uma superfície contínua se desfaz. Esse fenômeno, conhecido como **tiling visível** ou repetição perceptível, é o principal defeito das texturas que se repetem, e boa parte das técnicas modernas existe precisamente para combatê-lo. Compreendê-lo é entender que o problema não é a costura — essa o ladrilho seamless já resolve — mas a *regularidade*: a superfície real raramente é uniforme, e é justamente a uniformidade perfeita da repetição que denuncia a artificialidade.

As estratégias de combate operam todas pelo mesmo princípio: introduzir variação que quebre a periodicidade. A mais direta é a **variação macro**, em que uma segunda textura de baixa frequência e grande escala — uma mancha suave de cor ou de brilho, muito maior que o ladrilho — é combinada com a textura repetida, modulando-a de modo que regiões diferentes da superfície pareçam sutilmente distintas, ainda que partilhem o mesmo detalhe fino. *The Elder Scrolls IV: Oblivion* (Bethesda, 2006) foi um dos primeiros jogos a implementar esse princípio no terreno: um mapa de variação de cor de baixa frequência modulava as texturas de grama e pedra repetidas, e o estudo dos arquivos BSA do jogo pela comunidade de modding tornou essa implementação um dos primeiros exemplos documentados de quebra de tiling por variação macro em mundo aberto.

Outra estratégia é a **multiplicação de variantes**: usar não um, mas vários ladrilhos intercambiáveis, distribuídos de forma a não formar padrão — solução comum em terrenos, onde diferentes texturas se misturam por máscaras. Há ainda técnicas mais avançadas, como o ladrilhamento estocástico, em que o motor de jogo introduz rotações e deslocamentos aleatórios a cada repetição, dissolvendo a grade ao custo de processamento adicional. E há, sobretudo, a quebra deliberada por elementos sobrepostos — sujeira, desgaste, *decals* — que adicionam o singular sobre o genérico, devolvendo à superfície a irregularidade que a repetição lhe roubou. Essa última estratégia é tão importante que estrutura toda a lógica dos capítulos seguintes: a base tileable fornece o genérico e econômico; as camadas sobrepostas fornecem o particular e único.

### Trim sheets: organizar a repetição em faixas

> 📘 **Definição**
> Uma ***trim sheet*** (folha de acabamentos) é uma textura organizada em faixas horizontais, cada uma contendo um tipo de acabamento — rebites, molduras, painéis lisos, frisos ornamentais. Modelos são desdobrados de modo que suas superfícies caiam sobre as faixas adequadas, e como cada faixa é tileable na horizontal, uma única textura pode texturizar uma quantidade enorme de geometria variada.

A potência da técnica está nessa reutilização extrema, que reencontra o princípio da economia de memória mas o eleva a uma escala arquitetural: já não se trata de repetir um ladrilho sobre uma superfície, mas de montar superfícies inteiras a partir de um vocabulário compartilhado de acabamentos. A *trim sheet* exige, em contrapartida, um planejamento cuidadoso na fronteira entre modelagem e texturização — o desdobramento UV deixa de ser uma operação livre para se tornar o ato de "encaixar" a geometria nas faixas disponíveis, e a modelagem passa a ser pensada em função dos acabamentos existentes. Morten Olsen documentou um caso extremo e bem-sucedido: em *Sunset Overdrive* (Insomniac Games, 2014), todo o open world urbano da cidade foi texturizado com apenas seis *trim sheets*, cada uma cobrindo um estilo arquitetônico distinto — a restrição forçou a equipe a desenhar a geometria modular inteiramente em função das faixas disponíveis desde a fase de conceito, e o resultado foi uma cidade coerente e visualmente variada com consumo de memória mínimo.

> ⚠️ **Atenção**
> A *trim sheet* é menos uma técnica de textura do que uma *estratégia de produção* de ambientes inteiros. Planejá-la depois de a modelagem estar pronta força desdobramentos UV torturados sobre geometria não pensada para as faixas disponíveis. O correto é desenhar a *trim sheet* e a geometria em diálogo, desde a fase de conceito.

> **Figura 12.2** — Uma *trim sheet* esquemática organizada em faixas horizontais — rebites, moldura, painel liso, friso ornamental — e, ao lado, três peças de geometria distintas (uma viga, uma porta, um duto) cujas ilhas UV foram encaixadas sobre as mesmas faixas, ilustrando a reutilização.

**Screenshot sugerido:** Captura de um conjunto de ambientes modulares com a sobreposição das ilhas UV sobre uma *trim sheet*, evidenciando como peças diferentes compartilham as mesmas faixas de acabamento.

### Hotspot texturing: encaixe automático em um atlas de variantes

Levando adiante a lógica da *trim sheet*, o *hotspot texturing* (ou *hotspotting*) generaliza a ideia da reutilização para um atlas de retângulos variados. Em vez de faixas em uma direção, monta-se uma textura — o *hotspot* — contendo várias versões de um mesmo tipo de superfície em diferentes proporções e variações: várias larguras de tábua, vários painéis com desgastes distintos, várias placas. Ao desdobrar uma face, em vez de posicioná-la manualmente, o artista (ou uma ferramenta automatizada) "encaixa" a face na variante do atlas cuja proporção melhor corresponde à sua, escolhendo entre as opções disponíveis. O resultado é que faces semelhantes recebem variantes diferentes, e a repetição se dissolve naturalmente, pois a grade regular dá lugar a uma distribuição variada extraída do mesmo atlas.

*Dishonored* (Arkane Studios, 2012) é um dos primeiros usos documentados do *hotspot texturing* em jogo AAA: Viktor Antonov e Ben Lo descreveram como os ambientes vitoriano-steampunk de Dunwall eram montados encaixando cada face da geometria na variante de painel do atlas que melhor correspondia à sua proporção, produzindo uma cidade que aparenta ter sido construída ao longo de décadas de forma orgânica, a partir de um único atlas de variantes.

> 💡 **Dica Profissional**
> O *hotspot texturing* é especialmente poderoso para superfícies compostas de muitos painéis ou tábuas — pisos, paredes de madeira, revestimentos — onde acelera enormemente a texturização e ataca de frente o problema da repetição visível. Na prática, combina-se com a *trim sheet*: a *trim sheet* cobre os acabamentos lineares; o *hotspot* cobre os painéis variados.

O *hotspot texturing* ilustra de modo exemplar a filosofia que percorre este capítulo: a tensão produtiva entre o genérico reutilizável e a aparência de variedade. O atlas é finito e reutilizado — daí a economia —, mas a *seleção* variada entre suas opções produz a impressão de diversidade — daí a qualidade. É, no fundo, a mesma negociação das *trim sheets* e da variação macro, resolvida por um mecanismo distinto. A maturidade profissional está em reconhecer, diante de cada superfície, qual estratégia de repetição rende melhor — e é essa capacidade de decisão, não a execução em um software específico, o que esta apostila busca formar.

## Aplicação em Jogos

Na produção de jogos, as texturas tileables são a espinha dorsal da texturização de ambientes, e raramente um cenário extenso é construído de outro modo. Um artista de ambientes pensa, desde o início, em termos de superfícies que se repetem: define um conjunto de materiais-base tileables — concreto, asfalto, tijolo, metal, terra — que cobrirão a maior parte das áreas, e reserva a texturização única, custosa em memória, para os *assets* heróis de primeiro plano. As *trim sheets* e os kits modulares levam essa lógica à arquitetura: constroem-se peças (paredes, pisos, colunas, vigas) que se encaixam como blocos de montar, todas texturizadas por um punhado de *trim sheets* compartilhadas, permitindo erguer edifícios e corredores inteiros com um custo de textura ínfimo e uma coerência visual garantida pela origem comum.

A consciência da repetição visível permeia todas essas decisões. O artista competente nunca confia que um material tileable, por si só, sustentará uma área extensa; ele planeja, desde o início, as camadas de quebra — a variação macro, a mistura de materiais por máscaras, os *decals* de sujeira e dano — que devolverão irregularidade à superfície. Reencontramos aqui o equilíbrio entre fidelidade e custo que estrutura a apostila: a textura tileable é a face econômica, e as camadas de variação são o investimento seletivo que compra de volta a credibilidade onde ela importa.

## Estudo de Caso

### The Elder Scrolls V: Skyrim — Bethesda Game Studios, 2011

*The Elder Scrolls V: Skyrim* cobre 37 km² de mundo aberto — montanhas nevadas, florestas de pinheiro, tundras árticas, masmorras de pedra, cidades de madeira — com um orçamento de memória de textura que, em 2011, precisava caber nos 512 MB de RAM de vídeo do Xbox 360. A solução foi uma das aplicações mais bem documentadas de texturas tileables em mundo aberto: a Bethesda construiu um conjunto relativamente pequeno de materiais seamless — rocha, neve, terra, madeira, pedra cortada — e os repetiu em toda a extensão do mapa. O resultado viabilizou economicamente um mundo de uma dimensão antes impossível no hardware da época.

A comunidade de modding de *Skyrim* passou uma década estudando e reconstruindo essas texturas, produzindo análises comparativas que são, involuntariamente, os documentos mais didáticos já feitos sobre os limites da textura tileable em mundo aberto. Os projetos *Skyrim HD - 2K Textures* e *Vivid Landscapes*, amplamente documentados no Nexus Mods, identificaram metodicamente os pontos onde a repetição se denuncia: encostas longas de rocha onde o mesmo padrão aparece a intervalos regulares; pisos de masmorras onde o azulejo de pedra cortada forma uma grade óbvia; campos de neve onde a ausência de variação torna a superfície estranhamente estéril. Cada um desses projetos propôs uma solução diferente — texturas de maior resolução, adição de micro-detalhe ao material, mistura de dois padrões por máscara —, criando um arquivo comparativo de estratégias de quebra de repetição aplicadas ao mesmo jogo.

O caso de *Skyrim* também documenta o contraponto fundamental: a repetição funcional em contextos onde a câmera nunca está próxima o suficiente para revelar o padrão. As montanhas no horizonte, os penhascos distantes, o chão das regiões que o jogador atravessa rapidamente sem parar — nessas superfícies, a textura tileable de baixa resolução é absolutamente invisível, e qualquer detalhe extra seria gasto de memória sem retorno visual.

**O que aprender com isso:** A textura tileable tem contextos em que é perfeita e contextos em que precisa de reforço — e reconhecer a diferença é o julgamento profissional que este capítulo ensina.

> **Figura 12.3** — Encosta de pedra em *Skyrim* (2011, versão original) com iluminação oblíqua revelando a repetição regular do padrão de rocha tileable. À direita, a mesma superfície com textura substituída pelo mod *Vivid Landscapes*, onde a adição de ruído macro e mistura de dois padrões eliminam a repetição sem alterar o custo de polígonos.

**Screenshot sugerido:** Captura comparativa da encosta de pedra em Skyrim — versão original com padrão repetitivo visível vs. versão com mod de variação macro aplicado, sob iluminação oblíqua que ressalta as diferenças.

## Boas Práticas

A boa prática fundamental ao trabalhar com texturas que se repetem é planejar a repetição *desde o início*, e não tratá-la como detalhe de acabamento. Isso significa decidir, no começo do projeto, quais superfícies serão cobertas por materiais tileables, quais por *trim sheets* e kits modulares, e onde se justifica a texturização única; significa também planejar, junto, as camadas de quebra da repetição, pois um material tileable nunca deve ser pensado isoladamente da estratégia que impedirá sua monotonia.

Ao construir um ladrilho, garanta as duas faces da condição seamless — continuidade exata nas bordas e homogeneidade do interior, sem feições dominantes que se transformem em grade — e lembre-se de que *todos* os canais de textura do material (cor base, mapa de rugosidade, mapa de normais) precisam ser seamless em conjunto, não apenas a cor. Mantenha a cor base livre de iluminação assada, como exige o PBR da Parte III, e dimensione o detalhe do ladrilho conforme a densidade de texels planejada e a distância de visualização.

> 💡 **Dica Profissional**
> Verifique sempre a textura em modo de repetição, sobre uma área extensa e na escala real de uso, antes de aprová-la. Defeitos de tiling só se revelam na repetição e na distância corretas — uma textura que parece perfeita isolada pode apresentar padrões evidentes quando repetida em grade.

## Erros Comuns

O erro mais elementar é produzir um ladrilho que satisfaz apenas uma das faces da condição seamless: encaixar as bordas perfeitamente, mas deixar no interior uma feição dominante — uma mancha, um objeto, uma variação tonal forte — que, ao se repetir, forma um padrão em grade tão denunciador quanto uma costura. Próximo dele está o erro de tornar seamless apenas a cor base, esquecendo que o mapa de rugosidade e o mapa de normais precisam ladrilhar igualmente; uma costura no mapa de normais, invisível na cor, aparece como uma linha de relevo sob a luz rasante. Há também o erro de embutir iluminação na textura — sombras e realces capturados de uma fotografia — que, repetidos, produzem uma direção de luz impossível, repetida em grade, contradizendo o paradigma PBR.

Outro equívoco frequente é confiar que um bom material tileable basta, ignorando a repetição visível e entregando superfícies extensas monótonas, sem nenhuma camada de quebra. O inverso também ocorre: combater a repetição com força bruta, usando texturas únicas e gigantes onde uma base tileable com variação resolveria a um custo muito menor, esgotando a memória e derrubando a densidade de texels. No trabalho com *trim sheets*, o erro típico é planejá-las tarde demais, depois de a modelagem já estar feita, forçando desdobramentos UV torturados para encaixar geometria que não foi pensada para as faixas disponíveis. E persiste o erro de aprovar uma textura vendo-a isolada, num único quadrado, sem nunca verificá-la repetida sobre uma área extensa e na escala real.

## Resumo

Este capítulo abriu a Parte IV, dedicada aos fluxos de trabalho de produção de texturas, tratando do método mais econômico e onipresente: a textura que se repete. Vimos que a razão de existir das texturas tileables é a economia de memória — multiplicar área a custo constante, cobrindo grandes superfícies com a nitidez de um ladrilho pequeno e de alta densidade de texels —, e que sua limitação fundadora é a natureza genérica: o que se repete não pode ser singular. Estudamos a condição seamless como uma dupla exigência — continuidade exata nas bordas, pensada sobre o domínio toroidal, e homogeneidade do interior, sem feições dominantes — e os dois caminhos de construção, o fotográfico (tornar ladrilhável por correção) e o procedural (nascer ladrilhável por princípio, tema do próximo capítulo). Enfrentamos o principal defeito da repetição, o tiling visível, e as estratégias para combatê-lo: variação macro, multiplicação de variantes, ladrilhamento estocástico e, sobretudo, a sobreposição de elementos singulares sobre a base genérica. Por fim, apresentamos duas evoluções da repetição — as *trim sheets*, que organizam acabamentos em faixas reutilizáveis e articulam modelagem, desdobramento UV e textura na construção de ambientes modulares, e o *hotspot texturing*, que encaixa a geometria em um atlas de variantes, dissolvendo a repetição pela seleção variada.

## Exercícios

**1.** Explique, recorrendo ao conceito de densidade de texels da Parte II, por que as texturas tileables são economicamente indispensáveis para cobrir grandes superfícies. Em que sentido essa economia tem como contrapartida a natureza "genérica" da textura, e que tipo de conteúdo ela representa mal?

**2.** A condição seamless tem duas faces: a continuidade nas bordas e a homogeneidade do interior. Explique cada uma, descreva o defeito que surge quando apenas a primeira é satisfeita, e justifique por que uma textura pode ser tecnicamente seamless e ainda assim "tilear mal".

**3.** Diagnóstico: uma parede de tijolos texturizada com um material tileable apresenta, sob luz rasante, uma linha de relevo reta que se repete a cada ladrilho, embora a cor base não mostre nenhuma costura. Identifique a causa provável, indique em qual canal de textura o problema reside e relacione-o ao conteúdo da Parte III sobre o mapa de normais.

**4.** Comparação: contraste as *trim sheets* e o *hotspot texturing* como evoluções da textura tileable. Para cada uma, descreva o tipo de superfície a que melhor se aplica, como articula (ou não) a modelagem e o desdobramento UV, e de que modo ataca o problema da repetição visível. Em que situação você escolheria uma em vez da outra?

**5.** Planejamento: você deve texturizar uma vila medieval extensa — casas de pedra e madeira, ruas de terra batida, muralhas — com orçamento de memória limitado, para rodar em plataformas modestas. Elabore um plano que combine texturas tileables, *trim sheets* e camadas de quebra da repetição, justificando quais superfícies receberão cada estratégia, como você impedirá o tiling visível e onde reservaria a texturização única. Relacione suas escolhas ao equilíbrio entre fidelidade e custo que percorre a apostila.

## Glossário

**Costura (*seam*):** Descontinuidade visível na junção de bordas de um ladrilho seamless mal construído, ou linha de divisão entre ilhas UV no mapeamento UV.

**Densidade de texels:** Relação entre o tamanho físico de uma superfície e a quantidade de pixels de textura que a recobre; mede a nitidez relativa de diferentes superfícies de uma cena.

**Desdobramento UV:** Processo de "abrir" a superfície de uma malha 3D em um plano 2D para que uma textura possa ser aplicada com controle sobre a distribuição e a escala.

**Hotspot texturing:** Técnica de texturização em que as faces da malha são encaixadas em variantes pré-definidas de um atlas, produzindo diversidade visual a partir de uma única textura reutilizada.

**Ilhas UV:** Regiões contínuas da superfície de uma malha separadas no espaço UV após o desdobramento; cada ilha é uma "peça" plana do mapeamento UV.

**Mapeamento UV:** Sistema de coordenadas bidimensionais (U horizontal, V vertical) que define como uma textura se projeta sobre a superfície de uma malha 3D.

**Seamless:** Propriedade de uma textura cujas bordas opostas se encaixam perfeitamente, permitindo repetição sem costura visível.

**Tileable (ladrilhável):** Propriedade de uma textura projetada para ser repetida lado a lado cobrindo uma superfície maior do que a imagem em si.

**Tiling visível:** Fenômeno perceptivo em que a repetição periódica de um ladrilho se torna reconhecível como padrão artificial, desfazendo a ilusão de superfície contínua.

**Trim sheet (folha de acabamentos):** Textura organizada em faixas horizontais tileables, cada uma com um acabamento distinto, usada para texturizar grandes volumes de geometria arquitetônica e modular com um único material.

**Variação macro:** Textura de baixa frequência e grande escala combinada a uma base tileable para modular sua aparência, quebrando a periodicidade sem alterar o detalhe fino.

## Leituras Complementares

- **[OLSEN, Morten. *The Ultimate Trim — Texturing Techniques of Sunset Overdrive*, Insomniac Games]** — Apresentação que fundamenta a técnica das *trim sheets* e a filosofia de reutilização de acabamentos na produção de ambientes; base direta da seção sobre *trim sheets* deste capítulo. Leitura essencial para quem deseja aprofundar a estratégia de produção de ambientes modulares.

- **[*Hotspot Texturing* (Default Interactive) — https://www.defaultinteractive.co.uk/post/hotspot-texturing]** — Discussão prática da técnica de *hotspotting* e de seu papel no combate à repetição e na aceleração da texturização de ambientes. Recomendado como complemento direto à seção correspondente deste capítulo.

- **[AKENINE-MÖLLER; HAINES; HOFFMAN. *Real-Time Rendering*, 4. ed. CRC Press, 2018]** — Os capítulos sobre *texturing*, mapeamento e endereçamento de texturas (repetição, *wrapping*) fundamentam a base técnica do ladrilhamento tratado aqui.

- **[*Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/]** — Seções sobre materiais, repetição de UVs, mistura de materiais em terrenos e construção modular de ambientes. Útil para observar como a Unreal concretiza as técnicas deste capítulo.

- **[*Blender Manual* — https://docs.blender.org/]** — Seções sobre texturas, geração de texturas seamless e desdobramento UV aplicado a *trim sheets*. Recomendado para praticar os conceitos em uma ferramenta de código aberto antes de migrar para ferramentas especializadas.
