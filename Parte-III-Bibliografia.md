# Parte III — Bibliografia e Materiais Complementares

Esta seção reúne, de forma consolidada, as referências que sustentam os quatro capítulos da Parte III — fundamentos do *Physically Based Rendering* (Capítulo 8), os mapas que compõem um material PBR (Capítulo 9), construção e análise de materiais reais (Capítulo 10) e shaders, iluminação e aparência final (Capítulo 11). Como nas partes anteriores, ela separa o material de referência do projeto — a bibliografia oficial da disciplina — das fontes complementares externas incorporadas para atualizar, aprofundar e contextualizar os conceitos. As fontes complementares foram selecionadas por sua autoridade reconhecida na área (os guias de referência da própria indústria sobre PBR, os tratados de computação gráfica em tempo real, a documentação oficial das ferramentas de autoria e dos motores e os recursos didáticos abertos consolidados) e não substituem, mas ampliam, o material da disciplina.

As referências estão organizadas por tema, de modo a orientar o estudante sobre onde buscar aprofundamento conforme o assunto de interesse. Ao final, indica-se uma trilha de leitura sugerida.

---

## 1. Material de referência do projeto

Conjunto de documentos técnicos que constituem a bibliografia base da disciplina e que sustentam, de modo direto, os temas da Parte III.

- *The PBR Guide* (Wes McDermott; Allegorithmic / Adobe), versão 2018 (material de apoio da disciplina). Referência central de toda a Parte III. Expõe os fundamentos físicos do PBR — comportamento da luz nas superfícies, reflexão difusa e especular, microsuperfície e rugosidade, conservação de energia, Fresnel, condutores e dielétricos —, descreve em detalhe os dois fluxos de trabalho (metálico/rugosidade e especular/suavidade) e cada um de seus mapas, e fornece as faixas de valores de referência para calibração de cor base, metais e reflectância. Base direta dos Capítulos 8, 9, 10 e 11.
- *What Is Texture Baking?* e *Baking Guide* (materiais de apoio da disciplina). Introduzem e detalham o processo de *baking* — a transferência do detalhe geométrico da malha de alta densidade para os mapas aplicados à de baixa densidade —, incluindo o mapa de normais, a oclusão de ambiente e os mapas derivados, além dos artefatos típicos e suas causas. Base da seção sobre *baking* do Capítulo 10 e do mapa de normais no contexto PBR do Capítulo 9.
- *Blender Maps* (material de apoio da disciplina). Situa os mapas de um material dentro de um fluxo de trabalho concreto, conectando-os ao desdobramento da Parte II; apoio aos Capítulos 9 e 10.
- *Blender Normals* (material de apoio da disciplina). Aborda o trabalho com normais, grupos de suavização e sua relação com o mapa de normais e o *baking*, central para a qualidade dos mapas dos Capítulos 9 e 10.
- *AOD — Texel Density* (material de apoio da disciplina). Fundamenta a relação, retomada na calibração do Capítulo 10 e no desempenho do Capítulo 11, entre a densidade de texels e a manifestação efetiva dos valores autorados em um material.

---

## 2. Fontes complementares externas

### 2.1 Livros e guias de referência

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Referência central de gráficos em tempo real. Os capítulos sobre física da luz, modelos de sombreamento (*shading*) e BRDFs fundamentam os Capítulos 8 e 11; os capítulos sobre *texturing* e mapeamento, os Capítulos 9 e 10; e os relativos a iluminação baseada em imagem, HDR, *tone mapping* e otimização, o Capítulo 11.
- PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023. Disponível gratuitamente em https://www.pbr-book.org/. Tratado avançado sobre a teoria por trás da renderização baseada em física, para quem desejar aprofundar a base óptica dos princípios do Capítulo 8 e o modelo formal da BRDF do Capítulo 11.

### 2.2 Documentação oficial de ferramentas e motores

