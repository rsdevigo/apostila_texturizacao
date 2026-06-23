# Capítulo 11 — Sombreadores, Iluminação e Aparência Final

## Introdução

Chegamos ao capítulo que fecha a Parte III e, com ela, o arco que vinha sendo construído desde o início desta apostila. Temos um modelo preparado, desdobrado e dimensionado; temos um material PBR completo, com seus mapas calibrados e fisicamente corretos. Falta entender o que acontece no instante final — aquele em que todos esses ingredientes se encontram, dentro do motor de jogo, com a luz da cena, e produzem a imagem que o jogador efetivamente vê. Esse encontro é mediado por dois protagonistas que ainda tratamos apenas de passagem: o **sombreador** (*shader*), o programa que executa o cálculo de aparência, e a **iluminação**, o conjunto de luzes e de luz ambiente que alimenta esse cálculo. Este capítulo trata de como o material se torna aparência: do papel do sombreador, do modo como a iluminação determina o resultado e dos fatores do motor que afetam a imagem final.

Desde o Capítulo 1 repetimos que, em um jogo, a aparência de uma superfície é um *cálculo*. Podemos agora nomear quem faz esse cálculo: é o sombreador, executado pela placa de vídeo para cada pixel, combinando os mapas do material com a luz da cena segundo o modelo físico do PBR. Compreender o sombreador não significa, para os fins deste curso, aprender a programá-lo — embora indiquemos caminhos para quem quiser —, mas entender *o que ele faz* com os mapas que tanto trabalhamos para construir, e por que a mesma superfície pode parecer tão diferente sob iluminações distintas. É também o capítulo em que o ciclo se completa: veremos que um material só pode ser julgado em conjunto com a iluminação sob a qual aparecerá, e que a verificação no motor de jogo, tantas vezes recomendada nas partes anteriores, é o ato final que confirma todo o trabalho. Ao terminá-lo, o estudante terá percorrido o caminho completo, da geometria nua à imagem iluminada.

## Desenvolvimento

### O que é um sombreador

> 📘 **Definição**
> Um **sombreador** (*shader*) é um programa executado pela GPU (placa de vídeo) que calcula, para cada pixel visível de uma superfície, a cor que ele deve exibir — combinando os mapas do material com a posição das luzes, a posição da câmera e os parâmetros do ambiente segundo o modelo físico do PBR.

Um sombreador é, portanto, o executor concreto daquilo que chamamos, desde a Parte I, de "cálculo de aparência". Quando o motor de jogo renderiza um objeto, o sombreador é chamado uma vez para cada ponto visível da superfície e, naquele instante, dispõe de tudo o que precisa: os valores dos mapas do material naquele ponto — cor base, metálico, rugosidade, direção da normal —, a posição e a direção das luzes que atingem o ponto, a posição da câmera e os parâmetros do ambiente. Com esses dados, o sombreador aplica o modelo de iluminação do PBR — as fórmulas que traduzem os princípios do Capítulo 8, conservação de energia, Fresnel, dispersão pela rugosidade — e produz a cor final daquele pixel. Multiplicado por milhões de pixels, dezenas de vezes por segundo, esse cálculo é o que faz a imagem do jogo.

É esclarecedor perceber que os mapas que construímos ao longo desta parte são, do ponto de vista do sombreador, simplesmente *entradas*. A cor base entra como a cor difusa ou de reflexo; o mapa de rugosidade entra como o parâmetro que controla a dispersão do realce; o mapa de normais entra como a direção da superfície usada no cálculo da luz. O sombreador não "sabe" da maçaneta de bronze nem do baú de tesouro; ele apenas avalia uma fórmula com os números que os mapas lhe fornecem. Isso ilumina retrospectivamente tudo o que dissemos: a razão de a cor base não dever conter luz, de o mapa de rugosidade dever variar, de o espaço de cor dever estar correto, é que esses valores serão *consumidos por uma fórmula física*, e qualquer valor errado corrompe a fórmula.

