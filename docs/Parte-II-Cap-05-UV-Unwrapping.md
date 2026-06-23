# Capítulo 5 — UV Unwrapping

## Introdução

O capítulo anterior estabeleceu o vocabulário do mapeamento — coordenadas UV, costuras, ilhas, layout — e mostrou que as projeções automáticas, embora úteis, raramente bastam para um objeto de forma complexa. Resta enfrentar a tarefa que define boa parte da competência de um artista técnico: decidir, com critério, onde cortar a superfície e como abri-la sobre o plano da textura. Esse processo, chamado **UV unwrapping** ou **desdobramento UV**, é o coração da preparação de *assets* e o assunto deste capítulo.

A metáfora do costureiro, apresentada no Capítulo 4, ganha aqui seu sentido pleno. Desdobrar é o ato de descascar a superfície tridimensional, abrindo-a em pedaços planos da mesma forma como se descasca uma laranja e se tenta achatar a casca sobre a mesa. A casca não se achata sozinha: é preciso cortá-la, e onde se corta determina como ela se abre, quanto se distorce e quão visíveis ficam as emendas. O desdobramento é, portanto, um exercício permanente de equilíbrio entre objetivos que competem entre si — minimizar a distorção, esconder as costuras e manter o layout organizado e eficiente. Não há solução única e perfeita; há decisões mais ou menos acertadas para cada contexto. Aprender a tomar essas decisões com fundamento, e não por tentativa cega, é o propósito deste capítulo.

## Desenvolvimento

### O problema fundamental: por que cortar é inevitável

Existe um motivo matemático profundo para que toda superfície minimamente curva precise ser cortada antes de se abrir sobre um plano: superfícies com curvatura — como uma esfera ou qualquer forma orgânica — não são **desenvolvíveis**, isto é, não podem ser achatadas sem esticar, comprimir ou rasgar. É a mesma razão pela qual nenhuma projeção cartográfica consegue representar a superfície curva da Terra sobre uma folha plana sem distorcer áreas, ângulos ou distâncias. O cartógrafo escolhe qual distorção tolerar; o artista de UV, igualmente, escolhe.

Algumas superfícies, porém, são desenvolvíveis: o cilindro e o cone, por exemplo, podem ser abertos sobre o plano sem distorção alguma, bastando um corte ao longo de seu comprimento — pense no rótulo de uma lata, que é plano antes de ser enrolado. Compreender essa distinção orienta o desdobramento: onde a superfície se aproxima de uma forma desenvolvível, um único corte bem posicionado resolve o problema sem distorção; onde a curvatura é dupla, como numa esfera ou num rosto, nenhuma quantidade de cortes elimina toda a distorção, e o trabalho passa a ser distribuí-la de modo a torná-la imperceptível. Cortar, em suma, é a moeda com que se compra a redução da distorção: mais cortes significam menos esticamento, mas também mais costuras a esconder.

### Os três objetivos em tensão

Todo bom desdobramento persegue, simultaneamente, três objetivos que tendem a se opor. Compreendê-los como um sistema de compromissos é o que distingue o artista que decide do que apenas executa.

O primeiro objetivo é **minimizar a distorção**. As ilhas, depois de abertas, devem preservar tanto quanto possível as proporções e os ângulos da superfície original, para que a textura não apareça esticada ou comprimida quando aplicada. A textura de verificação, apresentada no capítulo anterior, é o instrumento que torna essa distorção visível: quadrados regulares indicam baixa distorção. Em geral, quanto mais se corta, menor a distorção — mas isso colide com o segundo objetivo.

O segundo objetivo é **minimizar e esconder as costuras**. Toda costura é uma descontinuidade no mapeamento e, potencialmente, uma linha visível na superfície final, onde a textura pode não casar perfeitamente de um lado para o outro. Quanto menos costuras, melhor; e as que forem inevitáveis devem ser posicionadas onde o olhar não as encontre. Esse objetivo puxa na direção contrária ao primeiro: menos cortes significam costuras mais discretas, porém mais distorção.

O terceiro objetivo é **aproveitar bem o espaço UV**. As ilhas precisam caber, organizadas, dentro do quadrado de 0 a 1, e o espaço vazio entre elas é espaço de textura desperdiçado — resolução que se paga em memória mas não se usa. Ilhas grandes e bem-arrumadas aproveitam mais resolução; ilhas pequenas e espalhadas a desperdiçam. Esse objetivo, a *organização* e o *empacotamento* do layout, conecta-se diretamente ao próximo capítulo, sobre densidade de texels, e será lá aprofundado.

