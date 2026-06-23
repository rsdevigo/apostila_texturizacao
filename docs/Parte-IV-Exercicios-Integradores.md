# Parte IV — Exercícios Integradores

Os exercícios a seguir não se prendem a um único capítulo. Eles exigem que o estudante articule os métodos de produção de textura — as texturas seamless e tileables e suas evoluções em *trim sheets* e *hotspot texturing* (Capítulo 12), a texturização procedural (Capítulo 13) e a pintura digital para jogos (Capítulo 14), da Parte IV — e os instrumentos que controlam onde o detalhe aparece, as máscaras, stencils e decais (Capítulo 17), já na Parte V — e, sempre que pertinente, os reconectem às partes anteriores: à ideia de que aparência é cálculo e cada mapa é uma entrada (Parte I), ao desdobramento, à densidade de texels e ao *baking* (Parte II) e ao paradigma PBR, aos mapas de um material e ao encontro com o shader e a luz (Parte III). Como nas demais partes, privilegiam a análise crítica, a comparação, o diagnóstico de problemas e o planejamento de pipelines, em detrimento de qualquer procedimento dependente de software. Em vários casos não há uma única resposta correta: espera-se que o estudante justifique suas decisões com fundamento conceitual.

---

## 1. O espectro do genérico ao singular: classificar os métodos

Toda a Parte IV pode ser lida como um espectro que vai do genérico e reutilizável ao singular e intencional. Disponha, em uma linha, os principais métodos e instrumentos estudados — textura tileable, *trim sheet*, *hotspot texturing*, material procedural parametrizado, máscara derivada da geometria, máscara procedural, stencil, pintura manual à mão livre, decal — ordenando-os do mais genérico ao mais singular, e justifique a posição de cada um.

Em seguida, escolha três pontos distintos desse espectro e, para cada um, descreva um tipo de conteúdo de uma superfície real de jogo para o qual aquele método é a escolha ideal, e um para o qual seria a pior escolha. Conclua explicando por que essa tensão entre o genérico e o singular, e não a superioridade de um método sobre os outros, é o verdadeiro fio condutor desta parte da apostila.

---

## 2. Diagnóstico em cadeia: superfícies que não convencem

Numa revisão de qualidade, quatro superfícies apresentam problemas. Para cada caso, identifique a causa mais provável, indique a que método ou conceito da Parte IV (ou das partes anteriores) o erro se relaciona e proponha a correção.

Caso A: uma parede de tijolos tileable encaixa perfeitamente nas bordas, mas, vista de longe e repetida, exibe uma mancha escura que reaparece em grade regular por toda a superfície.

Caso B: uma chapa metálica tileable parece impecável na cor, mas, sob luz rasante, mostra uma linha de relevo reta repetindo-se a cada ladrilho.

Caso C: um conjunto de cem barris foi texturizado proceduralmente com o mesmo material, e todos parecem idênticos, produzindo um cenário de "clones", apesar de o material ser fisicamente correto.

Caso D: um muro de concreto foi "individualizado" com decais, mas a mesma rachadura aparece, idêntica e na mesma posição, em todos os muros, parecendo tão artificial quanto a base repetida.

---

## 3. Procedural ou pintura: a decisão de autoria

Os Capítulos 13 e 14 apresentaram o procedural e a pintura digital como paradigmas complementares — o primeiro forte no genérico e repetível, o segundo no singular e intencional. Escolha três *assets* bastante distintos quanto à natureza: por exemplo, o revestimento de pedra das muralhas de um castelo extenso, o rosto envelhecido de um personagem principal visto em close, e uma única caixa narrativa com um logotipo específico e marcas de uma queda determinada.

Para cada um, decida como distribuiria o trabalho entre o procedural, a distribuição guiada pela geometria e a pintura manual, justificando à luz da necessidade de reutilização e variação, do grau de controle ponto a ponto exigido e do custo de produção. Explique, em cada caso, como os mapas derivados do *baking* (curvatura, oclusão, *ID*) guiariam máscaras para tornar o trabalho mais eficiente e fisicamente plausível, e onde a pintura à mão ainda seria indispensável. Relacione suas escolhas à economia da atenção: onde você concentraria o esforço, e por quê.