> ⚠️ **Atenção**
> O sombreador calcula exatamente o que os mapas lhe fornecem, sem corrigir enganos de autoria. Um mapa de rugosidade com valores fora das faixas de referência, um espaço de cor incorreto ou uma cor base contaminada por iluminação produzirão resultados errados que nenhum ajuste de cena corrige.

Os jogos usam, em sua maioria, um sombreador PBR padronizado fornecido pelo motor de jogo — o sombreador *Lit* da Unity nos pipelines modernos URP e HDRP, os materiais base da Unreal —, mas permitem também sombreadores customizados para efeitos especiais. *Dishonored* (Arkane Studios, 2012) é um caso documentado de sombreador customizado coexistindo com PBR no mesmo motor: o sombreador de contorno pintado — linhas de tinta ao longo das arestas de personagens e objetos, inspiradas na arte de Viktor Antonov — foi implementado como um sombreador de pós-processo separado do pipeline PBR padrão, sem alterar os materiais base dos *assets*; a coexistência de um visual estilizado e de superfícies PBR fisicamente corretas no mesmo quadro é um dos exemplos mais citados de uso criterioso de sombreadores customizados sem comprometer o restante do pipeline.

### Iluminação: a outra metade da equação

Se o sombreador é quem calcula, a **iluminação** é metade do que ele calcula com. Um material, por mais bem construído, não tem aparência alguma no escuro; sua aparência *emerge* do encontro com a luz, e luzes diferentes produzem aparências diferentes da mesma superfície — exatamente a propriedade que, no Capítulo 8, identificamos como a grande virtude do PBR. Vale, portanto, conhecer os tipos de luz que compõem a iluminação de uma cena. As **luzes diretas** — direcional (como o sol, com raios paralelos), pontual (como uma lâmpada, irradiando de um ponto), em foco (*spot*, como um holofote cônico) — fornecem a iluminação principal, produzindo os realces especulares nítidos e as sombras projetadas. A **iluminação ambiente** ou **indireta** representa a luz que ricocheteia pelo ambiente antes de atingir a superfície, e é o que impede que as sombras fiquem absolutamente pretas e que dá aos objetos o "preenchimento" de luz que os assenta na cena.

> 📘 **Definição**
> A **iluminação baseada em imagem** (*image-based lighting*, IBL) utiliza uma imagem panorâmica do ambiente — um céu, um interior — como fonte de luz envolvente. O IBL é a origem dos reflexos em superfícies metálicas e lisas: é dele que essas superfícies extraem o que refletem.

O IBL é decisivo para os reflexos: é dele que os metais e as superfícies lisas extraem o que refletem, e é por isso que um material metálico só revela plenamente seu caráter quando há um ambiente para refletir. Uma esfera de cromo num vazio preto parece quase invisível; a mesma esfera sob um IBL rico de um pôr do sol torna-se imediatamente reconhecível como metal polido. *Mirror's Edge Catalyst* (DICE, 2016) explorou isso intencionalmente: os vidros e metais brancos da cidade de Glass recebiam IBL de alta qualidade gerado pelo Frostbite 3, e o resultado — documentado pela DICE no SIGGRAPH 2016 — comunicava a utopia corporativa da ficção inteiramente através da qualidade reflexiva das superfícies PBR, sem truques de cor embutida; era o IBL, e não a textura, o responsável pela frieza e clareza que definiam visualmente a facção Conglomerate.

> ⚠️ **Atenção**
> Um material PBR não pode ser avaliado sem uma iluminação representativa. Julgar um metal sob iluminação pobre é como provar um prato frio: a aparência que importa é a que emerge sob a iluminação real da cena.

> **Figura 11.1** — Uma mesma esfera metálica exibida sob três iluminações — um vazio quase preto, um IBL de estúdio neutro e um IBL de pôr do sol — mostrando como o caráter metálico, invisível no primeiro caso, emerge plenamente nos outros, e como a cor e a direção da luz transformam a aparência sem que nenhum mapa tenha mudado.

