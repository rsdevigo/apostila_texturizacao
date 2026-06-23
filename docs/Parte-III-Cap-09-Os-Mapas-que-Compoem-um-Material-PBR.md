# Capítulo 9 — Os Mapas que Compõem um Material PBR

## Introdução

O capítulo anterior estabeleceu o princípio que governa toda a Parte III: no PBR, a textura descreve as propriedades intrínsecas da superfície e o motor de jogo calcula a interação com a luz. Mas dissemos pouco sobre *como*, concretamente, essas propriedades chegam ao motor. A resposta é: por meio de **mapas** — as imagens que, desde a Parte I, sabemos serem as entradas do cálculo de aparência. Cada propriedade física relevante de uma superfície é representada por um mapa próprio, e o conjunto desses mapas, associado a um mesmo modelo por meio das coordenadas UV preparadas na Parte II, constitui o **material**. Este capítulo apresenta, um a um, os mapas que compõem um material PBR: o que cada um descreve, que valores carrega, como é interpretado e como se relaciona com os princípios físicos do Capítulo 8.

É importante, desde o início, fixar uma distinção que atravessará todo o capítulo e que já anunciamos na Parte I: a diferença entre mapas que carregam **cor** e mapas que carregam **dados**. Um mapa de cor representa algo que o olho deve ver como cor — a cor base de um material, por exemplo — e por isso é codificado e interpretado no espaço de cor sRGB, ajustado à percepção humana. Um mapa de dados representa um *número* que o motor usará num cálculo — uma rugosidade, uma altura, uma direção — e por isso deve ser interpretado linearmente, sem o ajuste perceptual do sRGB. Tratar um pelo outro, como vimos, corrompe silenciosamente o resultado. Manter essa distinção em mente, mapa a mapa, é uma das principais competências que este capítulo pretende desenvolver. Trataremos os mapas na ordem do fluxo de trabalho metálico/rugosidade, o mais difundido na indústria, e ao final compararemos esse fluxo com a alternativa especular/suavidade, para que o estudante reconheça os dois quando os encontrar.

## Desenvolvimento

### A cor base (*base color* ou *albedo*)

O primeiro e mais intuitivo dos mapas é a **cor base**, também chamada de **albedo**. Ele descreve a cor própria da superfície — aquela que, conforme o Capítulo 8, corresponde à reflexão difusa dos dielétricos e à cor do reflexo dos metais. É o mapa que mais se parece com a noção ingênua de "a textura colorida do objeto", mas, no PBR, ele tem uma exigência rigorosa: deve conter *apenas* a cor intrínseca da superfície, sem qualquer luz, sombra, realce ou reflexo. Toda a iluminação é responsabilidade do motor de jogo; o albedo é a aparência da superfície sob luz branca, plana e neutra.

> ⚠️ **Atenção**
> Pintar sombras, realces ou reflexos dentro da cor base (*albedo*) é o erro fundamental do paradigma antigo de texturização. No PBR, a iluminação é calculada pelo sombreador em tempo real — incluir luz "pintada" na textura faz com que a superfície pareça iluminada de modo fixo, independentemente da luz da cena, produzindo um resultado visivelmente artificial.

A cor base é um mapa de *cor*, interpretado em espaço sRGB. Ela tem ainda uma sutileza que decorre diretamente da distinção entre condutores e dielétricos: o significado do albedo *muda* conforme o material seja metal ou não. Para um dielétrico, o albedo é a cor difusa de corpo. Para um metal, porém — que não tem componente difusa —, o albedo passa a representar a cor da *reflexão especular metálica*: é no albedo que se coloca o tom dourado do ouro ou o avermelhado do cobre. O motor de jogo sabe qual interpretação aplicar consultando o mapa metálico, que estudaremos a seguir. Essa dupla função da cor base é uma das fontes de confusão para iniciantes e merece atenção: o mesmo mapa significa "cor difusa" em regiões dielétricas e "cor do reflexo" em regiões metálicas, e a fronteira entre as duas é definida em outro mapa.

> 📘 **Definição**
> **Albedo** (ou *base color*) é o mapa que descreve a cor intrínseca de uma superfície, sem qualquer informação de iluminação. Em regiões dielétricas, representa a cor difusa; em regiões metálicas, representa a cor da reflexão especular. É um mapa de cor, interpretado em espaço sRGB.

