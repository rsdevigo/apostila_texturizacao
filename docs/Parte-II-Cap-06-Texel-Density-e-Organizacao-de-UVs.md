# Capítulo 6 — Texel Density e Organização de UVs

## Introdução

O capítulo anterior deixou as ilhas abertas e verificadas, mas com uma questão deliberadamente adiada: qual deve ser o tamanho relativo de cada ilha e como devem todas se arrumar dentro do espaço UV? Essa pergunta, que parece menor, governa de fato a nitidez percebida de cada superfície do jogo e o aproveitamento de cada quilobyte de memória de textura. Respondê-la com rigor exige introduzir o conceito que dá nome a este capítulo: a **densidade de texels**, a medida que quantifica quanta resolução de textura recai sobre cada porção da superfície do modelo.

A palavra *texel* — contração de *texture element* — designa o elemento da textura análogo ao que o *pixel* é para a imagem de tela: a menor unidade de cor de um mapa. Já encontramos os texels implicitamente na Parte I, quando se discutiu resolução de textura; aqui eles assumem o papel central. A densidade de texels é o elo quantitativo entre o tamanho da textura, em texels, e o tamanho do objeto, no mundo do jogo, e é por meio dela que se garante o que talvez seja o requisito de qualidade mais subestimado por iniciantes: a **consistência**. Quando objetos vizinhos numa mesma cena têm densidades de texels muito diferentes, um aparece nítido e o outro borrado, e a quebra de coerência denuncia o amadorismo da produção tão claramente quanto um erro de iluminação. Este capítulo ensina a medir, padronizar e organizar a densidade, fechando o ciclo da preparação geométrica antes que a textura comece a ser pintada.

## Desenvolvimento

### O conceito de densidade de texels

A densidade de texels é, em termos simples, **quantos texels de textura cobrem uma dada medida de superfície real do modelo**. A unidade mais difundida na indústria é o número de pixels por unidade métrica — tipicamente *pixels por centímetro* (px/cm) ou *pixels por metro* —, embora a escolha da unidade importe menos do que a coerência em mantê-la ao longo de um projeto. Uma densidade de 10 px/cm significa que cada centímetro da superfície do objeto, medido no mundo do jogo, recebe dez texels de textura. Quanto maior a densidade, mais detalhe a superfície pode exibir antes de parecer borrada quando vista de perto; quanto menor, mais cedo a textura revela seus texels individuais e parece de baixa resolução.

O ponto crucial é que a densidade de texels depende de três grandezas relacionadas: o tamanho físico do objeto, o tamanho da ilha UV que o representa e a resolução da textura. Aumentar a resolução da textura eleva a densidade; ampliar a ilha UV de uma parte (dando-lhe mais espaço no quadrado de 0 a 1) eleva a densidade daquela parte às custas das demais; e um objeto fisicamente maior, com a mesma textura e o mesmo tamanho de ilha, terá densidade menor, pois a mesma quantidade de texels precisa cobrir mais superfície. Essas três grandezas formam um sistema, e manipulá-las conscientemente é o que permite controlar a nitidez de cada *asset*.

> Figura: Esquema relacionando as três grandezas — tamanho físico do objeto, área da ilha UV e resolução da textura — com a densidade de texels resultante, indicando com setas o sentido em que cada variação aumenta ou diminui a densidade.
>
> **Screenshot sugerido:**
> Captura de uma ferramenta que exibe numericamente a densidade de texels de uma seleção, mostrando o valor em px/cm ao lado do modelo medido.

### Por que a consistência importa

Imagine uma sala de jogo com uma mesa, uma cadeira e uma caneca sobre a mesa. Se a caneca foi texturizada com densidade alta e a mesa com densidade baixa, o jogador que se aproxima vê uma caneca cristalina pousada sobre uma mesa borrada — e, embora talvez não saiba nomear o problema, percebe que algo está errado. A consistência de densidade entre objetos que coabitam uma cena é o que dá unidade visual ao mundo. Por isso, equipes profissionais definem, no início do projeto, um **padrão de densidade de texels** — um valor-alvo em px/cm — que todos os *assets* daquela categoria devem respeitar. Esse padrão funciona como uma régua compartilhada: cada artista, ao desdobrar um modelo, ajusta o tamanho de suas ilhas até que a densidade bata com o alvo, garantindo que sua peça conviva harmonicamente com as dos colegas.