O ofício do desdobramento consiste em equilibrar esses três objetivos conforme a importância do *asset*. Um herói visto de perto justifica muitos cortes bem escondidos e baixa distorção; um objeto distante de cenário tolera mais distorção em troca de rapidez. Não existe o desdobramento "certo" em abstrato — existe o desdobramento adequado a um propósito.

> Figura: Diagrama em forma de triângulo de compromissos, com os três objetivos — "baixa distorção", "poucas costuras visíveis" e "bom aproveitamento do espaço" — nos vértices, e setas indicando como a busca de um tende a prejudicar os outros.
>
> **Screenshot sugerido:**
> Captura de um mesmo modelo desdobrado de duas maneiras contrastantes — uma com muitos cortes e baixa distorção, outra com poucos cortes e mais distorção — lado a lado com suas texturas de verificação, para evidenciar o compromisso.

### Onde posicionar as costuras

Decidir onde cortar é a decisão mais consequente do desdobramento, e ela se guia por princípios bastante estáveis, independentes de software. O princípio mestre é **esconder as costuras onde não serão vistas**. As regiões naturais para isso são as cavidades, as dobras internas, as áreas ocultas por outros objetos ou pelo próprio corpo, as partes que ficam sob o personagem ou contra a parede, e as bordas onde a geometria muda bruscamente de direção. Numa personagem humanoide, por exemplo, as costuras tradicionalmente correm pela parte interna dos braços e pernas, pelas axilas, pela linha do couro cabeludo e por trás das orelhas — lugares onde o olhar raramente repousa e onde uma eventual descontinuidade da textura passa despercebida.

Um segundo princípio é **cortar onde a forma muda de direção**, isto é, ao longo das arestas marcantes do objeto. Posicionar a costura sobre uma quina viva — a beirada de uma caixa, o canto de um móvel — é vantajoso por dois motivos: ali a textura já tende a se interromper visualmente por causa da iluminação, o que disfarça a emenda, e ali a separação das ilhas acompanha a lógica natural da forma, reduzindo a distorção. Cortar no meio de uma superfície lisa e contínua, ao contrário, é quase sempre um erro, pois expõe a costura justamente onde ela seria mais notada.

Um terceiro princípio é **seguir a lógica da própria peça**. Objetos feitos de partes distintas — uma alça soldada a uma caneca, uma fivela presa a um cinto, painéis rebitados de um veículo — sugerem suas próprias linhas de corte, que coincidem com as junções reais entre as partes. Aproveitar essas junções para posicionar costuras é natural e quase invisível, porque o olho já espera uma descontinuidade ali.

### A questão da simetria

Muitos objetos — personagens, veículos, mobiliário — são simétricos, e essa simetria é uma oportunidade de economia que o desdobramento sabe explorar. Quando duas metades de um objeto são espelhadas, é possível desdobrar apenas uma delas e fazer a outra **compartilhar o mesmo espaço UV**, sobrepondo as ilhas das duas metades. Na prática, isso significa que a textura pintada para o lado direito de um rosto serve igualmente ao lado esquerdo, dobrando a resolução efetiva disponível, pois a mesma área de textura é usada duas vezes sobre o modelo.

Esse ganho tem um custo que o profissional precisa pesar: a textura fica perfeitamente simétrica, e qualquer detalhe assimétrico — uma cicatriz num só lado do rosto, um logotipo, manchas de desgaste distintas — torna-se impossível sem abandonar o compartilhamento naquela região. A decisão de espelhar ou não é, assim, mais uma das tensões do desdobramento: economia de resolução contra liberdade de detalhe. Em produção, é comum uma solução intermediária, em que a maior parte do modelo aproveita a simetria e apenas as regiões que exigem assimetria recebem espaço UV próprio. Voltaremos a essa técnica de sobreposição no Capítulo 6, ao tratar do reaproveitamento de espaço.

### O fluxo de trabalho do desdobramento

