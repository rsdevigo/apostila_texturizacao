# Capítulo 4 — Coordenadas UV e Projeções de Textura

## Introdução

A Parte I encerrou-se com uma constatação que organiza tudo o que vem a seguir: a aparência de uma superfície é o resultado de um cálculo executado pelo shader, e os mapas de textura são as entradas desse cálculo. Ficou, porém, uma pergunta em aberto. Quando o motor precisa saber qual cor do mapa de cor base corresponde a um ponto específico da malha — a ponta do nariz de um personagem, o canto de um tijolo, o centro de uma maçaneta —, como ele estabelece essa correspondência? Uma textura é uma imagem bidimensional, um retângulo de pixels; um modelo tridimensional é uma superfície que se dobra e se curva no espaço. Entre esses dois objetos de naturezas tão distintas é preciso haver uma ponte, e essa ponte é o sistema de coordenadas UV.

Este capítulo abre a Parte II, dedicada à preparação de modelos para receber texturas. Antes de pintar qualquer mapa, antes de aplicar qualquer material, é necessário resolver o problema de fundo: como "vestir" uma superfície tridimensional com uma imagem plana sem que essa imagem fique deformada, repetida de modo indevido ou mal posicionada. A resposta começa pelas coordenadas UV e pelas projeções de textura, assuntos deste capítulo. Eles são o alicerce sobre o qual se erguem os capítulos seguintes — o desdobramento (Capítulo 5), a densidade de texels (Capítulo 6) e a preparação final dos *assets* para produção (Capítulo 7).

A imagem mental que vale a pena cultivar desde já é a do costureiro. Para fazer uma roupa que vista bem um corpo tridimensional, o costureiro corta o tecido — que é plano — em pedaços de formatos cuidadosamente escolhidos e os costura ao redor do corpo. As coordenadas UV são, em essência, o registro de como o tecido plano da textura se assenta sobre a superfície do modelo. Compreender esse registro é o primeiro passo de todo o trabalho de texturização.

## Desenvolvimento

### O que são coordenadas UV

Uma textura é endereçada por duas coordenadas que percorrem a imagem. Para não confundi-las com as coordenadas espaciais X, Y e Z do modelo tridimensional, convencionou-se chamá-las de **U** e **V** — as duas letras imediatamente anteriores ao X no alfabeto. A coordenada U corre, por convenção, na horizontal da imagem, e a coordenada V na vertical. O espaço definido por elas é chamado de **espaço UV**, e é nele que todo o trabalho de mapeamento acontece.

A característica decisiva desse espaço é que ele é **normalizado**: independentemente de a textura ter 256, 1024 ou 4096 pixels de lado, suas coordenadas vão sempre de 0 a 1. O ponto (0, 0) corresponde a um canto da imagem e o ponto (1, 1) ao canto oposto; o ponto (0,5, 0,5) está sempre no centro, qualquer que seja a resolução. Essa normalização é uma decisão de projeto profunda e libertadora: ela desacopla o mapeamento da resolução da textura. Um modelo mapeado uma única vez pode receber uma textura de baixa resolução durante a produção e uma de alta resolução na entrega final, sem qualquer alteração no mapeamento, porque as coordenadas UV não descrevem pixels, e sim posições proporcionais dentro da imagem. Reencontramos aqui o mesmo princípio de independência que, na Parte I, separava o material da geometria: também o mapeamento se mantém independente da resolução do mapa que o preencherá.

Concretamente, cada vértice da malha carrega, além de sua posição no espaço tridimensional, um par de coordenadas UV que diz a que ponto da textura ele corresponde. Quando o shader precisa colorir o interior de um triângulo da malha, ele interpola as coordenadas UV dos três vértices — exatamente como interpola posições e direções —, obtendo, para cada ponto interno, a coordenada UV correspondente, e com ela consulta a textura. É assim que a imagem plana "gruda" na superfície curva: ponto a ponto, o shader pergunta ao mapeamento UV onde buscar a cor.

> Figura: Diagrama lado a lado mostrando, à esquerda, um modelo 3D simples (um cubo) com um vértice destacado e suas coordenadas espaciais (X, Y, Z); à direita, o quadrado do espaço UV de 0 a 1 com o ponto correspondente assinalado por suas coordenadas (U, V), ligados por uma seta que representa a correspondência.
>
> **Screenshot sugerido:**
> Captura de um editor 3D com a janela de visualização do modelo ao lado da janela do editor UV, com o mesmo vértice selecionado e realçado em ambas, evidenciando a ligação entre a posição no espaço e a posição na textura.