A indústria estabeleceu, ainda, faixas de valores de referência para o albedo — nenhuma superfície natural é perfeitamente preta nem perfeitamente branca —, e respeitá-las é parte de autorar materiais fisicamente corretos.

### O mapa metálico (*metallic*)

O **mapa metálico** é a expressão direta da distinção entre condutores e dielétricos apresentada no Capítulo 8. Ele responde, para cada ponto da superfície, a uma pergunta essencialmente binária: este ponto é metal ou não? O preto (valor zero) indica não metal, o branco (valor um) indica metal, e o motor de jogo usa essa informação para decidir como interpretar os demais mapas — sobretudo, como vimos, o significado da cor base e o comportamento da reflexão especular. Por descrever um número que o motor consome, e não uma cor que o olho vê, o mapa metálico é um mapa de *dados*, interpretado linearmente.

Embora conceitualmente binário, o mapa metálico admite, na prática, valores intermediários, usados sobretudo nas *fronteiras* entre regiões metálicas e dielétricas — a transição entre o metal exposto e a tinta que o recobre, por exemplo — e para representar estados mistos, como a corrosão ou a sujeira que cobrem parcialmente um metal. Ainda assim, a boa autoria evita o cinza médio como valor de superfície "real": uma superfície ou é metal ou não é, e os tons intermediários servem a transições, não a um "meio-metal" que não existe fisicamente.

> ❌ **Erro Comum**
> Usar cinza médio (valor 0,5) no mapa metálico como se representasse um material "meio-metálico" é fisicamente incorreto. Metais existem ou não existem — cinzas intermediários são válidos apenas nas transições de borda entre metal e dielétrico. Um mapa metálico poluído de cinzas produz reflexos que não correspondem a nenhum material real.

O mapa metálico é, em certo sentido, o mais conceitual do conjunto: ele não tem aparência própria que se reconheça à primeira vista, mas é ele que organiza o significado de todo o material, e um erro nele — marcar como metal uma região dielétrica, ou vice-versa — produz resultados visualmente muito errados, com superfícies que ou ganham reflexos metálicos indevidos ou perdem o reflexo que deveriam ter.

> **Figura 9.1** — Um objeto composto de partes metálicas e não metálicas — por exemplo, um machado com cabeça de aço e cabo de madeira — exibido em três painéis: a cor base, o mapa metálico (branco na cabeça, preto no cabo) e o resultado final renderizado, evidenciando como o mapa metálico decide onde há reflexo metálico.

**Screenshot sugerido:** Visualização dos canais separados de um material misto em uma ferramenta de autoria (Substance Painter ou similar), com o mapa metálico isolado mostrando a clara separação preto/branco entre cabo e lâmina.

### O mapa de rugosidade (*roughness*)

O **mapa de rugosidade** traduz em dados o conceito de microsuperfície do Capítulo 8. Para cada ponto, ele informa quão lisa ou rugosa é a superfície em escala microscópica e, portanto, quão concentrados ou dispersos serão seus reflexos. Convencionalmente, o preto representa uma superfície lisa, com reflexos nítidos e concentrados, e o branco uma superfície rugosa, com reflexos amplos e difusos — embora alguns sistemas invertam essa convenção sob o nome de mapa de **suavidade** (*glossiness*), em que o branco é liso. É um mapa de *dados*, interpretado linearmente.

> 📘 **Definição**
> **Mapa de rugosidade** (*roughness map*) é o mapa de dados que descreve a irregularidade microscópica de uma superfície. Valores baixos (próximos ao preto) indicam superfície lisa e reflexos concentrados e nítidos; valores altos (próximos ao branco) indicam superfície rugosa e reflexos amplos e difusos.

De todos os mapas PBR, a rugosidade é talvez o mais expressivo e o que mais distingue um material rico de um material chapado. É nela que se conta a *história* da superfície: as marcas de dedos numa maçaneta polida, que a tornam localmente menos lisa; os arranhões num metal, que riscam de fosco uma superfície brilhante; as poças e respingos que deixam regiões momentaneamente espelhadas sobre um chão seco; o desgaste que pule o que era áspero. Duas superfícies com a mesma cor base e o mesmo caráter metálico podem parecer materiais inteiramente distintos apenas pela rugosidade — um plástico novo e um plástico envelhecido, um aço polido e um aço escovado.

