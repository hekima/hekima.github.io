---
layout: post
title: "Brincando com dados: Prevendo ganhadores do Oscar de 2017"
date:  2017-02-21 08:44:19
summary: No último post post analisamos as features do Rotten Tomatoes. Agora, vamos juntar todas as features relativas as principais premiações com as geradas utilizando os valores do Rotten Tomatoes e tentar prever quais serão os ganhadores dos Oscar de “Melhor Filme”, “Melhor Atriz” e “Melhor Ator”.
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
  - scikit
  - scikit-learn
  - oscar
  - movie
  - filmes
  - La la land
  - cinema
author: Luiz Mendes
authurl: https://www.linkedin.com/in/lfomendes
---
Olá a todos!

No <a href="http://developers.hekima.com/machine%20learning/python/2017/02/07/oscar-rotten/" target="_blank">último post</a> analisamos as features do <a href="http://rottentomatoes.com/" target="_blank">Rotten Tomatoes</a>. Agora, vamos juntar todas as features relativas as principais premiações com as geradas utilizando os valores do Rotten Tomatoes e tentar prever quais serão os ganhadores dos Oscar de “Melhor Filme”, “Melhor Atriz” e “Melhor Ator”.

<div style="text-align: center;"> <img src="https://media.giphy.com/media/FJIYU0hUj6IcE/giphy.gif" alt="et" style="width: 400px;"/> </div>

Antes de cada previsão vou falar quais são os meus indicados preferidos, porque além de mente eu também tenho um coração. =P 

## Objetivo 

O objetivo desse post é mostrar a criação de algumas features novas baseadas em outras já existentes, além demonstrar funções e possibilidades da biblioteca <a href="http://scikit-learn.org/stable/index.html" target="_blank">scikit-learn</a> de python para aprendizado de máquina.


## Preparativos
A fim de tornar o processo mais simples e didático, decidimos utilizar o Jupyter Notebook. Para fazer a instalação dele, siga o tutorial descrito aqui: http://jupyter.readthedocs.org/en/latest/install.html 

Para rodar o código ou acompanhar o post pelo Jupyter Notebook baixe o arquivo <a href="https://github.com/lfomendes/oscarPredictions/blob/master/Oscar%20Analysis%202017.ipynb"  target="_blank"> https://github.com/lfomendes/oscarPredictions/blob/master/Oscar%20Analysis%202017.ipynb </a>

## Base

<a href="http://scikit-learn.org/stable/index.html" target="_blank">Scikit-learn</a>: biblioteca de python mais utilizada para o fluxo de aprendizado de máquina, com funções de pré-processamento, algoritmos e funções de avaliação.

<a href="https://www.mongodb.com" target="_blank"> MongoDB </a>: banco de dados não relacional. Ele é orientado a documentos e utiliza um formato chamado BSON, bem próximo ao JSON. Ele é bem simples de entender e utilizar.

<a href="http://pandas.pydata.org" target="_blank">Pandas</a>: biblioteca de python para estrutura e análise de dados.

<a href="https://api.mongodb.com/python/current" target="_blank">PyMongo</a>: biblioteca em python para acessar o MongoDB.



## Votação do Oscar

Antes de avançarmos,  precisamos definir como funciona a votação do Oscar. Ela é um pouco peculiar, o que torna a tarefa de descobrir o vencedor mais complicada.

Vou colocar aqui um link para um texto e um vídeo que ajudam a entender essa “fórmula” maluca:

[Texto] <a href="http://g1.globo.com/pop-arte/oscar/2016/noticia/2016/02/oscar-2016-entenda-como-funciona-o-processo-de-votacao-da-academia.html" target="_blank">http://g1.globo.com/pop-arte/oscar/2016/noticia/2016/02/oscar-2016-entenda-como-funciona-o-processo-de-votacao-da-academia.html</a>

[Vídeo] <a href="https://www.youtube.com/watch?v=fS8UjdJZ2X8" target="_blank"> Como ganhar o bolão do Oscar? - Fora do Padron #5</a>

<br>

<div style="text-align: center;"> <img src="https://media.giphy.com/media/xUySTJ22aTy4YI5LBS/giphy.gif" alt="et" style="width: 400px;"/> </div>


## Meus preferidos

Antes de entrarmos no código em si, quero aproveitar essa oportunidade para criar uma conexão pessoal com **você**, leitor. Por isso,  vou colocar aqui minha opinião sobre os indicados a melhor filme.

**Indicados**

<ul>
  <li>“La La Land” </li>
  <li>“Lion”</li>
  <li>“Moonlight”</li>
  <li>“Manchester by the Sea”</li>
  <li>“Hidden Figures”</li>
  <li>“Arrival”</li>
  <li>“Hacksaw Ridge”</li>
  <li>“Fences”</li>
  <li>“Hell or High Water”</li>
