# Capítulo 22 — Integração com Unreal Engine e Unity

## Introdução

Até este ponto da apostila, os motores de jogo apareceram como destino e como exemplo: o lugar onde as texturas são importadas, marcadas, comprimidas e iluminadas. Este capítulo encara o motor de frente, não para ensinar a interface de um programa específico — o que contraria as diretrizes desta obra e envelheceria a cada atualização —, mas para examinar **o que significa integrar um** *asset* **texturizado a um motor de jogo**, quais conceitos governam essa passagem e em que os dois motores dominantes, a Unreal Engine e a Unity, se assemelham e se distinguem. O objetivo é que o estudante compreenda os princípios que valem em qualquer motor, e saiba traduzi-los para a ferramenta que tiver à frente.

A integração é o momento da verdade da texturização. Todo o trabalho das partes anteriores — o desdobramento UV, a calibração PBR, a produção dos mapas, o *baking*, a organização e a compressão — converge para o instante em que o *asset* deixa a ferramenta de autoria e passa a viver no motor, sob a luz da cena (Capítulo 21), sujeito ao orçamento da plataforma e às convenções do motor de jogo. É aqui que erros silenciosos cometidos lá atrás se revelam, que materiais que pareciam corretos no Substance Painter mostram-se errados sob o sistema de iluminação do jogo, e que a disciplina de nomenclatura, espaço de cor e organização das texturas (Parte V) é recompensada ou cobrada. Compreender a integração é, portanto, compreender como todas as decisões anteriores se materializam — e por que o profissional precisa conhecer não só a autoria, mas também o destino.

## Desenvolvimento

### O que é, afinal, integrar um asset

Integrar um *asset* texturizado a um motor envolve três movimentos encadeados. Primeiro, **importar** a malha e suas texturas, traduzindo os arquivos da ferramenta de autoria para os formatos e as convenções internas do motor — e é aqui que entram as decisões de compressão, mipmaps e espaço de cor do Capítulo 20, pois é na importação que cada canal de textura é marcado como cor ou dado, sRGB ou linear. Segundo, **construir o material**: montar, dentro do motor, a rede que liga cada textura à entrada correspondente do sombreador — a cor base à cor base, o mapa de normais às normais, o mapa empacotado ORM às entradas de oclusão, mapa de rugosidade e mapa metálico (Capítulo 20). Terceiro, **instanciar e posicionar** o *asset* na cena, onde ele passa a receber a iluminação (Capítulo 21) e a compor o mundo do jogo. Cada um desses movimentos pode dar errado, e cada um exige que o *asset* tenha sido preparado corretamente na autoria.

> 📘 **Definição**
> **Sombreador** (*shader*) é o programa executado pela GPU responsável por calcular a aparência visual de cada fragmento de uma superfície, combinando as informações dos canais de textura com os parâmetros de iluminação para produzir o pixel final. No contexto dos motores de jogo, o sombreador PBR recebe entradas padronizadas — cor base, mapa de rugosidade, mapa metálico, mapa de normais — e calcula a resposta física da superfície à luz da cena.

O conceito central que organiza essa passagem é o de **material** como entidade do motor. Já vimos, na Parte III, que o material é a descrição completa da aparência de uma superfície e que o sombreador é o programa que a calcula (Capítulo 11). No motor, o material assume uma forma concreta: um objeto editável em que se ligam texturas, se ajustam parâmetros e se define o modelo de sombreamento. Tanto a Unreal quanto a Unity organizam-se em torno desse objeto, e ambos adotam o paradigma **PBR** estudado na Parte III — o que significa que um material construído com a disciplina física da Parte III tende a funcionar de modo previsível em qualquer dos dois, com os mesmos mapas cumprindo os mesmos papéis. Essa convergência é o que permite falar de princípios independentes de software: por baixo das interfaces distintas, os dois motores pedem as mesmas informações da superfície.

### Materiais e instâncias: a economia da reutilização

Um conceito decisivo na integração, comum aos dois motores, é a distinção entre o **material** e suas **instâncias**. Construir um material — montar a rede de texturas e a lógica do sombreador — é trabalhoso e, do ponto de vista do motor, custoso de compilar. Seria um desperdício refazer esse trabalho para cada variação de uma superfície. Por isso os motores permitem criar um **material base** (ou material-pai) com parâmetros expostos — uma cor que se pode trocar, uma intensidade de mapa de rugosidade que se pode ajustar, um canal de textura que se pode substituir — e dele derivar **instâncias** (ou variantes) que reaproveitam toda a lógica do base, alterando apenas os parâmetros. De um material de metal pintado, por exemplo, derivam-se dezenas de instâncias — vermelho, azul, enferrujado, novo — sem reconstruir nada, apenas trocando valores.

