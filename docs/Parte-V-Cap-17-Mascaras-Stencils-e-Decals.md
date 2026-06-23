# Capítulo 17 — Máscaras, Stencils e Decals

## Introdução

Os três capítulos de produção da Parte IV deixaram, cada um, uma referência adiada a um mesmo conjunto de instrumentos. O Capítulo 12 prometeu que a mistura de materiais em terrenos e a sobreposição de detalhe singular sobre a base repetível seriam desenvolvidas adiante. O Capítulo 13 identificou o mascaramento como uma operação tão central que mereceria capítulo próprio. O Capítulo 14 descreveu a máscara como a ponte entre a pintura manual e a distribuição guiada pela geometria, e os *decals* como a forma de acrescentar o particular sobre o genérico. E os Capítulos 15 e 16, que abriram esta parte, deram-nos o *baking* e os mapas de malha — curvatura, oclusão de ambiente, *ID* — de que vários desses instrumentos dependem. Este capítulo cumpre todas essas promessas de uma vez, pois trata precisamente do que faltava: os mecanismos pelos quais se controla *onde* cada coisa aparece numa superfície. Se os capítulos anteriores ensinaram a gerar e a pintar conteúdo, este ensina a *posicioná-lo* — e essa capacidade de governar a localização é o que articula todos os métodos da parte num fluxo único.

Três instrumentos organizam o capítulo, todos variações de uma mesma ideia. A **máscara** controla onde uma camada ou um efeito se aplica, sendo o mecanismo geral de seletividade que sustenta a pintura por camadas e a distribuição procedural. O **stencil** é o uso de uma imagem como molde para projetar uma forma específica sobre a superfície — um carimbo controlado. O **decal** é um elemento sobreposto, aplicado por cima do material existente, que acrescenta detalhe singular sem alterar a textura de base — o adesivo que se cola sobre a superfície já pronta. Os três respondem à tensão que estrutura os métodos de produção (Parte IV), entre o genérico reutilizável e o singular intencional: a máscara decide o que se revela e onde; o stencil imprime uma marca precisa; o decal cola o detalhe único sobre a base econômica. Compreendê-los é completar o repertório de produção de texturas e entender, enfim, como as peças apresentadas separadamente se montam num *pipeline* coerente.

## Desenvolvimento

### A máscara: o controle universal do "onde"

> 📘 **Definição**
> Uma **máscara** é uma imagem em tons de cinza que controla a intensidade com que um efeito ou camada se aplica em cada ponto da superfície. Onde a máscara é branca, o efeito aparece integralmente; onde é preta, não aparece; onde é cinza, aparece proporcionalmente.

A **máscara** é, conceitualmente, o instrumento mais fundamental deste capítulo, e já o encontramos em todos os anteriores sob nomes diversos. Toda seletividade na texturização passa por máscaras. Uma camada de ferrugem só existe nas quinas porque uma máscara a revela ali e a oculta no resto; dois materiais se misturam num terreno porque uma máscara decide, ponto a ponto, qual prevalece; um efeito de sujeira se concentra nas reentrâncias porque uma máscara o confina a elas. A máscara é, em suma, a resposta universal à pergunta "onde?", e dominar a texturização é, em grande medida, dominar a construção de máscaras.

A potência do conceito está em que uma máscara pode ser obtida de muitas fontes, e a escolha da fonte determina o caráter do controle. Uma máscara pode ser **pintada à mão**, oferecendo o controle singular e intencional do Capítulo 14 — desenha-se exatamente onde o efeito deve estar. Pode ser **gerada proceduralmente**, a partir de um ruído ou de um padrão, oferecendo a distribuição automática e variada do Capítulo 13 — a sujeira se espalha segundo uma regra. E pode ser **derivada da geometria**, calculada a partir da forma do próprio modelo, oferecendo a distribuição fisicamente plausível que mencionamos repetidamente. Essas fontes não se excluem: combinam-se livremente, somando-se e multiplicando-se, de modo que uma máscara final pode resultar de uma distribuição procedural confinada pela geometria e ajustada à mão. É essa combinação que torna a máscara a ponte entre os paradigmas — ela é o ponto exato em que a regra do procedural, a forma da geometria e a intenção da pintura se encontram e negociam quem manda em cada pixel.