Marek Madej documentou em entrevistas sobre *The Witcher 3: Wild Hunt* (CD Projekt RED, 2015) como o contraste de rugosidade foi o principal veículo narrativo dos personagens: a pele curtida e envelhecida de Geralt tem rugosidade alta e irregular, acumulada em décadas de cicatrizes e intempérie, enquanto a pele jovem de Ciri tem rugosidade baixa e mais uniforme — diferença expressa inteiramente no mapa de rugosidade, sem alterar a cor de base de nenhum dos dois. Por isso, dedicar atenção ao mapa de rugosidade é, muitas vezes, o que separa um material amador de um profissional: a variação de rugosidade ao longo da superfície é o que a faz parecer ter sido *usada*, *tocada* e *existido no mundo*, em vez de recém-saída de um gerador uniforme.

> 💡 **Dica Profissional**
> Um mapa de rugosidade uniforme — valor constante em toda a superfície — é a marca imediata de um material amador. Superfícies reais acumulam história: quinas desgastadas ficam mais lisas, reentrâncias acumulam sujeira e ficam mais rugosas, pontos de uso frequente brilham. Variar a rugosidade sistematicamente conforme a forma e o uso do objeto é a diferença entre um material "correto" e um material convincente.

### O mapa de normais (*normal map*)

O **mapa de normais** já nos é familiar desde a Parte I, onde o apresentamos como o recurso que faz uma superfície geometricamente simples parecer detalhada. Convém agora reposicioná-lo no contexto PBR e articulá-lo com o que aprendemos na Parte II. O mapa de normais descreve, para cada ponto, a direção para a qual a microgeometria da superfície aponta — codificando, nos canais de cor da imagem, um vetor de orientação. Com base nessa direção, o motor de jogo calcula a iluminação como se a superfície tivesse o relevo indicado, mesmo que a malha real seja lisa. É assim que rebites, dobras, poros e ranhuras aparecem iluminados de modo convincente sobre uma malha de baixa densidade. O mapa de normais é um mapa de *dados* — ele carrega vetores, não cores —, e por isso, apesar de seu aspecto azulado característico, jamais deve ser interpretado em sRGB.

> ⚠️ **Atenção**
> Importar o mapa de normais como sRGB — o espaço de cor padrão para imagens — é um dos erros mais comuns e mais difíceis de diagnosticar à primeira vista. O mapa de normais carrega vetores, não cores, e deve ser configurado como linear (ou como "normal map") no motor de jogo. A importação errada produz iluminação incorreta e costuras visíveis, mesmo que o mapa em si esteja perfeito.

É aqui que se completa o arco que vinha sendo construído desde a Parte II. Lembremos que o Capítulo 7 preparou a relação entre a malha de alta densidade — rica em detalhe geométrico — e a de baixa densidade, otimizada para o tempo real, e antecipou o processo de *baking* como o elo entre as duas. O mapa de normais é o principal produto desse *baking*: o detalhe geométrico da malha de alta densidade é "fotografado" como direções de superfície e gravado num mapa que se aplica à malha de baixa densidade. Tudo o que aprendemos sobre desdobramento UV, densidade de texels, *padding* e alinhamento de grupos de suavização com as costuras converge, agora, na qualidade desse mapa: um desdobramento ruim ou uma densidade insuficiente degradam diretamente o detalhe que o mapa de normais consegue carregar. O Capítulo 10 mostrará o *baking* em ação na construção de materiais, e a Parte V (Capítulos 15 e 16) o desenvolverá por inteiro; por ora, basta reconhecer o mapa de normais como o membro do conjunto PBR que traz o detalhe da forma sem o custo da geometria.

### Oclusão de ambiente, altura e os mapas de detalhe

Além dos quatro mapas centrais — cor base, metálico, rugosidade e normais —, um material PBR completo costuma incluir mapas auxiliares que refinam sua aparência. O **mapa de oclusão de ambiente** (*ambient occlusion*, ou AO) descreve o quanto cada ponto da superfície está exposto ou abrigado da luz ambiente difusa: fendas, cantos e cavidades recebem menos luz indireta e aparecem naturalmente mais escuros. O AO é tipicamente gerado no mesmo *baking* que produz o mapa de normais, capturando, a partir da malha de alta densidade, o sombreamento de contato que a geometria fina provocaria.

