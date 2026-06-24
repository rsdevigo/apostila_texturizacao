# Revisão Técnica — Lista de Problemas Encontrados

Revisão feita do ponto de vista de Computação Gráfica, PBR e produção de assets. Foco em: erros técnicos, simplificações excessivas, desatualização, terminologia inconsistente e problemas de PBR / Unreal / Unity.

Capítulos lidos integralmente nesta passagem técnica: 3, 8, 9, 10, 11, 15, 16, 17, 18, 19, 20, 21, 22 e o Glossário. Os capítulos 1, 2, 4–7, 12–14 e 23–25 não foram relidos linha a linha nesta passagem (são menos densos em PBR/motores); os problemas abaixo concentram-se no núcleo técnico revisado.

**Avaliação geral:** o conteúdo é tecnicamente correto e maduro na esmagadora maioria. Os pontos abaixo são, em sua maior parte, imprecisões e inconsistências, não erros graves. Estão ordenados por relevância.

---

## 1. Erros conceituais / simplificações

**1.1 — Cap. 3: índice de refração listado como propriedade autorada pelo artista.**
Na seção "Os princípios da renderização baseada em física", o texto diz que o PBR modela a superfície a partir de "o índice de refração, a rugosidade, a metalicidade...". No fluxo *metallic/roughness* — que a própria apostila adota como padrão — o artista **não** define o índice de refração (IOR): a reflectância dos dielétricos é fixada em ~4% (F0 ≈ 0,04, IOR ≈ 1,5). Listar o IOR entre as propriedades declaradas pode sugerir um controle que o fluxo padrão não oferece. Sugestão: deixar claro que o IOR é assumido como constante para dielétricos no fluxo metálico, sendo parâmetro explícito apenas em fluxos/shaders especializados.

**1.2 — Cap. 3: rugosidade "varia de modo linear de zero a um".**
Os valores armazenados na textura são, de fato, lineares de 0 a 1, mas a *resposta visual* à rugosidade é fortemente não linear — os modelos de microfaceta tipicamente usam α = rugosidade² (perceptual roughness) antes de alimentar a BRDF. A formulação atual pode dar a impressão de que dobrar o valor do mapa dobra o "tamanho" do realce, o que não ocorre. Vale um esclarecimento de que a relação valor→aparência não é proporcional.

**1.3 — Cap. 11: "material Standard da Unity" como exemplo do shader PBR — desatualizado.**
O texto cita "o material *Standard* da Unity, os materiais base da Unreal" como os shaders PBR padrão. O shader *Standard* pertence ao **Built-in Render Pipeline (legado)**. Em projetos novos de Unity (URP/HDRP, hoje o caminho recomendado) o shader PBR é o **Lit**. Como o Cap. 22 já trata corretamente dos múltiplos pipelines da Unity, a menção isolada ao "Standard" no Cap. 11 ficou inconsistente e datada. Sugestão: citar "Lit (URP/HDRP) / Standard (Built-in)".

---

## 2. Inconsistências de terminologia e de referências cruzadas

**2.1 — Cap. 15: baking atribuído à "Parte III", quando só é ensinado na Parte V.**
O capítulo afirma repetidamente que as máscaras de curvatura, oclusão e ID vêm "do *baking* estudado na Parte III" / "obtidas pelo *baking* da Parte III" (Desenvolvimento e Resumo). O *baking* é apenas **mencionado/antecipado** no Cap. 10 (Parte III); seu tratamento formal está nos Caps. 16–17 (Parte V). Para o leitor linear, isso (a) credita a uma parte um conteúdo que ela não desenvolve e (b) agrava o problema estrutural já identificado na sua análise — o Cap. 15 usa, como ferramentas centrais, mapas cujo processo de geração ainda não foi ensinado. Recomenda-se corrigir as referências para a Parte V (com nota de "que será detalhado adiante") ou reordenar.

**2.2 — Glossário: definição de "Texture array" imprecisa e inconsistente com o Cap. 19.**
O glossário define texture array como estrutura "usada para multiplicar a densidade de detalhe (por exemplo, em terrenos)". Isso contradiz a explicação (correta) do Cap. 19: o texture array serve para ter **muitas texturas uniformes sob uma única ligação**, acessadas por índice, para *mistura* e *variação* — não para multiplicar densidade de texels (isso é papel do UDIM/multi-tile). A definição do glossário troca a finalidade da técnica. Corrigir para algo como: "conjunto de texturas de mesmo tamanho/formato empilhadas e acessadas por índice numa única ligação, para mistura (terrenos) e variação sem custo de troca de textura".

**2.3 — Glossário: "Cor base ... tratado como dado de cor (sRGB)".**
Chamar a cor base de "dado de cor" embaralha a distinção central **cor versus dados** que a apostila constrói com cuidado (a cor base é "cor", interpretada em sRGB; "dado" é o que vai linear para cálculo). Sugestão: "tratada como textura de **cor** (sRGB)".

**2.4 — Oscilação entre "mapa metálico"/"metálico" e "metalicidade".**
Os Caps. 3, 8 e 9 falam em "mapa metálico" / "metálico"; o Glossário, o Cap. 20 e o Cap. 22 usam "metalicidade". Ambos são aceitáveis, mas convém padronizar um termo principal (e registrar o sinônimo no glossário) para manter a consistência terminológica que a obra preza.

---

## 3. Pontos de atualização / nuance (baixa prioridade)

**3.1 — Cap. 18: "compartilhar textura → pode ser agrupado em draw call" merece ressalva para motores atuais.**
A regra apresentada (mesmo material/textura ⇒ *batching*) é pedagogicamente válida e correta para *static/dynamic batching* clássico. Porém, em pipelines modernos ela é menos universal: o **SRP Batcher** da Unity agrupa por variante de shader/material (sem exigir atlas compartilhado), o **GPU instancing** agrupa malhas idênticas independentemente de atlas, e o **Nanite** (UE5) altera por completo a contabilidade de draw calls. O capítulo não está errado, mas, para não soar datado, valeria uma nota de que os mecanismos modernos de batching reduzem — sem eliminar — a dependência do atlas para esse fim.

**3.2 — Verificação recomendada (fora do escopo desta leitura): convenções de canal ORM/RMA e nomes de mapas de mesh por ferramenta.**
As convenções OpenGL/DirectX (Caps. 17, 22) e ORM/RMA (Cap. 20) estão descritas corretamente. Como são exatamente os pontos que mais variam entre versões de ferramentas/motores, recomenda-se manter uma checagem periódica contra a documentação atual (Substance, Unreal, Unity) a cada revisão do material.

---

## Observação final

Nenhum erro físico de PBR relevante foi encontrado nos fundamentos (conservação de energia, Fresnel, condutores/dielétricos, ~4% de reflectância dielétrica, espaço linear, microsuperfície/rugosidade): os Caps. 8, 9, 10 e 11 estão corretos. As convenções de normal map (OpenGL/Unity, DirectX/Unreal), o tratamento de cor vs. dados na compressão, mipmaps (~⅓ de memória extra), packing ORM e a distinção UDIM (autoria) × texture array (tempo real) estão todos tecnicamente precisos. Os problemas listados são pontuais e de correção simples.