### Máscaras derivadas da geometria: deixar a forma decidir

Merece desenvolvimento próprio a categoria de máscaras que mais transformou a texturização moderna: as **máscaras geradas a partir da geometria do modelo**, possíveis graças aos mapas derivados do *baking* — processo antecipado no Capítulo 10 e detalhado nos Capítulos 15 e 16, que abriram esta parte. Quando se assa a malha de alta densidade sobre a de baixa, geram-se, além do mapa de normais, mapas que descrevem propriedades da forma: a **curvatura**, que distingue as quinas salientes (convexas) das reentrâncias (côncavas); a **oclusão de ambiente**, que mede quão "escondido" cada ponto está; a **espessura**, a **posição** no espaço, e os mapas de **identificação** (*ID*), que rotulam diferentes partes do objeto com cores distintas. Cada um desses mapas pode servir de máscara, e o conjunto permite que o processo de texturização "perceba" a forma do objeto e reaja a ela automaticamente.

> 💡 **Dica Profissional**
> Ao iniciar a texturização de qualquer *asset*, gere imediatamente os mapas de curvatura, oclusão de ambiente e *ID* no mesmo processo de *baking*. Esses mapas custam pouco tempo extra de assamento e entregam de graça a distribuição fisicamente plausível de desgaste, sujeira e segmentação de material — trabalho que, feito à mão, consumiria horas.

A consequência é profunda, pois transforma regras físicas gerais em distribuição concreta sobre cada *asset* específico. Sabemos que o desgaste se concentra nas quinas salientes: basta usar o mapa de curvatura como máscara para que o metal exposto apareça exatamente nas arestas convexas de *qualquer* objeto, sem que o artista o pinte. Sabemos que a sujeira se acumula nas reentrâncias: a oclusão de ambiente como máscara a deposita ali automaticamente. Sabemos que partes diferentes são de materiais diferentes: o mapa de *ID* permite atribuir um material a cada parte de uma só vez. Essas máscaras são o que, no Capítulo 13, permitia ao *smart material* distribuir seus efeitos plausivelmente, e o que, no Capítulo 14, deixava a geometria guiar o que podia ser guiado. Elas incorporam, de forma reutilizável, o conhecimento físico de como as superfícies envelhecem — e por isso uma máscara de curvatura aplicada a mil objetos produz, em cada um, o desgaste certo para *sua* forma. É a automação inteligente da plausibilidade, e seu domínio é hoje uma competência central do texturizador. *Ratchet & Clank* (Insomniac Games, 2016) é um exemplo documentado dessa prática: a equipe descreveu em análises técnicas como todos os personagens e props do jogo recebiam, de forma padronizada, uma máscara de curvatura que dirigia o brilho especular para as arestas convexas e acumulava sombra de contato nas reentrâncias, sem pintura manual — a forma de cada objeto, ao ser assada, tornava-se literalmente a instrução de como ele devia envelhecer.

> **Figura 17.1** — Um mesmo objeto mecânico exibido com quatro de suas máscaras derivadas da geometria — curvatura (quinas em destaque), oclusão de ambiente (reentrâncias escuras), identificação de partes (cores chapadas) e o material final em que cada máscara controlou um efeito: desgaste nas quinas, sujeira nas reentrâncias, materiais distintos por parte.

**Screenshot sugerido:** Captura de uma ferramenta de texturização mostrando os mapas de curvatura, oclusão de ambiente e *ID* lado a lado e o material resultante, evidenciando como cada máscara guia um efeito.

### O stencil: projetar uma forma específica

