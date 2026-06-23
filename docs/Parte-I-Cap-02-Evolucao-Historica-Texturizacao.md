# Capítulo 2 — Evolução Histórica da Texturização em Jogos Digitais

## Introdução

A história da texturização em jogos é, em grande medida, a história de uma negociação permanente entre desejo e limitação. Os artistas sempre quiseram superfícies ricas, variadas e convincentes; o hardware, em cada época, ofereceu apenas uma fração da memória e do poder de cálculo necessários para tanto. Cada avanço técnico que estudaremos nesta apostila — do mapeamento UV à renderização baseada em física — surgiu como resposta a uma restrição concreta, e não como invenção abstrata. Compreender essa trajetória não é um exercício de nostalgia: é o que permite entender *por que* as técnicas atuais têm a forma que têm, e por que certas decisões que hoje parecem naturais foram, em outras épocas, impossíveis ou impensáveis.

Este capítulo percorre a evolução da texturização desde os primeiros jogos tridimensionais até o paradigma contemporâneo. O foco não está em datas e títulos isolados, mas nas mudanças de raciocínio que cada era impôs aos artistas. Veremos que muitos conceitos apresentados no Capítulo 1 — a separação entre forma e informação, a economia de representar detalhe como imagem em vez de geometria — não são invenções recentes, mas princípios que foram sendo descobertos, refinados e por fim sistematizados ao longo de décadas de tentativa, erro e engenhosidade.

Convém um alerta inicial. A texturização não evoluiu de forma linear e uniforme: avanços em uma frente (como a iluminação) frequentemente exigiram repensar outra (como o conteúdo das texturas de cor). A história, portanto, é menos uma escada e mais um sistema de vasos comunicantes, em que pressionar um ponto desloca todos os demais.

## Desenvolvimento

### A era das limitações severas: texturas como luxo

Nos primeiros jogos com gráficos tridimensionais em renderização em tempo real, a texturização mal existia como a entendemos hoje. As superfícies eram frequentemente preenchidas com cores sólidas ou com sombreamento simples, calculado por interpolação entre os vértices. A memória disponível para armazenar imagens era minúscula, medida em poucos kilobytes, e o custo de buscar um valor de textura para cada pixel era proibitivo para o hardware da época. Aplicar uma imagem a uma superfície tridimensional, processo que hoje tomamos como trivial, representava um avanço técnico considerável.

Quando as texturas finalmente se tornaram viáveis em tempo real, elas eram pequenas, de baixa resolução e fortemente reutilizadas. Uma mesma imagem de poucos texels de lado podia revestir paredes inteiras de um nível, repetida lado a lado. Essa repetição, conhecida como *tiling*, nasceu da necessidade de cobrir grandes áreas com pouquíssima memória, e tornou-se, ela própria, uma técnica fundamental que sobrevive até hoje.

> 💡 **Dica Profissional**
> O *tiling* não é uma técnica "velha" ou ultrapassada — ele continua sendo um dos recursos mais econômicos disponíveis. Compreender sua origem ajuda a usá-lo com consciência, e não como solução de menor esforço.

Em *Doom* (id Software, 1993), as texturas de parede eram fixadas ao espaço de mundo em vez de acompanhar a geometria — o que produzia o célebre *texture swimming* ao girar ou mover segmentos de cenário, artefato decorrente justamente da falta de um sistema de coordenadas UV ponto a ponto que o motor de jogo não era capaz de calcular em tempo real. A limitação gerou um método. Esse padrão — restrição que se converte em técnica duradoura — repetir-se-á em cada etapa da história.

### O nascimento da economia de detalhe: do conteúdo embutido ao mapa de relevo

Na ausência de iluminação dinâmica sofisticada, os artistas das primeiras gerações pintavam a iluminação diretamente nas texturas. Sombras, reflexos e volumes eram desenhados à mão sobre a imagem de cor, criando a ilusão de relevo e profundidade em superfícies completamente planas. Essa técnica, hoje chamada retrospectivamente de iluminação "assada" ou pré-calculada na textura, era extraordinariamente eficiente: todo o custo do realismo estava pago de antemão, embutido na imagem, sem nenhum cálculo em tempo de execução.

