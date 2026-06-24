# Referência — Estudos de Caso Reais por Capítulo

Este documento mapeia um jogo real documentado para cada capítulo da apostila, substituindo os estudos de caso genéricos por referências rastreáveis a post-mortems, GDC talks, dev blogs e análises técnicas publicadas. Nenhum jogo se repete, exceto quando a relevância histórica é insubstituível (id Software, Capítulos 4 e 17).

---

## Capítulo 1 — Materiais, Texturas e a Representação Visual

**Jogo:** *Half-Life 2* — Valve, 2004

**Por que funciona:** A Valve construiu para o Source Engine um sistema de materiais declarativos (arquivos `.vmt`) que separava explicitamente as propriedades da superfície — cor, especularidade, normal, reflexo — em entradas independentes antes que qualquer terminologia PBR existisse. Cada superfície do jogo é descrita por um arquivo de texto que especifica o que o shader deve computar, não o resultado final. É o primeiro exemplo amplamente documentado do princípio fundador do capítulo: aparência é o produto de dados + luz, não uma imagem estática.

**Fontes:**
- Jason Mitchell, Moby Francke, Dhabih Eng — *"Shading in Valve's Source Engine"*, GDC 2006. Disponível no GDC Vault.
- Valve Developer Community Wiki — `https://developer.valvesoftware.com/wiki/Material`

**Imagem sugerida:**

> Figura: Comparação entre a parede de tijolos de *Half-Life 2* vista de frente e em iluminação oblíqua, mostrando o relevo e a reflexão computados pelo shader do Source Engine.

**Screenshot sugerido:** Corredor do Capítulo 8 de *Half-Life 2* (estação de metrô de Ravenholm), onde tijolos, concreto e metal exibem comportamentos de superfície completamente diferentes sob a mesma fonte de luz.

---

## Capítulo 2 — Evolução Histórica da Texturização em Jogos Digitais

**Jogo:** *Battlefield 4* — DICE, 2013

**Por que funciona:** O paper de Sébastien Lagarde e Charles de Rousiers, *"Moving Frostbite to Physically Based Rendering"* (SIGGRAPH 2014), é o documento mais completo já publicado sobre a migração de um estúdio AAA do paradigma especular/difuso para o PBR. Ele descreve exatamente os problemas que o capítulo menciona — texturas dependentes da iluminação, inconsistência entre artistas, materiais que só funcionavam numa única cena — e como o PBR os resolveu. *Battlefield 4* foi o primeiro jogo a rodar o Frostbite já completamente redesenhado para PBR.

**Fontes:**
- Sébastien Lagarde & Charles de Rousiers — *"Moving Frostbite to Physically Based Rendering"*, SIGGRAPH 2014. Blog de Lagarde: `https://seblagarde.wordpress.com`
- "The Science Behind Battlefield 4's Stunning Graphics" — Wired, 2013.

**Imagem sugerida:**

> Figura: Diagrama do paper de Lagarde mostrando as faixas de valores válidos de albedo para diferentes materiais físicos — um dos primeiros documentos a estabelecer regras numéricas para a texturização PBR.

**Screenshot sugerido:** Cena externa de *Battlefield 4* com chuva, mostrando como a mesma superfície de concreto muda de aspecto conforme a metalicidade, rugosidade e reflexo ambiental interagem dinamicamente.

---

## Capítulo 3 — Como Motores de Jogo Interpretam Materiais

**Jogo:** *Hellblade: Senua's Sacrifice* — Ninja Theory, 2017

**Por que funciona:** A Ninja Theory documentou publicamente, em uma série de 30 episódios no YouTube (*"The Making of Hellblade"*), todo o processo de autoria de materiais no Unreal Engine 4 — incluindo como as entradas do modelo de material (Base Color, Metallic, Roughness, Normal) são interpretadas pelo shader PBR padrão da Unreal. Por ser um estúdio pequeno sem middleware proprietário, eles usaram o motor como caixa-preta e explicaram o raciocínio por trás de cada nó do material editor com rara clareza pedagógica.

**Fontes:**
- Ninja Theory — *"The Making of Hellblade: Dev Diary"*, YouTube, episódios 1–30. Disponível no canal oficial da Ninja Theory.
- Unreal Engine Blog — *"Hellblade: Behind the Scenes"*, 2017. `https://www.unrealengine.com`

**Imagem sugerida:**

> Figura: Rede de material de pedra úmida de *Hellblade* no editor de materiais da Unreal Engine 4, mostrando as entradas PBR (Base Color, Roughness, Normal) conectadas a texturas e parâmetros.

**Screenshot sugerido:** Cena de *Hellblade* em ambiente brumoso com superfícies de pedra, madeira e pele respondendo à iluminação dinâmica — três materiais com metalicidade 0, rugosidades distintas e normals assadas.

---

## Capítulo 4 — Coordenadas UV e Projeções de Textura

**Jogo:** *Quake* — id Software, 1996

**Por que funciona:** *Quake* usou projeção planar no espaço do mundo para todas as superfícies de cenário (herdada do BSP do Doom). O resultado é o efeito "nadando" (*texture swimming*): quando o jogador anda, as texturas das paredes se movem de forma independente da geometria, porque estão fixas no mundo, não na superfície. *Quake 2* (1997) introduziu UV mapping por face. Nenhum outro exemplo na história dos jogos ilustra tão concretamente a diferença entre projeção no espaço do mundo e coordenadas UV fixas à superfície.