Enquanto a máscara controla a intensidade de um efeito sobre toda uma camada, o **stencil** serve a um propósito mais pontual: imprimir uma *forma específica* sobre a superfície, como um molde vazado através do qual se pinta. O nome vem da técnica física do estêncil — a chapa recortada que, posta sobre uma parede, deixa a tinta passar apenas pelos vãos, reproduzindo um símbolo. Na texturização digital, um stencil é uma imagem que se projeta sobre o modelo e através da qual se aplica tinta ou material: o que é branco no stencil deixa passar, o que é preto bloqueia, e o resultado é a forma do stencil carimbada na superfície, no lugar e na orientação que o artista escolher.

> 📘 **Definição**
> Um **stencil** é uma imagem usada como molde para projetar formas específicas sobre a superfície do modelo. Diferente da máscara geral, o stencil carrega uma forma definida — um símbolo, número ou padrão — e é posicionado intencionalmente pelo artista sobre o modelo.

O stencil é o instrumento ideal para aplicar conteúdo *figurativo e específico* — um número pintado num avião, um símbolo numa caixa, uma marca de pneu, um padrão de camuflagem — que seria trabalhoso pintar à mão livre e impossível gerar por regra. O stencil ocupa, assim, uma posição intermediária no espectro genérico–singular que organiza a parte: é mais específico que uma máscara procedural, pois carrega uma forma definida, mas mais reutilizável que a pintura à mão livre, pois a mesma imagem-molde pode ser carimbada muitas vezes, em muitos lugares e objetos. A projeção do stencil sobre o modelo 3D — herdeira da pintura 3D do Capítulo 14 — garante que a forma se assente corretamente sobre a superfície, acompanhando seu relevo, sem a distorção que a aplicação no espaço desdobrado produziria.

### O decal: colar o singular sobre o genérico

O terceiro instrumento, e o que melhor encarna a filosofia de produção da Parte IV, é o **decal**. Um decal é um elemento de detalhe — uma imagem com sua própria transparência — aplicado *por cima* de uma superfície já texturizada, sem alterar a textura de base. O nome vem dos adesivos decalque: assim como se cola um adesivo sobre um objeto pronto, aplica-se um decal sobre um material existente para acrescentar um detalhe que não estava lá. Marcas de bala num muro, rachaduras numa parede, manchas de óleo no chão, pichações, sinais de trânsito, respingos de sangue, folhas caídas, placas e avisos — tudo isso são decais, sobrepostos à base sem que ela precise ser refeita.

> 📘 **Definição**
> Um **decal** é um elemento de detalhe aplicado por sobreposição sobre uma superfície já texturizada, sem modificar a textura de base. Pode incluir não apenas informação de cor, mas também modificações de mapa de normais e mapa de rugosidade da região coberta.

A diferença em relação ao stencil é de natureza: o stencil *modifica* a textura de base, carimbando nela; o decal é uma *camada adicional*, sobreposta em tempo de jogo ou de autoria, que pode até ser movida ou removida sem deixar vestígio na base.

O decal é a solução elegante para o problema fundador do Capítulo 12: a textura repetível é econômica mas genérica, e não pode conter o detalhe singular sem perder a economia. O decal resolve o impasse separando as duas camadas — a base permanece genérica, repetível e barata; o detalhe singular vem por cima, posicionado livremente onde se quer. Uma parede de concreto repetível, monótona em sua repetição, ganha vida e individualidade quando se aplicam sobre ela alguns decais de rachaduras, manchas e pichações, cada um único, cada um no seu lugar — e a parede deixa de repetir visivelmente não porque a base mudou, mas porque o singular sobreposto quebrou a regularidade. Os decais são, além disso, extremamente eficientes: um pequeno repertório de decais reutilizáveis, distribuído com variedade por um cenário inteiro, multiplica a aparência de detalhe a um custo mínimo, reencontrando a economia que rege toda a apostila.