Embora os detalhes variem entre ferramentas, o fluxo conceitual do desdobramento é estável e vale a pena ser interiorizado independentemente do software. Ele começa pela **análise da forma**, em que o artista examina o modelo e identifica suas regiões — o que é cilíndrico, o que é plano, o que é orgânico — e antecipa onde as costuras poderão se esconder. Segue-se a **marcação das costuras**, em que se definem as linhas de corte segundo os princípios já discutidos. A partir delas, executa-se o **desdobramento** propriamente dito, em que o algoritmo da ferramenta abre cada ilha sobre o plano, normalmente buscando minimizar a distorção. Vem, então, a **verificação**, com a aplicação da textura de quadriculado para inspecionar a uniformidade e localizar esticamentos residuais. E, por fim, a **organização e o empacotamento** das ilhas dentro do espaço UV, ajustando posições, rotações e tamanhos — etapa que prepara o terreno para a densidade de texels do próximo capítulo. Esse ciclo é frequentemente iterativo: a verificação revela problemas que mandam o artista de volta à marcação das costuras, num refinamento progressivo até que o resultado satisfaça os três objetivos.

> Figura: Sequência ilustrando o fluxo de desdobramento de um objeto simples (por exemplo, uma caneca) — análise das regiões, marcação das costuras destacadas em cor, ilhas abertas no editor UV, verificação com xadrez e empacotamento final.
>
> **Screenshot sugerido:**
> Captura de um editor 3D no momento da marcação de costuras, com as arestas de corte realçadas sobre o modelo, ao lado do resultado já desdobrado no editor UV.

### Desdobramento automático e desdobramento manual

As ferramentas atuais oferecem recursos de desdobramento automático que, a partir da análise da geometria, propõem costuras e abrem as ilhas sem intervenção do artista. Esses recursos evoluíram muito e são valiosos para objetos secundários, para protótipos e como ponto de partida. Sua limitação é que o algoritmo otimiza critérios geométricos, mas não conhece a *intenção* — não sabe quais regiões o jogador verá de perto, onde uma costura seria esteticamente inaceitável, ou que duas partes deveriam compartilhar espaço por serem simétricas. Por isso, para *assets* importantes, o desdobramento automático serve como rascunho a ser refinado, e não como produto final. O julgamento humano sobre onde esconder costuras e como distribuir a importância pelo espaço UV permanece insubstituível justamente porque envolve conhecimento sobre o uso do objeto, e não apenas sobre sua geometria.

## Aplicação em Jogos

Na produção de jogos, o desdobramento é onde se materializa boa parte da estratégia de qualidade e desempenho de um *asset*. A decisão sobre quantos cortes fazer, onde escondê-los e quanto espaço UV dedicar a cada região traduz, em termos concretos, quais partes do modelo o estúdio considera importantes. Um rosto de personagem jogável recebe, tipicamente, generosa fatia do espaço UV e costuras meticulosamente escondidas atrás das orelhas e na linha do cabelo, ao passo que a sola dos sapatos, que ninguém examina, recebe pouco espaço e tolera costuras grosseiras. Essa alocação intencional é uma forma de gastar resolução onde ela rende e poupá-la onde não faria diferença — exatamente o tipo de decisão econômica que distingue o trabalho profissional.

O aproveitamento da simetria é, igualmente, uma prática difundida em jogos, sobretudo em personagens e veículos, porque o ganho de resolução efetiva é considerável e o custo — a simetria da textura — costuma ser aceitável ou contornável. Onde a assimetria importa, recorre-se a soluções como uma camada de detalhe assimétrico aplicada por cima do mapeamento simétrico, ou ainda à reserva de espaço UV próprio apenas para as regiões que precisam ser únicas. Tudo isso revela que o desdobramento, longe de ser uma etapa mecânica, é um espaço de decisões de produção que reverberam na qualidade visual e no orçamento de memória do jogo inteiro.

## Estudo de Caso

### Monster Hunter: World e o Unwrapping de Criaturas Orgânicas Complexas — Capcom, 2018

![Rathalos, dragão alado de Monster Hunter: World, com escamas, asas membranosas e crista em close durante batalha](imagens/cap05-monster-hunter-world-rathalos.jpg)

<!-- Imagem: captura de tela oficial de Monster Hunter: World, Capcom — disponível em https://www.monsterhunter.com/world/us/topics/media/ (presskit oficial) ou via Steam em https://store.steampowered.com/app/582010/Monster_Hunter_World/ -->

Poucas categorias de *asset* testam os limites do UV unwrapping com tanta intensidade quanto as criaturas de *Monster Hunter: World*. Um Rathalos — dragão alado de escalas sobrepostas, asas membranosas com nervuras, crista com espinhos, patas com garras articuladas e uma cauda com lâmina óssea — apresenta ao artista todos os problemas centrais do desdobramento ao mesmo tempo: geometria orgânica contínua sem junções naturais, escamas que precisam de continuidade visual entre regiões mas não podem ser uma ilha única, membranas de asa que se dobram e precisam de costuras onde o mapa de normais não vai se comprometer, e simetria parcial que pode ser aproveitada em algumas regiões mas não em outras, onde macho e fêmea diferem.

