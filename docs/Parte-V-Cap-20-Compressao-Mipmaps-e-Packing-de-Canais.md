# Capítulo 20 — Compressão, Mipmaps e Packing de Canais

## Introdução

Os capítulos anteriores desta parte trataram de como gerar os mapas de um *asset* a partir do *baking* e de como organizar as texturas no espaço — reunindo-as em atlas e trim sheets, ou multiplicando-as em UDIMs e *texture arrays*. Resta a última etapa da jornada de uma textura rumo à eficiência: a forma como ela é, de fato, **armazenada na memória da placa de vídeo e lida durante a renderização**. Este capítulo final da Parte V trata desse nível, o mais próximo do *hardware*, e dos três mecanismos que governam o custo real de uma textura em jogo: a **compressão**, que reduz quanto ela ocupa na memória; os **mipmaps**, que controlam como ela é lida a distância; e o **packing de canais**, que aproveita ao máximo cada textura combinando informações distintas. São temas técnicos, mas de consequência direta sobre o desempenho e a qualidade visual, e dominá-los é o que separa um *asset* que apenas funciona de um *asset* verdadeiramente eficiente.

Há uma razão para que estes temas fechem a parte e, em certo sentido, a apostila inteira no que toca à produção. Tudo o que se ensinou — o rigor físico do PBR (Parte III), o cuidado com as UVs (Parte II), os métodos de produção (Parte IV), o *baking* e a organização (esta parte) — culmina em texturas que precisam, enfim, caber na memória de uma plataforma real e ser lidas dezenas de milhões de vezes por segundo sem degradar a imagem nem travar o jogo. A compressão, os mipmaps e o packing são os instrumentos dessa adequação final, e exigem reencontrar uma distinção que percorreu toda a apostila — a diferença entre uma textura de **cor**, a ser interpretada pelo olho, e uma textura de **dados**, a ser interpretada por um cálculo —, pois ela governa como cada textura pode ser comprimida, corrigida e empacotada.

## Desenvolvimento

### Por que a memória de textura importa

Antes das técnicas, é preciso dimensionar o problema. Uma textura é uma grade de pixels, e cada pixel guarda valores de cor; uma textura colorida comum de 2048 por 2048 pixels, sem compressão, ocupa na memória da placa de vídeo cerca de **16 megabytes** — e um único *asset* PBR usa vários mapas (cor base, mapa de normais, mapa de rugosidade, mapa metálico, mapa de oclusão ambiente...), de modo que um só objeto pode demandar dezenas de megabytes. Multiplicado pelos milhares de texturas de um jogo, o total ultrapassa de longe a memória de vídeo disponível, sobretudo em plataformas modestas. A **memória de vídeo é um recurso escasso e disputado**, e estourá-la não é uma questão de elegância: quando as texturas não cabem, o sistema é forçado a movê-las para fora e para dentro da memória constantemente, e o jogo trava, engasga ou exibe texturas borradas enquanto as corretas carregam. Gerir a memória de textura é, portanto, condição de viabilidade, não um luxo.

É esse o pano de fundo das três técnicas do capítulo. A **compressão** ataca o problema diretamente, reduzindo quanto cada textura ocupa — tipicamente a um quarto ou menos do tamanho original. Os **mipmaps**, embora acrescentem memória, resolvem um problema correlato de qualidade e desempenho na leitura a distância, e sem eles as texturas distantes ficariam ruidosas e custosas. O **packing de canais** reduz o **número** de texturas, combinando informações que caberiam em mapas separados num só, economizando tanto memória quanto ligações (e, portanto, *draw calls*). Juntas, as três formam o arsenal de adequação da textura ao *hardware*, e operam sobre a mesma escassez que torna todo este conhecimento necessário.

### Escolha de resolução: o tamanho certo para cada *asset*

Antes de comprimir, empacotar ou gerar mipmaps, existe uma decisão que precede todas as demais e que, quando errada, nenhuma técnica posterior corrige completamente: **qual a resolução da textura?** Uma textura desnecessariamente grande desperdiça memória e tempo de carregamento; uma textura desnecessariamente pequena degrada a qualidade visual mesmo com compressão ótima. A escolha de resolução é, portanto, a decisão de orçamento mais direta que um texturizador toma.

> 📘 **Definição**
> A **resolução de textura** é o número de texels em cada dimensão da imagem (por exemplo, 1024×1024 ou 2048×2048). Ela determina diretamente o custo de memória e a qualidade de detalhe, e deve ser proporcional à **densidade de texels necessária** — que depende do tamanho físico do objeto e da sua distância de visualização no jogo.

A resolução correta não é a maior disponível nem um padrão único para todos os *assets*: é aquela que fornece a densidade de texels adequada ao papel visual do objeto. O Capítulo 6 introduziu o conceito de texel density — a relação entre o tamanho físico de uma superfície e quantos texels a cobrem. A resolução de textura é uma das três variáveis dessa equação (junto ao tamanho físico do objeto e ao tamanho das ilhas UV), e escolhê-la com critério exige entender onde e como o objeto aparece no jogo.

