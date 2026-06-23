# Glossário da Disciplina

Este glossário reúne os termos técnicos empregados ao longo das seis partes da apostila, com definições concisas formuladas no contexto da texturização para jogos. Não substitui as explicações dos capítulos, onde cada conceito é desenvolvido em profundidade e em suas relações; serve como referência rápida e como instrumento de revisão. Sempre que pertinente, a definição indica entre parênteses o capítulo em que o termo é tratado com mais detalhe. Os termos estão organizados alfabeticamente.

**Albedo** — Ver *Cor base*.

**Aliasing** — Ruído ou cintilação que surge quando uma textura é amostrada de forma irregular, sobretudo a distância; a textura "ferve" quando o objeto ou a câmera se move. Combatido pelos *mipmaps* (Cap. 20).

**Asset** — Qualquer elemento de conteúdo de um jogo (modelo, textura, material, som); nesta apostila, sobretudo o objeto 3D texturizado e seus mapas.

**ASTC** — Família de formatos de compressão de textura para GPU dominante em plataformas móveis, flexível em qualidade e taxa (Cap. 20).

**Atlas (de textura)** — Reunião de várias imagens ou materiais distintos numa única textura, para reduzir o número de texturas e de *draw calls* (Cap. 18).

**Baking** — Processo de transferir, por projeção, o detalhe de uma malha rica (*high-poly*) para mapas de textura aplicados sobre uma malha leve (*low-poly*); gera mapas de normais, oclusão, curvatura, *ID*, entre outros (Cap. 16). Por extensão, *baking* de iluminação é o pré-cálculo da luz armazenado em *lightmaps* (Cap. 21).

**BC (Block Compression)** — Família de formatos de compressão de textura para GPU (também chamada DXT/DXTn) dominante em PC e consoles, com variantes especializadas por tipo de mapa, incluindo uma própria para normais (Cap. 20).

**Bitmap** — Ver *Textura raster*.

**Cage** — Malha auxiliar, ligeiramente maior que a *low-poly*, que define até onde a projeção do *baking* captura o detalhe da *high-poly*, evitando falhas de projeção (Cap. 16).

**Canal** — Cada componente de uma imagem (tipicamente vermelho, verde, azul e alfa — RGBA). Mapas de dados de valor único usam um só canal, o que permite o *packing de canais* (Cap. 20).

**Cor base** *(base color/albedo)* — Mapa PBR que descreve a cor própria da superfície, sem informação de luz ou sombra; tratado como dado de cor (sRGB). Não deve conter iluminação "pintada" (Caps. 9 e 23).

**Cor versus dados** — Distinção fundamental entre uma textura interpretada pelo olho (cor, em espaço sRGB) e uma interpretada por um cálculo (dado, em espaço linear); governa a produção, a marcação na importação, a compressão e o *packing* de cada textura (Caps. 8, 20 e 22).

**Curvatura (mapa de)** — Mapa assado que registra as arestas convexas e côncavas da superfície; usado como máscara para desgaste e sujeira (Caps. 16 e 15).

**Decal** — Detalhe de superfície aplicado localmente sobre um material de base, como um adesivo (sinais, rachaduras, manchas), sem ocupar espaço no mapeamento principal (Cap. 15).

**Densidade de texels (texel density)** — Quantidade de texels (pixels de textura) por unidade de superfície do modelo; mede a resolução efetiva da textura sobre o objeto e deve ser consistente e adequada à proximidade de visualização (Cap. 6).

**Detail normal** — Mapa de normais de detalhe fino, repetido sobre a superfície e somado ao mapa de normais principal para acrescentar microrrelevo (poros, fibras) sem custo de resolução (Cap. 17).

**Draw call** — Instrução de desenho enviada à GPU; cada mudança de material ou textura tende a gerar uma, e seu número é um recurso escasso a economizar (atlas, *trim sheets*, *batching*) (Cap. 18).

**DXT / DXTn** — Ver *BC*.

**Exploded bake** — Técnica de *baking* em que partes próximas do modelo são afastadas umas das outras para evitar que o detalhe de uma vaze sobre a outra na projeção (Cap. 16).

**Global illumination (iluminação global)** — Simulação da luz indireta, que ricocheteia entre superfícies tingindo-se de suas cores; cara em tempo real, é frequentemente pré-calculada em *lightmaps* (Cap. 21).

**Hotspot (texturing)** — Técnica de mapear faces a regiões pré-definidas de uma textura de *trim* compartilhada, encaixando cada superfície na faixa adequada (Caps. 12 e 18).

**High-poly** — Malha de alta contagem de polígonos, rica em detalhe esculpido, usada como origem do *baking* (Cap. 16).

**ID (mapa de)** — Mapa que atribui cores distintas a diferentes partes ou materiais do modelo, servindo de máscara para aplicar materiais por região (Caps. 16 e 15).

**Instância (de material)** — Variante de um material base que reaproveita sua lógica de *shader* alterando apenas parâmetros expostos (cor, rugosidade), economizando trabalho e custo de renderização (Cap. 22).

**Lightmap** — Textura que armazena a luz pré-calculada incidente sobre a geometria estática de uma cena, incluindo a luz indireta e as sombras; complementa, sem substituir, os mapas do material e exige um segundo canal de UV (Cap. 21).

**Linear (espaço)** — Espaço de cor sem a curva de correção perceptual; usado para texturas de dados (normais, rugosidade, metalicidade, oclusão, mapas empacotados), cujos valores alimentam cálculos (Caps. 8, 20 e 22).

