---
layout: post
title: "Brincando com dados: Vencedores do Oscar Parte 2"
date:       2016-02-11 19:44:19
summary: Na parte 2 vamos analisar os dados dos indicados ao prêmio de melhor filme dos últimos 15 anos e tentar prever o ganhador do ano de 2016.
categories:
  - data-science
  - brincando-com-dados
tags:
  - python
  - imdb
  - data science
  - dados
  - mongodb
  - pymongo
  - IMDbPy
  - oscar
  - movie
  - filme
  - Ipython Notebook
  
author: Luiz Felipe Mendes
authurl: https://twitter.com/lfmendes
---

![Oscar Winner](https://media.giphy.com/media/rD0tbQKCxQURq/giphy.gif "It is now or never")


Olá a todos, esse post é a segunda parte de um conjunto de posts intítulados "Brincando com os dados". A <a href="http://developers.hekima.com/data-science/brincando-com-dados/2016/02/11/importing_oscar_runners/" target="_blank">primeira parte</a> foi sobre como trabalhar com pymongo e fazer o carregamento dos filmes indicados a melhor filme do Oscar.

Esse segundo post será sobre análise de dados, vamos pegar os dados salvos na primeira parte e verificar se existem atributos que indicam que um filme será o vencedor ou não. Ao final dessa postagem iremos tentar prever qual será o **ganhador do Oscar de 2016**. 

Novamente, vamos trabalhar com <a href="https://api.mongodb.org/python/current/" target="_blank">PyMongo</a> para buscar os dados no <a href="https://www.mongodb.org/" target="_blank">MongoDB</a>. Para visualizar os dados vamos utilizar uma biblioteca de Machine Learning chamada <a href="https://dato.com/products/create/" target="_blank">GraphLab create</a> da empresa <a href="https://dato.com" target="_blank">Dato</a>. 

De forma que o post fique mais simples em termos de código, estou usando aqui o html gerado pelo próprio IPython Notebook, então os comentários e o código estão entrelaçados.

## Preparativos
Para ficar mais simples e didático decidimos utilizar o IPython Notebook. Para fazer a instalação dele siga o tutorial descrito aqui:  <a href="http://jupyter.readthedocs.org/en/latest/install.html" target="_blank">http://jupyter.readthedocs.org/en/latest/install.html</a>   

Para rodar o código ou acompanhar o post pelo IPython Notebook baixe o arquivo: <a href="https://s3.amazonaws.com/zahpee-public/ImportingOscarRunners.ipynb" target="_blank">ImportingOscarRunners.ipynb</a> 

## Base
**MongoDB**: O MongoDB é um banco de dados não relacional, ele é orientado a documentos e utiliza um formato chamado BSON, bem próximo ao JSON. Ele é bem simples de entender e utilizar.

**GraphLab Create**: É uma biblioteca de machine learning.

**PyMongo**: PyMongo é uma biblioteca em python para acessar o MongoDB.


```python

import graphlab
import json
from pymongo import MongoClient


client = MongoClient() # Primeiramente pegamos o client do Mongodb
db = client.imdb 
movie_db = db.movies
movie_cursor = movie_db.find()

i = 0
movies = []
for document in movie_cursor: 
    movies.append(document)    
```

## Carregamento dos dados

Inicialmente, criamos um cliente de mongodb e utilizamos a variável "db" para referenciar a database "imdb" e para fazer operações nos filmes da coleção "movies" utilizamos a variável movie_db.

Criamos um cursor utilizando uma consulta vazia, isto é, não estamos filtrando nada, todos filmes serão retornados. Logo depois, iteramos pelo cursor adicionando os documentos retornados na lista "movies"


```python

sf = graphlab.SFrame(movies)    
movies_sframe = sf.unpack('X1', column_name_prefix="")

```

A biblioteca GraphLab normalmente utiliza objetos <a href="https://dato.com/products/create/docs/generated/graphlab.SFrame.html" target="_blank">SFrame</a> para representar seus dados, nesse caso utilizamos a construtora que recebe uma lista e utilizamos a função unpack para transformar cada entrada da lista, que é um dicionário no formato do SFrame.


```python
# looking at the data, only the 4 first results
movies_sframe.head(4)
```




<div style="max-height:1000px;max-width:1500px;overflow:auto;font-size: 0.5em;"><table frame="box" rules="cols">
    <tr>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">_id</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">assistant director</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">budget</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">cast</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">casting director</th>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">1790885</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Tarik Afifi',<br>'id': '3199917'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">40000000</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Jason Clarke',<br>'id': '0164809'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Mark Bennett',<br>'id': '0071916'}, ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">1655442</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'James Canal',<br>'id': '0133511'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">15000000</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Jean<br>Dujardin', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Heidi Levitt',<br>'id': '0506217'}] ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">2505072</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'John<br>Bonaccorse', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">0</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Martin<br>Henderson', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Libby<br>Goldstein', 'id': ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">1454029</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Karen Davis',<br>'id': '0204929'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">25000000</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Emma Stone',<br>'id': '1297015'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Kerry Barden',<br>'id': '0054234'}, ...</td>
    </tr>
</table>
<table frame="box" rules="cols">
    <tr>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">countries</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">director</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">distributors</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">editor</th>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[USA]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Kathryn<br>Bigelow', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'A-Film<br>Distribution', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'William<br>Goldenberg', 'id': ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[France]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Michel<br>Hazanavicius', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name':<br>'Cin\xc3\xa9art', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Anne-Sophie<br>Bion', 'id': '2691134'}, ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[USA]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Randall<br>Einhorn', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'SundanceTV',<br>'id': '0490233'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Kristina<br>Boden', 'id': '009086 ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[USA, India, United Arab<br>Emirates] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Tate Taylor',<br>'id': '0853238'}] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Touchstone<br>Pictures', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Hughes<br>Winborne', 'id': ...</td>
    </tr>