Dois princípios orientam a decisão. O primeiro é o de **proximidade de visualização**: um personagem protagonista visto constantemente a menos de dois metros em close cinematográfico justifica texturas de 4096×4096 (ou múltiplos tiles UDIM) para que cada poro da pele seja legível; um bloco de pavimentação de um piso de fundo, visto a dez metros, pode ser adequadamente coberto por uma tileable de 1024×1024 que se repete em toda a superfície. O segundo é o de **potência de dois** (*power of two*): texturas devem ter dimensões em potências de dois — 256, 512, 1024, 2048, 4096 — para que o hardware gere corretamente a cadeia de mipmaps e para que os algoritmos de compressão em blocos (BC, ASTC) operem sem desperdício ou artefatos nas bordas. Texturas com dimensões arbitrárias (723×483, por exemplo) frequentemente causam expansão interna para a próxima potência de dois e consumo de memória maior do que o esperado.

A tabela a seguir sistematiza as faixas de resolução por categoria de *asset*, segundo a prática consolidada da indústria. Os valores assumem a visualização padrão de jogos de terceira pessoa ou FPS; projetos com câmera fixa muito próxima (aventura gráfica, VR) podem justificar uma resolução acima, e jogos móveis ou de câmera muito afastada (estratégia em tempo real) frequentemente ficam uma faixa abaixo.

| Categoria de *asset* | Resolução típica | Observações |
|---|---|---|
| Personagem protagonista — corpo | 2048×2048 | Frequentemente múltiplos texture sets (cabeça, corpo, acessórios) |
| Personagem protagonista — rosto | 2048×2048 a 4096×4096 | SSS, detalhe de pele, olhos em texture set separado |
| NPC com cenas próximas | 2048×2048 | Reduzir para 1024 se distância de visualização for moderada |
| NPC de fundo / multidão | 512×512 a 1024×1024 | Atlas com variação de cores pode servir vários NPCs semelhantes |
| Arma de personagem (jogável) | 2048×2048 | Vista constantemente em primeiro plano; detalhe de metal e desgaste relevante |
| Prop de ambiente — primário | 1024×1024 a 2048×2048 | Caixas, barris, móveis com interação; depende da cena |
| Prop de ambiente — secundário | 512×512 a 1024×1024 | Objetos decorativos, sem interação próxima |
| Prop de ambiente — fundo / LOD 2+ | 256×256 a 512×512 | Frequentemente em atlas compartilhado; distância grande |
| Superfície arquitetônica (muro, piso) | Tileable 1024×1024 ou 2048×2048 | Repetida via UV; resolução serve a uma área virtuamente ilimitada |
| Trim sheet / texture atlas de ambiente | 2048×2048 a 4096×4096 | Resolução se divide entre todas as faixas ou imagens no atlas |
| Terreno (textura base) | 2048×2048 a 4096×4096 por tile | Combinado com texture array e blending; texel density calibrado por bioma |
| Skybox / cubemap de céu | 2048×2048 por face | Mínimo para evitar pixelização visível no horizonte |
| Interface (HUD, menus) | Livre (mas potência de dois) | **Sem mipmap**; resolução depende da resolução-alvo da tela |
| Ícones de inventário / interface | 256×256 a 512×512 | Atlas de ícones; sem mipmap; sRGB |
| Lightmap de ambiente estático | 512×512 a 2048×2048 | Resolução proporcional ao tamanho da geometria e à proximidade |

> 💡 **Dica Profissional**
> A resolução de uma textura determina a metade do orçamento de memória que não se recupera por compressão — comprimir uma textura de 4096×4096 ocupa quatro vezes mais memória que comprimir uma de 2048×2048, independentemente do formato. Escolher a resolução certa desde o início é a decisão de orçamento de textura mais econômica de tomar.

A relação entre resolução e texel density é bidirecional. Se os UVs de um objeto ocupam pouco do espaço UV (ilhas pequenas, muito espaço desperdiçado), aumentar a resolução da textura não melhora proporcionalmente o resultado — a texel density efetiva permanece baixa porque as ilhas cobrem pouca área. A solução nesses casos é primeiro ajustar o layout UV (Capítulo 6) para que as ilhas ocupem melhor o espaço disponível; só então escolher a resolução. A resolução certa pressupõe UVs eficientes.

Por fim, resolução e número de texture sets são decisões complementares. Quando a geometria de um personagem é muito complexa para caber com densidade adequada num único texture set de 2048×2048 — uma armadura com muitas partes distintas, por exemplo — a solução não é necessariamente subir para 4096, mas considerar dividir o personagem em múltiplos texture sets (corpo, armadura, rosto, acessórios), mantendo cada um em 2048 com boa densidade. Essa divisão pode ser mais econômica em memória do que um 4096 único, dependendo de quantas partes se pode reutilizar ou agrupar em atlas.

### Compressão de textura: GPU não é JPEG