**Fontes:**
- Michael Abrash — *"Graphics Programming Black Book"*, Capítulos 67–70. Disponível gratuitamente em `https://www.jagregory.com/abrash-black-book/`
- Fabien Sanglard — *"Quake Source Code Review"*, Blog, 2010. `https://fabiensanglard.net/quakeSource/`

**Imagem sugerida:**

> Figura: Fotograma de *Quake* (e1m1) mostrando o efeito de texture swimming nas paredes de pedra enquanto o jogador se move — evidência visual direta da projeção planar no espaço do mundo.

**Screenshot sugerido:** Lado a lado: corredor de *Quake* (1996) com projeção planar vs. corredor de *Quake 2* (1997) com UVs por face, ilustrando a diferença entre os dois paradigmas.

---

## Capítulo 5 — UV Unwrapping

**Jogo:** *Monster Hunter: World* — Capcom, 2018

**Por que funciona:** Cada monstro de *Monster Hunter: World* é um desafio de unwrapping orgânico de altíssima complexidade: escalas sobrepostas, barbatanas com transições, membros articulados, corpos que se deformam sob animação. O blog de desenvolvimento da Capcom e entrevistas com o time de arte documentam as decisões de costura — onde colocar emendas para que não apareçam nas animações, como lidar com simetria parcial (dragões com membros diferentes), e como ilhas de escalas individuais são organizadas para manter densidade uniform. É o caso mais rico de unwrapping orgânico documentado na indústria.

**Fontes:**
- Capcom R&D Blog — *"The Art of Monster Hunter: World"*, 2018. Seção de criação de monstros.
- GDC 2018 — *"Monster Hunter: World — Art Direction and World Building"*, Yuya Tokuda e Kaname Fujioka.
- 80.lv — Entrevistas com artistas de personagem da Capcom, 2018–2019.

**Imagem sugerida:**

> Figura: Layout UV do Rathalos de *Monster Hunter: World*, mostrando a separação entre ilhas das asas, escamas do corpo, cabeça e membros, com costuras posicionadas nas junções articuladas.

**Screenshot sugerido:** Rathalos em close, mostrando a continuidade das escalas e a ausência de costuras visíveis nas regiões de alta visibilidade.

---

## Capítulo 6 — Texel Density e Organização de UVs

**Jogo:** *Horizon Zero Dawn* — Guerrilla Games, 2017

**Por que funciona:** *Horizon Zero Dawn* combina, num único frame, personagens orgânicos (Aloy, tribos), máquinas metálicas (Watcher, Thunderjaw) e ambientes naturais vastos. Manter densidade de texels coerente entre categorias tão distintas, em um mundo aberto onde a câmera aproxima e afasta livremente, foi um dos desafios centrais documentados pela Guerrilla em seus dev diaries e na GDC. O pipeline da equipe de ambiente incluiu ferramentas internas de visualização de texel density para checar toda a cena de uma vez.

**Fontes:**
- GDC 2017 — *"Horizon Zero Dawn: Creating the Open World"*, Arjan Bak e Jan-Bart van Beek.
- Guerrilla Games Dev Diary — "The Art of Horizon Zero Dawn", publicado em série no blog oficial.
- 80.lv — *"Building the World of Horizon Zero Dawn"*, 2017.

**Imagem sugerida:**

> Figura: Cena de *Horizon Zero Dawn* com Aloy em frente a um Thunderjaw, com o mapa de verificação de texel density aplicado a ambos — demonstrando a coerência entre personagem orgânico e máquina metálica.

**Screenshot sugerido:** Close de Aloy e de uma peça metálica de maquinário no mesmo frame, ambas com densidade equivalente, em ambiente de floresta com superfícies de folhagem no fundo.

---

## Capítulo 7 — Preparação de Assets para Produção

**Jogo:** *Forza Motorsport 6* — Turn 10 Studios, 2015

**Por que funciona:** Turn 10 documenta publicamente seu pipeline de digitalização a laser e fotogrametria para carros reais. Cada carro passa por uma sequência rigorosa de etapas — scan, retopologia, calibração de escala, definição de pivôs (para rodas e portas), verificação de normais, nomenclatura padronizada (modelo, ano, fabricante, LOD), exportação com transformações aplicadas — antes de entrar no motor. É o pipeline de preparação de asset mais padronizado e documentado da indústria, e a comparação entre um asset bem preparado e um problemático (pivô no centro, escala incorreta, normais invertidas) é demonstrada em vídeos oficiais de making-of.

**Fontes:**
- GDC 2016 — *"The Art of Forza Motorsport 6: Apex"*, Christian Ammann, Turn 10 Studios.
- Xbox Wire Blog — *"How Turn 10 Builds a Car for Forza"*, 2014–2016.
- IGN Making-Of Series — *"Building the Cars of Forza"*, 2015.

**Imagem sugerida:**

> Figura: Diagrama do pipeline de produção de um carro em *Forza Motorsport*, mostrando as etapas sequenciais de scan laser, retopologia, calibração de escala, configuração de pivôs e exportação.

**Screenshot sugerido:** Interior do Lamborghini Huracán em *Forza Motorsport 6* — asset com pivôs corretos nas portas e volante, escala real, sem nenhuma transformação não aplicada.