> 💡 **Dica Profissional**
> Essa arquitetura ecoa, no domínio dos materiais, a mesma filosofia de **reutilização extrema** que percorreu a Parte V — os *trim sheets*, os atlas, o *hotspot* (Capítulo 18). O ganho é o mesmo: produzir muito a partir de pouco, economizando trabalho de autoria, memória e, no caso das instâncias de material, tempo de compilação e de troca de estado na GPU. A Unreal chama essas variantes de *Material Instances*; a Unity, conforme o sistema, de *Material Variants* ou de materiais que compartilham um sombreador. O nome muda; a lógica permanece a mesma.

> **Figura 22.1** — Um material base de "metal pintado" com seus parâmetros expostos (cor, mapa de rugosidade, presença de ferrugem) e, ramificando-se dele, várias instâncias que apenas alteram esses valores — todas compartilhando a mesma rede de sombreador —, ilustrando a economia da reutilização por instâncias.

**Screenshot sugerido:** Captura de um material base e de suas instâncias em um motor, com o painel de parâmetros expostos de uma instância em destaque.

### Os dois grandes pontos de atrito: espaço de cor e convenção de normais

Embora os dois motores compartilhem o paradigma PBR, a integração tropeça com frequência em dois pontos técnicos que o texturizador precisa conhecer, ambos já anunciados em partes anteriores. O primeiro é o **espaço de cor**, a regra de cor versus dados do Capítulo 20: na importação, cada canal de textura deve ser marcado corretamente — a cor base como cor (sRGB), os mapas de dados (mapa de normais, mapa de rugosidade, mapa metálico, mapa de oclusão ambiente, mapas empacotados) como lineares. Errar essa marcação produz o "erro silencioso" já descrito: o material não quebra, apenas fica sutilmente errado. Os dois motores oferecem controles de importação para isso, e a disciplina de marcá-los corretamente é a mesma em ambos — mas os padrões e os nomes diferem, e o profissional precisa verificar, não confiar nos automatismos.

O segundo ponto de atrito, mais célebre, é a **convenção do mapa de normais**, já mencionada na Parte V. Existem duas convenções para o canal verde de um mapa de normais — uma usada pela tradição **OpenGL**, outra pela **DirectX** —, que diferem por inverter o sentido do eixo vertical. A consequência prática é direta: um mapa de normais produzido para uma convenção, usado num motor que espera a outra, produz um relevo **invertido** — as saliências parecem reentrâncias e vice-versa, e a iluminação "afunda" onde deveria sobressair.

> 📘 **Definição**
> As convenções **OpenGL** e **DirectX** para o mapa de normais diferem no sinal do canal verde (eixo Y): na convenção OpenGL, o canal verde aponta "para cima" no espaço de textura; na convenção DirectX, ele aponta "para baixo". O efeito visual de trocar as convenções é relevo completamente invertido — o que deveria projetar-se parece afundar, e vice-versa.

A regra de bolso que a indústria consagrou é que a **Unreal** tradicionalmente espera a convenção **DirectX** e a **Unity**, a **OpenGL** — de modo que um mesmo mapa de normais pode precisar ter seu canal verde invertido ao migrar de um motor para o outro, ou ser exportado já na convenção do destino.

> ❌ **Erro Comum**
> Relevo invertido em um *asset* importado é, em quase todos os casos, o sintoma de convenção de mapa de normais trocada — exportar o mapa para a convenção OpenGL e usá-lo na Unreal, ou o inverso. É um dos erros de integração mais comuns e, felizmente, um dos mais fáceis de diagnosticar: basta verificar se as protuberâncias parecem côncavas sob iluminação rasante.

### Diferenças de filosofia e o que delas decorre

Para além desses pontos técnicos, Unreal e Unity diferem em filosofia, e essas diferenças importam para a integração. A Unreal foi historicamente orientada à **alta fidelidade** e a produções de grande porte, com um editor de materiais baseado em **nós** (uma rede visual de operações conectadas, parente próxima dos editores procedurais do Capítulo 13) que dá enorme controle ao artista, e com sistemas avançados de iluminação e renderização que favorecem o realismo. A Unity, por sua vez, construiu sua reputação na **flexibilidade** e na ampla cobertura de plataformas — do celular ao console —, com uma arquitetura que oferece diferentes *pipelines* de renderização conforme o alvo: um voltado ao desempenho em *hardware* modesto, outro à alta fidelidade.

