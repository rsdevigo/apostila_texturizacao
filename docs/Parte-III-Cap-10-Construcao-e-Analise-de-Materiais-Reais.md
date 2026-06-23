# Capítulo 10 — Construção e Análise de Materiais Reais

## Introdução

Os dois capítulos anteriores nos deram os fundamentos do PBR e o vocabulário dos mapas que compõem um material. Sabemos *por que* o paradigma existe e *o que* cada mapa descreve. Falta o passo decisivo, que é também o mais difícil de ensinar sem cair no tutorial: como, diante de uma superfície do mundo real — um pedaço de couro gasto, uma chapa de aço pintada, uma parede de tijolos —, decidir que valores e que variações colocar em cada mapa para reconstruí-la de modo convincente. Este capítulo trata da *construção* e da *análise* de materiais reais: do raciocínio que conduz da observação de uma superfície à sua tradução em mapas PBR, e do caminho inverso, que parte de um material pronto e o decompõe para entender como foi feito. São duas faces de uma mesma competência, e ambas se aprendem menos por receita do que por método e prática crítica.

Manteremos, como exige o estilo desta apostila, o foco em conceitos e tomadas de decisão independentes de software. Não ensinaremos a operar o Substance Painter nem a configurar um nó no Blender; ensinaremos a *pensar* a construção de um material — a observar uma superfície com olhos de quem precisa reconstruí-la, a distribuir suas propriedades pelos mapas certos, a usar os processos de geração de mapas preparados desde a Parte II e a julgar criticamente o resultado. Trataremos também de um modo de trabalho que organiza a autoria moderna, o pensamento por **camadas**, e de como o *baking* — antecipado desde o Capítulo 7 e detalhado adiante na Parte V — entra concretamente no fluxo. Ao final, o estudante deverá ser capaz de olhar para uma superfície qualquer e esboçar mentalmente o material PBR que a reproduziria, e de olhar para um material pronto e reconhecer as decisões que o constituíram.

## Desenvolvimento

### Observar antes de construir: a leitura física de uma superfície

A construção de um bom material começa muito antes de qualquer mapa: começa na *observação*. Diante de uma superfície que se deseja reproduzir — seja um objeto físico em mãos, seja uma fotografia de referência —, o artista treinado faz uma leitura física, decompondo o que vê nas propriedades que os mapas PBR representam. Pergunta-se: de que material isto é feito, metal ou não? Onde está liso e onde está rugoso? Que cores reais tem, descontadas as luzes e sombras do momento? Onde há relevo de forma, e onde apenas variação de acabamento? Que história a superfície conta — onde foi tocada, gasta, suja, molhada, riscada? Essa decomposição mental é a tradução, em sentido inverso, do que o Capítulo 9 ensinou: se cada propriedade tem seu mapa, então observar uma superfície é separar mentalmente suas propriedades para depois distribuí-las.

> 💡 **Dica Profissional**
> Antes de abrir qualquer ferramenta, observe a superfície de referência e responda por escrito: (1) é metal ou dielétrico, onde e por quê? (2) Onde é mais liso e onde é mais rugoso, e qual processo físico causou isso? (3) Qual é a cor real, eliminando mentalmente os reflexos e sombras do momento? (4) Que história de uso, desgaste e sujeira ela conta, e onde cada efeito se concentra? Esse exercício de decomposição é a habilidade mais subestimada e mais decisiva do texturizador.

Esse hábito de observação é, talvez, a habilidade mais subestimada do texturizador. Materiais ruins quase sempre nascem de uma observação pobre — de aceitar uma cor "de metal" genérica em vez de notar que o metal real tem variações sutis de tom, manchas, regiões mais e menos polidas. Materiais excelentes nascem de uma observação atenta, que percebe que nenhuma superfície real é uniforme: a rugosidade varia, a cor varia, a sujeira se acumula em padrões que a gravidade e o uso determinam. O Capítulo 8 nos disse que o mapa de rugosidade conta a história da superfície; a observação física é o que permite *ler* essa história na referência para depois *reescrevê-la* nos mapas.

> **Figura 10.1** — Uma fotografia de referência de uma superfície complexa — por exemplo, uma maçaneta de bronze gasta — anotada com setas que separam suas propriedades: "metal (mapa metálico)", "polido pelo uso aqui / fosco aqui (rugosidade)", "tom de bronze real, sem brilho (cor base)", "relevo das molduras (normais)", "sujeira acumulada nas reentrâncias (cor + rugosidade)".