O padrão não é único para o jogo inteiro. É comum estabelecer densidades-alvo diferentes por categoria de *asset* conforme a proximidade típica com que o jogador os observa: personagens jogáveis e objetos de interação, vistos de perto, recebem densidade alta; objetos de cenário a média distância, densidade intermediária; elementos de fundo e paisagem distante, densidade baixa. Trata-se, novamente, da economia de gastar resolução onde ela rende. O que jamais se admite é a inconsistência *dentro* de uma mesma categoria ou entre objetos que aparecem juntos e à mesma distância.

### A textura de verificação como medidor de densidade

A textura de quadriculado, apresentada no Capítulo 4 como reveladora de distorção, presta-se igualmente a verificar a densidade — e é, na prática, a ferramenta mais usada para isso. Quando o xadrez tem quadrados de tamanho conhecido, basta observar o tamanho com que eles aparecem sobre a superfície: quadrados maiores indicam densidade menor; quadrados menores, densidade maior. Aplicando o mesmo xadrez a todos os objetos de uma cena, o artista enxerga imediatamente as discrepâncias — onde os quadrados destoam em tamanho de um objeto para o outro, as densidades estão desiguais. Assim, o mesmo instrumento que diagnostica distorção (quadrados deformados) diagnostica também inconsistência de densidade (quadrados de tamanhos diferentes entre peças), tornando-se o companheiro permanente da preparação de UVs.

> Figura: Cena com vários objetos cobertos pela mesma textura de verificação — alguns com quadrados uniformes entre si, indicando densidade consistente, e um objeto destoante com quadrados visivelmente maiores, sinalizando densidade insuficiente.
>
> **Screenshot sugerido:**
> Captura de uma cena de teste com móveis e objetos recebendo o mesmo xadrez numerado, evidenciando uma peça com densidade fora do padrão.

### A organização das ilhas: empacotamento

Definida a densidade, resta arrumar as ilhas dentro do quadrado de 0 a 1 — o processo chamado de **empacotamento** (*packing*). O objetivo é encaixar todas as ilhas, respeitando seus tamanhos relativos (que a densidade já determinou), de modo a deixar o mínimo possível de espaço vazio, pois cada região não ocupada é resolução de textura desperdiçada. Um bom empacotamento aproxima as ilhas, gira-as quando isso permite encaixe melhor e preenche os interstícios com ilhas menores, como em um quebra-cabeça que busca a maior cobertura possível.

O empacotamento, porém, não pode ser apertado ao ponto de as ilhas se tocarem, e a razão disso introduz um conceito técnico importante: o **preenchimento de borda** ou *padding*. Trata-se de uma margem de segurança deixada ao redor de cada ilha. Ela é necessária por dois motivos ligados ao funcionamento do motor. O primeiro é o **mipmapping**, técnica mencionada na Parte I pela qual o motor gera versões progressivamente menores da textura para objetos distantes; nessas versões reduzidas, ilhas muito próximas começam a "sangrar" cor umas sobre as outras, criando bordas espúrias, e o *padding* impede esse vazamento. O segundo é a **filtragem bilinear**, o processo pelo qual o motor mistura texels vizinhos para suavizar a textura; sem margem, essa mistura captura cores da ilha adjacente, produzindo emendas coloridas indevidas. Por isso, deixa-se sempre um respiro entre as ilhas e entre as ilhas e a borda do quadrado UV, dimensionado conforme a resolução da textura e o número de níveis de *mipmap* previstos.

### Reaproveitamento de espaço: sobreposição, espelhamento e UDIMs

Há técnicas que ampliam a densidade efetiva para além do que o quadrado único comportaria. A primeira, já antecipada no capítulo anterior, é a **sobreposição** (*overlapping*) de ilhas idênticas ou espelhadas: partes repetidas de um modelo — os dois lados simétricos, os elos iguais de uma corrente, os parafusos idênticos de uma máquina — podem ocupar o mesmo espaço UV, de modo que uma só região de textura sirva a várias partes da geometria. Como vimos, isso multiplica a resolução efetiva, ao custo de essas partes ficarem visualmente idênticas, sem variação individual.

Uma técnica relacionada é o **stacking**, em que ilhas semelhantes mas não necessariamente espelhadas são empilhadas para compartilhar textura. E, para *assets* que exigem altíssima densidade sem comprometer a memória num único mapa gigante, a indústria emprega os **UDIMs**: um sistema que estende o espaço UV para além do quadrado de 0 a 1, organizando as ilhas em uma grade de vários quadrados (*tiles*), cada um com sua própria textura. Os UDIMs são comuns em produção de cinema e em *assets* de herói de altíssima fidelidade, mas precisam ser usados com cautela em jogos, pois cada *tile* adicional consome memória e nem todo motor ou fluxo de tempo real os trata com a mesma naturalidade. A escolha entre adensar por sobreposição, por UDIMs ou simplesmente elevando a resolução de um mapa único é, mais uma vez, um compromisso entre fidelidade visual e orçamento de memória — o mesmo equilíbrio que perpassa toda a disciplina.