</ul>

Dos indicados o único que não vi foi "Fences".

**Meu preferido do ano de 2016**

<a href="https://www.rottentomatoes.com/m/captain_fantastic/" target="_blank"> Capitão Fantástico</a> -  Sei que não está indicado a melhor filme, mas foi meu preferido de 2016. Uma linda e inesperada pérola do fatídico ano de 2016.

**Meu preferido dos indicados**

Essa é uma decisão difícil. Nesse ano temos filmes com histórias fortes, que tocam sentimentos profundos e passam mensagens importantes (e temos também “La la land”).

<div style="text-align: center;"> <img src="https://media.giphy.com/media/n6KHppJdQNsbe/giphy.gif" alt="et" style="width: 400px;"/> </div>

Brincadeiras à parte, eu gostei MUITO de <a href="https://www.rottentomatoes.com/m/arrival_2016" target="_blank"> Arrival (A Chegada)</a> e de <a href="https://www.rottentomatoes.com/m/moonlight_2016" target="_blank"> Moonlight </a>. <a href="https://www.rottentomatoes.com/m/lion_2016" target="_blank">"Lion"</a>, <a href="https://www.rottentomatoes.com/m/manchester_by_the_sea" target="_blank">"Manchester by the Sea"</a>  e <a href="https://www.rottentomatoes.com/m/hidden_figures" target="_blank">“Hidden Figures”</a> são ótimos filmes também, mas acho que se eu pudesse escolher...

<div style="text-align: center;"> <img src="https://media.giphy.com/media/3oriOeYGl5MKFtb2FO/giphy.gif" alt="et" style="width: 400px;"/> </div>


</br>
**Meu MENOS preferido dos indicados**

Gente, eu gostei de "La la land". É, com certeza, um bom filme, mas acho ele superestimado. Já  <a href="https://www.rottentomatoes.com/m/hacksaw_ridge" target="_blank"> “Hacksaw Ridge” - "Até o Último Homem" </a> é um filme com uma história real muito interessante, mas para mim a direção do Mel Gibson exagerou em várias cenas, além de eu não concordar muito com a visão dele de mundo.


Para finalizar vou deixar uma cena maravilhosa de Moonlight.

<div style="text-align: center;"> <img src="https://3.bp.blogspot.com/-BLUHmcV1bgE/WIb3iT3mIsI/AAAAAAAABjA/nviShMS3jgQOJz0Q7XpdD7aQ6-MyKDEsgCLcB/s640/giphy.gif" alt="et" style="width: 400px;"/> </div>


Estamos utilizando a mesma base de todos os outros posts, então vamos criar a conexão com o banco e pegar uma referência da coleção “movies”.


```python
import json
from pymongo import MongoClient

client = MongoClient() # Primeiramente pegamos o client do Mongodb
db = client.imdb 
movie_db = db.movies
```

## Atualizando vencedores dos prêmios

Vamos agora atualizar no MongoDB quais foram os vencedores das principais premiações.


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

#NAO SAIU AINDA - DIA 19
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