**Screenshot sugerido:** Imagem de referência fotográfica ao lado de um esboço de decomposição, indicando, sobre a própria foto, que mapa capturaria cada característica observada.

### O *baking*: transferindo o detalhe da forma para os mapas

Antes de pintar valores, muitos materiais de jogo passam por uma etapa que vimos sendo preparada desde a Parte II e anunciada nos Capítulos 7 e 9: o **baking**, ou geração de mapas a partir da geometria. Para os fins deste capítulo, basta uma compreensão conceitual do que ele faz — seu mecanismo completo, com a projeção por raios, a *cage*, o *exploded bake* e os demais cuidados, é o tema da Parte V (Capítulos 15 e 16), para onde remetemos o leitor que quiser dominá-lo na prática.

> 📘 **Definição**
> ***Baking*** é o processo pelo qual o detalhe geométrico de uma **malha de alta densidade** — rebites, dobras, microrrelevo modelado — é capturado e gravado em mapas de textura aplicáveis à **malha de baixa densidade** que de fato roda no jogo. O principal produto é o mapa de normais; mapas auxiliares como oclusão de ambiente, curvatura e identificação de partes também são gerados nesse processo.

Aqui interessa reconhecer o *baking* como a fonte dos mapas que a construção de materiais consome. O principal produto é o mapa de normais, que registra a direção da microgeometria; mas o mesmo processo gera, tipicamente, o mapa de oclusão de ambiente, um mapa de **curvatura** (que identifica quinas e cavidades) e um mapa de **posição** ou de **identificação** (*ID*), que rotula regiões por cor para facilitar a aplicação seletiva de materiais. Esses mapas derivados não vão todos para o motor de jogo; vários servem como *insumos* para a autoria, como veremos adiante.

Para que o *baking* funcione, reencontramos as condições que a Parte II preparou: a malha de baixa densidade precisa estar bem desdobrada, com densidade de texels adequada e *padding* suficiente entre ilhas, e as duas malhas precisam estar alinhadas no espaço. Quando essas condições falham, surgem os artefatos clássicos do *baking* — detalhes que "vazam" de uma ilha para outra, costuras visíveis, faces que captam o detalhe errado por estarem mal alinhadas —, cujas causas e correções a Parte V trata uma a uma.

> ⚠️ **Atenção**
> Um desdobramento UV descuidado, que parecia inofensivo na Parte II, manifesta-se agora como um mapa de normais defeituoso: costuras visíveis, detalhe que vaza entre ilhas, regiões sem relevo onde deveria haver. O *baking* é o ponto em que a qualidade — ou a falta dela — do trabalho de mapeamento anterior se torna visível e irrecuperável sem refazer o UV.

É aqui que o cuidado da preparação se paga ou se cobra: um desdobramento UV descuidado, que parecia inofensivo na Parte II, manifesta-se agora como um mapa de normais defeituoso. O *baking* é, assim, o ponto em que a geometria preparada se converte em textura, e o estudante deve compreendê-lo não como um botão a apertar, mas como a materialização de todo o trabalho de mapeamento anterior.

### O pensamento por camadas

A autoria moderna de materiais organiza-se, em larga medida, em torno do **pensamento por camadas**. Em vez de pintar diretamente cada mapa, o artista constrói o material como uma pilha de camadas, cada uma representando um aspecto físico da superfície — o material de base, depois uma camada de desgaste nas quinas, depois sujeira acumulada nas reentrâncias, depois respingos, arranhões, poeira. Cada camada afeta simultaneamente vários mapas, exatamente como faria a alteração física real: uma camada de ferrugem escurece e alaranja a cor base, torna o metálico dielétrico e aumenta a rugosidade, tudo de uma vez. Essa é a grande virtude do modelo por camadas: ele alinha o trabalho de autoria à forma como as superfícies reais se constituem — por acúmulo de histórias sucessivas — em vez de tratar cada mapa isoladamente.

A Naughty Dog foi uma das primeiras equipes AAA a documentar publicamente a adoção do Substance Painter como ferramenta central de autoria por camadas, a partir de *The Last of Us* (2013); o artista Michael Barclay descreveu em posts da empresa como o modelo por camadas permitiu que artistas diferentes trabalhassem no mesmo *asset* em momentos distintos sem perder a lógica do material — algo impossível no pipeline de pintura manual que o estúdio usava anteriormente, onde cada revisão ameaçava apagar trabalho já feito por outro membro da equipe.

