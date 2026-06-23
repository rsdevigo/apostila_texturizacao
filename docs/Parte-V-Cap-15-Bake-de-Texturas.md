# Capítulo 15 — Bake de Texturas

## Introdução

As quatro partes anteriores construíram, peça por peça, a competência de produzir materiais convincentes: a Parte I situou o que é uma textura e como o motor de jogo a interpreta; a Parte II ensinou a mapear a superfície de um modelo no espaço plano das UVs; a Parte III estabeleceu os fundamentos físicos do PBR e os mapas que compõem um material; e a Parte IV reuniu os métodos de produção desse conteúdo. Em todas elas, porém, uma operação foi invocada repetidamente sem ser desenvolvida por inteiro. Quando os Capítulos 13 e 14 deixaram a geometria guiar a distribuição de efeitos, e quando, ainda na Parte III, falamos de mapas que descrevem a forma do objeto, sempre apontamos para a mesma fonte: o *baking* — e o capítulo sobre máscaras, *stencils* e *decals* (Capítulo 17) dependerá inteiramente dos mapas que ele gera. Esta parte da apostila trata, enfim, daquilo que torna um material não apenas correto, mas **eficiente** — capaz de rodar dentro do orçamento de uma plataforma de jogo —, e nenhum tema é mais central a essa eficiência do que o assar de texturas, ponto de partida obrigatório de quase todo *asset* moderno.

> 📘 **Definição**
> **Baking** (ou assar de texturas) é o processo de transferir informação de um modelo para outro, gravando numa imagem 2D — alinhada às UVs de uma malha simples — detalhes que de outro modo exigiriam geometria custosa. O termo consolida, num único arquivo de textura, o resultado de cálculos que seriam inviáveis em tempo real.

O *baking*, ou assar de texturas, é o processo de **transferir informação de um modelo para outro**, gravando numa imagem 2D — alinhada às UVs de um modelo simples — detalhes que de outro modo exigiriam geometria custosa. É a operação que permite que um personagem com milhões de polígonos esculpidos seja substituído, em jogo, por uma versão de poucos milhares que *parece* conservar todo aquele detalhe. Sem o *baking*, o realismo da Parte III seria incompatível com o desempenho em tempo real: ou se teria a forma rica e o jogo travaria, ou se teria a fluidez e a aparência pobre. O *baking* desfaz esse dilema convertendo geometria em textura, e é por isso que abre a parte dedicada a transformar materiais em *assets* eficientes. Este capítulo apresenta o conceito geral do *baking*, seu mecanismo — a projeção de um modelo sobre outro —, os ingredientes que o tornam possível e os problemas típicos de quem aprende a executá-lo; os capítulos seguintes detalharão os mapas específicos que dele resultam, a começar pelo mais importante de todos, o mapa de normais.

## Desenvolvimento

### O que é assar uma textura

Assar uma textura é **capturar uma propriedade calculada sobre uma malha e gravá-la numa imagem**, de modo que essa propriedade, depois, possa ser apenas lida em vez de recalculada. O termo "assar" (*bake*) vem da ideia de fixar, de tornar permanente algo que antes era dinâmico ou custoso: assim como se assa um bolo para que a massa líquida se torne sólida e estável, assa-se uma textura para que um cálculo pesado se cristalize numa imagem leve. A propriedade capturada pode ser de muitas naturezas — a orientação da superfície, o quanto cada ponto está escondido da luz, a curvatura local, a cor de um material complexo, a iluminação de uma cena inteira —, mas o princípio é sempre o mesmo: trocar **cálculo em tempo de execução por leitura de uma textura pré-computada**. Essa troca é a essência da otimização em gráficos de tempo real, e o *baking* é sua forma mais difundida.