> ⚠️ **Atenção**
> Uma textura com iluminação embutida — sombras e realces pintados diretamente na cor — só parece correta sob a luz para a qual foi desenhada. Mover uma fonte de luz ou mudar as condições de iluminação expõe a fraude: as sombras pintadas permanecem fixas enquanto a luz se move. Esse problema historicamente motivou a separação entre cor base e resposta à luz, que é a pedra angular de toda a renderização moderna.

À medida que os jogos passaram a desejar iluminação dinâmica — luzes que se movem, dia e noite, lanternas — tornou-se necessário separar aquilo que é cor própria da superfície daquilo que é resposta à luz. *Half-Life* (Valve, 1998) foi um dos primeiros jogos de grande circulação a introduzir um canal separado de brilho especular, admitindo que certas superfícies — metal, vidro — respondessem de modo diferente da pedra ou da madeira sob a mesma fonte de luz; era uma separação ainda primitiva, mas conceitual e historicamente significativa. Essa separação, que no Capítulo 1 apresentamos como boa prática, foi historicamente uma conquista difícil, e é a raiz conceitual de toda a renderização moderna.

A solução para reintroduzir relevo sem geometria veio com os **mapas de relevo**. A primeira família dessas técnicas, o *bump mapping*, usava uma textura para perturbar, ponto a ponto, a forma como a superfície reagia à luz, simulando reentrâncias e saliências que não existiam na malha. Mais tarde, o **mapa de normais** (*normal map*) refinou enormemente essa ideia: em vez de armazenar uma simples variação de altura, ele guarda, em seus canais de cor, a direção para a qual cada ponto da superfície "aponta". Com essa informação, o cálculo de iluminação trata uma superfície plana como se ela fosse cheia de microdetalhe, fazendo a luz incidir corretamente sobre saliências e reentrâncias inexistentes na geometria. Foi a concretização mais elegante do princípio do Capítulo 1: detalhe como informação, não como forma.

> 📘 **Definição**
> **Mapa de normais** é um canal de textura que armazena, em seus canais de cor, a direção ("normal") para a qual cada ponto da superfície aparenta apontar. O sombreador usa essa informação para calcular a iluminação como se existissem microdetalhes geométricos que fisicamente não estão na malha.

### A ponte entre alto e baixo detalhe: o surgimento do baking

O mapa de normais resolveu *como* representar microdetalhe, mas levantou uma questão prática: *de onde* viria a informação de direção de cada ponto? Desenhá-la à mão é antinatural, pois as cores de um mapa de normais não têm significado visual intuitivo. A resposta foi o **baking**, ou transferência de detalhe.

A ideia do baking é construir duas versões do mesmo objeto: uma de alta densidade de polígonos, esculpida com todo o detalhe desejado, e outra simplificada, leve o bastante para rodar em tempo real. O processo de baking então "fotografa" o detalhe da versão pesada e o projeta sobre a versão leve, registrando-o em texturas — notadamente o mapa de normais. O objeto que chega ao jogo é geometricamente simples, mas carrega, em suas texturas, a memória do detalhe que um dia teve em sua versão esculpida. Essa divisão de trabalho entre um modelo de alta resolução e um de baixa resolução tornou-se um dos pilares do pipeline moderno de produção de *assets*, e será estudada em profundidade na parte da disciplina dedicada ao baking. Historicamente, foi ela que permitiu conciliar o detalhe crescente exigido pelos jogadores com o orçamento limitado de polígonos imposto pelo hardware.

### A fragmentação dos métodos e o problema da inconsistência

À medida que as técnicas se multiplicavam — mapas de cor, de relevo, de brilho, de reflexo —, surgiu um novo problema. Cada estúdio, e às vezes cada artista, desenvolvia sua própria maneira de combinar esses mapas e de definir o que cada valor significava. Um material que parecia perfeito nas mãos de um artista podia parecer completamente errado quando reaproveitado em outra cena, sob outra iluminação, ou ajustado por outro profissional. Faltava um terreno comum, um conjunto de regras compartilhadas que garantisse que um material se comportasse de modo previsível e consistente independentemente de quem o criou e de onde foi usado.