As camadas tornam-se ainda mais poderosas quando *guiadas* pelos mapas derivados do *baking*. O mapa de curvatura permite que a camada de desgaste apareça automaticamente nas quinas salientes, onde o atrito de fato gastaria a superfície; o mapa de oclusão de ambiente permite que a sujeira se acumule nas cavidades, onde de fato se depositaria; o mapa de identificação de partes (*ID*) permite aplicar materiais diferentes a regiões distintas com precisão. Esse uso de máscaras procedurais guiadas pela geometria é o que dá à autoria moderna sua eficiência e seu realismo: o desgaste e a sujeira não são pintados arbitrariamente, mas *posicionados pela própria forma do objeto*, segundo a lógica física de onde tais efeitos ocorreriam.

> 💡 **Dica Profissional**
> Compreender a articulação entre as etapas é o que torna a cadeia eficiente: geometria gera mapas derivados no *baking*, mapas derivados guiam máscaras de camadas, camadas afetam os mapas finais. A qualidade das máscaras depende da qualidade dos *bakes*, que depende da qualidade do desdobramento UV. Por isso a preparação cuidadosa da Parte II não é burocracia — ela é a fundação de toda a autoria de materiais.

### Procedural e manual: dois polos complementares

Há duas grandes abordagens para gerar o conteúdo dos mapas, e a prática profissional combina ambas. A abordagem **procedural** constrói o material a partir de regras, padrões e ruídos matemáticos, parametrizados — uma textura de madeira definida por funções que geram veios e nós, um metal escovado definido por um padrão direcional de ruído. Sua força é a flexibilidade e a resolução infinita: um material procedural pode ser ajustado por parâmetros, aplicado a objetos de qualquer tamanho sem perda de nitidez e variado para gerar muitas instâncias distintas. A ferramenta emblemática dessa abordagem é o Substance Designer, no qual materiais são construídos como redes de nós que se combinam. A abordagem **manual**, por outro lado, parte da pintura direta ou do uso de fotografias e texturas digitalizadas, dando controle artístico ponto a ponto e capturando a irregularidade orgânica que regras puras dificilmente alcançam.

Nenhuma das duas é superior em absoluto; elas atendem a necessidades diferentes e se combinam. Um material de base pode ser procedural — uma pedra, um metal genérico —, e sobre ele o artista pinta manualmente as particularidades daquele *asset* específico: a rachadura exatamente ali, a mancha exatamente aqui. O procedural dá a base consistente e reutilizável; o manual dá o caráter único. Saber quando recorrer a cada um é parte do julgamento profissional: superfícies amplas, repetitivas e reutilizáveis pedem o procedural; detalhes narrativos, específicos de um objeto, pedem a mão.

> 📘 **Definição**
> **Material procedural** é aquele construído a partir de regras matemáticas e parâmetros, sem dependência de resolução e facilmente variável. **Material pintado** (ou *hand-painted*) é construído por pintura direta, com controle artístico ponto a ponto. A prática profissional combina as duas abordagens: o procedural fornece a base consistente; o manual acrescenta o caráter singular.

### Calibração e valores de referência

Como o PBR tem base física, ele admite valores *corretos*, e não apenas plausíveis — e isso torna a **calibração** uma etapa real da construção de materiais. A indústria consolidou tabelas de valores de referência: faixas de cor base para materiais comuns (o carvão não é preto absoluto, a neve não é branco absoluto), valores de reflectância para dielétricos, valores de cor para metais comuns (ouro, cobre, alumínio, ferro). Autorar um material fisicamente correto envolve confrontar os valores escolhidos com essas referências, em vez de defini-los apenas "no olho". Um material cuja cor base extrapola as faixas físicas — um branco puro, um preto absoluto — comportar-se-á mal sob iluminação variada, exatamente o problema que o PBR existe para evitar.

A calibração liga-se também à observação e à resolução. Os valores corretos só se manifestam se a superfície tiver densidade de texels suficiente para representá-los: um material magnificamente calibrado, aplicado a um *asset* com densidade insuficiente, aparecerá borrado e perderá a variação fina de rugosidade que lhe daria vida. Reencontramos aqui, mais uma vez, o elo com a Parte II — a densidade de texels que padronizamos no Capítulo 6 é a condição para que os valores deste capítulo se realizem.

