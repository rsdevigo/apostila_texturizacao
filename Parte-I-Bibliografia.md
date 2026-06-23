# Parte I — Bibliografia e Materiais Complementares

Esta seção reúne, de forma consolidada, as referências que sustentam os três capítulos da Parte I. Ela separa o material de referência do projeto — a bibliografia oficial da disciplina — das fontes complementares externas incorporadas para atualizar, aprofundar e contextualizar os conceitos. As fontes complementares foram selecionadas por sua autoridade reconhecida na área (artigos seminais, livros-referência da computação gráfica em tempo real e documentação oficial das principais ferramentas), e não substituem, mas ampliam, o material da disciplina.

As referências estão organizadas por tema, de modo a orientar o estudante sobre onde buscar aprofundamento conforme o assunto de interesse. Ao final, indica-se uma trilha de leitura sugerida.

---

## 1. Material de referência do projeto

Conjunto de apostilas e documentos técnicos que constituem a bibliografia base da disciplina.

- McDERMOTT, Wes. *The PBR Guide*. Allegorithmic/Adobe, 3. ed., 2018. Guia de referência sobre renderização baseada em física, tipos de mapas e fluxos de trabalho.
- *What Is Texture Baking?* (material de apoio da disciplina). Introdução ao processo de transferência de detalhe entre modelos de alta e baixa densidade.
- Demais materiais de apoio do projeto: *Baking Guide*, *Blender Maps*, *Blender Normals*, *AOD — Texel Density (2020)*, *The Ultimate Trim* (Morten Olsen), entre outros — referências aplicadas para as partes seguintes da disciplina.

---

## 2. Fontes complementares externas

### 2.1 Livros de referência em computação gráfica

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Referência central de gráficos em tempo real; o Capítulo 6 ("Texturing") cobre mapeamento, texels e detalhe como informação, e capítulos posteriores tratam do sombreamento físico.
- PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023. Disponível gratuitamente em https://www.pbr-book.org/. Tratado avançado sobre a teoria física da renderização e sua implementação.

### 2.2 Artigos seminais (fundamentos históricos)

- CATMULL, Edwin. *A Subdivision Algorithm for Computer Display of Curved Surfaces*. Tese de doutorado, University of Utah, 1974. Origem do mapeamento de texturas sobre superfícies curvas.
- BLINN, James F. Simulation of wrinkled surfaces. *ACM SIGGRAPH Computer Graphics*, v. 12, n. 3, p. 286–292, 1978. Artigo fundador do *bump mapping*.
- COOK, Robert L.; TORRANCE, Kenneth E. A reflectance model for computer graphics. *ACM SIGGRAPH Computer Graphics*, v. 16, n. 3, 1982. Modelo de reflectância que antecipou os fundamentos do PBR.
- WILLIAMS, Lance. Pyramidal parametrics. *ACM SIGGRAPH Computer Graphics*, v. 17, n. 3, p. 1–11, 1983. Introdução do *mipmapping*.

### 2.3 Documentos da indústria e cursos técnicos

- KARIS, Brian. *Real Shading in Unreal Engine 4*. SIGGRAPH 2013, curso "Physically Based Shading in Theory and Practice". Registro da adoção do fluxo PBR por um motor comercial. Disponível em https://blog.selfshadow.com/publications/s2013-shading-course/.
- IEZZI, Leonardo. *All You Need to Know about Texel Density*. ArtStation, 2016. Guia prático de densidade de texels, referência para a parte de mapeamento UV. Disponível em https://www.artstation.com/artwork/qbOqP.
- POLYCOUNT WIKI. Verbetes *Texel* e *Texel Density* (wiki.polycount.com / polycount.com). Conhecimento consolidado da comunidade de artistas técnicos.

### 2.4 Recursos didáticos abertos

- DE VRIES, Joey. *LearnOpenGL* — seções "Lighting" e "PBR". Disponível em https://learnopengl.com/. Funcionamento do shader e do cálculo de iluminação por ponto, com código comentado.
- GONZALEZ VIVO, Patricio; LOWE, Jen. *The Book of Shaders*. Disponível em https://thebookofshaders.com/. Introdução acessível à lógica dos shaders na GPU (com versão em português).
- *Scratchapixel — Introduction to Texturing*. Disponível em https://www.scratchapixel.com/. Reconstrução conceitual da relação entre superfície e textura.
- *Graphics Compendium — Cook-Torrance Reflectance Model*. Disponível em https://graphicscompendium.com/. Explicação didática do modelo de reflectância base do PBR.

### 2.5 Documentação oficial de ferramentas e motores

- *Blender Manual* — https://docs.blender.org/
- *Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/
- *Unity Manual* — https://docs.unity3d.com/
- *Adobe Substance 3D* — https://helpx.adobe.com/substance-3d.html
- *3DCoat Documentation* — https://3dcoat.com/documentation/
- *Godot Engine Documentation* — https://docs.godotengine.org/
- *OpenGL* — https://www.opengl.org/ — e *Direct3D 12 Programming Guide* — https://learn.microsoft.com/en-us/windows/win32/direct3d12/

---

## 3. Trilha de leitura sugerida

Para o estudante que deseja aprofundar a Parte I de forma progressiva, sugere-se a seguinte ordem. Comece por esta apostila, para fixar o vocabulário. Em seguida, leia o Capítulo 6 de *Real-Time Rendering* para consolidar a noção de textura e mapeamento, e o início de *The PBR Guide* para entender materiais como descrições físicas da superfície. Só então, havendo interesse pela fundamentação histórica, recorra aos artigos seminais de Catmull, Blinn, Cook-Torrance e Williams, que mostram como cada técnica surgiu para resolver um problema concreto. Por fim, para quem busca a fundamentação matemática completa, *Physically Based Rendering*, de Pharr, Jakob e Humphreys, oferece o tratamento mais rigoroso, devendo ser lido em paralelo aos recursos práticos *LearnOpenGL* e *The Book of Shaders*.

> Nota sobre o uso das fontes: conforme as diretrizes editoriais da disciplina, estas referências servem como base e ponto de partida para aprofundamento, não como fonte única. O estudante é encorajado a confrontar diferentes autores, observar como cada motor e ferramenta implementa os mesmos conceitos e formar, a partir dessa comparação, uma compreensão própria e crítica.