### A ilha, o seam e o layout UV

Salvo em casos triviais, uma superfície tridimensional fechada não pode ser aberta sobre um plano sem ser cortada — é a mesma impossibilidade que faz com que nenhum mapa-múndi represente a Terra sem distorção. Para assentar a textura, portanto, é necessário introduzir cortes na superfície, ao longo dos quais ela poderá ser "descascada" e estendida sobre o espaço UV. Cada um desses cortes é chamado de **costura** ou *seam*, em analogia direta com a costura de uma roupa, e cada pedaço contínuo de superfície resultante, já estendido no espaço UV, é chamado de **ilha UV** (*UV island* ou *UV shell*).

O conjunto de todas as ilhas dispostas dentro do quadrado de 0 a 1 constitui o **layout UV** do modelo — o mapa de como a superfície foi recortada e arrumada para receber a textura. Esse layout é o documento central da preparação de *assets*: é ele que determina onde, na textura, ficará cada parte do modelo, quanto espaço cada parte ocupará e por onde passam os cortes. Os Capítulos 5 e 6 tratarão, respectivamente, de como decidir onde cortar e de como arrumar e dimensionar as ilhas; por ora, basta fixar o vocabulário: cortamos a superfície em costuras, obtemos ilhas e as organizamos num layout dentro do espaço UV.

### Os modos de endereçamento fora do intervalo

Embora o intervalo canônico das coordenadas UV vá de 0 a 1, nada impede, matematicamente, que uma coordenada assuma valores fora desse intervalo — por exemplo, 2,5 ou −0,3. O que o shader faz nesses casos é determinado pelo **modo de endereçamento** (*wrap mode*) configurado para a textura, e compreender esses modos é importante porque eles habilitam técnicas largamente usadas em jogos.

O modo mais comum é o de **repetição** (*repeat* ou *tile*): valores fora do intervalo "dão a volta", de modo que a coordenada 1,5 é tratada como 0,5 e a textura se repete indefinidamente em ladrilhos. É esse comportamento que permite cobrir uma parede inteira, um piso ou um terreno com uma textura pequena que se repete, técnica que examinaremos adiante sob o nome de *tiling*. Uma variante, a **repetição espelhada** (*mirror*), inverte a imagem a cada repetição, fazendo as bordas coincidirem e disfarçando a emenda. Há ainda o modo de **fixação** (*clamp*), que, em vez de repetir, estende indefinidamente a cor da borda da textura para qualquer valor fora do intervalo, útil quando não se deseja repetição alguma. A escolha do modo de endereçamento é, portanto, uma decisão de mapeamento tão relevante quanto o próprio layout das ilhas.

### As projeções: maneiras automáticas de gerar coordenadas

Atribuir manualmente coordenadas UV a cada vértice seria impraticável. Por isso, as ferramentas oferecem métodos automáticos de **projeção**, que calculam as coordenadas UV lançando a geometria sobre uma forma de referência simples. Cada tipo de projeção se ajusta bem a uma classe de formas, e conhecê-los é indispensável porque eles são tanto ferramentas de trabalho diretas quanto o ponto de partida do desdobramento manual.

A **projeção planar** lança a geometria sobre um plano, como um projetor de slides que ilumina a superfície de uma única direção. É ideal para superfícies majoritariamente chatas — uma parede, o tampo de uma mesa, um chão, uma folha de papel. Seu ponto fraco é igualmente claro: as faces paralelas à direção da projeção, que a recebem "de raspão", ficam severamente esticadas, pois recebem pouquíssima área de textura. A projeção planar é excelente para o que é plano e desastrosa para o que escapa do plano.

A **projeção cilíndrica** envolve a geometria em um cilindro e projeta a textura radialmente, como um rótulo enrolado ao redor de uma lata. Serve naturalmente a formas tubulares ou aproximadamente cilíndricas: troncos de árvore, canos, garrafas, torsos, colunas, braços e pernas. Suas regiões problemáticas são as tampas, no topo e na base do cilindro, que tendem a distorcer porque ali a superfície deixa de acompanhar a forma cilíndrica.

A **projeção esférica** envolve a geometria em uma esfera, projetando a textura como os meridianos e paralelos de um globo. É a escolha natural para formas arredondadas e fechadas: planetas, cabeças estilizadas, frutas, domos. Seu defeito característico aparece nos polos, onde os meridianos convergem e a textura se comprime e gira, fenômeno familiar a quem já viu a distorção de um mapa-múndi sobre as regiões árticas.

