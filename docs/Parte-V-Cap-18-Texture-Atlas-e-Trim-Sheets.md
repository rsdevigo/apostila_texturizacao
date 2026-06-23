# Capítulo 18 — Texture Atlas e Trim Sheets

## Introdução

A abertura desta parte tratou da eficiência no nível do *asset* individual — o *baking* e o mapa de normais — e da colocação de detalhe pelas máscaras, *stencils* e *decals*. Este capítulo eleva a questão da eficiência a um nível acima — o da **cena inteira** — e introduz uma preocupação que ainda não havíamos enfrentado diretamente: não basta que cada *asset* seja leve; é preciso que o conjunto de *assets* de uma cena possa ser desenhado pela placa de vídeo de modo econômico. E aqui surge um custo que o iniciante costuma ignorar, porque não é o custo dos polígonos nem o da memória de textura, mas o custo de **trocar de material** durante a renderização. Compreender esse custo é o que dá sentido às técnicas centrais deste capítulo: o **texture atlas** e a **trim sheet**, formas de organizar muitas superfícies numa só textura para que o jogo as desenhe de uma vez.

A *trim sheet* já foi apresentada na Parte IV, no Capítulo 12, do ponto de vista da **produção** — como técnica de reutilização de acabamentos que combate a repetição e acelera a texturização de ambientes modulares. Aqui a retomamos por um ângulo complementar, o da **eficiência de renderização**: por que reunir muitas superfícies numa única textura não apenas poupa trabalho de autoria, mas reduz o custo que a placa de vídeo paga a cada quadro. O *texture atlas*, por sua vez, é o conceito mais geral do qual a *trim sheet* é um caso especializado, e merece tratamento próprio. Este capítulo, portanto, conecta a produção da Parte IV à otimização desta parte, mostrando que uma boa decisão de organização de texturas serve, ao mesmo tempo, à economia de trabalho e à economia de desempenho — e que é precisamente essa dupla economia o que torna o *atlas* uma das ferramentas mais valiosas do *pipeline*.

## Desenvolvimento

### O custo das draw calls

Para entender por que se reúnem texturas, é preciso entender o que custa **não** reuni-las, e isso exige um conceito de renderização ainda não desenvolvido na apostila: a ***draw call***.

> 📘 **Definição**
> Uma ***draw call*** é uma ordem enviada pelo processador à placa de vídeo para desenhar um conjunto de polígonos com um determinado material. Cada *draw call* tem um custo fixo de preparação no processador — independente de quantos polígonos estão sendo desenhados.

Quando o jogo desenha a cena, ele não a envia à placa de vídeo de uma vez só; envia-a em lotes, e cada lote é uma *draw call*. Cada *draw call* tem um custo fixo de preparação, pago pelo processador antes de a placa de vídeo trabalhar: configurar o material, indicar quais texturas usar, ajustar o estado da renderização. Esse custo, individualmente pequeno, multiplica-se pelo número de *draw calls* por quadro, e quando há milhares delas o processador se torna o gargalo — a placa de vídeo fica ociosa, esperando ordens, e a taxa de quadros despenca. Reduzir o número de *draw calls* é, por isso, uma das otimizações mais importantes de qualquer jogo, especialmente em plataformas modestas como celulares.

O ponto crucial, para o texturizador, é o que **força** uma nova *draw call*: a troca de material, e em particular a troca de textura. Objetos que compartilham o mesmo material e as mesmas texturas podem, em geral, ser desenhados juntos, num único lote (técnica chamada *batching*); objetos que usam materiais ou texturas diferentes exigem, cada um, sua própria *draw call*, pois a placa precisa ser reconfigurada entre eles. Disso decorre uma consequência direta e poderosa: **se muitos objetos compartilharem uma mesma textura, eles podem ser desenhados juntos, economizando** *draw calls*.