> ⚠️ **Atenção**
> Calibração e densidade de texels são inseparáveis. Um material com valores perfeitamente calibrados, aplicado a um *asset* com densidade de texels insuficiente, terá sua variação fina de rugosidade e de cor base borrada pela falta de resolução. O esforço de calibrar valores físicos corretos só se traduz em qualidade visual se o *asset* tiver resolução para exibi-los.

### Analisar um material pronto: o caminho inverso

A competência de construir tem um par indispensável: a de *analisar*. Dado um material pronto — seu próprio, o de um colega, o de uma biblioteca, o de um jogo de referência —, o artista treinado sabe decompô-lo para entender como foi feito. Isola cada mapa e o lê: o que a cor base revela sobre as cores reais escolhidas? O mapa metálico está limpo e binário, ou poluído por valores intermediários indevidos? O mapa de rugosidade tem variação rica, contando uma história, ou é chapado e uniforme? O mapa de normais carrega detalhe fino ou está liso demais? Há sinais dos artefatos típicos de *baking* — vazamentos, costuras? Essa leitura analítica é a forma mais eficaz de aprender: estudar bons materiais decompondo-os ensina mais do que qualquer descrição, e diagnosticar maus materiais identificando o mapa culpado é a base da revisão de qualidade.

A análise também é a ferramenta central da depuração. Quando um material "não parece certo", o caminho profissional não é ajustar valores ao acaso, mas isolar os mapas até localizar o problema: se o brilho está errado, suspeita-se da rugosidade ou do metálico; se a forma parece chapada, do mapa de normais; se as cores parecem lavadas ou saturadas demais, da cor base ou de um espaço de cor mal configurado. Essa disciplina diagnóstica — decompor para localizar — é a mesma que a Parte II aplicou aos problemas de mapeamento UV, e o estudante deve cultivá-la como hábito: diante de qualquer aparência indesejada, perguntar "qual mapa é responsável por isto?" e ir verificá-lo, em vez de mexer em tudo de uma vez.

## Aplicação em Jogos

Na produção real, a construção de materiais combina tudo o que vimos: parte-se da malha preparada na Parte II, faz-se o *baking* que gera o mapa de normais e os mapas derivados, constrói-se o material por camadas guiadas por esses mapas, mistura-se o procedural reutilizável com a pintura manual específica, e calibra-se o resultado contra valores de referência, verificando-o por fim no motor de destino. Esse fluxo, hoje padrão nos estúdios, é o que permite produzir, em escala, materiais consistentes e fisicamente corretos. Bibliotecas de materiais procedurais — *smart materials* e *materiais inteligentes* que já trazem camadas de desgaste guiadas por curvatura e AO — aceleram enormemente o trabalho, permitindo aplicar a um novo *asset* um "metal pintado e desgastado" completo e ajustá-lo, em vez de construí-lo do zero.

A análise, por sua vez, é central na vida de equipe. Revisões de qualidade decompõem materiais para verificar se seguem os padrões do projeto — faixas de valores, convenções de empacotamento, densidade adequada. Artistas estudam os materiais de jogos de referência para entender escolhas que admiram. E a depuração diagnóstica, mapa a mapa, é o que resolve os problemas que inevitavelmente surgem quando um material viaja da ferramenta de autoria ao motor de jogo. Em todos esses momentos, a competência dupla deste capítulo — construir observando e distribuindo, analisar decompondo e diagnosticando — é o que distingue o profissional do amador.

## Estudo de Caso

### Red Dead Redemption 2 e a Construção de Materiais a Partir do Mundo Real — Rockstar Games, 2018

> **Figura 10.2** — Arthur Morgan em close em *Red Dead Redemption 2*, mostrando couro da jaqueta (dielétrico, rugoso, com desgaste nas bordas), metal do revólver (condutor, polido nas arestas, oxidado nas reentrâncias) e tecido da camisa (dielétrico, textura uniforme, costura em relevo). Três materiais distintos, todos construídos por observação de referências físicas reais.

**Screenshot sugerido:** Captura de tela oficial de *Red Dead Redemption 2*, Rockstar Games — disponível em `https://www.rockstargames.com/reddeadredemption2/media` (press kit oficial, acesso público). Análise técnica visual: Digital Foundry — *"Red Dead Redemption 2: The Most Technically Impressive Open World Ever Made?"*, Eurogamer/Digital Foundry, 2018.

