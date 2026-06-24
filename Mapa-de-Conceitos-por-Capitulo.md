# Mapa de Conceitos por Capítulo

Este documento serve como guia de referência rápida para cada capítulo da apostila. Para cada um, são listados: um resumo em até dez tópicos, os pré-requisitos necessários para acompanhar o conteúdo, os conceitos centrais ensinados, e os conceitos que serão retomados e aprofundados em capítulos posteriores.

---

## Parte I — Fundamentos Conceituais

---

### Capítulo 1 — Materiais, Texturas e a Representação Visual em Computação Gráfica

**Resumo**

1. Aparência em computação gráfica é o resultado de um cálculo, não uma propriedade intrínseca do objeto.
2. Os três termos centrais da disciplina — material, textura e texturização — designam coisas distintas e são frequentemente confundidos.
3. A geometria descreve a forma por meio de malhas de vértices, arestas e faces, convertidas em triângulos pelo hardware.
4. O material é uma receita de como a superfície reage à luz; sozinho, não produz imagem.
5. A textura é, em geral, uma imagem bidimensional de texels usada para fornecer variação ponto a ponto às propriedades do material.
6. Texturas podem armazenar tanto dados visuais (cor) quanto dados não-visuais (rugosidade, direção de normais).
7. O mapeamento UV associa cada ponto (X, Y, Z) da superfície a um ponto (U, V) da textura.
8. Texel é a unidade mínima da textura, análogo ao pixel, mas com semântica dependente do canal armazenado.
9. A aparência final sempre resulta da combinação de material, textura e iluminação — nunca de apenas um desses elementos.
10. Compreender a distinção entre geometria, material e textura é o modelo mental que sustenta toda a disciplina.

**Pré-requisitos**

- Noção básica de modelagem 3D (saber o que é uma malha, vértice e face).
- Familiaridade com o conceito de motor de jogo como ambiente de renderização.

**Conceitos ensinados**

Material · Textura · Texturização · Mapeamento UV · Texel · Malha 3D (geometria) · Propriedade de superfície · Aparência como cálculo · Dados visuais e não-visuais em texturas

**Conceitos utilizados posteriormente**

- Material como receita → Caps. 3, 8, 9, 10
- Mapeamento UV → Parte II inteira (Caps. 4–7)
- Texel → Cap. 6 (Texel Density), Cap. 20 (mipmaps e compressão)
- Aparência como cálculo de iluminação → Caps. 3, 8, 11
- Dados não-visuais em texturas → Caps. 9, 15, 16, 20

---

### Capítulo 2 — Evolução Histórica da Texturização

**Resumo**

1. Os primeiros jogos usavam cores sólidas e sombreamento simples pela escassez de hardware.
2. As primeiras texturas eram pequenas e reutilizadas via tiling para economizar memória.
3. A iluminação embutida nas texturas (lightmapping pré-histórico) ofereceu realismo barato mas rígido.
4. A demanda por iluminação dinâmica forçou a separação entre cor própria da superfície e resposta à luz.
5. Os mapas de relevo (bump e normal maps) reintroduziram microdetalhe sem geometria adicional.
6. O baking surgiu como técnica de transferir detalhe de modelos densos para modelos leves.
7. A multiplicação descoordenada de técnicas gerou inconsistência entre artistas e motores.
8. O PBR (Physically Based Rendering) consolidou a indústria em torno de propriedades físicas mensuráveis.
9. Técnicas antigas não desaparecem: são absorvidas e combinadas conforme as restrições de cada projeto.
10. O profissional contemporâneo convive com camadas históricas de solução.

**Pré-requisitos**

- Cap. 1 (distinção entre material, textura e texturização).

**Conceitos ensinados**

Tiling · Lightmap histórico · Bump map · Normal map (introdução histórica) · Baking (introdução histórica) · PBR como consolidação · Evolução por restrição de hardware · Coexistência de técnicas de eras distintas

**Conceitos utilizados posteriormente**

- Normal map → Caps. 9, 16
- Baking (histórico) → Caps. 10, 15, 16
- PBR como paradigma → Caps. 3, 8, 9, 10, 11
- Tiling → Caps. 12, 18
- Lightmap → Cap. 21

---

### Capítulo 3 — Como Motores de Jogo Interpretam Materiais

**Resumo**

1. O motor é um intérprete de materiais, não apenas um visualizador de geometria.
2. O shader é o programa executado na GPU que calcula a cor de cada pixel a partir de geometria, texturas e luzes.
3. O modelo PBR descreve superfícies por propriedades com correspondência física: rugosidade, metalicidade, índice de refração.
4. Cada mapa de textura tem uma função definida dentro do cálculo do shader.
5. O fluxo metallic-roughness e o fluxo specular-glossiness são os dois padrões industriais de PBR.
6. Mapas de cor são interpretados em espaço sRGB; mapas de dados devem permanecer em espaço linear.
7. A GPU executa o shader em paralelo para milhares de pixels simultaneamente.
8. O PBR garante consistência entre artistas, cenas e ferramentas distintas.
9. A regra do espaço de cor é a distinção prática mais importante entre tipos de mapa.
10. Dados corretos produzem aparências corretas; dados incorretos, mesmo bem pintados, produzem ilusões inconsistentes.

**Pré-requisitos**

- Caps. 1 e 2 (modelo mental de material/textura e contexto histórico do PBR).

**Conceitos ensinados**

Shader · GPU e paralelismo · PBR como modelo de cálculo · Metallic-Roughness vs. Specular-Glossiness · Espaço de cor (sRGB vs. linear) · Rugosidade · Metalicidade · Mapa de normais (no contexto do motor) · Mapa de cor base

**Conceitos utilizados posteriormente**

- Shader → Cap. 11
- Espaço de cor sRGB vs. linear → Caps. 9, 20, 22, 23
- PBR (metallic-roughness) → Caps. 8, 9, 10, 11, 22
- Rugosidade e metalicidade → Caps. 9, 10
- Mapa de normais → Caps. 9, 16