</table>
<table frame="box" rules="cols">
    <tr>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">genres</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">languages</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">opening_weekend</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">original music</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">oscar_year</th>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Drama, History,<br>Thriller] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[English, Arabic]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">417150</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Alexandre<br>Desplat', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">2013</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Comedy, Drama, Romance]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[English, English,<br>French] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">204878</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Ludovic<br>Bource', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">2012</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Drama]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[English]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">0</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Daniel Licht',<br>'id': '0006171'}] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">2012</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Drama]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[English]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">26044590</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Thomas<br>Newman', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">2012</td>
    </tr>
</table>
<table frame="box" rules="cols">
    <tr>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">plot</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">production companies</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">runtime</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">special effects companies</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">title</th>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Maya is a CIA operative<br>whose first experienc ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Columbia<br>Pictures', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[157]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Image Engine<br>Design', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">Zero Dark Thirty</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Outside a movie<br>premiere, enthusiastic ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Studio 37',<br>'id': '0210188'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[100]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Digital<br>District', 'id': ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">The Artist</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Revolves around a local<br>cop struggling to keep ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[60]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">The Descendants</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[Set in Mississippi<br>during the 1960s, Ske ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'DreamWorks<br>SKG', 'id': '0040938'}, ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[USA:146]</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Pixel Magic',<br>'id': '0065379'}] ...</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">The Help</td>
    </tr>