Há, na prática, duas grandes famílias de *baking*, e convém distingui-las desde já para não confundir o estudante. A primeira, e o foco desta parte, é o ***baking* de transferência entre malhas**: capturar detalhes de um modelo de alta densidade (*high-poly*) e gravá-los em mapas alinhados às UVs de um modelo de baixa densidade (*low-poly*). É o que gera os mapas de normais, curvatura, oclusão e demais mapas derivados da geometria que sustentam toda a texturização moderna. A segunda família é o ***baking* de iluminação** — assar a luz de uma cena em *lightmaps*, fixando sombras e iluminação indireta numa textura para que não precisem ser calculadas a cada quadro. Embora ambas compartilhem o nome e o princípio da pré-computação, pertencem a momentos diferentes do *pipeline*: a transferência entre malhas é tarefa do texturizador, na autoria do *asset*; o *baking* de iluminação é tarefa do artista de iluminação, na montagem do nível. Esta parte trata sobretudo da primeira, mencionando a segunda apenas quando pertinente.

> ⚠️ **Atenção**
> Confundir as duas famílias de *baking* é um equívoco frequente. O *baking* de transferência entre malhas pertence ao fluxo de autoria do *asset* e é responsabilidade do texturizador; o *baking* de iluminação (*lightmaps*) pertence à montagem do nível e é responsabilidade do artista de iluminação. Aplicar a lógica de um ao outro produz resultados incorretos.

### O dilema que o baking resolve

Para entender por que o *baking* de transferência é tão central, é preciso retomar o dilema fundamental da renderização em tempo real, já anunciado na Parte I. A riqueza visual de uma superfície depende de detalhe geométrico — as reentrâncias de um tijolo, os poros de uma pele, os rebites e amassados de uma placa de metal —, mas cada polígono custa processamento, e o orçamento de polígonos de uma cena de jogo é finito e disputado por dezenas ou centenas de objetos simultâneos. Um modelo esculpido em ferramentas de *sculpting* pode ter dezenas de milhões de polígonos para representar todo esse detalhe; colocá-lo diretamente num jogo, multiplicado por todos os objetos da cena, é impossível em tempo real. O artista, portanto, precisa de duas versões do mesmo objeto: uma **rica em detalhe**, para servir de referência, e uma **leve**, para de fato rodar.

O *baking* é a ponte entre essas duas versões. Cria-se o modelo de alta densidade, com todo o detalhe; cria-se, por **retopologia**, o modelo de baixa densidade, com a silhueta correta e poucos polígonos, devidamente desdobrado em UVs (Parte II); e então se *assa* o detalhe do primeiro sobre o segundo. O resultado é um conjunto de texturas que, aplicadas à malha leve, fazem-na reagir à luz como se tivesse o detalhe da malha pesada — sem ter os polígonos. O olho, enganado pela maneira como a superfície responde à iluminação, percebe relevo onde há apenas uma imagem. É esta a operação que viabiliza, simultaneamente, o realismo e o desempenho, e por isso o *baking* é descrito como o ato de **converter geometria em textura**: o detalhe que era forma passa a ser informação gravada numa imagem, infinitamente mais barata de processar.

> **Figura 15.1** — O mesmo objeto em três representações lado a lado: a malha *high-poly* esculpida, com milhões de polígonos e todo o detalhe; a malha *low-poly* retopologizada, com a silhueta correta mas superfície lisa; e a *low-poly* com os mapas assados aplicados, exibindo, sob a luz, o detalhe da *high-poly* sem possuir sua geometria.

**Screenshot sugerido:** Captura de uma ferramenta de *baking* mostrando, em janelas adjacentes, a *high-poly*, a *low-poly* nua e a *low-poly* texturizada com os mapas assados, evidenciando que o detalhe foi transferido para a imagem.

### O mecanismo: projeção por raios

O coração técnico do *baking* de transferência é uma operação geométrica simples de enunciar: para cada ponto da superfície da malha *low-poly* — cada texel, cada pixel da textura a ser gerada —, o programa **lança um raio** que parte daquele ponto na direção da normal da superfície e procura o ponto correspondente na malha *high-poly*. Encontrado esse ponto, a propriedade desejada é medida ali (a orientação da superfície *high-poly*, sua curvatura, sua oclusão) e gravada no texel de origem. Repetida para todos os texels, a varredura preenche a textura inteira com a informação colhida da malha rica. É assim que o detalhe "salta" de uma malha para a outra: ponto a ponto, por amostragem dirigida pela normal, mediada pelas UVs da *low-poly*, que dizem a cada ponto da superfície onde ele mora na imagem.

