# Exercícios Finais Integradores da Disciplina

Os exercícios a seguir encerram a apostila e a disciplina. Diferentemente dos exercícios integradores de cada parte, que articulavam os capítulos de uma mesma seção, estes exigem o domínio **do conjunto** — a mobilização deliberada de conceitos das seis partes, da natureza dos materiais (Parte I) à condução do *pipeline* completo (Parte VI). São, por isso, mais amplos e mais abertos: na maioria, não há resposta única, e o que se avalia é a capacidade de **planejar, decidir e justificar** com fundamento conceitual, integrando o que foi aprendido num raciocínio coerente. Recomenda-se que sejam tratados como pequenos projetos de análise, em texto contínuo e bem articulado, no estilo de um livro universitário — e não como respostas curtas. Como ao longo de toda a obra, privilegiam a análise crítica, a comparação, o diagnóstico e o planejamento, em detrimento de qualquer procedimento dependente de software.

---

## 1. A apostila em uma tese: o fio condutor

Esta apostila afirmou, repetidas vezes, ter um fio condutor único — o **equilíbrio entre fidelidade visual e custo computacional** — e uma distinção recorrente que o atravessa — a diferença entre **cor e dados**. Escreva um ensaio que percorra as seis partes da disciplina demonstrando como esse fio condutor se manifesta em cada uma: na representação dos materiais e na interpretação dos motores (Parte I), no mapeamento e na densidade de texels (Parte II), no PBR e seus mapas (Parte III), nos métodos de produção (Parte IV), nas técnicas de eficiência (Parte V) e na integração e no *pipeline* (Parte VI).

Para cada parte, identifique ao menos uma decisão concreta em que o equilíbrio entre fidelidade e custo foi determinante, e explique como foi resolvido. Conclua argumentando se você concorda que esse é, de fato, o fio condutor da disciplina — ou se proporia outro —, justificando sua posição. O objetivo é demonstrar uma compreensão da disciplina como um todo articulado, e não como uma soma de tópicos.

---

## 2. Diagnóstico integral: o asset que falha em muitas frentes

Numa revisão técnica final, um único *asset* — uma porta de masmorra em madeira reforçada com ferro, vista em *close* e à distância, destinada a PC e a celular — apresenta, simultaneamente, os seguintes sintomas. Para cada um, identifique a causa mais provável, indique a parte e o capítulo da apostila a que o erro se relaciona, e proponha a correção. Em seguida, ordene as correções segundo a lógica do *pipeline* (Cap. 25), explicando por que algumas precisam ser feitas antes de outras.

(a) As tábuas da madeira aparecem borradas no *close*, enquanto as ferragens estão nítidas.

(b) Os rebites de ferro parecem furos afundados sob a iluminação.

(c) O ferro tem aparência "plástica" e não reflete como metal deveria.

(d) Vista de longe, a textura da madeira "ferve" e cintila quando a câmera se move.

(e) No celular, a cena que contém muitas dessas portas trava, com o contador de renderização indicando milhares de chamadas por quadro.

(f) Sob a luz dinâmica da masmorra, a porta parece ter sombras "duplicadas", como se já trouxesse sombras pintadas.

---

## 3. O mesmo asset, dois mundos: cinema e jogo

Escolha um *asset* de altíssima qualidade — por exemplo, uma armadura ornamentada de um personagem protagonista — e descreva, em análise comparativa, como ele seria produzido e tratado em dois contextos: um filme de animação (sem restrição de tempo real) e um jogo (com orçamento de desempenho).

Percorra deliberadamente as etapas do *pipeline* (Cap. 25) e mostre, em cada uma, o que muda entre os dois contextos: a contagem de polígonos e o papel do *baking* (Cap. 16); a densidade de texels e o número de texturas, incluindo o uso e a eventual consolidação de UDIMs (Caps. 6 e 19); a estrutura de materiais e o uso de instâncias (Cap. 22); a compressão, os mipmaps e o *packing* (Cap. 20); e a iluminação — global em tempo real ou pré-calculada em lightmaps (Cap. 21). Conclua explicando por que tantas técnicas da disciplina são, em essência, **respostas à restrição do tempo real** que o cinema não enfrenta — e por que, ainda assim, o profissional de jogos precisa entender o fluxo de cinema.