**Screenshot sugerido:** Captura de uma ferramenta de autoria ou motor com o mesmo material exibido sob diferentes mapas de ambiente (HDRI), evidenciando a dependência da aparência em relação à iluminação.

### O modelo de iluminação e a função de distribuição (BRDF)

> 📘 **Definição**
> A **BRDF** (*bidirectional reflectance distribution function*, ou função de distribuição de reflectância bidirecional) é o modelo matemático que descreve como a luz se reflete numa superfície. Dadas a direção de onde a luz vem e a direção para onde o olho olha, a BRDF determina quanta luz é refletida naquela combinação de ângulos.

Por baixo do sombreador PBR há, portanto, um modelo formal que incorpora os princípios do Capítulo 8 — a conservação de energia (a função nunca devolve mais do que entra), o efeito de Fresnel (mais reflexão nos ângulos rasantes) e a influência da rugosidade (que alarga ou estreita o lóbulo de reflexão). Os mapas do material entram na BRDF como parâmetros: o mapa de rugosidade molda o formato da função, o mapa metálico e a cor base determinam a intensidade e a cor da reflexão. Não precisamos manipular suas equações neste curso, mas compreender o que ela representa é necessário.

A importância de mencionar a BRDF, mesmo sem suas fórmulas, é dupla. Primeiro, ela explica *por que* o PBR é "baseado em física": há, de fato, um modelo derivado da óptica por trás de cada pixel, e não um amontoado de truques. Segundo, ela explica por que materiais autorados segundo as convenções PBR são *transferíveis* entre motores de jogo: como diferentes motores implementam BRDFs muito semelhantes — variações de um mesmo modelo padrão consolidado na indústria —, um material correto em um tende a permanecer correto em outro, com diferenças sutis.

> 💡 **Dica Profissional**
> A transferibilidade dos materiais PBR entre motores de jogo é uma consequência direta da convergência em torno de BRDFs semelhantes. Um material bem autorado no fluxo de trabalho da Unity pode ser portado para a Unreal com ajustes mínimos — desde que os espaços de cor e as faixas de valores estejam corretos.

### Da cena à tela: tone mapping, exposição e pós-processamento

O cálculo do sombreador produz, para cada pixel, um valor de luz que pode ter intensidade muito maior do que a tela é capaz de exibir — pensemos no brilho do sol refletido num metal, ordens de grandeza acima do branco de um monitor. A renderização em tempo real moderna trabalha, por isso, em **alta faixa dinâmica** (*HDR*), com valores de luz fisicamente proporcionais, e precisa, ao final, comprimi-los para a faixa exibível da tela. Esse processo de compressão chama-se **tone mapping**, e é acompanhado de um controle de **exposição**, análogo ao de uma câmera, que decide quão clara ou escura a imagem final ficará.

> ⚠️ **Atenção**
> O tone mapping e a exposição são controles de *cena*, não de material — afetam a imagem inteira. Um mesmo material parecerá mais claro ou mais escuro conforme a exposição, sem que nada nele tenha mudado. Ajustar o material para compensar uma exposição mal calibrada é um erro de diagnóstico.

Sobre a imagem tonalizada, o motor de jogo aplica ainda uma cadeia de **pós-processamento**: efeitos de tela inteira como o *bloom* (o halo luminoso ao redor de fontes muito brilhantes), a correção de cor, a oclusão de ambiente em espaço de tela, a profundidade de campo, o granulado. *Mass Effect 3* (BioWare, 2012) tornou-se referência didática desse efeito: o *bloom* excessivo na entrega original apagava detalhe de materiais cuidadosamente autorados, e capturas comparativas publicadas pela comunidade técnica documentaram como reduzi-lo revelava os mapas de normais, os mapas de rugosidade e os reflexos que o excesso de luminosidade havia obliterado.

> ❌ **Erro Comum**
> Ajustar o material para compensar problemas que pertencem à cadeia de pós-processamento — clarear a cor base para remediar um *bloom* excessivo, aumentar o valor metálico para forçar reflexos que faltavam por ausência de IBL — estraga materiais corretos e gera retrabalho: o material "corrigido" volta a parecer errado em qualquer outra configuração de cena.