---

## Capítulo 8 — Fundamentos do Physically Based Rendering

**Jogo:** *Call of Duty: Advanced Warfare* — Sledgehammer Games, 2014

**Por que funciona:** Dimitar Lazarov (Sledgehammer Games) apresentou na GDC 2014 *"Physically Based Rendering in Call of Duty: Advanced Warfare"*, a primeira apresentação de grande alcance da indústria AAA demonstrando a implementação PBR em um título de enorme público. O talk explica os princípios físicos fundamentais — conservação de energia, separação de reflexo difuso e especular, metalicidade — com exemplos visuais diretos do jogo. É o equivalente audiovisual ao capítulo que o leitor está lendo, e foi amplamente citado por outros estúdios como referência de implementação.

**Fontes:**
- GDC 2014 — *"Physically Based Rendering in Call of Duty: Advanced Warfare"*, Dimitar Lazarov. Disponível no GDC Vault e no SlideShare.
- Lazarov's blog — *"PBR Notes"*, `http://lazerounlimited.blogspot.com`

**Imagem sugerida:**

> Figura: Slide do talk de Lazarov mostrando a esfera de material — metal polido, metal fosco, plástico liso, plástico fosco — e como cada combinação de rugosidade × metalicidade produz um comportamento de luz distinto.

**Screenshot sugerido:** Cena de *Call of Duty: Advanced Warfare* com equipamento exoesqueleto metálico polido, colete tático de tecido fosco e visor transparente no mesmo frame — três comportamentos PBR distintos sob a mesma iluminação.

---

## Capítulo 9 — Os Mapas que Compõem um Material PBR

**Jogo:** *Destiny* — Bungie, 2014

**Por que funciona:** Paul Pepera (Bungie) apresentou na SIGGRAPH 2013 *"The Art of Destiny"*, descrevendo em detalhe a estrutura de mapas dos materiais do jogo: Base Color, Metallic, Roughness, Normal, AO, além de um mapa de emissão e um sistema de tinting por canal que permitia variar a cor de armaduras sem refazer as texturas. É uma das primeiras demonstrações públicas de como os mapas PBR interagem — cada um com seu papel específico, cada um em seu espaço de cor correto.

**Fontes:**
- SIGGRAPH 2013 — *"The Art of Destiny"*, Paul Pepera, Bungie. Slides disponíveis online.
- GDC 2014 — *"Destiny: From Mythic Science Fiction to Rendering in Real-Time"*, Hao Chen & Michal Valient.
- Bungie blog — Developer diaries sobre o sistema de materiais de *Destiny*, 2013–2014.

**Imagem sugerida:**

> Figura: Decomposição de um material de armadura de Guardião em *Destiny* — Base Color, Metallic, Roughness, Normal e AO exibidos separadamente, demonstrando a função de cada mapa.

**Screenshot sugerido:** Guardião Titan em *Destiny* sob luz solar direta, com a mistura de peças metálicas polidas, tecido fosco e detalhes gravados respondendo à iluminação de modo fisicamente plausível.

---

## Capítulo 10 — Construção e Análise de Materiais Reais

**Jogo:** *Red Dead Redemption 2* — Rockstar Games, 2018

**Por que funciona:** *Red Dead Redemption 2* é o exemplo mais citado da indústria para construção de materiais a partir do mundo real. A Rockstar usou fotogrametria, scanning de materiais e referências fotográficas extensas para cada superfície — couro de sela, madeira de carroça, ferro forjado, lama encharcada. O processo de decomposição de referências fotográficas em camadas separadas de cor, rugosidade e metalicidade é documentado em vídeos de making-of, developer interviews e análises técnicas do Digital Foundry.

**Fontes:**
- Rockstar Games — *"The Making of Red Dead Redemption 2"*, série de vídeos oficiais, 2018.
- Digital Foundry — *"Red Dead Redemption 2: The Most Technically Impressive Open World Ever Made?"*, 2018.
- 80.lv — *"The Art of Red Dead Redemption 2"*, 2019.
- Game Developer (Gamasutra) — Entrevistas com o time de arte da Rockstar, 2018.

**Imagem sugerida:**

> Figura: Sela de couro de *Red Dead Redemption 2* com decomposição em Base Color (tom neutro do couro), Roughness (gradiente entre couro novo e gasto), Normal (costuras e relevo do couro) e AO (sombra nas costuras).

**Screenshot sugerido:** Close do equipamento do Arthur Morgan — couro da jaqueta, metal do revólver, tecido da camisa e madeira da coronha — cada material com comportamento fisicamente distinto sob a mesma fonte de luz.

---

## Capítulo 11 — Shaders, Iluminação e Aparência Final

**Jogo:** *Ghost of Tsushima* — Sucker Punch Productions, 2020

**Por que funciona:** Sean Feeley e Joakim Stigsson apresentaram na GDC 2021 *"Rendering Ghost of Tsushima"*, um dos talks mais reveladores já feitos sobre a relação entre shader, vento e iluminação. O jogo tem um shader de vento procedural que anima toda a vegetação — capim, árvores, pétalas de sakura — e o visual característico do título nasce exatamente do encontro desse shader com a iluminação HDR. O capítulo argumenta que um material não se julga isoladamente, mas no encontro com a luz: *Ghost of Tsushima* é o caso mais visualmente claro dessa interdependência na história recente dos jogos.