A **projeção cúbica** ou *box* lança a geometria sobre as seis faces de um cubo, escolhendo, para cada parte da superfície, a face do cubo cuja direção melhor a enfrenta. É uma solução versátil para objetos angulosos e arquitetônicos — caixas, prédios, móveis, peças mecânicas —, pois trata bem qualquer superfície que se aproxime de um dos seis planos do cubo. Sua fragilidade está nas transições entre faces, onde podem surgir emendas e pequenas distorções nas regiões diagonais.

> Figura: Painel comparativo com quatro versões de um mesmo objeto de teste — uma figura que combina partes planas, cilíndricas e esféricas — submetido às projeções planar, cilíndrica, esférica e cúbica, cada uma com uma textura xadrez aplicada para evidenciar onde cada projeção acerta e onde distorce.
>
> **Screenshot sugerido:**
> Captura de um software 3D mostrando o mesmo modelo com cada tipo de projeção aplicado e uma textura de quadriculado, de modo que o estudante veja o esticamento característico de cada método.

### A textura de verificação: tornando a distorção visível

A distorção do mapeamento é, em si, invisível: olhando apenas o layout UV ou apenas o modelo, é difícil perceber que uma região está esticada ou comprimida. A indústria resolve isso com um instrumento simples e poderoso, a **textura de verificação** (*checker map*) — uma imagem de quadriculado regular, frequentemente numerada ou colorida, aplicada provisoriamente ao modelo durante a preparação. Onde os quadrados aparecem perfeitos e do mesmo tamanho, o mapeamento está uniforme; onde se esticam em retângulos, há distorção; onde encolhem, a região recebe densidade de textura excessiva. O xadrez transforma um problema abstrato e numérico em um defeito visível a olho nu, e por isso é uma das primeiras ferramentas que o artista aplica ao avaliar um mapeamento. Voltaremos a ela no Capítulo 6, pois é também o instrumento natural para diagnosticar e uniformizar a densidade de texels.

## Aplicação em Jogos

Na produção de jogos, a escolha entre uma projeção automática e o desdobramento manual cuidadoso é, antes de tudo, uma decisão econômica. Objetos de fundo, terrenos, paredes e pisos frequentemente se contentam com projeções simples combinadas com repetição de textura, porque o jogador raramente os examina de perto e porque essa abordagem custa pouco tempo de produção. Já os objetos que o jogador vê em destaque — o personagem principal, uma arma sempre presente na tela, um objeto de interação central — exigem mapeamento manual criterioso, assunto do próximo capítulo. Saber reconhecer em qual categoria um *asset* se enquadra é parte do julgamento profissional, e evita tanto o desperdício de polir o que ninguém vê quanto o erro de descuidar do que está sob os olhos do jogador o tempo todo.

A repetição de textura, habilitada pelo modo de endereçamento de repetição, é uma das técnicas mais econômicas do meio. Uma única textura de tijolos, de poucos centímetros de lado no espaço da cena, pode revestir a fachada inteira de um edifício se for repetida; uma textura de grama cobre um campo inteiro. O custo de memória é mínimo e o ganho de cobertura é imenso. O preço dessa economia é o risco da repetição perceptível — o olho humano detecta padrões que se repetem, e uma textura ladrilhada de forma descuidada produz a sensação de artificialidade conhecida como *tiling* visível. Bons materiais de repetição são projetados para que suas bordas coincidam (são *seamless*, sem emenda aparente) e para que não contenham elementos marcantes que denunciem a repetição. A combinação dessa técnica com mapeamentos mais sofisticados é o que permite aos jogos cobrir mundos enormes com orçamentos de memória modestos — uma preocupação que reencontra, no domínio do mapeamento, o equilíbrio entre fidelidade e desempenho discutido na Parte I.

## Estudo de Caso

### Quake e a Origem das Coordenadas UV — id Software, 1996

![Corredor e1m1 de Quake com texturas de pedra projetadas no espaço do mundo, mostrando o característico efeito de texture swimming](imagens/cap04-quake-texture-swimming.jpg)

<!-- Imagem: captura de tela de Quake (1996), id Software. O jogo foi relançado com código aberto em 1999; capturas de tela estão amplamente disponíveis. Fonte de referência: Quake Wiki em https://quake.fandom.com/wiki/E1M1:_The_Slipgate_Complex -->