---

## Parte II — Mapeamento UV e Preparação de Assets

---

### Capítulo 4 — Coordenadas UV e Projeções de Textura

**Resumo**

1. U e V são coordenadas normalizadas (0 a 1) que independem da resolução da textura.
2. Cada vértice carrega um par UV interpolado pelo shader para cobrir a superfície.
3. Abrir uma superfície fechada sobre um plano exige cortes chamados costuras (seams).
4. Os cortes geram ilhas UV organizadas num layout dentro do espaço UV.
5. Os modos de endereçamento (repetição, espelhado, fixado) governam o comportamento fora do intervalo 0–1.
6. A projeção planar mapeia uma direção sobre a superfície com distorção progressiva nos ângulos oblíquos.
7. A projeção cilíndrica serve a formas com simetria de revolução; a esférica, a formas globulares.
8. A projeção cúbica divide o objeto em seis faces e reduz a distorção em formas com ângulos retos.
9. Objetos reais quase sempre exigem a combinação de múltiplas projeções.
10. A textura de verificação (checker) torna visível a distorção que seria invisível a olho nu.

**Pré-requisitos**

- Caps. 1–3 (conceitos de material, textura, texturização e shader).
- Noção básica de geometria de malhas e coordenadas cartesianas.

**Conceitos ensinados**

Coordenadas UV (sistema normalizado) · Costura (seam) · Ilha UV · Layout UV · Modos de endereçamento (wrap, mirror, clamp) · Projeção planar · Projeção cilíndrica · Projeção esférica · Projeção cúbica · Textura de verificação (checker) · Distorção UV

**Conceitos utilizados posteriormente**

- Seams e ilhas UV → Caps. 5, 6, 7, 15, 16, 20
- Layout UV → Caps. 5, 6, 7, 18, 21
- Textura de verificação → Caps. 5, 6, 7
- Modos de endereçamento → Cap. 12 (repetição de texturas)
- Distorção UV → Cap. 5

---

### Capítulo 5 — UV Unwrapping

**Resumo**

1. Superfícies de curvatura dupla não são desenvolvíveis: não se achatam sem distorção.
2. O objetivo do unwrapping é equilibrar três tensões: minimizar distorção, esconder costuras e aproveitar o espaço UV.
3. Costuras devem ser posicionadas onde não serão vistas ou onde seguem as junções naturais da forma.
4. Cortar nas mudanças de direção da superfície reduz a distorção residual nas ilhas.
5. A simetria permite sobrepor ilhas espelhadas, dobrando a resolução efetiva ao custo de uma textura simétrica.
6. O fluxo de unwrapping é iterativo: analisar, marcar costuras, abrir, verificar, organizar.
7. O desdobramento automático otimiza geometria sem conhecer a intenção do artista.
8. O desdobramento automático é útil como rascunho, mas insuficiente como produto final para assets importantes.
9. A textura de verificação é o instrumento de verificação de distorção após o desdobramento.
10. Ilhas abertas e verificadas não são o produto final: ainda precisam ser dimensionadas e organizadas.

**Pré-requisitos**

- Cap. 4 (coordenadas UV, seams, ilhas UV, textura de verificação).

**Conceitos ensinados**

Superfície desenvolvível vs. não-desenvolvível · Princípios de posicionamento de costuras · Simetria e sobreposição de ilhas · Fluxo iterativo do unwrapping · Limites do desdobramento automático · Verificação de distorção

**Conceitos utilizados posteriormente**

- Simetria/sobreposição de ilhas → Cap. 6 (stacking), Cap. 18
- Fluxo iterativo → Cap. 7 (preparação de assets)
- Verificação de distorção → Cap. 6 (texel density)
- Qualidade do unwrapping → Caps. 15, 16 (baking depende do UV)

---

### Capítulo 6 — Texel Density e Organização de UVs

**Resumo**

1. Texel density é a medida de quantos texels recaem sobre cada unidade de superfície real.
2. Ela resulta da relação entre tamanho físico do objeto, área da ilha UV e resolução da textura.
3. Consistência de texel density entre objetos de uma mesma cena é requisito de qualidade profissional.
4. A indústria define valores-alvo de texel density por categoria de objeto.
5. A textura de verificação revela discrepâncias de densidade pelo tamanho desigual dos quadrados.
6. O empacotamento (packing) das ilhas maximiza a cobertura do espaço UV preservando margens (padding).
7. O padding é necessário para evitar sangramento de cor no mipmapping e na filtragem bilinear.
8. Sobreposição de ilhas simétricas e de peças repetidas é técnica de reaproveitamento de espaço.
9. Stacking e UDIMs são mecanismos para elevar a densidade efetiva além de um único tile.
10. A orientação e a legibilidade do layout servem à colaboração entre artistas da equipe.

**Pré-requisitos**

- Caps. 4 e 5 (coordenadas UV, ilhas, unwrapping, textura de verificação).

**Conceitos ensinados**

Texel density · Valores-alvo de densidade por categoria · Empacotamento (packing) · Padding UV · Sobreposição de ilhas · Stacking · UDIMs (introdução) · Consistência de densidade em cena

**Conceitos utilizados posteriormente**

- Texel density → Caps. 7, 10, 23
- Padding → Caps. 18, 20 (mipmaps e atlas)
- UDIMs → Cap. 19
- Stacking → Cap. 18
- Consistência de densidade → Cap. 23 (QA)

---

### Capítulo 7 — Preparação de Assets para Produção

**Resumo**