O blog de desenvolvimento da Capcom e os vídeos de making-of de *Monster Hunter: World* mostram que a equipe de arte de criaturas abordou o desdobramento como um problema de *anatomia*, não de geometria. As escalas do corpo recebem ilhas organizadas em tiras que acompanham o fluxo do corpo — costelas para a frente, flancos para os lados — de modo que as costuras coincidam com as junções entre grupos de escamas, onde a descontinuidade é visualmente esperada. As asas membranosas são tratadas em posição estendida, com a costura posicionada na aresta de inserção junto ao corpo — exatamente onde a asa entra na escama, região quase nunca vista de perto. A cauda é desdobrada como um tubo aberto na face inferior, que fica contra o chão durante os ataques mais fotogênicos.

O que torna o caso de *Monster Hunter: World* particularmente instrutivo é a questão da simetria parcial — uma decisão que não existe nos exemplos simétricos de personagens humanoides. O corpo do Rathalos macho e o da fêmea são suficientemente semelhantes para que o time de arte compartilhe o mesmo espaço UV, usando o espelho para dobrar a densidade efetiva; mas as diferenças de coloração e de padrão de escamas entre os dois são resolvidas na camada de textura por máscaras de variação, e não na estrutura UV. Essa combinação — UVs espelhadas para densidade, máscaras para variação — é precisamente o tipo de decisão pragmática que os tutoriais de UV raramente ensinam, mas que os artistas de criaturas de estúdios AAA tomam rotineiramente. Identificá-la num produto amplamente conhecido torna o princípio concreto e rastreável.

> **Figura 5.1** — Layout UV do Rathalos de *Monster Hunter: World*, com as principais ilhas identificadas: escamas do tronco em tiras horizontais, asas em posição estendida, crista e espinhos como ilhas independentes, e cauda aberta na face inferior. As ilhas simétricas das duas metades do corpo aparecem sobrepostas, duplicando a densidade efetiva.
>
> **Fonte da imagem:** Capcom — *Monster Hunter: World* (2018). Capturas de tela oficiais disponíveis em: `https://www.monsterhunter.com/world/us/topics/media/` (media kit oficial, acesso público). Análises do UV layout de criaturas por artistas da comunidade disponíveis em: `https://www.artstation.com` (busca: "Monster Hunter World creature UV").

## Boas Práticas

A boa prática central do desdobramento é tratar o posicionamento das costuras como uma decisão deliberada, guiada pela pergunta "quem vai ver esta emenda?". Esconder as costuras em cavidades, dobras, faces ocultas e arestas vivas, e fazê-las acompanhar as junções naturais da peça, resolve de uma só vez os problemas de visibilidade e, muitas vezes, de distorção. Cortar ao longo de mudanças de direção da forma, e nunca no meio de uma superfície lisa e contínua bem à vista, é um princípio que raramente falha.

É boa prática verificar o desdobramento com a textura de quadriculado antes de avançar, e tratar o ciclo como iterativo: marcar, abrir, verificar, ajustar e repetir até que a distorção seja aceitável e as costuras estejam bem escondidas. Recomenda-se aproveitar a simetria sempre que o objeto a tiver e a assimetria de detalhe não for indispensável, pesando conscientemente o ganho de resolução contra a perda de variação. E convém usar o desdobramento automático como ponto de partida para *assets* secundários ou como rascunho para os principais, reservando o refinamento manual para onde a qualidade importa — afinal, o algoritmo otimiza geometria, mas só o artista conhece a intenção.

## Erros Comuns

O erro mais visível é posicionar costuras em lugares expostos — no meio do rosto, na frente de um objeto, ao longo de uma superfície lisa e contínua —, produzindo emendas que saltam aos olhos no resultado final. Quase tão comum é o erro oposto: usar costuras de menos por receio de cortar, o que força a superfície a se abrir com forte distorção, esticando a textura de modo igualmente prejudicial. Ambos decorrem de não compreender o desdobramento como um equilíbrio entre cortar pouco e cortar bem.

