# Parte V — Exercícios Integradores

Os exercícios a seguir não se prendem a um único capítulo. Eles exigem que o estudante articule os conceitos de toda a Parte V — o *baking* como transferência de detalhe da malha rica para a leve (Capítulo 15), o mapa de normais e os demais mapas de transferência (Capítulo 16), as máscaras, *stencils* e *decals* que usam os mapas de malha (Capítulo 17), a organização de texturas em atlas e *trim sheets* para economizar *draw calls* (Capítulo 18), os UDIMs, *texture arrays* e o multi-tile para multiplicar densidade (Capítulo 19) e a compressão, os mipmaps e o packing de canais que adequam a textura ao *hardware* (Capítulo 20) — e, sempre que pertinente, os reconectem às partes anteriores: à ideia de que aparência é cálculo e cada mapa é uma entrada (Parte I), ao desdobramento, à densidade de texels e às UVs (Parte II), ao paradigma PBR e à distinção entre cor e dados (Parte III) e aos métodos de produção da Parte IV. Como nas demais partes, privilegiam a análise crítica, a comparação, o diagnóstico de problemas e o planejamento de pipelines, em detrimento de qualquer procedimento dependente de software. Em vários casos não há uma única resposta correta: espera-se que o estudante justifique suas decisões com fundamento conceitual.

---

## 1. A economia da eficiência: classificar as técnicas pelo recurso que poupam

Toda a Parte V pode ser lida como um conjunto de respostas à escassez de três recursos: **polígonos**, **memória de textura** e **draw calls**. Construa uma tabela em texto contínuo relacionando cada técnica principal estudada — *baking*/mapa de normais, atlas, *trim sheet*, *hotspot*, UDIM, *texture array*, compressão de GPU, mipmaps, packing de canais — ao recurso (ou recursos) que ela primordialmente economiza, explicando o mecanismo da economia em cada caso.

Em seguida, identifique pelo menos duas técnicas que economizam um recurso à custa de **aumentar** outro (por exemplo, gastar memória para ganhar em outra dimensão), e explique por que, ainda assim, a troca compensa. Conclua argumentando por que a eficiência em jogos nunca é a otimização de um único recurso isolado, mas o equilíbrio simultâneo dos três sob o orçamento da plataforma.

---

## 2. Diagnóstico em cadeia: assets que falham na produção

Numa revisão técnica, cinco *assets* apresentam problemas. Para cada caso, identifique a causa mais provável, indique a que conceito da Parte V (ou das partes anteriores) o erro se relaciona e proponha a correção.

Caso A: ao assar uma arma com mira e coronha próximas, a superfície da coronha exibe manchas com o desenho da mira, e uma saliência do cano ficou lisa, sem detalhe.

Caso B: ao integrar um personagem no motor, todos os rebites da armadura parecem furos afundados, e a iluminação parece "achatada".

Caso C: a pele de um personagem parece "plástica" e um metal próximo não reflete como deveria, embora os mapas pareçam corretos na ferramenta de autoria.

Caso D: um cenário modular roda bem em PC mas trava no celular, com o contador de renderização indicando milhares de chamadas por quadro.

Caso E: um piso ladrilhado que se estende até o horizonte "ferve" e cintila quando a câmera se move, sobretudo nas faixas distantes.

---

## 3. Cor ou dado: a decisão que atravessa o pipeline

A distinção entre uma textura de **cor** (interpretada pelo olho) e uma textura de **dados** (interpretada por um cálculo) reaparece, na Parte V, como regra prática em vários pontos. Escolha três momentos do *pipeline* em que essa distinção determina uma decisão concreta — por exemplo, na compressão, na marcação de espaço de cor na importação, no packing de canais — e, para cada um, explique o que muda conforme a textura seja de cor ou de dado, e qual o erro que resulta de tratar uma como a outra.