> 📘 **Definição**
> **Compressão de textura para GPU** é um método de compressão com perda que mantém a textura comprimida diretamente na memória de vídeo, descomprimindo apenas blocos locais de 4×4 texels no momento exato em que são lidos pelo hardware. Diferencia-se da compressão de arquivo (JPEG, PNG), que precisa ser descomprimida por inteiro antes do uso, voltando ao tamanho original na memória.

A primeira distinção crucial é entre a compressão que se conhece da informática comum — o JPEG, o PNG, o ZIP — e a **compressão de textura para GPU**, que é coisa diferente. Os formatos comuns comprimem para reduzir o tamanho do **arquivo em disco**, mas precisam ser **descomprimidos por inteiro** antes de uso: um JPEG vira uma imagem completa na memória antes de a placa de vídeo poder lê-lo. Isso não resolve o problema da memória de vídeo — na memória, a textura volta ao tamanho cheio. A compressão de textura para GPU é projetada para o oposto: a textura **permanece comprimida na memória de vídeo** e é descomprimida pela placa **em tempo real, em pequenos blocos, no momento exato em que cada texel é lido**. O ganho de memória é permanente, não apenas no disco, e é por isso que praticamente toda textura de jogo usa um desses formatos.

As famílias de formatos refletem as plataformas. Os formatos da família **BC** (*Block Compression*, também conhecidos como DXT/DXTn) dominam o mundo de PC e consoles baseados em DirectX, com variantes especializadas — uma para cor com transparência, outra otimizada para mapas de normais, outra para dados de canal único, e assim por diante. No mundo móvel, o formato **ASTC** tornou-se o padrão moderno, flexível em qualidade e taxa de compressão; antes dele, formatos como ETC e PVRTC serviam ao mesmo papel. *PUBG Mobile* (LightSpeed Studios, 2018) ilustra o desafio da transição: lançado quando o ASTC ainda não estava disponível em todos os dispositivos do mercado-alvo, o jogo implementou um sistema de detecção que entregava ASTC nos dispositivos compatíveis e ETC2 nos demais, garantindo cobertura máxima sem duplicar a arte — um exemplo concreto da gestão de múltiplos formatos por nível de *hardware* que os motores de jogo modernos hoje automatizam por plataforma.

O traço comum a todos é que comprimem a textura em **blocos de tamanho fixo** (tipicamente 4 por 4 texels), cada um reduzido a um punhado de bytes, o que permite a descompressão local e rápida — mas implica perda: a compressão é **com perda** (*lossy*), e introduz pequenos artefatos. A arte de comprimir bem está em escolher o formato e a taxa que minimizam a perda visível para cada tipo de mapa — e aqui reaparece a distinção entre cor e dados.

> **Figura 20.1** — Esquema da compressão em blocos: uma textura dividida numa grade de blocos de 4×4 texels, cada bloco reduzido a poucos bytes na memória de vídeo, com uma seta indicando a descompressão de um único bloco no momento da leitura — contrastada com a compressão de arquivo comum (JPEG), que precisa ser descomprimida por inteiro antes do uso.

**Screenshot sugerido:** Captura comparando uma textura sem compressão e a mesma textura comprimida em BC/ASTC, com ampliação de uma região para evidenciar os artefatos de bloco, e o tamanho em memória de cada versão.

### Cor versus dados: o que comprimir e como

A escolha do formato e do tratamento de compressão depende inteiramente da natureza da textura, e aqui a distinção entre cor e dados — recorrente desde a Parte III — torna-se uma regra de produção concreta. Uma textura de **cor**, como o mapa de cor base, destina-se ao olho, que é tolerante a pequenos erros de matiz; pode receber compressão mais agressiva e deve ser tratada no espaço de cor **sRGB**, a curva que, como vimos na Parte III, ajusta os valores à percepção humana. Já uma textura de **dados**, como o mapa de normais, o mapa de rugosidade ou o mapa metálico, destina-se a um **cálculo**, que não tolera os mesmos erros: comprimi-la como se fosse cor, ou aplicar-lhe a curva sRGB, **corrompe os valores** e o cálculo produz resultado errado — o sombreamento, a reflexão, a rugosidade saem deturpados.

> ⚠️ **Atenção**
> A regra é simples e deve ser lei: **trate cor como cor e dado como dado, sempre**. Aplicar a curva sRGB a um mapa de dados ou comprimi-lo no formato de cor produz o erro silencioso: o material não quebra, apenas fica "um pouco errado" — plástico, achatado, com reflexos estranhos —, e o diagnóstico é difícil para quem não conhece a causa.