1. Higiene da malha inclui normais coerentes, grupos de suavização alinhados às costuras e ausência de n-gons e geometria degenerada.
2. Modelar em tamanho real é requisito da física, da iluminação e da texel density.
3. A origem, o pivô e as transformações aplicadas afetam o comportamento do objeto no motor.
4. Nomenclatura e organização consistentes são requisito inegociável do trabalho em equipe.
5. A relação entre malha high-poly e low-poly define a possibilidade de baking.
6. A preparação madura antecipa LODs, colisão e modularidade.
7. Trim sheets e hotspot texturing são expressões máximas do princípio de reutilização de espaço UV.
8. A exportação (FBX, OBJ, glTF) é a ponte entre o software de modelagem e o motor.
9. A verificação no motor de destino confirma que a preparação foi bem-sucedida.
10. Preparação bem feita é condição necessária para baking sem artefatos.

**Pré-requisitos**

- Caps. 4, 5, 6 (coordenadas UV, unwrapping, texel density).
- Conhecimento básico de modelagem 3D (grupos de suavização, n-gons, transformações).

**Conceitos ensinados**

Higiene de malha · Escala real e unidades de medida · Origem e pivô · Nomenclatura de produção · High-poly / Low-poly · LOD (introdução) · Trim sheet (introdução) · Hotspot texturing (introdução) · Formatos de exportação (FBX, OBJ, glTF) · Verificação no motor

**Conceitos utilizados posteriormente**

- High-poly / Low-poly → Caps. 15, 16 (baking)
- Trim sheet → Caps. 12, 18
- Hotspot texturing → Cap. 18
- Higiene de malha → Cap. 15 (qualidade do bake)
- Escala real → Cap. 21 (lightmaps), Cap. 22 (integração)

---

## Parte III — Physically Based Rendering e Materiais

---

### Capítulo 8 — Fundamentos do Physically Based Rendering

**Resumo**

1. O PBR nasceu para resolver a mistura entre descrição da matéria e da luz no paradigma tradicional.
2. A inversão fundamental do PBR: a textura descreve propriedades intrínsecas, e o motor calcula a interação com a luz.
3. A luz em superfícies se divide em reflexão especular (direcional) e reflexão difusa (cor de corpo).
4. A microsuperfície determina a nitidez dos reflexos; o PBR resume seu efeito no parâmetro de rugosidade.
5. Condutores (metais): sem reflexão difusa, reflexo colorido e de alta intensidade.
6. Dielétricos (não-metais): com reflexão difusa e reflexo fraco, acromático.
7. O parâmetro "metálico" captura diretamente a distinção entre condutores e dielétricos.
8. O PBR incorpora automaticamente conservação de energia, efeito de Fresnel e cálculo em espaço linear.
9. O maior ganho do PBR é a portabilidade: autora-se o material uma vez e ele é correto em qualquer iluminação.
10. PBR tornou-se a língua comum de ferramentas e motores, habilitando colaboração em escala.

**Pré-requisitos**

- Caps. 1–3 (conceitos de material, shader, PBR como paradigma, espaço de cor).
- Cap. 2 (contexto histórico do surgimento do PBR).

**Conceitos ensinados**

Reflexão especular vs. difusa · Microsuperfície e rugosidade · Condutores vs. dielétricos · Parâmetro metálico · Conservação de energia · Efeito de Fresnel · Cálculo em espaço linear · Portabilidade do PBR

**Conceitos utilizados posteriormente**

- Reflexão especular/difusa → Caps. 9, 10, 11
- Condutores vs. dielétricos → Cap. 9 (mapa metálico), Cap. 10 (construção de materiais)
- Rugosidade → Caps. 9, 10
- Espaço linear → Cap. 20 (compressão), Cap. 22 (integração)
- Fresnel → Cap. 11 (shader e BRDF)

---

### Capítulo 9 — Os Mapas que Compõem um Material PBR

**Resumo**

1. A distinção entre mapas de cor (sRGB) e mapas de dados (linear) se aplica a cada mapa do material.
2. A cor base armazena a cor intrínseca da superfície sem qualquer luz; seu significado muda entre metais e não-metais.
3. O mapa metálico é essencialmente binário e organiza o significado de todos os demais mapas.
4. O mapa de rugosidade é o mais expressivo: conta a história da microsuperfície e do desgaste.
5. O mapa de normais é o principal produto do baking: transfere detalhe da high-poly para a low-poly.
6. O mapa de oclusão de ambiente (AO) simula sombras de contato e profundidade em reentrâncias.
7. O mapa de altura pode fornecer deslocamento real ou ser usado como máscara de distribuição de efeitos.
8. O mapa emissivo define superfícies que emitem luz própria, independentemente da iluminação da cena.
9. O packing de canais reúne mapas de canal único em canais distintos de uma só textura para economizar memória.
10. O fluxo specular-glossiness é o caminho alternativo para descrever a mesma física; metallic-roughness é o padrão em jogos.

**Pré-requisitos**

- Caps. 3, 8 (shader, PBR, condutores/dielétricos, espaço de cor, rugosidade, metalicidade).

**Conceitos ensinados**

Cor base · Mapa metálico · Mapa de rugosidade · Mapa de normais (no contexto PBR) · AO (oclusão de ambiente) · Mapa de altura · Mapa emissivo · Mapa de opacidade · Subsurface Scattering (SSS) · Mapa de espessura (*thickness map*) · Packing de canais · Specular-Glossiness vs. Metallic-Roughness

**Conceitos utilizados posteriormente**

- Todos os mapas → Cap. 10 (construção de materiais reais), Cap. 11 (shader e aparência final)
- Mapa de normais → Cap. 16 (em profundidade)
- AO e mapa de altura como máscaras → Cap. 17
- Packing de canais → Cap. 20 (compressão e formato ORM)
- Distinção sRGB/linear por mapa → Caps. 20, 22, 23
- SSS e mapa de espessura → Cap. 22 (Subsurface Profile em Unreal/Unity), Cap. 25 (pipeline de personagens)

---

### Capítulo 10 — Construção e Análise de Materiais Reais

**Resumo**