---

## 4. O argumento da camada: explicar o pipeline a um iniciante

Imagine que um colega iniciante afirma: "Texturizar é só fazer uma imagem bonita e colar no modelo. Não entendo por que falam em tantos métodos e camadas — parece complicação desnecessária."

Escreva uma resposta argumentativa que o convença, articulando os conceitos de toda a Parte IV. Sua resposta deve explicar: por que uma única imagem não resolve o problema da economia de memória em grandes superfícies (Cap. 12); por que pensar em camadas e em "onde cada coisa aparece" (máscaras) torna o trabalho flexível e revisável (Caps. 13, 14, 17); como a separação entre a base genérica e o detalhe singular sobreposto (decais) resolve simultaneamente a economia e a credibilidade; e por que a orquestração de vários métodos, longe de ser complicação, é o que torna possível produzir mundos vastos, coerentes e críveis dentro de um orçamento. Conclua relacionando esse "pensamento de camadas" ao equilíbrio entre fidelidade e custo que percorre a apostila inteira.

---

## 5. Da observação à composição de camadas

Escolha uma superfície real, ao seu alcance, que seja simultaneamente *repetitiva e singular* — por exemplo, um muro de tijolos com pichações e manchas específicas, um piso de madeira gasto de modo desigual, ou a lateral de um contêiner enferrujado com numerações e adesivos.

Descreva, em texto contínuo, como você reconstruiria essa superfície para um jogo decompondo-a em camadas, do genérico ao singular. Seu texto deve percorrer: que base tileable ou *trim sheet* capturaria o padrão que de fato se repete (Cap. 12); que variação procedural acrescentaria e como a tornaria seamless (Cap. 13); quais máscaras derivadas da geometria distribuiriam o desgaste de modo plausível e quais você pintaria à mão (Caps. 14 e 17); que stencils aplicariam o conteúdo figurativo (numerações, símbolos); e quais decais depositariam o detalhe singular que quebra a repetição (Cap. 17). Conclua explicando como essa composição em camadas entrega, ao mesmo tempo, economia (a base reutilizável) e individualidade (as camadas singulares sobrepostas).

---

## 6. Estudo de caso aberto: produzir um nível inteiro

Uma equipe precisa texturizar um nível completo de um jogo de ação-aventura: uma cidade portuária parcialmente destruída — ruas de pedra, fachadas de prédios em pedra e madeira, um cais de metal e concreto, navios atracados, e, em primeiro plano, a casa-base do protagonista, que o jogador examinará de perto. O nível deve rodar tanto num console potente quanto numa plataforma modesta, e a destruição precisa contar a história de um ataque recente.

Discuta, em texto argumentativo, como você conduziria a texturização completa desse nível, integrando os conceitos das quatro partes da apostila. Sua resposta deve contemplar, em sequência justificada: a estratégia de cobertura das grandes superfícies com texturas tileables, *trim sheets* e *kits modulares*, e como você impediria a repetição visível (Cap. 12); o uso de materiais procedurais parametrizados para produzir variação coerente em escala e a independência de resolução para atender às duas plataformas (Cap. 13); a distribuição de desgaste, sujeira e dano por máscaras derivadas da geometria, respeitando a forma de cada *asset* (Caps. 14 e 17); a pintura manual e os stencils reservados aos pontos de atenção — sobretudo a casa-base de primeiro plano — e ao conteúdo figurativo; o uso de decais, estáticos e em tempo de jogo, para narrar o ataque (marcas de impacto, fogo, escombros) e individualizar superfícies sem custo de memória proporcional; e a verificação no motor, sob iluminação representativa, com a calibração PBR e as adaptações de resolução e compressão necessárias entre as duas plataformas (Parte III). Encerre explicando por que a competência culminante do texturizador não é o domínio de um método isolado, mas a *orquestração* de todos eles — distribuindo o esforço e o orçamento onde rendem mais qualidade percebida —, e como cada decisão desta parte depende do cuidado das etapas das partes anteriores: a geometria preparada e desdobrada (Parte II), os materiais fisicamente corretos (Parte III) e a ideia fundadora de que aparência é cálculo (Parte I).