Convém registrar, no entanto, que os motores de jogo modernos sofisticaram esse quadro, e o atlas já não é a única via para reduzir *draw calls*. Mecanismos como o *SRP Batcher* da Unity agrupam objetos por variante de sombreador, sem exigir que compartilhem uma mesma textura — *Ori and the Will of the Wisps* (Moon Studios, 2020) foi documentado pela Unity em seu blog oficial como um caso em que a adoção do SRP Batcher reduziu drasticamente as *draw calls* sem reorganizar o atlas de texturas do jogo, demonstrando que a arquitetura de renderização do motor pode substituir parte do trabalho de consolidação de atlas; o *GPU instancing* desenha em lote muitas cópias de uma mesma malha independentemente do atlas; e sistemas como o *Nanite*, da Unreal Engine 5, reorganizam por completo a contabilidade de geometria e de chamadas. Nada disso anula o valor do atlas — ele continua reduzindo o número de texturas, simplificando a gestão e favorecendo o agrupamento —, mas o texturizador atual deve entender que a relação "uma textura compartilhada ⇒ menos *draw calls*" é uma simplificação útil, e não uma lei absoluta em todos os *pipelines*.

> ⚠️ **Atenção**
> A relação entre atlas compartilhado e redução de *draw calls* não é universal: depende do motor de jogo, da versão e das técnicas de agrupamento disponíveis. Antes de investir na criação de atlas visando desempenho, verifique o sistema de *batching* do motor de destino e meça o impacto real com as ferramentas de profiling disponíveis.

> **Figura 18.1** — Esquema comparando duas cenas com os mesmos dez objetos — na primeira, cada objeto tem sua própria textura, gerando dez *draw calls*; na segunda, todos compartilham um único *atlas*, permitindo que sejam agrupados em uma só *draw call* —, com uma barra indicando a queda no custo de processamento.

**Screenshot sugerido:** Captura do contador de *draw calls* (estatísticas de renderização) de um motor de jogo, mostrando a queda no número de chamadas ao reunir vários objetos sob um *atlas* compartilhado.

### O texture atlas

> 📘 **Definição**
> Um ***texture atlas*** é uma textura única que reúne, lado a lado, muitas imagens menores que de outro modo seriam texturas separadas. As UVs de cada objeto apontam para sua região correspondente dentro do *atlas*, permitindo que muitos objetos compartilhem uma só textura.

Um *texture atlas* serve a dois propósitos que convém distinguir. No primeiro, reúne **objetos diferentes** numa textura comum — os móveis de uma sala, os elementos de uma interface, os vários adereços de um cenário —, e o ganho é sobretudo de *draw calls*, pelo agrupamento. No segundo, reúne as **várias partes de um mesmo objeto** numa única textura — em vez de o personagem ter uma textura para o rosto, outra para a roupa, outra para os acessórios, tem um só *atlas* com tudo —, e o ganho é tanto de *draw calls* (o objeto inteiro vira um lote) quanto de simplicidade de gestão. Em ambos os casos, o princípio é o mesmo: **uma textura para muitos**, em vez de muitas para muitos.

O custo dessa reunião é o planejamento — é preciso distribuir as imagens no *atlas* sem desperdiçar espaço, respeitando a densidade de texels (Parte II) de cada parte e deixando margem entre elas para evitar sangramento, exatamente como o *padding* do *baking*. Mas o ganho de desempenho compensa amplamente, e o *atlas* é prática difundida em todos os tipos de jogo.

### A trim sheet revisitada: o atlas dos acabamentos

> 📘 **Definição**
> Uma ***trim sheet*** é um tipo especializado de *texture atlas* organizado em faixas horizontais, cada uma contendo um acabamento reutilizável — friso, rebordo, moldura, chapa. A geometria do cenário é construída para encaixar suas superfícies nessas faixas, de modo que uma única textura veste muitas geometrias distintas.

A *trim sheet*, apresentada no Capítulo 12, pode agora ser entendida com precisão como um **tipo especializado de** *atlas*: uma textura organizada em **faixas horizontais**, cada uma contendo um acabamento reutilizável. O que distingue a *trim sheet* do *atlas* genérico é essa lógica de faixas pensadas para serem aplicadas a **muitas geometrias diferentes** ao longo do cenário — uma mesma faixa de moldura serve a centenas de portas, janelas e painéis —, levando a reutilização ao extremo.