1. A construção de materiais começa na observação: ler fisicamente a superfície de referência e decompô-la nos mapas.
2. A leitura de superfície é a habilidade mais subestimada e mais decisiva do texturizador.
3. O baking materializa a Parte II: transfere o detalhe da high-poly para a low-poly em mapas concretos.
4. A qualidade do bake depende diretamente da qualidade do unwrapping, da texel density e do alinhamento de malhas.
5. O pensamento por camadas constrói o material como acúmulo de histórias físicas (sujeira, desgaste, corrosão).
6. Cada camada afeta múltiplos mapas simultaneamente e é guiada por máscaras procedurais.
7. Abordagens procedural e manual são polos complementares a combinar conforme a natureza da superfície.
8. A calibração de valores contra referências da indústria é possível graças à base física do PBR.
9. A análise (decomposição de um material pronto) é a habilidade inversa à construção e serve ao diagnóstico.
10. Construir e analisar são duas faces de uma mesma competência.

**Pré-requisitos**

- Parte II completa (UVs, unwrapping, texel density, preparação de assets).
- Caps. 8, 9 (PBR, todos os mapas do material).

**Conceitos ensinados**

Leitura de superfície · Baking como etapa de construção · Pensamento por camadas · Máscaras procedurais de bake (AO, curvatura, ID) · Calibração de valores PBR · Abordagem procedural vs. manual · Análise e diagnóstico de materiais

**Conceitos utilizados posteriormente**

- Pensamento por camadas e máscaras → Cap. 17
- Baking → Caps. 15, 16
- Calibração → Cap. 23 (QA)
- Diagnóstico de materiais → Cap. 23
- Abordagem procedural → Cap. 13

---

### Capítulo 11 — Shaders, Iluminação e Aparência Final

**Resumo**

1. O shader é o programa que, para cada pixel, combina os mapas com a luz segundo o modelo PBR.
2. O shader consome os mapas como uma fórmula consome números: não corrige enganos.
3. A iluminação é a outra metade da equação de aparência; o material PBR não pode ser avaliado sem iluminação representativa.
4. A iluminação baseada em imagem (IBL) fornece a iluminação indireta dos reflexos em metais e superfícies lisas.
5. A BRDF é o coração matemático do shader: o modelo de como a superfície redistribui a luz.
6. A BRDF é o modelo comum que torna materiais transferíveis entre motores distintos.
7. Tone mapping e exposição convertem a cena de alto intervalo dinâmico para a tela.
8. Pós-processamento pertence à cena, não ao material; ajustar um pelo outro é erro de pipeline.
9. O desempenho é uma variável de design: o shader pode ser simplificado para plataformas limitadas.
10. A compressão e o espaço de cor definidos na importação são o ponto em que princípios se tornam escolhas concretas.

**Pré-requisitos**

- Caps. 3, 8, 9, 10 (shader, PBR, todos os mapas, construção de materiais).

**Conceitos ensinados**

Shader como executor do cálculo PBR · IBL (iluminação baseada em imagem) · BRDF · Tone mapping · Exposição · Pós-processamento · Distinção cena vs. material · Custo de shader e desempenho

**Conceitos utilizados posteriormente**

- IBL e iluminação → Caps. 21, 22
- BRDF como modelo comum → Cap. 22 (transferência entre motores)
- Tone mapping e pós-processamento → Cap. 24 (apresentação)
- Custo de shader → Cap. 25 (pipeline e orçamento)

---

## Parte IV — Workflows de Produção de Texturas

---

### Capítulo 12 — Texturas Seamless e Tileables

**Resumo**

1. Texturas tileables existem para multiplicar área a custo constante de memória.
2. Sua limitação essencial: o que se repete é genérico e não pode ser singular.
3. A condição seamless exige continuidade exata nas bordas e homogeneidade do interior.
4. O domínio toroidal é o modelo conceitual que explica a condição seamless.
5. Tiling visível é o principal defeito das texturas repetidas; combatê-lo é o desafio técnico central.
6. Estratégias anti-repetição: variação macro, múltiplas variantes, ladrilhamento estocástico, sobreposição de elementos singulares.
7. Texturas tileables fotográficas precisam ser corrigidas para se tornarem seamless.
8. Texturas procedurais nascem seamless por princípio.
9. Trim sheets organizam acabamentos em faixas reutilizáveis, unindo economia de produção e de memória.
10. Hotspot texturing encaixa geometrias em variantes de um atlas, dissolvendo a repetição por seleção variada.

**Pré-requisitos**

- Caps. 4–6 (UVs, modos de endereçamento/repetição, texel density).
- Cap. 7 (trim sheet e hotspot texturing como conceitos introdutórios).

**Conceitos ensinados**

Textura tileable · Condição seamless · Domínio toroidal · Tiling visível e estratégias anti-repetição · Ladrilhamento estocástico · Trim sheet (em profundidade) · Hotspot texturing (em profundidade)

**Conceitos utilizados posteriormente**

- Trim sheet → Caps. 18, 25
- Hotspot texturing → Cap. 18
- Seamless/tileable → Caps. 17 (base para decals), 20 (conflito com atlas)
- Ladrilhamento estocástico → Cap. 13 (procedural para variação)

---

### Capítulo 13 — Texturização Procedural

**Resumo**

1. Na texturização procedural, autora-se o processo que gera a textura, não a textura em si.
2. A edição não-destrutiva permite revisar qualquer etapa sem desfazer as demais.
3. A independência de resolução permite amostrar a mesma receita em qualquer densidade.
4. Ruídos (Perlin, celular, fractal) injetam o acaso plausível da variação natural.
5. Padrões regulares e formas elementares constroem as estruturas da matéria ordenada.
6. Combinação, transformação, distorção (warp), ajuste e mascaramento são as operações fundamentais.
7. O grafo de nós representa a textura como uma receita visual de operações.
8. Smart materials parametrizados geram famílias inteiras de variantes a partir de uma única receita.
9. O procedural domina o genérico e o repetível; o singular pertence à pintura manual.
10. O trabalho maduro combina procedural para a base e pintura para o particular.