- *Adobe Substance 3D* — https://helpx.adobe.com/substance-3d.html. Referência sobre a autoria integrada de conjuntos de mapas, a pintura por camadas, os *smart materials*, as máscaras procedurais guiadas por curvatura e oclusão e a construção procedural de materiais (Substance Designer). Apoio direto aos Capítulos 9 e 10.
- *Blender Manual* — https://docs.blender.org/. Seções sobre materiais, *shading*, geração de mapas (*baking*), normais e exportação, para observar os conceitos da Parte III em uma ferramenta concreta e de código aberto.
- *Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/. Seções sobre materiais e shaders, importação de texturas e espaço de cor (sRGB vs. linear), empacotamento de canais, tipos de luz, *reflection probes* e IBL, pós-processamento, exposição/*tone mapping* e compressão de texturas. Apoio aos Capítulos 9 e 11.
- *Unity Manual* — https://docs.unity3d.com/. Seções correspondentes sobre o shader *Standard* e materiais PBR, importação e compressão de texturas, espaço de cor, iluminação e *reflection probes*, e pós-processamento. Apoio aos Capítulos 9 e 11.
- *3DCoat Documentation* — https://3dcoat.com/documentation/. Recursos de pintura PBR e geração de mapas, úteis para comparar abordagens de autoria entre softwares.
- *Godot Engine Documentation* — https://docs.godotengine.org/. Seções sobre materiais, *shading*, iluminação e ambiente em um motor de código aberto, para comparar a realização dos conceitos da Parte III.

### 2.3 Recursos didáticos abertos

- *Learn OpenGL* — https://learnopengl.com/. As seções sobre PBR (teoria, iluminação analítica e iluminação baseada em imagem) e sobre HDR e *tone mapping* apresentam, do ponto de vista de quem implementa um motor, a realização técnica dos princípios dos Capítulos 8 e 11.
- *The Book of Shaders* — https://thebookofshaders.com/?lan=pt. Introdução acessível à programação de shaders e à geração procedural de padrões e ruídos, útil para compreender por dentro tanto o programa que calcula a aparência (Capítulo 11) quanto a abordagem procedural de autoria (Capítulo 10).

---

## 3. Trilha de leitura sugerida

Para o estudante que deseja aprofundar a Parte III de forma progressiva, sugere-se a seguinte ordem. Comece por esta apostila, para fixar o paradigma e o vocabulário — a separação entre matéria e luz, os mapas, os fluxos de trabalho. Em seguida, leia *The PBR Guide* na íntegra: por ser a fonte direta de toda a parte, ele consolida, com mais detalhe e com as tabelas de valores de referência, tudo o que aqui foi apresentado, e é leitura praticamente obrigatória para quem pretende texturizar profissionalmente. Recorra então a *Real-Time Rendering* para fundamentar com rigor a física da luz, os modelos de sombreamento e a BRDF dos Capítulos 8 e 11, bem como a iluminação baseada em imagem e o *tone mapping*. Para a prática de autoria do Capítulo 10, estude a documentação do Substance 3D, observando a construção por camadas e as máscaras guiadas pela geometria, e confronte-a com os guias de *baking* do projeto, que explicam de onde vêm os mapas que guiam essas camadas. Quem quiser compreender por dentro o programa que calcula a aparência deve recorrer a *Learn OpenGL* e a *The Book of Shaders*, que mostram, respectivamente, como o shader PBR e os padrões procedurais são efetivamente implementados. Por fim, para observar como cada motor concretiza esses conceitos — materiais, importação, espaço de cor, iluminação e pós-processamento —, consulte a documentação da Unreal, da Unity, da Godot e do Blender, confrontando as diferentes implementações de uma mesma física.

> Nota sobre o uso das fontes: conforme as diretrizes editoriais da disciplina, estas referências servem como base e ponto de partida para aprofundamento, não como fonte única. O estudante é encorajado a confrontar diferentes autores e ferramentas, observar como cada motor implementa o mesmo modelo físico de materiais e iluminação, e formar, a partir dessa comparação, uma compreensão própria e crítica do *Physically Based Rendering* — o paradigma que, articulado ao mapeamento da Parte II e aos fundamentos da Parte I, completa o pipeline moderno de texturização para jogos.