Disso decorrem regras práticas que todo texturizador precisa internalizar. O mapa de **cor base** é tratado como cor: espaço sRGB e compressão de cor. Os mapas de **dados** — mapa de normais, mapa de rugosidade, mapa metálico, mapa de oclusão ambiente, altura — são tratados como dados **lineares**: nunca recebem a curva sRGB (que falsearia os valores) e usam formatos de compressão apropriados ao seu conteúdo. O **mapa de normais**, em particular, exige um formato de compressão **especializado** (como a variante BC própria para normais), porque a compressão de cor comum destruiria a precisão das direções e produziria o sombreamento facetado e ruidoso que denuncia um mapa de normais mal comprimido. E os mapas de **canal único** (uma rugosidade, uma máscara) usam formatos otimizados para um só canal, desperdiçando menos espaço. Marcar corretamente cada textura — cor ou dado, sRGB ou linear, formato adequado — na importação ao motor de jogo é uma das tarefas mais elementares e, ao mesmo tempo, mais frequentemente erradas da produção.

### Mipmaps: a textura em muitas resoluções

> 📘 **Definição**
> **Mipmaps** são cadeias de versões progressivamente menores de uma textura — a original, depois a metade, depois um quarto, até um único pixel —, pré-calculadas e armazenadas junto com ela. Durante a renderização, o hardware escolhe automaticamente o nível adequado à distância do objeto, tornando a leitura distante barata e livre de cintilação.

O segundo mecanismo, os **mipmaps**, responde a um problema que surge quando uma textura é vista a distância. Quando um objeto texturizado está longe, ele ocupa poucos pixels na tela, mas sua textura tem resolução plena — e o hardware precisaria, para cada pixel da tela, amostrar e combinar muitos texels da textura grande comprimida naquele pequeno espaço. Isso é, ao mesmo tempo, **custoso** (muitas leituras por pixel) e **feio** (a amostragem irregular produz cintilação e ruído, o chamado *aliasing*, em que a textura "ferve" quando o objeto ou a câmera se move). Os mipmaps resolvem ambos os problemas de uma vez: são uma **cadeia de versões progressivamente menores da mesma textura**, pré-calculadas e armazenadas junto com a original.

Durante a renderização, o hardware **escolhe automaticamente o nível de mipmap adequado à distância**: objeto perto, lê a textura grande; objeto longe, lê uma versão pequena e já reduzida, em que muitos texels foram corretamente combinados de antemão. Com isso, a leitura distante torna-se barata (a versão pequena tem poucos texels) e limpa (a redução foi feita com cuidado, sem cintilação). O custo dos mipmaps é de **memória**: a cadeia de versões menores acrescenta cerca de um terço ao tamanho da textura. Mas esse custo é quase sempre compensado, e por isso os mipmaps são **gerados por padrão** para praticamente toda textura de cena 3D.

Conectam-se, ainda, a dois temas já vistos: o *padding* do *baking* (Capítulo 15) e a margem entre imagens do atlas (Capítulo 18) existem precisamente para que os níveis menores de mipmap, ao combinar texels vizinhos, não puxem o vazio ou as imagens adjacentes para dentro da superfície — as costuras que assombram quem ignora os mipmaps. Há, por fim, uma exceção importante: texturas de **interface** (que aparecem sempre no mesmo tamanho na tela) e certos mapas de dados não se beneficiam de mipmaps, e gerá-los para elas é desperdício.

> **Figura 20.2** — A cadeia de mipmaps de uma textura: a imagem em resolução plena seguida de versões cada vez menores (1/2, 1/4, 1/8...) até um único pixel — e, ao lado, uma superfície em perspectiva indo até o horizonte, com indicação de qual nível de mipmap o hardware usa em cada faixa de distância, do mais detalhado perto ao menor longe.

**Screenshot sugerido:** Captura comparando uma superfície distante (um piso ladrilhado até o horizonte) renderizada sem mipmaps (cintilante e ruidosa) e com mipmaps (estável e limpa), evidenciando a redução do *aliasing*.

### Packing de canais: aproveitar cada bit

> 📘 **Definição**
> **Packing de canais** (empacotamento de canais) é a técnica de guardar múltiplos mapas de canal único (mapa de rugosidade, mapa metálico, mapa de oclusão ambiente) em canais distintos de uma única textura RGBA, reduzindo o número de texturas e de ligações de sombreador necessárias para um material.

O terceiro mecanismo, o **packing de canais**, explora uma característica simples das texturas: elas têm, em geral, **quatro canais** — vermelho, verde, azul e alfa (RGBA) —, mas muitos mapas de dados usam apenas **um** canal. O mapa de rugosidade é uma escala de cinza (um canal); o mapa metálico, idem; o mapa de oclusão ambiente, idem; muitas máscaras, idem. Manter cada um desses mapas como uma textura separada desperdiça três quartos do espaço de cada uma — três canais vazios por textura. O packing de canais elimina esse desperdício **guardando um mapa diferente em cada canal de uma única textura**: por exemplo, oclusão no vermelho, rugosidade no verde e metalicidade no azul, num só arquivo. Onde antes havia três texturas, há uma; e o sombreador, ao ler, sabe que deve buscar a oclusão no canal vermelho, a rugosidade no verde, e assim por diante.