**Pré-requisitos**

- Caps. 8, 9, 10 (PBR, mapas do material, pensamento por camadas).
- Cap. 12 (conceito de textura genérica vs. singular, trim sheets).

**Conceitos ensinados**

Procedural vs. pintura · Edição não-destrutiva · Independência de resolução · Ruído de Perlin · Ruído celular · Ruído fractal · Grafo de nós · Warp (distorção de textura) · Smart material

**Conceitos utilizados posteriormente**

- Grafo de nós e não-destrutividade → Cap. 14 (pintura em camadas), Cap. 17 (máscaras)
- Smart material → Cap. 25 (pipeline de produção)
- Procedural como base → Cap. 14, Cap. 17
- Ruídos como geradores de variação → Cap. 17 (máscaras procedurais)

---

### Capítulo 14 — Pintura Digital para Jogos

**Resumo**

1. A pintura digital é o paradigma do singular e do intencional, complemento do procedural.
2. A migração da pintura 2D (sobre UVs) para a pintura 3D (sobre o modelo) devolveu naturalidade ao processo.
3. A pintura 3D torna as costuras UV transparentes ao artista durante a autoria.
4. No contexto PBR, pintar significa aplicar materiais inteiros (cor + rugosidade + metálico) de uma vez.
5. Pincéis de material são a unidade de trabalho da pintura PBR.
6. A camada é a garantia da não-destrutividade na pintura.
7. A máscara é a ponte entre a intenção manual e a distribuição guiada pela geometria.
8. Princípios artísticos da pintura: ler a superfície, narrar o desgaste com intenção, economizar a atenção.
9. O fluxo híbrido padrão da indústria: base procedural → distribuição pela geometria → acabamento pela pintura manual.
10. Pintar e gerar são funções complementares; o texturizador competente transita entre as duas.

**Pré-requisitos**

- Caps. 8, 9 (PBR e mapas do material).
- Cap. 13 (texturização procedural como contexto complementar).
- Cap. 5 (noção de costuras UV que a pintura 3D dissolve).

**Conceitos ensinados**

Pintura 2D vs. 3D sobre o modelo · Pincel de material (PBR brush) · Camada (layer) · Máscara de pintura · Narrativa de desgaste · Fluxo híbrido procedural + pintura manual · Princípios artísticos de pintura de superfície

**Conceitos utilizados posteriormente**

- Camadas e máscaras → Cap. 17 (em profundidade)
- Fluxo híbrido → Caps. 25 (pipeline completo)
- Pintura sobre modelo 3D → Cap. 23 (verificação em contexto)

---

## Parte V — Otimização e Eficiência de Assets

---

### Capítulo 15 — Bake de Texturas

**Resumo**

1. Baking é a captura de uma propriedade calculada sobre uma malha e sua gravação em imagem.
2. Troca cálculo em tempo de execução por leitura de textura: o princípio mais difundido da otimização gráfica.
3. Baking de transferência entre malhas: o foco da Parte V, tarefa do texturizador.
4. Baking de iluminação em lightmaps: tarefa do artista de iluminação, antecipado conceitualmente.
5. A high-poly fornece o detalhe; a low-poly o recebe por meio de raios projetados.
6. A cage é uma cópia inflada da low-poly que governa o alcance dos raios.
7. Correspondência por nomenclatura (_low/_high) e exploded bake previnem o vazamento de detalhe entre partes.
8. Supersampling combate o serrilhado nas bordas dos detalhes assados.
9. O padding evita costuras nas bordas das ilhas UV no resultado do bake.
10. O baking colhe, sem corrigir, a qualidade das etapas anteriores (UV, texel density, higiene de malha).

**Pré-requisitos**

- Parte II completa (UVs, unwrapping, texel density, preparação — high/low poly).
- Cap. 9 (mapas que o bake gera: normais, AO, curvatura).
- Cap. 10 (baking como etapa de construção de materiais, introduzido conceitualmente).

**Conceitos ensinados**

Baking (definição e mecanismo) · Raio de projeção · Cage · Correspondência por nomenclatura (_low/_high) · Exploded bake · Supersampling · Padding no bake · Tipos de mapas gerados pelo bake

**Conceitos utilizados posteriormente**

- Baking → Cap. 16 (mapa de normais e mapas derivados)
- Cage e correspondência → Cap. 16 (qualidade do normal map)
- Padding → Cap. 20 (mipmaps), Cap. 18 (atlas)
- Baking de iluminação → Cap. 21

---

### Capítulo 16 — Normal Maps e Transferência de Detalhes

**Resumo**

1. O mapa de normais codifica a direção da normal em cada texel nos canais de cor da imagem.
2. Seu poder: dar orientação independente a cada texel, multiplicando o detalhe sem polígonos adicionais.
3. Object space: direções absolutas, preciso mas frágil à deformação.
4. Tangent space: direções relativas à superfície local, robusto ao movimento — padrão para quase tudo em jogos.
5. A convenção do canal verde difere entre OpenGL (Y+ para cima) e DirectX (Y+ para baixo), causando relevo invertido.
6. Mapas derivados (AO, curvatura, espessura, posição, ID) são assados na mesma passagem e usados como máscaras.
7. A combinação de um normal map base com um detail normal tileable adiciona microdetalhe a custo mínimo.
8. O mapa de normais altera o sombreamento, não a silhueta do objeto.
9. Parallax mapping e displacement mapping são evoluções com custos crescentes de desempenho.
10. O mapa de normais é o principal produto do baking e o mais importante dos mapas derivados.

**Pré-requisitos**

- Caps. 9, 10 (mapa de normais no contexto PBR, baking como etapa de construção).
- Cap. 15 (mecanismo do baking, cage, padding).