> Figura: Comparação entre três estratégias para um mesmo modelo simétrico com peças repetidas — layout sem reaproveitamento (cada peça com seu espaço), layout com sobreposição das peças simétricas e repetidas, e layout em UDIMs distribuído por vários *tiles* —, com a densidade efetiva resultante anotada em cada caso.
>
> **Screenshot sugerido:**
> Captura do editor UV mostrando ilhas sobrepostas (peças simétricas empilhadas) e, ao lado, uma disposição em grade de UDIMs com vários quadrados numerados.

### Orientação e legibilidade do layout

Um último cuidado da organização, frequentemente negligenciado, é a **orientação** das ilhas. Manter as ilhas alinhadas com os eixos horizontal e vertical do espaço UV, sempre que a forma permitir, traz benefícios concretos: facilita o trabalho posterior de pintura, reduz o serrilhado em detalhes retos como linhas e bordas, e torna o layout mais legível para qualquer pessoa da equipe que precise dar manutenção ao *asset*. Um layout bem orientado e organizado é também uma forma de documentação: outro artista consegue, ao abri-lo, entender rapidamente que parte do modelo corresponde a cada ilha. A organização de UVs, portanto, não serve apenas à máquina e à eficiência de memória; serve também à colaboração humana dentro de um pipeline, preocupação que o próximo capítulo desenvolverá ao tratar da preparação de *assets* para produção.

## Aplicação em Jogos

A densidade de texels é, na prática de produção, uma decisão tomada antes de qualquer *asset* ser desdobrado: o estúdio define os valores-alvo por categoria e os documenta como parte do guia de estilo técnico do projeto. Essa definição prévia é o que permite que dezenas de artistas trabalhem em paralelo e que suas peças, ao se encontrarem na cena montada, exibam nitidez coerente. Sem esse padrão compartilhado, cada artista calibraria a densidade por intuição própria, e o resultado seria o mosaico inconsistente que delata produções amadoras. O conceito é, assim, tanto técnico quanto organizacional: ele coordena o trabalho de uma equipe inteira em torno de uma régua comum.

O empacotamento e as técnicas de reaproveitamento têm impacto direto no orçamento de memória do jogo, um recurso especialmente crítico em plataformas modestas e em jogos com mundos grandes. Aproveitar bem o espaço UV significa entregar mais nitidez com a mesma textura, ou a mesma nitidez com uma textura menor — em ambos os casos, uma vitória econômica. A sobreposição de peças simétricas e repetidas é prática rotineira em personagens, veículos e arquitetura modular, e o domínio do *padding* evita uma classe inteira de defeitos de borda que, de outro modo, só apareceriam tarde, já no jogo rodando. Compreender densidade e organização é, em última análise, compreender como transformar um recurso escasso — a memória de textura — no máximo de qualidade percebida.

## Estudo de Caso

### Horizon Zero Dawn e a Consistência de Texel Density em Mundo Aberto — Guerrilla Games, 2017

![Aloy em frente a um Thunderjaw, máquina de grande porte, em floresta de Horizon Zero Dawn — personagem orgânica e maquinário metálico no mesmo frame](imagens/cap06-horizon-zero-dawn-texel-density.jpg)

<!-- Imagem: captura de tela oficial de Horizon Zero Dawn, Guerrilla Games — disponível em https://www.guerrilla-games.com/games/horizon-zero-dawn (press section) ou via PlayStation Media Center em https://media.playstation.com -->

*Horizon Zero Dawn* coloca num único frame dois tipos radicalmente distintos de *asset*: Aloy, uma personagem com pele orgânica, cabelos, tecido, couro e madeira — e as máquinas, criaturas mecânicas de tamanhos que variam de um gato a um edifício de três andares. Em um mundo aberto onde a câmera livre pode se aproximar de qualquer objeto a qualquer momento, a inconsistência de texel density não tem onde se esconder. A Guerrilla documentou no GDC 2017 o desafio central que isso representou: como garantir que o close no rosto de Aloy, a pata do Thunderjaw e o tronco de uma árvore ao fundo exibissem todos a mesma nitidez relativa ao espaço que cada superfície ocupa na tela?