> 💡 **Dica Profissional**
> Compreender que o *baking* é, no fundo, uma chuva de raios entre duas malhas é o que explica todos os problemas práticos da operação. Cada erro de *baking* — o buraco no mapa, o vazamento entre peças, a costura nas bordas das ilhas UV — tem sua causa nesse mecanismo de raios. Quem entende o mecanismo sabe diagnosticar e corrigir os problemas.

Compreender esse mecanismo de raios não é um preciosismo: é o que explica todos os problemas práticos do *baking*. Como a captura depende de raios que vão da *low* à *high* e precisam acertar a superfície certa, tudo o que perturbe esse encontro produz erro. Se o raio é curto demais e não alcança a *high-poly*, fica um buraco; se é longo demais e atravessa para o outro lado do objeto, captura o detalhe errado; se duas partes do modelo estão próximas, o raio de uma pode acertar a outra, "vazando" detalhe de um lugar para outro. Os ingredientes e cuidados do *baking* que veremos a seguir existem todos para governar esse lançamento de raios — para garantir que cada raio percorra a distância certa e acerte a superfície pretendida.

### A cage: governar o alcance dos raios

> 📘 **Definição**
> **Cage** (gaiola) é uma cópia da malha *low-poly* inflada para fora ao longo das normais, de modo a envolver completamente a malha *high-poly*. Sua função é definir de onde partem e até onde vão os raios da projeção durante o *baking*, garantindo que cada raio percorra a distância certa e acerte a superfície pretendida.

O primeiro instrumento de controle do *baking* é a ***cage*** — literalmente, a "gaiola". A *cage* é uma cópia da malha *low-poly* **inflada**, expandida para fora ao longo das normais, de modo a **envolver** completamente a malha *high-poly*. Sua função é definir de onde partem e até onde vão os raios da projeção: em vez de partirem da superfície exata da *low-poly*, os raios partem da *cage* externa e viajam para dentro, varrendo o espaço entre a *cage* e a *low-poly* à procura da *high-poly*. Ao envelopar a *high-poly* inteira, a *cage* garante que nenhum detalhe saliente fique de fora do alcance dos raios e que a direção de cada raio seja suave e coerente, evitando as descontinuidades que surgiriam se cada face lançasse seu raio isoladamente.

O ajuste da *cage* é uma das competências práticas mais importantes do *baking*, e ilustra bem o equilíbrio que a operação exige. Se a *cage* for **estreita demais** e não envolver alguma protuberância da *high-poly*, os raios não alcançarão aquela parte e o mapa apresentará falhas justamente onde há mais detalhe. Se for **larga demais**, os raios percorrerão distâncias longas e correrão o risco de capturar detalhe de partes vizinhas indesejadas, produzindo o "vazamento" mencionado. A *cage* ideal é a menor que ainda envolve toda a *high-poly* — ajustada de perto à silhueta, justa o suficiente para não vazar, folgada o suficiente para não deixar nada de fora. Muitas ferramentas geram uma *cage* automaticamente a partir de uma distância de projeção uniforme, e isso basta para objetos simples; objetos com partes próximas e relevo acentuado exigem o ajuste manual da *cage*, parte por parte, e dominá-lo distingue o texturizador iniciante do experiente.

> **Figura 15.2** — Corte transversal de um objeto mostrando a malha *low-poly* (linha interna), a malha *high-poly* (contorno detalhado) e a *cage* (linha externa inflada que envolve ambas), com setas indicando os raios partindo da *cage* para dentro e acertando a *high-poly* — e, ao lado, dois casos de erro: uma *cage* estreita que deixa uma saliência de fora e uma larga que faz o raio acertar a parte vizinha errada.

**Screenshot sugerido:** Captura de um *baker* exibindo a *cage* sobreposta ao modelo, com controle de distância de projeção, mostrando o envelopamento correto da *high-poly*.

### Ingredientes do baking: nomenclatura, exploded bake e antialiasing