**Conceitos ensinados**

Normal map (codificação de vetores) · Object space vs. tangent space · Convenção OpenGL vs. DirectX (canal verde) · Mapas derivados (AO, curvatura, espessura, posição, ID) · Detail normal map · Parallax mapping · Displacement mapping · Limite do normal map (silhueta)

**Conceitos utilizados posteriormente**

- Convenção OpenGL vs. DirectX → Cap. 22 (Unity vs. Unreal)
- Mapas derivados como máscaras → Cap. 17 (retrospectivo), Cap. 23 (QA)
- Detail normal → Cap. 20 (combinação de texturas)
- Normal map no motor → Caps. 22, 23

---

### Capítulo 17 — Máscaras, Stencils e Decals

**Resumo**

1. A máscara é o controle universal do "onde": uma imagem em tons de cinza que decide a intensidade de cada efeito ponto a ponto.
2. Máscaras podem ser pintadas à mão (intenção), geradas proceduralmente (variação) ou derivadas da geometria (plausibilidade).
3. Máscaras derivadas da geometria (curvatura, AO, ID) automatizam a distribuição plausível do desgaste.
4. O baking fornece as máscaras de geometria: curvatura, oclusão de ambiente, ID de partes, espessura.
5. O stencil projeta formas figurativas específicas sobre o modelo, posição intermediária entre máscara e pintura.
6. O decal é uma camada singular sobreposta à base genérica, essencial para individualizar superfícies.
7. Decals podem ser estáticos (de produção) ou dinâmicos (gerados em tempo de jogo).
8. A superfície de um jogo bem produzido é uma composição: base tileable + procedural + pintura + decals.
9. A máscara é o tecido conjuntivo que une todos os métodos de produção.
10. Orquestrar esses instrumentos dentro do orçamento é a competência culminante da Parte IV.

**Pré-requisitos**

- Caps. 10, 13, 14 (pensamento por camadas, procedural, pintura e máscaras como conceito).
- Caps. 15/17 antecipados conceitualmente: máscaras de geometria vêm do baking.

**Conceitos ensinados**

Máscara (grayscale) · Máscara pintada vs. procedural vs. derivada da geometria · Curvatura como máscara · AO como máscara · Máscara de ID · Stencil · Decal estático · Decal dinâmico · Composição em camadas de superfície

**Conceitos utilizados posteriormente**

- Máscaras de geometria (curvatura, AO, ID) → Caps. 15, 16 (geradas pelo bake)
- Decal → Cap. 22 (decal system nos motores), Cap. 25 (pipeline)
- Composição em camadas → Cap. 25

---

### Capítulo 18 — Texture Atlas e Trim Sheets

**Resumo**

1. Draw calls são ordens de desenho que o processador envia para a GPU; multiplicam-se a cada troca de material.
2. Objetos que compartilham textura podem ser agrupados em batches, reduzindo draw calls.
3. O texture atlas reúne muitas imagens numa textura única, habilitando o batching de múltiplos objetos.
4. A trim sheet é um atlas especializado em faixas de acabamentos reutilizáveis para ambientes modulares.
5. O hotspot texturing encaixa geometrias de tamanhos variados em variantes de um atlas.
6. Texturas tileables não cabem em atlas: seu modo de endereçamento (repetição) é incompatível.
7. Reunir imagens num atlas divide a resolução disponível entre elas, afetando a texel density de cada uma.
8. O teto de resolução da plataforma limita o tamanho do atlas.
9. A perda de flexibilidade (não se pode alterar uma parte sem regenerar o atlas) é o custo do batching.
10. A economia de atlas é sobre o nível da cena; a economia de texel density é sobre o nível do asset.

**Pré-requisitos**

- Caps. 6 (texel density, padding), 12 (trim sheet, hotspot), 7 (hotspot texturing).
- Noção básica de renderização em tempo real (draw calls, GPU).

**Conceitos ensinados**

Draw call · Batching · Texture atlas · Economia de draw calls por compartilhamento de textura · Trim sheet (como atlas e como economia de desempenho) · Hotspot texturing (como atlas) · Conflito entre atlas e tileable · Teto de resolução de plataforma

**Conceitos utilizados posteriormente**

- Draw calls e batching → Cap. 20 (packing de canais reduz draw calls), Cap. 25 (pipeline)
- Atlas → Cap. 20 (mipmaps e padding em atlas)
- Trim sheet como ferramenta de pipeline → Cap. 25

---

### Capítulo 19 — UDIMs, Texture Arrays e Multi-Tile Texturing

**Resumo**

1. O problema oposto ao do atlas: quando uma textura não basta para uma superfície de alta resolução exigida.
2. A solução conceitual: dividir a superfície em partes com texturas próprias, multiplicando a densidade total.
3. O UDIM estende o espaço UV numa grade numerada de tiles, cada um com sua textura.
4. O UDIM habilita pintura contínua entre tiles e gestão automática de texturas durante a autoria.
5. O UDIM brilha no cinema; em jogos, cada tile é uma textura com custo de draw call.
6. Antes de entrar no motor, o UDIM deve ser consolidado em atlas ou formatos adequados ao tempo real.
7. O texture array empilha texturas uniformes numa ligação única, acessadas por índice sem custo de troca.
8. Texture arrays são ideais para mistura de materiais em terreno e variação de objetos repetidos.
9. Texture arrays exigem uniformidade de resolução e formato entre todas as texturas empilhadas.
10. UDIM (autoria) → atlas (consolidação) → texture array (renderização) é o encadeamento natural no pipeline.

**Pré-requisitos**

- Cap. 6 (UDIMs como introdução, texel density).
- Cap. 18 (atlas, draw calls, batching).

**Conceitos ensinados**