O remake de *Shadow of the Colossus* (Bluepoint Games, 2018) é um caso documentado de como o rebake de AO transforma a leitura visual de *assets* legados: a equipe rebakeou os mapas de oclusão dos colossus da versão PS2 para o novo pipeline PBR do PS4, e as dobras e juntas de cada criatura passaram a mergulhar em sombra de modo convincente, transformando formas que antes pareciam chapadas em volumes com peso e profundidade reais. É um mapa de *dados*, e seu papel é acentuar a sensação de profundidade e de assentamento das formas.

> 💡 **Dica Profissional**
> O mapa de oclusão de ambiente deve ser dosado com parcimônia. O motor de jogo já calcula sombras de contato e iluminação indireta; escurecer demais as reentrâncias no AO duplica, de modo incorreto, o que o sistema já faz, produzindo superfícies excessivamente escuras que não respondem corretamente à iluminação dinâmica da cena.

O **mapa de altura** ou de **deslocamento** (*height/displacement*) descreve o relevo da superfície como uma elevação em cada ponto — branco para o mais alto, preto para o mais baixo. Ele se distingue do mapa de normais por descrever a *altura* em vez da *direção*: enquanto o mapa de normais altera apenas a iluminação simulando relevo, o mapa de altura pode, em certas técnicas, deslocar de fato a superfície ou criar efeitos de paralaxe que dão profundidade real ao olhar. *Steep* (Ubisoft Annecy, 2016) utilizou *parallax occlusion mapping* em neve compactada e fissuras de rocha para criar a ilusão de profundidade sem geometria adicional, documentado nos posts técnicos da Ubisoft sobre o motor Snowdrop. Os dois são complementares e frequentemente derivam da mesma malha de alta densidade.

Há ainda o **mapa emissivo** (*emissive*), que indica regiões que *emitem* luz própria — telas, lâmpadas, lava, sinais luminosos — e que, por descrever cor luminosa, é um mapa de cor; e o **mapa de opacidade** (*opacity/alpha*), que define onde a superfície é transparente ou recortada, essencial para folhagens, grades e tecidos rendados. Nem todo material usa todos esses mapas: um plástico liso pode dispensar altura e AO, enquanto uma rocha intrincada se beneficia de todos. Saber *quais* mapas um material exige é parte do julgamento que o Capítulo 10 desenvolverá.

> **Figura 9.2** — Painel com os mapas de um único material complexo (por exemplo, uma placa de metal pintada e enferrujada) dispostos lado a lado e rotulados — cor base, metálico, rugosidade, normais, oclusão de ambiente e altura —, seguidos do resultado final que emerge de sua combinação, evidenciando a contribuição de cada um.

**Screenshot sugerido:** Captura de uma ferramenta de autoria mostrando a lista de canais de um material PBR completo, com cada mapa visível em miniatura e o material final renderizado sobre uma esfera ou plano de teste.

### O canal alfa, o empacotamento e a economia de mapas

Vale conhecer, ainda na formação inicial, uma prática de produção que conecta este capítulo à preocupação com desempenho que atravessa toda a apostila: o **empacotamento de canais** (*channel packing*). Como vários mapas PBR — metálico, rugosidade, oclusão de ambiente — carregam um único valor por ponto (são imagens em tons de cinza), é comum agrupá-los nos três ou quatro canais de uma única imagem colorida: o vermelho carrega a oclusão de ambiente, o verde a rugosidade, o azul o metálico, por exemplo. Em vez de três texturas, o material usa uma só, economizando memória e acessos. Diferentes motores de jogo e estúdios adotam convenções próprias de empacotamento — a ordem dos canais varia —, e parte da preparação de um material para um motor específico é organizar os mapas segundo a convenção esperada. Reencontramos aqui o princípio econômico da Parte II: extrair o máximo de informação do mínimo de memória, agora aplicado à própria organização interna dos mapas.

> 📘 **Definição**
> **Empacotamento de canais** (*channel packing*) é a técnica de combinar múltiplos mapas de dados de canal único (metálico, rugosidade, oclusão de ambiente) nos canais R, G e B de uma única imagem, reduzindo o número de texturas no material e economizando memória de GPU.

### Os dois fluxos de trabalho: metálico/rugosidade e especular/suavidade