Além da *cage*, alguns cuidados práticos determinam a qualidade do resultado. O primeiro é a **correspondência entre as malhas**: o *baker* precisa saber qual parte da *high-poly* corresponde a qual parte da *low-poly*, e há duas estratégias. Na correspondência por **nomenclatura** (*name matching*), cada par de malhas recebe sufixos convencionados — tipicamente `_low` e `_high` — para que o programa só projete cada *high* sobre sua *low* homônima, e nunca sobre outra. Esse simples acordo de nomes resolve a maior fonte de vazamento em objetos com muitas peças: sem ele, o raio de uma fivela poderia capturar a textura do cinto vizinho; com ele, cada peça só "enxerga" sua própria contraparte. A Naughty Dog documentou em posts técnicos de *Uncharted 4: A Thief's End* (2016) que a equipe integrou a convenção de nomenclatura ao sistema de controle de versões dos *assets*, de modo que o *baker* de produção nunca misturasse pares de malhas por descuido de nome — transformando um cuidado artesanal numa regra verificável pelo próprio pipeline.

> 💡 **Dica Profissional**
> A convenção de nomenclatura `_low` e `_high` é simples e poderosa. Adotá-la desde o início do projeto, e fazê-la parte do padrão de entrega de cada artista, evita toda uma classe de erros de vazamento sem nenhum custo de execução.

A segunda estratégia, complementar, é o ***exploded bake*** — assar com as peças **afastadas** umas das outras. Separam-se espacialmente todas as partes do objeto, tanto na *high* quanto na *low*, de modo que nenhuma fique perto de outra; assa-se nesse estado explodido; e, depois, reúnem-se as peças. Como durante a captura nenhuma peça tem vizinha próxima, é fisicamente impossível um raio vazar de uma para outra. O *exploded bake* é a solução definitiva para o vazamento em objetos complexos — armas, veículos, personagens com muitos acessórios — e é prática padrão na produção profissional. Por fim, há o **antialiasing** do *baking*: como qualquer imagem, os mapas assados sofrem de serrilhado nas bordas do detalhe, e os *bakers* oferecem amostragem múltipla (*supersampling*) para suavizá-las, assando em resolução maior e reduzindo depois. O custo é tempo de processamento; o ganho é a limpeza das bordas, importante sobretudo no mapa de normais que o próximo capítulo detalhará.

### Padding e o problema das bordas das UVs

> 📘 **Definição**
> **Padding** (ou margem de sangria) é a extensão da cor da borda de cada ilha UV alguns pixels para fora, criando uma faixa de transição que impede que os *mipmaps* e a interpolação do hardware "puxem" o vazio adjacente para dentro da superfície, evitando costuras visíveis nas bordas das ilhas.

Um último cuidado merece desenvolvimento próprio porque conecta o *baking* diretamente à organização de UVs da Parte II: o ***padding***, ou margem de sangria. Como vimos, o desdobramento UV divide a superfície em ilhas, e entre elas há espaço vazio na textura. O *baking* preenche cada ilha com a informação assada, mas, exatamente na borda da ilha, a interpolação do hardware e, sobretudo, os *mipmaps* (que o Capítulo 20 detalhará) podem "puxar" a cor do vazio adjacente para dentro da superfície, produzindo costuras escuras ou claras visíveis no objeto. O *padding* resolve isso **estendendo a cor da borda da ilha alguns pixels para fora**, criando uma faixa de transição que absorve essa puxada sem que o vazio invada a superfície. Todo *baking* bem-feito gera *padding*, e quanto menores os *mipmaps* que o *asset* usará, maior a margem necessária.

O *padding* exemplifica uma lição que percorrerá toda esta parte: a eficiência de um *asset* não é um ato isolado no fim do *pipeline*, mas o resultado de decisões coordenadas desde o início. Um *padding* adequado depende de ilhas UV com espaçamento suficiente entre si (Parte II); um *baking* limpo depende de UVs sem sobreposição e bem distribuídas; e tudo isso depende de uma retopologia que produziu uma *low-poly* de boa silhueta. O *baking*, portanto, não conserta más UVs nem má retopologia — apenas transfere fielmente o que a geometria e o desdobramento permitem. É por isso que ele se situa na confluência de tudo o que foi ensinado: é o momento em que a modelagem, o desdobramento e a texturização se encontram, e em que a qualidade de cada etapa anterior se revela.