Vista pelo ângulo desta parte, a *trim sheet* é uma máquina de economia em duas frentes simultâneas. Na frente da **produção**, como já vimos no Capítulo 12, ela permite texturizar um cenário inteiro com pouquíssimas texturas, pois o mesmo conjunto de acabamentos veste número virtualmente ilimitado de superfícies — daí sua importância na construção modular. Na frente do **desempenho**, que agora destacamos, ela reduz drasticamente as *draw calls*: como todo o cenário modular usa a mesma *trim sheet* (e talvez mais uma ou duas texturas repetíveis para as grandes áreas), enormes porções do nível compartilham material e podem ser agrupadas e desenhadas juntas. Essa dupla economia — menos trabalho de autoria e menos custo de renderização — é o que tornou a *trim sheet* a espinha dorsal da texturização de ambientes em jogos de grande porte.

> **Figura 18.2** — Uma *trim sheet* exibida como textura — faixas horizontais com molduras, frisos, chapas e parafusos — ao lado de um trecho de cenário modular cujas superfícies (vigas, portas, painéis) leem cada uma a sua faixa, ilustrando como uma única textura veste muitas geometrias distintas.

**Screenshot sugerido:** Captura de uma *trim sheet* e do cenário construído sobre ela, com linhas ligando faixas da textura às superfícies do cenário que as utilizam.

### Hotspotting: encaixe automático em variantes

Mencionada também no Capítulo 12, a técnica do ***hotspotting*** merece situar-se aqui como uma evolução do *atlas* voltada à variedade com reutilização. Um *atlas* de *hotspot* contém **muitas variantes** de um mesmo tipo de superfície — por exemplo, vários painéis de parede de tamanhos e desenhos diferentes —, e um sistema (manual ou assistido por ferramenta) **encaixa** cada face da geometria na variante do *atlas* que melhor lhe cabe pelo tamanho e proporção. O resultado é que paredes de dimensões variadas, em vez de esticarem ou repetirem uma mesma textura, recebem cada uma a variante adequada, ganhando diversidade visual — combatendo a repetição, como visto na Parte IV — sem que se acrescente nenhuma textura: tudo vem do mesmo *atlas* de *hotspot*, e portanto com o mesmo benefício de *draw call*.

> 💡 **Dica Profissional**
> O *hotspotting* é especialmente valioso em cenários com superfícies de muitos tamanhos — corredores, paredes irregulares, halls de proporções variadas. Em vez de criar texturas específicas para cada dimensão, mantém-se um único *atlas* com variantes e deixa-se o sistema de encaixe escolher a mais adequada para cada face.

O *hotspotting* exemplifica bem a maturidade da ideia de *atlas*. As três técnicas — *atlas*, *trim sheet*, *hotspot* — formam uma família que responde à mesma pergunta com graus crescentes de sofisticação: como fazer poucas texturas servirem a muitas superfícies, economizando ao mesmo tempo memória, trabalho e *draw calls*. Dominar essa família é dominar a economia de texturas no nível da cena.

### Os limites e os custos do atlas

O *atlas* não é gratuito nem universal, e o texturizador maduro conhece seus limites. O primeiro é o conflito com a **repetição**: uma textura repetível, por definição, repete-se ao longo da superfície ultrapassando as bordas das UVs, mas uma imagem dentro de um *atlas* não pode repetir-se sem invadir as imagens vizinhas. Por isso, grandes superfícies que precisam de uma textura repetida — um piso extenso, uma parede longa — geralmente **não** vão para um *atlas*, e usam sua própria textura repetível; o *atlas* serve melhor a superfícies de área limitada e mapeamento UV único.

O segundo custo é o da **densidade de texels** (Parte II): ao reunir muitas imagens numa textura de tamanho fixo, cada imagem recebe apenas uma fração da resolução, e é preciso distribuir o espaço do *atlas* conforme a importância e a proximidade de cada superfície — dar mais área às que o jogador vê de perto, menos às distantes —, sob pena de algumas ficarem borradas. Há ainda a perda de flexibilidade: alterar a textura de um único objeto que está num *atlas* compartilhado obriga a mexer no *atlas* inteiro, afetando todos os que o usam, o que complica iterações tardias. E há o limite de **resolução máxima de textura** da plataforma — um *atlas* não pode crescer indefinidamente para caber tudo —, o que obriga a equilibrar quantas imagens reunir e a que resolução.