> ⚠️ **Atenção**
> A multiplicidade de *pipelines* de renderização da Unity é um ponto de atenção importante na integração: um material construído para um *pipeline* pode não funcionar em outro, aparecendo quebrado ou com cor magenta — o sinal de que o motor não conseguiu encontrar o sombreador adequado. O texturizador precisa saber, antes de começar, para qual *pipeline* da Unity está produzindo.

*Among Us* (Innersloth, 2018) e *Hollow Knight* (Team Cherry, 2017) são exemplos de títulos construídos no *pipeline* Built-in (legado) da Unity — o de menor curva de aprendizado —, demonstrando que o *pipeline* correto para um projeto é o que serve ao estilo e ao orçamento da equipe, não necessariamente o mais sofisticado.

Apesar das diferenças, o aprendizado central deste capítulo é que **os princípios transcendem o motor**. Um *asset* bem desdobrado, com texturas PBR fisicamente calibradas, mapas corretamente marcados como cor ou dado, mapa de normais na convenção certa e organização eficiente, integra-se bem a qualquer dos dois — e a um terceiro, como a Godot, ou a um motor proprietário de estúdio. As interfaces, os nomes dos nós e os padrões de importação mudam; o que não muda é o que a superfície precisa informar ao sombreador e as regras físicas que governam essa informação. Por isso esta apostila insiste em conceitos independentes de software: quem os domina aprende qualquer motor rapidamente, ao passo que quem decorou cliques de uma interface fica preso a uma versão de um programa. Integrar bem é, antes de tudo, ter preparado bem — e o preparo é tudo o que as partes anteriores ensinaram.

## Aplicação em Jogos

Na prática de estúdio, a integração é regida por **convenções de projeto** que padronizam a passagem da autoria ao motor: nomenclatura de arquivos e de canais (para que cada canal de textura seja reconhecido automaticamente), formatos de exportação acordados (frequentemente o mapa empacotado ORM do Capítulo 20, na ordem de canais que o motor espera), convenção de mapa de normais definida pelo motor de destino e estrutura de materiais base e instâncias planejada de antemão. Essas convenções existem porque a integração, feita à mão *asset* por *asset*, não escala: um jogo tem milhares de texturas, e só a padronização permite importá-las e montá-las de modo previsível. Boa parte do trabalho de um *technical artist* — a função que faz a ponte entre arte e programação — consiste em criar essas convenções e as ferramentas que automatizam a importação e a construção de materiais.

> 💡 **Dica Profissional**
> A Insomniac Games é conhecida pela maturidade de suas descrições públicas dessa função: postagens de emprego e apresentações técnicas documentam uma equipe que combina domínio de pipelines de produção com automação por scripts, exemplificando como o *technical artist* transforma a integração manual e frágil numa operação automática e auditável.

A escolha do motor molda decisões de produção que retroagem sobre a texturização. Um jogo voltado a celular, feito na Unity com um *pipeline* leve, imporá texturas mais comprimidas, materiais mais simples e packing agressivo (Capítulo 20); um jogo de alta fidelidade para PC e console, feito na Unreal, comportará materiais por nós mais ricos, maior resolução e mais mapas. O texturizador competente pergunta, antes de produzir, qual o motor e o *pipeline* de destino, pois disso dependem o orçamento e as convenções de tudo o que fará.

## Estudo de Caso

### *Cuphead* (Studio MDHR, 2017) — Integração de um Estilo Visual Radicalmente Incomum a um Motor Moderno

*Cuphead* é um dos estudos de caso mais citados quando se discute integração de estilo visual personalizado em motor de jogo, precisamente porque o jogo desafia todo o paradigma PBR descrito nesta apostila — e o supera através de uma integração cuidadosamente arquitetada. O Studio MDHR adotou como objetivo produzir um jogo de ação que reproduzisse fielmente a estética das animações de estúdio dos anos 1930 — aquarela nos fundos, linha de tinta nos personagens, granulação de película, paleta de cores envelhecida. Não havia motor PBR no mundo que entregasse isso por padrão, e a integração foi documentada por Maja Moldenhauer (GDC 2015 e entrevistas posteriores) como um exercício de dobrar o motor a uma necessidade artística radical, não de adaptar a arte às convenções do motor.