Apresentamos os mapas na ordem do fluxo de trabalho **metálico/rugosidade**, o mais usado em jogos, no qual o caráter especular da superfície é *deduzido* pelo motor a partir do mapa metálico e da cor base. Existe, porém, um fluxo alternativo, o **especular/suavidade** (*specular/glossiness*), que descreve a reflexão especular *diretamente*: em vez de um mapa metálico, ele usa um mapa **especular** que define explicitamente a cor e a intensidade do reflexo, e um mapa de **albedo** que contém apenas a cor difusa. Onde o fluxo metálico diz "esta região é metal, deduza seu reflexo", o fluxo especular diz "o reflexo desta região tem esta cor e esta intensidade".

Ambos descrevem a mesma física e podem produzir resultados equivalentes; a diferença é de *controle* e de *conveniência*. O fluxo metálico/rugosidade é mais compacto — menos mapas, menos memória — e mais à prova de erros, pois impede, por construção, valores de reflectância fisicamente impossíveis; em contrapartida, dá menos controle direto sobre o reflexo dos dielétricos e pode produzir, nas fronteiras entre metal e não metal, um pequeno artefato de borda branca. O fluxo especular/suavidade dá controle explícito sobre a cor do reflexo e evita esse artefato de borda, mas usa mais mapas, ocupa mais memória e abre espaço para erros físicos quando o artista define valores de reflectância irreais. A indústria de jogos, pesando esses fatores, consagrou majoritariamente o fluxo metálico/rugosidade, e é nele que concentraremos os capítulos seguintes; mas o estudante encontrará materiais e bibliotecas no outro fluxo e deve reconhecer que se trata de dois caminhos para descrever a mesma superfície, e não de duas físicas diferentes.

## Aplicação em Jogos

Na produção, o conjunto de mapas de um material é tratado como uma unidade — frequentemente chamada de *texture set* —, gerada de forma integrada nas ferramentas de autoria e exportada para o motor de jogo segundo as convenções deste. Ferramentas como o Substance Painter permitem pintar simultaneamente sobre todos os canais: ao pintar ferrugem numa região, o artista altera de uma vez a cor base (para o tom alaranjado), a rugosidade (para mais áspero) e o metálico (para dielétrico, pois a ferrugem não é metal), refletindo a forma como uma alteração física real afeta várias propriedades ao mesmo tempo. Esse trabalho integrado é uma das grandes vantagens das ferramentas PBR modernas e uma razão de sua adoção: pensar em termos de *materiais reais* — "aqui há ferrugem", "aqui o metal foi polido pelo uso" — em vez de canais isolados.

A escolha de *quais* mapas incluir e em que resolução é uma decisão de produção que reencontra o equilíbrio entre fidelidade e desempenho. Um *asset* de primeiro plano, visto de perto, justifica o conjunto completo de mapas em alta resolução; um *asset* de fundo distante pode prescindir de altura, de AO e até de variação fina de rugosidade, economizando memória. O empacotamento de canais, as convenções de nomenclatura herdadas da Parte II e a correspondência entre o nome do modelo e o de suas texturas tornam-se, nesse contexto, parte do que faz um conjunto de mapas utilizável por uma equipe. E a distinção entre mapas de cor e de dados, longe de ser um detalhe acadêmico, é configurada explicitamente na importação de cada mapa no motor de jogo: marcar corretamente quais texturas são sRGB e quais são lineares é uma etapa concreta e obrigatória do pipeline, cujo descuido produz materiais sutilmente — ou grosseiramente — errados.

## Estudo de Caso

### Destiny e o Sistema de Mapas PBR das Armaduras de Guardião — Bungie, 2014

> **Figura 9.3** — Decomposição do material de armadura de Guardião Titan de *Destiny* em canais individuais: Base Color (cor pura, sem luz), Metallic (condutor em branco, dielétrico em preto), Roughness (história de uso e desgaste), Normal (relevo de costuras e parafusos), AO (sombra de contato nas reentrâncias), Tint Mask (regiões tingíveis pelos três canais RGB). A imagem final à direita é o material completo sob iluminação solar.

**Screenshot sugerido:** Captura de tela oficial de *Destiny*, Bungie/Activision — disponível em `https://www.bungie.net/en/Explore/Detail/News/14597` (Bungie blog) ou via press kit em `https://www.activision.com/games/destiny/destiny`.