O ganho é duplo e alinha-se com toda esta parte. Reduz-se a **memória**, pois três mapas de um canal ocupam o espaço de um só de três canais em vez de três texturas inteiras; e reduz-se o **número de texturas**, e portanto de ligações e de *draw calls* (Capítulo 18), pois o sombreador lê uma textura em vez de três. Há convenções consagradas — a combinação de mapa de oclusão ambiente, mapa de rugosidade e mapa metálico num só mapa é tão comum que recebe nomes próprios (como *ORM* ou *RMA*, conforme a ordem dos canais) e é o formato esperado por muitos motores de jogo. A Unreal Engine e o Substance Painter convergiram, a partir de 2016, numa convenção que se tornou padrão de fato na indústria: oclusão no canal vermelho, rugosidade no verde e metalicidade no azul, exportados com sufixo `_ORM`. A partir do Substance Painter 2 e da Unreal Engine 4.14, essa trinca passou a ser exportada e importada automaticamente, reduzindo de três a uma a ligação de sombreador dos mapas de dados mais comuns.

> ⚠️ **Atenção**
> Um mapa empacotado deve sempre ser tratado como **dado linear** (nunca sRGB), pois seus canais carregam valores de cálculo, não cor a ser vista. Empacotar por engano um mapa de cor junto a mapas de dados, ou aplicar sRGB ao mapa empacotado, reintroduz o erro silencioso descrito na seção anterior.

> **Figura 20.3** — Um mapa empacotado *ORM* decomposto em seus canais: a textura combinada à esquerda e, à direita, os três canais separados mostrando o mapa de oclusão ambiente (R), o mapa de rugosidade (G) e o mapa metálico (B) como imagens em escala de cinza — ilustrando como três mapas de um canal cabem numa só textura.

**Screenshot sugerido:** Captura de um material num motor de jogo mostrando a textura *ORM* ligada às entradas de oclusão, rugosidade e metalicidade simultaneamente, evidenciando uma textura servindo a três propósitos.

### A síntese: a textura adequada ao hardware

Reunidos os três mecanismos, fecha-se o ciclo da eficiência que toda a Parte V construiu. Uma textura de jogo verdadeiramente eficiente é **comprimida** no formato certo para sua natureza (de cor ou de dado), tem seus **mipmaps** gerados para ser lida com economia e limpeza a qualquer distância, e participa, quando possível, de um **packing** que reduz o número de texturas. Cada uma dessas decisões é governada pela distinção entre cor e dados que percorreu a apostila desde a Parte III, e cada uma materializa, no nível do *hardware*, o equilíbrio entre fidelidade e custo que é o fio condutor de toda a obra: comprimir o quanto se pode sem que o olho note, gastar memória com mipmaps porque o ganho de qualidade e desempenho compensa, empacotar canais para que nada se desperdice. O texturizador que domina esse nível entrega não apenas materiais corretos e bonitos, mas *assets* que cabem, rodam e brilham na plataforma real.

> 💡 **Dica Profissional**
> Equipes de produção mantêm presets de importação por tipo de mapa — cor base (BC/sRGB), mapa de normais (BC especializado/linear), mapa ORM (BC/linear), interface (sem compressão de GPU/sem mipmap) — e aplicam esses presets consistentemente a todo *asset*. A consistência elimina o erro silencioso e poupa o diagnóstico de materiais "ligeiramente errados" sem causa aparente.

## Aplicação em Jogos

Na produção, compressão, mipmaps e packing são definidos sobretudo na **importação** das texturas ao motor de jogo e nas configurações de plataforma, e constituem uma parte significativa do trabalho de otimização. Cada motor oferece, na importação, controles para o formato de compressão (com presets por tipo de mapa e por plataforma), a geração de mipmaps e a marcação de espaço de cor (sRGB ou linear), e equipes mantêm convenções rígidas para que cada tipo de mapa seja sempre importado corretamente — o mapa de normais no formato especializado, os mapas de dados em linear, os mapas empacotados sem sRGB. O packing, por sua vez, costuma ser decidido na autoria: o texturizador exporta já os mapas combinados na convenção esperada pelo motor (frequentemente *ORM*), poupando texturas desde a origem. Erros nessas configurações são uma das causas mais comuns de materiais com aparência sutilmente errada em jogos, e diagnosticá-los é parte da rotina.

A importância dessas técnicas cresce na razão inversa do poder da plataforma. Em PCs e consoles de ponta, há mais memória de vídeo e mais margem, e as decisões de compressão podem ser mais generosas; em celulares e plataformas modestas, a memória é escassa, e a compressão agressiva (ASTC em taxas altas), os mipmaps e o packing tornam-se obrigatórios para que o jogo sequer caiba e rode. Os motores de jogo oferecem, por isso, configurações **por plataforma**: a mesma textura pode ser exportada em formato e resolução diferentes para PC, console e celular, automaticamente, conforme o orçamento de cada um. O texturizador competente entende esse sistema e o usa para que um mesmo *asset* sirva a múltiplas plataformas sem refazimento.

