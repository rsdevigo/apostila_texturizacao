# Parte I — Exercícios Integradores

Os exercícios a seguir não pertencem a um único capítulo. Eles foram concebidos para forçar a conexão entre os três pilares construídos nesta primeira parte: a distinção conceitual entre material, textura e texturização (Capítulo 1), a trajetória histórica das técnicas e suas razões (Capítulo 2) e o modo como o motor interpreta materiais por meio do shader e do modelo PBR (Capítulo 3). Espera-se que as respostas mobilizem, sempre que possível, vocabulário e ideias de mais de um capítulo, e que privilegiem a análise crítica e o raciocínio de projeto, em detrimento de procedimentos de software.

## Eixo 1 — Síntese conceitual

**1.** Percorra mentalmente a "vida" de um único detalhe visual — por exemplo, um arranhão em uma maçaneta de metal — desde a decisão do artista até o pixel exibido na tela. Descreva como esse arranhão é representado (forma ou informação?), em quais mapas ele aparece, como o mapeamento UV o posiciona sobre a geometria e o que o shader faz com ele no momento da renderização. Sua resposta deve articular conceitos dos três capítulos.

**2.** O princípio "aparência é cálculo, não imagem" aparece, sob formas diferentes, nos três capítulos. Identifique uma manifestação desse princípio em cada capítulo e explique como elas se reforçam mutuamente, formando um único argumento coerente sobre a natureza da texturização.

## Eixo 2 — Análise histórica e técnica combinadas

**3.** A separação entre cor própria e resposta à luz é apresentada no Capítulo 2 como uma conquista histórica difícil e no Capítulo 3 como uma regra técnica (a cor base não deve conter iluminação). Explique como um problema histórico se cristalizou em uma boa prática técnica. Por que um erro que era a norma em eras anteriores tornou-se proibido no paradigma atual?

**4.** O mapa de normais é discutido no Capítulo 2 como solução histórica para reintroduzir detalhe sem geometria e no Capítulo 3 como entrada do cálculo de iluminação. Construa uma explicação única que conecte as duas perspectivas: por que essa solução só faz sentido quando se compreende que aparência resulta da interação entre dados de superfície e luz?

## Eixo 3 — Diagnóstico integrado

**5.** Um estúdio relata que seus objetos, criados por artistas diferentes, parecem coerentes entre si quando vistos em uma cena de teste, mas tornam-se visualmente inconsistentes quando reunidos em um nível com iluminação dinâmica complexa: alguns metais parecem plástico, algumas superfícies foscas reluzem indevidamente e há variações inexplicáveis de tonalidade. Com base nos três capítulos, levante um conjunto de hipóteses para as causas — abrangendo possíveis erros conceituais, heranças de fluxos antigos e problemas de interpretação pelo motor — e proponha uma ordem de investigação, do mais provável ao menos provável.

**6.** Uma textura de cor base de uma parede foi pintada com sombras e oclusão já embutidas, no estilo das primeiras gerações de jogos. Explique, recorrendo aos três capítulos, por que essa escolha pode ter parecido razoável em determinado contexto histórico, por que ela viola as regras do modelo PBR e que sintomas concretos ela produzirá quando o motor renderizar a parede sob iluminação dinâmica.

## Eixo 4 — Planejamento e tomada de decisão

**7.** Você recebe a tarefa de definir a estratégia de texturização para um conjunto de objetos de cenário (rochas, troncos, paredes de pedra) que se repetirão centenas de vezes em um mundo aberto, com restrição severa de memória de vídeo. Articulando conceitos dos três capítulos, justifique quais técnicas históricas você reaproveitaria, como organizaria os mapas PBR para esses objetos e que compromissos de desempenho a interpretação desses materiais pelo motor imporia às suas escolhas.

**8.** Compare dois projetos hipotéticos — um jogo de estética retrô deliberada e um jogo de alta fidelidade fotorrealista — quanto às decisões de material, textura e interpretação pelo motor. Para cada projeto, indique como os conceitos dos três capítulos seriam mobilizados de forma diferente, deixando claro que dominar os fundamentos não significa aplicar sempre a técnica mais avançada, e sim a mais adequada.

## Eixo 5 — Argumentação crítica

**9.** "O PBR tornou os artistas técnicos obsoletos, pois agora basta seguir regras físicas para obter bons materiais." Avalie criticamente essa afirmação, mobilizando os três capítulos. Em que medida ela contém um fundo de verdade e em que medida ela ignora o papel do julgamento artístico e da compreensão do motor?

**10.** Redija um parágrafo único, em linguagem própria e sem listas, que sirva como definição integradora de "texturização" para um colega que está começando a disciplina. A definição deve, simultaneamente, distinguir texturização de material e de textura, situá-la como herdeira de uma trajetória histórica e explicá-la como processo cujos resultados só se realizam quando o motor interpreta os dados. Considere este exercício uma síntese de toda a Parte I.