Em 2013, quando *Destiny* ainda estava em desenvolvimento, Paul Pepera (Bungie) apresentou na SIGGRAPH *"The Art of Destiny"* — a primeira demonstração detalhada de um sistema de mapas PBR para um jogo de tiro em desenvolvimento. A apresentação é notável não pela teoria, mas pela clareza com que decompõe o material de uma armadura de Guardião canal por canal, explicando em termos precisos o que cada mapa armazena e por que está ali. O exemplo escolhido — o peito de um Titan com placas de metal polido, partes de tecido tático e ornamentos pintados — é praticamente idêntico aos materiais mistos que os capítulos desta apostila propõem como exercícios.

O sistema de Bungie ia além da separação padrão de cor base, metálico, rugosidade e normais. A equipe adicionou dois mecanismos que tornaram as armaduras de *Destiny* únicas: um **mapa de paleta** (*tint map*), com canais RGB codificando regiões tingíveis da armadura independentemente, e um **mapa de emissão** para os elementos energizados das armaduras do futuro. O canal de paleta resolvia um problema econômico elegante: com centenas de armaduras a variar por facção, raridade e personalização, criar texturas completas para cada combinação de cor era inviável. Em vez disso, o mapa de paleta marcava as regiões tingíveis com valores de índice, e o motor de jogo aplicava as cores da facção ou da customização do jogador em tempo de render — um único conjunto de mapas servia a todas as variações de cor da armadura, com a personalização do jogador nunca exigindo um novo *asset*.

O que a apresentação de Pepera torna especialmente didático é a demonstração de cada canal isolado. Vendo o **mapa metálico** sozinho, o estudante observa exatamente quais regiões são condutores (branco) e quais são dielétricos (preto) — as placas de metal são brancas, as correias de couro são pretas, os tecidos são pretos, os ornamentos pintados são pretos mesmo sendo dourados, porque dourado pintado é tinta, não metal. Vendo o **mapa de rugosidade**, observa a história de uso: a placa de ombro polida tem valor baixo nas faces e valor alto nas arestas gastas; o couro tem rugosidade média e uniforme; as costuras têm valor alto. Nenhum desses valores precisa ser visto para ser compreendido — mas vê-los separados, num material amplamente conhecido, é a maneira mais eficiente de internalizar a função de cada canal.

**O que aprender com isso:** O sistema de mapas PBR de *Destiny* demonstra que o conjunto de mapas de um material é uma *arquitetura*, não uma lista. Cada canal tem uma função precisa, e adicionar mapas além do padrão — como o mapa de paleta — resolve problemas de produção reais (escala de variações) de modo elegante e econômico. A chave é entender *o que* cada mapa representa antes de decidir *quais* usar.

## Boas Práticas

A boa prática que sintetiza este capítulo é distribuir cada propriedade física para o mapa que a representa, em vez de concentrar a aparência num único mapa. A cor base deve conter apenas a cor intrínseca, sem luz; o caráter metálico vai para o mapa metálico, mantido próximo do binário; a história da microsuperfície vai para a rugosidade; o detalhe de forma vai para os normais e, quando útil, para a altura; o sombreamento de contato vai para o AO. Ao pintar uma alteração física — ferrugem, desgaste, sujeira, umidade —, o artista deve perguntar-se quais propriedades ela altera e atualizar *todos* os mapas afetados, pois uma alteração real raramente toca um só canal.

É boa prática tratar com rigor a distinção entre mapas de cor e de dados, configurando corretamente o espaço de cor de cada um na importação: apenas a cor base e os mapas emissivos são sRGB; metálico, rugosidade, mapa de normais, AO, altura e opacidade são lineares. Recomenda-se respeitar as faixas de valores de referência da indústria, especialmente para a cor base — evitando pretos e brancos absolutos — e para o mapa metálico — evitando o cinza médio como valor de superfície real. Convém ainda dosar o AO, lembrando que o motor de jogo já calcula sombras e que escurecer demais à mão duplica, de modo incorreto, o que o sistema faz. E vale conhecer, desde já, as convenções de empacotamento de canais e de nomenclatura do motor de destino, herança direta da disciplina de produção da Parte II, que tornam o conjunto de mapas utilizável pela equipe e pelo pipeline.

## Erros Comuns

> ❌ **Erro Comum**
> Colocar luz, sombra ou reflexo na cor base — erro herdado do paradigma tradicional e já denunciado no Capítulo 8 — é o problema mais persistente. A cor base deve conter apenas a cor intrínseca da superfície. A iluminação "pintada" entra em conflito com o cálculo do motor de jogo e produz aparência achatada ou duplamente iluminada.

