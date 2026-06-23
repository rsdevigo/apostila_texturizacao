# Capítulo 2 — Evolução Histórica da Texturização em Jogos Digitais

## Introdução

A história da texturização em jogos é, em grande medida, a história de uma negociação permanente entre desejo e limitação. Os artistas sempre quiseram superfícies ricas, variadas e convincentes; o hardware, em cada época, ofereceu apenas uma fração da memória e do poder de cálculo necessários para tanto. Cada avanço técnico que estudaremos nesta apostila — do mapeamento UV à renderização baseada em física — surgiu como resposta a uma restrição concreta, e não como invenção abstrata. Compreender essa trajetória não é um exercício de nostalgia: é o que permite entender *por que* as técnicas atuais têm a forma que têm, e por que certas decisões que hoje parecem naturais foram, em outras épocas, impossíveis ou impensáveis.

Este capítulo percorre a evolução da texturização desde os primeiros jogos tridimensionais até o paradigma contemporâneo. O foco não está em datas e títulos isolados, mas nas mudanças de raciocínio que cada era impôs aos artistas. Veremos que muitos conceitos apresentados no Capítulo 1 — a separação entre forma e informação, a economia de representar detalhe como imagem em vez de geometria — não são invenções recentes, mas princípios que foram sendo descobertos, refinados e por fim sistematizados ao longo de décadas de tentativa, erro e engenhosidade.

Convém um alerta inicial. A texturização não evoluiu de forma linear e uniforme: avanços em uma frente (como a iluminação) frequentemente exigiram repensar outra (como o conteúdo das texturas de cor). A história, portanto, é menos uma escada e mais um sistema de vasos comunicantes, em que pressionar um ponto desloca todos os demais.

## Desenvolvimento

### A era das limitações severas: texturas como luxo

Nos primeiros jogos com gráficos tridimensionais em tempo real, a texturização mal existia como a entendemos hoje. As superfícies eram frequentemente preenchidas com cores sólidas ou com sombreamento simples, calculado por interpolação entre os vértices. A memória disponível para armazenar imagens era minúscula, medida em poucos kilobytes, e o custo de buscar um valor de textura para cada pixel era proibitivo para o hardware da época. Aplicar uma imagem a uma superfície tridimensional, processo que hoje tomamos como trivial, representava um avanço técnico considerável.

Quando as texturas finalmente se tornaram viáveis em tempo real, elas eram pequenas, de baixa resolução e fortemente reutilizadas. Uma mesma imagem de poucos texels de lado podia revestir paredes inteiras de um nível, repetida lado a lado. Essa repetição, conhecida como *tiling*, nasceu da necessidade de cobrir grandes áreas com pouquíssima memória, e tornou-se, ela própria, uma técnica fundamental que sobrevive até hoje. A limitação gerou um método. Esse padrão — restrição que se converte em técnica duradoura — repetir-se-á em cada etapa da história.

### O nascimento da economia de detalhe: do conteúdo embutido ao mapa de relevo

Na ausência de iluminação dinâmica sofisticada, os artistas das primeiras gerações pintavam a iluminação diretamente nas texturas. Sombras, reflexos e volumes eram desenhados à mão sobre a imagem de cor, criando a ilusão de relevo e profundidade em superfícies completamente planas. Essa técnica, hoje chamada retrospectivamente de iluminação "assada" ou pré-calculada na textura, era extraordinariamente eficiente: todo o custo do realismo estava pago de antemão, embutido na imagem, sem nenhum cálculo em tempo de execução.

O preço dessa eficiência, contudo, era a rigidez. Uma textura com sombras pintadas só parece correta sob a luz para a qual foi desenhada. Mover uma fonte de luz, ou permitir que o jogador carregue uma tocha, exporia a fraude: as sombras pintadas permaneceriam fixas enquanto a luz se moveria. À medida que os jogos passaram a desejar iluminação dinâmica — luzes que se movem, dia e noite, lanternas — tornou-se necessário separar aquilo que é cor própria da superfície daquilo que é resposta à luz. Essa separação, que no Capítulo 1 apresentamos como boa prática, foi historicamente uma conquista difícil, e é a raiz conceitual de toda a renderização moderna.

A solução para reintroduzir relevo sem geometria veio com os **mapas de relevo**. A primeira família dessas técnicas, o *bump mapping*, usava uma textura para perturbar, ponto a ponto, a forma como a superfície reagia à luz, simulando reentrâncias e saliências que não existiam na malha. Mais tarde, o **mapa de normais** refinou enormemente essa ideia: em vez de armazenar uma simples variação de altura, ele guarda, em seus canais de cor, a direção para a qual cada ponto da superfície "aponta". Com essa informação, o cálculo de iluminação trata uma superfície plana como se ela fosse cheia de microdetalhe, fazendo a luz incidir corretamente sobre saliências e reentrâncias inexistentes na geometria. Foi a concretização mais elegante do princípio do Capítulo 1: detalhe como informação, não como forma.