> **Figura 17.2** — Uma parede de concreto repetível em dois estados — repetida sobre uma área extensa, com a regularidade visível e monótona, e a mesma parede após a aplicação de decais singulares (rachaduras, manchas, pichações, marcas de impacto), cada um posicionado individualmente, quebrando a repetição sem que a base tenha mudado.

**Screenshot sugerido:** Comparação de uma superfície repetível antes e depois da aplicação de *decals*, destacando que a base permaneceu idêntica e que apenas a camada sobreposta foi acrescentada.

### A síntese: orquestrar os instrumentos num pipeline único

Reunidos os três instrumentos, e somados aos métodos dos capítulos anteriores, completa-se o quadro da produção de texturas, e vale articulá-lo como um todo, pois é essa visão de conjunto o que a Parte IV pretende formar. Uma superfície real, num jogo bem produzido, é tipicamente o resultado de várias camadas sobrepostas, cada uma proveniente de um método. Na base, uma textura **repetível** ou uma ***trim sheet*** (Capítulo 12) cobre a grande área com economia. Sobre ela, materiais **procedurais** (Capítulo 13) estabelecem variação e desgaste coerentes, distribuídos por **máscaras geradas pela geometria** (este capítulo) que respeitam a forma do objeto. A **pintura manual** (Capítulo 14), guiada por **máscaras pintadas à mão** e por **stencils**, acrescenta as marcas singulares e intencionais. E, por cima de tudo, **decais** depositam o detalhe único que quebra a repetição e individualiza cada ponto.

> 💡 **Dica Profissional**
> Pense em qualquer superfície complexa como uma composição de camadas com responsabilidades distintas: a base repetível cuida da cobertura econômica; os procedurais e as máscaras de geometria distribuem o envelhecimento plausível; os stencils e a pintura manual adicionam marcas específicas; os decais individualizam e narram. Cada método faz o que faz melhor — e nenhum precisa fazer tudo.

Essa orquestração é a competência culminante do texturizador, e a verdadeira lição dos métodos de produção da Parte IV. Não se trata de escolher *um* método, mas de saber, diante de cada superfície, qual combinação de métodos rende o melhor resultado dentro do orçamento — exatamente o juízo de investimento seletivo que percorre toda a apostila.

## Aplicação em Jogos

Na produção, máscaras, stencils e decais são instrumentos cotidianos que permeiam todos os estágios da texturização. As máscaras derivadas da geometria são a base do fluxo moderno de autoria de materiais: ao texturizar qualquer *asset*, o artista gera de imediato os mapas de curvatura, oclusão de ambiente e *ID* a partir do *baking*, e os usa para distribuir desgaste, sujeira e materiais por parte, automatizando a plausibilidade física e poupando trabalho manual. Os stencils aceleram a aplicação de conteúdo figurativo — numerações, símbolos, padrões, logotipos — que de outro modo consumiria horas de pintura. E os decais são onipresentes nos cenários: estúdios mantêm bibliotecas de decais reutilizáveis — rachaduras, manchas, sinais, danos — que os artistas de ambiente distribuem pelos níveis para individualizar superfícies, quebrar a repetição das texturas repetíveis e contar pequenas histórias visuais sem custo de memória de textura proporcional.

O uso de decais em tempo de jogo merece nota, pois conecta a texturização à jogabilidade. Muitos efeitos dinâmicos — as marcas de tiro que aparecem onde o jogador atira, os rastros de pneu, as poças que se formam — são decais aplicados pelo motor de jogo em tempo real sobre as superfícies existentes, permitindo que o mundo reaja às ações sem que cada superfície precise de uma textura única para cada estado possível. *Unreal Tournament* (Epic Games, 1999) é considerado um dos primeiros jogos comerciais a implementar decais dinâmicos de marca de projétil — o círculo de impacto colado na parede onde a bala acertou. *Battlefield: Bad Company 2* (DICE, 2010) expandiu o conceito para a escala da destruição em tempo real: fragmentos de parede demolida geravam rastros de entulho e marcas de explosão como decais sobre o terreno, multiplicando a sensação de caos sem custo de geometria adicional. Isso reencontra, no plano dinâmico, a mesma economia do decal estático: a base permanece genérica e o singular vem por cima, agora gerado pela própria jogabilidade.