Para entender o valor das coordenadas UV, é preciso conhecer o que existia antes delas — e nenhum exemplo é mais instrutivo do que *Quake* (id Software, 1996). O motor BSP (*Binary Space Partitioning*) de *Quake* projetava as texturas no **espaço do mundo**, não no espaço de cada face. Uma textura de pedra era definida por sua posição, escala e ângulo em relação ao eixo global da cena, e cada polígono exibia o recorte dessa textura que correspondia à sua posição no espaço. O resultado era eficiente e permitia criar cenários complexos com pouco poder computacional, mas criava um problema perceptível que os jogadores da época aprenderam a reconhecer: quando o personagem se movia, as texturas das paredes **nadavam** (*swimming*). Como a textura estava ancorada ao mundo e não à superfície, a perspectiva em movimento revelava que a imagem deslizava sobre a geometria em vez de ser parte dela.

Michael Abrash descreveu em detalhes o sistema de mapeamento do *Quake* no *Graphics Programming Black Book* (Capítulos 67–70), obra que documenta os fundamentos da computação gráfica em tempo real da era id Software. O mesmo livro antecipa os problemas do modelo: Abrash e Carmack sabiam que a projeção planar era uma simplificação, e que objetos de geometria complexa — criaturas, armas — não podiam ser texturizados corretamente por ela. Para esses casos, *Quake* usava *skins*: imagens 2D indexadas diretamente aos polígonos do modelo, o precursor direto das coordenadas UV. Na prática, o mundo estático de *Quake* usava projeção no espaço do mundo, e os modelos dinâmicos usavam um esquema primitivo de UV — dois paradigmas coexistindo no mesmo motor, cada um adequado a um tipo diferente de geometria.

*Quake 2* (1997) deu o passo seguinte: o motor id Tech 2 passou a exigir UVs explícitas por face para toda a geometria de cenário, eliminando o *texture swimming* do predecessor. O efeito foi imediato — as paredes de *Quake 2* permaneciam fixas enquanto o jogador se movia — e o custo foi um trabalho de mapeamento manual para cada superfície do cenário que, nos motores anteriores, era gerado automaticamente pela projeção. Essa transição entre os dois *Quakes* é o momento histórico em que as coordenadas UV se tornaram o padrão universal da texturização em jogos, e permanece a melhor demonstração do que as projeções resolvem, do que falham em resolver, e por que o mapeamento manual por face é insubstituível.

> **Figura 4.1** — Corredor E1M1 (*The Slipgate Complex*) de *Quake*, mostrando a textura de pedra projetada no espaço do mundo. Com o motor em pausa, o padrão das pedras parece correto; em movimento, o deslizamento da textura sobre a geometria revela que a projeção está ancorada ao mundo, não à superfície.
>
> **Fonte da imagem:** id Software — *Quake* (1996). O código-fonte do motor foi liberado sob GPL em 1999; capturas de tela do jogo estão disponíveis em: `https://quake.fandom.com/wiki/E1M1:_The_Slipgate_Complex` (Quake Wiki, licença de uso educacional e documental).

## Boas Práticas

A boa prática que fundamenta todo este capítulo é escolher a projeção segundo a forma da superfície, e não segundo a forma global do objeto. Decompor o modelo em regiões — esta parte é cilíndrica, aquela é plana, aquela outra é arredondada — e aplicar a cada região a projeção que a trata melhor produz resultados muito superiores a forçar um único método sobre a peça inteira. O objeto raramente é uma primitiva pura; o mapeamento competente reconhece isso.

Recomenda-se aplicar uma textura de verificação desde os primeiros momentos da preparação, em vez de deixar para descobrir a distorção quando o mapa final já estiver sendo pintado. Diagnosticar cedo poupa retrabalho caro. É igualmente boa prática manter as coordenadas dentro do intervalo de 0 a 1 sempre que se deseje mapeamento único, reservando conscientemente os valores fora do intervalo para os casos em que a repetição é intencional. E vale, desde já, pensar o mapeamento com vistas à textura que virá: decidir o modo de endereçamento adequado, prever onde a repetição ajudará e onde prejudicará, e tratar o layout UV não como uma formalidade técnica, mas como o projeto de como a imagem vai vestir o modelo.

## Erros Comuns

O erro mais frequente entre iniciantes é aplicar uma única projeção automática ao objeto inteiro e aceitar o resultado sem inspecioná-lo, ignorando o esticamento severo nas regiões que a projeção não sabe tratar. Uma esfera projetada cilindricamente, uma peça angulosa projetada esfericamente ou uma forma composta projetada por um único método produzem distorções que comprometerão toda a texturização subsequente, por melhor que seja o trabalho de pintura. Relacionado a esse, está o erro de não usar a textura de verificação e, por isso, simplesmente não enxergar a distorção até que seja tarde — quando o mapa já foi pintado sobre um mapeamento defeituoso.