### Desempenho: o custo da aparência

Toda essa beleza tem um custo, e nenhum capítulo desta apostila estaria completo sem reencontrar o equilíbrio entre fidelidade e desempenho que é o seu fio condutor. Cada pixel calculado pelo sombreador consome tempo de GPU; quanto mais complexo o sombreador, mais luzes na cena, mais alta a resolução das texturas, mais caro o quadro. Os motores de jogo oferecem técnicas para gerir esse custo — variantes mais simples de sombreador para *assets* distantes, limites ao número de luzes que afetam cada objeto, e a compressão de texturas, que reduz a memória ocupada pelos mapas ao preço de pequenas perdas de qualidade. As decisões de texturização que tomamos ao longo de toda a apostila — a resolução dos mapas, quais mapas incluir, a densidade de texels, a reutilização por trim sheets e atlas — são, em última análise, decisões sobre esse custo.

> 💡 **Dica Profissional**
> Um material extravagante num *asset* que mal se vê é desperdício; um material econômico num *asset* de primeiro plano é mesquinhez. O julgamento maduro distribui o orçamento de desempenho onde ele rende em qualidade percebida, exatamente como o Capítulo 6 distribuía a densidade de texels.

A compressão de texturas merece nota final por fechar o arco do espaço de cor. Os mapas, ao serem importados, são comprimidos em formatos específicos da GPU, e essa compressão é mais ou menos agressiva conforme o tipo de mapa. Um mapa de normais, por carregar vetores sensíveis, exige uma compressão que preserve sua precisão; um mapa de cor tolera mais perda. Configurar corretamente a compressão e o espaço de cor de cada mapa na importação é a etapa concreta em que a distinção entre mapas de cor e de dados, que nos acompanha desde a Parte I, se torna uma escolha de pipeline com consequências diretas na imagem final.

## Aplicação em Jogos

Na produção, sombreador, iluminação e pós-processamento são responsabilidades que se entrelaçam com a texturização sem se confundir com ela. O texturizador autora os materiais; os artistas de iluminação montam as luzes e o IBL; a direção de arte calibra o tone mapping, a exposição e o pós-processamento. Mas essas funções dialogam constantemente, pois nenhuma decide isoladamente a aparência final: um material é aprovado ou rejeitado *sob a iluminação da cena*, e mudanças na iluminação ou no pós-processamento repercutem em todos os materiais. Por isso os estúdios estabelecem cenas de referência iluminadas — ambientes padronizados sob os quais todos os materiais são avaliados —, garantindo que a aprovação de um material num contexto previsível se traduza em boa aparência no jogo.

A consciência de desempenho, por sua vez, permeia todas as decisões. A escolha de quantos mapas usar, em que resolução, com que compressão, sob quantas luzes, define o custo do material em tempo real, e a plataforma de destino — de um console de ponta a um celular modesto — impõe o orçamento. Um mesmo jogo pode exigir versões mais ricas e mais econômicas de seus materiais conforme a plataforma, e antecipar isso é parte do trabalho.

## Estudo de Caso

### Ghost of Tsushima — Sucker Punch Productions, 2020

O visual característico de *Ghost of Tsushima* — campos de capim ondulando no vento, pétalas de cerejeira em redemoinhos, névoa dourada no horizonte — não nasce de texturas excepcionais. Nasce do encontro entre sombreadores customizados e uma direção de iluminação precisa. Sean Feeley e Joakim Stigsson, no talk *"Rendering Ghost of Tsushima"* da GDC 2021, descreveram o problema central que a Sucker Punch enfrentou: os campos de vegetação precisavam comunicar o *vento como personagem* — não como efeito de partícula, mas como força que deforma e anima o mundo inteiro de modo coerente. Isso exigiu um sombreador de vento procedural que, partindo de um campo escalar de direção e intensidade atualizado no CPU a cada quadro, animava independentemente cada palheiro e cada flor segundo sua altura, flexibilidade e posição no campo.

