---
layout: post
title: "Brincando com dados: Ganhadores do Oscar Parte 1"
date:       2016-02-11 19:44:19
summary: O objetivo desses posts intitulados "Brincando com os dados" é dar exemplos de como importar, limpar e analisar dados. Nesse primeiro exemplo iremos importar no MongoDB os dados do IMDb dos filmes indicados a categoria de melhor filme do Oscar.
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

![Oscar Winner](https://ioneglobalgrind.files.wordpress.com/2014/02/tumblr_mnu5h5zgaf1rnjh5ho1_500.gif?w=600&h=510 "Will it ever happen?")

Olá a todos, esse post será o primeiro de muitos (como diria a minhoca do worms) sobre análise de dados. O objetivo desses posts intitulados “Brincando com os dados” é dar exemplos de como importar, limpar, analisar dados e muito mais.

O assunto desse primeiro exemplo vai ser “Ganhadores do Oscar” e o objetivo dele é ser bem simples e didático, mostrar alguns conceitos e exemplos de ferramentas e bibliotecas. 

A primeira parte será sobre como importar a base de dados. Irei mostrar como utilizar a biblioteca <a href="http://imdbpy.sourceforge.net" target="_blank">IMDbPY</a> para buscar filmes e o 
<a href="https://api.mongodb.org/python/current/" target="_blank">PyMongo</a> para salvar os dados no <a href="https://www.mongodb.org/" target="_blank">MongoDB</a>.

De forma que o post fique mais simples em termos de código, estou usando aqui o html gerado pelo próprio IPython Notebook, então os comentários e o código estão entrelaçados.

## Preparativos
Para ficar mais simples e didático decidimos utilizar o IPython Notebook. Para fazer a instalação dele siga o tutorial descrito aqui:  <a href="http://jupyter.readthedocs.org/en/latest/install.html" target="_blank">http://jupyter.readthedocs.org/en/latest/install.html</a>   

Para rodar o código ou acompanhar o post pelo IPython Notebook baixe o arquivo: <a href="https://s3.amazonaws.com/zahpee-public/ImportingOscarRunners.ipynb" target="_blank">ImportingOscarRunners.ipynb</a> 

## Base
**MongoDB**: O MongoDB é um banco de dados não relacional, ele é orientado a documentos e utiliza um formato chamado BSON, bem próximo ao JSON. Ele é bem simples de entender e utilizar.

**IMDbPy**: IMDbPy é uma biblioteca em python para acessar os dados do IMDb.

**PyMongo**: PyMongo é uma biblioteca em python para acessar o MongoDB.


## Let’s do this! (Leeroy Jenkins)

```python

from imdb import IMDb
import pymongo
from pymongo import MongoClient   
```


```python

movies = {}
movies['2016'] = ['The Big Short','Bridge of Spies','Brooklyn','Mad Max: Fury Road','The Martian','The Revenant','Room','Spotlight']
movies['2015'] = ['Birdman','American Sniper','Boyhood','The Grand Budapest Hotel','Imitation Game','Selma','The Theory of Everything','Whiplash']
movies['2014'] = ['American Hustle','Captain Phillips','Dallas Buyers Club','Gravity','Her (2013)','Nebraska','Philomena','The Wolf of Wall Street', '12 Years a Slave']
movies['2013'] = ['Argo','Amour','tt2125435','Django Unchained','tt1707386','Life of Pi','Lincoln','Silver Linings Playbook','Zero Dark Thirty']
movies['2012'] = ['The Artist','The Descendants','Extremely Loud & Incredibly Close','The Help','Hugo','Midnight in Paris','Moneyball','The Tree of Life','War Horse']
movies['2011'] = ['The King\'s Speech','127 Hours','Black Swan','The Fighter','Inception','The Kids Are All Right','The Social Network','Toy Story 3','True Grit','Winter\'s Bone']
movies['2010'] = ['The Hurt Locker','Avatar','tt0878804','District 9','An Education (2009)','Inglorious Bastards','Precious','A Serious Man','up','Up in the air (2009)']
movies['2009'] = ['Slumdog Millionaire','Benjamin Button','tt0870111','Milk','The Reader']
movies['2008'] = ['No Country for Old Men','Atonement','Juno','Michael Clayton','There Will Be Blood']
movies['2007'] = ['The Departed','Babel','Letters from Iwo Jima','Little Miss Sunshine (2006)','The Queen (2006)']
movies['2006'] = ['Crash','Brokeback Mountain','Capote','Good Night, and Good Luck.','tt0408306']
movies['2005'] = ['Million Dollar Baby','The Aviator','Finding Neverland','Ray','Sideways']
movies['2004'] = ['The Lord of the Rings: The Return of the King','Lost in Translation','Master and Commander: The Far Side of the World','Mystic River','Seabiscuit']
movies['2003'] = ['Chicago','tt0217505','tt0274558','The Lord of the Rings: The Two Towers','The Pianist']
movies['2002'] = ['A Beautiful Mind','Gosford Park','tt0247425','The Lord of the Rings: The Fellowship of the Ring','Moulin Rouge!']      
movies['2001'] = ['Gladiator','Chocolat','Crouching Tiger, Hidden Dragon','Erin Brockovich','Traffic']          

exceptions = {'tt0878804':'The Blind Side','tt1707386':'Les Misérables', 'tt0870111':'Frost/Nixo', 'tt2125435': 'Beasts of the Southern Wild', 'tt0247425': 'In the Bedroom', 'tt0408306':'Munich', 'tt0274558':'The Hours', 'tt0217505':'Gangs of New York'}


```

Primeiramente, foi feito o import das bibliotecas que vamos precisar. Logo depois, criamos a lista dos filmes indicados ao Oscar de “Melhor Filme” de 2000 a 2016. 

É importante salientar, que a melhor forma de utilizar o IMDbPy não é essa. O processo normal dessa biblioteca é importar a base inteira do IMDb para seu MySQL <a href="http://imdbpy.sourceforge.net/docs/README.sqldb.txt" target="_blank">(tutorial)</a>, mas como queria demonstrar o MongoDB, decidi fazer dessa maneira.

### Primeiramente, vamos inserir os indicados no MongoDB



```python

# First, we are going to insert the Oscar Nominees in the MongoDB database
# initiating the imdb api
ia = IMDb()

# starting the client
client = MongoClient()

# getting the db for the imdb database
db = client.imdb

# getting the movies colletion
movie_db = db.movies

```

Agora, iniciamos a api do IMDb e o cliente do MongoDB, logo estamos definindo que iremos trabalhar na database chamada ‘imdb’ e a variável movie_db será utilizada para trabalhar na coleção ‘movies’. 


```python
# creating a function that maps the movie json to a dictionary
def map_movie_to_dic(title, movie, year, exceptions):
              
    movie_dic = {}
    movie_dic['_id'] = movie.movieID
    movie_dic['oscar_year'] = year
    map_person_list(movie,movie_dic,'assistant director')
    map_person_list(movie,movie_dic,'director')   
    map_person_list(movie,movie_dic,'writer')  
    map_person_list(movie,movie_dic,'editor')
    map_person_list(movie,movie_dic,'cast') 
    map_person_list(movie,movie_dic,'editor') 
    map_person_list(movie,movie_dic,'original music')
    map_company_list(movie,movie_dic,'distributors')
    map_company_list(movie,movie_dic,'production companies')  
    map_company_list(movie,movie_dic,'special effects companies')         
    map_person_list(movie,movie_dic,'casting director') 
    
    if title in exceptions.keys():
        movie_dic['title'] = exceptions[title]
    else:
        movie_dic['title'] = title
    
    print 'Title searched: ' + title + ' - Title in the result: ' + movie['title']
        
    if 'plot' in movie.data:
        movie_dic['plot'] = movie['plot']
    if 'rating' in movie.data:    
        movie_dic['user_rating'] = movie['rating']
    else:
        print 'Movie ' + title + ' has no rating\n' + str(movie.data)        
    if 'countries' in movie.data:    
        movie_dic['countries'] = movie['countries']
    if 'genres'  in movie.data:
        movie_dic['genres'] = movie['genres']      
    else:
        print 'Movie ' + title + ' has no genres\n' + str(movie.data)
    movie_dic['runtime'] = movie['runtime']
    if 'languages' in movie.data:
        movie_dic['languages'] = movie['languages']  
    if 'opening weekend' in movie['business']:
        movie_dic['opening_weekend'] = get_money(movie['business']['opening weekend'])
    else:
        movie_dic['opening_weekend'] = 0  
    if 'budget' in movie['business']:
        movie_dic['budget'] = get_money(movie['business']['budget'])
    else:
        movie_dic['budget'] = 0        

    return movie_dic
```

A função *map_movie_to_dic()* mapeia o resultado da biblioteca IMDbPy (em formato JSON) para um dicionário de python. Mas para isso são necessárias as funções listadas abaixo.


```python
import re
# Creating a function that maps the money figure to a integer
def get_money(raw):
    regex = '\$([0-9],*)*'
    for item in raw:
        match = re.match(regex, item)
        if match:
            # return the first group matched without the $ and commas
            return int(match.group(0).strip('$').replace(',',''))
```

A função *get_money()* utiliza de um regex simples para transformar uma string no formato $100,00 em um inteiro 100.


```python
# Creating function that maps a person list. Saving only the name and id
def map_person_list(movie,movie_dic,field):
    movie_dic[field] = []  
    if field in movie.data:
        for person in movie[field]:    
            subdoc = {}
            subdoc['name'] = person['name']
            subdoc['id'] = person.personID    
            movie_dic[field].append(subdoc)
```

A função *map_person_list()* cria uma lista com dicionários, a idéia é que em um filme pode existir mais de um diretor, escritor e ator. 


```python
# Creating function that maps a company list. Saving only the name and id
def map_company_list(movie,movie_dic,field):
    movie_dic[field] = []  
    if field in movie.data:
        for person in movie[field]:    
            subdoc = {}
            subdoc['name'] = person['name']
            subdoc['id'] = person.companyID    
            movie_dic[field].append(subdoc)
```

A função *map_company_list* é muito parecida com a *map_person_list()* mas é utilizada para campos relativos a empresas.


```python
import time

# iterating in the movies list by year
for year in movies:
    
    #for each movie
    for movie in movies[year]:        
        # setting the number of retries
        for attempt in range(15):        
            # Check if this movie is already in the MongoDB database
            title = ''
            if movie in exceptions.keys():
                title = exceptions[movie]
            else:
                title = movie
            
            if movie_db.find_one({'title':title}):
                print 'Movie '+ movie +' already in mongodb'
                break;

            # search in the IMDB database by title
            try:
                imdb_result = ia.search_movie(movie)
            except Exception as e:
                print 'Oops!  Error getting movie \n' + str(e) 
                time.sleep(10)

            # if the result is not empty
            if len(imdb_result) > 0:
                best_match = imdb_result[0]

                #update necessary fiels
                ia.update(best_match)
                ia.update(best_match, 'business')

                # saving the movie in the database. 
                # pymongo is easy, just save the dictionary
                # Observation: if the database is going to be big, it is better to use
                # short field names in MongoDB, example 'title' turns into 't'
                movie_db.save(map_movie_to_dic(movie, best_match, int(year), exceptions))
                print 'Just saved the movie ' + movie  +' into the database'
                
                # sucess will break the attempts while
                break

            else:
                print 'No result for the movie: ' + movie +'. Trying again after 30 seconds.'
                ia = IMDb()
                time.sleep(30)
```

    Movie Slumdog Millionaire already in mongodb
    Movie Benjamin Button already in mongodb
    Movie tt0870111 already in mongodb
    Movie Milk already in mongodb
    Movie The Reader already in mongodb
    Movie A Beautiful Mind already in mongodb
    Movie Gosford Park already in mongodb
    Movie tt0247425 already in mongodb
    Movie The Lord of the Rings: The Fellowship of the Ring already in mongodb
    Movie Moulin Rouge! already in mongodb
    Movie Chicago already in mongodb
    Movie tt0217505 already in mongodb
    Movie tt0274558 already in mongodb
    Movie The Lord of the Rings: The Two Towers already in mongodb
    Movie The Pianist already in mongodb
    Movie Gladiator already in mongodb
    Movie Chocolat already in mongodb
    Movie Crouching Tiger, Hidden Dragon already in mongodb
    Movie Erin Brockovich already in mongodb
    Movie Traffic already in mongodb
    Movie Crash already in mongodb
    Movie Brokeback Mountain already in mongodb
    Movie Capote already in mongodb
    Movie Good Night, and Good Luck. already in mongodb
    Movie tt0408306 already in mongodb
    Movie The Departed already in mongodb
    Movie Babel already in mongodb
    Movie Letters from Iwo Jima already in mongodb
    Movie Little Miss Sunshine (2006) already in mongodb
    Movie The Queen (2006) already in mongodb
    Movie The Lord of the Rings: The Return of the King already in mongodb
    Movie Lost in Translation already in mongodb
    Movie Master and Commander: The Far Side of the World already in mongodb
    Movie Mystic River already in mongodb
    Movie Seabiscuit already in mongodb
    Movie Million Dollar Baby already in mongodb
    Movie The Aviator already in mongodb
    Movie Finding Neverland already in mongodb
    Movie Ray already in mongodb
    Movie Sideways already in mongodb
    Movie Birdman already in mongodb
    Movie American Sniper already in mongodb
    Movie Boyhood already in mongodb
    Movie The Grand Budapest Hotel already in mongodb
    Movie Imitation Game already in mongodb
    Movie Selma already in mongodb
    Movie The Theory of Everything already in mongodb
    Movie Whiplash already in mongodb
    Movie American Hustle already in mongodb
    Movie Captain Phillips already in mongodb
    Movie Dallas Buyers Club already in mongodb
    Movie Gravity already in mongodb
    Movie Her (2013) already in mongodb
    Movie Nebraska already in mongodb
    Movie Philomena already in mongodb
    Movie The Wolf of Wall Street already in mongodb
    Movie 12 Years a Slave already in mongodb
    Movie No Country for Old Men already in mongodb
    Movie Atonement already in mongodb
    Movie Juno already in mongodb
    Movie Michael Clayton already in mongodb
    Movie There Will Be Blood already in mongodb
    Title searched: The Big Short - Title in the result: The Big Short
    Just saved the movie The Big Short into the database
    Title searched: Bridge of Spies - Title in the result: Bridge of Spies
    Just saved the movie Bridge of Spies into the database
    Title searched: Brooklyn - Title in the result: Brooklyn
    Just saved the movie Brooklyn into the database
    Title searched: Mad Max: Fury Road - Title in the result: Mad Max: Fury Road
    Just saved the movie Mad Max: Fury Road into the database
    Title searched: The Martian - Title in the result: The Martian
    Just saved the movie The Martian into the database
    Title searched: The Revenant - Title in the result: The Revenant
    Just saved the movie The Revenant into the database
    Title searched: Room - Title in the result: Room

## Importando os filmes
O código acima itera nos anos selecionados e para cada ano ele pega a lista de filmes indicados. 

Cada filme é requisitado no banco de dados pelo seu título (**movie_db.find_one({'title':title})**), se não for encontrado então iremos na api do IMDb (usando a função **search_movie** ).

Como a api não é muito confiável/estável coloquei um *for* com 15 tentativas por filme. 

A função *ia.update()* é utilizada para buscar mais campos de cada filme.

A api retorna um JSON representando o filme, utilizamos então a função *map_movie_to_dic()* para transformar ela em um dicionário de python. Esse dicionário pode ser salvo diretamente na coleção 'movies' utilizando a função **save** do pyMongo (sim, é simples assim).


## Fazendo atualização dos ganhadores do Oscar


```python
# first we are going to create a list with all oscar winners.
# I could not find this information in the IMDB api
oscar_winners = ['Gladiator', 'A Beautiful Mind','Chicago', 'The Lord of the Rings: The Return of the King', 
                 'Million Dollar Baby', 'Crash', 'The Departed', 'No Country for Old Men', 'Slumdog Millionaire',
                 'The Hurt Locker', 'The King\'s Speech','The Artist', 'Argo', '12 Years a Slave', 'Birdman']

```

Primeiramente, listaremos aqui todos ganhadores do Oscar de melhor filme de 2000 a 2015.


```python
# Now we are going to iterate trough this list updating a new field
# with mongodb you do not have a schema, so we can update a non-existent field. It will create the field.

for winner in oscar_winners:
    # the update with pymongo is simple, is json-like operation
    # .update_one({QUERY}, {UPDATE})  https://docs.mongodb.org/getting-started/python/update/
    db.movies.update({'title': winner},{"$set": { "winner": True}})

# Now we can check if everything went smoothly, lets query for the oscar winners
# we are sorting by the oscar year (descending)
cursor = movie_db.find({"winner": True}).sort('oscar_year', pymongo.DESCENDING)
print 'The list to be updated has ' + str(len(oscar_winners)) + ' movies, the query returned '+ str(cursor.count())
for document in cursor:
    print(document['title'])
```

    The list to be updated has 15 movies, the query returned 15
    Birdman
    12 Years a Slave
    Argo
    The Artist
    The King's Speech
    The Hurt Locker
    Slumdog Millionaire
    No Country for Old Men
    The Departed
    Crash
    Million Dollar Baby
    The Lord of the Rings: The Return of the King
    Chicago
    A Beautiful Mind
    Gladiator


Agora iteramos pela lista de ganhadores e realizamos um update no documento adicionando um novo campo chamado "winner", esse campo será *True* se esse filme ganhou o oscar e não existirá se ele não ganhou.

O update no pymongo é simples: o primeiro parâmetro é a consulta que será realizada, no nosso caso, queremos o filme que o campo 'title' seja igual ao título do filme que ganhou o oscar. O segundo parâmetro é a operação de atualização que será realizada nos documentos que foram buscados pela consulta, nesse caso, estamos usando o operador de *$set* para atualizar/criar um campo chamado 'winner' e colocando o valor True para ele.

Depois disso, fazemos uma consulta por todos os filmes que tem o campo *winner* igual a True ordenando por data(invertido). Conferimos então o total da lista buscada pelo *find* com a lista inicial.


```python
# Now lets list all movies from 2013 and then that oscar winner from that year
cursor = movie_db.find({'oscar_year':2013})
print('The nominees for best picture in 2013 were:\n')
for document in cursor:
    print(document['title'])
    
result = movie_db.find_one({'oscar_year':2013,'winner':True})    
print('\nThe Winner from 2013 was: \n'+ result['title'])
```

    The nominees for best picture in 2013 were:
    
    Zero Dark Thirty
    Amour
    Beasts of the Southern Wild
    Django Unchained
    Les Misérables
    Life of Pi
    Lincoln
    Silver Linings Playbook
    Argo
    
    The Winner from 2013 was: 
    Argo


Por último, vamos listar todos os filmes de 2013 seguido pelo ganhador do mesmo.


Finalizando esse código temos um banco de dados no Mongodb chamado **"imdb"** com uma coleção **"movies"** com todos os filmes indicados ao Oscar de melhor de 2000 a 2016.

No próximo post iremos utilizar esses filmes para analisar os dados e tentar descobrir peculiaridades e padrões entre os filmes ganhadores.

Abraços e até o próximo