## Estudo de Caso

### *Genshin Impact* (HoYoverse, 2020) — Compressão Cross-Platform e a Arte de Servir PC, Console e Celular com as Mesmas Texturas

*Genshin Impact* é um dos casos mais bem documentados de gestão de texturas multi-plataforma na história recente dos jogos. Lançado simultaneamente para PC, PlayStation 4, iOS e Android, o jogo precisava entregar o mesmo mundo aberto — Teyvat, com seus biomas elaborados, personagens de altíssimo detalhe e efeitos visuais complexos — em plataformas cujos orçamentos de memória de vídeo variam por um fator de dez ou mais. Um celular de gama média de 2020 dispõe de uma fração mínima da memória de vídeo de um PC de alto desempenho, e a HoYoverse documentou em apresentações técnicas (incluindo a GDC 2021 e publicações internas) o sistema de exportação por plataforma que tornou isso possível: as mesmas texturas artísticas, processadas em pipelines de compressão distintos para cada destino.

Para PC e PlayStation, o estúdio utilizou compressão BC nas texturas de cor e formatos especializados para mapas de normais, obtendo boa qualidade com ocupação de memória gerenciável. Para Android, o padrão ASTC foi aplicado em taxas progressivamente mais agressivas conforme a categoria do dispositivo — a mesma textura que chega ao PC em alta resolução e compressão leve pode chegar a um celular de entrada em resolução reduzida à metade e ASTC em taxa alta, sem que o artista precise criar um *asset* separado. Esse sistema de configuração por plataforma, com presets automáticos por categoria de dispositivo, é exatamente o que este capítulo descreve: a mesma decisão artística, expressada em múltiplos formatos de armazenamento conforme o orçamento de cada *hardware*.

O que torna o caso especialmente instrutivo é a escala do desafio: *Genshin Impact* conta com centenas de personagens jogáveis e não-jogáveis, cada um com múltiplos mapas (cor, mapa de normais, mapa de rugosidade, mapa de oclusão ambiente), e um mundo aberto que exige mipmaps em praticamente toda textura de cenário para manter as superfícies estáveis durante a câmera em movimento. A HoYoverse adotou packing de canais em seus materiais — combinando mapa de rugosidade, mapa metálico e máscaras em texturas multicanal — para reduzir o número de texturas por material e aliviar tanto a memória quanto as ligações de sombreador. O resultado é um jogo que, em um celular moderno de gama média, parece visualmente coeso e detalhado, sem os travamentos e as texturas borradas que marcam projetos que ignoram a adequação das texturas ao *hardware* de destino.

**O que aprender com isso:** Compressão, mipmaps e packing não são ajustes de fim de projeto, mas decisões arquiteturais que precisam estar embutidas no pipeline desde o início. O sistema de exportação por plataforma, com presets automáticos, é a forma industrial de garantir essa consistência em escala.

> **Figura 20.4** — Diagrama de pipeline de exportação de texturas de *Genshin Impact*: uma textura de personagem processada em três formatos distintos (PC/console em BC, Android em ASTC de alta taxa, Android de entrada em ASTC de taxa máxima e resolução reduzida), ilustrando o sistema de configuração por plataforma com um único *asset* artístico de origem.

**Screenshot sugerido:** Comparação de capturas do mesmo cenário de *Genshin Impact* em qualidade máxima (PC) e em qualidade média (celular), evidenciando que o conteúdo artístico é o mesmo e que a diferença resulta de configurações de compressão e resolução por plataforma.

## Boas Práticas

A boa prática soberana deste capítulo é a regra de **cor versus dados**, aplicada com disciplina na importação: o mapa de cor base é tratado como cor (espaço sRGB, compressão de cor); todos os mapas de dados — mapa de normais, mapa de rugosidade, mapa metálico, mapa de oclusão ambiente, altura, máscaras, mapas empacotados — são tratados como dados lineares, **nunca** com sRGB, com o formato de compressão apropriado a cada um, e o mapa de normais sempre no formato especializado.

> 💡 **Dica Profissional**
> Comprima toda textura de jogo num formato de GPU (BC em PC e console, ASTC em celular) e use as configurações **por plataforma** para que um mesmo *asset* sirva a destinos de orçamentos diferentes sem refazimento. Gere mipmaps por padrão para texturas de cena 3D e dispense-os apenas onde não ajudam, como em texturas de interface de tamanho fixo. Empacote canais seguindo as convenções consagradas (como *ORM*) que os motores de jogo esperam, tratando sempre o mapa empacotado como dado linear.

A eficiência, aqui como em toda a parte, é fruto de decisões coordenadas ao longo do *pipeline*, não de um ajuste isolado no fim. O *padding* do *baking* e a margem do atlas existem para que os mipmaps não produzam costuras, de modo que quem ignorou essas margens cedo pagará pelo descuido na fase de compressão.

## Erros Comuns