O processo de construção de materiais em *Red Dead Redemption 2* começa antes de qualquer ferramenta ser aberta. A Rockstar documentou em dev diaries que cada *asset* do jogo parte de um banco de referências fotográficas coletadas em campo: selas de couro do século XIX em museus, ferragens de rifle fotografadas em diferentes condições de luz, amostras de tecido de lã desgastada, madeira de carroça com variação de verniz e envelhecimento. Essa obsessão pela referência não é estética — é funcional. O objetivo é identificar, para cada material, os valores corretos de cor base, rugosidade e metalicidade antes de qualquer pintura, de modo que o material se comporte corretamente em qualquer condição de iluminação sem ajustes posteriores.

A sela do cavalo de Arthur Morgan é um dos *assets* mais analisados da indústria em tutoriais de texturização PBR — não porque a Rockstar a divulgou, mas porque analistas técnicos e artistas de comunidade a decompuseram metodicamente a partir de capturas do jogo. A decomposição revela um sistema de camadas que implementa exatamente o princípio deste capítulo. A base de couro tem uma cor castanha média sem sombras, derivada de fotografias de couro cromo-curtido do período; a rugosidade varia entre as regiões novas (costuras reforçadas, ainda macias) e as regiões gastas pelo uso contínuo (assentos, flancos); o desgaste das bordas é guiado pela geometria — quinas expostas sempre mais brilhantes — e não por pintura manual. Os ornamentos metálicos têm metalicidade plena e rugosidade baixa nas cabeças de parafuso polidas, rugosidade alta nos parafusos enferrujados. E a sujeira acumula-se nas reentrâncias segundo a lógica da oclusão, não por decisão artística de onde parece bonito.

O que torna *Red Dead Redemption 2* um estudo de caso particularmente valioso para este capítulo é justamente a escala em que esse cuidado se replica. O jogo tem centenas de *assets* de prop — armas, ferramentas, móveis, veículos — e todos seguem o mesmo processo de construção por observação, referência e camadas físicas. O resultado é uma coerência visual que a crítica especializada e o Digital Foundry identificaram como sem precedente na indústria: qualquer objeto, em qualquer luz, se comporta de modo fisicamente plausível porque foi construído segundo as propriedades físicas reais do material que representa.

**O que aprender com isso:** O processo da Rockstar demonstra que a referência fotográfica não é inspiração estética — é dado de calibração. Coletar referências antes de construir significa definir os valores corretos de cada mapa com base em evidências físicas reais, em vez de estimativas subjetivas. Essa disciplina, aplicada sistematicamente a todo um jogo, é o que produz a coerência visual que diferencia um projeto.

## Boas Práticas

A boa prática que organiza este capítulo é começar pela observação e pelo raciocínio físico, não pela ferramenta. Antes de construir, decomponha a superfície de referência em propriedades — material, rugosidade, cor real, relevo, história — e deixe que essa leitura oriente a distribuição da informação pelos mapas. Prepare a geometria segundo a Parte II e faça o *baking* com cuidado, garantindo desdobramento UV, densidade e alinhamento adequados, pois a qualidade dos mapas derivados condiciona tudo o que vem depois. Construa por camadas que correspondam a aspectos físicos da superfície, deixando cada camada afetar todos os mapas que a alteração real afetaria, e guie as máscaras pelos mapas de curvatura e oclusão para que desgaste e sujeira se posicionem onde a física os colocaria.

É boa prática combinar o procedural e o manual segundo a natureza da superfície — o geral e reutilizável de modo procedural, o particular e narrativo à mão — e calibrar os valores contra as referências consolidadas da indústria, evitando extremos fisicamente impossíveis. Garanta que o *asset* tenha densidade de texels suficiente para exibir a riqueza autorada, sob pena de borrar todo o esforço. E cultive a análise como hábito: estude bons materiais decompondo-os, revise os seus isolando cada mapa e depure problemas perguntando "qual mapa é responsável por isto?" antes de ajustar valores ao acaso. Por fim, verifique sempre o material no motor de destino, sob iluminação representativa, pois só aí se confirma que a construção cumpriu seu papel.

## Erros Comuns