### A ponte entre alto e baixo detalhe: o surgimento do baking

O mapa de normais resolveu *como* representar microdetalhe, mas levantou uma questão prática: *de onde* viria a informação de direção de cada ponto? Desenhá-la à mão é antinatural, pois as cores de um mapa de normais não têm significado visual intuitivo. A resposta foi o **baking**, ou transferência de detalhe.

A ideia do baking é construir duas versões do mesmo objeto: uma de alta densidade de polígonos, esculpida com todo o detalhe desejado, e outra simplificada, leve o bastante para rodar em tempo real. O processo de baking então "fotografa" o detalhe da versão pesada e o projeta sobre a versão leve, registrando-o em texturas — notadamente o mapa de normais. O objeto que chega ao jogo é geometricamente simples, mas carrega, em suas texturas, a memória do detalhe que um dia teve em sua versão esculpida. Essa divisão de trabalho entre um modelo de alta resolução e um de baixa resolução tornou-se um dos pilares do pipeline moderno de produção de *assets*, e será estudada em profundidade na parte da disciplina dedicada ao baking. Historicamente, foi ela que permitiu conciliar o detalhe crescente exigido pelos jogadores com o orçamento limitado de polígonos imposto pelo hardware.

### A fragmentação dos métodos e o problema da inconsistência

À medida que as técnicas se multiplicavam — mapas de cor, de relevo, de brilho, de reflexo —, surgiu um novo problema. Cada estúdio, e às vezes cada artista, desenvolvia sua própria maneira de combinar esses mapas e de definir o que cada valor significava. Um material que parecia perfeito nas mãos de um artista podia parecer completamente errado quando reaproveitado em outra cena, sob outra iluminação, ou ajustado por outro profissional. Faltava um terreno comum, um conjunto de regras compartilhadas que garantisse que um material se comportasse de modo previsível e consistente independentemente de quem o criou e de onde foi usado.

Essa ausência de padronização cobrava um preço alto em produções grandes, com dezenas de artistas trabalhando simultaneamente. A aparência dos objetos variava de forma incoerente, exigindo retrabalho constante e dificultando a manutenção de uma identidade visual uniforme ao longo de um jogo inteiro. A indústria precisava de um paradigma que removesse a adivinhação do processo.

### A consolidação: a renderização baseada em física

A resposta a essa fragmentação foi a adoção, ao longo da década de 2010, da **renderização baseada em física**, ou PBR (*Physically Based Rendering*). O PBR não é uma técnica única, mas um conjunto de princípios que ancoram a descrição dos materiais em propriedades com correspondência física: o quanto uma superfície é metálica, o quão rugosa ela é, como ela reflete e absorve a luz. Ao basear os materiais em fundamentos físicos, o PBR removeu boa parte da adivinhação que assolava os fluxos anteriores: um material definido segundo essas regras tende a parecer correto sob qualquer iluminação e a se comportar de forma consistente entre artistas e cenas distintas.

A importância histórica do PBR está justamente em ter resolvido, de uma só vez, dois problemas que vinham se acumulando: a inconsistência entre materiais e a dificuldade de produzir realismo previsível. Ele consolidou a separação entre cor própria e resposta à luz — herança das lutas contra a iluminação embutida —, sistematizou o uso de múltiplos mapas com significados bem definidos e ofereceu um vocabulário comum a toda a indústria. É por isso que o PBR ocupa, nesta disciplina, um capítulo próprio: ele é menos um ponto na linha do tempo e mais o paradigma sob o qual praticamente toda a texturização contemporânea opera. A mecânica detalhada de seus mapas e fluxos de trabalho será o objeto central do Capítulo 3 e das partes seguintes.

> Figura: Linha do tempo conceitual da texturização, marcando quatro grandes transições — "cor sólida e sombreamento por vértice", "texturas com iluminação embutida e tiling", "mapas de relevo e baking", "renderização baseada em física" — com uma seta indicando o aumento progressivo de realismo e de complexidade de pipeline.
>
> **Screenshot sugerido:**
> Montagem comparativa de uma mesma superfície (por exemplo, uma parede de pedra) representada em quatro estilos correspondentes a cada era, da cor chapada ao material PBR completo, evidenciando o salto qualitativo entre as gerações.

## Aplicação em Jogos

A relevância prática desta trajetória aparece toda vez que uma equipe escolhe uma técnica de texturização. Nem todo jogo precisa, ou se beneficia, do paradigma mais recente. Um jogo de estética deliberadamente retrô pode usar texturas pequenas, com tiling visível e iluminação embutida, justamente para evocar a sensação das primeiras gerações. Um jogo mobile, restrito por hardware modesto e por orçamento de memória apertado, pode recorrer a técnicas mais antigas e econômicas por necessidade, não por escolha estética. Conhecer a história é, portanto, conhecer um repertório de soluções, cada uma com seu custo e seu efeito, que continua disponível ao profissional.