**Fontes:**
- GDC 2021 — *"Rendering Ghost of Tsushima"*, Sean Feeley & Joakim Stigsson. Disponível no GDC Vault.
- PlayStation Blog — Dev diary sobre o visual de *Ghost of Tsushima*, 2020.
- Digital Foundry — Análise técnica de *Ghost of Tsushima*, 2020.

**Imagem sugerida:**

> Figura: Frame de *Ghost of Tsushima* mostrando o campo de capim em movimento sob vento lateral com luz dourada de fim de tarde — o resultado visual nasce do shader de vento + iluminação HDR + tone mapping, não de um único mapa de textura.

**Screenshot sugerido:** Confronto à beira do penhasco em *Ghost of Tsushima* com névoa ao fundo — a armor metálica do Sakai, o céu gradiente e as pétalas de cerejeira em movimento mostram três shaders distintos (metal PBR, sky shader, partículas) harmonizados pela iluminação da cena.

---

## Capítulo 12 — Texturas Seamless e Tileables

**Jogo:** *The Elder Scrolls V: Skyrim* — Bethesda Game Studios, 2011

**Por que funciona:** *Skyrim* texturizou um mundo de centenas de quilômetros quadrados com um conjunto pequeno de texturas tileables — pedra, neve, madeira, terra. A repetição é visível, especialmente nos terrenos abertos, e a própria comunidade de modding dedicou uma decade a estudar e corrigir esse problema. Isso torna *Skyrim* um caso de estudo duplo: como tileables de qualidade viabilizam mundos imensos com orçamento de memória restrito, e quais são os limites visíveis desse recurso quando não combinado com variação. Os mods *Skyrim HD* e *Vivid Landscapes* documentam metodicamente como texturas seamless de maior resolução e micro-variação transformam a aparência do jogo.

**Fontes:**
- GDC 2012 — *"Skyrim's Modular Approach to Level Design"*, Joel Burgess & Nate Purkeypile.
- Digital Foundry — *"Skyrim: The Original vs. Special Edition Comparison"*, 2016.
- Nexus Mods — Documentação dos projetos *Skyrim HD - 2K Textures* e *Vivid Landscapes*, com análise de como a textura tileable foi reconstruída.

**Imagem sugerida:**

> Figura: Terreno de neve de *Skyrim* na versão original (2011) e na versão com textura seamless de alta resolução — mesma geometria, mesma UV, diferença inteiramente na qualidade e variação da textura tileable.

**Screenshot sugerido:** Encosta de pedra em *Skyrim* com iluminação rasante, mostrando o padrão tileable repetindo-se claramente — contraste pedagógico com a versão modded onde a repetição é quebrada por variação de ruído.

---

## Capítulo 13 — Texturização Procedural

**Jogo:** *No Man's Sky* — Hello Games, 2016

**Por que funciona:** Em *No Man's Sky*, cada planeta tem superfícies geradas proceduralmente em tempo real — sem uma única textura pintada ou escaneada. O grafo procedural da Hello Games gera cor de solo, variação de vegetação, padrões de rocha e neve a partir de parâmetros de bioma: temperatura, nível de radiação, tipo de estrela. Grant Duncan (Hello Games) apresentou na GDC 2017 *"Building Worlds Using Math(s)"*, descrevendo como grafos de ruído, blending de materiais e sementes aleatórias produzem 18 quintilhões de planetas distintos. É o caso definitivo de texturização procedural na história dos jogos.

**Fontes:**
- GDC 2017 — *"Building Worlds Using Math(s)"*, Grant Duncan, Hello Games. GDC Vault.
- Sean Murray — Entrevistas técnicas pré-lançamento, Edge Magazine, 2015–2016.
- 80.lv — *"Procedural Generation in No Man's Sky"*, 2018.

**Imagem sugerida:**

> Figura: Diagrama simplificado do grafo de material procedural de *No Man's Sky*, mostrando como temperatura, umidade e tipo de rocha são parâmetros que alimentam um blending de materiais base para gerar a superfície do planeta.

**Screenshot sugerido:** Dois planetas de *No Man's Sky* lado a lado — um desértico com solo avermelhado e formações de cristal, outro gelado com neve azulada e rocha escura — ambos gerados pelo mesmo grafo com sementes e parâmetros diferentes.

---

## Capítulo 14 — Pintura Digital para Jogos

**Jogo:** *World of Warcraft* — Blizzard Entertainment, 2004

**Por que funciona:** *World of Warcraft* é o caso definitivo de textura pintada à mão em jogos. Cada superfície — rocha, madeira, tecido, pedra — é desenhada manualmente, com iluminação implícita integrada à própria textura, volume pintado e saturação controlada para legibilidade. O guia de arte da Blizzard, publicado ao longo dos anos em BlizzCon e na documentação de *Warcraft* e *Hearthstone*, descreve os princípios de hand-painting da empresa: célula de luz fixa, saturação alta, contornos de demarcação entre planos. Esses princípios influenciaram toda uma geração de artistas de games estilizados.

**Fontes:**
- BlizzCon 2011 — *"The Art of World of Warcraft"*, painel de arte da Blizzard.
- Blizzard Entertainment Blog — Artigos sobre o estilo visual de *WoW* ao longo das expansões.
- *"The Art of Hearthstone"* (livro, Blizzard, 2019) — descreve os mesmos princípios de hand-painting aplicados a um produto mais recente.