> ⚠️ **Atenção**
> Não coloque no *atlas* texturas que precisam repetir-se ao longo de grandes superfícies. A repetição UV de uma imagem dentro de um *atlas* invade as imagens vizinhas, produzindo sangramento e artefatos. Texturas repetíveis de piso, parede e terreno devem permanecer como texturas independentes.

## Aplicação em Jogos

Na produção, atlas e trim sheets são decisões de arquitetura de projeto tomadas cedo e que estruturam todo o trabalho de cenário. Equipes de ambiente definem, no início, o conjunto de *trim sheets* e texturas repetíveis que vestirá o jogo inteiro — uma paleta de acabamentos e materiais base —, e modelam a geometria modular já pensando em encaixá-la nessas texturas, num fluxo em que modelagem, desdobramento UV e texturização são planejados em conjunto, exatamente como antecipava a Parte II. Esse plano viabiliza cenários enormes com pouquíssimas texturas e pouquíssimas *draw calls*, e é por isso que jogos de mundo aberto e de grande porte dependem tão fortemente dessas técnicas. Em jogos para celular, onde o orçamento de *draw calls* é ainda mais apertado, o *atlas* é praticamente obrigatório: reúnem-se em *atlas* não só cenários, mas personagens, efeitos e, sempre, os elementos de interface.

O *atlas* tem ainda aplicações especializadas que vale conhecer. Os elementos de interface (botões, ícones, molduras) são quase sempre reunidos num *atlas* — o chamado *sprite sheet* —, para que toda a interface seja desenhada em pouquíssimas *draw calls*. *Diablo III* (Blizzard Entertainment, 2012) é um caso amplamente citado dessa prática: toda a interface — barras de vida, ícones de habilidades, molduras de inventário, minimapa e textos de diálogo — foi construída sobre *sprite sheets* empacotados, de modo que uma tela de inventário que, peça a peça, exigiria centenas de *draw calls* passava a ser desenhada em dezenas. As animações em folhas de sprites, em jogos 2D e em efeitos de partícula, são *atlases* em que cada quadro da animação ocupa uma célula.

## Estudo de Caso

### *Titanfall* (Respawn Entertainment, 2014) — Trim Sheets como Espinha Dorsal de um Mundo Modular

*Titanfall* é uma das produções mais frequentemente citadas em discussões sobre texturização de ambientes de grande escala pela forma como a Respawn Entertainment sistematizou o uso de *trim sheets* como estrutura central de toda a produção visual do jogo. O *pipeline* da equipe de ambiente foi documentado por artistas do estúdio em palestras da GDC e em postagens técnicas que descrevem um princípio central: o mundo do jogo — cidades de arranha-céus, fábricas, depósitos, zonas de batalha — seria texturizado inteiramente com um conjunto mínimo de *trim sheets* temáticas, cada uma cobrindo um vocabulário arquitetônico específico. Paredes, vigas, dutos, painéis, bordas e acabamentos de toda a produção convergiam para essas folhas, e a geometria era construída desde o início para se encaixar nelas, num *pipeline* em que modelagem e texturização eram planejadas em conjunto, não em sequência.

A decisão arquitetural tem consequência direta de desempenho: como enormes porções do cenário de *Titanfall* compartilham as mesmas *trim sheets*, o motor de jogo pode agrupá-las em lotes mínimos de *draw calls*, permitindo que o jogo rode fluidamente em consoles de geração anterior com cenários de escala urbana. Mas os artistas do estúdio também documentaram o ganho de produção: uma vez definidas as *trim sheets*, texturizar um novo módulo de ambiente significa encaixá-lo nas faixas existentes, não criar novas texturas. Esse fluxo de trabalho — modelagem modular encaixada em *trim sheets* pré-definidas — tornou possível construir e iterar sobre o enorme volume de cenários do jogo com uma equipe de ambiente relativamente enxuta.