Em seguida, relacione essa regra ao mapa de normais especificamente (Capítulo 16): explique por que ele é o exemplo mais sensível dessa distinção, listando os tratamentos de "imagem comum" que o corromperiam e o porquê de cada um. Conclua explicando por que o capítulo chama o erro de cor-versus-dados de "silencioso" e por que isso o torna especialmente perigoso.

---

## 4. Planejamento de pipeline: do high-poly ao asset na plataforma

Você é responsável por levar um *asset* importante — um portão ornamentado de ferro forjado, com brasão em relevo, rebites e dobradiças, que aparecerá tanto em close (quando o jogador o abre) quanto à distância (visto do outro lado do pátio) — desde a escultura até sua forma final eficiente em um motor de jogo que rodará em PC e celular.

Descreva, em texto contínuo, o *pipeline* completo, nomeando e justificando cada decisão à luz da Parte V (e reconectando às anteriores quando pertinente): a relação *high-poly*/*low-poly* e a retopologia; o desdobramento e a densidade de texels (com atenção a close versus distância); o *baking* e os cuidados contra vazamento (*cage*, nomenclatura, *exploded bake*, *padding*); quais mapas assaria e como os usaria (de normais para o relevo, curvatura e oclusão como máscaras); se e como usaria *detail normals*; como organizaria as texturas (precisa este *asset* de atlas, *trim sheet*, UDIM ou *texture array*? por quê?); e como o prepararia para o *hardware* (compressão por plataforma, mipmaps, packing de canais, marcação de cor/dados). Justifique cada escolha pelo equilíbrio entre fidelidade e custo e pelos orçamentos de PC e celular.

---

## 5. A integração das cinco partes: uma superfície, do conceito ao hardware

Este exercício pede a síntese de toda a apostila no eixo da produção. Escolha uma superfície complexa de jogo — por exemplo, o casco corroído de um navio encalhado, examinável de perto e visto também de longe — e descreva, em um texto contínuo e bem articulado, como você a produziria do início ao fim, mobilizando deliberadamente conceitos das cinco partes.

Seu texto deve, no mínimo: explicar como a aparência da superfície é, em última instância, cálculo do shader sobre mapas (Parte I); descrever o desdobramento e a organização das UVs com sua densidade de texels (Parte II); especificar os mapas PBR que comporiam o material e por que seus valores precisam ser fisicamente calibrados (Parte III); indicar quais métodos de produção usaria para gerar o conteúdo — tileable, procedural, pintura, máscaras, decais — e por quê (Parte IV); e, enfim, detalhar a transformação em *asset* eficiente segundo toda a Parte V — *baking*, mapas de transferência, organização espacial das texturas, compressão, mipmaps e packing. Conclua com uma reflexão própria sobre o que é, afinal, o fio condutor de toda a apostila — o equilíbrio entre fidelidade visual e custo computacional —, ilustrando-o com pelo menos três decisões concretas do seu projeto em que esse equilíbrio foi determinante.

---

## 6. Cinema e jogo: por que o mesmo asset é tratado de modos diferentes

A Parte V tocou, em mais de um ponto, na diferença entre o fluxo de produção de cinema (sem restrição de tempo real) e o de jogo (com orçamento de desempenho), de modo especialmente nítido no caso dos UDIMs (Capítulo 19). Escreva uma análise comparando como um mesmo *asset* de altíssima qualidade — digamos, um personagem protagonista — seria tratado nos dois contextos, percorrendo: a contagem de polígonos e o papel do *baking*; a densidade de texels e o número de texturas; o uso de UDIMs e sua eventual consolidação; e as exigências de compressão e de *draw calls*.

Conclua explicando por que muitas das técnicas desta parte (atlas, packing, compressão agressiva, consolidação de UDIMs) são, em essência, **respostas à restrição do tempo real** que o cinema não precisa enfrentar — e por que, ainda assim, o estudante de jogos precisa entender o fluxo de cinema para dialogar com *assets* e ferramentas que dele provêm.