> ❌ **Erro Comum**
> Aplicar a curva sRGB a um mapa de dados (mapa de rugosidade, mapa metálico, mapa de normais) falseia seus valores e produz o erro silencioso: o material não quebra, apenas fica "um pouco errado" — plástico, achatado, com reflexos estranhos. O diagnóstico é difícil para quem não conhece a causa; a prevenção é a disciplina de marcar cada textura por sua natureza, sem exceção.

O erro mais grave e mais comum é violar a regra de cor versus dados na importação: aplicar a curva **sRGB a um mapa de dados** (mapa de rugosidade, mapa metálico, mapa de normais), falseando seus valores, ou comprimir um **mapa de normais no formato de cor** comum, destruindo a precisão das direções. Ambos produzem o erro **silencioso**: o material não quebra, apenas fica "um pouco errado".

> ❌ **Erro Comum**
> Não comprimir as texturas, ou comprimi-las apenas como arquivo (supondo que o JPEG resolve a memória de vídeo, quando ele só reduz o tamanho em disco), deixa as texturas no tamanho cheio na memória da placa de vídeo — potencialmente estourando a memória disponível e causando travamentos e texturas borradas.

Há o erro de **não comprimir** as texturas, ou comprimi-las apenas como arquivo, e estourar a memória da plataforma — com o jogo travando e exibindo texturas borradas. Seu oposto é comprimir **agressivamente demais** mapas sensíveis, introduzindo artefatos de bloco visíveis em superfícies importantes — falta de calibração da taxa ao tipo e à importância do mapa. Há o erro de **esquecer os mipmaps** em texturas de cena, resultando em cintilação e ruído nas superfícies distantes e em leitura mais custosa — ou, inversamente, gerá-los onde não servem, como na interface, desperdiçando memória. No packing, o equívoco é combinar canais sem respeitar as convenções do motor de jogo (entregando um *ORM* na ordem trocada) ou empacotar por engano um mapa de cor junto a dados e tratar o conjunto com sRGB. E persiste o erro de fundo de toda a parte: tratar a otimização como etapa acessória, deixada para o fim e feita às pressas, quando ela é a condição de o *asset* caber e rodar.

## Resumo

Este capítulo final da Parte V tratou de como a textura, depois de gerada e organizada, é efetivamente armazenada na memória da placa de vídeo e lida na renderização. Dimensionamos o problema da memória escassa e estabelecemos o ponto de partida da eficiência: a **escolha de resolução** — a decisão de potência de dois que determina o orçamento base de cada textura, orientada pela proximidade de visualização e pela tabela de categorias de *asset*. Apresentamos a **compressão de textura para GPU** — distinta da compressão de arquivo comum por manter a textura comprimida na memória e descomprimi-la em blocos no momento da leitura —, com suas famílias (BC/DXT em PC e console, ASTC em celular) e seu caráter com perda, e estabelecemos a regra que governa tudo: tratar **cor como cor** (sRGB, compressão de cor) e **dados como dados** (linear, formatos apropriados, mapa de normais em formato especializado), sob pena do erro silencioso que deturpa materiais. Apresentamos os **mipmaps** — a cadeia de versões progressivamente menores da textura, escolhidas pelo hardware conforme a distância — como solução para a cintilação (*aliasing*) e o custo da leitura distante, ao preço de cerca de um terço de memória extra, conectando-os ao *padding* e à margem do atlas que evitam costuras. E apresentamos o **packing de canais** — guardar mapas de canal único em canais distintos de uma só textura, na convenção *ORM* e afins — como forma de reduzir memória e número de texturas (e *draw calls*) de uma vez, sempre tratado como dado linear.

## Exercícios

**1.** Explique por que a memória de vídeo é um recurso crítico em jogos e qual a diferença fundamental entre a compressão de textura para GPU (BC, ASTC) e a compressão de arquivo comum (JPEG, PNG). Por que apenas a primeira resolve o problema da memória de vídeo?

**2.** A regra de "cor versus dados" é apresentada como soberana neste capítulo. Explique-a e aplique-a: para cada um dos mapas a seguir — cor base, mapa de normais, mapa de rugosidade, mapa metálico, mapa de oclusão ambiente —, indique se deve ser tratado como cor ou como dado, se recebe sRGB ou linear, e quais cuidados de compressão exige.

**3.** Explique o que são mipmaps, qual duplo problema (de qualidade e de desempenho) eles resolvem na leitura de texturas a distância, e qual seu custo. Relacione os mipmaps ao *padding* do *baking* (Capítulo 15) e à margem entre imagens do atlas (Capítulo 18), explicando por que essas margens existem.

**4.** Explique o que é o packing de canais e por que combinar mapas de canal único (como num *ORM*) reduz tanto a memória quanto o número de *draw calls*. Que cuidado de espaço de cor um mapa empacotado exige, e por quê?