> ❌ **Erro Comum**
> Pintar ferrugem apenas na cor base, esquecendo de alterar o mapa metálico (para dielétrico) e o mapa de rugosidade (para mais áspero), produz uma ferrugem que tem a cor certa mas o brilho errado — ela reflete como metal polido em vez de óxido fosco. Uma alteração física sempre afeta múltiplos mapas simultaneamente.

Há também o erro recorrente de espaço de cor: importar o mapa de normais, de rugosidade ou metálico como sRGB, ou a cor base como linear, corrompendo o cálculo da luz — um erro tão comum quanto invisível à primeira vista. Outro equívoco frequente é usar valores extremos ou fisicamente implausíveis: pretos e brancos absolutos na cor base, que não existem em superfícies reais; cinza médio no mapa metálico, criando um "meio-metal" inexistente; ou rugosidade uniforme, que deixa a superfície chapada e sem história. Por fim, persiste o erro de exagerar no AO, escurecendo à mão cavidades que o motor já sombreia, e o de ignorar as convenções de empacotamento e nomenclatura do motor de destino, gerando um conjunto de mapas que funciona na ferramenta de autoria mas se desconfigura ao chegar ao jogo.

## Resumo

Este capítulo apresentou os mapas que, em conjunto, constituem um material PBR, traduzindo em dados concretos os princípios físicos do Capítulo 8. Reafirmamos a distinção fundamental entre mapas de cor — interpretados em sRGB — e mapas de dados — interpretados linearmente, e a aplicamos a cada mapa. Estudamos a cor base, que descreve a cor intrínseca sem qualquer luz e cujo significado se desdobra entre cor difusa, nos dielétricos, e cor de reflexo, nos metais. Vimos o mapa metálico como a expressão direta da distinção entre condutores e dielétricos, essencialmente binário e organizador do significado de todo o material. Detalhamos o mapa de rugosidade como o mais expressivo do conjunto, aquele que conta a história da microsuperfície e distingue materiais usados de materiais chapados. Reposicionamos o mapa de normais no contexto PBR, fechando o arco iniciado na Parte II ao reconhecê-lo como o principal produto do *baking* que transfere o detalhe da malha de alta para a de baixa densidade. Apresentamos os mapas auxiliares — oclusão de ambiente, altura, emissivo, opacidade — e o critério de incluir apenas os que cada material exige. Conhecemos o empacotamento de canais como prática de economia de memória e, por fim, comparamos os dois fluxos de trabalho, metálico/rugosidade e especular/suavidade, reconhecendo-os como dois caminhos para descrever a mesma física, com o primeiro consagrado nos jogos. Com o vocabulário dos mapas estabelecido, o próximo capítulo passa da descrição à prática: como construir e analisar materiais reais a partir desses ingredientes.

## Exercícios

**1.** Explique a distinção entre mapas de cor e mapas de dados e classifique, justificando, cada um dos seguintes mapas: cor base, metálico, rugosidade, normais, oclusão de ambiente, altura, emissivo e opacidade. Que erro concreto ocorre ao importar um mapa de dados como se fosse sRGB?

**2.** A cor base tem um significado que muda conforme o material seja metálico ou dielétrico. Explique essa dupla função e descreva como o motor de jogo sabe qual interpretação aplicar a cada ponto da superfície. Por que isso torna o mapa metálico o "organizador" do significado do material?

**3.** Análise: duas superfícies têm a mesma cor base e o mesmo mapa metálico, mas parecem materiais completamente diferentes — uma como plástico novo e brilhante, outra como plástico velho e fosco. Qual mapa explica a diferença e o que ele descreve fisicamente? Explique por que esse mapa é frequentemente apontado como o que mais distingue um material profissional de um amador.

**4.** Diagnóstico: um estudante pintou manchas de ferrugem alaranjada apenas na cor base de um material metálico, sem alterar os outros mapas. Descreva como a ferrugem aparecerá no resultado final e por que não convencerá. Liste todos os mapas que deveriam ter sido alterados e explique, para cada um, que mudança a ferrugem real exige.

