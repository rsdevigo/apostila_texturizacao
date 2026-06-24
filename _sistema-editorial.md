# Sistema Editorial — Apostila de Texturização

Este documento define as convenções visuais e editoriais da apostila. Todos os capítulos devem seguir estas convenções para garantir consistência e aparência de livro didático profissional.

---

## 1. Estrutura Obrigatória de Capítulo

Todo capítulo deve seguir esta ordem de seções:

```
# Título do Capítulo

## Introdução
## [Seções de desenvolvimento — títulos livres]
## Aplicação em Jogos
## Estudo de Caso
## Boas Práticas
## Erros Comuns
## Resumo
## Exercícios
## Glossário
## Leituras Complementares
```

---

## 2. Caixas de Destaque

Usadas ao longo do texto para chamar atenção para informações especiais. Sempre usar blockquote com ícone e rótulo em negrito.

### 📘 Definição

Usado para introduzir termos técnicos com precisão conceitual.

```markdown
> 📘 **Definição**
> **Texel density** é a relação entre o tamanho físico de uma superfície no espaço 3D e a quantidade de pixels de textura que a recobre, expressa geralmente em pixels por centímetro ou pixels por metro.
```

**Resultado:**

> 📘 **Definição**
> **Texel density** é a relação entre o tamanho físico de uma superfície no espaço 3D e a quantidade de pixels de textura que a recobre, expressa geralmente em pixels por centímetro ou pixels por metro.

---

### ⚠️ Atenção

Usado para alertar sobre conceitos frequentemente mal compreendidos ou decisões de alto impacto.

```markdown
> ⚠️ **Atenção**
> Texturas com resoluções inconsistentes entre assets de uma mesma cena produzem um efeito visual de "colagem" que compromete a coerência artística do jogo, mesmo que cada asset individualmente esteja tecnicamente correto.
```

**Resultado:**

> ⚠️ **Atenção**
> Texturas com resoluções inconsistentes entre assets de uma mesma cena produzem um efeito visual de "colagem" que compromete a coerência artística do jogo, mesmo que cada asset individualmente esteja tecnicamente correto.

---

### 💡 Dica Profissional

Usado para compartilhar práticas consolidadas da indústria que não são óbvias para iniciantes.

```markdown
> 💡 **Dica Profissional**
> Estúdios de médio e grande porte mantêm um asset de referência de calibração — geralmente um cubo de 1×1×1 metro com uma textura de checker — que todos os artistas usam para verificar a density antes de exportar qualquer modelo.
```

**Resultado:**

> 💡 **Dica Profissional**
> Estúdios de médio e grande porte mantêm um asset de referência de calibração — geralmente um cubo de 1×1×1 metro com uma textura de checker — que todos os artistas usam para verificar a density antes de exportar qualquer modelo.

---

### ❌ Erro Comum

Usado para documentar equívocos frequentes entre estudantes e profissionais iniciantes.

```markdown
> ❌ **Erro Comum**
> Aplicar texturas antes de finalizar o unwrap UV é um dos erros mais frequentes. Qualquer alteração posterior na malha invalida o mapeamento e exige retrabalho completo.
```

**Resultado:**

> ❌ **Erro Comum**
> Aplicar texturas antes de finalizar o unwrap UV é um dos erros mais frequentes. Qualquer alteração posterior na malha invalida o mapeamento e exige retrabalho completo.

---

## 3. Figuras

Não gerar imagens. Indicar com o seguinte formato:

```markdown
> **Figura X.Y** — Descrição da imagem.

**Screenshot sugerido:** Descrição detalhada do que deve ser capturado.
```

A numeração segue o padrão `Capítulo.Figura` (ex: Figura 3.2 = segundo figura do capítulo 3).

---

## 4. Seção: Estudo de Caso

Apresenta um exemplo real ou verossímil da indústria. Deve ter:

- Nome do jogo ou contexto de produção
- Problema ou desafio encontrado
- Solução adotada
- Lição aplicável

```markdown
## Estudo de Caso

### [Nome do Jogo ou Contexto]

[Descrição do contexto e desafio]

[Solução adotada e raciocínio por trás dela]

**O que aprender com isso:** [Síntese da lição aplicável a outros projetos]
```

---

## 5. Seção: Exercícios

Os exercícios priorizam análise crítica, comparação e planejamento. Evitar exercícios dependentes de software.

Cada exercício deve ter um enunciado claro e, quando pertinente, critérios de avaliação ou pontos de reflexão.

```markdown
## Exercícios

**1.** [Enunciado do exercício]

**2.** [Enunciado do exercício]
```

Tipos preferidos:
- Análise e comparação de abordagens
- Diagnóstico de problemas em imagens ou descrições de cenários
- Planejamento de pipeline para um projeto hipotético
- Tomada de decisão justificada

---

## 6. Seção: Glossário

Lista os termos técnicos introduzidos no capítulo com definições concisas. Ordem alfabética.

```markdown
## Glossário

**Albedo:** Canal de textura que armazena a cor base de uma superfície sem informação de iluminação.

**Bake:** Processo de transferir informações de uma malha de alta resolução para um mapa de textura aplicado a uma malha de baixa resolução.
```

---

## 7. Seção: Leituras Complementares

Lista os materiais do projeto que aprofundam o tema do capítulo. Formato:

```markdown
## Leituras Complementares

- **[Título do material]** — Breve indicação do que o leitor encontrará nele e por que vale consultar.
```

---

## 8. Terminologia Padronizada

Para garantir consistência entre capítulos, os seguintes termos devem ser usados conforme indicado:

| Termo preferido | Evitar |
|---|---|
| Mapeamento UV | UV mapping (em inglês) |
| Ilhas UV | UV islands |
| Densidade de texels | Texel density (em inglês isolado) |
| Espaço UV | UV space |
| Canal de textura | Texture channel / texture map |
| Malha | Mesh |
| Motor de jogo | Game engine |
| Renderização em tempo real | Real-time rendering |
| Sombreador | Shader |
| Mapa de normais | Normal map |
| Mapa de rugosidade | Roughness map |
| Mapa de oclusão ambiente | Ambient occlusion map / AO map |
| Mapa metálico | Metallic map |
| Desdobramento UV | UV unwrapping |
| Costura | Seam |

> ⚠️ **Atenção**
> Termos em inglês podem ser usados entre parênteses na primeira ocorrência de um capítulo, quando isso facilitar a compreensão. Nas ocorrências seguintes, usar apenas o termo em português.

---

## 9. Numeração e Referências Cruzadas

- Capítulos são numerados sequencialmente (Capítulo 1, Capítulo 2…)
- Figuras seguem o padrão `Figura X.Y`
- Referências internas usam: *"como discutido no Capítulo 2"* ou *"conforme apresentado na seção de Mapeamento UV"*
- Evitar referências a "capítulos anteriores" sem especificar qual

---

## 10. Tom e Voz

- Voz ativa, terceira pessoa ou impessoal quando possível
- Evitar "você deve", "você vai", "vamos ver" — registros de tutorial
- Preferir "o artista", "a equipe", "o profissional", "o estudante"
- Períodos longos são bem-vindos quando a complexidade do conceito exige
- Conectivos de raciocínio são preferíveis a listas: *"isso implica que"*, *"por consequência"*, *"é justamente por essa razão que"*
