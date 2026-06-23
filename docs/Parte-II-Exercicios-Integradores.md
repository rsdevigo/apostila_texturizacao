# Parte II — Exercícios Integradores

Os exercícios a seguir não se prendem a um único capítulo. Eles exigem que o estudante articule os conceitos de toda a Parte II — coordenadas e projeções (Capítulo 4), desdobramento (Capítulo 5), densidade de texels e organização (Capítulo 6) e preparação de *assets* para produção (Capítulo 7) — e, quando pertinente, os reconecte aos fundamentos da Parte I, sobretudo à ideia de que aparência é cálculo e de que cada mapa é uma entrada desse cálculo. Privilegiam a análise crítica, a comparação, o diagnóstico de problemas e o planejamento de pipelines, em detrimento de qualquer procedimento dependente de software. Não há, em vários casos, uma resposta única correta: espera-se que o estudante justifique suas decisões com fundamento conceitual.

---

## 1. Da forma ao layout: planejamento completo de um *asset*

Você recebeu o modelo de um lampião de rua antigo, composto por uma base larga apoiada no chão, um poste cilíndrico e alto, e uma luminária superior em forma de caixa de vidro com armação de metal. O *asset* aparecerá em primeiro plano num jogo de aventura e será visto de perto pelo jogador.

Elabore um plano de preparação completo, do mapeamento à entrega, justificando cada decisão. Seu plano deve, no mínimo: (a) indicar que projeção ou combinação de projeções você usaria para cada parte da forma e por quê; (b) descrever onde posicionaria as costuras e como as esconderia; (c) explicar como aproveitaria a simetria do objeto, se houver, e que limitação isso imporia; (d) dizer como garantiria que a densidade de texels do lampião fosse consistente com a dos demais *assets* de cenário próximos; e (e) listar os cuidados de preparação para produção (escala, pivô, normais, nomenclatura) que aplicaria antes de exportá-lo ao motor. Conclua avaliando quais decisões mudariam se o lampião fosse, em vez disso, um objeto de fundo distante, e por quê.

---

## 2. Diagnóstico em cadeia

Numa revisão de qualidade, três *assets* diferentes apresentam problemas. Para cada caso, identifique a causa mais provável, indique em qual etapa da preparação o erro foi cometido e proponha a correção, citando o conceito da Parte II (ou da Parte I) envolvido.

Caso A: uma espada exibe, ao longo da lâmina lisa e bem visível, uma linha onde a textura não casa de um lado para o outro.

Caso B: numa mesma sala, um armário aparece nitidamente borrado ao lado de um lampião perfeitamente nítido, embora ambos usem texturas de mesma resolução.

Caso C: quando o objeto se afasta da câmera e fica pequeno, surgem finas bordas coloridas espúrias ao redor de certas regiões da textura, que não apareciam de perto.

---

## 3. O triângulo de compromissos na prática

O Capítulo 5 apresentou os três objetivos em tensão do desdobramento — minimizar a distorção, esconder as costuras e aproveitar bem o espaço UV. Escolha dois tipos de *asset* bastante distintos quanto ao uso (por exemplo, o rosto de um personagem jogável visto de perto e uma rocha de cenário distante) e, para cada um, explique como você equilibraria os três objetivos de maneira diferente. Em que ordem de prioridade colocaria cada objetivo em cada caso, e que justificativa de produção sustenta essa escolha? Relacione sua resposta à ideia, recorrente em toda a Parte II, de gastar resolução e esforço onde eles rendem.

---

## 4. Comparação de estratégias de densidade

Um *asset* de portão de madeira, fisicamente grande, está com densidade de texels muito abaixo do padrão do projeto e aparece borrado. Compare três estratégias possíveis para corrigir o problema — (a) aumentar a resolução do mapa, (b) sobrepor as ilhas de tábuas repetidas para compartilhar textura e (c) recorrer a UDIMs — analisando, para cada uma, o efeito sobre a nitidez, o custo de memória, a adequação ao tempo real e eventuais limitações visuais (como a perda de variação individual). Em seguida, recomende uma estratégia justificando sua escolha, e explique como você verificaria, na prática, se a densidade resultante ficou consistente com a dos *assets* vizinhos.

---

## 5. Da preparação ao *baking*: antecipando a Parte III

O Capítulo 7 apresentou a relação entre a malha de alta densidade e a de baixa densidade e antecipou o processo de *baking*, que será o tema da Parte III. Imagine que você precisa preparar a malha de baixa densidade de um escudo metálico ornamentado, cujos detalhes em relevo (brasão, rebites, arranhões) existem apenas na malha de alta densidade e serão transferidos para um mapa de normais.

Explique, articulando os conceitos de toda a Parte II: (a) por que o desdobramento e a densidade de texels da malha de baixa densidade afetam diretamente a qualidade do detalhe transferido; (b) por que o *padding* entre ilhas e o alinhamento dos grupos de suavização com as costuras importam para esse processo; e (c) que problema apareceria no resultado final se a malha de baixa densidade chegasse ao *baking* com normais invertidas ou escala incorreta. Conclua relacionando a afirmação da Parte I — "aparência é cálculo, e cada mapa é uma entrada desse cálculo" — ao papel que a preparação cuidadosa desempenha para que esse cálculo produza o resultado pretendido.

---

## 6. Estudo de caso aberto: o cenário modular

Uma equipe precisa construir o interior de um castelo extenso — corredores, salões e torres — mantendo alta qualidade visual sob um orçamento de memória de textura severo, pois o jogo deve rodar também em plataformas modestas.

Discuta, em texto argumentativo, como você abordaria o problema integrando os conceitos da Parte II. Sua resposta deve contemplar: o uso de construção modular e a razão de ela ser adequada a esse caso; o papel das trim sheets e do hotspot texturing na economia de memória; como a reutilização de espaço UV (sobreposição, *stacking*) se articula com essas técnicas; como você manteria a densidade de texels consistente entre as muitas peças modulares; e que convenções de preparação para produção (nomenclatura, escala, organização) seriam indispensáveis para coordenar uma equipe trabalhando em paralelo nesse cenário. Encerre explicando por que o equilíbrio entre fidelidade visual e desempenho — fio condutor desde a Parte I — é o critério que deve guiar todas essas decisões.