A resposta da equipe foi institucionalizar a densidade como uma especificação de projeto, não uma intuição artística. O time de arte técnica definiu um valor-alvo de texels por metro para cada categoria: personagens principais, criaturas de médio porte, criaturas de grande porte, elementos de cenário de primeiro plano, vegetação de fundo. Essa tabela foi convertida em uma ferramenta de visualização interna ao motor que sobrepunha um mapa de calor em cada objeto da cena — verde para densidade dentro do alvo, vermelho para abaixo, azul para acima —, permitindo ao artista de qualidade percorrer o mundo e identificar inconsistências a uma distância de um olhar. A ferramenta tornou visível o que, de outro modo, só seria detectável pela comparação tedosa de xadrez em objeto por objeto.

O caso mais instrutivo documentado pela Guerrilla envolveu exatamente o tipo de problema descrito no capítulo: as máquinas de grande porte, cujas superfícies físicas são vastamente maiores que as de Aloy, chegavam ao pipeline com mapas de mesma resolução que o personagem — e portanto com densidade efetiva muito menor. Elevar a resolução de cada mapa foi a solução óbvia, mas cara demais em memória; a solução real foi uma combinação de técnicas: sobreposição de peças repetidas (o Thunderjaw tem muitos painéis idênticos nas laterais do corpo), *trim sheets* para os acabamentos metálicos das arestas, e aumento seletivo de resolução apenas nas peças de maior visibilidade, como a cabeça e os ombros. O resultado foi uma cena onde Aloy, olhando de cima para o Thunderjaw, ou o Thunderjaw olhando de baixo para Aloy, exibiam nitidez coerente — não porque tinham mapas iguais, mas porque a densidade havia sido equalizada por meios distintos em cada caso.

> **Figura 6.1** — Frame de *Horizon Zero Dawn* com Aloy em frente a um Thunderjaw em ambiente florestal. A mesma nitidez de textura é mantida na pele de Aloy, nas placas metálicas do Thunderjaw e no tronco das árvores em primeiro plano — resultado de uma definição de texel density por categoria aplicada por ferramenta interna de visualização.
>
> **Fonte da imagem:** Guerrilla Games — *Horizon Zero Dawn* (2017). Capturas de tela oficiais disponíveis em: `https://media.playstation.com/en/us/games/horizon-zero-dawn/` (PlayStation Media Center, acesso público).

## Boas Práticas

A boa prática que organiza este capítulo é estabelecer um valor-alvo de densidade de texels no início do projeto, documentá-lo por categoria de *asset* e medir cada modelo contra esse alvo usando a textura de verificação. Tratar a densidade como uma régua compartilhada pela equipe, e não como uma intuição individual, é o que garante a consistência visual da cena montada. Dimensionar as ilhas para igualar a densidade entre objetos de tamanhos físicos diferentes — em vez de dar a todos a mesma resolução de mapa — é o corolário direto desse princípio.

No empacotamento, recomenda-se buscar a maior cobertura possível do espaço UV, girando e encaixando as ilhas como num quebra-cabeça, mas sempre reservando o *padding* adequado para evitar o sangramento de cores no mipmapping e na filtragem. É boa prática aproveitar a sobreposição de peças simétricas e repetidas para elevar a densidade efetiva onde a variação individual não for necessária, e considerar UDIMs apenas quando a fidelidade exigida justificar seu custo de memória e o motor os suportar bem. Por fim, manter as ilhas orientadas e o layout legível serve tanto à qualidade técnica quanto à colaboração da equipe, transformando o layout UV em uma peça de documentação que o próximo da fila consegue entender.

## Erros Comuns

O erro mais consequente é ignorar a densidade de texels e dimensionar as ilhas a olho, produzindo a inconsistência em que objetos vizinhos exibem nitidez díspar — o sintoma mais delator de uma produção sem padrão técnico. Aparentado a ele está o erro de combater a baixa densidade de um objeto grande apenas inflando a resolução do mapa, desperdiçando memória quando a sobreposição de partes repetidas resolveria o problema com mais economia.

No empacotamento, o erro clássico é dispensar o *padding*, colando as ilhas umas nas outras e descobrindo, só quando o objeto se afasta e o mipmapping age, que as cores vazam entre ilhas e surgem emendas espúrias — defeito difícil de rastrear para quem não conhece sua causa. O oposto também ocorre: deixar espaço vazio em excesso por empacotamento descuidado, desperdiçando resolução paga. Há ainda o erro de sobrepor ilhas que não deveriam ser idênticas, fazendo aparecerem repetições visíveis e indesejadas onde se esperava variação, ou de adotar UDIMs num projeto de tempo real sem avaliar o custo de memória e o suporte do motor. E persiste o erro de mentalidade de tratar a organização de UVs como arrumação cosmética, sem perceber que ela decide, ao mesmo tempo, a nitidez de cada superfície, o consumo de memória e a facilidade de manutenção do *asset*.