A decisão central foi adotar a Unity como plataforma e construir sobre ela materiais e sombreadores completamente customizados que implementassem cada característica visual do estilo: o contorno animado dos personagens desenhado à mão sobre papel, digitalizado e animado; os fundos de aquarela pintados fisicamente, fotografados e compostos em camadas; os efeitos de granulação de película aplicados em pós-processamento. Nenhum desses elementos usa texturas PBR com mapas de rugosidade ou mapa metálico — usam mapas de cor, mapas de transparência e máscaras de mistura que o sombreador customizado interpreta segundo a lógica do estilo, não a da física.

O que conecta o caso ao conteúdo deste capítulo é que os princípios de integração foram respeitados mesmo nesse contexto incomum: cada canal de textura foi importado com as marcações corretas de espaço de cor, a estrutura de material base com instâncias foi usada para gerar as variantes de personagens e cenários, e as convenções de nomenclatura e exportação do projeto foram documentadas para garantir que a equipe pequena mantivesse consistência ao longo dos anos de produção.

O que *Cuphead* ensina sobre integração é, paradoxalmente, o princípio mais universal do capítulo: o motor é uma plataforma, não um estilo. Conhecer os princípios de importação, construção de materiais, instâncias e convenções técnicas permite integrá-los tanto a um jogo PBR de alta fidelidade quanto a uma recriação de animação dos anos 1930 — porque essas são as perguntas que qualquer motor precisa que o artista responda: onde está cada informação de textura, como ela está marcada, de onde o sombreador deve ler o quê. *Cuphead* respondeu a essas perguntas com um vocabulário completamente diferente do PBR, mas com o mesmo rigor técnico que qualquer integração bem-sucedida exige.

**O que aprender com isso:** Os princípios de integração — espaço de cor correto, estrutura de material base e instâncias, convenções de nomenclatura e exportação — são independentes do paradigma visual adotado. Um estilo não-PBR não dispensa rigor técnico; apenas o direciona para convenções diferentes das físicas.

> **Figura 22.2** — Diagrama de integração de *Cuphead* mostrando o fluxo de um personagem: da animação desenhada à mão digitalizada às texturas compostas em camadas no motor, com o sombreador customizado ligando cada mapa (linha, aquarela, máscara de transparência) a uma saída visual — em contraste com o diagrama de integração PBR padrão do capítulo.

**Screenshot sugerido:** Comparação de duas telas de *Cuphead* — um personagem em ação e um fundo de cenário — mostrando a coerência do estilo e a qualidade da integração visual. Fonte: galeria oficial em cupheadgame.com ou modo captura do jogo (Steam).

**Fonte:** Maja Moldenhauer, "Cuphead: A Year of Bugs and Fixes" (GDC 2015); Chad e Jared Moldenhauer, entrevistas técnicas em *80.lv* e Gamasutra (2017); análises técnicas de Digital Foundry.

## Boas Práticas

Verifique sempre, na importação, o **espaço de cor** de cada canal de textura — cor base como sRGB, todos os mapas de dados como lineares — em vez de confiar nos automatismos do motor, pois o erro de marcação é silencioso e os padrões diferem entre Unreal e Unity. Conheça a **convenção do mapa de normais** do motor de destino (DirectX na Unreal, OpenGL na Unity, como regra de bolso) e exporte o mapa de normais já na convenção certa, ou inverta seu canal verde ao migrar; lembre-se de que relevo invertido é o sintoma inequívoco desse erro. Estruture seus materiais em **base e instâncias** sempre que houver variações de uma superfície, reaproveitando a lógica do sombreador e economizando trabalho e custo de renderização — a mesma filosofia de reutilização da Parte V.

Pergunte, antes de produzir, qual o **motor e o** *pipeline* **de destino**, pois deles dependem o orçamento de texturas, a resolução, o número de mapas e as convenções; um *asset* para Unity móvel e um para Unreal de alta fidelidade exigem preparos diferentes a partir do mesmo conceito. Adote e respeite as **convenções de nomenclatura e exportação** do projeto, que permitem a importação automática e a construção previsível de materiais em escala. E mantenha presente o princípio que organiza todo o capítulo: prepare o *asset* segundo os princípios independentes de software — desdobramento UV limpo, PBR calibrado, marcação correta, organização eficiente — e a integração a qualquer motor será uma tradução, não uma reconstrução.