> ❌ **Erro Comum**
> Pular a observação e começar a construir a partir de pressupostos genéricos — um "metal" uniforme, uma "madeira" chapada — produz materiais sem variação e sem história, que delatam a falta de leitura física. Antes de abrir qualquer ferramenta, observe e decomponha a superfície de referência.

> ❌ **Erro Comum**
> Tratar o *baking* como um botão mágico — sem verificar o desdobramento UV, o *padding* e o alinhamento das malhas — e descobrir tarde que o mapa de normais tem vazamentos e costuras é um dos erros mais custosos do pipeline, pois exige refazer a preparação da Parte II para resolver.

> ❌ **Erro Comum**
> Pintar desgaste e sujeira em posições arbitrárias, em vez de deixá-los guiar-se pela curvatura e pela oclusão, resulta em efeitos que contradizem a lógica física de onde ocorreriam. Desgaste em superfícies planas internas, sujeira em quinas salientes — quando deveria ser o oposto — denuncia que a pintura não seguiu a geometria.

Frequente é também a confusão entre as abordagens, usando o procedural onde se precisava de controle manual narrativo, ou pintando à mão o que um padrão procedural resolveria com mais consistência. Persiste o erro de não calibrar — definir valores apenas "no olho", extrapolando as faixas físicas e produzindo materiais que se comportam mal sob luz variada — e o erro complementar de calibrar valores perfeitos sobre um *asset* com densidade de texels insuficiente, desperdiçando o esforço em uma superfície borrada.

> ❌ **Erro Comum**
> Diante de uma aparência errada, ajustar muitos parâmetros ao acaso em vez de isolar os mapas e localizar metodicamente o responsável abandona a disciplina diagnóstica que torna a correção rápida e segura. Se o brilho está errado, verifique primeiro o mapa de rugosidade e o metálico; se a forma parece chapada, verifique o mapa de normais. Mexer em tudo de uma vez sem diagnóstico cria novos problemas enquanto tenta resolver o original.

## Resumo

Este capítulo passou da descrição dos mapas à prática de construir e analisar materiais reais. Vimos que a construção começa na observação — na leitura física da superfície de referência, decomposta nas propriedades que cada mapa representa —, e que essa leitura é a habilidade mais subestimada e mais decisiva do texturizador. Estudamos o *baking* como a materialização do trabalho de mapeamento da Parte II: o processo que transfere o detalhe da malha de alta para a de baixa densidade, gerando o mapa de normais e os mapas derivados — oclusão, curvatura, identificação — cujas qualidades dependem do desdobramento UV, da densidade e do alinhamento preparados anteriormente. Apresentamos o pensamento por camadas, no qual o material se constrói como um acúmulo de histórias físicas, cada camada afetando vários mapas e guiada pelas máscaras procedurais que os mapas derivados fornecem. Distinguimos as abordagens procedural e manual como polos complementares, a serem combinados segundo a natureza da superfície. Tratamos da calibração contra valores de referência da indústria como etapa real possibilitada pela base física do PBR, e de sua dependência da densidade de texels para se realizar. E desenvolvemos a competência inversa, a análise: decompor um material pronto para entender como foi feito e para depurar metodicamente os problemas, isolando o mapa responsável em vez de ajustar valores ao acaso. Construir e analisar revelaram-se duas faces de uma mesma competência, que costura todos os conceitos das partes anteriores numa superfície convincente.

## Exercícios

**1.** Descreva o método de leitura física de uma superfície. Escolha um objeto à sua volta (uma caneca, uma chave, um sapato gasto) e decomponha-o por escrito nas propriedades que cada mapa PBR representaria, indicando onde e por que cada propriedade varia ao longo da superfície.

**2.** Explique o que é o *baking* e por que ele depende diretamente do trabalho de mapeamento UV da Parte II. Liste os mapas que o *baking* tipicamente gera, distinguindo os que vão para o motor de jogo dos que servem como insumos para a autoria, e descreva dois artefatos clássicos de *baking* e suas causas em termos de desdobramento UV ou alinhamento.

**3.** Análise: explique o pensamento por camadas e como os mapas de curvatura e de oclusão de ambiente guiam as máscaras de desgaste e sujeira. Por que posicionar esses efeitos pela geometria, em vez de pintá-los arbitrariamente, produz resultados mais convincentes? Relacione sua resposta à lógica física de onde o desgaste e a sujeira de fato ocorrem.