Essa ausência de padronização cobrava um preço alto em produções grandes, com dezenas de artistas trabalhando simultaneamente. A aparência dos objetos variava de forma incoerente, exigindo retrabalho constante e dificultando a manutenção de uma identidade visual uniforme ao longo de um jogo inteiro. A indústria precisava de um paradigma que removesse a adivinhação do processo.

### A consolidação: a renderização baseada em física

A resposta a essa fragmentação foi a adoção, ao longo da década de 2010, da **renderização baseada em física** (do inglês *Physically Based Rendering*, ou PBR). O PBR não é uma técnica única, mas um conjunto de princípios que ancoram a descrição dos materiais em propriedades com correspondência física: o quanto uma superfície é metálica, o quão rugosa ela é, como ela reflete e absorve a luz. Ao basear os materiais em fundamentos físicos, o PBR removeu boa parte da adivinhação que assolava os fluxos anteriores: um material definido segundo essas regras tende a parecer correto sob qualquer iluminação e a se comportar de forma consistente entre artistas e cenas distintas.

> 📘 **Definição**
> **Renderização baseada em física (PBR)** é um conjunto de princípios e modelos matemáticos que descreve os materiais a partir de propriedades com correspondência física — metalicidade, mapa de rugosidade, cor base — em vez de ajustes arbitrários definidos a olho. O objetivo central é garantir que os materiais pareçam corretos sob qualquer iluminação e sejam consistentes entre diferentes artistas e cenas.

A importância histórica do PBR está justamente em ter resolvido, de uma só vez, dois problemas que vinham se acumulando: a inconsistência entre materiais e a dificuldade de produzir realismo previsível. *The Order: 1886* (Ready at Dawn, 2015) e *Star Wars Battlefront* (DICE, 2015) são frequentemente apontados como marcos desse ponto de virada: em ambos os casos as equipes operaram inteiramente sob regras PBR e documentaram publicamente o impacto sobre a coerência visual — superfícies que permaneciam plausíveis ao passar de uma localização exterior ensolarada para um interior com luz artificial sem qualquer retoque manual. O PBR consolidou a separação entre cor própria e resposta à luz — herança das lutas contra a iluminação embutida —, sistematizou o uso de múltiplos mapas com significados bem definidos e ofereceu um vocabulário comum a toda a indústria. É por isso que o PBR ocupa, nesta disciplina, um capítulo próprio: ele é menos um ponto na linha do tempo e mais o paradigma sob o qual praticamente toda a texturização contemporânea opera. A mecânica detalhada de seus mapas e fluxos de trabalho será o objeto central do Capítulo 3 e das partes seguintes.

> **Figura 2.1** — Linha do tempo conceitual da texturização, marcando quatro grandes transições — "cor sólida e sombreamento por vértice", "texturas com iluminação embutida e tiling", "mapas de relevo e baking", "renderização baseada em física" — com uma seta indicando o aumento progressivo de realismo e de complexidade de pipeline.

**Screenshot sugerido:** Montagem comparativa de uma mesma superfície (por exemplo, uma parede de pedra) representada em quatro estilos correspondentes a cada era, da cor chapada ao material PBR completo, evidenciando o salto qualitativo entre as gerações.

## Aplicação em Jogos

A relevância prática desta trajetória aparece toda vez que uma equipe escolhe uma técnica de texturização. Nem todo jogo precisa, ou se beneficia, do paradigma mais recente. Um jogo de estética deliberadamente retrô pode usar texturas pequenas, com tiling visível e iluminação embutida, justamente para evocar a sensação das primeiras gerações. Um jogo mobile, restrito por hardware modesto e por orçamento de memória apertado, pode recorrer a técnicas mais antigas e econômicas por necessidade, não por escolha estética. Conhecer a história é, portanto, conhecer um repertório de soluções, cada uma com seu custo e seu efeito, que continua disponível ao profissional.