## Estudo de Caso

### *Cyberpunk 2077* (CD Projekt RED, 2020) — Night City e a Arquitetura de Decais do REDengine 4

Night City, o ambiente central de *Cyberpunk 2077*, representa um dos maiores desafios de individualização de superfície já enfrentados pela indústria. A cidade é composta por milhares de fachadas, becos, interiores e infraestruturas que precisam parecer organicamente saturados de publicidade, grafite, desgaste e narrativa urbana — mas que compartilham, por baixo, um conjunto relativamente limitado de materiais repetíveis. A solução documentada pela CD Projekt RED em apresentações técnicas de 2020 e 2021 reside precisamente na orquestração de decais descrita neste capítulo: o REDengine 4 emprega um sistema de decais projetados em tempo real que não apenas sobrepõem cor, mas modificam simultaneamente o mapa de rugosidade e o mapa de normais da superfície receptora, produzindo a ilusão de que cada grafite, cada cartaz rasgado e cada mancha de óleo altera genuinamente a microgeometria da parede sobre a qual foi aplicado.

O princípio arquitetural é econômico: o estúdio mantém um repertório relativamente pequeno de decais individuais — grafismos de gangues, publicidade corporativa, manchas de umidade, marcas de impacto, pichações — e os distribui sobre a cidade com variação sistemática de posição, rotação, escala e sobreposição. O efeito de saturação visual que o jogador percorre nas ruas de Japantown ou no Pacifica não resulta de texturas únicas por fachada, mas de combinações praticamente infinitas desse vocabulário limitado. Decais de publicidade holográfica identificam territórios corporativos; decais de grafite das Tyger Claws ou dos Valentinos demarcam domínios de gangue; manchas de sangue e marcas de queimado narram confrontos recentes. A leitura do espaço urbano é construída inteiramente por essa camada de "onde" sobre bases compartilhadas.

O que distingue o sistema do REDengine 4 — e conecta diretamente ao conteúdo deste capítulo — é a profundidade do decal: ele não é uma sobreposição de cor plana, mas uma instrução completa de material que reescreve cor, mapa de rugosidade e mapa de normais na região projetada. Um cartaz de papel colado numa parede de concreto não apenas adiciona cor: aumenta ligeiramente a rugosidade nas bordas levantadas, modifica o mapa de normais para sugerir a espessura do papel e preserva o mapa de normais subjacente do concreto nas frestas onde o papel não aderiu. Essa cadeia de modificações, aplicada por um decal sobre um material base genérico, é exatamente a arquitetura de camadas descrita neste capítulo.

**O que aprender com isso:** A Night City funciona, em última análise, como uma demonstração em escala de cidade aberta do argumento central da Parte V: a base econômica e repetível fornece a superfície; a camada de decais fornece a identidade, a narrativa e a memória cultural do espaço.

> **Figura 17.3** — Comparação de uma fachada de Night City em dois estados: o material-base repetível sem decais, e o estado final com camadas de decais de publicidade, grafite e desgaste, evidenciando como o sistema do REDengine 4 modifica cor, mapa de rugosidade e mapa de normais simultaneamente sobre a mesma superfície base.

**Screenshot sugerido:** Captura do modo foto de *Cyberpunk 2077* (PC: tecla G) mostrando uma fachada densa de Night City — cartazes sobrepostos, grafites de gangues, manchas de umidade sobre concreto. Galeria oficial: cyberpunk.net/us/en/media

**Fonte:** CD Projekt RED, "The Art of Cyberpunk 2077", SIGGRAPH 2021; Marthe Jonkers & Jakub Knapik, entrevistas técnicas em *80.lv* (2020–2021).

