# Capítulo 7 — Preparação de Assets para Produção

## Introdução

Os três capítulos anteriores prepararam o modelo do ponto de vista do mapeamento: aprendemos a estabelecer as coordenadas UV, a desdobrar a superfície com costuras bem posicionadas e a organizar as ilhas com densidade de texels consistente. Um modelo assim preparado está geometricamente pronto para receber textura — mas ainda não está pronto para *produção*. Entre o arquivo que vive no software de modelagem e o *asset* que entra no motor de jogo, funciona dentro de uma equipe e roda no jogo final, existe uma série de cuidados que, embora menos glamorosos que o desdobramento UV, são igualmente decisivos. É deles que trata este capítulo, que encerra a Parte II e faz a ponte para a Parte III, dedicada à criação das texturas propriamente ditas.

A preparação de *assets* para produção é, em larga medida, um exercício de disciplina e previsão. Trata-se de antecipar tudo o que o modelo encontrará adiante — o motor de jogo que o importará, os colegas que trabalharão sobre ele, o processo de geração de mapas que o aguarda — e deixá-lo em condições de atravessar essas etapas sem atrito. Um modelo mal preparado pode ter um desdobramento impecável e, ainda assim, chegar ao motor de jogo com a escala errada, a origem deslocada, normais invertidas ou um nome que ninguém entende, gerando horas de retrabalho que uma boa preparação teria evitado. Este é, portanto, o capítulo em que o trabalho artístico se encontra com a engenharia de produção, e em que o estudante começa a pensar não apenas como criador de um modelo isolado, mas como integrante de um pipeline.

## Desenvolvimento

### A higiene da malha

Antes de qualquer exportação, o modelo precisa estar geometricamente saudável, e há um conjunto de verificações que constitui a **higiene da malha**. A primeira diz respeito às **normais** — não o mapa de normais da Parte I, mas a propriedade geométrica que indica para que lado cada face está voltada, isto é, qual é o seu "lado de fora". Faces com normais invertidas ficam, no motor de jogo, transparentes ou pretas, pois o motor as renderiza voltadas para o lado errado. Garantir que todas as normais apontem coerentemente para fora é uma das primeiras conferências da preparação.

Relacionada às normais está a questão do **sombreamento suave ou anguloso** (*smooth/flat shading*) e dos **grupos de suavização** (*smoothing groups*), que determinam onde a superfície parece arredondada e onde apresenta arestas vivas. Essa configuração não é cosmética: ela interage diretamente com o mapa de normais que será gerado na Parte III, e uma incoerência entre os grupos de suavização e as costuras UV é uma fonte clássica de artefatos. A boa prática, que se consolidará no estudo da geração de mapas, é alinhar as quebras de suavização com as costuras do desdobramento.

> 💡 **Dica Profissional**
> Alinhar as quebras de grupos de suavização com as costuras UV é uma das práticas mais importantes para evitar artefatos no mapa de normais. Sempre que uma costura divide a malha, deve haver ali também uma quebra de suavização — e vice-versa. Essa correspondência garante que a geração de mapas produzirá normais limpas e sem borda visível.

Convém ainda eliminar problemas geométricos comuns: faces com mais de quatro lados (*n-gons*), que podem se comportar de modo imprevisível em diferentes motores de jogo; vértices duplicados ou soltos; faces internas invisíveis que só consomem recursos; e geometria degenerada. Uma malha limpa exporta sem surpresas e gera mapas sem artefatos; uma malha suja propaga seus defeitos por todas as etapas seguintes.

> ❌ **Erro Comum**
> Exportar sem conferir a higiene da malha é o erro mais comum e mais custoso da preparação: normais invertidas produzem faces transparentes ou pretas no motor de jogo; *n-gons* podem gerar triangulação imprevisível em diferentes plataformas; grupos de suavização desalinhados das costuras produzirão artefatos no mapa de normais — e todos esses problemas só se manifestam depois que o *asset* já percorreu etapas subsequentes.