**4.** Compare as abordagens procedural e manual quanto à flexibilidade, à reutilização, ao controle artístico e à captura de irregularidade orgânica. Para cada um dos seguintes casos, indique qual abordagem (ou combinação) você adotaria e por quê: (a) o revestimento de pedra de um castelo extenso; (b) o rosto de um personagem principal; (c) um arranhão narrativo específico na tampa de um baú.

**5.** Diagnóstico: um material de metal "não parece certo" — o brilho está chapado e a forma parece plana, embora a malha de alta densidade tivesse muito detalhe. Descreva o método de análise que você usaria para localizar o problema, indicando que mapas suspeitaria em cada sintoma e em que ordem os verificaria. Por que essa disciplina diagnóstica é preferível a ajustar valores ao acaso?

## Glossário

***Baking*:** Processo de transferir o detalhe geométrico de uma malha de alta densidade para mapas de textura aplicáveis a uma malha de baixa densidade. O principal produto é o mapa de normais; também gera mapas de oclusão de ambiente, curvatura e identificação de partes.

**Calibração:** Etapa da construção de materiais PBR em que os valores escolhidos para cada mapa são confrontados com tabelas de referência da indústria, garantindo que a cor base, a reflectância e demais propriedades correspondam a valores fisicamente plausíveis.

**Curvatura (mapa de):** Mapa derivado do *baking* que identifica quinas salientes (branco) e cavidades (preto) na geometria. Usado como máscara para guiar o desgaste — que se concentra nas protuberâncias — e a sujeira — que se acumula nas reentrâncias.

**Densidade de texels:** Como definido no Capítulo 6, é a relação entre o tamanho físico de uma superfície e os pixels de textura que a recobrem. Determina se os valores calibrados terão resolução suficiente para se manifestar visualmente.

**Identificação de partes (mapa de ID):** Mapa derivado do *baking* que rotula regiões do modelo com cores distintas, permitindo aplicar materiais diferentes a partes específicas com precisão e sem pintura manual de bordas.

**Mapa de normais:** Como definido no Capítulo 9, é o principal produto do *baking*, que transfere o detalhe de direção de superfície da malha de alta para a de baixa densidade.

**Material procedural:** Material construído a partir de regras matemáticas e parâmetros, sem dependência de resolução, facilmente variável e reutilizável. Ferramentas como o Substance Designer são especializadas em autoria procedural.

**Pensamento por camadas:** Modo de organizar a autoria de materiais em que cada contribuição física à superfície — base, desgaste, sujeira, marcas — vive em sua própria camada, com máscara própria, afetando simultaneamente todos os mapas que a alteração física real afetaria.

## Leituras Complementares

- **[What Is Texture Baking? e Baking Guide — materiais de apoio da disciplina]** — Descrevem em detalhe o processo de *baking*, os mapas gerados a partir da malha de alta densidade e os artefatos típicos e suas causas. Base direta da seção sobre *baking* deste capítulo e preparação essencial para a Parte V.

- **[The PBR Guide — Allegorithmic / Adobe, versão 2018 (material de apoio da disciplina)]** — Fornece as faixas de valores de referência para calibração de cor base, metais e reflectância, e discute as ferramentas de autoria PBR. Referência para a seção de calibração.

- **[Blender Maps e Blender Normals — materiais de apoio da disciplina]** — Situam a geração de mapas e o trabalho com normais em um fluxo concreto, úteis para conectar o *baking* à prática em uma ferramenta acessível.

- **[AOD — Texel Density (material de apoio da disciplina)]** — Fundamenta a relação, retomada na seção de calibração, entre densidade de texels e a manifestação dos valores autorados. Leitura complementar direta ao Capítulo 6.

- **[Adobe Substance 3D — documentação (helpx.adobe.com/substance-3d.html)]** — Referência sobre autoria por camadas, *smart materials*, máscaras procedurais guiadas por curvatura e AO, e construção procedural de materiais no Substance Designer. Ferramenta central do fluxo descrito neste capítulo.

- **[The Book of Shaders (thebookofshaders.com)]** — Introdução aos padrões e ruídos procedurais que fundamentam a geração algorítmica de texturas, útil para compreender a abordagem procedural por dentro.

- **[Real-Time Rendering — AKENINE-MÖLLER, HAINES, HOFFMAN; CRC Press, 2018]** — Base teórica sobre a relação entre detalhe geométrico, mapas e aparência, que fundamenta a lógica do *baking* e do pensamento por camadas.