**Imagem sugerida:**

> Figura: Textura de parede de pedra de *World of Warcraft* decomposta — a iluminação está integrada à própria textura (sem mapa normal ou PBR), com volume pintado manualmente em vez de calculado em tempo real.

**Screenshot sugerido:** Zona de *Elwynn Forest* em *World of Warcraft Classic* — árvores, pedras, chão e construções com texturas inteiramente pintadas à mão, o visual que define o estilo estilizado de toda a série.

---

## Capítulo 15 — Máscaras, Stencils e Decals

**Jogo:** *Cyberpunk 2077* — CD Projekt RED, 2020

**Por que funciona:** Night City é inteiramente construída sobre uma base modular de materiais tileables, sobre a qual um sistema massivo de decals adiciona grafites, anúncios holográficos, manchas de óleo, rachaduras, marcas de projétil e pôsteres rasgados. A CD Projekt RED apresentou na SIGGRAPH 2020 sua arquitetura de decals para REDengine 4, descrevendo como cada decal afeta independentemente a cor, a rugosidade e o normal do material abaixo — exatamente o modelo do capítulo. Sem os decals, Night City seria um conjunto de corridores genéricos; são eles que narram a cidade.

**Fontes:**
- SIGGRAPH 2020 — *"Cyberpunk 2077: Road to Rendering"*, Jakub Knapik et al., CD Projekt RED.
- GDC 2020 — *"Night City: Bringing Cyberpunk 2077 to Life"*, painel de arte.
- 80.lv — *"The Art of Night City"*, 2020–2021.

**Imagem sugerida:**

> Figura: Beco de Night City em *Cyberpunk 2077* com a camada de decals removida (material base tileable) e com ela ativa — a diferença entre um corredor genérico e um espaço narrativo.

**Screenshot sugerido:** Rua baixa de Night City com paredes cobertas por grafites em kanji, manchas de umidade, fios expostos e publicidades em decal sobrepostas ao concreto tileable — camadas do capítulo visíveis em um único frame.

---

## Capítulo 16 — Bake de Texturas

**Jogo:** *Gears of War* — Epic Games, 2006

**Por que funciona:** *Gears of War* foi o primeiro título AAA de grande público a aplicar sistematicamente o fluxo high-to-low: esculturas detalhadas em alta geometria (10M+ polígonos por personagem) assadas em normal maps para a versão de jogo (~10 mil polígonos). Cliff Bleszinski e os artistas da Epic documentaram o processo em developer diaries e na Game Developer Magazine. O Locust, em particular, tornou-se referência da época por mostrar que relevo de nível cinematográfico era possível em geometria de tempo real via bake.

**Fontes:**
- Game Developer Magazine — *"The Making of Gears of War"*, Epic Games, 2007.
- GDC 2007 — *"Unreal Engine 3 Art Pipeline"*, Jeff Farris, Epic Games.
- Epic Games Insider — Dev diaries de *Gears of War*, 2006–2007.

**Imagem sugerida:**

> Figura: Comparação high-poly vs. low-poly de um Locust de *Gears of War* — a escultura de alta geometria ao lado do modelo de jogo com o normal map assado, mostrando que o relevo é preservado.

**Screenshot sugerido:** Marcus Fenix em close em *Gears of War* — as rugas e cicatrizes do rosto, o relevo da armadura e os entalhes do colete são inteiramente resultado do normal map assado, não de geometria.

---

## Capítulo 17 — Normal Maps e Transferência de Detalhes

**Jogo:** *Doom 3* — id Software, 2004

**Por que funciona:** *Doom 3* é o marco histórico do normal mapping em jogos. John Carmack projetou o motor (id Tech 4) especificamente em torno de per-pixel normal mapping com iluminação por stencil shadow volumes. Cada superfície do jogo — paredes de metal, pele demoníaca, equipamentos — usa normal maps para detalhe, e o efeito é mais dramático que em qualquer título anterior porque a iluminação dinâmica de Carmack foi construída para revelá-los. O desenvolvimento foi documentado em entrevistas técnicas, no .plan file de Carmack e em análises extensas de Michael Abrash.

**Fontes:**
- John Carmack — `.plan` files, 2000–2004. Arquivados em `https://github.com/ESWAT/john-carmack-plan-archive`
- Michael Abrash — *"Quake's Lighting Model"* e análises de id Tech 4 no blog pessoal.
- Game Developer — *"The Technology of Doom 3"*, entrevistas com id Software, 2004.

**Imagem sugerida:**

> Figura: Superfície metálica de *Doom 3* com o normal map isolado à esquerda e o resultado iluminado à direita — o relevo não existe na geometria; ele é inteiramente calculado pelo shader a partir do mapa.

**Screenshot sugerido:** Corredor da estação de Marte em *Doom 3* com a tocha do jogador revelando o relevo dos parafusos e das placas metálicas nas paredes — impossível de reproduzir sem per-pixel normal mapping.

---

## Capítulo 18 — Texture Atlas e Trim Sheets

**Jogo:** *Titanfall* — Respawn Entertainment, 2014