### Escala, unidades e a importância de um padrão comum

Um dos atritos mais frequentes — e mais facilmente evitáveis — entre modelagem e motor de jogo é a **escala**. Cada software e cada motor de jogo adota um sistema de unidades, e se o modelo é construído sem atenção a esse sistema, ele chega ao motor de jogo gigante, minúsculo ou simplesmente incoerente com os demais *assets*. As consequências vão além do incômodo visual: a escala afeta a física, a iluminação (que é calculada em unidades reais), a percepção de tamanho pelo jogador e, como vimos no capítulo anterior, a própria densidade de texels, que é medida em pixels por unidade métrica e, portanto, depende de o objeto ter o tamanho físico correto.

> ⚠️ **Atenção**
> Uma escala incorreta invalida diretamente a densidade de texels padronizada no Capítulo 6. Se o objeto entra no motor de jogo maior ou menor do que o previsto, a relação entre texels e unidades métricas se desfaz, e toda a calibração anterior perde o sentido.

A boa prática é modelar desde o início em **escala real**, tomando uma unidade do mundo como referência fixa — tipicamente um metro — e usando objetos de referência de tamanho conhecido, como uma silhueta humana, para calibrar as proporções. Estabelecer um padrão de unidades para todo o projeto e a ele aderir é uma decisão de pipeline tão importante quanto o padrão de densidade de texels, e pelas mesmas razões: coordena o trabalho de muitas pessoas em torno de uma régua comum.

### Origem, pivô e orientação

Todo *asset* tem um ponto de origem (*pivot*) em torno do qual ele é posicionado, rotacionado e escalado no motor de jogo, e a escolha desse ponto tem consequências práticas que o iniciante costuma descobrir tarde. Uma porta cujo pivô esteja no centro gira em torno do meio, como uma porta giratória, em vez de abrir em torno das dobradiças; uma árvore cujo pivô esteja acima do solo flutua quando posicionada no terreno. A regra geral é colocar o pivô onde o objeto naturalmente se apoia ou articula — a base, para objetos que assentam no chão; o eixo de rotação, para os que giram. A Bungie documentou em posts técnicos do desenvolvimento de *Destiny 2* (2017) que erros de pivô em módulos de nave e corredor — peças que precisavam girar e encaixar em procedimentos de instanciamento automático — causavam rotações incorretas que só apareciam no motor de jogo; a correção retroativa representou horas de retrabalho por *asset* e levou o estúdio a institucionalizar uma etapa obrigatória de conferência de pivô antes de qualquer importação.

Igualmente importante é **aplicar as transformações** antes de exportar, isto é, zerar as rotações e escalas acumuladas durante a modelagem, de modo que o motor de jogo receba o objeto com transformações limpas. Transformações não aplicadas são uma fonte notória de comportamento estranho no motor de jogo — objetos que escalam de forma assimétrica, física que se comporta mal, normais que se distorcem. Por fim, alinhar o objeto a uma orientação convencional (definir claramente qual é a frente, qual é o cima) facilita seu uso por quem o receber.

### Nomenclatura e organização

Um *asset* de produção não vive sozinho: ele convive com centenas ou milhares de outros arquivos, e a forma como é nomeado e organizado determina se a equipe consegue encontrá-lo, entendê-lo e mantê-lo. Uma **convenção de nomenclatura** clara e consistente — que identifique o tipo do objeto, sua função, sua variação e seus mapas associados — é o que permite que o pipeline funcione em escala. Nomes como "objeto_final_2_real_agora" são a marca de uma produção sem disciplina; nomes estruturados, que sigam um padrão acordado pela equipe, são o que torna possível automatizar importações, localizar dependências e dar manutenção meses depois.