UDIM (sistema de tiles numerados) · Grade UDIM · Texture array · Multi-tile texturing como família de estratégias · Consolidação de UDIM para tempo real · Diagnóstico de problema de densidade vs. estratégia de solução

**Conceitos utilizados posteriormente**

- UDIM → Cap. 22 (suporte nos motores), Cap. 25 (pipeline)
- Texture array → Cap. 22 (terreno e variação de objetos)
- Consolidação UDIM → Cap. 20 (formato e compressão)

---

### Capítulo 20 — Compressão, Mipmaps e Packing de Canais

**Resumo**

1. A memória de vídeo é escassa; texturas mal geridas travam e borram o jogo.
2. A compressão de textura para GPU mantém a textura comprimida na memória e a descomprime em blocos na leitura.
3. Famílias de compressão: BC/DXT para PC e console, ASTC para dispositivos móveis.
4. A compressão é com perda; a regra que governa tudo é tratar cor como cor e dados como dados.
5. Cor base → sRGB, compressão de cor (BC1/BC3); mapas de dados → linear, formatos específicos.
6. Normais exigem formato especializado (BC5/ATI2) por preservar dois canais com precisão.
7. Mipmaps são versões progressivamente menores da textura, escolhidas pelo hardware conforme a distância.
8. Mipmaps resolvem o aliasing em objetos distantes e reduzem o custo da leitura de textura.
9. Mipmaps custam cerca de um terço de memória extra mas pagam em qualidade e desempenho.
10. Packing de canais (convenção ORM) reúne Oclusão, Rugosidade e Metálico numa textura só, reduzindo memória e draw calls.

**Pré-requisitos**

- Cap. 3 (espaço de cor sRGB vs. linear como regra prática).
- Caps. 9, 16 (tipos de mapa e seus requisitos de precisão).
- Caps. 6, 15 (padding nas ilhas UV e no bake).
- Cap. 18 (atlas e noção de draw calls).

**Conceitos ensinados**

Resolução de textura · Critérios de resolução por categoria de *asset* · Potência de dois · Compressão de textura para GPU · BC/DXT · ASTC · Compressão com perda · Regra cor vs. dados na compressão · Mipmaps · Aliasing de textura · Custo de memória dos mipmaps · Packing de canais · Convenção ORM

**Conceitos utilizados posteriormente**

- Resolução por categoria → Cap. 22 (configurações de importação), Cap. 23 (QA de texel density)
- Compressão e espaço de cor → Caps. 22, 23 (integração e QA)
- Mipmaps e padding → Cap. 21 (lightmaps têm mipmaps próprios)
- Convenção ORM → Caps. 22, 25 (pipeline)
- Regra cor vs. dados → Cap. 23 (critério objetivo de QA)

---

## Parte VI — Integração e Pipeline de Produção

---

### Capítulo 21 — Lightmaps e Iluminação em Motores

**Resumo**

1. Iluminação global (GI) simula o ricochete da luz indireta; calculá-la em tempo real é proibitivamente caro.
2. A estratégia análoga ao baking de texturas: pré-calcular a GI e armazená-la em lightmaps.
3. Regime estático: tudo pré-calculado, máxima qualidade indireta, imóvel.
4. Regime dinâmico: tudo em tempo real, reativo mas caro.
5. Regime misto: combinação que domina a produção contemporânea.
6. O lightmap complementa os mapas do material operando em camada distinta da textura de cor.
7. A geometria estática precisa de um segundo canal de UV dedicado ao lightmap.
8. UVs de lightmap têm regras opostas às do material: sem sobreposição, margens generosas, área proporcional à importância.
9. A resolução do lightmap é mais um recurso a orçar por importância visual.
10. Soluções de GI em tempo real (Lumen, DDGI) avançam para reduzir a dependência do bake de iluminação.

**Pré-requisitos**

- Caps. 4–6 (UVs, ilhas, texel density — para o segundo canal UV).
- Cap. 15 (baking como conceito — analogia direta).
- Caps. 11 (iluminação indireta, IBL).

**Conceitos ensinados**

Iluminação global (GI) · Lightmap · Regime estático / dinâmico / misto · Segundo canal de UV · Regras de UV para lightmap · Resolução de lightmap · GI em tempo real (Lumen, DDGI)

**Conceitos utilizados posteriormente**

- Lightmap → Cap. 22 (configuração nos motores), Cap. 25 (pipeline)
- Segundo canal de UV → Cap. 22 (Unreal e Unity)
- Regime de iluminação → Cap. 25 (decisões de pipeline)

---

### Capítulo 22 — Integração com Unreal Engine e Unity

**Resumo**

1. Integrar um asset ao motor envolve três movimentos: importar, construir o material e instanciar na cena.
2. As decisões de compressão e espaço de cor ocorrem na importação.
3. Construir o material significa ligar texturas às entradas do shader PBR do motor.
4. Instanciar na cena expõe o asset à iluminação real do projeto.
5. Unreal e Unity convergem no paradigma PBR; um asset bem preparado funciona previsivelmente em ambos.
6. Material base e instâncias: um molde reutilizável do qual brotam variações baratas de parâmetros.
7. Ponto de atrito 1 — espaço de cor: Unreal e Unity têm configurações padrão distintas.
8. Ponto de atrito 2 — convenção de normais: DirectX na Unreal (Y+ para baixo), OpenGL na Unity (Y+ para cima).
9. A Unreal prioriza alta fidelidade e edição por nós (Material Graph); a Unity, flexibilidade e cobertura de plataformas.
10. Os fundamentos transcendem o motor: quem prepara bem o asset, integra-o em qualquer ferramenta.

**Pré-requisitos**

- Parte II (UVs e preparação, incluindo segundo canal para lightmap).
- Caps. 9, 11 (mapas PBR, shader, IBL).
- Cap. 16 (convenção de normais OpenGL vs. DirectX).
- Cap. 20 (compressão e espaço de cor na importação).
- Cap. 21 (lightmaps e regime de iluminação).