Além disso, muitas técnicas antigas não foram abandonadas: foram absorvidas. O tiling, nascido da escassez de memória, continua essencial para revestir grandes superfícies. A iluminação pré-calculada, hoje sob a forma de mapas de luz e oclusão, ainda é usada para elementos estáticos do cenário, por ser mais barata que o cálculo em tempo real. O baking, criado para alimentar mapas de normais, é hoje rotina indispensável. O profissional contemporâneo não escolhe entre o antigo e o novo, mas combina camadas históricas de técnica conforme a necessidade de cada objeto e de cada plataforma.

## Estudo de Caso

### Battlefield 4 e a Migração do Frostbite para PBR — DICE, 2013

> **Figura 2.2** — Diagrama extraído do paper de Lagarde e de Rousiers mostrando as faixas de valores de albedo fisicamente corretos para metais e dielétricos. O gráfico estabelece os limites numéricos dentro dos quais qualquer textura PBR deve operar para produzir resultados plausíveis em qualquer iluminação.

**Screenshot sugerido:** Captura de tela oficial de Battlefield 4, disponível em https://www.ea.com/games/battlefield/battlefield-4/media — press kit oficial da EA/DICE. O diagrama de faixas de albedo está disponível no paper de Lagarde e de Rousiers (SIGGRAPH 2014), acessível em https://seblagarde.wordpress.com/2015/07/14/siggraph-2014-moving-frostbite-to-physically-based-rendering/.

Em 2013, a DICE se deparou com um problema que todo grande estúdio enfrentaria nos anos seguintes: a biblioteca de texturas de *Battlefield 3*, construída com anos de trabalho, estava se tornando um obstáculo. Cada material havia sido pintado para a iluminação de uma cena específica — a cor base de um colete tático carregava realces embutidos que combinavam com a luz de um nível, mas pareciam estranhos em outro. Metais que reluziam corretamente sob o sol de Port Valparaíso ficavam opacos e mortos no interior do porta-aviões. Para cada novo contexto de iluminação, os artistas retocavam as texturas manualmente — um ciclo de retrabalho que tornava cada cena refém dos assets criados para ela.

A resposta da DICE foi documentada por Sébastien Lagarde e Charles de Rousiers no paper *"Moving Frostbite to Physically Based Rendering"*, apresentado na SIGGRAPH 2014 e considerado até hoje o documento de referência sobre a migração de um motor AAA para PBR. O princípio central do paper era deceptivamente simples: em vez de armazenar "como esta superfície parece", as texturas deveriam armazenar "o que esta superfície é". A cor base passou a conter apenas a coloração própria do material, sem sombras ou realces. O mapa de rugosidade passou a descrever a microgeometria da superfície — lisa, áspera, acetinada — independentemente da cena. A metalicidade passou a informar ao sombreador se o material era condutor ou isolante.

O efeito dessa mudança em *Battlefield 4* foi imediato e revelador. Um colete tático de nylon, texturizado segundo as novas regras, aparecia convincente tanto na praia de Hainan quanto nas ruas de Xangai quanto no interior de um submersível — sem um único retoque. Dois artistas diferentes, trabalhando em mapas separados do mesmo jogo, chegavam a materiais visualmente coerentes porque ambos seguiam o mesmo modelo físico. E quando o estúdio precisou refazer um nível com condições de iluminação diferentes, os assets migraram sem ajustes. O paper de Lagarde e de Rousiers também estabeleceu as faixas de valores fisicamente válidos para albedo de diferentes materiais — metal polido entre 70% e 100%, superfícies dielétricas entre 4% e 60% —, transformando em regras numéricas o que antes era intuição artística. Essa normalização é o que a indústria inteira adotou nos anos seguintes, e é a razão pela qual os guias de calibração PBR publicados por Allegorithmic e Epic Games convergem para os mesmos valores: todos partem do trabalho da DICE.

## Boas Práticas

A principal boa prática extraída deste capítulo é compreender que escolher uma técnica de texturização é uma decisão de projeto, condicionada pela plataforma-alvo, pela estética desejada e pelo orçamento de produção. Não existe uma técnica universalmente superior; existe a técnica adequada a um conjunto de restrições. O profissional maduro avalia esses fatores antes de comprometer um pipeline.

