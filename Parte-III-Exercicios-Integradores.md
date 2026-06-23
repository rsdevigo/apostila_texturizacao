# Parte III — Exercícios Integradores

Os exercícios a seguir não se prendem a um único capítulo. Eles exigem que o estudante articule os conceitos de toda a Parte III — os fundamentos do PBR (Capítulo 8), os mapas que compõem um material (Capítulo 9), a construção e a análise de materiais reais (Capítulo 10) e o encontro do material com o shader e a iluminação (Capítulo 11) — e, sempre que pertinente, os reconecte às partes anteriores: à preparação de *assets* e ao mapeamento da Parte II e à ideia, fundadora desde a Parte I, de que aparência é cálculo e de que cada mapa é uma entrada desse cálculo. Como nas demais partes, privilegiam a análise crítica, a comparação, o diagnóstico de problemas e o planejamento de pipelines, em detrimento de qualquer procedimento dependente de software. Em vários casos não há uma única resposta correta: espera-se que o estudante justifique suas decisões com fundamento conceitual.

---

## 1. Da observação ao material: reconstrução completa de uma superfície

Escolha um objeto real, ao seu alcance, que combine pelo menos duas naturezas físicas distintas — por exemplo, uma chave de fenda (cabo plástico, haste metálica), um sapato gasto (couro, sola de borracha, ilhoses metálicos) ou uma garrafa térmica (metal escovado, tampa plástica, base emborrachada).

Elabore o plano completo de construção do seu material PBR, justificando cada decisão. Seu plano deve, no mínimo: (a) fazer a leitura física da superfície, decompondo-a nas propriedades que cada mapa representa e indicando onde e por que cada propriedade varia ao longo do objeto; (b) classificar cada região como condutora ou dielétrica e explicar a consequência dessa classificação para a cor base e o mapa metálico; (c) descrever o que conteria o mapa de rugosidade, contando a "história" da superfície — onde foi tocada, gasta, polida, suja; (d) indicar quais mapas o material exige e quais dispensaria, justificando; e (e) explicar que cuidados de preparação da Parte II (desdobramento, densidade de texels, *baking*) precederiam essa construção e por quê. Conclua dizendo como o material deveria ser verificado para se confirmar correto.

---

## 2. Diagnóstico em cadeia: o material que não convence

Numa revisão de qualidade, quatro materiais apresentam problemas. Para cada caso, identifique a causa mais provável, indique em qual etapa (autoria, importação, iluminação ou preparação) o erro ocorreu e proponha a correção, citando o conceito da Parte III — ou das partes anteriores — envolvido.

Caso A: um material que parecia perfeito numa iluminação de teste fica estranho em todas as outras, com brilhos e sombras que não acompanham a luz da cena.

Caso B: o mapa de normais de uma superfície apresenta detalhes que "vazam" de uma região para outra e uma costura visível atravessando uma face lisa.

Caso C: as cores de um material parecem lavadas e o brilho, exagerado e plástico; ao inspecionar a importação, nota-se que o mapa de rugosidade foi marcado como sRGB.

Caso D: um cavaleiro de aço polido aparece escuro e sem vida numa masmorra, embora o material estivesse impecável na visualização de autoria.

---

## 3. O ganho do PBR explicado a um cético

Imagine que um colega, acostumado ao paradigma tradicional de pintar luz e sombra diretamente na textura, afirma: "Não vejo vantagem no PBR; é só mais trabalho separar tudo em vários mapas, e eu consigo fazer minha textura parecer ótima do jeito antigo."

Escreva uma resposta argumentativa que o convença, articulando os conceitos do Capítulo 8. Sua resposta deve explicar: a inversão conceitual entre "pintar a luz" e "descrever a superfície"; por que a abordagem tradicional obriga a refazer o material a cada mudança de iluminação; como os princípios de conservação de energia e Fresnel poupam trabalho ao serem calculados pelo motor; e por que o "trabalho a mais" de separar em mapas resulta, no conjunto de um projeto, em muito menos trabalho. Conclua relacionando essa vantagem à ideia de material como *linguagem comum* transferível entre ferramentas e motores.