---

## 4. Planejamento de pipeline sob medida: três assets, três caminhos

O Capítulo 25 sustentou que o *pipeline* é um **repertório de decisão, não uma receita**, e que nem todo *asset* percorre todas as etapas. Para demonstrá-lo, planeje o *pipeline* de três *assets* muito diferentes e contraste os três percursos:

(a) uma **textura tileable** de calçamento de pedra para cobrir grandes áreas de chão;

(b) um **personagem protagonista** examinável de perto em cutscenes;

(c) um **prop modular** (uma viga de madeira) que se repetirá centenas de vezes para montar estruturas.

Para cada um, descreva quais etapas do *pipeline* canônico se aplicam e quais se dispensam (e por quê), que métodos de produção da Parte IV são adequados, que estratégia de organização de texturas (tileable, *trim sheet*, atlas, UDIM) faz sentido e que cuidados de eficiência são prioritários. Conclua comparando os três caminhos e explicando o que essa comparação revela sobre a tese de que o *pipeline* se adapta a cada *asset*.

---

## 5. Cor ou dado: a decisão que percorre a disciplina inteira

A distinção entre uma textura de **cor** (interpretada pelo olho, em sRGB) e uma de **dados** (interpretada por um cálculo, em linear) reapareceu da Parte III à Parte VI como uma das ideias mais consequentes da disciplina. Escreva uma análise que rastreie essa distinção ao longo do *pipeline*, identificando ao menos quatro momentos distintos em que ela determina uma decisão concreta — por exemplo, na produção do mapa, na sua compressão, na sua marcação de espaço de cor na importação, no seu *packing* e na verificação do controle de qualidade.

Para cada momento, explique o que muda conforme a textura seja de cor ou de dado e qual o erro que resulta de tratar uma como a outra. Detenha-se no mapa de normais como o caso mais sensível (Caps. 17 e 22), listando todos os tratamentos de "imagem comum" que o corromperiam. Conclua explicando por que vários desses erros são chamados de **silenciosos**, por que isso os torna especialmente perigosos e como o controle de qualidade (Cap. 23) os torna audíveis.

---

## 6. Da superfície ao portfólio: o percurso completo de um asset

Este é o exercício de síntese máxima da disciplina. Escolha uma superfície complexa de jogo — por exemplo, o casco de um navio naufragado, de madeira apodrecida com chapas de metal corroído e cracas, examinável de perto e visto também de longe, num jogo para PC e console — e descreva, em um texto contínuo e bem estruturado, **todo o seu percurso**, do conceito à apresentação no portfólio, mobilizando conscientemente conceitos das seis partes.

Seu texto deve, no mínimo: explicar como a aparência da superfície é, em última instância, cálculo do *shader* sobre mapas, e como o motor a interpreta (Parte I); descrever o planejamento, a relação *high/low poly*, a retopologia, o desdobramento e a densidade de texels (Parte II); especificar os mapas PBR que comporiam o material e por que seus valores precisam ser fisicamente calibrados (Parte III); indicar quais métodos de produção usaria para cada elemento — tileable, procedural, pintura, máscaras, decais — e por quê (Parte IV); detalhar a transformação em *asset* eficiente — *baking*, mapas de transferência, organização espacial, compressão, mipmaps, *packing* (Parte V); e descrever a integração ao motor, o ajuste sob a iluminação da cena, o controle de qualidade e a apresentação final no portfólio (Parte VI).

Conclua com uma reflexão pessoal: o que significou, ao longo da disciplina, deixar de ver a texturização como um conjunto de técnicas isoladas e passar a vê-la como um **processo integrado de decisão**? Ilustre essa reflexão com ao menos três decisões concretas do seu projeto em que o equilíbrio entre fidelidade e custo foi determinante, e com ao menos uma decisão em que a distinção entre cor e dados foi decisiva. O objetivo deste exercício é que você demonstre, num único trabalho, o domínio integrado de tudo o que a apostila construiu — a competência que define o texturizador profissional de jogos digitais.