Outro equívoco frequente é ignorar a verificação com o xadrez e descobrir a distorção tarde demais, quando o mapa já foi pintado e o retrabalho é caro. Há também o erro de desperdiçar espaço UV, deixando grandes áreas vazias entre ilhas mal-arrumadas — resolução paga e não usada, assunto que o próximo capítulo aprofundará. Frequente, ainda, é aceitar acriticamente o resultado de um desdobramento automático para um *asset* importante, sem perceber que o algoritmo escondeu costuras em lugares ruins ou desperdiçou espaço, porque não conhecia o uso do objeto. E, por fim, um erro de mentalidade que atravessa toda a etapa: encarar o desdobramento como uma formalidade técnica a ser despachada depressa, em vez de reconhecê-lo como o momento em que se decide, de fato, como a textura vai vestir o modelo e onde a qualidade será gasta.

## Resumo

Este capítulo tratou do UV unwrapping, o processo de cortar e abrir a superfície tridimensional sobre o plano da textura. Vimos a razão de fundo pela qual cortar é inevitável: superfícies de curvatura dupla não são desenvolvíveis e não se achatam sem distorção, ao passo que formas como o cilindro e o cone admitem um único corte limpo. Estabelecemos os três objetivos em permanente tensão — minimizar a distorção, esconder as costuras e aproveitar bem o espaço UV — e mostramos que o ofício consiste em equilibrá-los conforme a importância do *asset*. Detalhamos os princípios de posicionamento das costuras: escondê-las onde não serão vistas, cortar nas mudanças de direção da forma e seguir as junções naturais da peça. Tratamos da simetria como oportunidade de dobrar a resolução efetiva por meio da sobreposição de ilhas, ao custo de uma textura simétrica. Percorremos o fluxo conceitual e iterativo do desdobramento — analisar, marcar, abrir, verificar, organizar — e situamos o papel e os limites do desdobramento automático, útil como rascunho, insuficiente como produto final para *assets* importantes, porque otimiza geometria sem conhecer a intenção. Com as ilhas abertas e verificadas, falta resolver com precisão a questão de seus tamanhos relativos e de seu arranjo no espaço UV — a densidade de texels e a organização, tema do próximo capítulo.

## Exercícios

1. Explique, com suas palavras, por que uma esfera não pode ser desdobrada sem distorção, enquanto um cilindro pode. Relacione sua resposta ao conceito de superfície desenvolvível e à analogia com as projeções cartográficas.

2. Descreva os três objetivos em tensão no desdobramento e explique, com um exemplo concreto, como a busca de um deles tende a prejudicar os outros. Em que situação você priorizaria a baixa distorção sobre a discrição das costuras, e em que situação faria o contrário?

3. Planejamento: você precisa desdobrar um personagem humanoide que será o protagonista jogável de um jogo, visto de perto na maior parte do tempo. Indique onde posicionaria as principais costuras e justifique cada escolha em termos de visibilidade e distorção. O personagem tem uma cicatriz no lado esquerdo do rosto — como isso afeta sua decisão sobre aproveitar a simetria?

4. Comparação: contraste o desdobramento automático com o manual. Para que tipos de *asset* cada um é mais adequado, e por que o automático, apesar de avançado, não substitui o julgamento humano em modelos importantes?

5. Diagnóstico: no jogo final, uma linha visível atravessa o meio da bochecha de um personagem, onde a textura não casa de um lado para o outro. Levante a hipótese mais provável sobre a origem do problema no desdobramento e proponha como corrigi-lo, indicando para onde a costura deveria ter sido movida.

## Leituras Complementares

### Material de referência do projeto

- *Blender Maps* e *Blender Normals* (materiais de apoio da disciplina) — situam o desdobramento dentro de um fluxo de trabalho concreto e mostram sua relação com os mapas que serão gerados posteriormente.

### Fontes complementares externas

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. O Capítulo 6 discute o mapeamento de texturas e a parametrização de superfícies, fundamento teórico do desdobramento.
- *Blender Manual* (docs.blender.org), seção "UV Editing" — descreve a marcação de costuras (*marking seams*), o desdobramento e o uso do editor UV em uma ferramenta concreta.
- Documentação do Adobe Substance 3D (helpx.adobe.com/substance-3d.html) e do 3DCoat (3dcoat.com/documentation) — apresentam recursos de desdobramento e ferramentas de costura úteis para comparar abordagens entre softwares.
- POLYCOUNT WIKI, verbete *UV Mapping* (wiki.polycount.com) — conhecimento consolidado da comunidade sobre posicionamento de costuras e boas práticas de desdobramento na indústria.