Além disso, muitas técnicas antigas não foram abandonadas: foram absorvidas. O tiling, nascido da escassez de memória, continua essencial para revestir grandes superfícies. A iluminação pré-calculada, hoje sob a forma de mapas de luz e oclusão, ainda é usada para elementos estáticos do cenário, por ser mais barata que o cálculo em tempo real. O baking, criado para alimentar mapas de normais, é hoje rotina indispensável. O profissional contemporâneo não escolhe entre o antigo e o novo, mas combina camadas históricas de técnica conforme a necessidade de cada objeto e de cada plataforma.

## Estudo de Caso

Um caso instrutivo é o da transição vivida por estúdios que, em meados da década de 2010, migraram de fluxos de trabalho próprios para o paradigma PBR. Antes da migração, era comum que uma equipe mantivesse bibliotecas de texturas cuja aparência dependia fortemente da iluminação específica para a qual haviam sido criadas. Reaproveitar um material de cena diurna em uma cena noturna produzia resultados estranhos: metais que não pareciam metálicos, superfícies foscas que reluziam indevidamente. Cada reaproveitamento exigia ajustes manuais, e a ausência de regras comuns fazia com que dois artistas, partindo do mesmo objeto, chegassem a materiais visivelmente diferentes.

Ao adotar o PBR, esses estúdios reorganizaram suas bibliotecas em torno de propriedades físicas. A cor base passou a conter apenas a coloração própria do material, livre de sombras e reflexos. A rugosidade e a metalicidade passaram a descrever o comportamento da superfície de modo independente da cena. O efeito imediato foi a previsibilidade: um material de madeira parecia madeira tanto ao meio-dia quanto sob o luar, e dois artistas diferentes, seguindo as mesmas regras, produziam resultados coerentes entre si. O ganho não foi apenas estético; foi, sobretudo, de produtividade e de consistência em escala industrial. Esse caso ilustra por que a padronização, mais do que qualquer efeito visual isolado, foi a verdadeira revolução do período.

> Figura: Esquema mostrando um mesmo material aplicado a um objeto sob três condições de iluminação distintas — diurna, noturna e interior artificial — antes e depois da adoção do PBR, evidenciando a inconsistência anterior e a coerência posterior.
>
> **Screenshot sugerido:**
> Três renderizações do mesmo objeto PBR sob diferentes mapas de iluminação ambiental, demonstrando que a aparência se mantém plausível em todas as condições.

## Boas Práticas

A principal boa prática extraída deste capítulo é compreender que escolher uma técnica de texturização é uma decisão de projeto, condicionada pela plataforma-alvo, pela estética desejada e pelo orçamento de produção. Não existe uma técnica universalmente superior; existe a técnica adequada a um conjunto de restrições. O profissional maduro avalia esses fatores antes de comprometer um pipeline.

É igualmente recomendável reconhecer o valor das técnicas herdadas. Tratar tiling, iluminação pré-calculada e baking como recursos plenamente atuais — e não como relíquias — amplia o repertório de soluções disponíveis e frequentemente resolve problemas de desempenho que o paradigma mais recente, sozinho, não resolveria de forma econômica. Por fim, ao adotar um paradigma como o PBR, vale comprometer-se com suas regras de forma disciplinada: os benefícios de consistência só se realizam quando toda a equipe respeita as mesmas convenções, evitando misturar, sem critério, hábitos de fluxos antigos com as exigências do novo.

## Erros Comuns

Um erro frequente é supor que a técnica mais recente é sempre a melhor escolha, aplicando-a indiscriminadamente mesmo quando a plataforma ou a estética do projeto pediriam outra abordagem. Esse deslumbramento com o estado da arte leva a desperdício de recursos e, por vezes, a resultados visualmente incoerentes com a proposta do jogo.

Outro equívoco é desprezar as técnicas antigas como ultrapassadas, ignorando que muitas delas permanecem em uso justamente por sua eficiência. Quem descarta o tiling ou a iluminação pré-calculada por considerá-los "primitivos" costuma esbarrar mais tarde em problemas de memória e desempenho que essas técnicas resolveriam facilmente. Há ainda o erro de adotar um paradigma novo apenas em parte — usar o vocabulário do PBR, por exemplo, mas continuar embutindo iluminação na cor base, como se fazia em eras anteriores —, anulando precisamente os benefícios de consistência que motivaram a mudança. Por fim, é comum tratar a história da texturização como mera curiosidade, sem perceber que cada técnica atual carrega a marca do problema que veio resolver; sem essa percepção, o estudante decora procedimentos sem compreender suas razões.