Essa organização se estende à estrutura de pastas, à separação entre as malhas de diferentes níveis de detalhe e à correspondência entre o nome do modelo e o nome de suas texturas. Embora as convenções específicas variem de estúdio para estúdio, o princípio é universal e ecoa o que se disse sobre a legibilidade do layout UV no capítulo anterior: a preparação não serve apenas à máquina, mas à colaboração humana, e um *asset* bem nomeado é um *asset* que a equipe inteira consegue usar. A Epic Games publicou sua convenção de nomenclatura para projetos Unreal Engine — sufixos como `_D` para albedo, `_N` para normais e `_ORM` para o canal empacotado de oclusão/rugosidade/metálico —, e ela se tornou padrão de fato na indústria, adotada até por projetos fora da Unreal; é um dos raros casos em que a documentação técnica aberta de um motor de jogo criou convergência prática em toda a cadeia de ferramentas do mercado.

> 💡 **Dica Profissional**
> Definir a convenção de nomenclatura no início do projeto — antes de criar qualquer *asset* — e registrá-la num documento acessível à equipe toda é um investimento pequeno que poupa horas de busca e retrabalho ao longo de meses de produção. Renomear centenas de arquivos a meio caminho é sempre mais caro do que acertar desde o início.

### A relação entre malha de alta e de baixa densidade

A preparação de *assets* para jogos frequentemente envolve duas versões do mesmo modelo, e compreender sua relação é essencial para o que vem na Parte III. A **malha de alta densidade** (*high-poly*) é uma versão ricamente detalhada, com toda a riqueza de formas — rebites, dobras, desgaste, microrrelevo — que se deseja que o objeto aparente ter. Ela é, em geral, pesada demais para rodar em tempo real. A **malha de baixa densidade** (*low-poly*) é a versão otimizada, leve, que de fato entra no jogo, e que possui o desdobramento UV preparado nos capítulos anteriores.

> 📘 **Definição**
> **Baking** (ou geração de mapas) é o processo de transferir o detalhe geométrico da malha de alta densidade para mapas de textura — sobretudo o mapa de normais — aplicáveis à malha de baixa densidade. O resultado é um objeto leve, que roda em tempo real, mas que aparenta possuir o detalhe da versão pesada.

O elo entre as duas versões é o processo de *baking*, que a Parte III introduz no contexto dos mapas PBR e que a Parte V (Capítulos 15 e 16) desenvolve em detalhe: por meio dele, o detalhe geométrico da malha de alta densidade é "transferido" para mapas de textura — sobretudo o mapa de normais — aplicados sobre a malha de baixa densidade. Assim, o objeto leve que roda no jogo *parece* possuir o detalhe do objeto pesado, exatamente o princípio que a Parte I anunciou ao tratar do mapa de normais. Para que essa transferência funcione, a preparação precisa garantir certas condições — que a malha de baixa densidade esteja bem desdobrada, com densidade adequada e *padding* suficiente, e que as duas malhas estejam alinhadas no espaço — e é por isso que a preparação de *assets* é o passo que torna possível tudo o que se seguirá.

### Níveis de detalhe, colisão e modularidade

A preparação madura antecipa ainda três necessidades do *asset* em jogo. A primeira são os **níveis de detalhe** (*LODs*): versões progressivamente mais simples do modelo, exibidas conforme ele se afasta da câmera, de modo que o motor de jogo não gaste recursos detalhando o que está distante. A existência de LODs influencia decisões de mapeamento UV, pois todas as versões compartilham idealmente o mesmo conjunto de texturas e devem manter densidade coerente. A segunda é a **geometria de colisão**: uma forma simplificada, invisível, que o motor de jogo usa para calcular contatos físicos, distinta da malha visível e quase sempre muito mais leve. A terceira é a **modularidade** — a prática de construir cenários a partir de peças reutilizáveis que se encaixam, como paredes, pisos e colunas padronizados.