---

## 4. Procedural versus manual: decisões de autoria

O Capítulo 10 apresentou as abordagens procedural e manual como polos complementares da construção de materiais. Escolha três *assets* bastante distintos quanto ao uso — por exemplo, o revestimento de pedra das muralhas de um castelo extenso, o rosto de um personagem principal visto de perto e um único objeto narrativo com uma marca específica (uma carta queimada, um retrato rasgado).

Para cada um, decida como combinaria as abordagens procedural e manual e justifique, considerando: a necessidade de reutilização e de variação; o grau de controle artístico ponto a ponto exigido; a captura de irregularidade orgânica; e o custo de produção. Em seguida, explique como, em cada caso, os mapas derivados do *baking* (curvatura, oclusão de ambiente, ID) poderiam guiar máscaras de desgaste e sujeira para tornar o trabalho mais eficiente e mais fisicamente plausível. Relacione suas escolhas à tensão, recorrente em toda a apostila, entre o que é geral e reutilizável e o que é particular e único.

---

## 5. Do mapa ao pixel: rastreando o cálculo da aparência

Esta apostila afirma, desde a Parte I, que "aparência é cálculo, e cada mapa é uma entrada desse cálculo". Agora você dispõe de todos os elementos para detalhar essa afirmação.

Escolha um único ponto da superfície de um material metálico pintado e desgastado e descreva, em texto contínuo, todo o percurso que leva esse ponto a virar a cor de um pixel na tela. Seu texto deve percorrer: que valores os mapas (cor base, metálico, rugosidade, normais) fornecem naquele ponto e o que cada um significa fisicamente; como o espaço de cor de cada mapa é interpretado; como o shader, por meio da BRDF, combina esses valores com a direção da luz e a posição da câmera, incorporando conservação de energia, Fresnel e rugosidade; como o IBL contribui para o reflexo; e como tone mapping e exposição transformam o valor calculado na cor finalmente exibida. Conclua explicando por que um erro em qualquer etapa — uma cor base com luz pintada, um espaço de cor mal configurado, uma iluminação pobre — corrompe o resultado, e por que isso fundamenta todo o rigor exigido nas partes anteriores.

---

## 6. Estudo de caso aberto: o pipeline completo de um *asset* herói

Uma equipe precisa produzir, do zero, um *asset* "herói" para um jogo de ação-aventura: um elmo cerimonial ornamentado, em metal gravado com incrustações de esmalte colorido e couro no forro, que aparecerá em close numa cena cinematográfica e também durante o jogo, sob iluminações variadas, e que deve rodar tanto num console potente quanto numa plataforma modesta.

Discuta, em texto argumentativo, como você conduziria o pipeline completo desse *asset*, integrando os conceitos das três partes da apostila. Sua resposta deve contemplar, em sequência justificada: a preparação da geometria e do desdobramento (Parte II) e as condições que o *baking* exigirá; a estratégia de densidade de texels adequada a um *asset* de close; o processo de *baking* e os mapas que ele geraria; a construção do material por camadas, combinando procedural e manual e usando as máscaras guiadas pela geometria para distribuir desgaste, sujeira e oxidação de modo fisicamente plausível; a classificação correta de cada região (metal gravado, esmalte dielétrico, couro) e suas consequências para os mapas; a calibração contra valores de referência; e, por fim, a verificação no motor sob iluminação representativa, com IBL, e as adaptações de shader, resolução e compressão necessárias para conciliar a versão de console com a de plataforma modesta. Encerre explicando por que o equilíbrio entre fidelidade visual e desempenho — fio condutor desde a Parte I — é o critério que deve guiar todas essas decisões, e como cada etapa do pipeline depende do cuidado das etapas anteriores.
