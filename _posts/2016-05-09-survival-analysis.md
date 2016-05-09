---
layout: post
title: Análise de sobrevivência (Survival Analysis)
date:  2016-05-09 11:44:19
summary: Pequeno estudo sobre Análise de sobreviência que é um ramo da estatística que analisa tempos até ao acontecimento de determinado evento. 
categories:
  - statiscs
  - studies
tags:
  - survival analysis
  - análise de sobrevivência
  - estatística
  - churn
  - churning
author: Luiz Mendes
authurl: https://www.linkedin.com/in/lfomendes
---

Na <a href="http://www.hekima.com" target="_blank">Hekima</a>, os desenvolvedores fazem de tempos em tempos apresentações sobre algoritmos, artigos científicos, tećnicas de programação e outras coisas. A minha apresentação foi sobre uma técnica estatística chamada Análise de Sobrevivência (Survival Analysis). Decidi aproveitar a oportunidade e transformar a apresentação em um post neste blog.

Nesse post vou passar pelas definições necessárias para entender o que é essa técnica e quando utilizá-la.

<h2>O que é?</h2>

<div style="text-align: center; margin: 1em">
<img src="/images/bear.jpg" alt="Bear Grylls" title="Bear Grylls">
</div>

**Survival analysis** ou análise de sobrevivência é geralmente definido como um método de analisar dados em que a saída é definida pelo tempo até a ocorrência de um evento. O evento pode ser a morte de um indivíduo, a ocorrência de uma doença, casamento, divórcio, o churn de um usuário, a duração de uma greve, entre outros.

Normalmente essa análise é feita acompanhando os “subjects” durante um período de tempo, focando na ocorrência do evento de interesse.

Tipicamente, os dados observados não são ”completos”, são censurados.

<h2>Definições dentro de Survival Analysis</h2>

**Censored Data**

<div style="text-align: center; margin: 1em">
<img src="/images/censored.png" alt="Censored" title="Censored">
</div>

Uma observação é chamada de dado censurado quando a informação sobre seu tempo de sobrevivência está incompleta. Por exemplo: se o caso estudado é de churn, se um cliente não sair do serviço durante o estudo, então ele é considerado censored (censurado).
Outra forma de uma observação ser "censored" é se o usuário observado sair do estudo antes que aconteça o evento esperado.

**Survival Function**

<div style="text-align: center; margin: 1em">
<img src="/images/survival.gif" alt="Survival" title="Survival Function">
</div>

A distribuição de tempos de sobrevivência pode ser caracterizada por uma **função de sobrevivência** representada por S(t). Para um dado t, S(t) especifica a proporção de indivíduos que ainda não sofreram a ocorrência do evento observado.

**Hazard Function**

<div style="text-align: center; margin: 1em">
<img src="/images/hazard.gif" alt="Hazard" title="Hazard Function">
</div>

A Hazard Function tem um significado quase contrário ao da Survival Function. Ela mostra no tempo qual o potencial do evento ocorrer dado que o indivíduo sobreviveu até aquele momento.

A Hazard Function h(x) é a razão da função densidade de probabilidade P(t) pela função de sobrevivência S(t), dada pela fórmula:

h(x)	=	(P(x))/(S(x))	=	(P(x))/(1-D(x))

 Sendo D(x) a <a href="http://mathworld.wolfram.com/DistributionFunction.html" target="_blank">função de distribuição</a>.  
	
	
**Kaplan-Meier Curve**
	
<div style="text-align: center; margin: 1em">
<img src="/images/meier.png" alt="Censored" title="Censored">
</div>

<div style="text-align: center; margin: 1em">
<img src="/images/tabela.png" alt="Censored" title="Censored">
</div>

A Kaplan-Meier Curve é utilizada para visualizar os dados de um estudo de análise de sobrevivência. A tabela acima mostra um estudo feito e o gráfico foi produzido com os valores do estimador de Kaplan-Meier.

Na tabela criada, a primeira coluna é o tempo, a segunda coluna é preenchida com o número de "subjects" naquele momento, e a terceira coluna contém o número de eventos ocorridos entre a última observação e essa. A última coluna possui o valor do estimador de Kaplan-Meier.

No geral a análise de Kaplan-Meier é utilizada com objetivo de estimar a curva de sobrevivência de uma população baseada em uma amostra, ela permite a estimativa no tempo mesmo possuindo casos censurados.

Para saber mais visite: <a href="http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3059453/" target="_blank">http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3059453/</a>

**Modelo de Cox**

<div style="text-align: center; margin: 1em">
<img src="/images/cox.gif" alt="Cox" title="Cox">
</div>

O modelo de Cox é caracterizado pelos coeficientes &Beta; que medem os efeitos das covariáveis sobre a função de risco (hazard function).

Muitas vezes o interesse não é estimar os parâmetros desse tempo e sim o efeito dessas covariáveis

A fórmula do modelo de riscos proporcionais, o modelo de Cox é:

<div style="text-align: center; margin: 1em">
<img src="/images/formulacox.png" alt="Cox" title="Cox">
</div>

Sendo h(t) a função de risco, h0(t) o risco basal, x o vetor de covariáveis e &Beta; o vetor de parâmetros das covariáveis. Note que as covariáveis tem um efeito multiplicativo no modelo.

O modelo de Cox é bem utilizado para fazer experimentos e análises com as covariáveis. Por exemplo, é possível verificar o que acontece com a chance de um usuário realizar o churning se ele mudar de plano (sendo o plano uma covariável do modelo).

<h2>Pra que Serve</h2>

A análise de sobrevivência é um conceito antigo mas muito útil. É bastante utilizada em estudos de medicina e de marketing.
Um dos principais objetivos dessa análise é entender melhor sobre o "evento" estudado, principalmente como ele se relaciona com diversas variáveis. Uma maneira de realizar isso é desenhando as funções de Survival e Hazard para essas diferentes variáveis. Por exemplo, traçando os gráficos de sobrevivência de grupos de idades, gêneros e raças distintas pode-se entender como uma doença afeta pessoas diferentes.
No caso de churning, esse estudo pode esclarecer quais os usuários mais propícios a saírem ou em qual momento esses usuários tendem a sair.


<h2>Churning Analysis VS Survival Analysis</h2>

<div style="text-align: center; margin: 1em">
<img src="/images/churning.gif" alt="Churning" title="Cox">
</div>

Para finalizar gostaria de fazer uma pequena comparação dos estudos de churning normalmente feitos com a análise de sobrevivência.

<div style="text-align: center; margin: 1em">
<img src="/images/tabelachurn.png" alt="Cox" title="Cox">
</div>


<h2>Conclusão</h2>

Survival analysis não foi desenvolvido para prever eventos. Seu uso principal é estimar a curva de sobrevivência que mede a probabilidade de um evento não ocorrer no tempo, levando em conta todos os eventos prévios e as observações censuradas.

A análise de sobrevivência é uma ferramenta muito importante para analisar eventos, dando insights sobre retenção de clientes e quais são os fatores que levam a esse evento.