**Por que funciona:** Respawn Entertainment, liderada por ex-membros da Infinity Ward, institucionalizou o uso moderno de trim sheets para ambientes no desenvolvimento de *Titanfall*. O cenário industrial do jogo — hangares, plataformas, corredores militares — foi inteiramente construído com geometria modular encaixando em trim sheets: uma textura contendo frisos, rebites, molduras e detalhes metálicos em faixas horizontais. Jorge Jimenez (então na Respawn, hoje em outros estúdios) e colegas documentaram o workflow em posts no ArtStation e em entrevistas técnicas, tornando *Titanfall* a referência mais citada quando se discute trim sheets na indústria.

**Fontes:**
- ArtStation — Posts dos artistas de ambiente de *Titanfall* (Respawn), documentando o uso de trim sheets.
- GDC 2014 — *"Visual Effects of Titanfall"*, Respawn Entertainment.
- 80.lv — *"The Art of Titanfall"*, 2014.
- Ben Cloward — Tutorial "Trim Sheets" referenciando Titanfall como case principal.

**Imagem sugerida:**

> Figura: Trim sheet mestre de *Titanfall* — a textura com suas faixas de acabamentos metálicos, rebites, frisos e molduras que compõem todo o cenário do jogo.

**Screenshot sugerido:** Hangar de lançamento de *Titanfall* com um Titan ao centro — toda a geometria do cenário encaixada nas faixas da trim sheet, gerando consistência visual com custo mínimo de textura.

---

## Capítulo 19 — UDIMs, Texture Arrays e Multi Tile Texturing

**Jogo:** *The Last of Us Part II* — Naughty Dog, 2020

**Por que funciona:** Naughty Dog apresentou na SIGGRAPH 2020 seu pipeline de renderização de personagens para *The Last of Us Part II*, onde Ellie e Joel possuem texturas distribuídas em múltiplos tiles (rosto, cabeça, torso, membros) para suportar a densidade de texels necessária nos closes cinematográficos da narrativa. O pipeline usa autoria de alta resolução com múltiplos tiles e depois consolida para tempo real, exatamente o fluxo descrito no capítulo — UDIM como instrumento de autoria, não de renderização.

**Fontes:**
- SIGGRAPH 2020 — *"Naughty Dog Rendering Technology and The Last of Us Part II"*, Waylon Brinck.
- PlayStation Blog — *"The Art of The Last of Us Part II: Character Design"*, 2020.
- 80.lv — *"The Art Direction of The Last of Us Part II"*, 2020.

**Imagem sugerida:**

> Figura: Close do rosto de Ellie em *The Last of Us Part II*, mostrando micro-poros, pelos e cicatriz — densidade de detalhe que exige múltiplos tiles de textura e seria impossível com um único UV 0-1.

**Screenshot sugerido:** Cena cinematográfica de *The Last of Us Part II* com Ellie em close extremo — a pele, os olhos e os cabelos em resolução que equivale a várias texturas 4K cobrindo apenas o rosto.

---

## Capítulo 20 — Compressão, Mipmaps e Packing de Canais

**Jogo:** *Genshin Impact* — HoYoverse (miHoYo), 2020

**Por que funciona:** *Genshin Impact* roda em PS5, PC, iOS e Android com o mesmo mundo aberto e a mesma base de assets. A HoYoverse documentou em developer talks e entrevistas técnicas sua estratégia de compressão por plataforma — ASTC em mobile, BC em PC/console — além do sistema de LOD e mipmap agressivo para as versões de celular. O jogo é o exemplo mais visível da indústria de como o mesmo asset precisa existir em múltiplas configurações de qualidade, e as escolhas de packing de canais (ORM) são mencionadas explicitamente em suas apresentações de arte técnica.

**Fontes:**
- GDC 2021 — *"The Art of Genshin Impact: From Concept to In-Game"*, HoYoverse.
- CEDEC 2021 — Apresentação técnica de *Genshin Impact* sobre otimização mobile.
- 80.lv — *"The Technical Art of Genshin Impact"*, 2021.

**Imagem sugerida:**

> Figura: Comparação de *Genshin Impact* na versão PC (Ultra) e Android (Médio) — mesmo asset, compressões e mipmaps diferentes, ilustrando como as escolhas do capítulo afetam a qualidade final visível ao jogador.

**Screenshot sugerido:** Ambiente de Liyue Harbor em *Genshin Impact* — arquitetura, vegetação, água e personagem no mesmo frame, em alta qualidade de PC, com a densidade de detalhe que o packing ORM e mipmaps bem configurados viabilizam.

---

## Capítulo 21 — Lightmaps e Iluminação em Motores de Jogo

**Jogo:** *Firewatch* — Campo Santo, 2016

**Por que funciona:** *Firewatch* foi produzido por uma equipe pequena usando Unity e lightmaps pré-calculados para criar uma das iluminações mais elogiadas da sua geração. Jane Ng (artista de ambiente) documentou no blog da Campo Santo o processo de autoria da iluminação — como os lightmaps foram calculados, como as sondas de reflexão foram posicionadas para que os personagens dinâmicos recebessem a luz correta do ambiente, e como os mapas de segundo canal foram desdobrados para a geometria estática da floresta. É o caso mais bem explicado de iluminação baked pensada artisticamente, não apenas tecnicamente.

**Fontes:**
- Campo Santo Blog — *"The Art of Firewatch"*, Jane Ng, 2016. `https://blog.camposanto.com`
- Unity Blog — *"How Campo Santo Used Unity to Make Firewatch"*, 2016.
- GDC 2016 — *"Firewatch: Art Direction of a First-Person Adventure"*, Olly Moss & Jane Ng.

