---
title: A Network analysis of Game of Thrones (Part 1)
author: Lei Yan
date: '2019-01-04'
slug: a-network-analysis-of-game-of-thrones
categories:
  - Python Project
tags:
  - Python
  - networkx
---
## Which one is the most important character in *Game of Thrones*?
## How the importance of characters evolves over the books?
## How to define the importance of characters?
## Is there a strong correlation between different measures of importance? 


### After you read this post, you can easily answer the above questions!  

This actually is a [DataCamp Project](https://www.datacamp.com/projects/76). But I modified some code to draw three more beautiful pictures and added some comments. You can download the dataset at [here](https://pan.baidu.com/s/1YqZopId8iI0bbjjJ0N9t9g).          
In this project, the author assumes you are already familiar with <code>networkx</code> and <code>`pandas`</code>. If you don't know how to use them, you can learn these courses:

- [Network Analysis in Python (Part 1)](https://www.datacamp.com/courses/network-analysis-in-python-part-1) 
- [pandas Foundations](https://www.datacamp.com/courses/pandas-foundations)

I actually finished the first course and half of the second. But for this project, I think it is a better choice to look the offical documentation at [here](https://networkx.github.io/documentation/stable/tutorial.html) and [here](http://pandas.pydata.org/pandas-docs/stable/install.html) respectively. Or you can just google/baidu problems you encounter.    

If you don't want to know technical details, **you can just read the problem descriptions and results, totally skip the code blocks.**   

(PS: <code>degree centrality</code> and <code>betweeness centrality</code> are two different measures of importance of a node or a character in this project.)   


## 1. Winter is Coming. Let's load the dataset ASAP
<p>If you haven't heard of <em>Game of Thrones</em>, then you must be really good at hiding. Game of Thrones is the hugely popular television series by HBO based on the (also) hugely popular book series <em>A Song of Ice and Fire</em> by George R.R. Martin. In this notebook, we will analyze the co-occurrence network of the characters in the  Game of Thrones books. Here, two characters are considered to co-occur if their names appear in the vicinity of 15 words from one another in the books. </p> 

![Fo3mWQ.jpg](https://s2.ax1x.com/2019/01/03/Fo3mWQ.jpg)
<p>This dataset constitutes a network and is given as a text file describing the <em>edges</em> between characters, with some attributes attached to each edge. Let's start by loading in the data for the first book <em>A Game of Thrones</em> and inspect it.</p>


```python
# Importing modules
import pandas as pd
import matplotlib.pyplot as plt

# Reading in datasets/book1.csv
book1 = pd.read_csv('datasets/book1.csv')

# Printing out the head of the dataset
book1.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Source</th>
      <th>Target</th>
      <th>Type</th>
      <th>weight</th>
      <th>book</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Addam-Marbrand</td>
      <td>Jaime-Lannister</td>
      <td>Undirected</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Addam-Marbrand</td>
      <td>Tywin-Lannister</td>
      <td>Undirected</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aegon-I-Targaryen</td>
      <td>Daenerys-Targaryen</td>
      <td>Undirected</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aegon-I-Targaryen</td>
      <td>Eddard-Stark</td>
      <td>Undirected</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aemon-Targaryen-(Maester-Aemon)</td>
      <td>Alliser-Thorne</td>
      <td>Undirected</td>
      <td>4</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



## 2. Time for some Network of Thrones
<p>The resulting DataFrame <code>book1</code> has 5 columns: <code>Source</code>, <code>Target</code>, <code>Type</code>, <code>weight</code>, and <code>book</code>. Source and target are the two nodes that are linked by an edge. A network can have directed or undirected edges and in this network all the edges are undirected. The weight attribute of every edge tells us the number of interactions that the characters have had over the book, and the book column tells us the book number.</p>
<p>Once we have the data loaded as a pandas DataFrame, it's time to create a network. We will use <code>networkx</code>, a network analysis library, and create a graph object for the first book.</p>


```python
# Importing modules
import networkx as nx

# Creating an empty graph object
G_book1 = nx.Graph()
```

## 3. Populate the network with the DataFrame
<p>Currently, the graph object <code>G_book1</code> is empty. Let's now populate it with the edges from <code>book1</code>. And while we're at it, let's load in the rest of the books too!</p>  

[Documentation of .add_edge()](https://networkx.github.io/documentation/stable/reference/classes/generated/networkx.Graph.add_edge.html)


```python
# Iterating through the DataFrame to add edges
for _, edge in book1.iterrows(): # iterrows method returns a iterable object contains both index and row
    G_book1.add_edge(edge['Source'], edge['Target'], weight=edge['weight'])

# Creating a list of networks for all the books
books = [G_book1]
book_fnames = ['datasets/book2.csv', 'datasets/book3.csv', 'datasets/book4.csv', 'datasets/book5.csv']
for book_fname in book_fnames:
    book = pd.read_csv(book_fname)
    G_book = nx.Graph()
    for _, edge in book.iterrows():
        G_book.add_edge(edge['Source'], edge['Target'], weight=edge['weight'])
    books.append(G_book)
```

## 4. Finding the most important character in Game of Thrones
<p>Is it Jon Snow, Tyrion, Daenerys, or someone else? Let's see! Network Science offers us many different metrics to measure the importance of a node in a network. Note that there is no "correct" way of calculating the most important node in a network, every metric has a different meaning.</p>
<p>First, let's measure the importance of a node in a network by looking at the number of neighbors it has, that is, the number of nodes it is connected to. For example, an influential account on Twitter, where the follower-followee relationship forms the network, is an account which has a high number of followers. This measure of importance is called <em>degree centrality</em>.</p>
<p>Using this measure, let's extract the top ten important characters from the first book (<code>book[0]</code>) and the fifth book (<code>book[4]</code>).</p>


```python
# Calculating the degree centrality of book 1
deg_cen_book1 = nx.degree_centrality(books[0])

# Calculating the degree centrality of book 5
deg_cen_book5 = nx.degree_centrality(books[4])

# Sorting the dictionaries according to their degree centrality and storing the top 10
sorted_deg_cen_book1 = sorted(deg_cen_book1.items(), key=lambda x:x[1], reverse=True)[0:10]

# Sorting the dictionaries according to their degree centrality and storing the top 10
sorted_deg_cen_book5 = sorted(deg_cen_book5.items(), key=lambda x:x[1], reverse=True)[0:10]

# Printing out the top 10 of book1 and book5
print('top10 of book1: ', sorted_deg_cen_book1, '/n')
print('top10 of book5: ', sorted_deg_cen_book5)
```

    top10 of book1: [('Eddard-Stark', 0.3548387096774194), ('Robert-Baratheon', 0.2688172043010753), ('Tyrion-Lannister', 0.24731182795698928), ('Catelyn-Stark', 0.23118279569892475), ('Jon-Snow', 0.19892473118279572), ('Sansa-Stark', 0.18817204301075272), ('Robb-Stark', 0.18817204301075272), ('Bran-Stark', 0.17204301075268819), ('Joffrey-Baratheon', 0.16129032258064518), ('Cersei-Lannister', 0.16129032258064518)]
    
    top10 of book5: [('Jon-Snow', 0.1962025316455696), ('Daenerys-Targaryen', 0.18354430379746836), ('Stannis-Baratheon', 0.14873417721518986), ('Tyrion-Lannister', 0.10443037974683544), ('Theon-Greyjoy', 0.10443037974683544), ('Cersei-Lannister', 0.08860759493670886), ('Barristan-Selmy', 0.07911392405063292), ('Hizdahr-zo-Loraq', 0.06962025316455696), ('Asha-Greyjoy', 0.056962025316455694), ('Melisandre', 0.05379746835443038)]
    

## 5. Evolution of importance of characters over the books
<p>According to degree centrality, the most important character in the first book is Eddard Stark but he is not even in the top 10 of the fifth book. The importance of characters changes over the course of five books because, you know, stuff happens... ;)</p>
<p>Let's look at the evolution of degree centrality of a couple of characters like Eddard Stark, Jon Snow, and Tyrion, which showed up in the top 10 of degree centrality in the first book.</p>


```python
%matplotlib inline

# Creating a list of degree centrality of all the books
evol = [nx.degree_centrality(book) for book in books]
 
# Creating a DataFrame from the list of degree centralities in all the books
# Because it's possible that a character is not in every book, so use .fillna(0) to prevent `NaN` entries.
idx = ['book1', 'book2', 'book3', 'book4', 'book5']
degree_evol_df = pd.DataFrame.from_records(evol, index=idx).fillna(0)

# Plotting the degree centrality evolution of Eddard-Stark, Tyrion-Lannister and Jon-Snow
degree_evol_df.plot(y=['Eddard-Stark', 'Tyrion-Lannister', 'Jon-Snow'])
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f785d62ec18>




![FoYa4K.png](https://s2.ax1x.com/2019/01/03/FoYa4K.png)


## 6. What's up with Stannis Baratheon?
<p>We can see that the importance of Eddard Stark dies off as the book series progresses. With Jon Snow, there is a drop in the fourth book but a sudden rise in the fifth book.</p>
<p>Now let's look at various other measures like <em>betweenness centrality</em> and <em>PageRank</em> to find important characters in our Game of Thrones character co-occurrence network and see if we can uncover some more interesting facts about this network. Let's plot the evolution of betweenness centrality of this network over the five books. We will take the evolution of the top four characters of every book and plot it.</p>


```python
# Creating a list of betweenness centrality of all the books just like we did for degree centrality
evol = [nx.betweenness_centrality(book) for book in books]

# Making a DataFrame from the list
betweenness_evol_df = pd.DataFrame.from_records(evol).fillna(0)

# Finding the top 4 characters in every book
set_of_char = set()
for i in range(5):
    set_of_char |= set(list(betweenness_evol_df.T[i].sort_values(ascending=False)[0:4].index))
list_of_char = list(set_of_char)

# Plotting the evolution of the top characters in subplot
fig, axes = plt.subplots(nrows=4, ncols=3)
#print(list_of_char)    11
for i in range(3):
    for j in range(3):
        betweenness_evol_df.plot(ax=axes[i, j], y=list_of_char[i*3+j], figsize=(13, 7), ylim=(0, 0.5))
for j in range(2):
    betweenness_evol_df.plot(ax=axes[3, j], y=list_of_char[9+j], figsize=(13, 7), ylim=(0, 0.5))
```

[![FoW6q1.md.png](https://s2.ax1x.com/2019/01/03/FoW6q1.md.png)](https://imgchr.com/i/FoW6q1)


## 7. What does the Google PageRank algorithm tell us about Game of Thrones?
<p>We see a peculiar rise in the importance of Stannis Baratheon over the books. In the fifth book, he is significantly more important than other characters in the network, even though he is the third most important character according to degree centrality.</p>
<p>PageRank was the initial way Google ranked web pages. It evaluates the inlinks and outlinks of webpages in the world wide web, which is, essentially, a directed network. Let's look at the importance of characters in the Game of Thrones network according to PageRank. </p>


```python
# Creating a list of pagerank of all the characters in all the books
evol = [nx.pagerank(book) for book in books]

# Making a DataFrame from the list
pagerank_evol_df = pd.DataFrame.from_records(evol).fillna(0)

# Finding the top 4 characters in every book
set_of_char = set()
for i in range(5):
    set_of_char |= set(list(pagerank_evol_df.T[i].sort_values(ascending=False)[0:4].index))
list_of_char = list(set_of_char)
# print(len(list_of_char))    14
# Plotting the top characters
fig, axes = plt.subplots(nrows=5, ncols=3)
for i in range(5):
    for j in range(3):
        pagerank_evol_df.plot(ax=axes[i, j], y=list_of_char[3*i+j], ylim=(0, 0.08), figsize=(13, 9))
# Although an IndexError will be thrown, but the output picture is correct
```

[![FohqC8.md.png](https://s2.ax1x.com/2019/01/03/FohqC8.md.png)](https://imgchr.com/i/FohqC8)

## 8. Correlation between different measures
<p>Stannis, Jon Snow, and Daenerys are the most important characters in the fifth book according to PageRank. Eddard Stark follows a similar curve but for degree centrality and betweenness centrality: He is important in the first book but dies into oblivion over the book series.</p>
<p>We have seen three different measures to calculate the importance of a node in a network, and all of them tells us something about the characters and their importance in the co-occurrence network. We see some names pop up in all three measures so maybe there is a strong correlation between them?</p>
<p>Let's look at the correlation between PageRank, betweenness centrality and degree centrality for the fifth book using Pearson correlation.</p>


```python
# Creating a list of pagerank, betweenness centrality, degree centrality
# of all the characters in the fifth book.
measures = [nx.pagerank(books[4]), 
            nx.betweenness_centrality(books[4], weight='weight'), 
            nx.degree_centrality(books[4])]

# Creating the correlation DataFrame
cor = pd.DataFrame.from_records(measures)

# Calculating the correlation
cor.T.corr() # remember to transpose the dataframe, otherwise the correlation between characters will be computed. 
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.000000</td>
      <td>0.793372</td>
      <td>0.971493</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.793372</td>
      <td>1.000000</td>
      <td>0.833816</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.971493</td>
      <td>0.833816</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



## 9. Conclusion
<p>We see a high correlation between these three measures for our character co-occurrence network.</p>
<p>So we've been looking at different ways to find the important characters in the Game of Thrones co-occurrence network. According to degree centrality, Eddard Stark is the most important character initially in the books. But who is/are the most important character(s) in the fifth book according to these three measures? </p>  

This task can easily be completed using the [idxmax method](http://pandas.pydata.org/pandas-docs/version/0.17.0/generated/pandas.DataFrame.idxmax.html).

```python
# Finding the most important character in the fifth book,  
# according to degree centrality, betweenness centrality and pagerank.
p_rank, b_cent, d_cent = cor.idxmax(axis=1)

# Printing out the top character accoding to the three measures
print(p_rank, b_cent, d_cent)
```
Jon-Snow Stannis-Baratheon Jon-Snow