O que transforma esse sombreador de vento num estudo de caso do presente capítulo é justamente a questão que ele levanta: o material da vegetação, visto isoladamente no editor de autoria, é correto mas sem vida — uma malha de capim com textura de folha seca e mapa de rugosidade adequado. Importado para a cena de jogo sem o sombreador de vento e sem a iluminação HDR de final de tarde, seria indistinguível de qualquer campo de capim de jogo convencional. É o sombreador — programado para consumir o campo escalar de vento e deformar as normais das folhas em tempo real — que dá ao capim a sensação de estar sendo empurrado. E é a iluminação HDR, com o sol rasante aquecendo o topo das hastes enquanto a base permanece em sombra fria, que revela o movimento como contraste entre realce e penumbra. Nem o sombreador, nem a iluminação, nem a textura bastam sozinhos: o visual nasce apenas quando os três se encontram.

A decisão mais importante da equipe — documentada no talk — foi **não alterar os materiais da vegetação** para compensar as variações de luz ao longo do dia. Quando os playtesters relataram que o campo parecia "morto" ao meio-dia, sem o sol rasante, a primeira proposta foi clarear a cor base das folhas e adicionar autoemissão para simular o brilho. A Sucker Punch recusou a solução, porque ela teria produzido vegetação que brilhava mesmo no interior de uma caverna. A solução correta foi ajustar o ciclo de luz do jogo para que o sol nunca ficasse diretamente acima — uma decisão de direção de arte sobre a iluminação, não sobre o material. O material permaneceu fisicamente correto; o mundo foi ajustado para que a iluminação o revelasse sempre de modo favorável.

**O que aprender com isso:** Quando um material parece incorreto na cena, o problema pode estar na iluminação, não no material. Antes de alterar qualquer mapa, a equipe deve diagnosticar se a causa pertence ao material ou à cadeia de cena — e respeitar essa distinção evita retrabalho e preserva a correção física do material.

> **Figura 11.2** — Campo de capim em *Ghost of Tsushima* com vento e luz de entardecer. O mesmo material de vegetação, visto num editor de autoria sob IBL neutro, não produziria essa imagem — o visual nasce do sombreador de deformação por vento + iluminação HDR rasante + tone mapping calibrado para a hora dourada.

**Screenshot sugerido:** Captura de tela oficial de *Ghost of Tsushima* mostrando campo de capim com iluminação rasante dourada; ao lado, o mesmo material visto no editor de autoria sob IBL neutro, evidenciando a diferença produzida pela iluminação e pelo sombreador.

## Boas Práticas

A boa prática que encerra a Parte III é avaliar e corrigir materiais sempre em conjunto com a iluminação sob a qual aparecerão, nunca isoladamente. Use ambientes de IBL representativos durante a autoria, verifique cada material no motor de jogo de destino sob a iluminação real da cena e, diante de uma aparência indesejada, distinga com cuidado o que pertence ao material do que pertence à iluminação, à exposição ou ao pós-processamento — evitando o erro de ajustar o material para compensar problemas que são da cena. Compreenda que o sombreador consome os mapas como uma fórmula consome números, sem corrigir enganos, e que esse é o motivo último de todo o rigor de autoria: cor base sem luz, mapa de rugosidade com variação, espaço de cor correto.

É boa prática manter consciência da cadeia final — tone mapping, exposição, pós-processamento — e do fato de que ela afeta todos os materiais por igual, sendo de cena e não de material. Recomenda-se configurar corretamente, na importação, a compressão e o espaço de cor de cada mapa, tratando os mapas de normais e demais dados com a precisão que exigem e reservando o sRGB apenas para os de cor. E recomenda-se, acima de tudo, raciocinar sobre o desempenho como parte da aparência: dosar a resolução, o número de mapas, a complexidade do sombreador e o número de luzes segundo a importância do *asset* e o orçamento da plataforma, distribuindo o custo onde ele mais rende em qualidade percebida.

## Erros Comuns