Outro equívoco comum é confundir o esticamento causado por má projeção com um problema da textura, e tentar corrigir na pintura o que é, na verdade, um defeito de mapeamento — esforço inútil, pois nenhuma textura conserta um UV esticado. Há ainda a confusão entre o eixo U e o eixo V, ou a inversão de um deles, que faz a textura aparecer rotacionada ou espelhada de modo indevido. E, por fim, o desconhecimento dos modos de endereçamento, que leva o iniciante a estranhar por que uma textura se repete quando não deveria, ou por que não se repete quando ele esperava que repetisse — quando a causa é simplesmente o modo configurado e os valores das coordenadas fora ou dentro do intervalo canônico.

## Resumo

Este capítulo estabeleceu a ponte entre a textura bidimensional e o modelo tridimensional: o sistema de coordenadas UV. Vimos que U e V são coordenadas normalizadas, variando sempre de 0 a 1 independentemente da resolução da textura, e que cada vértice da malha carrega um par dessas coordenadas, interpolado pelo shader para que a imagem plana se assente ponto a ponto sobre a superfície curva. Como uma superfície fechada não se abre sobre um plano sem ser cortada, introduzimos as costuras (*seams*), as ilhas UV resultantes e o layout que as organiza no espaço UV — vocabulário que sustentará toda a Parte II. Examinamos os modos de endereçamento — repetição, repetição espelhada e fixação — que governam o comportamento das coordenadas fora do intervalo canônico e habilitam a repetição de textura. Estudamos as quatro projeções fundamentais — planar, cilíndrica, esférica e cúbica —, cada uma adequada a uma classe de formas e com suas regiões características de distorção, e firmamos o princípio de que objetos reais quase sempre exigem a combinação de projeções, decompondo a forma complexa em regiões tratáveis. Por fim, conhecemos a textura de verificação, instrumento que torna visível a distorção invisível. Com esses fundamentos, estamos prontos para o problema central do próximo capítulo: como decidir, com critério, onde cortar a superfície e como desdobrá-la — o UV unwrapping.

## Exercícios

1. Explique por que as coordenadas UV são normalizadas no intervalo de 0 a 1 em vez de corresponderem diretamente aos pixels da textura. Que vantagem prática essa decisão traz quando se troca uma textura de baixa resolução por uma de alta resolução ao longo da produção?

2. Para cada uma das quatro projeções fundamentais (planar, cilíndrica, esférica e cúbica), descreva a classe de formas que ela trata bem e identifique a região onde ela caracteristicamente produz distorção. Ilustre cada caso com um objeto do mundo real.

3. Análise: você recebe um modelo de um machado, composto por um cabo longo e aproximadamente cilíndrico e uma lâmina larga e aproximadamente plana. Que combinação de projeções você usaria e como dividiria o objeto em ilhas? Justifique cada decisão com base na forma de cada parte.

4. Diagnóstico: ao aplicar uma textura de verificação a um modelo, você observa que os quadrados aparecem perfeitamente regulares em uma região, esticados em retângulos longos em outra e comprimidos em uma terceira. Interprete o que cada uma dessas três aparências revela sobre o mapeamento e proponha como abordar cada problema.

5. Explique a diferença entre os modos de endereçamento de repetição, repetição espelhada e fixação. Para revestir a fachada de um prédio com uma textura de tijolos pequena, qual modo você escolheria, e que cuidado a textura precisaria ter para que a repetição não fosse perceptível?

## Leituras Complementares

### Material de referência do projeto

- *Blender Maps* (material de apoio da disciplina) — apresenta, de forma aplicada, como as coordenadas e os mapas se relacionam dentro de um fluxo de trabalho concreto.

### Fontes complementares externas

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. O Capítulo 6 ("Texturing") trata, com rigor, do mapeamento de texturas, das coordenadas e dos modos de endereçamento aqui apresentados.
- *Scratchapixel — Introduction to Texturing* (scratchapixel.com) — reconstrói, de forma conceitual e progressiva, a relação entre a superfície e a textura e o papel das coordenadas de mapeamento.
- *Blender Manual* (docs.blender.org), seção "UV Editing" — descreve os métodos de projeção (planar, cilíndrica, esférica, cúbica) e o uso de costuras e ilhas em uma ferramenta concreta.
- Documentação da Unreal Engine (dev.epicgames.com) e da Unity (docs.unity3d.com), seções sobre coordenadas de textura e *texture wrapping* — para observar como cada motor expõe os modos de endereçamento e a repetição de texturas discutidos neste capítulo.