A modularidade conecta-se diretamente à texturização por meio de duas técnicas que a indústria consagrou e que vale conhecer desde já. As **trim sheets** são texturas organizadas em faixas horizontais de detalhes reutilizáveis — molduras, frisos, bordas, tubulações —, sobre as quais várias peças diferentes mapeiam seus UVs, de modo que uma só textura vista muitos objetos. O **hotspot texturing**, técnica relacionada, mapeia automaticamente cada face para a região mais adequada de uma textura-atlas de variações, permitindo revestir muitas superfícies com aparência variada a partir de um único mapa. Ambas levam ao extremo o princípio econômico que perpassa toda a Parte II: extrair o máximo de variedade e qualidade visual do mínimo de memória de textura, reutilizando o mesmo espaço UV em muitos lugares.

> **Figura 7.1** — Esquema de uma trim sheet — uma textura dividida em faixas horizontais de molduras, frisos e bordas — com setas mostrando como diferentes peças modulares (uma parede, uma coluna, um batente) mapeiam seus UVs para as faixas apropriadas, todas servidas por um único mapa.

**Screenshot sugerido:** Captura de uma trim sheet ao lado de um conjunto de peças modulares texturizadas a partir dela, evidenciando a reutilização da mesma textura em geometrias distintas.

### Exportação e importação

O elo físico entre o software de modelagem e o motor de jogo é o **arquivo de intercâmbio**, sendo os formatos mais comuns o FBX e o glTF. A exportação correta envolve decidir o que incluir (malha, UVs, normais, eventuais esqueletos de animação), em que escala e com que orientação, e conferir que essas escolhas sobrevivem à viagem até o motor de jogo. Do lado do motor, a **importação** envolve configurar como o *asset* será interpretado: como suas texturas serão associadas ao material, em que espaço de cor cada mapa será lido (recuperando a regra da Parte I, em que apenas a cor base é sRGB), como os mipmaps serão gerados e que compressão será aplicada. Um *asset* só está verdadeiramente pronto quando atravessa essa ponte e aparece no motor de jogo com a aparência, a escala e o comportamento esperados — e verificar isso, dentro do motor de destino, é a etapa final que confirma que toda a preparação foi bem-sucedida.

> ⚠️ **Atenção**
> Considerar o trabalho concluído quando o modelo "fica bonito" no software de modelagem é um dos equívocos mais frequentes. O *asset* só está realmente pronto quando atravessa a ponte para o motor de jogo e funciona na escala, na orientação, no comportamento e na organização que a equipe e a plataforma esperam.

## Aplicação em Jogos

Em produção real, a preparação de *assets* é o que separa um modelo bonito de um modelo *utilizável*. Estúdios mantêm guias de estilo técnico que padronizam unidades, nomenclatura, organização de pastas, densidade de texels e convenções de exportação justamente porque, em equipes grandes, a falta de padrão multiplica o atrito: cada *asset* fora da norma exige correção manual, cada nome obscuro custa tempo de busca, cada escala errada quebra a física ou a iluminação de uma cena inteira. A disciplina de preparação é, nesse sentido, um investimento coletivo: o cuidado de um artista poupa o tempo de muitos.

As técnicas de reutilização — LODs, modularidade, trim sheets, hotspot texturing — são, por sua vez, o que viabiliza economicamente os jogos de grande escala. Um mundo aberto com centenas de edifícios não pode ter uma textura única para cada um; ele é montado a partir de peças modulares que compartilham trim sheets e atlas, e cuja preparação foi pensada desde o início para essa reutilização. Compreender a preparação de *assets* é, portanto, compreender como a ambição visual de um jogo se concilia com os limites de memória e processamento da plataforma — o mesmo equilíbrio entre fidelidade e desempenho que, desde a Parte I, se mostra como o fio condutor de toda a disciplina.

## Estudo de Caso

### Forza Motorsport e o Pipeline de Preparação de Carros Digitalizados — Turn 10 Studios, 2015

> **Figura 7.2** — Diagrama do pipeline de produção de um carro em *Forza Motorsport*, mostrando as etapas sequenciais: scan laser → retopologia → higiene (normais, transformações, pivô) → desdobramento UV → baking → texturização → LODs → exportação e verificação no motor. Cada caixa tem um critério de aprovação antes de avançar.