O aspecto que conecta o caso diretamente à discussão técnica deste capítulo é a escolha de organizar as *trim sheets* em faixas horizontais deliberadamente projetadas para máxima reutilização: a mesma faixa de rebordo metálico que aparece na borda de um painel serve à beirada de uma plataforma, ao remate de uma viga e ao encaixe de uma porta, porque a geometria modular foi desenhada para ler sempre a mesma região da *trim sheet*. Isso é a *trim sheet* em sua forma mais madura — não uma textura que reúne acabamentos por conveniência, mas um sistema de linguagem arquitetônica visual, em que o vocabulário de superfícies é definido primeiro e a geometria construída para falar esse vocabulário.

**O que aprender com isso:** *Titanfall* demonstrou, em escala industrial, que a *trim sheet* é simultaneamente uma ferramenta de produção, de coerência visual e de otimização de renderização — a tripla economia que este capítulo defende.

> **Figura 18.3** — Uma *trim sheet* típica de *Titanfall* com suas faixas de acabamentos (rebordos, painéis, frisos, parafusos) ao lado de um fragmento de ambiente do jogo, com linhas conectando cada faixa às superfícies da geometria que as leem.

**Screenshot sugerido:** Imagem de breakdowns de arte de ambiente de *Titanfall* disponíveis no ArtStation por artistas da Respawn Entertainment, mostrando a trim sheet e o cenário construído sobre ela lado a lado.

**Fonte:** Palestras de artistas da Respawn Entertainment na GDC 2014 e postagens técnicas no ArtStation; referência complementar em Morten Olsen, "The Ultimate Trim — Texturing Techniques of Sunset Overdrive" (Insomniac Games), que documenta *pipeline* equivalente.

## Boas Práticas

A boa prática que organiza o capítulo é planejar a organização das texturas **no nível da cena, e cedo**: decidir, no início do projeto, quais superfícies irão para *trim sheets*, quais para *atlases* de objetos, quais para texturas repetíveis individuais, e construir a geometria já pensando nesse encaixe. Reúna em *atlas* os muitos objetos pequenos e as partes de um mesmo objeto, para que compartilhem material e sejam agrupados, reduzindo *draw calls* — preocupação tanto mais crítica quanto mais modesta a plataforma. Use a *trim sheet* como espinha dorsal da texturização modular de cenários, levando a reutilização ao extremo e colhendo a dupla economia de produção e desempenho, e recorra ao *hotspotting* para dar variedade às superfícies de tamanhos variados sem acrescentar texturas.

Ao montar um *atlas*, distribua o espaço conforme a densidade de texels desejada — mais área às superfícies vistas de perto, menos às distantes — e deixe margem (*padding*) entre as imagens para evitar sangramento, sobretudo nos *mipmaps* (Capítulo 20). Reserve as grandes superfícies que exigem repetição para texturas repetíveis próprias, fora do *atlas*, pois imagens em *atlas* não repetem sem invadir as vizinhas. Respeite o teto de resolução da plataforma ao decidir quantas imagens reunir e a que qualidade, equilibrando quantidade e nitidez. E lembre-se de que o *atlas* troca flexibilidade por economia: alterá-lo afeta todos os que o usam, de modo que convém estabilizar seu conteúdo antes de produzir em escala.

## Erros Comuns

O erro de fundo é ignorar o custo das *draw calls* e tratar a organização de texturas como questão apenas de produção, dando a cada objeto sua própria textura e descobrindo tarde, quando o desempenho cai, que o cenário está fragmentado em milhares de materiais que o motor de jogo não pode agrupar.

> ❌ **Erro Comum**
> Dar a cada objeto sua própria textura individualmente é o caminho mais simples na autoria, mas resulta em tantas *draw calls* quanto o número de objetos — um cenário com quinhentos props pode estar custando quinhentas chamadas quando poderia custar uma dezena com atlas bem planejados.

