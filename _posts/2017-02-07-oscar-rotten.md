---
layout: post
title: "Brincando com dados: Análise do Oscar - Rotten Tomatoes"
date:  2017-02-07 08:44:19
summary: Chegamos naquela época do ano, praticamente o carnaval dos cinéfilos, aquela época que a Meryl Streep vai ser indicada. A época das premiações “mais importantes” do mundo do cinema. Então vamos aproveitar essa época e fazer uma análise sobre as notas no Rotten Tomatoes dos indicados ao Oscar de melhor filme.
categories:
  - machine learning
  - python
tags:
  - machine learning
  - python
  - aprendizado de máquina
  - pandas
  - data analysis
  - análise de dados
  - correlation
  - correlação
author: Luiz Mendes
authurl: https://www.linkedin.com/in/lfomendes
---

Chegamos naquela época do ano, praticamente o carnaval dos cinéfilos, aquela época que a Meryl Streep vai ser indicada. A época das premiações "mais importantes" do mundo do cinema. 

<div style="text-align: center;"> <img src="https://media.giphy.com/media/OpAkQ1sPBnBUA/giphy.gif" alt="meryl" style="width: 300px;"/> </div>

Então vamos aproveitar essa vibe cinematográfica para fazer umas análises?

Ano passado fiz dois posts sobre o mesmo assunto <a href="http://developers.hekima.com/data-science/brincando-com-dados/2016/02/11/importing_oscar_runners/" target="_blank">Brincando com dados: Ganhadores do Oscar Parte 1</a> e <a href="http://developers.hekima.com/data-science/brincando-com-dados/2016/02/11/oscar-analysis/" target="_blank">Brincando com dados: Ganhadores do Oscar Parte 2</a> . Neles eu fiz algumas análises e tentei prever alguns prêmios. Acertei os ganhadores de melhor ator e melhor atriz mas errei a previsão do ganhador de melhor filme... =(

<div style="text-align: center;">  <img src="https://media.giphy.com/media/QJY9tY29IQGMU/giphy.gif" alt="Sadness" style="width: 400px;"/> </div>

Mas eu sou brasileiro e não desisto nunca, fiquei pensando por qual motivo o algoritmo errou o resultado. Aparentemente ano passado foi um outlier, dado que nem o DGA nem o PGA bateram com o ganhador do Oscar.

O Oscar tem um sistema peculiar de votação, que muitas vezes valoriza mais os filmes "menos" odiados, pensei então que os valores do <a href="http://rottentomatoes.com/" target="_blank">Rotten Tomatoes</a> poderiam refletir algo nesse sentido.

## Como funciona o Rotten Tomatoes


O Rotten Tomatoes é um site para avaliação de filmes, seu sistema difere um pouco de sites como <a href="https://www.imdb.com" target="_blank">IMDb</a> pois cada pessoa ou crítico avalia o filme entre 1 e 5 estrelas, mas o site considera esse voto de uma forma binária, isto é, ou o filme é *Fresh* (Fresco) ou *Rotten* (Podre). O score final do filme é o % de notas Fresh que ele recebeu.

Outra diferença do Rotten Tomatoes é que ele possui dois valores, um para críticos e outro para o público. Nesse post vamos analisar esses dois scores separadamente e em conjunto.


## Objetivo 

Então, o objetivo desse post é demonstrar formas de analisar uma feature nova, como visualizar os valores, estudar relações, fazer transformações e quais insights elas podem gerar. Além disso, vou mostrar aqui um pouco de sobre <a href="https://www.mongodb.com" target="_blank"> MongoDB </a>, <a href="https://api.mongodb.com/python/current" target="_blank">PyMongo</a> e principalmente sobre a biblioteca <a href="http://pandas.pydata.org" target="_blank"> Pandas </a>.

## Preparativos
Para ficar mais simples e didático decidimos utilizar o Jupyter Notebook, para fazer a instalação dele siga o tutorial descrito aqui: http://jupyter.readthedocs.org/en/latest/install.html 

Para rodar o código ou acompanhar o post pelo Jupyter Notebook baixe o arquivo <a href="https://github.com/lfomendes/oscarPredictions/blob/master/Oscar%20Analysis%202017.ipynb"  target="_blank"> https://github.com/lfomendes/oscarPredictions/blob/master/Oscar%20Analysis%202017.ipynb </a>

## Base
<a href="https://www.mongodb.com" target="_blank"> MongoDB </a>: O MongoDB é um banco de dados não relacional, ele é orientado a documentos e utiliza um formato chamado BSON, bem próximo ao JSON. Ele é bem simples de entender e utilizar.

<a href="http://pandas.pydata.org" target="_blank">Pandas</a>: Biblioteca de python para estrutura e análise de dados.

<a href="https://api.mongodb.com/python/current" target="_blank">PyMongo</a>: PyMongo é uma biblioteca em python para acessar o MongoDB.


Vamos lá então?


<div style="text-align: center;">  <img src="https://media.giphy.com/media/W3z1M5ek70Ztm/source.gif" alt="rambo" style="width: 300px;"/> </div>


Nós vamos utilizar a mesma base de dados criadas nos posts do ano passado, vamos então criar um cliente e ligar a coleção chamada "movies".


```python
import json
from pymongo import MongoClient

client = MongoClient() # Primeiramente pegamos o client do Mongodb
db = client.imdb 
movie_db = db.movies
```

## Atualizando vencedores dos prêmios

Vamos agora atualizar no MongoDB quais os vencedores das principais premiações.


```python
# Now we are going to add the Sag Winners

sag_winners = ["Hidden Figures","Spotlight","Birdman", "American Hustle", "Argo", "The Help", "The King's Speech", 
               "Inglorious Bastards", "Slumdog Millionaire", "No Country for Old Men", "Little Miss Sunshine (2006)",
               "Crash", "Sideways", "Chicago", "The Lord of the Rings: The Return of the King", "Gosford Park", "Traffic"]

golden_globe = ['Moonlight', 'La la land' ,"The Martian","The Revenant","Boyhood", "The Grand Budapest Hotel", "12 Years a Slave", "American Hustle", "Argo", "Les Misérables", 
               "The Descendants", "The Artist", "The Social Network", "The Kids Are All Right", "Avatar", "The Hangover", 
               "Slumdog Millionaire", "Vicky Cristina Barcelona", "Atonement", "Sweeney Todd", "Babel", "Dreamgirls", 
               "Brokeback Mountain", "Walk the Line", "The Aviator", "Sideways", "The Lord of the Rings: The Return of the King",
               "Lost in Translation", "The Hours", "Chicago", "A Beaultiful Mind", "Moulin Rouge!", "Gladiator", "Almost Famous"]

nyfcc_winners = ['La la land', "Carol","Son of Saul","Boyhood", "Zero Dark Thirty", "The Artist", "The Social Network",
                 "The Hurt Locker", "Slumdog Millionaire","There Will Be Blood", "United 93", 
                 "Sideways", "Brokeback Mountain", "The Lord of the Rings: The Return of the King",
                "Far from Heaven", "Mulholland Drive"]

dga_winners = ["La la land","The Revenant","Birdman","Gravity", "Argo", "The Artist", "The King's Speech", "The Hurt Locker",
               "Slumdog Millionaire", 
               "No Country for Old Men","The Departed", "Brokeback Mountain", "Million Dollar Baby", 
               "The Lord of the Rings: The Return of the King",  "Chicago",
               "A Beautiful Mind", "Crouching Tiger, Hidden Dragon"]

#NAO SAIU AINDA
wga_winners = ['The Big Short',"Spotlight","The Imitation Game","The Grand Budapest Hotel", "Captain Phillips", "Her (2013)", "Argo", "Zero Dark Thirty",
               "The Descendants", 
               "Inception","The Social Network", "The Hurt Locker", "Up in the Air", "Milk", "Juno",
               "Slumdog Millionaire", "No Country for Old Men",  "The Departed",
               "Little Miss Sunshine (2006)","Brokeback Mountain", "Crash", "Sideways", "Eternal Sunshine of the Spotless Mind",
               "American Splendor", "Lost in Translations", "Bowling for Columbine", "The Hours", "A Beautiful Mind", "Gosford Park"
               , "Traffic", "You Can Count on Me"]

pga_winners = ['La la land','The Big Short', "Birdman","12 Years a Slave", "Argo", "The Artist", "The King's Speech", 
               "The Hurt Locker", "Slumdog Millionaire", "No Country for Old Men", 
               "Little Miss Sunshine", "Brokeback Mountain", "The Aviator", "The Lord of the Rings: The Return of the King",
               "Chicago", "Moulin Rouge!", "Gladiator"]

#NAO SAIU AINDA
bafta_winners = ["The Revenant", "Boyhood","12 Years a Slave", "Argo", "The Artist", "The King's Speech", 
               "The Hurt Locker", "Slumdog Millionaire", "Atonement", 
               "The Queen", "Brokeback Mountain", "The Aviator", "The Lord of the Rings: The Return of the King",
               "The Pianist", "The Lord of the Rings: The Fellowship of the Ring", "Gladiator"]
```


```python
# Starting everyone with false
db.movies.update_many({},{"$set": {"sag_winner": False, "golden_globe": False, 
                                   "nyfcc_winner": False,'dga_winner': False,
                                   "wga_winner": False,'pga_winner': False,
                                   "bafta_winner": False}})

for winner in sag_winners:
    db.movies.update_one({'title': winner},{"$set": {"sag_winner": True}})
for winner in golden_globe:
    db.movies.update_one({'title': winner},{"$set": {"golden_globe": True}})
for winner in nyfcc_winners:
    db.movies.update_one({'title': winner},{"$set": {"nyfcc_winner": True}})    
for winner in dga_winners:
    db.movies.update_one({'title': winner},{"$set": {"dga_winner": True}})
for winner in wga_winners:
    db.movies.update_one({'title': winner},{"$set": {"wga_winner": True}})   
for winner in pga_winners:
    db.movies.update_one({'title': winner},{"$set": {"pga_winner": True}})  
for winner in bafta_winners:
    db.movies.update_one({'title': winner},{"$set": {"bafta_winner": True}})    
```

## Carregamento dos dados

Criamos um cursor utilizando uma consulta vazia, isto é, não estamos filtrando nada, todos filmes serão retornados. Logo depois, iteramos pelo cursor adicionando os documentos retornados na lista "movies".

Depois disso, vamos criar um <a href="http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame." target="_blank">DataFrame</a> com todos os filmes e aplicar uma transformação em suas colunas.


```python
movie_cursor = movie_db.find()
i = 0
movies = []
for document in movie_cursor: 
    movies.append(document)    
```


```python
import pandas as pd

df = pd.DataFrame(movies)
df = df.apply(lambda x: pd.to_numeric(x, errors='ignore')) # set all columns that are a number to numeric 
```


```python
df.columns.values
```




    array([u'_id', u'assistant director', u'bafta_winner', u'bafta_winners',
           u'budget', u'cast', u'casting director', u'countries',
           u'dga_winner', u'director', u'distributors', u'editor', u'genres',
           u'golden_globe', u'languages', u'nyfcc_winner', u'opening_weekend',
           u'original music', u'oscar_year', u'pga_winner', u'plot',
           u'production companies', u'rotten_critic', u'rotten_people',
           u'runtime', u'sag_winner', u'special effects companies', u'title',
           u'user_rating', u'wga_winner', u'winner', u'writer'], dtype=object)



# Visualizando Dados

Vamos então analisar nossas features "rotten_critic" e "rotten_public", lembrando que seus valores podem ir de 0 a 100. 

Para visualizar os top filmes em relação a "rotten_critic" vamos fazer uma ordenação no Dataframe, seguido de um filtro de colunas e chamar a função *head()*.



```python
print('O score médio relativo a nota dos críticos dos filmes disputando Oscar de melhor filme foi\n'+ str(df['rotten_critic'].mean()))
print('\nA mediana do score relativo a nota dos críticos dos filmes disputando Oscar de melhor filme foi\n'+str(df['rotten_critic'].median()))

df.sort_values(by='rotten_critic',ascending=False).filter(items=['title', 'rotten_critic', 'oscar_year','winner',]).head(10)
```

    O score médio relativo a nota dos críticos dos filmes disputando Oscar de melhor filme foi
    88.4188034188
    
    A mediana do score relativo a nota dos críticos dos filmes disputando Oscar de melhor filme foi
    91.0





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>59</th>
      <td>Selma</td>
      <td>99</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Toy Story 3</td>
      <td>99</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>89</th>
      <td>The Hurt Locker</td>
      <td>98</td>
      <td>2010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>97</th>
      <td>up</td>
      <td>98</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Boyhood</td>
      <td>98</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Moonlight</td>
      <td>98</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hell or High Water</td>
      <td>98</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>43</th>
      <td>The Queen (2006)</td>
      <td>97</td>
      <td>2007</td>
      <td>False</td>
    </tr>
    <tr>
      <th>73</th>
      <td>Brooklyn</td>
      <td>97</td>
      <td>2016</td>
      <td>False</td>
    </tr>
    <tr>
      <th>74</th>
      <td>Mad Max: Fury Road</td>
      <td>97</td>
      <td>2016</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Podemos ver que dentre os 10 filmes de maior nota dos críticos só existe um ganhador de Oscar, que foi o "The Hurt Locker" com valor 98.

<div style="text-align: center;">  <img src="http://arts.columbia.edu/files/soa/hurt_locker_poster_m_0.jpg" alt="Drawing" style="width: 400px;"/> </div>

Agora vamos ordenar pela nota do público.


```python
print('O score médio relativo a nota do público dos filmes disputando Oscar de melhor filme foi\n'+ str(df['rotten_people'].mean()))
print('\nA mediana do score relativo a nota do público dos filmes disputando Oscar de melhor filme foi\n'+str(df['rotten_people'].median()))

df.sort_values(by='rotten_people',ascending=False).filter(items=['title','rotten_people','oscar_year','winner']).head(10)
```

    O score médio relativo a nota do público dos filmes disputando Oscar de melhor filme foi
    84.3675213675
    
    A mediana do score relativo a nota do público dos filmes disputando Oscar de melhor filme foi
    85.0





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_people</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>The Pianist</td>
      <td>95</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>17</th>
      <td>The Lord of the Rings: The Two Towers</td>
      <td>95</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>22</th>
      <td>The Lord of the Rings: The Fellowship of the Ring</td>
      <td>95</td>
      <td>2002</td>
      <td>False</td>
    </tr>
    <tr>
      <th>104</th>
      <td>Life of Pi</td>
      <td>94</td>
      <td>2013</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Hacksaw Ridge</td>
      <td>94</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hidden Figures</td>
      <td>94</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Whiplash</td>
      <td>94</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>39</th>
      <td>The Departed</td>
      <td>94</td>
      <td>2007</td>
      <td>True</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Room</td>
      <td>93</td>
      <td>2016</td>
      <td>False</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Spotlight</td>
      <td>93</td>
      <td>2016</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



Nesse caso, temos 2 vencedores dentre os 10 primeiros da lista do público (inclusive aquele "maldito" ganhador de 2016), mas será que isso é relevante?

Parece também que o público avalia os filmes com valores **mais baixos**, com uma média quase 4 pontos mais baixa e uma mediana 6 pontos mais baixas. Além disso, podemos perceber que a mediana é bem alta (91) o que mostra que os filmes indicados tendem a ter uma avaliação boa pelos críticos.


Podemos perceber também que os valores da média e da mediana são bem próximos, o que pode indicar que a distribuição seja parecida com uma normal.


```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.style.use('ggplot')

plt.figure()

df['rotten_critic'].value_counts().sort_index().plot(kind='bar'); 
plt.show()
```

<div style="text-align: center;"> <img src="/images/output_16_0.png" alt="chart"> </div>

Parece que existem alguns filmes com scores abaixo de 90 mas a grande maioria fica com valores acima de 91, inclusive alguns filmes com valores abaixo de 70, mas vamos analisá-los mais a frente.


```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.style.use('ggplot')

plt.figure()

df['rotten_people'].value_counts().sort_index().plot(kind='bar'); 
plt.show()
```

<div style="text-align: center;"> <img src="/images/output_18_0.png" alt="chart"> </div>


Já no gráfico relativo ao público vemos um comportamento mais regular, com um pico peculiar no valor 86, variando mais entre 79 e 91. 

Agora que já analisamos a diferença entre o score do crítico e do público, vamos ver se existe uma diferença entre os ganhadores do oscar e os outros?

Vamos criar então dois Dataframes, um com todos ganhadores do Oscar e outros com os perdedores.


```python
winners = df[(df['winner'] == True) & (df['oscar_year'] != 2017)]
losers = df[(df['winner'] == False) & (df['oscar_year'] != 2017)]

print('------- Críticos -------')
print('Vencedores média: '+ str(winners['rotten_critic'].mean()))
print('Perdedores média: '+ str(losers['rotten_critic'].mean()))

print('\nVencedores mediana: '+ str(winners['rotten_critic'].median()))
print('Perdedores mediana: '+ str(losers['rotten_critic'].median()))

print('\n------ Público -------')
print('Vencedores média: '+ str(winners['rotten_people'].mean()))
print('Perdedores média: '+ str(losers['rotten_people'].mean()))

print('\nVencedores mediana: '+ str(winners['rotten_people'].median()))
print('Perdedores mediana: '+ str(losers['rotten_people'].median()))
```

    ------- Críticos -------
    Vencedores média: 90.125
    Perdedores média: 87.6304347826
    
    Vencedores mediana: 92.5
    Perdedores mediana: 91.0
    
    ------ Público -------
    Vencedores média: 88.125
    Perdedores média: 83.402173913
    
    Vencedores mediana: 89.0
    Perdedores mediana: 84.0


Parece que o score do público seria um "termômetro" mais interessante do que o score dos críticos e parece que utilizar esses valores como feature pode ser bom para o modelo final. Imagino que fazer uma normalização dos valores seja importante.

Vamos agora analisar os extremos para ver os **outliers**?

Primeiramente vamos ver quais os **"piores"** filmes a disputar a estatueta?


```python
df.sort_values(by='rotten_critic',ascending=True).filter(items=['title','rotten_critic','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>110</th>
      <td>Extremely Loud &amp; Incredibly Close</td>
      <td>46</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>13</th>
      <td>The Reader</td>
      <td>61</td>
      <td>2009</td>
      <td>False</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Chocolat</td>
      <td>63</td>
      <td>2001</td>
      <td>False</td>
    </tr>
    <tr>
      <th>91</th>
      <td>The Blind Side</td>
      <td>66</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Babel</td>
      <td>69</td>
      <td>2007</td>
      <td>False</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Les Misérables</td>
      <td>69</td>
      <td>2013</td>
      <td>False</td>
    </tr>
    <tr>
      <th>55</th>
      <td>American Sniper</td>
      <td>72</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Benjamin Button</td>
      <td>72</td>
      <td>2009</td>
      <td>False</td>
    </tr>
    <tr>
      <th>111</th>
      <td>The Help</td>
      <td>75</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Crash</td>
      <td>75</td>
      <td>2006</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



Parece que o filme com menor score dos críticos foi <a href="https://www.rottentomatoes.com/m/extremely_loud_and_incredibly_close" target="_blank">Extremely Loud & Incredibly Close</a> com Tom Hanks (Shame on you Tom...) , seguido por <a href="https://www.rottentomatoes.com/m/reader" target="_blank">O Leitor</a> e <a href="https://www.rottentomatoes.com/m/1103080-chocolat" target="_blank">Chocolat</a>.

<div style="text-align: center;"> <img src="https://media.giphy.com/media/3oEjHWzZQaCrZW2aWs/giphy.gif" alt="Hanks" style="width: 400px;"/> </div>


O único ganhador do Oscar que está no bottom 10 dos críticos é <a href="https://www.rottentomatoes.com/m/1144992-crash" target="_blank">Crash</a> , que como já foi dito em posts anteriores foi uma vitória bem discutível em 2006 quando disputou contra <a href="https://www.rottentomatoes.com/m/munich" target="_blank">Munique</a>, <a href="https://www.rottentomatoes.com/m/1151898-capote" target="_blank">Capote</a>, <a href="https://www.rottentomatoes.com/m/brokeback_mountain" target="_blank">O Segredo de Brokeback Mountain</a> e <a href="https://www.rottentomatoes.com/m/1152019-good_night_and_good_luck" target="_blank">Boa Noite e Boa Sorte</a>.

Vamos ver agora com relação ao público?


```python
df.sort_values(by='rotten_people',ascending=True).filter(items=['title','rotten_people','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_people</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>115</th>
      <td>The Tree of Life</td>
      <td>60</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>110</th>
      <td>Extremely Loud &amp; Incredibly Close</td>
      <td>61</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>96</th>
      <td>A Serious Man</td>
      <td>67</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Michael Clayton</td>
      <td>69</td>
      <td>2008</td>
      <td>False</td>
    </tr>
    <tr>
      <th>84</th>
      <td>The Kids Are All Right</td>
      <td>73</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>116</th>
      <td>War Horse</td>
      <td>74</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>62</th>
      <td>American Hustle</td>
      <td>74</td>
      <td>2014</td>
      <td>False</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Seabiscuit</td>
      <td>76</td>
      <td>2004</td>
      <td>False</td>
    </tr>
    <tr>
      <th>88</th>
      <td>Winter's Bone</td>
      <td>76</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Beasts of the Southern Wild</td>
      <td>76</td>
      <td>2013</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Parece que <a href="https://www.rottentomatoes.com/m/the_tree_of_life_2011" target="_blank">Árvore da Vida</a> não foi muito curtido pelo público de maneira geral, outro ponto interessante é que nesse bottom 10 não temos nenhum ganhador de Oscar.

<div style="text-align: center;">  <img src="https://media.giphy.com/media/LPGtFowd29hza/source.gif" alt="Hanks" style="width: 400px;"/> </div>

O quão diferente será que é a nota do público com a dos críticos para o mesmo filme? 

Vamos ver os casos extremos relativos a essa diferença, vamos criar uma nvoa coluna que é a nota do crítico menos a nota do público.


```python
df['diff'] = df['rotten_critic'].sub(df['rotten_people'])
print('Média da diferença: ' + str(df['diff'].mean()))
print('Mediana da diferença: ' + str(df['diff'].median()))
df.sort_values(by='diff',ascending=True).filter(items=['title','rotten_critic','rotten_people','diff','oscar_year','winner']).head(10)
```

    Média da diferença: 4.05128205128
    Mediana da diferença: 5.0





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>diff</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>30</th>
      <td>Chocolat</td>
      <td>63</td>
      <td>83</td>
      <td>-20</td>
      <td>2001</td>
      <td>False</td>
    </tr>
    <tr>
      <th>91</th>
      <td>The Blind Side</td>
      <td>66</td>
      <td>85</td>
      <td>-19</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>13</th>
      <td>The Reader</td>
      <td>61</td>
      <td>79</td>
      <td>-18</td>
      <td>2009</td>
      <td>False</td>
    </tr>
    <tr>
      <th>19</th>
      <td>A Beautiful Mind</td>
      <td>75</td>
      <td>93</td>
      <td>-18</td>
      <td>2002</td>
      <td>True</td>
    </tr>
    <tr>
      <th>110</th>
      <td>Extremely Loud &amp; Incredibly Close</td>
      <td>46</td>
      <td>61</td>
      <td>-15</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>111</th>
      <td>The Help</td>
      <td>75</td>
      <td>89</td>
      <td>-14</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Moulin Rouge!</td>
      <td>76</td>
      <td>89</td>
      <td>-13</td>
      <td>2002</td>
      <td>False</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Crash</td>
      <td>75</td>
      <td>88</td>
      <td>-13</td>
      <td>2006</td>
      <td>True</td>
    </tr>
    <tr>
      <th>55</th>
      <td>American Sniper</td>
      <td>72</td>
      <td>84</td>
      <td>-12</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Gladiator</td>
      <td>76</td>
      <td>87</td>
      <td>-11</td>
      <td>2001</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



Temos diferenças de 20 pontos!! Parece que público e críticos não tem a mesma visão sobre filmes como <a href="https://www.rottentomatoes.com/m/1103080-chocolat)" target="_blank">Chocolat</a>, <a href="https://www.rottentomatoes.com/m/beautiful_mind" target="_blank">Uma mente brilhante</a> e <a href="https://www.rottentomatoes.com/m/gladiator" target="_blank">Gladiador</a>. (Será que tem alguma ligação com o Russell Crowe?)

<div style="text-align: center;">  <img src="https://media.giphy.com/media/rvaQRHCzisFeo/giphy.gif" alt="Hanks" style="width: 400px;"/> </div>


Outra coisa interessante que podemos notar é que existem 3 ganhadores do Oscar nesse top 10 em que o público gostou muito mais que os críticos. 

Vamos ver como funciona no caso inverso.



```python
df.sort_values(by='diff',ascending=False).filter(items=['title','rotten_critic','rotten_people','diff','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>diff</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>115</th>
      <td>The Tree of Life</td>
      <td>84</td>
      <td>60</td>
      <td>24</td>
      <td>2012</td>
      <td>False</td>
    </tr>
    <tr>
      <th>96</th>
      <td>A Serious Man</td>
      <td>89</td>
      <td>67</td>
      <td>22</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Michael Clayton</td>
      <td>90</td>
      <td>69</td>
      <td>21</td>
      <td>2008</td>
      <td>False</td>
    </tr>
    <tr>
      <th>43</th>
      <td>The Queen (2006)</td>
      <td>97</td>
      <td>76</td>
      <td>21</td>
      <td>2007</td>
      <td>False</td>
    </tr>
    <tr>
      <th>84</th>
      <td>The Kids Are All Right</td>
      <td>93</td>
      <td>73</td>
      <td>20</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>62</th>
      <td>American Hustle</td>
      <td>93</td>
      <td>74</td>
      <td>19</td>
      <td>2014</td>
      <td>False</td>
    </tr>
    <tr>
      <th>88</th>
      <td>Winter's Bone</td>
      <td>94</td>
      <td>76</td>
      <td>18</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Sideways</td>
      <td>96</td>
      <td>78</td>
      <td>18</td>
      <td>2005</td>
      <td>False</td>
    </tr>
    <tr>
      <th>56</th>
      <td>Boyhood</td>
      <td>98</td>
      <td>81</td>
      <td>17</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Hugo</td>
      <td>94</td>
      <td>78</td>
      <td>16</td>
      <td>2012</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



A maior diferença ficou com <a href="https://www.rottentomatoes.com/m/the_tree_of_life_2011" target="_blank">Árvore da Vida</a>, que não ganhou o Oscar mas merecia o prêmio "Fui ver porque tinha o Brad Pitt". E novamente podemos perceber que quando os críticos **gostam mas o público não a lista gerada não possui nenhum ganhador**. 

Vamos fazer essa lista apenas para os filmes desse ano de 2017?? 

Então vamos listar aqui pela maior diferença entre público e críticos.


```python
print('Média dos criticos: ' + str(df[df["oscar_year"]== 2017]['rotten_critic'].mean()))
print('Média do publico: ' + str(df[df["oscar_year"]== 2017]['rotten_people'].mean()))
print('\n')
print('Mediana dos criticos: ' + str(df[df["oscar_year"]== 2017]['rotten_people'].median()))
print('Mediana do publico: ' + str(df[df["oscar_year"]== 2017]['rotten_critic'].median()))
print('\n')
print('Média da diferença: ' + str(df[df["oscar_year"]== 2017]['diff'].mean()))
print('Mediana da diferença: ' + str(df[df["oscar_year"]== 2017]['diff'].median()))
df[df["oscar_year"]== 2017].sort_values(by='diff',ascending=True).filter(items=['title','rotten_critic','rotten_people','diff','oscar_year','winner']).head(10)
```

    Média dos criticos: 93.4444444444
    Média do publico: 87.5555555556
    
    
    Mediana dos criticos: 88.0
    Mediana do publico: 94.0
    
    
    Média da diferença: 5.88888888889
    Mediana da diferença: 9.0





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>diff</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Hacksaw Ridge</td>
      <td>86</td>
      <td>94</td>
      <td>-8</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Lion</td>
      <td>88</td>
      <td>93</td>
      <td>-5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hidden Figures</td>
      <td>93</td>
      <td>94</td>
      <td>-1</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>5</th>
      <td>La la land</td>
      <td>93</td>
      <td>86</td>
      <td>7</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hell or High Water</td>
      <td>98</td>
      <td>89</td>
      <td>9</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Moonlight</td>
      <td>98</td>
      <td>88</td>
      <td>10</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Arrival</td>
      <td>94</td>
      <td>83</td>
      <td>11</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Fences</td>
      <td>95</td>
      <td>80</td>
      <td>15</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Manchester by the sea</td>
      <td>96</td>
      <td>81</td>
      <td>15</td>
      <td>2017</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Por tudo que já vimos até agora nessa análise: <a href="https://www.rottentomatoes.com/m/manchester_by_the_sea" target="_blank">CManchester by the sea</a> e <a href="https://www.rottentomatoes.com/m/moonlight_2016" target="_blank">Moonlight</a> que eram considerados fortes concorrentes estão com chances menores. Já <a href="https://www.rottentomatoes.com/m/la_la_land" target="_blank">La la land</a> que é o forte favorito ganha forças apesar de estar em 5 lugar em nota do público. 

Agora vamos fazer uma análise da média entre a nota dos críticos e do público e ver qual a correlação desse valor com a vitória do oscar?



```python
df['mean_rotten'] = df['rotten_critic'].add(df['rotten_people']).div(2)

df.sort_values(by='mean_rotten',ascending=False).filter(items=['title','rotten_critic','rotten_people','mean_rotten','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>mean_rotten</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>The Pianist</td>
      <td>96</td>
      <td>95</td>
      <td>95.5</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>17</th>
      <td>The Lord of the Rings: The Two Towers</td>
      <td>96</td>
      <td>95</td>
      <td>95.5</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Spotlight</td>
      <td>96</td>
      <td>93</td>
      <td>94.5</td>
      <td>2016</td>
      <td>True</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Whiplash</td>
      <td>94</td>
      <td>94</td>
      <td>94.0</td>
      <td>2015</td>
      <td>False</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Toy Story 3</td>
      <td>99</td>
      <td>89</td>
      <td>94.0</td>
      <td>2011</td>
      <td>False</td>
    </tr>
    <tr>
      <th>97</th>
      <td>up</td>
      <td>98</td>
      <td>90</td>
      <td>94.0</td>
      <td>2010</td>
      <td>False</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Room</td>
      <td>94</td>
      <td>93</td>
      <td>93.5</td>
      <td>2016</td>
      <td>False</td>
    </tr>
    <tr>
      <th>79</th>
      <td>The King's Speech</td>
      <td>95</td>
      <td>92</td>
      <td>93.5</td>
      <td>2011</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hell or High Water</td>
      <td>98</td>
      <td>89</td>
      <td>93.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hidden Figures</td>
      <td>93</td>
      <td>94</td>
      <td>93.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



O filme com a média de público e crítica mais alta foi <a href="https://www.rottentomatoes.com/m/pianist" target="_blank">O Pianista</a> (que deveria ter ganho em 2003), coincidentemente o segundo filme é também de 2003 e também não ganhou que foi <a href="https://www.rottentomatoes.com/m/the_lord_of_the_rings_the_two_towers" target="_blank">Senhor dos Anéis e as duas torres</a>. 

<div style="text-align: center;">  <img src="https://media.giphy.com/media/J3ZUjaOtahqM0/giphy.gif" alt="dog" style="width: 400px;"/> </div>
<center><p style="font-size:14px">Imagem retirada diretamente do filme "O Pianista"</a></center>

Aparecem 2 filmes ganhadores de Oscar nesse top 10, dois que foram bem "disputados" em seus respectivos anos.

Onde será que está <a href="https://www.rottentomatoes.com/m/chicago" target="_blank">Chicago</a> (ganhador de 2003) com relação os filmes do mesmo ano?



```python
df[df["oscar_year"]== 2003].sort_values(by='mean_rotten',ascending=False).filter(items=['title','rotten_critic','rotten_people','mean_rotten','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>mean_rotten</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>The Lord of the Rings: The Two Towers</td>
      <td>96</td>
      <td>95</td>
      <td>95.5</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>18</th>
      <td>The Pianist</td>
      <td>96</td>
      <td>95</td>
      <td>95.5</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Chicago</td>
      <td>86</td>
      <td>83</td>
      <td>84.5</td>
      <td>2003</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>The Hours</td>
      <td>81</td>
      <td>84</td>
      <td>82.5</td>
      <td>2003</td>
      <td>False</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Gangs of New York</td>
      <td>75</td>
      <td>81</td>
      <td>78.0</td>
      <td>2003</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Chicago aparece em 3 lugar, com 10 pontos de distância para <a href="https://www.rottentomatoes.com/m/pianist" target="_blank">O Pianista</a> e <a href="https://www.rottentomatoes.com/m/the_lord_of_the_rings_the_two_towers" target="_blank">"Senhor dos Anéis e as duas torres"</a>. Será que foi uma zebra?

Vamos ver como fica essa mesma tabela com os filmes de 2017?


```python
df[df["oscar_year"]== 2017].sort_values(by='mean_rotten',ascending=False).filter(items=['title','rotten_critic','rotten_people','mean_rotten','oscar_year','winner']).head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>rotten_critic</th>
      <th>rotten_people</th>
      <th>mean_rotten</th>
      <th>oscar_year</th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>Hell or High Water</td>
      <td>98</td>
      <td>89</td>
      <td>93.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hidden Figures</td>
      <td>93</td>
      <td>94</td>
      <td>93.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Moonlight</td>
      <td>98</td>
      <td>88</td>
      <td>93.0</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Lion</td>
      <td>88</td>
      <td>93</td>
      <td>90.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Hacksaw Ridge</td>
      <td>86</td>
      <td>94</td>
      <td>90.0</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>5</th>
      <td>La la land</td>
      <td>93</td>
      <td>86</td>
      <td>89.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Arrival</td>
      <td>94</td>
      <td>83</td>
      <td>88.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Manchester by the sea</td>
      <td>96</td>
      <td>81</td>
      <td>88.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Fences</td>
      <td>95</td>
      <td>80</td>
      <td>87.5</td>
      <td>2017</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>



Primeiramente podemos perceber que a diferença entre os filmes é bem menor do que em 2003. De <a href="https://www.rottentomatoes.com/m/hell_or_high_water" target="_blank">Hell or High Water</a> e <a href="https://www.rottentomatoes.com/m/hidden_figures" target="_blank">Hidden Figures</a> para <a href="https://www.rottentomatoes.com/m/la_la_land" target="_blank">La la land</a> temos 4 pontos de diferença apenas.


```python
df['rotten_position'] = 0.0

# Lets get each year 
years = sorted(df['oscar_year'].unique())
years.remove(2017)

analysis_by_year = [] 
# we filter the dataframe by the year and sort by the critics rotten
for year in years:
    result = df[df['oscar_year'] == year].sort_values(by='rotten_people',ascending=False).filter(items=['title', 'rotten_critic', 'rotten_people','oscar_year','winner',])
    
    analysis = {}
    analysis['year'] = year
    analysis['mean'] = result['rotten_people'].mean()
    analysis['median'] = result['rotten_people'].median()    
    
    #print('Analisando ano %s'%year)
    #now we get the average and the median for this year
    #print('Média das notas do publico: %s'%analysis['mean'])
    #print('Mediana das notas do publico: %s'%analysis['median'])
    
    i = 0
    for movie in result.iterrows():        
        if(movie[1]['winner']):
        #print('O ganhador "{}" do oscar estava na posição {} com o score de {}'.format(movie[1]['title'],i,movie[1]['rotten_people']))                        
            analysis['position'] = i 
            analysis['winner'] = movie[1]['title']
            
        index = df[df['title'] == movie[1]["title"]].index.tolist()[0] 
        df.set_value(index, 'rotten_position', (result.shape[0]-i)/float(result.shape[0]))
        i += 1
            
    analysis_by_year.append(analysis)
    #print('\n')   
    
df_by_year = pd.DataFrame(analysis_by_year)
df_by_year.head(20)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
      <th>median</th>
      <th>position</th>
      <th>winner</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>84.200000</td>
      <td>85.0</td>
      <td>0</td>
      <td>Gladiator</td>
      <td>2001</td>
    </tr>
    <tr>
      <th>1</th>
      <td>87.200000</td>
      <td>89.0</td>
      <td>1</td>
      <td>A Beautiful Mind</td>
      <td>2002</td>
    </tr>
    <tr>
      <th>2</th>
      <td>87.600000</td>
      <td>84.0</td>
      <td>3</td>
      <td>Chicago</td>
      <td>2003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>83.400000</td>
      <td>86.0</td>
      <td>1</td>
      <td>The Lord of the Rings: The Return of the King</td>
      <td>2004</td>
    </tr>
    <tr>
      <th>4</th>
      <td>84.200000</td>
      <td>87.0</td>
      <td>0</td>
      <td>Million Dollar Baby</td>
      <td>2005</td>
    </tr>
    <tr>
      <th>5</th>
      <td>83.400000</td>
      <td>83.0</td>
      <td>0</td>
      <td>Crash</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>6</th>
      <td>84.800000</td>
      <td>86.0</td>
      <td>0</td>
      <td>The Departed</td>
      <td>2007</td>
    </tr>
    <tr>
      <th>7</th>
      <td>81.800000</td>
      <td>86.0</td>
      <td>1</td>
      <td>No Country for Old Men</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>8</th>
      <td>85.200000</td>
      <td>88.0</td>
      <td>0</td>
      <td>Slumdog Millionaire</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>9</th>
      <td>81.700000</td>
      <td>82.0</td>
      <td>3</td>
      <td>The Hurt Locker</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>10</th>
      <td>85.000000</td>
      <td>85.5</td>
      <td>0</td>
      <td>The King's Speech</td>
      <td>2011</td>
    </tr>
    <tr>
      <th>11</th>
      <td>77.444444</td>
      <td>79.0</td>
      <td>1</td>
      <td>The Artist</td>
      <td>2012</td>
    </tr>
    <tr>
      <th>12</th>
      <td>84.222222</td>
      <td>82.0</td>
      <td>2</td>
      <td>Argo</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>13</th>
      <td>84.444444</td>
      <td>83.0</td>
      <td>1</td>
      <td>12 Years a Slave</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>14</th>
      <td>85.375000</td>
      <td>85.0</td>
      <td>7</td>
      <td>Birdman</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>15</th>
      <td>88.625000</td>
      <td>87.5</td>
      <td>1</td>
      <td>Spotlight</td>
      <td>2016</td>
    </tr>
  </tbody>
</table>
</div>



Parece que a comparação dos scores do públicos em um dado ano pode ser um fator relevante para encontrarmos o possível vencedor, grande parte dos ganhadores está na posição 0 ou 1. A maior exceção é <a href="https://www.rottentomatoes.com/m/birdman_2014" target="_blank">Birdman</a> que estava em 7 lugar no ano de 2015!


Para normalizar a posição dos filmes, dado que não é um número fixo de concorrentes, criamos a coluna "rotten_position" que vai de 0.0 a 1.0, sendo 1.0 o filme que está em primeiro lugar em termos de Rotten Tomatoes no ano e 0.0 é o último da lista.

Vamos ver como fica essa feature para os ganhadores do Oscar.


```python
df[df["oscar_year"]!= 2017].filter(items=['title','winner','rotten_position','oscar_year']).sort_values(by='winner',ascending=False).head(16)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>winner</th>
      <th>rotten_position</th>
      <th>oscar_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>Slumdog Millionaire</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Spotlight</td>
      <td>True</td>
      <td>0.875000</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>44</th>
      <td>The Lord of the Rings: The Return of the King</td>
      <td>True</td>
      <td>0.800000</td>
      <td>2004</td>
    </tr>
    <tr>
      <th>39</th>
      <td>The Departed</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2007</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Crash</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Million Dollar Baby</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2005</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Argo</td>
      <td>True</td>
      <td>0.777778</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Gladiator</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2001</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Birdman</td>
      <td>True</td>
      <td>0.125000</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>24</th>
      <td>No Country for Old Men</td>
      <td>True</td>
      <td>0.800000</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>79</th>
      <td>The King's Speech</td>
      <td>True</td>
      <td>1.000000</td>
      <td>2011</td>
    </tr>
    <tr>
      <th>89</th>
      <td>The Hurt Locker</td>
      <td>True</td>
      <td>0.700000</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Chicago</td>
      <td>True</td>
      <td>0.400000</td>
      <td>2003</td>
    </tr>
    <tr>
      <th>19</th>
      <td>A Beautiful Mind</td>
      <td>True</td>
      <td>0.800000</td>
      <td>2002</td>
    </tr>
    <tr>
      <th>70</th>
      <td>12 Years a Slave</td>
      <td>True</td>
      <td>0.888889</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>108</th>
      <td>The Artist</td>
      <td>True</td>
      <td>0.888889</td>
      <td>2012</td>
    </tr>
  </tbody>
</table>
</div>



## Correlação

Vamos agora analisar a correlação dessas features com relação aos ganhadores?

Vamos remover o ano de 2017 pois não temos o resultado desse ano ainda.

Para isso vamos usar a função <a href="http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.corr." target="_blank">corr()</a> do Dataframe e selecionar apenas a coluna "winner" (Essa função retorna a correlação entre todas colunas com todas as outras).



```python
df[df["oscar_year"]!= 2017].corr(method='spearman', min_periods=1).filter(items=['winner']).sort_values(by='winner',ascending=False)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>winner</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>winner</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>dga_winner</th>
      <td>0.706522</td>
    </tr>
    <tr>
      <th>pga_winner</th>
      <td>0.661557</td>
    </tr>
    <tr>
      <th>sag_winner</th>
      <td>0.486413</td>
    </tr>
    <tr>
      <th>bafta_winner</th>
      <td>0.435455</td>
    </tr>
    <tr>
      <th>bafta_winners</th>
      <td>0.435455</td>
    </tr>
    <tr>
      <th>rotten_position</th>
      <td>0.352077</td>
    </tr>
    <tr>
      <th>user_rating</th>
      <td>0.291143</td>
    </tr>
    <tr>
      <th>rotten_people</th>
      <td>0.288959</td>
    </tr>
    <tr>
      <th>wga_winner</th>
      <td>0.278639</td>
    </tr>
    <tr>
      <th>nyfcc_winner</th>
      <td>0.226465</td>
    </tr>
    <tr>
      <th>mean_rotten</th>
      <td>0.222958</td>
    </tr>
    <tr>
      <th>golden_globe</th>
      <td>0.203698</td>
    </tr>
    <tr>
      <th>rotten_critic</th>
      <td>0.124818</td>
    </tr>
    <tr>
      <th>budget</th>
      <td>-0.028455</td>
    </tr>
    <tr>
      <th>_id</th>
      <td>-0.058523</td>
    </tr>
    <tr>
      <th>diff</th>
      <td>-0.070719</td>
    </tr>
    <tr>
      <th>opening_weekend</th>
      <td>-0.080572</td>
    </tr>
    <tr>
      <th>oscar_year</th>
      <td>-0.097222</td>
    </tr>
  </tbody>
</table>
</div>



Como era esperado o DGA e o PGA são as premiações mais correlacionadas com os ganhadores do Oscar. E é importante salientar que os Golden Globes estão com uma correlação baixíssima.

Das novas features adicionadas parece que o rotten position relativo ao score do público é a mais correlacionada, sendo a melhor feature depois dos principais prêmios, mas mesmo assim 0.35 é um valor baixo. Mas percebam também que a posição no ano em questão (rotten_position) é melhor do que o valor bruto do score (rotten_people).

Outra observação é que a correlação do score do público é muito próxima a nota no IMDB, apesar dos dois usarem um sistema de rating um pouco diferente.

## Conclusão

Gostaria de agradecer a todos que duraram até aqui, depois de tantas tabelas e **ótimos** gifs animados. Queria brincar um pouco com esses dados do Rotten Tomatoes e espero que esse post ajude algumas pessoas a entender um pouco mais sobre análise de dados.

No próximo post vamos tentar novamente "adivinhar" o ganhador do Oscar (*La la land... Cof Cof*) desse ano de 2017, agora adicionando essas novas features =D

Abraços, bons filmes (plágio) e até a próxima

<div style="text-align: center;"> <img src="https://media.giphy.com/media/l41lQLVaDm8he8Gk0/giphy.gif" alt="paul" style="width: 400px;"/> </div>