## Resumo

Este capítulo apresentou a texturização como uma trajetória de negociação contínua entre o desejo de realismo e as limitações de hardware. Vimos que os primeiros jogos recorriam a cores sólidas e sombreamento simples, e que as primeiras texturas, pequenas e reutilizadas por meio de tiling, nasceram da escassez de memória. A iluminação embutida nas texturas ofereceu realismo barato à custa de rigidez, e a demanda por iluminação dinâmica forçou a separação entre cor própria e resposta à luz — separação que se tornaria pedra angular das técnicas modernas. Os mapas de relevo, e em especial o mapa de normais, reintroduziram microdetalhe sem geometria, alimentados pelo processo de baking, que transfere detalhe de um modelo de alta densidade para um de baixa. A multiplicação descoordenada de métodos gerou inconsistência, problema resolvido pela consolidação da renderização baseada em física, que ancorou os materiais em propriedades físicas e ofereceu à indústria um vocabulário comum e previsível. Acima de tudo, fixou-se a lição de que técnicas antigas não desaparecem: são absorvidas, e o profissional contemporâneo combina camadas históricas de solução conforme as restrições de cada projeto. O paradigma do PBR, central nessa consolidação, será detalhado no próximo capítulo.

## Exercícios

1. Escolha duas técnicas de texturização de eras diferentes apresentadas neste capítulo e explique qual limitação concreta de hardware ou de produção cada uma surgiu para resolver. Em seguida, discuta se essa limitação ainda existe hoje e em que medida.

2. A iluminação embutida nas texturas é descrita no capítulo como uma solução eficiente, porém rígida. Construa um argumento defendendo seu uso em um projeto contemporâneo específico e, ao final, apresente as objeções que um colega poderia levantar contra essa escolha.

3. Compare conceitualmente o mapa de normais e a iluminação pré-calculada na textura de cor. Ambos buscam criar a ilusão de relevo, mas de maneiras fundamentalmente distintas. Em que situações cada um falha em convencer o observador?

4. O capítulo afirma que a maior contribuição do PBR foi a consistência, e não um efeito visual isolado. Avalie criticamente essa afirmação. Você concorda que a previsibilidade entre artistas e cenas é mais importante que o ganho de realismo? Justifique.

5. Planejamento: imagine que você precisa definir a abordagem de texturização para dois jogos simultâneos — um título mobile de baixo orçamento e um título para console de alta fidelidade. Para cada um, indique quais técnicas históricas e contemporâneas você combinaria e por quê, relacionando suas escolhas às restrições de cada plataforma.

## Leituras Complementares

### Material de referência do projeto

- *What Is Texture Baking?* (material de apoio da disciplina) — introduz o processo de transferência de detalhe entre modelos, central na história dos mapas de relevo.
- McDERMOTT, Wes. *The PBR Guide* (Allegorithmic/Adobe, ed. 2018) — contextualiza a ascensão da renderização baseada em física e os problemas de fluxo de trabalho que ela veio resolver.

### Fontes complementares externas

- BLINN, James F. Simulation of wrinkled surfaces. *ACM SIGGRAPH Computer Graphics*, v. 12, n. 3, p. 286–292, 1978. Artigo fundador do *bump mapping*: a primeira proposta de simular relevo perturbando a forma como a luz incide sobre a superfície, sem alterar a geometria. Marco histórico do princípio "detalhe como informação".
- WILLIAMS, Lance. Pyramidal parametrics. *ACM SIGGRAPH Computer Graphics*, v. 17, n. 3, p. 1–11, 1983. Introduz o *mipmapping*, técnica que organiza versões reduzidas de uma textura para evitar serrilhado e ruído quando o objeto se afasta da câmera — solução nascida das limitações de hardware e ainda hoje onipresente.
- COOK, Robert L.; TORRANCE, Kenneth E. A reflectance model for computer graphics. *ACM SIGGRAPH Computer Graphics*, v. 16, n. 3, 1982. Modelo de reflectância fisicamente embasado que antecipou, por décadas, os fundamentos teóricos do PBR; leitura recomendada para entender de onde vem a "física" da renderização baseada em física.
- KARIS, Brian. *Real Shading in Unreal Engine 4*. SIGGRAPH 2013, curso "Physically Based Shading in Theory and Practice". Documento histórico que registra a adoção do fluxo PBR por um dos principais motores comerciais, marcando a virada da indústria descrita neste capítulo. Disponível em blog.selfshadow.com.
- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Reúne, em perspectiva histórica e técnica, a evolução das técnicas de texturização e iluminação tratadas aqui.
- *The Book of Shaders*, de Patricio Gonzalez Vivo e Jen Lowe (thebookofshaders.com) — para compreender, em nível mais fundamental, como o cálculo por pixel tornou-se viável e barateou as técnicas que sustentam as eras mais recentes.