**Low-poly** — Malha de baixa contagem de polígonos que vai ao jogo, recebendo, via *baking*, o detalhe da *high-poly* em seus mapas (Cap. 16).

**Máscara** — Imagem em escala de cinza que controla onde um efeito, material ou detalhe se aplica sobre a superfície; base do fluxo procedural e em camadas (Cap. 15).

**Material** — Descrição completa da aparência de uma superfície, reunindo os mapas e os parâmetros que o *shader* interpreta; no motor, objeto editável que liga texturas às entradas do *shader* (Caps. 1, 11 e 22).

**Metalicidade (metallic)** — Mapa PBR que classifica a superfície entre metal e não metal, idealmente em valores próximos de 0 ou 1; determina como a luz é refletida (Cap. 9).

**Mipmap** — Cadeia de versões progressivamente menores de uma textura, pré-calculadas e escolhidas pelo hardware conforme a distância, para leitura barata e sem *aliasing* a distância, ao custo de cerca de um terço de memória extra (Cap. 20).

**Normais (mapa de)** — Mapa de dados que codifica, em cor, a direção da superfície em cada ponto, simulando relevo sob a iluminação sem geometria adicional; sensível à convenção (OpenGL/DirectX) e ao espaço de cor (Caps. 9, 17 e 22).

**Oclusão de ambiente (ambient occlusion)** — Mapa que registra o quanto cada ponto da superfície é encoberto da luz ambiente pela própria geometria, escurecendo reentrâncias; usado na composição e como máscara (Caps. 9 e 16).

**ORM / RMA** — Mapa empacotado que reúne oclusão, rugosidade e metalicidade em canais distintos de uma única textura (a sigla indica a ordem dos canais); convenção comum de *packing* esperada por muitos motores, tratada como dado linear (Cap. 20).

**Packing de canais** — Técnica de armazenar mapas de dados de valor único em canais distintos (RGBA) de uma só textura, reduzindo memória e número de texturas (Cap. 20).

**Padding** — Margem de pixels expandida para fora das ilhas de UV no *baking* e nas texturas, para que a filtragem e os *mipmaps* não puxem o vazio ou imagens vizinhas para dentro da superfície (Caps. 16 e 18).

**PBR (Physically Based Rendering)** — Paradigma de renderização que descreve os materiais por propriedades físicas (cor base, rugosidade, metalicidade) calibradas segundo o comportamento real da luz, garantindo aparência plausível sob qualquer iluminação (Cap. 8).

**Pipeline** — Sequência encadeada de etapas que leva um *asset* do conceito à sua forma final no jogo, em que cada etapa entrega o que a seguinte consome; mais do que uma receita, um repertório de decisão adaptável a cada *asset* (Cap. 25).

**Procedural (texturização)** — Geração de texturas e materiais por regras, ruídos e operações combinadas, em vez de pintura manual direta, resultando em conteúdo não destrutivo e ajustável (Cap. 13).

**Retopologia** — Reconstrução de uma malha limpa e leve (*low-poly*) a partir de uma escultura densa (*high-poly*), preparando-a para o desdobramento e o *baking* (Caps. 7 e 16).

**Rugosidade (roughness)** — Mapa PBR que descreve quanto a superfície dispersa a luz refletida, do espelhado (liso) ao fosco (rugoso); um dos mapas mais determinantes da aparência de um material (Cap. 9).

**Seamless / Tileable** — Textura cujas bordas se encaixam ao se repetir, permitindo cobrir grandes superfícies sem costuras visíveis (Cap. 12).

**Shader** — Programa executado na GPU que calcula a cor final de cada ponto da superfície a partir dos mapas do material e da iluminação; é ele que torna a aparência um cálculo (Cap. 11).

**sRGB** — Espaço de cor com curva de correção ajustada à percepção humana; usado para texturas de cor (como a cor base), nunca para texturas de dados (Caps. 8 e 20).

**Stencil** — Máscara ou padrão usado para aplicar um detalhe sobre a superfície de forma controlada, como um estêncil físico (Cap. 15).

**Texel** — Pixel de uma textura ("texture element"); a unidade da densidade de texels (Cap. 6).

**Textura** — Imagem aplicada sobre uma superfície 3D para fornecer informação visual ou de dados ao material (Cap. 1).

**Texture array** — Estrutura que reúne múltiplas texturas de mesmo tamanho num único objeto indexável, usada para multiplicar a densidade de detalhe (por exemplo, em terrenos) (Cap. 19).

**Trim sheet** — Textura organizada em faixas de molduras, bordas e materiais reutilizáveis, mapeadas sobre muitas superfícies diferentes para máxima reutilização (Caps. 12 e 18).

**UDIM** — Sistema de organização que distribui as UVs de um *asset* por múltiplos blocos (tiles) de textura, cada um com sua própria imagem, para obter alta densidade em *assets* complexos (Cap. 19).

**UV (coordenadas)** — Sistema de coordenadas bidimensionais que define como uma textura se assenta sobre a superfície 3D; o desdobramento (*unwrapping*) produz o mapeamento, e a geometria estática com *lightmap* exige um segundo canal de UV (Caps. 4, 5 e 21).

**Unwrapping (desdobramento)** — Processo de "abrir" a superfície 3D num plano para criar suas coordenadas UV, com costuras escondidas e ilhas bem organizadas (Cap. 5).

**Wireframe** — Representação da malha pelas suas arestas; usada nas imagens de processo do portfólio para evidenciar a topologia e a contagem de polígonos (Caps. 24).