**5.** Compare os fluxos de trabalho metálico/rugosidade e especular/suavidade quanto ao número de mapas, ao consumo de memória, ao controle sobre o reflexo, à propensão a erros físicos e aos artefatos característicos. Por que a indústria de jogos consagrou majoritariamente o primeiro? Em que situação o segundo poderia ser preferível?

## Glossário

**Albedo:** Canal de textura que armazena a cor base de uma superfície sem informação de iluminação. Em dielétricos, representa a cor difusa; em metais, representa a cor da reflexão especular.

**Empacotamento de canais:** Técnica de combinar múltiplos mapas de dados de canal único (metálico, rugosidade, oclusão de ambiente) nos canais R, G e B de uma única imagem, reduzindo o número de texturas e economizando memória.

**Espaço de cor sRGB:** Espaço de cor ajustado à percepção humana, usado para mapas que carregam informação visual (cor base, emissivo). Mapas de dados (rugosidade, metálico, normais) devem ser configurados como lineares, não sRGB.

**Mapa de altura:** Mapa de dados que descreve o relevo de uma superfície como elevações em tons de cinza — branco para o ponto mais alto, preto para o mais baixo. Pode ser usado para deslocamento geométrico real ou efeitos de paralaxe.

**Mapa de normais:** Mapa de dados que codifica, em seus canais de cor, vetores de orientação da microgeometria de uma superfície. Permite que o sombreador calcule iluminação como se a malha tivesse detalhe geométrico sem que a geometria real exista. Deve ser configurado como linear no motor de jogo.

**Mapa de oclusão de ambiente:** Mapa de dados que descreve o quanto cada ponto da superfície está exposto à luz ambiente difusa. Reentrâncias e cavidades recebem valores mais escuros. Gerado tipicamente no mesmo processo de *baking* que o mapa de normais.

**Mapa de rugosidade:** Mapa de dados que descreve a irregularidade microscópica de uma superfície. Valores baixos indicam superfície lisa (reflexos nítidos); valores altos indicam superfície rugosa (reflexos difusos).

**Mapa emissivo:** Mapa de cor que identifica regiões que emitem luz própria (telas, lâmpadas, lava). Por descrever cor luminosa, é interpretado em sRGB.

**Mapa metálico:** Mapa de dados essencialmente binário — preto para dielétrico, branco para condutor — que informa ao motor de jogo como interpretar os demais mapas de cada região da superfície.

**Material PBR:** Conjunto coordenado de mapas — cor base, metálico, rugosidade, normais e auxiliares — que descreve as propriedades físicas intrínsecas de uma superfície para ser calculado pelo sombreador segundo os princípios da renderização fisicamente baseada.

## Leituras Complementares

- **[The PBR Guide — Allegorithmic / Adobe, versão 2018 (material de apoio da disciplina)]** — Descreve em detalhe cada mapa dos fluxos metálico/rugosidade e especular/suavidade, com faixas de valores de referência para cor base e reflectância. Base direta deste capítulo e referência obrigatória para calibração de materiais.

- **[Blender Maps e Blender Normals — materiais de apoio da disciplina]** — Situam os mapas, em especial o mapa de normais, dentro de um fluxo de trabalho concreto, conectando-os ao desdobramento UV da Parte II. Úteis para observar como os conceitos se manifestam em uma ferramenta específica.

- **[What Is Texture Baking? e Baking Guide — materiais de apoio da disciplina]** — Explicam como os mapas de normais e de oclusão de ambiente são gerados a partir da malha de alta densidade, processo retomado no Capítulo 10 e desenvolvido por inteiro na Parte V.

- **[Real-Time Rendering, caps. de sombreamento e texturização — AKENINE-MÖLLER, HAINES, HOFFMAN; CRC Press, 2018]** — Fundamenta o significado de cada mapa e sua interpretação pelo motor de jogo. Indicado para aprofundar a base matemática e física dos conceitos.

- **[Adobe Substance 3D — documentação (helpx.adobe.com/substance-3d.html)]** — Referência sobre a autoria integrada de conjuntos de mapas e a exportação segundo convenções de motores de jogo. Complementa com perspectiva de ferramenta.

- **[Unreal Engine Documentation (dev.epicgames.com) e Unity Manual (docs.unity3d.com) — seções sobre materiais, importação de texturas e espaço de cor]** — Para observar como cada motor de jogo consome os mapas descritos neste capítulo e como configurar corretamente sRGB versus linear na importação.