**Screenshot sugerido:** Captura de tela oficial de Forza Motorsport 6, Turn 10 Studios/Microsoft — disponível em https://www.forzamotorsport.net/en-US/news (blog oficial) ou via Xbox Wire em https://news.xbox.com/en-us/category/forza/

Turn 10 Studios produz, para cada *Forza Motorsport*, mais de trezentos carros com nível de detalhe de showroom: cada parafuso da carroceria, cada costura dos bancos de couro, cada símbolo do painel. Para alcançar esse padrão sem explodir o orçamento de produção, a equipe construiu um dos pipelines de preparação de *assets* mais rígidos e documentados da indústria. Christian Ammann descreveu o sistema na GDC 2016: cada carro passa por uma sequência obrigatória de etapas antes de entrar no motor de jogo, e qualquer *asset* que não cumpra a lista completa é devolvido ao início do fluxo — sem exceções.

O ponto de partida é o scan a laser do veículo real, realizado num estúdio próprio da Turn 10 com equipamento de precisão milimétrica. O modelo resultante tem dezenas de milhões de polígonos e não pode entrar no jogo — é o *high-poly* de referência. A retopologia manual produz o modelo de jogo, com geometria limpa e LODs bem definidos. É nessa etapa que surgem os primeiros problemas documentados pelo estúdio: transformações acumuladas do scanner (rotações e escalas não aplicadas), grupos de suavização desalinhados com as quebras visuais da carroceria, e normais inconsistentes em regiões de transição entre materiais diferentes. O pipeline da Turn 10 inclui uma etapa de higiene obrigatória — conferência de normais, aplicação de transformações, zeragem do pivô — antes que qualquer modelo avance para o desdobramento UV. A posição do pivô, em particular, é rigorosamente padronizada: cada carro tem o pivô no centro geométrico do chassi ao nível do chão, o que garante que o comportamento de física e o alinhamento na pista sejam idênticos para todos os veículos, independentemente do artista que o preparou.

A nomenclatura é outro ponto onde o pipeline da Turn 10 não negocia. Com mais de trezentos carros e dezenas de variantes por modelo (ano, pintura, pacote de corrida), um sistema de nomenclatura ambíguo produziria conflitos e retrabalho em escala industrial. Cada arquivo segue a convenção `fabricante_modelo_ano_variante_LOD`, e as texturas seguem o mesmo padrão com sufixo de mapa (`_albedo`, `_roughness`, `_normal`). O resultado é que qualquer artista, em qualquer momento do projeto, consegue localizar, identificar e substituir um *asset* sem ambiguidade — o que o blog da Xbox Wire identificou como um dos fatores centrais para a escala de produção do estúdio.

**O que aprender com isso:** Um pipeline de preparação rígido — com critérios de aprovação explícitos antes de cada etapa — não é burocracia: é o mecanismo que permite produzir centenas de *assets* por equipes grandes sem perder controle de qualidade ou coerência técnica. A padronização do pivô, das transformações e da nomenclatura não sacrifica a criatividade; ela libera o artista de corrigir erros básicos para dedicar-se ao que importa.

## Boas Práticas

A boa prática que sintetiza este capítulo é tratar a preparação de *assets* como parte integrante do trabalho, e não como uma formalidade apressada ao final. Conferir a higiene da malha — normais coerentes, grupos de suavização alinhados às costuras, ausência de *n-gons* e geometria degenerada — antes de exportar evita que defeitos se propaguem por todas as etapas seguintes. Modelar em escala real desde o início, com objetos de referência, e aplicar as transformações antes de exportar previne a classe inteira de problemas de tamanho e comportamento no motor de jogo.

Recomenda-se posicionar o pivô segundo a função do objeto, adotar e respeitar uma convenção de nomenclatura e uma estrutura de organização compartilhadas pela equipe, e manter a correspondência entre o nome do modelo e o de suas texturas. É boa prática planejar desde a preparação a relação entre malha de alta e de baixa densidade, garantindo as condições que o *baking* exigirá, e antecipar as necessidades do *asset* em jogo — LODs, colisão, modularidade — em vez de remendá-las depois. Por fim, a prática que fecha o ciclo: verificar o *asset* dentro do motor de jogo de destino, conferindo escala, orientação, materiais e espaço de cor, pois só a chegada bem-sucedida ao motor confirma que a preparação cumpriu seu papel.