**5.** Diagnóstico integrador: um jogo apresenta, ao mesmo tempo, (a) travamentos e texturas borradas no celular, (b) personagens com pele "plástica" e metais sem reflexo correto, e (c) pisos distantes que cintilam quando a câmera se move. Para cada sintoma, identifique a causa provável entre as técnicas deste capítulo e descreva a correção. Em seguida, explique por que o capítulo afirma que "a eficiência não é etapa acessória, mas condição da viabilidade do que se produziu".

## Glossário

**ASTC:** Formato de compressão de textura para GPU predominante em dispositivos móveis, flexível em taxa de compressão e qualidade, que mantém a textura comprimida na memória de vídeo.

**Potência de dois (*power of two*):** Convenção de dimensionamento de texturas segundo a qual largura e altura devem ser potências de dois (256, 512, 1024, 2048, 4096). Garante a geração correta da cadeia de mipmaps e o funcionamento sem desperdício dos algoritmos de compressão em blocos.

**Resolução de textura:** Número de texels em cada dimensão de uma imagem de textura (ex.: 2048×2048). Determina diretamente o custo de memória e deve ser proporcional à densidade de texels necessária conforme a categoria de *asset* e a sua distância de visualização.

**BC (Block Compression / DXT):** Família de formatos de compressão de textura para GPU amplamente usados em PC e consoles, com variantes especializadas para cor, mapas de normais e dados de canal único.

**Compressão de textura para GPU:** Método de compressão com perda que mantém a textura comprimida na memória de vídeo, descomprimindo blocos locais de 4×4 texels no momento de leitura pelo hardware.

**Draw call:** Comando enviado pela CPU à GPU para renderizar um objeto. O packing de canais reduz o número de texturas por material, diminuindo o número de ligações e potencialmente de *draw calls*.

**Espaço de cor sRGB:** Curva de correção gamma aplicada a texturas de cor para ajustar os valores armazenados à percepção humana. Nunca deve ser aplicada a mapas de dados.

**Mapa de normais:** Textura que armazena informações de orientação da superfície, exigindo formato de compressão especializado para preservar a precisão das direções.

**Mapa de oclusão ambiente:** Textura que registra o quanto cada ponto da superfície está obstruído da luz ambiente. Dado de canal único, adequado para packing.

**Mapa de rugosidade:** Textura que define a irregularidade microscópica da superfície, determinando a dispersão da reflexão. Dado de canal único, linear, adequado para packing.

**Mapa metálico:** Textura que define quais partes de uma superfície se comportam como metal na equação de iluminação PBR. Dado de canal único, linear, adequado para packing.

**Mipmaps:** Cadeia de versões progressivamente menores de uma textura, armazenadas junto com a original e selecionadas automaticamente pelo hardware conforme a distância de visualização.

**Motor de jogo:** Sistema de software que gerencia renderização, física e lógica de um jogo em tempo real, oferecendo ferramentas de importação de texturas com configurações de compressão, mipmaps e espaço de cor por plataforma.

**ORM:** Convenção de packing de canais em que o mapa de oclusão ambiente ocupa o canal vermelho (R), o mapa de rugosidade o verde (G) e o mapa metálico o azul (B), numa única textura tratada como dado linear.

**Packing de canais:** Técnica de armazenar múltiplos mapas de canal único nos canais RGBA de uma única textura, reduzindo o número de texturas e de ligações de sombreador.

**Sombreador:** Programa executado pela GPU para calcular a aparência de cada fragmento renderizado, determinando como os mapas de textura são combinados e como a superfície reage à iluminação.

## Leituras Complementares

- **[The PBR Guide]** (Wes McDermott; Allegorithmic / Adobe), versão 2018 — Fundamenta a distinção entre dados de cor (sRGB) e dados lineares e a exigência de mapas calibrados que governa o tratamento e a compressão de cada tipo de textura neste capítulo. Leitura obrigatória para compreender a base física da regra cor versus dados.
- **[Blender Maps] e [Blender Normals]** — Situam os mapas de um material e o tratamento de normais, úteis para compreender por que cada mapa exige compressão e espaço de cor próprios — especialmente relevante para a seção sobre mapa de normais.
- **[AOD — Texel Density]** — Relaciona resolução, densidade de texels e proximidade — base para decidir a resolução e a taxa de compressão adequadas a cada *asset* conforme sua distância de visualização.
- **[Unreal Engine Documentation]** — https://dev.epicgames.com/documentation/unreal-engine/. Seções sobre formatos de compressão de textura, configurações por plataforma, geração de mipmaps, marcação de espaço de cor (sRGB/linear) e convenções de packing de canais (*ORM*/máscaras): referências práticas diretas para implementar tudo o que o capítulo descreve.
- **[Unity Manual]** — https://docs.unity3d.com/. Seções sobre importação e compressão de texturas, mipmaps e canais, para comparar a implementação em outro motor de jogo.
- **[Real-Time Rendering]** — AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. 4. ed. Boca Raton: CRC Press, 2018. Os capítulos sobre compressão de textura, filtragem, mipmapping e *antialiasing* de textura fundamentam teoricamente todos os mecanismos deste capítulo.