## Boas Práticas

A boa prática que organiza este capítulo é pensar a texturização em *camadas de "onde"*: separar claramente o conteúdo (o que se aplica) do controle de localização (onde se aplica), e dominar as máscaras como o instrumento universal desse controle. Aproveite as máscaras derivadas da geometria — curvatura, oclusão de ambiente, *ID* — para automatizar a distribuição fisicamente plausível de desgaste, sujeira e materiais, deixando a forma do objeto decidir o que pode decidir e reservando o esforço manual para o que ela não alcança. Combine livremente as fontes de máscara — procedural para a variação, geometria para a plausibilidade, pintura à mão para a intenção —, lembrando que a máscara final é onde os três paradigmas da parte se encontram e negociam.

É boa prática usar *stencils* para o conteúdo figurativo e específico — numerações, símbolos, padrões — que é trabalhoso pintar à mão e impossível gerar por regra, projetando-os sobre o modelo 3D para que se assentem sem distorção. E é boa prática usar *decais* para acrescentar o detalhe singular sobre a base genérica, separando a camada econômica e repetível da camada única e posicionada — a estratégia mais eficaz contra a repetição visível do Capítulo 12 e a forma mais econômica de individualizar e narrar superfícies. Mantenha, em tudo, a consciência de custo: máscaras, decais e mistura de materiais têm um preço de renderização, e seu uso deve ser dosado conforme o orçamento da plataforma, concentrando a individualização onde o jogador olhará — a economia da atenção que percorre a apostila inteira.

## Erros Comuns

O erro conceitual mais básico é confundir o conteúdo com o seu controle de localização — pintar o detalhe diretamente e de modo irreversível na textura-base, em vez de aplicá-lo por uma camada mascarada ou por um decal —, perdendo toda a flexibilidade de reposicionar, ajustar ou remover sem refazer. Próximo dele está o subaproveitamento das máscaras de geometria: distribuir desgaste e sujeira por ruído puro ou à mão, ignorando os mapas de curvatura e oclusão de ambiente que dariam, de graça e fisicamente corretos, a distribuição que se está penando para imitar — repetindo o erro de implausibilidade já alertado no Capítulo 13.

> ❌ **Erro Comum**
> Subaproveitamento das máscaras de geometria: pintar desgaste e sujeira inteiramente à mão quando os mapas de curvatura e oclusão de ambiente, gerados durante o *baking*, já entregariam essa distribuição de forma fisicamente plausível e automática. O resultado do trabalho manual costuma ser menos convincente do que o gerado pelos mapas.

No uso dos decais, o erro mais comum é o excesso e a repetição visível: aplicar tantos decais, ou os mesmos decais tão reconhecíveis, que a própria camada de quebra passa a "tilear" — a mesma rachadura colada em vinte paredes idênticas denuncia o truque tanto quanto a base repetida que ela deveria disfarçar.

> ❌ **Erro Comum**
> Aplicar o mesmo decal repetidamente na mesma posição e escala cria um padrão reconhecível que denuncia o truque tanto quanto a textura repetida que deveria disfarçar. A solução é variar posição, rotação, escala e repertório de decais.

O remédio é variar posição, rotação, escala e repertório. Há também o erro de ignorar o custo: empilhar decais e misturas de materiais sem consciência do preço de renderização, esgotando o orçamento da plataforma com sobreposições que o jogador mal percebe. No uso dos stencils, o equívoco típico é aplicá-los no espaço desdobrado, produzindo distorção nas costuras, quando a projeção sobre o modelo 3D resolveria. E persiste o erro estratégico de fundo, que esta parte inteira combate: tratar os métodos como alternativas excludentes — querer fazer tudo só com texturas repetíveis, ou só com pintura, ou só com decais — em vez de orquestrá-los, deixando cada instrumento fazer aquilo em que é forte.

## Resumo