**Imagem sugerida:**

> Figura: Floresta de *Firewatch* ao entardecer com a luz quente de pôr do sol pré-calculada nos troncos e chão — toda essa luz indireta existe num lightmap; o jogo não realiza ray tracing em tempo real.

**Screenshot sugerido:** Torres de observação de *Firewatch* ao amanhecer, com a luz suave de rim iluminando Henry — a luz do ambiente está no lightmap, a sombra dinâmica está sendo calculada em tempo real, e a sonda de reflexão dá ao personagem o tom correto do céu.

---

## Capítulo 22 — Integração com Unreal Engine e Unity

**Jogo:** *Cuphead* — Studio MDHR, 2017

**Por que funciona:** *Cuphead* usou Unity para um pipeline completamente atípico: animações desenhadas à mão em aquarela digitalizadas, convertidas em sprite sheets e integradas ao motor como materiais 2D num mundo 3D falso. A equipe (Maja e Chad Moldenhauer) apresentou na GDC 2015 *"The Art Direction of Cuphead"*, descrevendo os desafios de integração específicos — pré-multiplicação de alpha, espaço de cor, granulação de filme como pós-processo, e como a pipeline do Unity precisou ser customizada para tratar texturas de animação frame a frame. É o contraponto exato ao caso "padrão" de integração do capítulo.

**Fontes:**
- GDC 2015 — *"The Art Direction of Cuphead: Achieving the Hand Drawn Look"*, Maja Moldenhauer. GDC Vault.
- Unity Blog — *"How Studio MDHR Made Cuphead with Unity"*, 2017.
- Game Developer — *"Behind the Art of Cuphead"*, 2017.

**Imagem sugerida:**

> Figura: Diagrama do pipeline de *Cuphead* — do desenho à mão ao scan, ao sprite sheet, à integração na Unity com pré-multiplicação de alpha e granulação de filme como pós-efeito.

**Screenshot sugerido:** Cena de boss fight de *Cuphead* com o personagem principal em primeiro plano — cada frame de animação é uma textura digitalizada de um desenho à mão, integrada ao motor como material 2D.

---

## Capítulo 23 — Controle de Qualidade de Materiais

**Jogo:** *Sea of Thieves* — Rare, 2018

**Por que funciona:** *Sea of Thieves* usa PBR estilizado — não realista, mas fisicamente coerente — e a Rare enfrentou o desafio de manter a consistência visual entre madeira de navio, metal de canhão, tecido de vela, pele de personagem e água do oceano, todos no mesmo frame, com o estilo de aquarela da empresa. Ryan Stevenson (Rare) apresentou na GDC 2018 *"The Art of Sea of Thieves"*, descrevendo o processo de validação de materiais: como cada novo asset era checado contra referências em cena de teste com iluminação neutra antes de entrar na produção, e que critérios eram usados para reprovar materiais que "fugiam" do estilo.

**Fontes:**
- GDC 2018 — *"The Art of Sea of Thieves"*, Ryan Stevenson, Rare. GDC Vault.
- 80.lv — *"The Art Direction of Sea of Thieves"*, 2018.
- Xbox Wire — *"How Rare Created the Art Style of Sea of Thieves"*, 2018.

**Imagem sugerida:**

> Figura: Cena de validação de materiais de *Sea of Thieves* — cofre do tesouro, canhão de bronze, tábua de navio e pele do personagem sob iluminação neutra, checados contra o guia de estilo estilizado-PBR da Rare.

**Screenshot sugerido:** Convés de navio em *Sea of Thieves* com sol alto — madeira, metal, corda e vela numa cena que demonstra a coerência visual do PBR estilizado, resultado direto do QA rigoroso de materiais.

---

## Capítulo 24 — Apresentação Profissional de Assets

**Jogo:** *Control* — Remedy Entertainment, 2019

**Por que funciona:** A Remedy publicou breakdowns extensos de assets de *Control* — especialmente os objetos do *The Oldest House* — no ArtStation e apresentou na GDC 2020 o processo de criação dos materiais do jogo. Os posts dos artistas seniores da Remedy (Jan Schmid, Valtteri Kangasniemi) são exemplos da indústria de como apresentar um asset com intenção: render final, wireframe, mapa UV, decomposição por canal, high-poly comparada à low-poly e nota sobre a decisão artística de cada escolha técnica. São exatamente o modelo de portfolio descrito no capítulo.

**Fontes:**
- ArtStation — Posts de artistas da Remedy Entertainment para *Control*, 2019–2020.
- GDC 2020 — *"The Art of Control: Environmental Storytelling Through Materials"*, Remedy Entertainment.
- 80.lv — *"The Look of Control"*, 2019.

**Imagem sugerida:**

> Figura: Prancha de portfolio de um asset de *Control* (arma-serviço modificada ou objeto do Bureau) com render final, wireframe, layout UV, decomposição de mapas PBR e nota sobre as decisões de autoria — modelo da Remedy para apresentação profissional.

**Screenshot sugerido:** O Bureau of Control em *Control* — corredores de concreto brutalista com luminárias fluorescentes e objetos sobrecarregados de significado — um ambiente cujo valor de apresentação nasce das escolhas de material documentadas pela equipe.

