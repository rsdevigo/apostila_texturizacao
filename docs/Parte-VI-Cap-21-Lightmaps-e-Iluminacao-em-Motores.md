# Capítulo 21 — Lightmaps e Iluminação em Motores de Jogo

## Introdução

As cinco partes anteriores conduziram um *asset* desde a representação visual de seus materiais (Parte I) até sua forma final, eficiente e adequada ao *hardware* (Parte V). Em todo esse percurso, uma premissa foi tratada como dada, mas nunca examinada de frente: a de que a aparência de uma superfície resulta do encontro entre seus mapas e a **luz** que incide sobre ela. O Capítulo 11 já havia estabelecido que o material só se completa quando iluminado, que a mesma textura parece uma coisa sob o sol do meio-dia e outra sob a penumbra de um interior. Esta sexta e última parte da apostila, dedicada à integração e ao *pipeline* profissional, começa por aquele que é o elo entre a textura e a luz no motor de jogo: o sistema de iluminação, e em particular os **lightmaps**, as texturas que guardam a luz pré-calculada de uma cena.

Há uma razão para abrir a Parte VI por este tema. Tudo o que se ensinou sobre texturização supõe um contexto de iluminação que o texturizador raramente controla sozinho, mas que precisa compreender para que seu trabalho se revele como pretendido. Um material PBR fisicamente calibrado, com seus mapas de cor, rugosidade e metalicidade corretos, pode parecer chapado, sujo ou irreal se a iluminação da cena estiver mal resolvida — e, inversamente, uma iluminação bem construída valoriza até materiais modestos. Compreender como o motor calcula e armazena a luz, e como os lightmaps exigem um tipo particular de coordenada de textura, é parte do repertório do profissional de jogos, mesmo quando a iluminação propriamente dita cabe a outra função da equipe. Este capítulo apresenta os conceitos fundamentais da iluminação em tempo real, distingue suas estratégias e detalha o papel dos lightmaps e da segunda camada de UVs que eles requerem, conectando o tema às UVs da Parte II e ao PBR da Parte III.

## Desenvolvimento

### O problema do custo da luz em tempo real

Calcular como a luz se comporta em uma cena é caro. A luz não apenas vem diretamente das fontes — o sol, uma lâmpada, uma fogueira — e atinge as superfícies; ela também **ricocheteia**, rebatendo de uma superfície a outra, tingindo-se das cores que toca, preenchendo de claridade suave as áreas que nenhuma fonte ilumina diretamente. Essa luz indireta, ou **iluminação global** (*global illumination*), é o que faz uma parede branca próxima a um tapete vermelho ganhar um tom rosado, e o que impede que as sombras sejam negros absolutos. Simular esse ricochete fisicamente, em tempo real, a sessenta quadros por segundo, foi durante muito tempo proibitivamente caro — e ainda hoje, embora as técnicas de iluminação global em tempo real tenham avançado enormemente, há um custo de desempenho que nem toda plataforma comporta.

> 📘 **Definição**
> **Iluminação global** (*global illumination*) é o conjunto de efeitos de iluminação que resultam do ricochete da luz entre superfícies — a luz indireta que preenche áreas fora da linha de visão das fontes, as tintas de cor que as superfícies trocam entre si e as sombras suaves produzidas pela geometria estática. Simulá-la em tempo real é o principal desafio de desempenho que os lightmaps foram criados para resolver.

Diante disso, a indústria desenvolveu uma estratégia que ecoa, no domínio da luz, a mesma lógica do *baking* estudada no Capítulo 15: **pré-calcular** o que é caro e **armazenar o resultado** para leitura barata em tempo real. Assim como o *baking* transfere o detalhe de uma malha rica para uma textura que a malha leve apenas consulta, a iluminação pré-calculada resolve, antes do jogo rodar, o custoso ricochete da luz sobre a geometria estática, e guarda o resultado em texturas que o motor apenas lê durante a partida. Essas texturas são os **lightmaps**. A escolha entre pré-calcular a luz, calculá-la em tempo real ou combinar as duas abordagens é uma das decisões arquiteturais mais importantes de um jogo, e governa tudo o que se segue.

### Luz estática, dinâmica e mista