Próximo dele está o erro oposto e ingênuo: tentar enfiar **tudo** num *atlas*, inclusive grandes superfícies que precisam de repetição, e descobrir que a textura repetível não funciona dentro do *atlas* — pois a repetição invadiria as imagens vizinhas. Saber o que vai para o *atlas* (objetos e partes de mapeamento UV único) e o que fica fora (grandes superfícies repetidas) é a distinção que evita ambos.

No uso do *atlas*, o erro de distribuição de espaço é frequente: dar a mesma área a todas as imagens, independentemente de quão de perto serão vistas, resultando em superfícies importantes borradas e superfícies distantes com resolução desperdiçada — a falha de densidade de texels da Parte II, agora dentro do *atlas*. Há também o erro de margem insuficiente entre as imagens, que produz sangramento de uma para outra, agravado pelos *mipmaps* — o mesmo problema do *padding* do *baking*, na escala do *atlas*.

> ❌ **Erro Comum**
> Esquecer o *padding* (margem) entre as imagens de um *atlas* produz sangramento quando os *mipmaps* são gerados, pois as versões reduzidas da textura misturam pixels de imagens vizinhas. A margem mínima recomendada depende do número de níveis de *mipmap* — quanto mais *mipmaps*, mais padding é necessário.

Por fim, persiste o equívoco conceitual de confundir os ganhos: achar que o *atlas* reduz a memória de textura (seu ganho principal é de *draw calls* e de gestão; a memória pode até não cair) ou que a *trim sheet* serve só à produção (esquecendo seu papel decisivo no desempenho) — confusões que levam a usar a técnica errada para o objetivo em vista.

## Resumo

Este capítulo elevou a questão da eficiência do nível do *asset* ao nível da cena, introduzindo o custo das *draw calls* — as ordens de desenho que o processador prepara e que se multiplicam a cada troca de material e de textura — como a razão de desempenho para reunir texturas. Mostramos que objetos que compartilham textura podem ser agrupados (*batching*) e desenhados juntos, e que disso decorre o valor do **texture atlas**: uma textura única que reúne muitas imagens menores, fazendo muitos objetos, ou as muitas partes de um objeto, usarem a mesma textura e caírem no mesmo lote. Revisitamos a **trim sheet** do Capítulo 12 como um *atlas* especializado em faixas de acabamentos reutilizáveis, destacando sua dupla economia — de produção, já vista, e de desempenho, agora enfatizada —, e situamos o **hotspotting** como a evolução que encaixa geometrias de tamanhos variados em variantes de um *atlas*, dando diversidade sem novas texturas. Examinamos os limites e custos dessas técnicas: o conflito com a repetição (texturas repetíveis não cabem em *atlas*), a partilha da densidade de texels entre as imagens reunidas, a perda de flexibilidade e o teto de resolução da plataforma — limites que determinam quando usar cada estratégia. O próximo capítulo trata de um conjunto de técnicas que respondem ao problema inverso — quando uma única textura não basta para um objeto que exige altíssima resolução —, os UDIMs, os texture arrays e o multi-tile.

## Exercícios

**1.** Explique o que é uma *draw call* e por que a troca de material e de textura força novas *draw calls*. A partir daí, justifique por que reunir muitos objetos numa única textura melhora o desempenho, e por que esse ganho é especialmente importante em plataformas modestas.

**2.** Comparação: distinga o *texture atlas* genérico, a *trim sheet* e o *hotspotting* quanto ao que cada um reúne, ao tipo de superfície que melhor servem e ao problema específico que cada um resolve. Mostre em que sentido os três formam uma família com graus crescentes de sofisticação.

**3.** O capítulo afirma que a *trim sheet* gera uma "economia em duas frentes". Explique as duas frentes — produção e desempenho — relacionando a primeira ao que foi visto no Capítulo 12 e a segunda ao custo das *draw calls* deste capítulo.

**4.** Diagnóstico: uma equipe colocou num único *atlas* tanto os pequenos adereços de uma sala quanto a textura do piso, que precisa repetir-se ao longo de um salão extenso. O piso aparece com costuras e não consegue repetir corretamente. Identifique o erro, explique por que uma textura que precisa repetir não funciona dentro de um *atlas* e proponha a correção.