O erro mais característico deste estágio é avaliar ou corrigir um material isoladamente, sem a iluminação sob a qual ele aparecerá, e em seguida ajustá-lo para compensar problemas que são, na verdade, da cena — clarear uma cor base para remediar uma exposição baixa, reduzir o mapa de rugosidade para forçar brilho que faltava por ausência de IBL. Esse erro estraga materiais corretos e gera retrabalho, pois o material assim "corrigido" volta a parecer errado em qualquer outra iluminação. Próximo dele está o de julgar metais e superfícies lisas sem um ambiente para refletir, concluindo que estão sem vida quando lhes falta apenas algo a refletir.

Outro equívoco é ignorar a distinção entre o que pertence ao material e o que pertence à cena — confundir os efeitos de tone mapping, exposição e pós-processamento com propriedades do material, e tentar resolver no material o que se resolve na direção de arte da cena. Há o erro técnico de configurar mal a compressão e o espaço de cor na importação, comprimindo um mapa de normais como se fosse cor e degradando-o, ou marcando como linear um mapa que deveria ser sRGB. E persiste o erro de desconsiderar o desempenho — empregar sombreadores caros, texturas em resolução excessiva e luzes em profusão sobre *assets* que não os justificam, esgotando o orçamento da plataforma em detalhes que o jogador mal percebe.

## Resumo

Este capítulo final da Parte III mostrou como o material se torna a imagem que o jogador vê. Nomeamos o executor do cálculo de aparência que acompanhamos desde a Parte I: o sombreador, programa que roda na GPU e, para cada pixel, combina os mapas do material com a luz da cena segundo o modelo físico do PBR — consumindo os mapas como uma fórmula consome números, sem corrigir enganos, o que justifica retrospectivamente todo o rigor de autoria. Estudamos a iluminação como a outra metade da equação: os tipos de luz direta e indireta e, sobretudo, a iluminação baseada em imagem (IBL), da qual os metais e superfícies lisas extraem seus reflexos, concluindo que um material PBR não pode ser avaliado sem uma iluminação representativa. Apresentamos, sem suas fórmulas, a BRDF como o coração matemático do sombreador, onde os princípios físicos e os mapas se encontram, e como o modelo comum que torna os materiais transferíveis entre motores de jogo. Percorremos a cadeia que leva da cena à tela — tone mapping, exposição e pós-processamento —, distinguindo o que pertence à cena do que pertence ao material e alertando contra o erro de ajustar um pelo outro. E fechamos com o desempenho, reencontrando o equilíbrio entre fidelidade e custo que é o fio condutor da apostila, e mostrando que a configuração de compressão e espaço de cor na importação é o ponto em que a distinção entre mapas de cor e de dados se torna escolha concreta de pipeline.

## Exercícios

**1.** Explique o que é um sombreador e qual é o seu papel no "cálculo de aparência" mencionado desde a Parte I. Por que a afirmação de que "o sombreador consome os mapas como uma fórmula consome números, sem corrigir enganos" justifica retrospectivamente todo o rigor exigido na autoria dos mapas?

**2.** Explique por que um material PBR não pode ser avaliado sem uma iluminação representativa, recorrendo ao exemplo de uma superfície metálica e ao papel do IBL. Que erro de produção decorre de julgar um metal sob iluminação pobre, e como evitá-lo?

**3.** Distinga o que pertence ao *material* do que pertence à *cena* (iluminação, exposição, tone mapping, pós-processamento). Dado o sintoma "o material parece escuro e sem reflexos na cena, embora estivesse correto na autoria", descreva como diagnosticar se a causa está no material ou na cena, e por que ajustar o material seria, neste caso, um erro.

**4.** Análise: explique, sem usar fórmulas, o que a BRDF representa e como ela incorpora os princípios de conservação de energia, Fresnel e rugosidade do Capítulo 8. Por que o fato de diferentes motores de jogo implementarem BRDFs semelhantes é o que torna os materiais PBR transferíveis entre eles?