</table>
<table frame="box" rules="cols">
    <tr>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">user_rating</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">winner</th>
        <th style="padding-left: 1em; padding-right: 1em; text-align: center">writer</th>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">7.4</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">None</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Mark Boal',<br>'id': '1676793'}] ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">8.0</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">1</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Michel<br>Hazanavicius', 'id': ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">7.4</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">None</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Aaron<br>Guzikowski', 'id': ...</td>
    </tr>
    <tr>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">8.1</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">None</td>
        <td style="padding-left: 1em; padding-right: 1em; text-align: center; vertical-align: top">[{'name': 'Tate Taylor',<br>'id': '0853238'}, ...</td>
    </tr>
</table>
[4 rows x 22 columns]<br/>
</div>




A função head() do SFrame mostra as primeiras linhas e o cabeçalho dessa "matriz". É importante dar uma olhada para saber qual tipo de dados estamos lidando, se o formato e tipo estão corretos.

# Visualizando Dados


```python
# now we set the target to this window (ipython notebook)
graphlab.canvas.set_target('ipynb')
# we use the show function, this function show some metrics about each feature
movies_sframe.show(view="Summary")
```

![Oscar Winner](https://s3.amazonaws.com/zahpee-public/summary1.png "Sumário")


O Graphlab tem um sistema de visualizações bem interessante e simples, para visualizar direamente no python notebook utilizamos a função graphlab.canvas.set_target("ipynb"). Depois, para ver uma resumo dos dados de um SFrame podemos utilizar a função show()*. 

*No caso, quando exporto o arquivo ele não gera a visualização, então estou adicionando aqui uma imagem para mostrar como os dados são mostrados.

A função show() gera uma tabela com todas as colunas da matriz, gerando informações bem simples mas interesantes, como: 

* Tipo
* Número de valores únicos
* Número de entradas com esse coluna vazia ou undefined
* Valor máximo dessa coluna
* Valor mínimo dessa coluna
* Mediana 
* Média
* Desvio padrão
* Itens mais frequentes
  
Para colunas numéricas ele mostra um gráfico de distribuição.  


```python
# now we take only the Oscar winners
oscar_winners = movies_sframe[movies_sframe['winner'] == 1]
oscar_winners.show(view="Summary")
```

![Oscar Winner](https://s3.amazonaws.com/zahpee-public/summary2.png "Sumário")


Agora vamos utilizar outra funcionalidade muito legal do SFrame. É possível utilizar um filtro, retornando outro SFrame em que os valores respeitam a expressão boleana.

Nesse caso, utilizamos a expressão boleana "movies_sframe['winner'] == 1" para filtrar apenas os vencedores do Oscar, logo depois, utilizamos denovo a função show(), agora nesse novo SFrame.

Analisando um pouco esse sumário dos filmes vencedores do oscar podemos perceber que filmes de Drama dominam, além disso, é interessante analisar um pouco as notas no imdb: A média e a mediana ficaram em 8, o filme de menor nota teve 7.2. Qual será que foi esse filme? Vamos verificar.


```python

print "Título: " + oscar_winners[oscar_winners['user_rating'] == 7.2]['title'][0]
print "Ano: " + str(oscar_winners[oscar_winners['user_rating'] == 7.2]['oscar_year'][0])

```

    Título: Chicago
    Ano: 2003


Utilizamos denovo o filtro, agora buscando qual o primeiro filme com valor de "user_rating" igual a 7.2, No caso, foi o filme Chicago do Oscar de 2003. 
Vamos agora ver o filme de maior avaliação dos vencedores.


```python

print "Título: " + oscar_winners[oscar_winners['user_rating'] == 8.9]['title'][0]
print "Ano: " + str(oscar_winners[oscar_winners['user_rating'] == 8.9]['oscar_year'][0])

```

    Título: The Lord of the Rings: The Return of the King
    Ano: 2004


![Frodo](https://s3.amazonaws.com/zahpee-public/frodo.jpg "Frodo Felizão")


Como nosso querido Frodo pode mostrar com seu sorriso, o filme de maior nota que levou a estatueta foi "Senhor dos Anéis. Retorno do Rei".

Vamos agora analisar algumas features dos vencedores do oscar e comparar com todos os filmes que concorreram mas NÃO ganharam. Para isso vamos usar a função show() relativo ao tipo <a href="https://dato.com/products/create/docs/generated/graphlab.SArray.html?highlight=sarray" target="_blank">SArray</a>. No caso o show padrão mostra dados como mínimo, máximo, média e um gráfico com os valores.
Vamos visualizar então agora os ratings dos concorrentes e depois dos vencedores.


```python

graphlab.canvas.set_target('ipynb')
oscar_losers = movies_sframe[movies_sframe['winner'] == None]
oscar_losers['user_rating'].show()
oscar_winners['user_rating'].show()

```

![Barras](https://s3.amazonaws.com/zahpee-public/barras.png "Barras")



Existe uma pequena diferença entre todos os filmes que não ganharam com os que levaram a estatueta, aparentemente os vencedores do oscar possuem uma nota mais alta do que os que não ganharam. ( média e mediana maiores no segundo caso)  
Agora vamos comparar o tempo de duração dos filmes que concorreram com os que ganharam. Mas antes disso, precisamos trabalhar nos dados, alguns tempos estão em formato string do tipo "Extended cut: 130". 


```python

import re
p = re.compile('([0-9]+)')
runtime_list = []
#first we will transform every runtime into a number
for movie in movies_sframe: 
    match = p.search(movie['runtime'][0])    
    if len(match.groups()) > 0:
        runtime_list.append(int(match.group(0)))
    else:
        runtime_list.append(0)
        
movies_sframe.add_column(graphlab.SArray(runtime_list),name='runtime_oficial') 

print movies_sframe['runtime_oficial'].show()

```

![Barras](https://s3.amazonaws.com/zahpee-public/barras2.png "Barras")


Nos criamos um lista com o resultado de um regex simples que busca apenas os números da duração (em minutos). Logo depois, adicionamos ao nosso SFrame uma coluna nomeada runtime_oficial e mostramos o resultado dele, percebe-se que todos os valores fazem sentido.


```python
# first lets fill the winner field with 0 when None
movies_sframe['winner'] = movies_sframe['winner'].fillna(0)
movies_sframe['budget'] = movies_sframe['budget'].fillna(0)
movies_sframe['opening_weekend'] = movies_sframe['opening_weekend'].fillna(0)

#print movies_sframe[movies_sframe['budget'] == 0]["title"]
#print movies_sframe[movies_sframe['opening_weekend'] == 0]["title"]
movies_sframe['budget'] = movies_sframe.apply(lambda x: 35000000 if x['title'] == 'The Descendants' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 7500000 if x['title'] == 'An Education (2009)' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 40000000 if x['title'] == 'Extremely Loud & Incredibly Close' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 15000000 if x['title'] == 'The Queen (2006)' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 30000000 if x['title'] == 'The Grand Budapest Hotel' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 6000000 if x['title'] == 'Room' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 10000000 if x['title'] == 'Brooklyn' else x['budget'])
movies_sframe['budget'] = movies_sframe.apply(lambda x: 20000000 if x['title'] == 'Spotlight' else x['budget'])

movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 15371203   if x['title'] == 'Bridge of Spies' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 11201500  if x['title'] == 'Room' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 1190096 if x['title'] == 'The Descendants' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 159017 if x['title'] == 'An Education (2009)' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 68266 if x['title'] == 'Amour' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 72348 if x['title'] == 'Extremely Loud & Incredibly Close' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 208763 if x['title'] == 'The Theory of Everything' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 135388 if x['title'] == 'Whiplash' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 260382 if x['title'] == 'Her (2013)' else x['opening_weekend'])
movies_sframe['opening_weekend'] = movies_sframe.apply(lambda x: 128435 if x['title'] == 'Philomena' else x['opening_weekend'])

# now we create another column with only on genre
movies_sframe['main_genre'] = movies_sframe.apply(lambda x: str(x['genres'][0]))
```

Nessa parte do código estamos preenchendo com 0 os campos inexistentes usando a função fillna(0) e preenchendo os campos de budget e openining_weekend de alguns filmes que não possuiam.

Na próxima etapa, vamos adicionar novos campos ao nosso SFrame, para cada um dos prêmios escolhidos vamos adicionar um campos que valerá 1 se o filme ganhou aquela premiação e 0 se não.


```python

# Now we are going to add the Sag Winners
sag_winners = ["Spotlight","Birdman", "American Hustle", "Argo", "The Help", "The King's Speech", 
               "Inglorious Bastards", "Slumdog Millionaire", "No Country for Old Men", "Little Miss Sunshine (2006)",
               "Crash", "Sideways", "Chicago", "The Lord of the Rings: The Return of the King", "Gosford Park", "Traffic"]

golden_globe = ["The Martian","The Revenant","Boyhood", "The Grand Budapest Hotel", "12 Years a Slave", "American Hustle", "Argo", "Les Misérables", 
               "The Descendants", "The Artist", "The Social Network", "The Kids Are All Right", "Avatar", "The Hangover", 
               "Slumdog Millionaire", "Vicky Cristina Barcelona", "Atonement", "Sweeney Todd", "Babel", "Dreamgirls", 
               "Brokeback Mountain", "Walk the Line", "The Aviator", "Sideways", "The Lord of the Rings: The Return of the King",
               "Lost in Translation", "The Hours", "Chicago", "A Beaultiful Mind", "Moulin Rouge!", "Gladiator", "Almost Famous"]

nyfcc_winners = ["Carol","Son of Saul","Boyhood", "Zero Dark Thirty", "The Artist", "The Social Network",
                 "The Hurt Locker", "Slumdog Millionaire","There Will Be Blood", "United 93", 
                 "Sideways", "Brokeback Mountain", "The Lord of the Rings: The Return of the King",
                "Far from Heaven", "Mulholland Drive"]

dga_winners = ["The Revenant","Birdman","Gravity", "Argo", "The Artist", "The King's Speech", "The Hurt Locker",
               "Slumdog Millionaire", 
               "No Country for Old Men","The Departed", "Brokeback Mountain", "Million Dollar Baby", 
               "The Lord of the Rings: The Return of the King",  "Chicago",
               "A Beautiful Mind", "Crouching Tiger, Hidden Dragon"]

wga_winners = ["The Imitation Game","The Grand Budapest Hotel", "Captain Phillips", "Her (2013)", "Argo", "Zero Dark Thirty",
               "The Descendants", 
               "Inception","The Social Network", "The Hurt Locker", "Up in the Air", "Milk", "Juno",
               "Slumdog Millionaire", "No Country for Old Men",  "The Departed",
               "Little Miss Sunshine (2006)","Brokeback Mountain", "Crash", "Sideways", "Eternal Sunshine of the Spotless Mind",
               "American Splendor", "Lost in Translations", "Bowling for Columbine", "The Hours", "A Beautiful Mind", "Gosford Park"
               , "Traffic", "You Can Count on Me"]

pga_winners = ['The Big Short', "Birdman","12 Years a Slave", "Argo", "The Artist", "The King's Speech", 
               "The Hurt Locker", "Slumdog Millionaire", "No Country for Old Men", 
               "Little Miss Sunshine", "Brokeback Mountain", "The Aviator", "The Lord of the Rings: The Return of the King",
               "Chicago", "Moulin Rouge!", "Gladiator"]

movies_sframe['sag_winner'] = movies_sframe.apply(lambda x: 1 if x['title'] in sag_winners else 0)
movies_sframe['golden_globe'] = movies_sframe.apply(lambda x: 1 if x['title'] in golden_globe else 0) 
movies_sframe['nyfcc_winner'] = movies_sframe.apply(lambda x: 1 if x['title'] in nyfcc_winners else 0)
movies_sframe['dga_winner'] = movies_sframe.apply(lambda x: 1 if x['title'] in dga_winners else 0)
movies_sframe['wga_winner'] = movies_sframe.apply(lambda x: 1 if x['title'] in wga_winners else 0)
movies_sframe['pga_winner'] = movies_sframe.apply(lambda x: 1 if x['title'] in pga_winners else 0)

```

Utilizamos as premiações mais reconhecidas por serem "termômetros" do oscar e o Golden Globes.

Utilizamos nessa parte a função "apply()" do SFrame, ela aplica uma operação a cada elemento do SFrame, retornando um SFrame. Nesse caso, nós verificamos se o título do filme atual está na lista de vencedores de cada prêmio e adicionamos 1 se sim e 0 se não. 

Na pŕoxima etapa vamos analisar quantas vezes os vencedores coincidiram, isto é, quantas vezes cada premiação coincidiu com o Oscar.



```python

# now we check which prizes coincide more 
sag_oscar = movies_sframe.apply(lambda x: x['title'] if ((x['sag_winner'] * x['winner']) == 1) else 'none', dtype=str)
sag_oscar = sag_oscar.filter(lambda x: x != 'none')
print "Sag: " + str(len(sag_oscar))
                              
golden_oscar = movies_sframe.apply(lambda x: x['title'] if x['golden_globe'] * x['winner'] == 1 else None, dtype=str)
golden_oscar = golden_oscar.filter(lambda x: x != None)
print "Golden Globes: " + str(len(golden_oscar))
    
ny_oscar = movies_sframe.apply(lambda x: x['title'] if x['nyfcc_winner'] * x['winner'] == 1 else None, dtype=str)
ny_oscar = ny_oscar.filter(lambda x: x != None)
print "NYFCC: " + str(len(ny_oscar))

dga_oscar = movies_sframe.apply(lambda x: x['title'] if x['dga_winner'] * x['winner'] == 1 else None, dtype=str)
dga_oscar = dga_oscar.filter(lambda x: x != None)
print "DGA: " + str(len(dga_oscar))

wga_oscar = movies_sframe.apply(lambda x: x['title'] if x['wga_winner'] * x['winner'] == 1 else None, dtype=str)
wga_oscar = wga_oscar.filter(lambda x: x != None)
print "WGA: " + str(len(wga_oscar))

pga_oscar = movies_sframe.apply(lambda x: x['title'] if x['pga_winner'] * x['winner'] == 1 else None, dtype=str)
pga_oscar = pga_oscar.filter(lambda x: x != None)
print "PGA: " + str(len(pga_oscar))

bafta_oscar = movies_sframe.apply(lambda x: x['title'] if x['bafta_winner'] * x['winner'] == 1 else None, dtype=str)
bafta_oscar = bafta_oscar.filter(lambda x: x != None)
print "BAFTA: " + str(len(bafta_oscar))

```

    Sag: 8
    Golden Globes: 7
    NYFCC: 4
    DGA: 12
    WGA: 7
    PGA: 11
    BAFTA: 8


Podemos perceber que os dois melhores termômetros do Oscar são:

* DGA (Directors Guild of America Award): 12 ocasiões
* PGA (Producers Guild of America Award): 11 ocasiões

Os piores são:

* NYFCC: 4 ocasiões
* Golden Globes: 7 ocasiões (contando aqui melhor filme de drama E comédia e musical)

Agora vamos fazer um pequeno experimento, vamos usar os vencedores dessas premiações e alguns valores dos filmes para tentar prever a probabilidade de cada indicado ser o vencedor do Oscar.

Para os atributos, utilizei as premiações que deram o melhor resultado e alguns campos que já existem em nossa tabela.

Utilizamos o algoritmo de <a href="https://dato.com/products/create/docs/graphlab.toolkits.classifier.html#random-forest" target="_blank">random_forest_classifier</a>. 

Para verificar o resultado anotamos quantas vezes acertamos o ganhador na primeira tentativa ou se ele estava presente em nosso top 3.


```python

features = ["bafta_winner","pga_winner","dga_winner", "golden_globe", "sag_winner", "budget", "opening_weekend", "runtime_oficial", "user_rating"]

print 'Features:' + str(features)

years = range(2001,2016)
print years

top3_correct = 0
top1_correct = 0
for year in years:        
    train_data = movies_sframe[movies_sframe['oscar_year'] != year and movies_sframe['oscar_year'] != 2016]
    model = graphlab.random_forest_classifier.create(train_data, target='winner', features=features, verbose=False, num_trees=50)
    test_data = movies_sframe[movies_sframe['oscar_year'] == year]
    
    results = model.predict_topk(test_data, k=2)
    result_sorted = results[results['class'] == 1].sort("probability")
    #print result_sorted
    first = len(results[results['class'] == 1])-1
    second = len(results[results['class'] == 1])-2
    third = len(results[results['class'] == 1])-3
    
    top3 = []
    top3.append(test_data['title'][result_sorted[first]['id']])
    top3.append(test_data['title'][result_sorted[second]['id']])
    top3.append(test_data['title'][result_sorted[third]['id']])
    top1 = test_data['title'][result_sorted[first]['id']]
    
    
    if test_data[test_data['winner'] == 1][0]['title'] in top3:
        top3_correct = top3_correct + 1
    if test_data[test_data['winner'] == 1][0]['title'] == top1:
        top1_correct = top1_correct + 1        
    
    print '\n\nVencedor real:' + test_data[test_data['winner'] == 1][0]['title']
    print '----------'
    print 'Vencedor previsto: ' + test_data['title'][result_sorted[first]['id']]
    print 'Segundo previsto: ' + test_data['title'][result_sorted[second]['id']]
    print 'Terceito previsto: ' + test_data['title'][result_sorted[third]['id']]

print 'Acertos em 3: ' + str(top3_correct)
print 'Acertos em 1: ' + str(top1_correct)

print 'Acurácia em 3: ' + str(top3_correct/float(15))
print 'Acurácia: ' + str(top1_correct/float(15))
```

    Features:['pga_winner', 'dga_winner', 'golden_globe', 'sag_winner', 'budget', 'opening_weekend', 'runtime_oficial', 'user_rating']
    [2001, 2002, 2003, 2004, 2005, 2006, 2007, 2008, 2009, 2010, 2011, 2012, 2013, 2014, 2015]
    
    
    Vencedor real:Gladiator
    ----------
    Vencedor previsto: Gladiator
    Segundo previsto: Crouching Tiger, Hidden Dragon
    Terceito previsto: Traffic
    
    
    Vencedor real:A Beautiful Mind
    ----------
    Vencedor previsto: A Beautiful Mind
    Segundo previsto: Moulin Rouge!
    Terceito previsto: The Lord of the Rings: The Fellowship of the Ring
    
    
    Vencedor real:Chicago
    ----------
    Vencedor previsto: Chicago
    Segundo previsto: The Pianist
    Terceito previsto: The Lord of the Rings: The Two Towers
    
    
    Vencedor real:The Lord of the Rings: The Return of the King
    ----------
    Vencedor previsto: The Lord of the Rings: The Return of the King
    Segundo previsto: Mystic River
    Terceito previsto: Seabiscuit
    
    
    Vencedor real:Million Dollar Baby
    ----------
    Vencedor previsto: Million Dollar Baby
    Segundo previsto: The Aviator
    Terceito previsto: Sideways
    
    
    Vencedor real:Crash
    ----------
    Vencedor previsto: Brokeback Mountain
    Segundo previsto: Crash
    Terceito previsto: Munich
    
    
    Vencedor real:The Departed
    ----------
    Vencedor previsto: The Departed
    Segundo previsto: Little Miss Sunshine (2006)
    Terceito previsto: The Queen (2006)
    
    
    Vencedor real:No Country for Old Men
    ----------
    Vencedor previsto: No Country for Old Men
    Segundo previsto: There Will Be Blood
    Terceito previsto: Michael Clayton
    
    
    Vencedor real:Slumdog Millionaire
    ----------
    Vencedor previsto: Slumdog Millionaire
    Segundo previsto: The Reader
    Terceito previsto: Milk
    
    
    Vencedor real:The Hurt Locker
    ----------
    Vencedor previsto: The Hurt Locker
    Segundo previsto: Inglorious Bastards
    Terceito previsto: up
    
    
    Vencedor real:The King's Speech
    ----------
    Vencedor previsto: The King's Speech
    Segundo previsto: Inception
    Terceito previsto: Winter's Bone
    
    
    Vencedor real:The Artist
    ----------
    Vencedor previsto: The Artist
    Segundo previsto: The Help
    Terceito previsto: War Horse
    
    
    Vencedor real:Argo
    ----------
    Vencedor previsto: Argo
    Segundo previsto: Django Unchained
    Terceito previsto: Beasts of the Southern Wild
    
    
    Vencedor real:12 Years a Slave
    ----------
    Vencedor previsto: 12 Years a Slave
    Segundo previsto: Gravity
    Terceito previsto: The Wolf of Wall Street
    
    
    Vencedor real:Birdman
    ----------
    Vencedor previsto: Birdman
    Segundo previsto: Whiplash
    Terceito previsto: Imitation Game
    Acertos em 3: 15
    Acertos em 1: 14
    Acurácia em 3: 1.0
    Acurácia: 0.9333333


Considerando o top 3 acertamos 15 vezes das 15. Mas no caso de considerar apenas o filme de maior probabilidade foram 14 acertos de 15 possíveis. Sendo que, sinceramente, "Brokeback Mountain" deveria ter ganhado de "Crash", logo, considero que nós acertamos e o Oscar errou =P

Essa forma de avaliar um algoritmo é chamada de Leave-one-out que é praticamente um k-fold cross validation que o k=n, sendo n o número de itens a serem testados.


Agora vamos tentar adivinhar o ganhador do Oscar de 2016 =D


PS: Estou rodando com os atributos do dia 12 de fevereiro.






```python
train_data = movies_sframe[movies_sframe['oscar_year'] != 2016]
model = graphlab.random_forest_classifier.create(train_data, target='winner', features=features, verbose=False, num_trees=50)
test_data = movies_sframe[movies_sframe['oscar_year'] == 2016]

results = model.predict_topk(test_data, k=2)
result_sorted = results[results['class'] == 1].sort("probability")
#print result_sorted
first = len(results[results['class'] == 1])-1
second = len(results[results['class'] == 1])-2
third = len(results[results['class'] == 1])-3

top3 = []
top3.append(test_data['title'][result_sorted[first]['id']])
top3.append(test_data['title'][result_sorted[second]['id']])
top3.append(test_data['title'][result_sorted[third]['id']])
top1 = test_data['title'][result_sorted[first]['id']]

print '----------'
print 'Vencedor previsto: ' + test_data['title'][result_sorted[first]['id']]
print 'Segundo previsto: ' + test_data['title'][result_sorted[second]['id']]
print 'Terceito previsto: ' + test_data['title'][result_sorted[third]['id']]
```

    ----------
    Vencedor previsto: The Revenant
    Segundo previsto: The Big Short
    Terceito previsto: Spotlight


![Oscar](https://s3.amazonaws.com/zahpee-public/oscar.gif "OSCAAARRR")

Pelo nosso top 3 o filme com maior probabilidade de ganhar é **"O Regresso"** do nosso querido Leonardo DiCaprio, logo depois dele está o "A Grande Aposta" que ganhou o PGA.

Logicamente, isso foi só um pequeno exemplo, para mostrar algumas funcionalidades dessas ferramentas, para criar um classificador é necessário muitos exemplos, apenas 15 exemplos positivos é um número muito pequeno.

No próximo post, pretendo buscar os dados no MySQL e fazer um classificador mais interessante com mais exemplos, mostrando algumas técnicas de Machine Learning.

Abraços e até a próxima