## Erros Comuns

O erro mais emblemático da integração é o **relevo invertido** por convenção de mapa de normais trocada — exportar o mapa para OpenGL e usá-lo na Unreal, ou o inverso —, que faz saliências parecerem reentrâncias e a iluminação afundar; é comum, mas felizmente fácil de reconhecer e corrigir. Logo atrás vem o **erro silencioso de espaço de cor** (Capítulo 20) na importação — marcar um mapa de dados como sRGB ou a cor base como linear —, que deturpa o material sem quebrá-lo e é difícil de diagnosticar para quem não conhece a causa.

Há o erro de **não usar instâncias**, construindo um material novo do zero para cada variação de uma superfície, desperdiçando trabalho, memória e tempo de compilação — quando um material base com parâmetros expostos resolveria tudo. Há o erro de produzir para o *pipeline* errado, sobretudo na Unity com seus múltiplos *pipelines* de renderização: um material montado para um *pipeline* aparece quebrado ou rosa-magenta em outro. Há o erro de ignorar as **convenções de projeto** — nomes de arquivos e canais fora do padrão, ordem de canais do ORM trocada —, que impede a importação automática e força retrabalho manual. E há o erro de fundo, contra o qual todo o capítulo se dirige: tratar a integração como questão de "saber usar o programa", decorando cliques de uma interface, em vez de entender os princípios — pois quem só conhece a interface fica perdido na próxima versão ou no próximo motor, enquanto quem domina os conceitos integra a qualquer um.

## Resumo

Este capítulo tratou da integração de um *asset* texturizado a um motor de jogo, não pela interface de um programa específico, mas pelos princípios que valem em qualquer um. Descreveram-se os três movimentos da integração — **importar** (com as decisões de compressão e espaço de cor do Capítulo 20), **construir o material** (ligando canais de textura às entradas do sombreador PBR da Parte III) e **instanciar na cena** (sob a iluminação do Capítulo 21) — e mostrou-se que os dois motores dominantes, Unreal e Unity, convergem no paradigma PBR, de modo que um *asset* bem preparado funciona previsivelmente em ambos. Apresentou-se a distinção entre **material base e instâncias** como a economia da reutilização aplicada aos materiais — um molde reaproveitável do qual brotam muitas variações baratas —, ecoando a filosofia dos *trim sheets* e dos atlas da Parte V. Detalharam-se os dois grandes pontos de atrito da integração: o **espaço de cor** (a regra de cor versus dados, com seus padrões distintos entre motores) e a **convenção do mapa de normais** (DirectX na Unreal, OpenGL na Unity, como regra de bolso), cujo erro produz o relevo invertido. E contrastaram-se as filosofias dos dois motores — a Unreal voltada à alta fidelidade e à edição por nós, a Unity à flexibilidade e à cobertura de plataformas com seus múltiplos *pipelines* —, mostrando que essas diferenças moldam decisões de produção, mas não alteram o princípio central: os fundamentos transcendem o motor, e quem prepara bem o *asset*, segundo os conceitos das partes anteriores, integra-o a qualquer ferramenta como quem traduz, não como quem reconstrói. O próximo capítulo examinará como se garante que esses materiais integrados estejam, de fato, corretos: o controle de qualidade.

## Exercícios

**1.** Descreva os três movimentos da integração de um *asset* a um motor (importar, construir o material, instanciar) e indique, para cada um, qual decisão ou cuidado das partes anteriores é mobilizado. Por que se diz que a integração é "o momento da verdade" da texturização?

**2.** Explique a distinção entre material base e instâncias e relacione-a à filosofia de reutilização da Parte V (*trim sheets*, atlas). Que recursos (trabalho de autoria, memória, custo de renderização) a instanciação economiza, e como?

**3.** A convenção do mapa de normais (OpenGL versus DirectX) é uma das maiores fontes de erro na integração. Explique o que difere entre as duas convenções, qual o sintoma visual de usá-las trocadas, e como a regra de bolso (Unreal/DirectX, Unity/OpenGL) orienta a correção. Por que esse erro é, ao mesmo tempo, comum e fácil de diagnosticar?

**4.** A apostila insiste que "os princípios transcendem o motor". Argumente a favor dessa tese listando ao menos quatro propriedades de um *asset* bem preparado que funcionam igualmente em Unreal, Unity e Godot, e explique por que decorar a interface de um motor é uma estratégia de aprendizado frágil.