**5.** Planejamento integrado: você precisa entregar o mesmo *asset* de primeiro plano para duas plataformas — um console potente e um celular modesto. Discuta como as decisões de sombreador, resolução de texturas, número de mapas, compressão e número de luzes mudariam entre as duas versões, relacionando suas escolhas ao equilíbrio entre fidelidade e desempenho e aos conceitos de densidade de texels e reutilização das partes anteriores.

## Glossário

**BRDF (função de distribuição de reflectância bidirecional):** Modelo matemático que descreve como a luz se reflete numa superfície dados os ângulos de incidência e observação; coração formal do sombreador PBR.

**Canal de textura:** Cada um dos mapas de informação que compõem um material PBR — cor base, mapa de rugosidade, mapa metálico, mapa de normais etc. — consumido separadamente pelo sombreador.

**Densidade de texels:** Relação entre o tamanho físico de uma superfície no espaço 3D e a quantidade de pixels de textura que a recobre, expressa em pixels por centímetro ou por metro.

**Espaço de cor:** Sistema que define como os valores numéricos de uma imagem correspondem a cores percebidas. Mapas de cor usam sRGB; mapas de dados usam espaço linear.

**IBL (iluminação baseada em imagem):** Técnica em que uma imagem panorâmica do ambiente serve como fonte de luz envolvente, determinando os reflexos em superfícies metálicas e lisas.

**Mapa de normais:** Canal de textura que armazena direções de superfície codificadas como cores RGB, permitindo simular relevo sem geometria adicional.

**Mapa de rugosidade:** Canal de textura que controla a dispersão da luz na superfície — valores baixos resultam em reflexos nítidos; valores altos em aparência fosca.

**Motor de jogo:** Plataforma de software (Unreal Engine, Unity, Godot etc.) que gerencia a renderização em tempo real, incluindo a execução dos sombreadores e o cálculo de iluminação.

**Pós-processamento:** Cadeia de efeitos aplicados à imagem renderizada após o cálculo do sombreador — *bloom*, correção de cor, granulado etc. — pertencentes à cena, não ao material.

**Sombreador (*shader*):** Programa executado pela GPU que calcula a cor de cada pixel visível combinando os mapas do material com a iluminação da cena segundo o modelo PBR.

**Tone mapping:** Processo de compressão dos valores de alta faixa dinâmica (HDR) do renderizador para a faixa exibível da tela, afetando todos os materiais da cena por igual.

## Leituras Complementares

- **[*The PBR Guide* — Allegorithmic / Adobe, 2018]** — Discute o modelo de iluminação do PBR, a renderização em espaço linear e a relação entre os mapas e o cálculo do sombreador; base conceitual direta deste capítulo. Recomendado para aprofundar a relação entre os valores dos mapas e as fórmulas do sombreador.

- **[AKENINE-MÖLLER; HAINES; HOFFMAN. *Real-Time Rendering*, 4. ed. CRC Press, 2018]** — Os capítulos sobre *shading*, BRDFs, iluminação baseada em imagem, HDR e tone mapping são a referência central para todo o conteúdo deste capítulo. Leitura indispensável para quem deseja fundamentação matemática sólida.

- **[*Learn OpenGL* — https://learnopengl.com/]** — Seções sobre PBR, iluminação baseada em imagem e HDR/tone mapping. Apresentação prática de como o sombreador, o IBL e o tone mapping são implementados, conectando os conceitos à sua realização técnica.

- **[*The Book of Shaders* — https://thebookofshaders.com/?lan=pt]** — Introdução acessível à programação de sombreadores para quem quiser compreender, por dentro, o programa que calcula a aparência. Ponto de partida recomendado antes de explorar a documentação dos motores.

- **[*Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/]** — Seções sobre materiais e sombreadores, tipos de luz, *reflection probes* e IBL, pós-processamento, exposição/tone mapping e compressão de texturas. Útil para observar como a Unreal concretiza os conceitos deste capítulo.

- **[*Unity Manual* — https://docs.unity3d.com/]** — Seções equivalentes para o pipeline URP e HDRP da Unity, incluindo sombreadores, iluminação e compressão de mapas. Recomendado em paralelo à documentação da Unreal para comparar abordagens.