## Erros Comuns

O erro mais recorrente é exportar sem aplicar as transformações e sem conferir a escala, fazendo o *asset* chegar ao motor de jogo gigante, minúsculo ou com comportamento físico anômalo — e, de quebra, com a densidade de texels comprometida, já que ela depende do tamanho físico correto. Igualmente comum é negligenciar a higiene da malha, deixando normais invertidas que produzem faces transparentes ou pretas no motor de jogo, ou grupos de suavização desalinhados das costuras, que gerarão artefatos no mapa de normais da etapa seguinte.

Frequente, também, é descuidar do pivô, descobrindo só no motor de jogo que o objeto rotaciona ou se apoia de modo errado, e abandonar qualquer convenção de nomenclatura, gerando a confusão dos múltiplos "final" indistinguíveis que paralisa o trabalho em equipe. Há o erro de tratar a malha de baixa densidade sem pensar no *baking* que virá, descobrindo tarde que o desdobramento UV ou o *padding* eram inadequados para a transferência de detalhe. E persiste o erro de mentalidade que atravessa toda a etapa: considerar o trabalho concluído quando o modelo "fica bonito" no software de modelagem, sem perceber que o *asset* só está realmente pronto quando atravessa a ponte para a produção e funciona, na escala, na orientação, no comportamento e na organização que a equipe e o motor de jogo esperam.

## Resumo

Este capítulo encerrou a Parte II tratando dos cuidados que transformam um modelo geometricamente preparado em um *asset* de produção. Começamos pela higiene da malha — normais coerentes, grupos de suavização alinhados às costuras, ausência de *n-gons* e geometria degenerada —, condição para uma exportação sem surpresas e mapas sem artefatos. Tratamos da escala e das unidades, mostrando que modelar em tamanho real é requisito não só de coerência visual, mas da física, da iluminação e da própria densidade de texels do capítulo anterior. Discutimos a origem, o pivô e a aplicação de transformações, cuja negligência gera comportamentos estranhos no motor de jogo, e a nomenclatura e a organização, sem as quais o trabalho em equipe não escala. Apresentamos a relação entre as malhas de alta e de baixa densidade e o processo de *baking* que as une, antecipando um tema que a Parte III retoma e a Parte V detalha, e mostrando que a preparação é o que torna a transferência de detalhe possível. Vimos como a preparação madura antecipa LODs, colisão e modularidade, e conhecemos as trim sheets e o hotspot texturing como expressões máximas do princípio de reutilização de espaço UV para construir mundos ricos com pouca memória. Por fim, tratamos da exportação e da importação como a ponte entre software de modelagem e motor de jogo, e da verificação no motor de destino como a etapa que confirma a preparação. Com isso, completa-se a Parte II: o modelo está mapeado, desdobrado, dimensionado, organizado e preparado para a produção. A Parte III poderá, então, dedicar-se a fazer aquilo para que toda essa preparação existe — criar as texturas que vestirão o *asset*.

## Exercícios

**1.** Liste e explique os principais itens da higiene da malha que devem ser conferidos antes de exportar um *asset*. Para cada um, descreva que problema ele causaria no motor de jogo se fosse negligenciado.

**2.** Explique por que modelar em escala real é importante, relacionando a escala a pelo menos três aspectos distintos do *asset* em produção. Como uma escala incorreta afeta a densidade de texels estabelecida no capítulo anterior?

**3.** Diagnóstico: um *asset* de porta, ao ser girado no motor de jogo para abrir, gira em torno do seu centro como uma porta giratória, em vez de articular pelas dobradiças. Identifique a causa, explique o conceito envolvido e descreva como corrigi-lo. Onde o pivô deveria estar?

