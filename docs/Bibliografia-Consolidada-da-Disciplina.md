# Bibliografia Consolidada da Disciplina

Esta seção reúne, num único lugar, todas as referências mobilizadas ao longo das seis partes da apostila de Texturização. Ao longo da obra, cada parte trouxe sua própria bibliografia comentada, relacionando as fontes aos capítulos que delas se serviram; aqui, essas referências são consolidadas e organizadas por natureza, para servir de panorama geral da literatura da disciplina e de ponto de partida para quem deseje aprofundar qualquer tema. A bibliografia separa, como toda a apostila, o **material de referência do projeto** — os documentos que constituem a base oficial da disciplina — das **fontes complementares externas** incorporadas para atualizar, aprofundar e contextualizar os conceitos, conforme autorizam as diretrizes editoriais.

Convém reafirmar, ao consolidar, o princípio que regeu o uso das fontes em toda a obra. O material de referência do projeto serviu como base conceitual, não como fonte única nem como texto a resumir; sobre ele, os conceitos foram reconstruídos em linguagem própria e complementados, sempre que necessário, por fontes externas de autoridade reconhecida — a documentação oficial das ferramentas, dos motores e das APIs gráficas, os tratados de computação gráfica em tempo real e os recursos da prática profissional. Como vários dos temas da disciplina — formatos de compressão, *pipelines* de motores, convenções de iluminação e de apresentação — evoluem rapidamente, o estudante é encorajado a consultar sempre as versões mais atuais da documentação técnica e a formar, pela comparação de fontes, uma compreensão própria e crítica.

---

## 1. Material de referência do projeto

Documentos técnicos que constituem a bibliografia base da disciplina, com indicação dos principais temas que sustentam.