> ⚠️ **Atenção**
> O *baking* não corrige más UVs nem má retopologia — apenas transfere fielmente o que a geometria e o desdobramento permitem. Investir na qualidade das etapas anteriores antes de assar é obrigatório; corrigir um desdobramento defeituoso após o *baking* significa refazer toda a texturização subsequente.

## Aplicação em Jogos

Na produção de jogos, o *baking* é uma etapa obrigatória e padronizada do fluxo de criação de quase todo *asset* de alta qualidade. O fluxo canônico é: esculpir ou modelar a *high-poly* com todo o detalhe; retopologizar para obter a *low-poly* otimizada; desdobrar suas UVs (mapeamento UV); assar o conjunto de mapas derivados; e só então texturizar, usando esses mapas tanto diretamente (o de normais, aplicado para fingir o relevo) quanto como máscaras (curvatura e oclusão, para guiar desgaste e sujeira, como visto no Capítulo 17). Esse encadeamento — *high*, retopo, UV, *bake*, textura — é tão consolidado que estrutura a divisão de trabalho dos estúdios e a própria sequência de aprendizado do modelador de jogos digitais. Ferramentas como o Marmoset Toolbag, o Substance Painter e o próprio Blender oferecem *bakers* dedicados, e dominar pelo menos um deles é requisito básico da profissão.

*DOOM* (id Software, 2016) é um caso documentado de sculpt/bake levado a escala extrema: Hugo Martin, art director, descreveu na GDC 2017 que cada demônio partia de uma sculpt com dezenas de milhões de polígonos — artistas passavam semanas detalhando anatomia, textura de pele e protuberâncias —, e o bake transferia esse trabalho para modelos dentro do orçamento do id Tech 6. O resultado foi o nível de detalhe de criaturas normalmente reservado a *cutscenes* aplicado em tempo real, sobre a mesma contagem de polígonos de qualquer outro jogo da época.

A importância econômica do *baking* fica evidente quando se considera a escala de uma cena de jogo. Um cenário moderno pode conter centenas de objetos visíveis simultaneamente, cada um precisando parecer detalhado e responder corretamente à iluminação PBR da Parte III. Sem o *baking*, cada objeto exigiria geometria pesada, e a cena seria impossível em tempo real; com ele, cada objeto carrega seu detalhe em textura, leve de processar, e a cena cabe no orçamento da plataforma — seja um console, um PC ou um celular. O *baking* é, nesse sentido, o que torna possível a densidade visual dos jogos contemporâneos.

## Estudo de Caso

### Gears of War e a Popularização do Bake High-to-Low em AAA — Epic Games, 2006

Antes de *Gears of War* (2006), o bake high-to-low era uma técnica de produção cinematográfica e de efeitos visuais raramente aplicada a jogos em larga escala. O artigo de Jeff Farris na GDC 2007, *"Unreal Engine 3 Art Pipeline"*, documentou como a Epic Games construiu um pipeline completo de sculpting → retopologia → bake para cada personagem e arma do jogo — um processo que, na época, representava a adoção do fluxo de trabalho de VFX pela indústria de jogos.

O case central do time da Epic era o Locust, a criatura antagonista do jogo. Em vez de modelar o relevo da pele, os ossos expostos e os detalhes de armadura diretamente na malha de baixa resolução — o que exigiria centenas de milhares de polígonos por criatura —, os artistas esculpiram uma versão de alta resolução (na época usando ZBrush e Mudbox, ainda em seus primórdios) com todos os poros, rugas e detalhes de superfície. Sobre essa *high-poly*, retopologizaram uma *low-poly* limpa de apenas alguns milhares de polígonos. O bake transferiu o detalhe inteiro da sculpt para o mapa de normais, e o resultado foi um Locust que parecia ter geometria de cinema rodando numa máquina de jogo. A comparação high vs. low lado a lado no material de imprensa de *Gears of War* tornou-se uma das imagens mais reproduzidas na história da documentação técnica de games — a demonstração visual definitiva do que o bake promete.