Este capítulo retomou e completou os métodos de produção da Parte IV — agora munido dos mapas do *baking* (Capítulos 15 e 16) — apresentando os instrumentos que controlam *onde* cada coisa aparece numa superfície. A **máscara** revelou-se o controle universal do "onde" — uma imagem em tons de cinza que decide a intensidade de um efeito em cada ponto — e a ponte entre os paradigmas, podendo ser pintada à mão (intenção), gerada proceduralmente (variação) ou derivada da geometria (plausibilidade), e combinando livremente essas fontes. Demos destaque às **máscaras derivadas da geometria** — curvatura, oclusão de ambiente, identificação de partes, obtidas pelo *baking* dos Capítulos 15 e 16 —, que transformam regras físicas gerais em distribuição concreta sobre cada *asset*, automatizando a plausibilidade do desgaste e da sujeira e constituindo hoje uma competência central. Apresentamos o **stencil** como o carimbo controlado, projetando formas figurativas específicas sobre o modelo, numa posição intermediária entre a máscara procedural e a pintura à mão. E apresentamos o **decal** como a encarnação da filosofia da parte: a camada singular sobreposta à base genérica, solução elegante para individualizar e narrar superfícies, quebrar a repetição das texturas repetíveis e acrescentar detalhe — estático ou dinâmico em tempo de jogo — a custo mínimo de memória. Reunindo tudo, descrevemos a superfície de um jogo bem produzido como uma composição de camadas — base repetível, procedural guiado pela geometria, pintura e stencils, decais por cima —, em que a máscara é o tecido conjuntivo, e afirmamos que a competência culminante do texturizador é orquestrar esses instrumentos, dando a cada superfície a combinação de métodos que melhor rende dentro do orçamento. Com isso completa-se o repertório de produção de texturas iniciado na Parte IV; os próximos capítulos voltam-se à organização e ao armazenamento eficiente dessas texturas — atlas, UDIMs e compressão.

## Exercícios

**1.** Explique por que a máscara pode ser chamada de "controle universal do onde" na texturização. Descreva as três grandes fontes de máscara — pintura à mão, geração procedural e derivação da geometria — e o tipo de controle que cada uma oferece, justificando por que a máscara é descrita como a ponte entre os paradigmas da Parte IV.

**2.** As máscaras derivadas da geometria são apontadas como uma das maiores transformações da texturização moderna. Explique o que são os mapas de curvatura, oclusão de ambiente e identificação (*ID*), de onde vêm (relacione ao *baking*, antecipado no Capítulo 10 e detalhado nos Capítulos 15 e 16 desta parte) e, com um exemplo de cada, como permitem que "a forma do objeto decida" a distribuição de desgaste, sujeira e materiais.

**3.** Comparação: distinga stencil e decal quanto à natureza (o que cada um faz à textura de base), quanto ao tipo de conteúdo a que melhor se aplicam e quanto à sua posição no espectro entre o genérico reutilizável e o singular intencional. Dê um exemplo de uso adequado para cada um.

**4.** Diagnóstico: um cenário de muros de concreto repetível foi "individualizado" colando-se a mesma rachadura de decal em todos os muros, sempre na mesma posição e escala. O resultado, em vez de quebrar a repetição, parece tão artificial quanto a base repetida. Identifique o erro, explique por que a solução de quebra acabou reproduzindo o problema que deveria resolver, e proponha a correção.

**5.** Síntese dos métodos de produção: escolha uma superfície complexa de jogo — por exemplo, a fachada de um prédio abandonado, vista de perto e de longe. Descreva, em texto contínuo, como você a produziria combinando os métodos de produção: que base repetível ou *trim sheet* usaria (Capítulo 12), que variação procedural aplicaria e como (Capítulo 13), o que pintaria à mão (Capítulo 14) e quais máscaras, stencils e decais empregaria e por quê (Capítulo 17). Justifique cada escolha pela tensão entre o genérico e o singular e pelo equilíbrio entre fidelidade e custo que percorrem a apostila.