**5.** Diagnóstico integrador: um *asset* importado na Unreal apresenta, simultaneamente, parafusos com relevo afundado, metal de aparência plástica e um aviso de que o material está designado para o *pipeline* de renderização errado. Para cada problema, identifique a causa entre os conceitos do capítulo (e do Capítulo 20) e descreva a correção, indicando o que mudaria se o motor de destino fosse a Unity.

## Glossário

**Convenção DirectX (mapa de normais):** Padrão de mapa de normais em que o canal verde codifica vetores apontando "para baixo" no espaço de textura. Adotado tradicionalmente pela Unreal Engine. Um mapa de normais exportado na convenção OpenGL e importado na Unreal produzirá relevo invertido.

**Convenção OpenGL (mapa de normais):** Padrão de mapa de normais em que o canal verde codifica vetores apontando "para cima" no espaço de textura. Adotado tradicionalmente pela Unity. Um mapa de normais exportado na convenção DirectX e importado na Unity produzirá relevo invertido.

**Instância de material:** Variante de um material base que reutiliza toda a lógica do sombreador do original, alterando apenas valores de parâmetros expostos. Permite produzir múltiplas variações visuais (cores, rugosidades, texturas) sem recompilar o sombreador, economizando trabalho e custo de renderização.

**Material base (material-pai):** Material com parâmetros expostos a partir do qual se derivam instâncias. Define a lógica de sombreamento que todas as instâncias compartilham. Análogo, no domínio dos materiais, ao *trim sheet* ou ao atlas no domínio das texturas.

**Mapeamento UV:** Sistema de coordenadas bidimensionais (U e V) que associa pontos da malha tridimensional a posições no espaço de textura, determinando como os canais de textura são projetados sobre a superfície.

**Motor de jogo:** Plataforma de software que integra sistemas de renderização, física, áudio, entrada e scripting para a produção de jogos interativos em tempo real. Para a texturização, o motor de jogo é o destino final dos *assets* e o contexto em que os materiais se revelam sob iluminação real.

**Pipeline de renderização:** Conjunto de estágios que o motor de jogo executa para transformar dados de cena (malhas, texturas, luzes) em pixels na tela. Diferentes pipelines de renderização (como os da Unity: Built-in, URP, HDRP) têm capacidades distintas e materiais incompatíveis entre si.

**Sombreador (*shader*):** Programa executado pela GPU para calcular a aparência de cada fragmento de superfície, combinando informações de textura, iluminação e parâmetros do material. No contexto PBR, recebe entradas padronizadas (cor base, mapa de rugosidade, mapa metálico, mapa de normais) e calcula a resposta física à luz.

## Leituras Complementares

- **[The PBR Guide]** (Wes McDermott; Allegorithmic / Adobe, 2018) — Fundamenta o paradigma PBR comum aos dois motores e a calibração dos mapas que torna um material previsível na integração. Essencial para compreender por que o PBR funciona de modo consistente em Unreal e Unity.

- **[Blender Normals]** (material de apoio da disciplina) — Trata da natureza vetorial do mapa de normais e de suas convenções de canal, base indispensável para compreender o problema OpenGL versus DirectX discutido neste capítulo.

- **[Unreal Engine Documentation]** — https://dev.epicgames.com/documentation/unreal-engine/ — Seções sobre importação de texturas e malhas, o editor de materiais por nós, instâncias de material, a convenção DirectX do mapa de normais e os *pipelines* de renderização. Referência prática obrigatória para a integração na Unreal.

- **[Unity Manual]** — https://docs.unity3d.com/ — Seções sobre importação de texturas e modelos, materiais e sombreadores, variantes de material, a convenção OpenGL do mapa de normais e os diferentes *pipelines* de renderização (desempenho versus fidelidade), com a abordagem correspondente na Unity.

- **[Godot Engine Documentation]** — https://docs.godotengine.org/ — Seções sobre materiais PBR (*StandardMaterial3D*) e importação de texturas em um motor de código aberto, úteis para confirmar que os princípios de integração transcendem qualquer motor específico.

- **[Adobe Substance 3D]** — https://helpx.adobe.com/substance-3d.html — Documentação sobre exportação de texturas com predefinições por motor (incluindo a convenção do canal verde do mapa de normais e o empacotamento ORM), que automatiza boa parte da preparação para a integração.