O pipeline tinha um problema específico: os Locusts tinham presas salientes, espinhos de armor e ornamentos que criavam peças próximas com alto risco de vazamento. A equipe da Epic desenvolveu, para isso, convenções de nomenclatura (`_LP` para low-poly, `_HP` para high-poly) e recorreu ao exploded bake para partes críticas — técnicas que entrariam na documentação padrão do Unreal Engine e que artistas de todo o mundo aprenderiam a replicar nos anos seguintes. É esse legado pedagógico que torna *Gears of War* tão relevante para este capítulo: não apenas o resultado, mas o conjunto de práticas que o estúdio tornou públicas e que o pipeline de bake da indústria toda herdou.

> **Figura 15.3** — Comparação high-poly vs. low-poly de um Locust de *Gears of War*: à esquerda, a sculpt de alta resolução com poros, rugas e detalhes de textura de pele; à direita, o modelo de jogo com poucos milhares de polígonos carregando exatamente o mesmo detalhe via mapa de normais assado.

**Screenshot sugerido:** Capturas comparando a *high-poly* e a *low-poly* do Locust publicadas no material de imprensa de *Gears of War* e na apresentação GDC 2007 — *"Unreal Engine 3 Art Pipeline"*, Jeff Farris. Fonte: `https://gearsofwar.com/en-us/games/gears-of-war` (seção de imprensa oficial) e arquivos da GDC.

**O que aprender com isso:** A publicação das convenções de nomenclatura e do exploded bake como padrões documentados, e não como segredo de estúdio, transformou uma boa prática artesanal em norma verificável. O pipeline de bake da indústria inteira é herdeiro direto dessas decisões de transparência.

## Boas Práticas

A boa prática que organiza este capítulo é tratar o *baking* como a etapa que **colhe** o que as etapas anteriores plantaram: ele só transfere fielmente o detalhe se a *high-poly* foi bem esculpida, a *low-poly* bem retopologizada e as UVs bem desdobradas. Invista, portanto, na qualidade de cada etapa anterior antes de assar, pois o *baking* não corrige seus defeitos. Em particular, garanta UVs sem sobreposição, com ilhas bem espaçadas e com a densidade de texels planejada (Parte II), pois delas dependem a limpeza do mapa e a margem de *padding*.

No próprio *baking*, controle o lançamento de raios com método. Ajuste a *cage* para que seja a menor que ainda envolve toda a *high-poly*, e prefira ajustá-la parte por parte em objetos com relevo e peças próximas. Use a correspondência por nomenclatura (`_low`/`_high`) sempre que o objeto tiver múltiplas peças, e recorra ao *exploded bake* quando as peças forem próximas demais para a nomenclatura sozinha resolver — é a defesa definitiva contra o vazamento. Ative o *supersampling* para limpar as bordas e gere *padding* suficiente para a menor resolução de *mipmap* que o *asset* usará.

> 💡 **Dica Profissional**
> Inspecione sempre cada mapa assado antes de iniciar a texturização. Um erro de *baking* não detectado — um buraco, um vazamento, uma costura — contaminará toda a texturização subsequente e, descoberto tarde, exigirá refazer tudo o que foi pintado por cima.

## Erros Comuns

> ❌ **Erro Comum**
> Tratar o *baking* como um botão mágico que conserta problemas anteriores é o equívoco mais frequente do iniciante. Assar sobre UVs ruins, malha *low-poly* mal retopologizada ou *high-poly* incompleta não produz um bom resultado. O *baking* apenas transfere; lixo na entrada produz lixo na saída.

O erro mais frequente do iniciante é tratar o *baking* como um botão mágico que conserta tudo: assar sobre UVs ruins, *low-poly* mal retopologizada ou *high-poly* incompleta e esperar um bom resultado. O *baking* apenas transfere; lixo na entrada produz lixo na saída, e nenhum ajuste de *cage* salva um desdobramento sobreposto. Ligado a ele está o erro de não inspecionar os mapas assados antes de seguir adiante, descobrindo os defeitos só no fim, quando já se texturizou por cima e o conserto custa refazer tudo.

Entre os erros técnicos específicos, o vazamento (*projection bleeding*) é o mais comum e o mais característico: detalhe de uma peça capturado por outra próxima, produzindo manchas e fantasmas no mapa. Sua causa é quase sempre uma *cage* larga demais ou peças próximas assadas juntas sem nomenclatura nem *exploded bake* — e essas são exatamente as defesas a aplicar.