## Glossário

**Curvatura:** Mapa assado que distingue regiões convexas (quinas salientes) e côncavas (reentrâncias) da superfície. Usado como máscara para concentrar desgaste nas arestas e sujeira nas cavidades de forma automaticamente plausível.

**Decal:** Elemento de detalhe aplicado como camada sobreposta sobre uma superfície já texturizada, sem alterar o material de base. Pode modificar cor, mapa de normais e mapa de rugosidade da região coberta.

**Mapa de identificação (ID map):** Mapa assado que pinta cada parte ou submaterial do objeto com uma cor sólida distinta, permitindo selecionar e atribuir materiais por região de forma eficiente.

**Mapa de oclusão de ambiente:** Mapa assado que registra o grau de exposição de cada ponto à luz ambiente. Reentrâncias aparecem escuras; superfícies expostas, claras. Usado como máscara para depositar sujeira e desgaste nas cavidades.

**Máscara:** Imagem em tons de cinza que controla a intensidade com que um efeito ou camada se aplica em cada ponto da superfície. Pode ser pintada à mão, gerada proceduralmente ou derivada da geometria.

**Máscara derivada da geometria:** Máscara gerada automaticamente a partir de mapas obtidos no *baking* — curvatura, oclusão de ambiente, posição, espessura. Distribui efeitos de acordo com a forma real do objeto, sem pintura manual.

**Smart material:** Conjunto pré-configurado de camadas e geradores em ferramentas de texturização que aplica automaticamente efeitos de desgaste, sujeira e variação usando máscaras derivadas da geometria.

**Stencil:** Imagem usada como molde para projetar uma forma específica sobre a superfície do modelo. Funciona como um carimbo controlado, posicionado pelo artista, que aplica formas figurativas definidas.

## Leituras Complementares

- **What Is Texture Baking?** e **Baking Guide** (materiais de apoio da disciplina) — Detalham a geração dos mapas derivados da geometria — curvatura, oclusão de ambiente, identificação — que sustentam as máscaras automáticas centrais deste capítulo; leitura essencial para compreender de onde vêm essas máscaras.

- **OLSEN, Morten. *The Ultimate Trim — Texturing Techniques of Sunset Overdrive*** (Insomniac Games; material de apoio da disciplina) — Conecta a lógica de reutilização das *trim sheets* (Capítulo 12) à aplicação de detalhe singular por sobreposição, contexto das estratégias deste capítulo. Vale consultar para observar como máscaras e decais integram um *pipeline* de grande porte.

- **Adobe Substance 3D** — https://helpx.adobe.com/substance-3d.html. Documentação do Substance Painter sobre geradores e máscaras guiadas pela geometria (curvatura, oclusão de ambiente, *ID*), aplicação de stencils e fluxo de camadas mascaradas — referência direta deste capítulo.

- **Unreal Engine Documentation** (https://dev.epicgames.com/documentation/unreal-engine/) e **Unity Manual** (https://docs.unity3d.com/) — Seções sobre sistemas de *decal*, projeção de decais, mistura de materiais em terrenos e seu custo de renderização; indispensáveis para observar como os motores de jogo implementam os instrumentos aqui tratados, inclusive em tempo de jogo.

- **Godot Engine Documentation** — https://docs.godotengine.org/. Seções sobre *decals* e mistura de materiais em um motor de código aberto, úteis para comparar implementações.

- **Blender Manual** — https://docs.blender.org/. Seções sobre pintura com stencils, máscaras e geração de mapas pelo *baking*, para praticar os conceitos em uma ferramenta de código aberto.

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. ***Real-Time Rendering***. 4. ed. Boca Raton: CRC Press, 2018 — Os capítulos sobre *decaling*, mistura de texturas e composição de materiais fundamentam tecnicamente os instrumentos deste capítulo. Indicado para aprofundamento na teoria de renderização por trás das técnicas.