**5.** Planejamento de *pipeline*: você vai texturizar uma vila medieval modular (casas, muros, portões, adereços) para rodar em um aparelho de desempenho limitado. Descreva, em texto contínuo, como distribuiria as superfícies entre *trim sheets*, *atlases* de objetos, *hotspot* e texturas repetíveis individuais, justificando cada escolha pelos objetivos de economia de *draw calls*, de memória e de trabalho, e respeitando os limites discutidos no capítulo.

## Glossário

**Batching:** Técnica de renderização que agrupa múltiplos objetos que compartilham o mesmo material numa única *draw call*, reduzindo o custo de preparação no processador.

**Densidade de texels:** Relação entre o tamanho físico de uma superfície e a quantidade de pixels de textura que a recobre, geralmente expressa em pixels por metro. Determina o nível de detalhe visível na textura aplicada.

**Draw call:** Ordem enviada pelo processador à placa de vídeo para desenhar um conjunto de polígonos com um determinado material. Cada *draw call* tem um custo fixo de preparação; seu acúmulo pode se tornar o gargalo de desempenho de um jogo.

**Hotspotting:** Técnica em que um *atlas* contém múltiplas variantes de um mesmo tipo de superfície, e cada face da geometria é encaixada na variante mais adequada pelo tamanho e proporção, conferindo diversidade visual sem acrescentar novas texturas.

**Mapeamento UV:** Sistema de coordenadas bidimensionais (U e V) que determina como a textura é aplicada sobre a superfície tridimensional da malha.

**Padding:** Margem de pixels deixada entre as imagens dentro de um *atlas*, necessária para evitar que os *mipmaps* ou a filtragem de textura causem sangramento entre as regiões adjacentes.

**Sprite sheet:** Tipo de *texture atlas* especializado para elementos de interface ou animações 2D, em que cada célula contém um estado ou quadro distinto.

**Texture atlas:** Textura única que reúne, lado a lado, muitas imagens menores. Permite que objetos distintos compartilhem a mesma textura, favorecendo o *batching* e reduzindo *draw calls*.

**Trim sheet:** Tipo especializado de *texture atlas* organizado em faixas horizontais de acabamentos reutilizáveis. Combina economia de produção (reutilização em muitas superfícies) e de desempenho (redução de *draw calls* em cenários modulares).

## Leituras Complementares

- **OLSEN, Morten. *The Ultimate Trim — Texturing Techniques of Sunset Overdrive*** (Insomniac Games; material de apoio da disciplina) — Referência central deste capítulo: documenta, a partir de um jogo de grande porte, a técnica das *trim sheets* e a filosofia de reutilização extrema que sustenta tanto a economia de produção quanto a de desempenho aqui discutidas. Leitura obrigatória para quem trabalha com cenários modulares.

- **AOD — Texel Density** (material de apoio da disciplina) — Fundamenta a distribuição de resolução entre as imagens reunidas num *atlas* e a relação entre densidade de texels e economia que orienta o planejamento das texturas no nível da cena.

- **Hotspot Texturing** (Default Interactive) — https://www.defaultinteractive.co.uk/post/hotspot-texturing. Discussão prática do *hotspotting* — o encaixe da geometria em um *atlas* de variantes — e de seu papel na variedade visual e na economia de texturas; base direta da seção correspondente.

- **Unreal Engine Documentation** (https://dev.epicgames.com/documentation/unreal-engine/) e **Unity Manual** (https://docs.unity3d.com/) — Seções sobre *draw call batching*, *static/dynamic batching*, empacotamento de *atlas* e construção modular de ambientes; indispensáveis para observar como cada motor implementa o agrupamento e a economia de chamadas.

- **Godot Engine Documentation** — https://docs.godotengine.org/. Seções sobre *atlas* de textura, *batching* e otimização de renderização em um motor de código aberto.

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. ***Real-Time Rendering***. 4. ed. Boca Raton: CRC Press, 2018 — Os capítulos sobre arquitetura de *pipeline* gráfico, *batching* e custo de mudança de estado fundamentam tecnicamente a discussão das *draw calls* e do *atlas*.