> ❌ **Erro Comum**
> *Padding* insuficiente manifesta-se como costuras escuras nas bordas das ilhas UV, agravadas pelos *mipmaps* — e frequentemente é diagnosticado tarde, quando o objeto é visto de longe no jogo. O momento certo de definir o *padding* adequado é durante o *baking*, não depois.

O oposto também ocorre: uma *cage* estreita demais que deixa saliências fora do alcance dos raios, produzindo buracos no mapa onde havia mais detalhe. Há ainda o erro de *padding* insuficiente, que se manifesta como costuras nas bordas das ilhas UV, agravadas pelos *mipmaps* — frequentemente diagnosticado tarde, quando o objeto é visto de longe no jogo. E persiste o equívoco de confundir as duas famílias de *baking*: aplicar a um *asset* o pensamento do *lightmap*, ou vice-versa, sem perceber que pertencem a momentos e finalidades distintos do *pipeline*.

## Resumo

Este capítulo abriu a Parte V — dedicada a transformar materiais em *assets* eficientes — apresentando o *baking*, ou assar de texturas, como a operação que viabiliza, simultaneamente, o realismo da Parte III e o desempenho em tempo real. Definimos o *baking* como a captura de uma propriedade calculada sobre uma malha e sua gravação numa imagem, trocando cálculo em tempo de execução por leitura de textura — o princípio mais difundido da otimização gráfica. Distinguimos suas duas famílias: o *baking* de transferência entre malhas, tarefa do texturizador e foco da parte, e o *baking* de iluminação em *lightmaps*, tarefa do artista de iluminação. Mostramos que o *baking* de transferência resolve o dilema entre detalhe e desempenho convertendo geometria em textura: cria-se uma *high-poly* rica e uma *low-poly* leve, e assa-se o detalhe da primeira sobre a segunda, fazendo a malha leve responder à luz como se tivesse a forma da pesada. Explicamos seu mecanismo — a projeção por raios que partem da *low-poly* em busca da *high-poly* — e mostramos que dele decorrem todos os problemas práticos. Apresentamos os instrumentos de controle: a *cage*, cópia inflada da *low-poly* que envolve a *high-poly* e governa o alcance dos raios; a correspondência por nomenclatura (`_low`/`_high`) e o *exploded bake*, defesas contra o vazamento; o *supersampling*, contra o serrilhado; e o *padding*, contra as costuras nas bordas das ilhas UV. Por fim, situamos o *baking* na confluência da modelagem, do desdobramento e da texturização, insistindo que ele colhe, sem corrigir, a qualidade das etapas anteriores. Os capítulos seguintes detalham os mapas que dele resultam — a começar pelo mapa de normais, o mais importante de todos, tema do próximo capítulo.

## Exercícios

**1.** Explique, com suas palavras, por que o *baking* é descrito como a operação que "converte geometria em textura" e como isso resolve o dilema entre detalhe visual e desempenho em tempo real. Relacione sua resposta ao orçamento de polígonos de uma cena de jogo com muitos objetos simultâneos.

**2.** Descreva o mecanismo de projeção por raios do *baking* de transferência entre malhas. A partir dele, explique por que cada um dos três problemas a seguir ocorre: o buraco no mapa, o vazamento entre peças e a costura na borda da ilha UV.

**3.** Comparação: distinga as duas estratégias contra o vazamento — a correspondência por nomenclatura (`_low`/`_high`) e o *exploded bake*. Em que situação cada uma é suficiente sozinha, e por que o *exploded bake* é considerado a defesa definitiva para objetos com muitas peças próximas?

**4.** Diagnóstico: um artista assou o mapa de um capacete com viseira e descobriu que a superfície do rosto, sob a viseira, apresenta manchas com o desenho da viseira, e que uma das saliências do topo ficou sem detalhe (lisa). Identifique os dois erros prováveis de *cage* e proponha, para cada um, a correção adequada.

**5.** O capítulo afirma que "o *baking* não corrige más UVs nem má retopologia — apenas transfere fielmente o que a geometria e o desdobramento permitem". Explique essa afirmação relacionando o *baking* às etapas anteriores do *pipeline* (modelagem, retopologia, desdobramento) e justifique por que a inspeção dos mapas assados deve preceder a texturização.