Os motores de jogo organizam a iluminação em torno de uma distinção fundamental entre o que **não se move** e o que **se move**. A geometria **estática** — paredes, pisos, terreno, edifícios, tudo o que permanece fixo na cena — pode ter sua iluminação inteiramente pré-calculada, porque a relação entre as fontes de luz e as superfícies nunca muda. É para essa geometria que os lightmaps existem. Já a geometria **dinâmica** — personagens, veículos, objetos que o jogador manipula — não pode usar luz pré-calculada da mesma forma, porque se desloca pela cena, e a luz que a atinge muda a cada instante; ela depende de iluminação calculada em tempo real, complementada por técnicas que capturam a luz do ambiente ao redor de cada objeto móvel.

Disso resultam três estratégias clássicas, que os motores nomeiam de modos próprios mas que correspondem à mesma lógica. A iluminação **estática** (ou *baked*) pré-calcula tudo e oferece a melhor qualidade de luz indireta ao menor custo em tempo real, ao preço de não reagir a nenhuma mudança — uma luz pré-calculada não pode ser apagada, nem projetar a sombra de uma porta que se abre. A iluminação **dinâmica** (ou em tempo real) calcula tudo durante o jogo, reage a qualquer mudança, mas custa caro em desempenho e, historicamente, oferecia luz indireta mais pobre. A iluminação **mista** (*mixed*/*stationary*) combina as duas: a luz direta e as sombras de objetos móveis são calculadas em tempo real, enquanto a luz indireta e as sombras da geometria estática vêm pré-calculadas — um equilíbrio que domina a produção de jogos há mais de uma década.

> 💡 **Dica Profissional**
> O texturizador precisa saber a qual regime de iluminação pertence o *asset* que produz, pois isso determina se ele exigirá lightmaps e, portanto, um cuidado adicional com suas UVs. Essa informação deve ser combinada com o artista de iluminação ou o *level artist* antes de o desdobramento UV ser concluído.

*Fortnite* (Epic Games, 2017) documentou esse equilíbrio em escala de mundo em constante mutação: a ilha e as estruturas permanentes recebem iluminação pré-calculada em lightmaps, enquanto as construções que os jogadores erguem e destroem a cada partida dependem de sondas de luz (*light probes*) para capturar a iluminação do ambiente e aplicá-la dinamicamente, sem forçar o recálculo do lightmap inteiro a cada rodada.

> **Figura 21.1** — Um mesmo ambiente interior renderizado sob os três regimes — totalmente estático (luz indireta rica, mas tudo imóvel), totalmente dinâmico (sombras que reagem a um objeto em movimento, mas luz indireta mais pobre) e misto (sombras dinâmicas do objeto móvel sobre a luz indireta pré-calculada da sala) —, evidenciando o que cada estratégia ganha e perde.

**Screenshot sugerido:** Captura das configurações de modo de iluminação (estático/estacionário/dinâmico) de uma luz direcional em um motor, ao lado da cena resultante.

### O que é um lightmap

> 📘 **Definição**
> **Lightmap** é uma textura que armazena a iluminação incidente sobre as superfícies estáticas de uma cena, pré-calculada antes do jogo rodar. Em vez de recalcular, a cada quadro, quanta luz e de que cor atinge cada ponto de uma parede, o motor de jogo consulta o lightmap — exatamente como consultaria qualquer outro canal de textura — e lê ali o valor já resolvido, incluindo a iluminação global indireta e as sombras suaves produzidas pela geometria estática.

O lightmap é, nesse sentido, uma fotografia da luz: tira-se uma vez, no processo chamado *baking* de iluminação, e usa-se durante todo o jogo.

É crucial entender que o lightmap **não substitui** os mapas do material — não há concorrência entre o lightmap e o mapa de cor base ou o mapa de rugosidade. Eles operam em camadas distintas e complementares: os mapas do material descrevem as **propriedades da superfície** (sua cor, sua rugosidade, seu relevo), enquanto o lightmap descreve a **luz que incide** sobre ela. Na renderização, o motor combina os dois — multiplica, grosso modo, a cor do material pela luz do lightmap — para produzir a aparência final. Por isso um material pode ser reutilizado em mil paredes com um único conjunto de texturas, enquanto cada parede tem sua própria região de lightmap, com sua própria luz e suas próprias sombras. Essa separação entre a descrição da superfície (compartilhável) e a descrição da luz (única por instância) é a chave para entender por que os lightmaps exigem um segundo conjunto de coordenadas.

> ⚠️ **Atenção**
> Reduzir a qualidade dos mapas de material com o argumento de que "o lightmap resolve a iluminação" é um equívoco grave. O lightmap descreve a luz; os mapas do material descrevem a superfície. Materiais mal calibrados em PBR permanecerão incorretos mesmo sob um lightmap de alta qualidade, pois os dois operam em camadas complementares, não substitutas.

### A segunda UV: por que o lightmap precisa de seu próprio mapeamento

Aqui o tema reencontra diretamente a Parte II. As UVs estudadas no desdobramento UV (Capítulos 4 e 5) servem ao material: elas determinam como o mapa de cor, o mapa de normais e os demais canais de textura se assentam sobre a malha, e foram organizadas para distribuir bem a densidade de texels, com costuras escondidas e ilhas que podem se sobrepor ou se repetir livremente — pois uma textura de tijolos pode, e deve, se repetir sobre uma fachada inteira. O lightmap, porém, tem uma exigência incompatível com isso: como ele guarda a luz **única** de cada ponto da superfície — esta esquina da parede está na sombra, aquela recebe o sol —, **nenhuma região da malha pode compartilhar o mesmo espaço de textura no lightmap**. Duas paredes que reutilizam a mesma textura de tijolos não podem reutilizar o mesmo trecho de lightmap, pois recebem luzes diferentes.

Por isso a geometria estática que recebe lightmap precisa de um **segundo canal de UV** — uma segunda camada de coordenadas, distinta da que serve ao material —, projetada segundo regras próprias:

- Todas as ilhas **sem sobreposição alguma** (cada ponto da superfície ocupa um lugar exclusivo no espaço de textura);
- **Margens generosas** entre ilhas para que o ricochete da luz e os mipmaps não vazem de uma ilha para outra (o mesmo princípio de *padding* visto nos Capítulos 15 e 18);
- Distribuição de espaço proporcional à importância de cada superfície na captação de luz e sombra.

> ❌ **Erro Comum**
> Reaproveitar o canal de mapeamento UV do material para o lightmap é um dos erros mais frequentes na preparação de *assets* estáticos. Como o canal do material costuma ter ilhas sobrepostas e repetidas para reutilizar texturas, usá-lo para lightmap faz com que regiões distintas da malha compartilhem a mesma luz, gerando sombras absurdas e manchas inexplicáveis.

Os motores de jogo oferecem geração automática desse segundo canal, e ela basta para muitos *assets*; mas superfícies grandes e importantes — o piso de um salão, a fachada de um edifício — frequentemente exigem um desdobramento UV manual do canal de lightmap para que as sombras fiquem nítidas e sem artefatos. Compreender que existe essa segunda camada de UVs, e por que ela obedece a regras opostas às da primeira, é uma das competências que distinguem o profissional que entende o *pipeline* inteiro.

> **Figura 21.2** — Um mesmo modelo com seus dois canais de UV lado a lado — à esquerda, o canal do material, com ilhas que se sobrepõem e se repetem para reutilizar texturas; à direita, o canal de lightmap, com todas as ilhas separadas, sem sobreposição e com margens amplas —, ilustrando por que os dois mapeamentos obedecem a regras opostas.

**Screenshot sugerido:** Captura do editor de UVs mostrando os dois canais do mesmo *asset*, com destaque para a ausência de sobreposição no canal de lightmap.

### Resolução de lightmap e o orçamento da luz

Como todo canal de textura, o lightmap tem uma **resolução**, e ela governa a nitidez das sombras pré-calculadas. Um lightmap de baixa resolução produz sombras borradas e manchas suaves; um de alta resolução produz sombras nítidas, ao custo de memória — pois os lightmaps, somados sobre toda a geometria estática de um nível, podem ocupar uma fatia considerável da memória de vídeo, aquela mesma escassez dimensionada no Capítulo 20. A resolução de lightmap é atribuída **por** *asset* ou por superfície, e equilibra qualidade contra custo segundo a importância visual de cada elemento: o piso de uma arena de combate, onde o jogador passa horas, merece lightmap denso; uma parede distante e sempre na penumbra pode ter lightmap mínimo.

> 💡 **Dica Profissional**
> Reaparece aqui, transposta para a luz, a mesma lógica de **densidade de texels** da Parte II e de **orçamento de plataforma** da Parte V: distribuir um recurso finito conforme a relevância, gastando resolução onde o olho repara e poupando onde não repara. A diferença é que, no lightmap, o recurso escasso não é a memória de textura do material, mas a memória total de iluminação do nível.

O *baking* de iluminação é, ele próprio, um processo demorado — pode levar minutos ou horas para uma cena complexa; a Naughty Dog relatou que o *baking* de *The Last of Us Part II* (2020) era executado em uma fazenda de renderização com mais de 256 núcleos de processamento, em ciclos de até 12 horas por cena de alta qualidade —, o que tem consequências práticas para a produção: cada vez que a geometria estática ou as luzes mudam, o lightmap precisa ser recalculado, e isso impõe um ritmo ao trabalho de montagem de níveis. É em parte para escapar dessa lentidão que as técnicas de iluminação global em **tempo real** ganharam força nos motores recentes, prometendo luz indireta de qualidade sem o passo de *baking*; mas elas exigem *hardware* mais capaz, e a iluminação pré-calculada por lightmaps permanece a escolha padrão para garantir desempenho em plataformas modestas.

> ⚠️ **Atenção**
> Sempre que a geometria estática ou as fontes de luz de uma cena forem alteradas, o lightmap precisa ser recalculado. Deixar o nível com lightmap desatualizado produz artefatos visíveis: sombras de paredes que já foram removidas, claridade onde agora existe um obstáculo, manchas de luz sem origem. Em produção, congelar a geometria estática e as luzes antes do *baking* final é prática obrigatória.

O profissional competente conhece as duas vias — iluminação pré-calculada e em tempo real — e entende que a decisão entre elas — como tantas nesta apostila — é um equilíbrio entre fidelidade, custo e as restrições da plataforma de destino.

## Aplicação em Jogos

Na produção, a iluminação de um nível é tipicamente responsabilidade de um artista de iluminação ou de um *level artist*, mas o trabalho do texturizador e do artista de *assets* é condição para que ela funcione. Cada *asset* estático destinado a uma cena com lightmaps precisa chegar ao motor com um **segundo canal de UV** adequado — sem sobreposição e com margens —, sob pena de produzir sombras vazadas, manchadas ou serrilhadas que comprometem todo o trabalho de iluminação. Muitos defeitos visuais que parecem problemas de luz são, na verdade, problemas de mapeamento UV de lightmap mal resolvida, e diagnosticá-los exige entender essa segunda camada. Por isso equipes mantêm convenções rígidas: resolução de lightmap padronizada por tipo de *asset*, canal de UV de lightmap verificado na revisão, margens mínimas garantidas.

A escolha do regime de iluminação molda o jogo inteiro. Títulos de mundo aberto com ciclo de dia e noite não podem depender só de luz pré-calculada, pois a posição do sol muda — recorrem a iluminação dinâmica ou a soluções híbridas que misturam lightmaps com ajustes em tempo real. Jogos de ambientes fechados e fixos, ao contrário, exploram ao máximo a iluminação estática, obtendo luz indireta riquíssima a baixo custo. *Fortnite*, ao migrar para a Unreal Engine 5 em 2022, tornou-se o caso de referência do Lumen (o sistema de iluminação global em tempo real da Epic), demonstrando em escala comercial de dezenas de milhões de jogadores que a iluminação indireta calculada em tempo real pode substituir o lightmap em *hardware* moderno — referência indispensável para quem precisa tomar essas decisões na prática.

## Estudo de Caso

### *Firewatch* (Campo Santo, 2016) — Iluminação Pré-Calculada e a Arte de Fazer o Mundo Aberto Parecer Pintado

*Firewatch* é um caso raro na indústria: um jogo de mundo aberto de equipe pequena — a Campo Santo tinha apenas um punhado de pessoas — que alcançou uma qualidade visual de luz e atmosfera que rivalizou com produções de estúdios muito maiores. O segredo técnico, documentado pela artista Jane Ng em um blog post técnico de 2016 que se tornou referência da área, estava precisamente na gestão de lightmaps: o jogo usa iluminação pré-calculada de maneira sofisticada para entregar a luz indireta rica dos bosques de Wyoming — o calor do final da tarde filtrando pelas copas, as sombras suaves da folhagem no solo, a luz refletida nos troncos — a um custo computacional que permitia rodar no PS4 sem comprometer a taxa de quadros.

A decisão arquitetural central foi tratar o ambiente exterior de *Firewatch* como geometria predominantemente estática — as árvores, as rochas, o terreno, as estruturas da torre de vigia — e investir o orçamento de lightmap em capturar a iluminação com alta resolução nas superfícies que o jogador mais percorre. Jane Ng documentou a distribuição deliberada de resolução de lightmap: o solo dos caminhos principais, onde Henry caminha e onde o jogador olha mais, recebe densidades muito superiores às de penhascos distantes ou copas de árvore. Essa priorização é a transposição exata da lógica de orçamento de texels da Parte II para o domínio da luz: gastar resolução onde o olho pousa, poupar onde não pousa. O resultado é que o limitado orçamento de lightmap da equipe aparece todo nos lugares certos, produzindo a impressão de que o mundo inteiro é densamente iluminado quando, na verdade, apenas as partes visíveis o são com precisão.

O que *Firewatch* ilustra com clareza adicional é a atenção ao canal de UV de lightmap como condição de qualidade. Ng descreveu a importância de desenrolar manualmente o canal de lightmap das árvores e das rochas mais visíveis — os *assets* que mais contribuem para a identidade visual do jogo —, garantindo ilhas sem sobreposição e margens adequadas para que as sombras das copas não vazassem nas superfícies abaixo. O aspecto estético do jogo — aquela qualidade quase de ilustração, de luz pintada — não é um filtro de pós-processamento; é consequência direta de lightmaps bem resolvidos, com resolução distribuída com inteligência e UVs de lightmap geridas com cuidado, demonstrando que a excelência visual de iluminação é, em grande parte, excelência técnica na gestão de suas texturas.

**O que aprender com isso:** A qualidade visual da iluminação de um ambiente depende tanto das decisões artísticas quanto das decisões técnicas de mapeamento UV e distribuição de resolução de lightmap. Um lightmap bem planejado é, em essência, um orçamento de luz gerido com o mesmo rigor com que se gere o orçamento de densidade de texels.

> **Figura 21.3** — Comparação de uma seção da floresta de *Firewatch* com a visualização do mapa de densidade de lightmap sobreposta, mostrando a concentração de resolução nos caminhos e nas superfícies próximas ao jogador e a redução progressiva nas copas distantes e nos fundos.

**Screenshot sugerido:** Captura panorâmica de *Firewatch* no horário dourado (entardecer), evidenciando a qualidade da luz indireta nas superfícies florestais. Fonte: modo foto ou galeria oficial em firewatchgame.com.

**Fonte:** Jane Ng, "The Lighting of Firewatch" (blog post técnico, Campo Santo, 2016); palestrante convidada em Unity Unite 2016; análises técnicas em 80.lv e Digital Foundry.

## Boas Práticas

Trate o **segundo canal de UV** como parte obrigatória de todo *asset* estático destinado a cenas com lightmaps: ilhas sem sobreposição alguma, margens generosas entre elas para conter o vazamento de luz e os mipmaps, e distribuição de espaço proporcional à importância da superfície na captação de luz e sombra. Aceite a geração automática do canal de lightmap para *assets* pequenos e secundários, mas realize o desdobramento UV manualmente para superfícies grandes e protagonistas, onde sombras nítidas e sem artefatos importam. Calibre a **resolução de lightmap** por *asset* segundo sua relevância visual, gastando-a onde o jogador repara e poupando-a onde não repara — a mesma lógica de orçamento que governa a densidade de texels e a compressão.

Saiba a qual **regime de iluminação** pertence cada *asset*: a geometria estática usa lightmaps; a dinâmica depende de iluminação em tempo real e de sondas que captam a luz do ambiente; e o regime misto, que combina as duas, é a escolha mais comum. Lembre-se de que o lightmap descreve a **luz**, não a superfície, e que ele se combina aos mapas do material sem substituí-los — de modo que materiais corretamente calibrados em PBR (Parte III) continuam essenciais, pois é sobre eles que a luz pré-calculada incide. E mantenha presente que o *baking* de iluminação é demorado: planeje a montagem do nível para minimizar recálculos desnecessários, congelando a geometria estática e as luzes antes do *baking* final.

## Erros Comuns

O erro mais característico do capítulo é entregar um *asset* estático **sem canal de UV de lightmap adequado** — ausente, com ilhas sobrepostas (herdadas do canal do material) ou com margens insuficientes —, o que produz sombras vazadas, manchadas ou serrilhadas que parecem defeitos de iluminação mas são, na origem, defeitos de mapeamento UV. Aparentado a ele está o erro de reaproveitar o canal do material para o lightmap: como o canal do material costuma ter ilhas sobrepostas e repetidas para reutilizar texturas, usá-lo para lightmap faz com que regiões distintas da malha compartilhem a mesma luz, gerando sombras absurdas.

Há o erro de **resolução de lightmap mal calibrada**: baixa demais em superfícies importantes (sombras borradas e manchas) ou alta demais em todo lugar (memória desperdiçada), por não distribuir o recurso conforme a relevância. Há o erro de esperar que a geometria **dinâmica** se ilumine como a estática, deixando personagens "colados" e apagados sobre o cenário por não usar sondas de luz ou iluminação em tempo real. Há o equívoco conceitual de tratar o lightmap como se concorresse com os mapas do material — reduzindo a qualidade das texturas porque "a luz resolve" —, quando os dois operam em camadas complementares. E há o erro de processo de refazer o *baking* sem necessidade, ou de esquecer de refazê-lo após mudar a geometria estática, deixando o nível com luz desatualizada — sombras de paredes que já não existem, claridade onde agora há um obstáculo.

## Resumo

Este capítulo abriu a Parte VI tratando do elo entre a textura e a luz no motor de jogo. Partiu-se do custo proibitivo de simular, em tempo real, o ricochete da luz indireta (a iluminação global), e da estratégia — análoga ao *baking* da Parte V — de **pré-calcular** esse custo e armazená-lo em texturas: os **lightmaps**. Distinguiram-se os três regimes de iluminação — **estático** (tudo pré-calculado, máxima qualidade indireta ao menor custo, mas imóvel), **dinâmico** (tudo em tempo real, reativo mas caro) e **misto** (a combinação que domina a produção) — e mostrou-se que cada um corresponde a um tipo de geometria, estática ou dinâmica. Definiu-se o lightmap como uma fotografia da luz que **complementa**, sem substituir, os mapas do material, operando em camada distinta. E detalhou-se a exigência central do tema: como o lightmap guarda a luz única de cada ponto, a geometria estática precisa de um **segundo canal de UV**, com regras opostas às do canal do material — ilhas sem sobreposição alguma, margens generosas contra o vazamento de luz, espaço proporcional à importância —, reencontrando aqui as UVs da Parte II e o *padding* da Parte V. Por fim, tratou-se da **resolução de lightmap** como mais um recurso a orçar conforme a relevância visual, e do custo de tempo do *baking* de iluminação, que motiva o avanço das soluções de iluminação global em tempo real. Compreender esse sistema — ainda que a iluminação caiba a outra função da equipe — é condição para que o trabalho de texturização se revele como pretendido, pois é a luz que torna o material visível. Os capítulos seguintes desta parte mostrarão como esse e os demais conhecimentos se integram, primeiro nos motores específicos e depois no *pipeline* completo.

## Exercícios

**1.** Explique, com suas palavras, por que a iluminação global (o ricochete da luz indireta) é cara em tempo real e como os lightmaps respondem a esse custo. Relacione explicitamente essa estratégia ao *baking* de texturas do Capítulo 15: o que há de comum na lógica das duas técnicas?

**2.** Um lightmap e um mapa de cor base são ambos canais de textura, mas descrevem coisas diferentes e operam em camadas distintas. Explique o que cada um descreve, como se combinam na renderização e por que um material pode ser reutilizado em mil instâncias enquanto cada instância tem sua própria região de lightmap.

**3.** A geometria estática que recebe lightmap exige um **segundo canal de UV** com regras opostas às do canal do material. Liste essas regras (sobreposição, repetição, margens) para os dois canais lado a lado e explique por que a sobreposição, desejável no canal do material, é proibida no de lightmap.

**4.** Distinga os três regimes de iluminação (estático, dinâmico e misto) quanto ao que cada um pré-calcula e ao que calcula em tempo real, e indique a qual tipo de geometria — estática ou dinâmica — cada regime melhor serve. Em que situação de jogo o regime totalmente estático seria inadequado, e por quê?

**5.** Diagnóstico: em um nível com iluminação pré-calculada, (a) as colunas têm sombras borradas e manchadas, (b) a luz vaza nas quinas de uma sala para áreas que deveriam estar na sombra, e (c) os personagens parecem "colados" e apagados sobre o cenário bem iluminado. Para cada sintoma, identifique a causa provável entre os conceitos do capítulo e descreva a correção.

## Glossário

**Canal de UV de lightmap:** Segunda camada de coordenadas de mapeamento UV aplicada à malha de um *asset* estático, destinada exclusivamente a receber o lightmap. Distingue-se do canal de mapeamento UV do material por exigir ilhas sem qualquer sobreposição e com margens generosas entre elas.

**Densidade de texels (no lightmap):** Relação entre a resolução do lightmap e a área da superfície que ele cobre. Uma densidade elevada produz sombras nítidas e detalhadas; uma densidade baixa resulta em sombras borradas. Deve ser distribuída conforme a importância visual e a proximidade do ponto de visualização.

**Iluminação global:** Conjunto de efeitos de iluminação resultantes do ricochete da luz entre superfícies — luz indireta, tingimento de cor por reflexo e sombras suaves. É o efeito mais custoso de calcular em tempo real e o principal motivo de existência dos lightmaps.

**Lightmap:** Canal de textura que armazena a iluminação pré-calculada incidente sobre as superfícies estáticas de uma cena. Lido pelo motor de jogo em tempo real como qualquer outro canal de textura, complementando os mapas do material sem substituí-los.

**Regime de iluminação estático (*baked*):** Estratégia em que toda a iluminação — direta, indireta, sombras — é pré-calculada e armazenada em lightmaps. Oferece a maior qualidade de luz indireta ao menor custo de desempenho, mas não reage a mudanças na cena.

**Regime de iluminação dinâmico:** Estratégia em que toda a iluminação é calculada em tempo real durante o jogo. Reage a qualquer mudança, mas tem custo de desempenho elevado e, historicamente, oferecia luz indireta mais pobre do que a pré-calculada.

**Regime de iluminação misto (*mixed/stationary*):** Estratégia que combina iluminação pré-calculada (luz indireta e sombras da geometria estática) com iluminação em tempo real (luz direta e sombras de objetos dinâmicos). É o equilíbrio dominante na produção de jogos.

**Sonda de luz (*light probe*):** Objeto posicionado na cena que captura a iluminação do ambiente ao redor de um ponto e a aplica a objetos dinâmicos que passam por ele. Permite que a geometria dinâmica receba influência da iluminação ambiente sem depender de lightmaps.

## Leituras Complementares

- **[AOD — Texel Density]** (material de apoio da disciplina) — A lógica de distribuir resolução de textura conforme a importância da superfície, ali aplicada aos materiais, transpõe-se diretamente para a resolução de lightmap discutida neste capítulo. Leitura complementar indispensável para compreender o conceito de orçamento de resolução.

- **[The PBR Guide]** (Wes McDermott; Allegorithmic / Adobe, 2018) — Fundamenta a relação entre material e luz — o material só se revela sob iluminação — que sustenta a compreensão do papel complementar do lightmap. Capítulos sobre albedo e calibração de valores são especialmente relevantes.

- **[Unreal Engine Documentation]** — https://dev.epicgames.com/documentation/unreal-engine/ — Seções sobre modos de mobilidade de luz (estática/estacionária/móvel), *Lightmass*, geração e resolução de UV de lightmap e, nas versões recentes, o sistema de iluminação global em tempo real Lumen. Referência prática obrigatória para quem trabalha neste motor.

- **[Unity Manual]** — https://docs.unity3d.com/ — Seções sobre modos de iluminação (*Baked*/*Mixed*/*Realtime*), *lightmapping*, sondas de luz para objetos dinâmicos e configuração do canal de UV de lightmap, com a abordagem correspondente ao motor Unity.

- **[Real-Time Rendering]** (Akenine-Möller, Haines, Hoffman; 4. ed., CRC Press, 2018) — Os capítulos sobre iluminação global, *precomputed lighting* e sombras fundamentam teoricamente os mecanismos pré-calculados deste capítulo. Recomendado para aprofundamento teórico.

- **[Physically Based Rendering: From Theory to Implementation]** (Pharr, Jakob, Humphreys; 4. ed., MIT Press, 2023) — Disponível em https://www.pbr-book.org/. Para aprofundar a teoria do transporte de luz e do ricochete indireto que os lightmaps pré-calculam.