> 💡 **Dica Profissional**
> Ao adotar um paradigma como o PBR, vale comprometer-se com suas regras de forma disciplinada para toda a equipe. Os benefícios de consistência só se realizam quando todos os artistas respeitam as mesmas convenções — evitar misturar hábitos de fluxos antigos (como iluminação embutida na cor base) com as exigências do novo paradigma é condição necessária para que o sistema funcione.

É igualmente recomendável reconhecer o valor das técnicas herdadas. Tratar tiling, iluminação pré-calculada e baking como recursos plenamente atuais — e não como relíquias — amplia o repertório de soluções disponíveis e frequentemente resolve problemas de desempenho que o paradigma mais recente, sozinho, não resolveria de forma econômica.

## Erros Comuns

> ❌ **Erro Comum**
> Supor que a técnica mais recente é sempre a melhor escolha, aplicando-a indiscriminadamente mesmo quando a plataforma ou a estética do projeto pediriam outra abordagem. Esse deslumbramento com o estado da arte leva a desperdício de recursos e, por vezes, a resultados visualmente incoerentes com a proposta do jogo.

Outro equívoco é desprezar as técnicas antigas como ultrapassadas, ignorando que muitas delas permanecem em uso justamente por sua eficiência. Quem descarta o tiling ou a iluminação pré-calculada por considerá-los "primitivos" costuma esbarrar mais tarde em problemas de memória e desempenho que essas técnicas resolveriam facilmente.

> ❌ **Erro Comum**
> Adotar um paradigma novo apenas em parte — usar o vocabulário do PBR, por exemplo, mas continuar embutindo iluminação na cor base, como se fazia em eras anteriores — anula precisamente os benefícios de consistência que motivaram a mudança.

Por fim, é comum tratar a história da texturização como mera curiosidade, sem perceber que cada técnica atual carrega a marca do problema que veio resolver; sem essa percepção, o estudante decora procedimentos sem compreender suas razões.

## Resumo

Este capítulo apresentou a texturização como uma trajetória de negociação contínua entre o desejo de realismo e as limitações de hardware. Vimos que os primeiros jogos recorriam a cores sólidas e sombreamento simples, e que as primeiras texturas, pequenas e reutilizadas por meio de tiling, nasceram da escassez de memória. A iluminação embutida nas texturas ofereceu realismo barato à custa de rigidez, e a demanda por iluminação dinâmica forçou a separação entre cor própria e resposta à luz — separação que se tornaria pedra angular das técnicas modernas. Os mapas de relevo, e em especial o mapa de normais, reintroduziram microdetalhe sem geometria, alimentados pelo processo de baking, que transfere detalhe de um modelo de alta densidade para um de baixa. A multiplicação descoordenada de métodos gerou inconsistência, problema resolvido pela consolidação da renderização baseada em física, que ancorou os materiais em propriedades físicas e ofereceu à indústria um vocabulário comum e previsível. Acima de tudo, fixou-se a lição de que técnicas antigas não desaparecem: são absorvidas, e o profissional contemporâneo combina camadas históricas de solução conforme as restrições de cada projeto. O paradigma do PBR, central nessa consolidação, será detalhado no próximo capítulo.

## Exercícios

**1.** Escolha duas técnicas de texturização de eras diferentes apresentadas neste capítulo e explique qual limitação concreta de hardware ou de produção cada uma surgiu para resolver. Em seguida, discuta se essa limitação ainda existe hoje e em que medida.

**2.** A iluminação embutida nas texturas é descrita no capítulo como uma solução eficiente, porém rígida. Construa um argumento defendendo seu uso em um projeto contemporâneo específico e, ao final, apresente as objeções que um colega poderia levantar contra essa escolha.

**3.** Compare conceitualmente o mapa de normais e a iluminação pré-calculada na textura de cor. Ambos buscam criar a ilusão de relevo, mas de maneiras fundamentalmente distintas. Em que situações cada um falha em convencer o observador?

**4.** O capítulo afirma que a maior contribuição do PBR foi a consistência, e não um efeito visual isolado. Avalie criticamente essa afirmação. Você concorda que a previsibilidade entre artistas e cenas é mais importante que o ganho de realismo? Justifique.