bafta_winners = ["La la land","The Revenant", "Boyhood","12 Years a Slave", "Argo", "The Artist", "The King's Speech", 
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

Criamos um cursor utilizando uma consulta vazia – isto é, não estamos filtrando nada, todos os filmes serão retornados. Logo depois, iteramos pelo cursor adicionando os documentos retornados na lista “movies”.

Depois disso, vamos criar um [DataFrame](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html) com todos os filmes e aplicar uma transformação em suas colunas.


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




    array(['_id', 'assistant director', 'bafta_winner', 'bafta_winners',
           'budget', 'cast', 'casting director', 'countries', 'dga_winner',
           'director', 'distributors', 'editor', 'genres', 'golden_globe',
           'languages', 'nyfcc_winner', 'opening_weekend', 'original music',
           'oscar_year', 'pga_winner', 'plot', 'production companies',
           'rotten_critic', 'rotten_people', 'rotten_position', 'runtime',
           'sag_winner', 'special effects companies', 'title', 'user_rating',
           'wga_winner', 'winner', 'writer'], dtype=object)



# Vamos fazer as previsões

Primeiro vamos transformar nossa entrada em tipos que são aceitos pelo scikit.
Vamos gerar um X (Matriz com features) e um Y (vetor de respostas) de entrada. O X será uma matriz com entradas numéricas/booleanas que serão nossa features, e o Y será nosso vetor de labels. , isto é, se o filme for ganhador do Oscar, ele terá valor 1, e 0 no caso contrário.

```python
from sklearn.ensemble import ExtraTreesClassifier
from sklearn.model_selection import cross_val_score

df_with_features = df.filter(items=['budget','title','rotten_position','user_rating','pga_winner','wga_winner','sag_winner','bafta_winner','dga_winner','golden_globe','rotten_people','oscar_year','rotten_critic','nyfcc_winner','winner'])
df_with_features.head()

year_df = {}
# lets create one DF for each year
for year in df['oscar_year'].unique():
    year_df[str(year)] = df_with_features[df_with_features['oscar_year'] == year]  
```

Antes de qualquer coisa, vamos gerar algumas features novas. Vamos combinar entradas booleanas fazendo um **AND** entre elas, ou seja, a nova feature só será **True** se as outras duas forem **True** também.

Com isso teremos uma feature que demonstra se o filme ganhou os dois prêmios no mesmo ano, por exemplo.


```python
df_with_features['pga_and_dga'] = df_with_features['pga_winner'] & df_with_features['dga_winner']
df_with_features['dga_and_sag'] = df_with_features['sag_winner'] & df_with_features['dga_winner']
df_with_features['pga_and_sag'] = df_with_features['sag_winner'] & df_with_features['pga_winner']
df_with_features['dga_and_wga'] = df_with_features['wga_winner'] & df_with_features['dga_winner']
df_with_features['pga_and_wga'] = df_with_features['wga_winner'] & df_with_features['pga_winner']
```

Agora vamos ver como se comportam os filmes que venceram o DGA e o PGA no mesmo ano.


```python
df_with_features[df_with_features['pga_and_dga']].filter(items=['title','pga_and_dga','winner','oscar_year']).head(100)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>pga_and_dga</th>
      <th>winner</th>
      <th>oscar_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>La la land</td>
      <td>True</td>
      <td>False</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Slumdog Millionaire</td>
      <td>True</td>
      <td>True</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Chicago</td>
      <td>True</td>
      <td>True</td>
      <td>2003</td>
    </tr>
    <tr>
      <th>24</th>
      <td>No Country for Old Men</td>
      <td>True</td>
      <td>True</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Brokeback Mountain</td>
      <td>True</td>
      <td>False</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>44</th>
      <td>The Lord of the Rings: The Return of the King</td>
      <td>True</td>
      <td>True</td>
      <td>2004</td>
    </tr>
    <tr>
      <th>54</th>
      <td>Birdman</td>
      <td>True</td>
      <td>True</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>79</th>
      <td>The King's Speech</td>
      <td>True</td>
      <td>True</td>
      <td>2011</td>
    </tr>
    <tr>
      <th>89</th>
      <td>The Hurt Locker</td>
      <td>True</td>
      <td>True</td>
      <td>2010</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Argo</td>
      <td>True</td>
      <td>True</td>
      <td>2013</td>
    </tr>
    <tr>
      <th>108</th>
      <td>The Artist</td>
      <td>True</td>
      <td>True</td>
      <td>2012</td>
    </tr>
  </tbody>
</table>
</div>



Apenas o filme  <a href="https://www.rottentomatoes.com/m/brokeback_mountain" target="_blank">O Segredo de Brokeback Mountain</a> "conseguiu" não ganhar o Oscar tendo sido o vitorioso nessas duas premiações.

Isso já mostra que o “La la land” é realmente o favorito…. Mas espera um momento. “La la land” ganhou também o WGA, o BAFTA, o Golden Globe e o NYFCC.


```python
df_with_features['except_sag'] = df_with_features['pga_winner'] & df_with_features['dga_winner'] & df_with_features['golden_globe'] & df_with_features['nyfcc_winner'] & df_with_features['bafta_winner']

df_with_features[df_with_features['except_sag']].filter(items=['title','except_sag','winner','oscar_year']).head(100)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>except_sag</th>
      <th>winner</th>
      <th>oscar_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>La la land</td>
      <td>True</td>
      <td>False</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Slumdog Millionaire</td>
      <td>True</td>
      <td>True</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Brokeback Mountain</td>
      <td>True</td>
      <td>False</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>44</th>
      <td>The Lord of the Rings: The Return of the King</td>
      <td>True</td>
      <td>True</td>
      <td>2004</td>
    </tr>
    <tr>
      <th>108</th>
      <td>The Artist</td>
      <td>True</td>
      <td>True</td>
      <td>2012</td>
    </tr>
  </tbody>
</table>
</div>


Pouquíssimos filmes ganharam todos esses prêmios. E, novamente, <a href="https://www.rottentomatoes.com/m/brokeback_mountain" target="_blank">"O Segredo de Brokeback Mountain"</a> é a exceção. Talvez o Oscar tenha errado em 2006, não é mesmo?

<div style="text-align: center;"> <img src="https://media.giphy.com/media/arKHzF88N0ctW/giphy.gif" alt="et" style="width: 400px;"/> </div>

Vamos agora testar um classificador com todas as features de prêmios, os votos do IMDb e as criadas a partir das notas do Rotten Tomatoes e criadas no <a href="http://developers.hekima.com/machine%20learning/python/2017/02/07/oscar-rotten/" target="_blank">último post</a>. 

Inicialmente vamos testar esse classificador nas premiações de 2001 a 2016 e ver quais acertamos e quais erramos.


```python
def mk_string(collection, sep=', ', start='', end=''):
    ret = start
    for item in collection:
        ret = ret + str(item) + sep
    # Removing the last inserted separator.
    ret = ret[:len(ret) - len(sep)]
    return ret + end
```


```python
from sklearn.linear_model import LogisticRegression
import numpy as np

# Vamos usar todos os outros anos para treino e ver qual a previsão para aquele ano

columns = ["pga_and_wga","dga_and_wga","pga_and_sag","dga_and_sag","pga_and_dga","user_rating", "rotten_people",'pga_winner','wga_winner','sag_winner','bafta_winner', 'dga_winner', 'golden_globe', 'rotten_critic', 'rotten_position'] 

def get_accuracy(model, df, debug= False):
    
    got_it = 0
    errors = []
    
    for year in sorted(df['oscar_year'].unique()):
        if(year == 2017):
            continue
        train = df_with_features[((df_with_features['oscar_year'] != year) & (df_with_features['oscar_year'] != 2017))] 
        test = df_with_features[df_with_features['oscar_year'] == year] 
        
        train_y = train["winner"].values
        train_X = train[list(columns)].values

        model = model.fit(train_X, train_y)

        test_y = test["winner"].values
        test_X = test[list(columns)].values

        probs = model.predict_proba(test_X)

        titles = np.array(test['title'].tolist())
        winner = np.array(test['winner'].tolist())

        prob_of_true = []
        i=0
        for prob in probs:
            prob_of_true.append(prob[1])

        prob_of_true = np.array(prob_of_true)   
        inds = prob_of_true.argsort()[::-1]
        if debug:
            print(inds)
            print(winner)
            print(titles)
            print(winner[inds][0])
            print(titles[inds][0])       
        
        if(winner[inds][0]):
            got_it += 1
        else:
            errors.append({'title':titles[inds][0],'year':year})
        
    accuracy = got_it/float(len(df['oscar_year'].unique())-1)    
    return got_it, errors
            
def print_results(acc,errors,df, model_name):
    print('Resultado para ' + model_name)
    print(str(acc) + ' acertos em ' + str(len(df['oscar_year'].unique())-1))
    accuracy = acc/float(len(df['oscar_year'].unique())-1) 
    print('Accuracy: ' + str(accuracy))
    print('\nAno do erro e qual filme foi escolhido pelo algoritmo.')
    print(mk_string(errors,sep='\n'))
           

logistic = LogisticRegression(solver='liblinear')
acc,errors = get_accuracy(logistic,df,False)

print_results(acc,errors,df,'Logistic Regression')
```

    Resultado para Logistic Regression
    13 acertos em 16
    Accuracy: 0.8125
    
    Ano do erro e qual filme foi escolhido pelo algoritmo.
    {'year': 2005, 'title': 'The Aviator'}
    {'year': 2006, 'title': 'Brokeback Mountain'}
    {'year': 2016, 'title': 'The Revenant'}


Podemos ver que, com uma Regressão Logística simples, o algoritmo acertou 13 de 16 anos, apontando  uma acurácia de 0.8125.

**81% de acerto!!!!**

<div style="text-align: center;"> <img src="https://media.giphy.com/media/xUySTWjvtZonjfyNNu/giphy.gif" alt="crystalball" style="width: 400px;"/></div>

Não acertamos apenas nos anos de 2005 com “O Aviador”, 2006 com “BrokeBack Mountain” e em 2016 com “O Regresso”.

Importante salientar que, em 2005 e 2006, os filmes ganhadores eram os segundos mais prováveis da lista. Já em  2016, Spotlight estava em terceiro lugar nas probabilidades.

Agora vamos testar outros algoritmos para ver a diferença da acurácia.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.ensemble import ExtraTreesClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.linear_model import SGDClassifier

models = [RandomForestClassifier(n_estimators=50,n_jobs=-1,random_state=12, criterion='entropy'),
          GradientBoostingClassifier(n_estimators=200,random_state=12),
          ExtraTreesClassifier(n_jobs=-1,random_state=12),
          GaussianNB()
          ]

#"budget", "opening_weekend"
for m in models:   
    acc,errors = get_accuracy(m,df)
    print_results(acc,errors,df,str(m))
    print('\n----------------- \n')
```

    Resultado para RandomForestClassifier(bootstrap=True, class_weight=None, criterion='entropy',
                max_depth=None, max_features='auto', max_leaf_nodes=None,
                min_impurity_split=1e-07, min_samples_leaf=1,
                min_samples_split=2, min_weight_fraction_leaf=0.0,
                n_estimators=50, n_jobs=-1, oob_score=False, random_state=12,
                verbose=0, warm_start=False)
    13 acertos em 16
    Accuracy: 0.8125
    
    Ano do erro e qual filme foi escolhido pelo algoritmo.
    {'year': 2001, 'title': 'Crouching Tiger, Hidden Dragon'}
    {'year': 2006, 'title': 'Brokeback Mountain'}
    {'year': 2016, 'title': 'The Revenant'}
    
    ----------------- 
    
    Resultado para GradientBoostingClassifier(criterion='friedman_mse', init=None,
                  learning_rate=0.1, loss='deviance', max_depth=3,
                  max_features=None, max_leaf_nodes=None,
                  min_impurity_split=1e-07, min_samples_leaf=1,
                  min_samples_split=2, min_weight_fraction_leaf=0.0,
                  n_estimators=200, presort='auto', random_state=12,
                  subsample=1.0, verbose=0, warm_start=False)
    11 acertos em 16
    Accuracy: 0.6875
    
    Ano do erro e qual filme foi escolhido pelo algoritmo.
    {'year': 2001, 'title': 'Crouching Tiger, Hidden Dragon'}
    {'year': 2005, 'title': 'Sideways'}
    {'year': 2006, 'title': 'Brokeback Mountain'}
    {'year': 2010, 'title': 'Inglorious Bastards'}
    {'year': 2016, 'title': 'The Revenant'}
    
    ----------------- 
    
    Resultado para ExtraTreesClassifier(bootstrap=False, class_weight=None, criterion='gini',
               max_depth=None, max_features='auto', max_leaf_nodes=None,
               min_impurity_split=1e-07, min_samples_leaf=1,
               min_samples_split=2, min_weight_fraction_leaf=0.0,
               n_estimators=10, n_jobs=-1, oob_score=False, random_state=12,
               verbose=0, warm_start=False)
    13 acertos em 16
    Accuracy: 0.8125
    
    Ano do erro e qual filme foi escolhido pelo algoritmo.
    {'year': 2005, 'title': 'The Aviator'}
    {'year': 2006, 'title': 'Brokeback Mountain'}
    {'year': 2016, 'title': 'The Big Short'}
    
    ----------------- 
    
    Resultado para GaussianNB(priors=None)
    13 acertos em 16
    Accuracy: 0.8125
    
    Ano do erro e qual filme foi escolhido pelo algoritmo.
    {'year': 2005, 'title': 'The Aviator'}
    {'year': 2006, 'title': 'Brokeback Mountain'}
    {'year': 2016, 'title': 'The Big Short'}
    
    ----------------- 
    


Pelo visto nenhum outro algoritmo conseguiu vencer o Logistic Regression, tendo 3 deles empatados nos acertos. Interessante observar que, mesmo tendo o mesmo número de acertos, dois deles fizeram a previsão de 2016 como “A Grande Aposta” ao invés de “O Regresso”.

Agora vamos fazer uma análise da importância das features.


```python
import numpy as np
import matplotlib.pyplot as plt

from sklearn.datasets import make_classification

forest = RandomForestClassifier(n_estimators=100,n_jobs=-1,random_state=12, criterion='entropy')

train = df_with_features[df_with_features['oscar_year'] != 2017] 

y = train["winner"].values
X = train[list(columns)].values

forest.fit(X, y)
importances = forest.feature_importances_
std = np.std([tree.feature_importances_ for tree in forest.estimators_],
             axis=0)
indices = np.argsort(importances)[::-1]

# Print the feature ranking
print("Feature ranking:")

for f in range(X.shape[1]):
    print("%d. feature %s (%f)" % (f + 1, columns[indices[f]], importances[indices[f]]))

# Plot the feature importances of the forest
plt.figure()
plt.title("Feature importances")
plt.bar(range(X.shape[1]), importances[indices],
       color="r", yerr=std[indices], align="center")
plt.xticks(range(X.shape[1]), indices)
plt.xlim([-1, X.shape[1]])
plt.show()
```

    Feature ranking:
    1. feature dga_winner (0.212265)
    2. feature rotten_position (0.113525)
    3. feature rotten_people (0.098893)
    4. feature pga_winner (0.087128)
    5. feature pga_and_dga (0.073356)
    6. feature rotten_critic (0.069652)
    7. feature user_rating (0.064661)
    8. feature sag_winner (0.059707)
    9. feature dga_and_sag (0.055013)
    10. feature pga_and_sag (0.049090)
    11. feature bafta_winner (0.034363)
    12. feature dga_and_wga (0.032665)
    13. feature wga_winner (0.026198)
    14. feature golden_globe (0.017712)
    15. feature pga_and_wga (0.005773)



<div style="text-align: center;"> <img src="/images/feat_oscar2017.png" alt="et" style="width: 400px;"/> </div>


Como já havíamos visto ano passado o <a href="https://en.wikipedia.org/wiki/Directors_Guild_of_America_Award" target="_blank">DGA</a>  é o principal prêmio que "guia" o Oscar. Parece que as features **rotten_position** e **rotten_people** foram úteis para o modelo. 

Podemos ver também que o **golden_globe** tem uma importância muito baixa no modelo criado.



<div style="text-align: center;"> <img src="https://media.giphy.com/media/l0MYtXwvCZ3ZOnyec/giphy.gif" alt="et" style="width: 400px;"/> </div>


## Previsão de 2017

Apesar de neste ano  “La la land” estar claramente na frente, vamos continuar o fluxo completo, montar um classificador com todos os dados até 2016 e ver qual o resultado para os Oscar desse ano.

<div style="text-align: center;"> <img src="https://media.giphy.com/media/BZUXTEvJSPsUo/giphy.gif" alt="crystalball" style="width: 400px;"/></div>

Vamos utilizar um Logistic Regression simples.



```python
test = df_with_features[df_with_features['oscar_year'] == 2017] 

logistic = LogisticRegression(solver='liblinear')
logistic.fit(X, y)

X_test = test[list(columns)].values
probs = logistic.predict_proba(X_test)
titles = np.array(test['title'].tolist())

prob_of_true = []
i=0
for prob in probs:
    prob_of_true.append(prob[1])

prob_of_true = np.array(prob_of_true)   
inds = prob_of_true.argsort()[::-1]

print('Ganhador previsto: "' + titles[inds][0] + '"                Probabilidade: ' + str(prob_of_true[inds][0]))
print('Correndo pelas beiradas: "' + titles[inds][1] + '"      Probabilidade: ' + str(prob_of_true[inds][1]))
print('Correndo pelas beiradas 2: "' + titles[inds][2] + '"     Probabilidade: ' + str(prob_of_true[inds][2]))
```

    Ganhador previsto: "La la land"                Probabilidade: 0.346224872931
    Correndo pelas beiradas: "Hidden Figures"      Probabilidade: 0.0451501339342
    Correndo pelas beiradas 2: "Hacksaw Ridge"     Probabilidade: 0.0210192693871


<div style="text-align: center;">  <img src="https://media.giphy.com/media/3o7aTJzfTFG2UYjLaM/giphy.gif" alt="dog" style="width: 400px;"/> </div>

<center><p style="font-size:14px">Momento de dança do filme "La la land"....</p></center>


É, parece que não tem para mais ninguém: vai dar **“La la land”** mesmo. O filme “Estrelas Além do Tempo” ficou em segundo por ter ganho o SAG e ter um alto valor na avaliação do público no Rotten Tomatoes.

<div style="text-align: center;">  <img src="https://media.giphy.com/media/3ofT5AANGg5LKLHIC4/giphy.gif" alt="dog" style="width: 400px;"/> </div>

<center><p style="font-size:14px">Agora sim, me desculpem a confusão</p></center>

Agora que já fizemos a previsão de **Melhor Filme**, vamos tentar prever a vencedora  do prêmio de melhor atriz e o vencedor de melhor ator.


## Prevendo melhor atriz

Para melhor atriz temos as seguintes candidatas:

* **Emma Stone** por "La La Land"
* **Isabelle Huppert** por "Elle"
* **Ruth Negga** por "Loving"
* **Meryl Streep** por "Florence Foster Jenkins"
* **Natalie Portman** por Jackie

Infelizmente, desses filmes eu só vi “La la land”, então não posso dar minha opinião pessoal. Portanto, vamos deixar os dados decidirem.

Vamos usar as seguintes features para fazer essa previsão:

* Ganhou o Sag Award do ano
* Ganhou o Golden Globe do ano
* Ganhou o NYFCC do ano
* Ganhou o BAFTA do ano
* Ganhou o NSFCA  do ano
* Soma dos prêmios ganhos no ano
* Quantas vezes já foi indicada para um Oscar
* Quantas vezes já foi indicada para um Oscar e não ganhou
* Quantas vezes já foi indicada para um Oscar e ganhou
* Idade em décadas

</br>
<div style="text-align: center;">  <img src="https://media.giphy.com/media/3o72F7ruEnE1Pc5tgQ/giphy.gif" alt="dog" style="width: 400px;"/> </div>
</br>



```python
X_until_2016 = np.loadtxt('X_train_actress.csv', delimiter=",")
y_until_2016 = np.loadtxt('y_train_actress.csv', delimiter=",")
X_2017 = np.loadtxt('X_test_actress.csv', delimiter=",")

actress = np.array(['Emma Stone','Isabelle Huppert','Ruth Negga','Meryl Streep','Natalie Portman'])

print(X_2017)
```

    [[  1.   1.   0.   1.   0.   3.   1.   1.   0.   2.]
     [  0.   1.   1.   0.   1.   3.   1.   1.   0.   6.]
     [  0.   0.   0.   0.   0.   0.   0.   0.   0.   3.]
     [  0.   0.   0.   0.   0.   0.  17.  14.   4.   6.]
     [  0.   0.   0.   0.   0.   0.   2.   1.   1.   3.]]


Estou fazendo o load das features de um arquivo externo.


```python
rfc = RandomForestClassifier(n_estimators=1000)
rfc.fit(X_until_2016,y_until_2016)
probs = rfc.predict_proba(X_2017)

prob_of_true = []
i=0
for prob in probs:
    prob_of_true.append(prob[1])

prob_of_true = np.array(prob_of_true)   
inds = prob_of_true.argsort()[::-1]

print('Ganhadora previsto: "' + actress[inds][0] + '"               Probabilidade: ' + str(prob_of_true[inds][0]))
print('Correndo pelas beiradas: "' + actress[inds][1] + '"    Probabilidade: ' + str(prob_of_true[inds][1]))
print('Correndo pelas beiradas 2: "' + actress[inds][2] + '"      Probabilidade: ' + str(prob_of_true[inds][2]))
```

    Ganhadora previsto: "Emma Stone"               Probabilidade: 0.989
    Correndo pelas beiradas: "Isabelle Huppert"    Probabilidade: 0.367
    Correndo pelas beiradas 2: "Meryl Streep"      Probabilidade: 0.04


Parece que a **Emma Stone** leva esse ano, sendo seguida pela "Isabelle Huppert".

<div style="text-align: center;">  <img src="http://i.imgur.com/xZVs9Gv.png" alt="dog" style="width: 400px;"/> </div>
<center><p style="font-size:14px">Emma Stone no filme "La la land"</p></center>

Lembrando a todos que a **probabilidade** indicada pelo algoritmo não é a chance de aquela atriz ganhar, e sim do valor do label ser **True**. Isto significa que o algoritmo não sabe que apenas uma pessoa ganha, e por isso a soma das probabilidades não é igual a 1.

## Prevendo melhor ator

Para melhor ator temos os seguintes indicados.

* **Andrew Garfield** por "Hacksaw Ridge"
* **Ryan Gosling** por "La La Land"
* **Denzel Washington** por "Fences"
* **Casey Affleck** por "Manchester by the Sea"
* **Viggo Mortensen** por "Captain Fantastic"

Desses filmes,  eu ainda não vi o “Fences”, mas meu preferido para esse prêmio é o **Viggo Mortensen** em “Capitão Fantástico”. A atuação de Mortensen não é apenas boa; eu diria que ela é…. FANTÁSTICA!

<div style="text-align: center;">  <img src="https://typeset-beta.imgix.net/2017/1/4/169752f0-feb1-4581-9fcb-6f6bacb8f0d8.gif?w=740&h=384&fit=max&auto=format" alt="dog" style="width: 400px;"/> </div>

Inclusive vou mostrar aqui cenas que simbolizam como alguém consegue atuar e passar várias emoções sem falar muito. Bastam  olhares e movimentos corporais.


<div style="text-align: center;">  <img src="http://68.media.tumblr.com/83474dbb9ddb7c4ebe205d9a8b4ece42/tumblr_o6mj1gFRp91rtpbuxo4_250.gif" alt="dog" style="width: 400px;"/> </div>

Nessa primeira cena, Viggo recebe um telefonema muito importante, enquanto na outra cena ele está sozinho em um ônibus. Inclusive já vou recomendar dois filmes em que ele atua: <a href="https://www.rottentomatoes.com/m/eastern_promises" target="_blank">"Eastern Promises"</a>  e <a href="https://www.rottentomatoes.com/m/history_of_violence" target="_blank">"History of Violence"</a>.

Agora que já puxei o saco do <a href="https://www.rottentomatoes.com/celebrity/viggo_mortensen/" target="_blank">Viggo Mortensen</a>, vou dizer que provavelmente Casey Affleck vai ganhar (veremos nos dados) e isso não será injusto. Sua atuação em “Manchester by the Sea” é espetacular.

Nesse ano temos uma peculiaridade nessa categoria, pois Casey é um candidato fortíssimo, mas teve problemas **SÉRIOS** fora do set. Ele foi acusado de assédio e abuso verbal por mulheres que trabalharam no set <a href="http://www.thedailybeast.com/articles/2016/11/22/casey-affleck-s-dark-secret-the-disturbing-allegations-against-the-oscar-hopeful.html" target="_blank">(fonte)</a>. 

<div style="text-align: center;">  <img src="https://media.giphy.com/media/3o7TKFjfeq2yZHloQw/giphy.gif" alt="dog" style="width: 400px;"/> </div>

Isso torna essa previsão um tanto quanto difícil, pois aparentemente ele é o mais forte candidato, tendo sido o vencedor do Golden Globe, NYFCC e NSFCA ( ele perdeu apenas o SAG para  Denzel Washington). 

Mas como é difícil incorporar essa visão sobre a pessoa Casey Affleck, então vamos nos limitar às features de premiações para fazer essa previsão. Pessoalmente, acho difícil separar a imagem do ator de sua atuação, pois apesar de Casey estar ótimo no filme “Manchester by the sea”, as histórias que circulam a seu respeito vão e deveriam afetar os votos.

```python
# 0 Won the Sag Award that year
# 1 Won the Golden Globes that year
# 2 Won the NYFCC that year
# 3 Won the BAFTA that year
# 4 won the NSFCA

X_until_2016 = np.loadtxt('X_train_actor.csv', delimiter=",")
y_until_2016 = np.loadtxt('y_train_actor.csv', delimiter=",")
X_2017 = np.loadtxt('X_test_actor.csv', delimiter=",")

actors = np.array(['Andrew Garfield','Denzel Washington','Casey Affleck','Viggo Mortensen','Ryan Gosling'])
print(X_2017)
```

    [[ 0.  0.  0.  0.  0.  0.]
     [ 1.  0.  0.  0.  0.  1.]
     [ 0.  1.  1.  1.  1.  4.]
     [ 0.  0.  0.  0.  0.  0.]
     [ 0.  1.  0.  0.  0.  1.]]


Utilizei menos features para atores, em relação às atrizes. Como eu busquei os dados de atrizes depois, acabei tendo novas ideias no caminho, então os atores ficaram apenas com as features de premiação.


```python
rfc = RandomForestClassifier(n_estimators=1000)
rfc.fit(X_until_2016,y_until_2016)
probs = rfc.predict_proba(X_2017)

prob_of_true = []
i=0
for prob in probs:
    prob_of_true.append(prob[1])

prob_of_true = np.array(prob_of_true)   
inds = prob_of_true.argsort()[::-1]

print('Ganhador previsto: "' + actors[inds][0] + '"            Probabilidade: ' + str(prob_of_true[inds][0]))
print('Correndo pelas beiradas: "' + actors[inds][1] + '"  Probabilidade: ' + str(prob_of_true[inds][1]))
print('Correndo pelas beiradas 2: "' + actors[inds][2] + '"     Probabilidade: ' + str(prob_of_true[inds][2]))
```

    Ganhador previsto: "Casey Affleck"            Probabilidade: 0.442384199134
    Correndo pelas beiradas: "Denzel Washington"  Probabilidade: 0.164549711502
    Correndo pelas beiradas 2: "Ryan Gosling"     Probabilidade: 0.124373451548


Parece que o Affleck será o grande vencedor, mas a distância para o Denzel e para Ryan Gosling não é muito grande, dado que um deles ganhou o SAG e o outro o Golden Globe.
<br>
<div style="text-align: center;">  <img src="https://68.media.tumblr.com/8e176f9217d8076feb4e57704633af0e/tumblr_oewutvzjof1vfii50o1_400.gif" alt="dog" style="width: 400px;"/> </div>
<br>

## Conclusão

Para concluir, gostaria de explicar que em **Aprendizado de Máquina** quase nunca é possível mapear tudo que envolve uma previsão. Sempre existirá alguma variável  que não dá para incorporar ao modelo (o caso do Casey Affleck, por exemplo). E é por isso que normalmente escolhemos uma métrica guia (acurácia, F1, precisão etc) e fazemos o teste com os dados antigos.

Outro ponto importante é que, no caso das previsões desse post, estou considerando que um evento de Oscar é **independente** dos outros. Isto significa  que o Oscar de um ano não influencia no outro.

Nesse post mostramos várias fucionalidades do Scikit-learn. Espero que, com esse  conhecimento, agora vocês possam brincar um pouco com essa biblioteca.

Então é isso, pessoal! Espero não demorar muito para postar novamente por aqui. =D

Abraços e bons filmes.

<br>
<div style="text-align: center;">  <img src="https://media.giphy.com/media/gnsrxBc8QM7HW/giphy.gif" alt="dog" style="width: 400px;"/> </div>