**Conceitos ensinados**

Fluxo de integração (importar → material → cena) · Material base vs. instância · Atrito de espaço de cor entre motores · Convenção de normais por motor (Unreal/Unity) · Material Graph · Pipeline de renderização da Unity (URP, HDRP) · Decal system nos motores

**Conceitos utilizados posteriormente**

- Material base e instâncias → Cap. 25 (eficiência de pipeline)
- Pontos de atrito → Cap. 23 (QA: critérios de verificação)
- Filosofia Unreal vs. Unity → Cap. 25 (escolhas de pipeline)

---

### Capítulo 23 — Controle de Qualidade de Materiais

**Resumo**

1. Um material só se manifesta em contexto; o contexto de autoria não é o do jogo.
2. Defeitos invisíveis na autoria emergem no destino: luz real, distância variável, compressão, vizinhança.
3. Primeiro princípio: verificar no contexto de destino o quanto antes.
4. Cenas de validação padronizadas tornam a verificação sistemática e comparável entre assets.
5. Critério 1 — plausibilidade física: valores de rugosidade, metálico e cor dentro do intervalo PBR.
6. Critério 2 — espaço de cor: mapas de cor em sRGB, mapas de dados em linear.
7. Critério 3 — normais: sem relevo invertido, sem artefatos de costura.
8. Critério 4 — texel density: consistência entre assets da mesma cena.
9. Critério 5 — compressão: ausência de artefatos visíveis nas bordas de bloco.
10. Qualidade é processo, não inspeção final: erros verificados cedo são erros baratos de corrigir.

**Pré-requisitos**

- Partes I–V completas (todos os conceitos que se tornam critérios de QA).
- Cap. 22 (integração, o contexto de destino onde os erros emergem).

**Conceitos ensinados**

Verificação em contexto de destino · Cena de validação · Decomposição de material por canal · Critérios objetivos de QA (plausibilidade PBR, espaço de cor, normais, texel density, compressão, tiling) · Qualidade como processo contínuo vs. inspeção final

**Conceitos utilizados posteriormente**

- QA contínuo → Cap. 25 (verificação em cada etapa do pipeline)
- Cena de validação → Cap. 24 (cena de apresentação como análogo)
- Critérios de QA → Cap. 25 (pontos de verificação no pipeline completo)

---

### Capítulo 24 — Apresentação Profissional de Assets

**Resumo**

1. Apresentar é uma competência distinta de criar; exige aprendizado próprio.
2. QA examina o asset em condições adversas (para expor defeitos); apresentação, em condições favoráveis (para revelar virtudes).
3. Contexto de revisão interna: técnico e honesto, com estatísticas e decomposição de mapas.
4. Contexto de portfólio: prova competência mostrando o processo (wireframe, UVs, mapas, high/low).
5. Contexto de apresentação a cliente/público: encanta com o resultado visual.
6. A iluminação de apresentação (principal + preenchimento + contorno) transforma a percepção do mesmo trabalho.
7. Fundo neutro, bom HDRI e ângulo que pega o relevo são decisões básicas de mise en scène.
8. Imagens de processo (wireframe, UVs, mapas) convertem critérios técnicos em argumentos de portfólio.
9. Fronteira ética: realçar o melhor de um trabalho honesto (legítimo) vs. esconder defeitos ou atribuir-se trabalho alheio (ilegítimo).
10. Competência sem apresentação frequentemente não é vista; apresentação sem competência é autodestrutiva.

**Pré-requisitos**

- Cap. 23 (QA: só se apresenta com confiança o que se validou com rigor).
- Caps. 11, 21 (iluminação de cena — a ferramenta principal da apresentação).

**Conceitos ensinados**

Apresentação como competência · Contextos de apresentação (interno, portfólio, cliente) · Iluminação de apresentação (três pontos) · Imagens de processo para portfólio · Renderização técnica de apresentação (FOV, modo de renderização, anti-aliasing, render passes, formatos de exportação) · Fronteira ética da apresentação · HDRI para apresentação

**Conceitos utilizados posteriormente**

- Apresentação → Cap. 25 (etapa final do pipeline completo)

---

### Capítulo 25 — Pipeline Completo de Texturização para Jogos

**Resumo**

1. O pipeline tem uma forma: uma cadeia de dependências em que cada etapa produz o que a seguinte consome.
2. Fluxo canônico: planejamento → high/low poly → retopologia → UV → baking → texturização PBR → organização → integração → iluminação → QA → apresentação.
3. As decisões transversais governam todas as etapas: orçamento de plataforma, cor vs. dados, reutilização, distância de visualização.
4. O fio condutor de toda a apostila é o equilíbrio entre fidelidade e custo.
5. Erros precoces propagam e encarecem todas as etapas posteriores.
6. O controle de qualidade contínuo intercepta erros enquanto a correção ainda é barata.
7. O pipeline não é uma receita uniforme, mas um repertório de decisão adaptado a cada asset.
8. A competência síntese da disciplina é o julgamento: fazer as perguntas certas para cada asset.
9. Perguntas-chave: o que é o asset? onde aparece? sob que orçamento? com que materiais e métodos?
10. Dominar texturização para jogos é conduzir, com julgamento e rigor, um asset inteiro do conceito ao jogo.

**Pré-requisitos**

- A apostila completa (Caps. 1–24).

**Conceitos ensinados**

Pipeline como cadeia de dependências · Propagação de erros entre etapas · Decisões transversais de pipeline · Pipeline como repertório de decisão, não receita · Julgamento como competência síntese

**Conceitos utilizados posteriormente**

- Este é o capítulo de síntese. Não há capítulos subsequentes: os conceitos retornam para o estudante como instrumentos de prática profissional.

---

*Documento gerado como guia de estudo e referência curricular para a disciplina Texturização — Curso Superior de Tecnologia em Jogos Digitais.*