- *The PBR Guide* (Wes McDermott; Allegorithmic / Adobe), versão 2018. Fundamento do paradigma PBR (Parte III): a descrição física dos materiais, a distinção entre dados de cor (sRGB) e dados lineares, a calibração de cor base, rugosidade e metalicidade. Sustenta, em cadeia, a compressão e o *packing* (Parte V), a integração aos motores, o controle de qualidade e a iluminação de apresentação (Parte VI).
- *What Is Texture Baking?* Introduz o conceito de *baking* como transferência de detalhe da malha rica para a leve e a economia de memória que o justifica (Parte V, Cap. 15).
- *Baking Guide*. Aprofunda o processo do *baking* — projeção, *cage*, mapas gerados, cuidados contra falhas, nomenclatura, *exploded bake*, *padding* (Parte V, Caps. 15 e 16).
- *Blender Normals*. Trata da natureza vetorial e das convenções (*tangent*/*object*, OpenGL/DirectX) dos mapas de normais (Partes V e VI, Caps. 16 e 22).
- *Blender Maps*. Situa o conjunto de mapas de um material (normais, oclusão, curvatura, altura) e seu uso num fluxo concreto (Partes III, V e VI).
- *AOD — Texel Density* (com os mapas de verificação *TD_Checker*). Fundamenta a relação entre densidade de texels, tamanho de textura e proximidade de visualização (Parte II, Cap. 6) e oferece um instrumento concreto de controle de qualidade (Parte VI, Cap. 23).
- OLSEN, Morten. *The Ultimate Trim — Texturing Techniques of Sunset Overdrive* (Insomniac Games). Documenta a técnica das *trim sheets* e a filosofia de reutilização extrema (Partes IV e V, Caps. 12 e 18), retomada na arquitetura de materiais base e instâncias (Parte VI, Cap. 22).
- *Hotspot Texturing* (Default Interactive — https://www.defaultinteractive.co.uk/post/hotspot-texturing). Apresenta o mapeamento por *hotspot* sobre texturas de *trim* compartilhadas (Partes IV e V, Caps. 12 e 18).
- *The Book of Shaders* (Patricio Gonzalez Vivo; Jen Lowe — https://thebookofshaders.com/?lan=pt). Introdução didática à programação de *shaders* e ao pensamento procedural, base para a compreensão de como a aparência é cálculo (Partes I, III e IV, Caps. 11 e 13).
- *Blender Secrets* (Jan van den Hemel). Compêndio de técnicas práticas de Blender, referência de apoio para modelagem, UVs e materiais ao longo da disciplina.
- *Projeto Pedagógico do Curso Superior de Tecnologia em Jogos Digitais* (IFMS — Campus Dourados, 2025 — https://www.ifms.edu.br/). Documento institucional que situa a disciplina de Texturização no currículo do curso.

---

## 2. Fontes complementares externas

### 2.1 Documentação oficial de ferramentas e motores

- *Adobe Substance 3D* — https://helpx.adobe.com/substance-3d.html. Autoria de materiais PBR, *baking* de *mesh maps*, fluxo procedural (Designer) e em camadas (Painter), exportação com presets por motor, validação PBR e renderização de apresentação. Referência transversal das Partes III a VI.
- *Blender Manual* — https://docs.blender.org/manual/en/latest/index.html. Modelagem, desdobramento UV, *baking*, materiais por nós, iluminação, HDRI e renderização em ferramenta de código aberto, ao longo de toda a disciplina.
- *Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/unreal-engine-5-8-documentation. Materiais por nós e instâncias, convenção DirectX de normais, *render pipelines*, compressão e mipmaps, UDIM e *texture arrays*, mobilidade de luz, *Lightmass* e Lumen, modos de depuração e renderização de apresentação. Referência prática das Partes V e VI.
- *Unity Manual* — https://docs.unity3d.com/ (versões 6000.4 e 6000.5). Materiais e *Material Variants*, convenção OpenGL de normais, os diferentes *render pipelines*, compressão por plataforma, *Texture2DArray*, modos de iluminação (*Baked*/*Mixed*/*Realtime*) e *Light Probes*, e modos de depuração. Referência prática correspondente em outro motor.
- *3DCoat Documentation* — https://3dcoat.com/documentation/. *Baking* e pintura PBR diretamente sobre o modelo, para comparar abordagens de geração de mapas (Parte V, Caps. 15 e 16).
- *Godot Engine Documentation* — https://docs.godotengine.org/en/stable/. Materiais PBR (*StandardMaterial3D*), importação e compressão de texturas, *texture arrays*, iluminação e *lightmapping* em motor de código aberto, para confirmar que os princípios transcendem a ferramenta.

### 2.2 APIs gráficas e fundamentos de hardware

- *DirectX 12 Programming Guide* — https://learn.microsoft.com/en-us/windows/win32/direct3d12/directx-12-programming-guide. Formatos de textura comprimidos (família BC), arranjos de textura e amostragem, no nível do *hardware* (Parte V, Caps. 19 e 20).
- *OpenGL* — https://www.opengl.org/ — e *LearnOpenGL* — https://learnopengl.com/. Formatos de textura, *mipmapping*, filtragem e arranjos de textura, fundamentando os mecanismos de baixo nível (Parte V) e a programação gráfica que sustenta o conceito de *shader* (Parte I).

### 2.3 Recursos da prática profissional

- *ArtStation* — https://www.artstation.com/. Plataforma de referência para a apresentação de *assets* de jogos: padrões de portfólio, pranchas com *render* final e imagens de processo, estatísticas técnicas (Parte VI, Cap. 24).

### 2.4 Livros e fundamentos teóricos

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Referência abrangente de gráficos em tempo real: *texturing*, *normal/bump mapping*, *parallax* e *displacement*, *batching* e custo de estado, arranjos de textura e arquitetura de GPU, compressão, filtragem, *mipmapping* e *antialiasing*, iluminação global e *precomputed lighting*, sombreamento físico. Apoio teórico de todas as partes.
- PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023. Disponível em https://www.pbr-book.org/. Base teórica do PBR (Parte III), da amostragem e filtragem de texturas (Parte V) e do transporte de luz que os lightmaps pré-calculam (Parte VI).

---

## 3. Nota final sobre o uso das fontes

Conforme as diretrizes editoriais da disciplina, esta bibliografia consolidada não é uma lista de leituras obrigatórias a percorrer linearmente, mas um mapa do território da texturização para jogos, do qual o estudante seleciona conforme seu interesse e sua necessidade. As fontes do projeto fornecem a base; as externas atualizam, aprofundam e diversificam. Nenhuma é fonte única, e a competência que a disciplina busca formar — o julgamento que conduz um *asset* do conceito ao jogo — não nasce da memorização de qualquer texto, mas da compreensão crítica que se constrói confrontando fontes, praticando e refletindo. Que esta bibliografia sirva, portanto, menos como ponto de chegada e mais como ponto de partida para o estudo continuado de uma área que não para de evoluir.