**4.** Planejamento de pipeline: descreva as condições de preparação que uma malha de baixa densidade precisa satisfazer para que o *baking* a partir de uma malha de alta densidade funcione corretamente. Relacione sua resposta aos conceitos de desdobramento UV, densidade de texels e *padding* dos capítulos anteriores.

**5.** Análise: explique como as trim sheets e o hotspot texturing permitem construir cenários grandes com pouca memória de textura, relacionando essas técnicas ao princípio de reutilização de espaço UV (sobreposição) visto no Capítulo 6. Em que tipo de cenário essas técnicas trazem mais ganho, e por quê?

## Glossário

**Baking:** Processo de transferir o detalhe geométrico da malha de alta densidade para mapas de textura — sobretudo o mapa de normais — aplicáveis à malha de baixa densidade. Produz também mapas derivados como oclusão de ambiente e curvatura.

**Geometria de colisão:** Forma simplificada e invisível, distinta da malha visível, usada pelo motor de jogo para calcular contatos físicos entre objetos.

**Grupos de suavização (*smoothing groups*):** Configuração que determina quais arestas da malha serão renderizadas como suaves (aparência arredondada) e quais como angulosas (aresta viva).

**Higiene da malha:** Conjunto de verificações e correções geométricas — normais coerentes, ausência de *n-gons*, vértices soltos, faces degeneradas — realizadas antes da exportação para garantir que o modelo se comportará corretamente no motor de jogo.

**Hotspot texturing:** Técnica de mapeamento UV que associa automaticamente cada face do modelo à região mais adequada de uma textura-atlas de variações, permitindo grande variedade visual a partir de um único mapa.

**LOD (*level of detail*):** Versão simplificada de um modelo, exibida pelo motor de jogo quando o objeto está distante da câmera, para reduzir o custo de processamento sem perda perceptível de qualidade.

**Malha de alta densidade (*high-poly*):** Versão detalhada e pesada de um modelo, usada como referência para o *baking* mas não diretamente no jogo.

**Malha de baixa densidade (*low-poly*):** Versão otimizada e leve de um modelo, que de fato entra no jogo e recebe as texturas geradas a partir da malha de alta densidade.

**Pivô (*pivot*):** Ponto de origem de um *asset*, em torno do qual ele é posicionado, rotacionado e escalado no motor de jogo.

**Trim sheet:** Textura organizada em faixas horizontais de elementos reutilizáveis — molduras, frisos, bordas —, compartilhada por múltiplas peças modulares que mapeiam seus UVs para as faixas apropriadas.

## Leituras Complementares

- **What Is Texture Baking?** e **Baking Guide** (materiais de apoio da disciplina) — introduzem o processo de transferência de detalhe entre a malha de alta e a de baixa densidade, que a preparação deste capítulo torna possível e que a Parte V (Capítulos 15 e 16) desenvolverá em detalhe.
- **OLSEN, Morten. *The Ultimate Trim*** (material de apoio da disciplina) — estudo detalhado da técnica de trim sheets e de sua aplicação à construção modular de cenários; leitura recomendada para quem trabalha com arquitetura e cenário modular.
- **Blender Maps** e **Blender Normals** (materiais de apoio da disciplina) — abordam a relação entre grupos de suavização, normais e costuras, central para a higiene da malha tratada neste capítulo.
- ***Hotspot Texturing*** (defaultinteractive.co.uk/post/hotspot-texturing) — explicação prática da técnica de hotspot e de seu uso na texturização modular eficiente; complementa diretamente a seção de modularidade deste capítulo.
- **AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018** — as seções sobre níveis de detalhe (LOD) e otimização fundamentam as decisões de preparação para tempo real discutidas aqui.
- **Documentação da Unreal Engine** (dev.epicgames.com) **e da Unity** (docs.unity3d.com), seções sobre importação de malhas, escala, LODs e colisão — para observar como cada motor de jogo interpreta os *assets* preparados e como configurar corretamente a importação.