## Glossário

**Baking (assar de texturas):** Processo de transferir informações calculadas de uma malha de alta resolução para uma imagem de textura alinhada às UVs de uma malha de baixa resolução, substituindo cálculo em tempo real por leitura de dados pré-computados.

**Cage (gaiola):** Cópia inflada da malha *low-poly* que envolve a malha *high-poly* durante o *baking*, definindo o ponto de partida e o alcance máximo dos raios de projeção.

**Costura:** Limite entre ilhas UV no desdobramento, que pode tornar-se visível na superfície do objeto quando o *padding* é insuficiente ou o *baking* é mal executado.

**Desdobramento UV:** Processo de "abrir" a superfície tridimensional de uma malha num plano bidimensional para receber texturas, equivalente a desembrulhar a superfície do objeto.

**Densidade de texels:** Relação entre o tamanho físico de uma superfície no espaço 3D e a quantidade de pixels de textura que a recobre, expressa em pixels por unidade de medida.

**Exploded bake:** Técnica de *baking* em que as peças do objeto são afastadas espacialmente antes da projeção, eliminando o risco de vazamento entre peças próximas.

**High-poly:** Modelo de alta densidade de polígonos, rico em detalhe geométrico, usado como fonte de informação no *baking* e substituído em jogo pela versão *low-poly*.

**Ilhas UV:** Regiões contíguas da superfície de uma malha, separadas por costuras, que aparecem como áreas planas e delimitadas no espaço UV.

**Low-poly:** Modelo de baixa densidade de polígonos, com silhueta correta mas superfície simplificada, que receberá os mapas assados e será usado no motor de jogo.

**Mapa de normais:** Textura que armazena informações de orientação da superfície da malha *high-poly* de modo que, aplicada à malha *low-poly*, simule o efeito de iluminação do relevo original.

**Mapeamento UV:** Processo de associar pontos da superfície tridimensional de uma malha a coordenadas bidimensionais (U e V) que indexam sua posição numa textura.

**Padding (sangria):** Extensão da cor das bordas de cada ilha UV para o espaço vazio ao redor, evitando que a interpolação e os *mipmaps* produzam costuras visíveis.

**Retopologia:** Processo de reconstruir a superfície de um modelo com uma nova malha de topologia limpa e contagem de polígonos controlada, partindo de um modelo de alta densidade.

**Vazamento (projection bleeding):** Artefato de *baking* em que raios de uma peça do objeto capturam informação de uma peça vizinha, produzindo manchas ou "fantasmas" no mapa assado.

## Leituras Complementares

- **[What Is Texture Baking?]** — Introdução direta ao conceito de *baking* como transferência de detalhe da *high-poly* para a *low-poly*, com a explicação da *cage* e da pré-computação que fundamentam este capítulo. Ponto de partida recomendado para quem está encontrando o tema pela primeira vez.
- **[Baking Guide]** — Aprofunda o processo prático do *baking*, os mapas gerados e os cuidados de projeção, complementando a discussão sobre *cage*, vazamento e *padding*. Consultar em paralelo à leitura das seções sobre instrumentos de controle.
- **[Blender Maps] e [Blender Normals]** — Situam o *baking* dentro de um fluxo concreto de geração de mapas em Blender, úteis para observar o processo aplicado a um modelo real e para compreender como o motor interpreta os mapas resultantes.
- **[Blender Manual]** — https://docs.blender.org/. Seções sobre *baking* de texturas, configuração de *cage* e geração de mapas a partir da geometria, para praticar o processo num ambiente de código aberto e relacionar a teoria deste capítulo a controles concretos de ferramenta.
- **[Adobe Substance 3D]** — https://helpx.adobe.com/substance-3d.html. Documentação do Substance Painter sobre o *baking* de mapas de malha (*mesh maps*), nomenclatura de malhas, *cage* e *exploded bake*, referência direta do fluxo profissional descrito neste capítulo.
- **[Real-Time Rendering]** — AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. 4. ed. Boca Raton: CRC Press, 2018. Os capítulos sobre *texturing* e representação de detalhe de superfície fundamentam tecnicamente a troca entre geometria e textura que o *baking* realiza, para quem quiser compreender a base matemática da operação.