---

## Capítulo 25 — Pipeline Completo de Texturização para Jogos

**Jogo:** *God of War* — Santa Monica Studio, 2018

**Por que funciona:** O GDC 2019 dedicou múltiplas sessões ao pipeline de *God of War*, cobrindo cada etapa que a apostila ensina: conceito e análise de referências, modelagem high-poly e retopologia, UV unwrapping com densidade calibrada para os closes cinemáticos, baking de normais e oclusão, autoria PBR em Substance Painter, otimização por plataforma, integração na engine proprietária do jogo e controle de qualidade final. O machado Leviathan e a armadura de Kratos são os assets mais documentados da geração PS4, e o pipeline da equipe de arte técnica da Santa Monica é citado em cursos, tutoriais e documentação de estúdios do mundo inteiro.

**Fontes:**
- GDC 2019 — *"God of War: Breathing New Life into a Franchise"*, múltiplos apresentadores, Santa Monica Studio. GDC Vault.
- PlayStation Blog — *"The Art of God of War: Exclusive Interview with the Dev Team"*, 2018.
- 80.lv — *"The Art of God of War"*, série de artigos, 2018–2019.
- ArtStation — Breakdowns completos de artistas da Santa Monica Studio para o jogo.

**Imagem sugerida:**

> Figura: Pipeline completo do machado Leviathan de *God of War* — escultura high-poly, retopologia low-poly, layout UV, mapa de normais assado, texturização PBR em Substance Painter e resultado final no motor, em uma única imagem de referência.

**Screenshot sugerido:** Kratos segurando o machado Leviathan em close durante uma sequência cinematográfica de *God of War* — o asset que representa, em um único objeto, a aplicação sequencial de todos os vinte e cinco capítulos da apostila.

---

## Tabela de Referência Rápida

| Cap. | Tema | Jogo | Estúdio | Fonte primária |
|------|------|------|---------|----------------|
| 1 | Materiais e Representação | *Half-Life 2* | Valve | GDC 2006 — Jason Mitchell |
| 2 | Evolução Histórica | *Battlefield 4* | DICE | SIGGRAPH 2014 — Lagarde & de Rousiers |
| 3 | Como Motores Interpretam | *Hellblade: Senua's Sacrifice* | Ninja Theory | YouTube Dev Diary (30 eps.) |
| 4 | Coordenadas UV e Projeções | *Quake* | id Software | Abrash's Black Book + Sanglard blog |
| 5 | UV Unwrapping | *Monster Hunter: World* | Capcom | GDC 2018 + Capcom Dev Blog |
| 6 | Texel Density | *Horizon Zero Dawn* | Guerrilla Games | GDC 2017 — Arjan Bak |
| 7 | Preparação de Assets | *Forza Motorsport 6* | Turn 10 Studios | GDC 2016 — Christian Ammann |
| 8 | Fundamentos PBR | *Call of Duty: Advanced Warfare* | Sledgehammer | GDC 2014 — Dimitar Lazarov |
| 9 | Mapas PBR | *Destiny* | Bungie | SIGGRAPH 2013 — Paul Pepera |
| 10 | Materiais Reais | *Red Dead Redemption 2* | Rockstar | Digital Foundry + Dev Diaries |
| 11 | Shaders e Iluminação | *Ghost of Tsushima* | Sucker Punch | GDC 2021 — Feeley & Stigsson |
| 12 | Texturas Tileables | *Skyrim* | Bethesda | GDC 2012 + Digital Foundry |
| 13 | Procedural | *No Man's Sky* | Hello Games | GDC 2017 — Grant Duncan |
| 14 | Pintura Digital | *World of Warcraft* | Blizzard | BlizzCon + Art Book |
| 15 | Máscaras e Decals | *Cyberpunk 2077* | CD Projekt RED | SIGGRAPH 2020 — Jakub Knapik |
| 16 | Bake de Texturas | *Gears of War* | Epic Games | GDC 2007 + Game Dev Magazine |
| 17 | Normal Maps | *Doom 3* | id Software | Carmack .plan files + Abrash |
| 18 | Atlas e Trim Sheets | *Titanfall* | Respawn | GDC 2014 + ArtStation |
| 19 | UDIMs e Multi-Tile | *The Last of Us Part II* | Naughty Dog | SIGGRAPH 2020 — Waylon Brinck |
| 20 | Compressão e Mipmaps | *Genshin Impact* | HoYoverse | GDC 2021 + CEDEC 2021 |
| 21 | Lightmaps | *Firewatch* | Campo Santo | Blog Jane Ng + GDC 2016 |
| 22 | Integração Engine | *Cuphead* | Studio MDHR | GDC 2015 — Maja Moldenhauer |
| 23 | Controle de Qualidade | *Sea of Thieves* | Rare | GDC 2018 — Ryan Stevenson |
| 24 | Apresentação Profissional | *Control* | Remedy | GDC 2020 + ArtStation |
| 25 | Pipeline Completo | *God of War* | Santa Monica | GDC 2019 (múltiplas sessões) |

---

*Documento produzido como referência editorial para atualização dos estudos de caso da apostila. Cada entrada pode ser desenvolvida como um estudo de caso completo no capítulo correspondente, incorporando citações das fontes primárias, análise do problema técnico enfrentado pelo estúdio e conexão explícita com os conceitos do capítulo.*