## Resumo

Este capítulo introduziu a densidade de texels — quantos texels de textura recaem sobre cada medida de superfície real — como a medida que governa a nitidez percebida e o uso de memória. Vimos que ela resulta da relação entre três grandezas, o tamanho físico do objeto, a área da ilha UV e a resolução da textura, e que manipulá-las conscientemente é o que permite controlar a qualidade de cada *asset*. Estabelecemos a consistência como requisito central: objetos que coabitam uma cena à mesma distância devem partilhar a mesma densidade, sob pena de a disparidade denunciar amadorismo, e por isso a indústria define valores-alvo por categoria, como uma régua compartilhada pela equipe. Mostramos que a textura de verificação, já conhecida como reveladora de distorção, é também o medidor prático de densidade, pois o tamanho dos quadrados denuncia discrepâncias. Tratamos da organização das ilhas pelo empacotamento, que busca a máxima cobertura do espaço UV preservando o *padding* necessário para evitar o sangramento de cores no mipmapping e na filtragem bilinear. Apresentamos as técnicas de reaproveitamento — sobreposição de peças simétricas e repetidas, *stacking* e UDIMs — como meios de elevar a densidade efetiva, cada qual com seu compromisso entre fidelidade e memória. E destacamos a orientação e a legibilidade do layout como serviço tanto à qualidade técnica quanto à colaboração humana. Resolvidas a forma das ilhas, suas costuras, seus tamanhos e seu arranjo, o modelo está geometricamente pronto; falta integrá-lo ao pipeline de produção, com nomenclatura, escala, organização e exportação corretas — o tema do próximo capítulo.

## Exercícios

1. Defina densidade de texels com suas próprias palavras e explique como cada uma das três grandezas relacionadas — tamanho físico do objeto, área da ilha UV e resolução da textura — afeta o seu valor. Se um objeto aparece borrado, quais dessas grandezas você poderia ajustar para corrigi-lo, e que consequência cada ajuste traz?

2. Explique por que a consistência de densidade entre objetos de uma mesma cena é importante e por que a indústria define valores-alvo diferentes por categoria de *asset*. Dê um exemplo de duas categorias que mereceriam densidades distintas e justifique.

3. Diagnóstico: numa cena, uma estátua grande aparece borrada ao lado de pequenos vasos nítidos, embora todos usem texturas de mesma resolução. Explique a causa provável em termos de densidade de texels e proponha duas soluções diferentes, comparando seu custo de memória.

4. Explique o que é o *padding* e por que ele é necessário, relacionando sua resposta ao mipmapping e à filtragem bilinear. O que acontece, concretamente, quando se empacotam as ilhas sem deixar margem alguma?

5. Comparação: contraste três estratégias para elevar a densidade efetiva de um *asset* — aumentar a resolução do mapa, sobrepor ilhas de partes repetidas e usar UDIMs. Em que situação cada uma é preferível, considerando fidelidade visual, consumo de memória e adequação ao tempo real?

## Leituras Complementares

### Material de referência do projeto

- *AOD — Texel Density (2020)* (material de apoio da disciplina) — referência aplicada sobre como medir, padronizar e verificar a densidade de texels em um fluxo de produção, incluindo o uso de mapas de verificação (*checker maps*).
- *AOD — TD Checker Maps* (material de apoio da disciplina) — conjunto de texturas de verificação para inspeção de densidade e distorção, instrumento prático discutido neste capítulo.

### Fontes complementares externas

- IEZZI, Leonardo. *All You Need to Know about Texel Density*. ArtStation, 2016 (artstation.com/artwork/qbOqP) — guia prático amplamente adotado na indústria sobre cálculo e padronização de densidade de texels.
- POLYCOUNT WIKI, verbetes *Texel* e *Texel Density* (wiki.polycount.com) — conhecimento consolidado da comunidade sobre densidade, empacotamento e *padding*.
- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. As seções sobre mipmapping e filtragem de texturas fundamentam tecnicamente a necessidade de *padding* e o comportamento das texturas a distância.
- Documentação da Unreal Engine (dev.epicgames.com) e da Unity (docs.unity3d.com), seções sobre importação de texturas, mipmaps e UDIMs — para observar como cada motor trata a resolução, a filtragem e os múltiplos *tiles* discutidos aqui.