**5.** Planejamento: imagine que você precisa definir a abordagem de texturização para dois jogos simultâneos — um título mobile de baixo orçamento e um título para console de alta fidelidade. Para cada um, indique quais técnicas históricas e contemporâneas você combinaria e por quê, relacionando suas escolhas às restrições de cada plataforma.

## Glossário

**Baking:** Processo de transferir informações de detalhe de uma malha de alta resolução (com muitos polígonos) para um canal de textura aplicado a uma malha de baixa resolução. O resultado mais comum é um mapa de normais que "guarda" o microdetalhe da versão pesada.

**Bump mapping:** Técnica que pertuba a resposta à luz de uma superfície ponto a ponto, simulando relevo por meio de uma textura de escala de cinza. Precursor histórico do mapa de normais, com menor precisão de iluminação.

**Iluminação embutida (baked lighting):** Técnica em que sombras, realces e informação de iluminação são pintados diretamente na textura de cor, em vez de serem calculados em tempo de execução. Eficiente, mas rígida: a textura só parece correta sob a iluminação para a qual foi desenhada.

**Mapa de normais:** Canal de textura que armazena, em seus canais RGB, a direção para a qual cada ponto da superfície aparenta apontar. Permite simular microdetalhe geométrico sem alterar a malha.

**Mapa de rugosidade:** Canal de textura que descreve, em valores de 0 a 1, o quanto a superfície é áspera ou lisa em cada ponto, controlando a dispersão dos reflexos especulares.

**Motor de jogo:** Sistema de software responsável por renderizar a cena em tempo real, coordenando geometria, materiais e iluminação para gerar a imagem final.

**Renderização baseada em física (PBR):** Conjunto de princípios que ancora a descrição dos materiais em propriedades com correspondência física, garantindo comportamento consistente sob qualquer iluminação e entre diferentes artistas e cenas.

**Tiling:** Repetição de uma textura em ladrilhos para cobrir superfícies maiores do que a própria textura. Técnica nascida da limitação de memória; permanece em uso por sua eficiência.

## Leituras Complementares

- ***What Is Texture Baking?* (material de apoio da disciplina)** — introduz o processo de transferência de detalhe entre modelos, central na história dos mapas de relevo e indispensável para compreender o pipeline moderno de produção de *assets*.
- **McDERMOTT, Wes. *The PBR Guide* (Allegorithmic/Adobe, ed. 2018)** — contextualiza a ascensão da renderização baseada em física e os problemas de fluxo de trabalho que ela veio resolver; leitura direta para os conceitos do Capítulo 3.
- **BLINN, James F. Simulation of wrinkled surfaces. *ACM SIGGRAPH Computer Graphics*, v. 12, n. 3, p. 286–292, 1978** — artigo fundador do *bump mapping*: a primeira proposta de simular relevo perturbando a forma como a luz incide sobre a superfície sem alterar a geometria. Marco histórico do princípio "detalhe como informação".
- **WILLIAMS, Lance. Pyramidal parametrics. *ACM SIGGRAPH Computer Graphics*, v. 17, n. 3, p. 1–11, 1983** — introduz o *mipmapping*, técnica que organiza versões reduzidas de uma textura para evitar serrilhado quando o objeto se afasta da câmera; solução nascida das limitações de hardware e ainda hoje onipresente.
- **COOK, Robert L.; TORRANCE, Kenneth E. A reflectance model for computer graphics. *ACM SIGGRAPH Computer Graphics*, v. 16, n. 3, 1982** — modelo de reflectância fisicamente embasado que antecipou, por décadas, os fundamentos teóricos do PBR; leitura recomendada para entender de onde vem a "física" da renderização baseada em física.
- **KARIS, Brian. *Real Shading in Unreal Engine 4*. SIGGRAPH 2013** — documento histórico que registra a adoção do fluxo PBR por um dos principais motores de jogo comerciais, marcando a virada da indústria descrita neste capítulo. Disponível em blog.selfshadow.com.
- **AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018** — reúne, em perspectiva histórica e técnica, a evolução das técnicas de texturização e iluminação tratadas aqui.
