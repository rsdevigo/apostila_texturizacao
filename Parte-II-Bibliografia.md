# Parte II — Bibliografia e Materiais Complementares

Esta seção reúne, de forma consolidada, as referências que sustentam os quatro capítulos da Parte II — coordenadas UV e projeções (Capítulo 4), UV unwrapping (Capítulo 5), densidade de texels e organização de UVs (Capítulo 6) e preparação de *assets* para produção (Capítulo 7). Como na Parte I, ela separa o material de referência do projeto — a bibliografia oficial da disciplina — das fontes complementares externas incorporadas para atualizar, aprofundar e contextualizar os conceitos. As fontes complementares foram selecionadas por sua autoridade reconhecida na área (livros-referência da computação gráfica em tempo real, guias técnicos amplamente adotados na indústria, conhecimento consolidado das comunidades de artistas técnicos e documentação oficial das principais ferramentas), e não substituem, mas ampliam, o material da disciplina.

As referências estão organizadas por tema, de modo a orientar o estudante sobre onde buscar aprofundamento conforme o assunto de interesse. Ao final, indica-se uma trilha de leitura sugerida.

---

## 1. Material de referência do projeto

Conjunto de apostilas e documentos técnicos que constituem a bibliografia base da disciplina e que sustentam, de modo direto, os temas da Parte II.

- *AOD — Texel Density (2020)* (material de apoio da disciplina). Referência aplicada sobre como medir, padronizar e verificar a densidade de texels em um fluxo de produção; base do Capítulo 6.
- *AOD — TD Checker Maps* (material de apoio da disciplina). Conjunto de texturas de verificação (*checker maps*) para inspeção de distorção e densidade, instrumento prático discutido nos Capítulos 4 e 6.
- OLSEN, Morten. *The Ultimate Trim* (material de apoio da disciplina). Estudo detalhado da técnica de *trim sheets* e de sua aplicação à construção modular de cenários; base da discussão de reutilização e modularidade do Capítulo 7.
- *Blender Maps* (material de apoio da disciplina). Situa as coordenadas, o desdobramento e os mapas dentro de um fluxo de trabalho concreto; apoio aos Capítulos 4 e 5.
- *Blender Normals* (material de apoio da disciplina). Aborda a relação entre normais, grupos de suavização e costuras, central para a higiene da malha do Capítulo 7.
- *What Is Texture Baking?* e *Baking Guide* (materiais de apoio da disciplina). Introduzem a transferência de detalhe entre a malha de alta e a de baixa densidade, processo que a preparação do Capítulo 7 torna possível e que a Parte III desenvolverá.

---

## 2. Fontes complementares externas

### 2.1 Livros de referência em computação gráfica

- AKENINE-MÖLLER, Tomas; HAINES, Eric; HOFFMAN, Naty. *Real-Time Rendering*. 4. ed. Boca Raton: CRC Press, 2018. Referência central de gráficos em tempo real; o Capítulo 6 ("Texturing") trata do mapeamento de texturas, das coordenadas UV, dos modos de endereçamento, do *mipmapping* e da filtragem — fundamentos dos Capítulos 4 e 6 —, e capítulos posteriores cobrem níveis de detalhe (LOD) e otimização, base de decisões de preparação do Capítulo 7.
- PHARR, Matt; JAKOB, Wenzel; HUMPHREYS, Greg. *Physically Based Rendering: From Theory to Implementation*. 4. ed. Cambridge: MIT Press, 2023. Disponível gratuitamente em https://www.pbr-book.org/. Tratado avançado que aprofunda a parametrização de superfícies e a amostragem de texturas, pano de fundo teórico do mapeamento.

### 2.2 Guias técnicos e conhecimento da indústria

- IEZZI, Leonardo. *All You Need to Know about Texel Density*. ArtStation, 2016. Disponível em https://www.artstation.com/artwork/qbOqP. Guia prático amplamente adotado sobre cálculo e padronização de densidade de texels; referência direta do Capítulo 6.
- POLYCOUNT WIKI. Verbetes *UV Mapping*, *Texel* e *Texel Density* (wiki.polycount.com / polycount.com). Conhecimento consolidado da comunidade de artistas técnicos sobre posicionamento de costuras, empacotamento, *padding* e densidade.
- *Hotspot Texturing*. Default Interactive, disponível em https://www.defaultinteractive.co.uk/post/hotspot-texturing. Explicação prática da técnica de *hotspot* e de seu uso na texturização modular eficiente; apoio ao Capítulo 7.

### 2.3 Documentação oficial de ferramentas e motores

- *Blender Manual* — https://docs.blender.org/. Seções "UV Editing" (projeções, marcação de costuras, ilhas e empacotamento) e as relativas a normais, grupos de suavização, aplicação de transformações e exportação FBX/glTF.
- *Unreal Engine Documentation* — https://dev.epicgames.com/documentation/unreal-engine/. Seções sobre coordenadas de textura, *texture wrapping*, importação de malhas e texturas, *mipmaps*, UDIMs, escala, LODs e colisão.
- *Unity Manual* — https://docs.unity3d.com/. Seções correspondentes sobre importação de malhas e texturas, filtragem, *wrap modes*, UDIMs e níveis de detalhe.
- *Adobe Substance 3D* — https://helpx.adobe.com/substance-3d.html. Recursos de desdobramento, ferramentas de costura e uso de *checker maps*.
- *3DCoat Documentation* — https://3dcoat.com/documentation/. Recursos de desdobramento e organização de UVs, úteis para comparar abordagens entre softwares.
- *Godot Engine Documentation* — https://docs.godotengine.org/. Importação de *assets*, materiais e texturas em um motor de código aberto.

### 2.4 Recursos didáticos abertos

- *Scratchapixel — Introduction to Texturing*. Disponível em https://www.scratchapixel.com/. Reconstrução conceitual e progressiva da relação entre superfície e textura e do papel das coordenadas de mapeamento; apoio ao Capítulo 4.

---

## 3. Trilha de leitura sugerida

Para o estudante que deseja aprofundar a Parte II de forma progressiva, sugere-se a seguinte ordem. Comece por esta apostila, para fixar o vocabulário do mapeamento — coordenadas, costuras, ilhas, densidade. Em seguida, leia o Capítulo 6 de *Real-Time Rendering* para consolidar, com rigor, as noções de coordenadas de textura, modos de endereçamento, *mipmapping* e filtragem, que fundamentam tanto as projeções do Capítulo 4 quanto o *padding* do Capítulo 6. Recorra então à documentação prática — o *Blender Manual* para observar o desdobramento e o empacotamento em uma ferramenta concreta, e o guia de Leonardo Iezzi e os verbetes da Polycount Wiki para a densidade de texels — confrontando como cada fonte descreve os mesmos procedimentos. Para a parte de produção e reutilização (Capítulo 7), estude *The Ultimate Trim* e o artigo sobre *hotspot texturing*, que mostram, na prática, como mundos grandes são construídos com pouca memória. Por fim, quem quiser antecipar a Parte III deve começar por *What Is Texture Baking?* e pelo *Baking Guide*, que introduzem a relação entre as malhas de alta e de baixa densidade preparada ao final desta parte.

> Nota sobre o uso das fontes: conforme as diretrizes editoriais da disciplina, estas referências servem como base e ponto de partida para aprofundamento, não como fonte única. O estudante é encorajado a confrontar diferentes autores, observar como cada motor e ferramenta implementa os mesmos conceitos de mapeamento, densidade e preparação, e formar, a partir dessa comparação, uma compreensão própria e crítica.
