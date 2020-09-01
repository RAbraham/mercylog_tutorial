### TLDR:

Datalog is like SQL + Recursion. Its derivatives have reduced the code base by 50% or more.

### Introduction
Today, I would like to explore a language called Datalog. Its a form of relational programming(like SQL) with a bit of logic programming(like Prolog) added to it, to make it a concise, elegant language. It may not be as expressive as Python and C++ but its benefits lie in its constraints. 

### The Power in Constraint
Relational programming constrains us to describe and capture the state of an application in relations(i.e. tables). The moment we can serialize state in relations, we can leverage parallelism or distributed systems as tables can be distributed and synchronized across processes/nodes using well understood database strategies like partitioning and replication. (RA: Ref Overlog). We could also simulate functional programming as we can imagine operations on a table or even the map reduce style popularized in big data. We could have multiple threads on the same machine work without sharing state(RA: This could be done with functional programming, making copies of objects and letting threads go wild but then how do we sync data changes. )

---------- Overlog: start  ------------------

1. Distributed  systems  benefit  substantially  from  adata-centricdesign style that focuses the programmer’s atten-tion on carefully capturing all the important state of thesystem as a family of collections (sets, relations, streams,etc.) Given such a model, the state of the system can bedistributed naturally and flexibly across nodes via familiarmechanisms like partitioning and replication.

2.The key behaviors of such systems can be naturally im-plemented usingdeclarativeprogramming languages thatmanipulate these collections, abstracting the programmerfrom both the physical layout of the data and the fine-grained orchestration of data manipulation
----------- Overlog: End
RA: Why are constraints good? What do we lose out if we have constraints.
RA: Once we talk about constraints, then explain how Datalog's constraints help

* Declarative: Declarative languages allow us to make 'statements' about our code i.e. 'what' we want to achieve without specifying 'how' we want to go about doing it. Some examples are:
- A self driving car sofware can take declarative statments like 'Drive me from Home to Work'. How the software achieves that task can depend on traffic conditions, toll road preferences etc.
- SQL is _theoretically_ declarative. When we write "SELECT * from TABLE_A INNER JOIN TABLE_B", each database(Oracle or PostgreSQL) are free to choose 'how' to achieve that goal. For e.g, Oracle may load `TABLE_A` into memory first because it's a small table while PostgreSQL may load `Table_B` into memory first because it is used more frequently. This is probably the worst explanation of SQL engines :P but hopefully you get the idea.

- RAJ: Another aspect of declarativity may be that the order in which the statements appear does not matter unlike imperative languages.

What does this declarativity give us? RAJ

RAJ->  What does declarative mean? "statements of a Datalog program can be stated in any order. " Allows optimization. See the success of SQL. 
* Easy to build backends for
* captures common patterns and prevent us from duplicating a lot of forms of programming

RAJ: Check OneNote if you have noted other benefits of Datalog

Because its constrained unlikeconstrained form of logic programming(e.g. Prolog) and may not be as expressive as C++ or Python. But its derivatives have been known to reduce the numbers of lines of code down by 50% or more([Overlog](https://dl.acm.org/citation.cfm?id=1755913.1755937), [Yedalog](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43462.pdf)). 

### Syntax
Let's get started:


Datalog has a *minimalist syntax* which I love. Let's take an example.
Suppose our data is about fathers and sons, mothers and daughters. If we had an excel sheet, we would enter the data like:
```
Father  Son
Aks     Bob
Bob     Cad
Yan     Zer
```
and another excel sheet for mothers and daughters:

```
Mother  Daughter
Mary    Marla
Marla   Kay
Jane    Zanu
```

In Datalog, we express the same data as below. For convenience, we show it together:

```
father('Aks', 'Bob')
father('Bob', 'Cad')
father('Yan', 'Zer')
mother('Mary', 'Marla')
mother('Marla', 'Kay')
mother('Jane', 'Zanu')
```
Here we are trying to say `Aks` is the father of `Bob` and `Bob` is the father of `Cad`. The datum `father('Aks', 'Bob')` is called a _**fact**_ i.e. it is true. `Aks` and `Bob` are called `terms`.

So Datalog can be used to express data. Not very interesting so far but a building block. These facts above can also be viewed as the existing state of the system, like we store state in files, or databases.

But that's not enough. What about code? For Datalog, code is specified as rules(i.e. `if` like statements) which apply to all the facts in the database. These `if` like statements look slightly different from what you are used in programming languages.

Let's say our program needs to find out who's a grandfather. In plain English, We would write a rule like:


`A person X is the grandfather of Z if X is the father of Y and Y is the father of Z` 

In Datalog, this rule is written as:
 
```datalog
grandfather(X, Z) :- father(X,Y), father(Y, Z)
```
The LHS (i.e. `grandfather`) is known as the `head` and the RHS after the `:-` is known as the `body` of the rule.


X, Y, Z are special variables called logic variables. They are different from regular variables. They are used as placeholders for data(e.g. `Aks`) and give a name to link the fact in the head to facts in the body.

---------------> Rename Facts as Relations?

To further understand logical variables, consider these two rules:


```bash
grandfather(X, Z) :- father(X,Y), father(Y, Z)
grandmother(X, Z) :- mother(X,Y), mother(Y, Z)
```
 Here the `X`, `Z` and `Y` used in `grandfather` are completely different from the `X`, `Y` and `Z` in `grandmother`. In rules, the variables only make sense `in` that single rule. So we can reuse the same logic variables in different rules without worrying that they have some logical connection.


The next concept is _*queries*_. How do we feed input and get back some output. 
Queries are similar to rules but without a head.
 
```bash
father(X, 'Aks'), mother(Z, 'Aks')
 
```
we mean, find the mother and father of 'Aks'
or 
```bash
father(X, 'Raj'), mother(Z, 'Raj')
```
we mean, find the father and mother of 'Raj'
 
Suppose, we want to say find the mother and father of all the children in the database, we make the query

```bash
father(X, Y), mother(Z, Y) 
```
Datalog will link `Y` for all `mother` and `father` facts and find the mothers and fathers for a child. It will not mix up fathers and mothers :)

Now, If you opened a datalog interpreter and fed the above and made the following queries, you would get the results shown after the # sign


```
father(X,_) # ['Aks', 'Bob', 'Yan']
father(_,X) # ['Bob', 'Cad', 'Zer']
father(X, Y) # [('Aks', 'Bob'), ('Bob', 'Cad'), ('Yan', 'Zer')
father(X, 'Zer'), father('Zer', Y) # [] as there are no facts that match the query
grandfather(X, Y) # [('Aks', 'Cad')]
grandfather(X,_) # ['Aks']
```
Here '_' is a special variable indicating that you don't care for the result.


I was always interested in the Datalog syntax and it's power. I kept delaying it until I met [Bashlog](https://github.com/thomasrebele/bashlog). Because, the *syntax of datalog is so simple*, it makes it *easy to write interpeters* for different targets. What Bashlog did was take Datalog syntax and convert it to bash scripts! Because, it used awk(mawk actually), sed, grep, which are tuned for high performance on Unix like platforms, it was incredibly fast in parsing big text files, comparable with all the specialized databases out there. **Just Bash Scripts**. It blew my mind. So if you are interested in pure Datalog, check out Bashlog

With Bashlog, you can run any bash like command and read that using Bashlog. Imagine there was a file('~/data.tsv') with tab separated values of
```bash
Aks Bob
Bob Cad
Yan Zer
```

We could read that data like:
```bash
facts(F, S) :~ cat ~/data.tsv
father(X, Y) :- facts(X, Y)
```
And then we proceed the same manner like before. What's awesome is that you can run any Unix command(e.g. `ls -l`) as long as it returns an output of tab separated values.

But I wanted to  use Datalog in my day to day programming. I wanted to see if I could use and leverage Datalog along with Python. Some benefits of Datalog in Python are:

- Modularity. How do we abstract out patterns in our rules and facts.
- Possible access to exisitng rich source of libraries.

So I built [Mercylog](https://github.com/RAbraham/mercylog) in Python. 

So let's translate the above rules to Mercylog syntax.

### Installation

If you are using the Bashlog variant,
 - then you need Java 8 already installed
 
```bash
git clone https://github.com/RAbraham/mercylog_tutorial.git
cd mercylog_tutorial
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python tutorial.py
```
That should print
```bash
['Aks']
['Mary']
```

Read below on the explanation and make tweaks if you want and run `python tutorial.py` again.

### Usage   
```python
import mercylog
m = mercylog.BashlogV1()
```
```python

# father('Aks', 'Bob')
# father('Bob', 'Cad')
# father('Yan', 'Zer')
# mother('Mary', 'Marla')
# mother('Marla', 'Kay')
# mother('Jane', 'Zanu')

father = m.relation('father')
mother = m.relation('mother')
facts = [
    father('Aks', 'Bob'),
    father('Bob', 'Cad'),
    father('Yan', 'Zer'),
    mother('Mary', 'Marla'),
    mother('Marla', 'Kay'),
    mother('Jane', 'Zanu'),
] 

```
```python
# grandfather(X, Z) :- father(X,Y), father(Y, Z)
grandfather = m.relation('grandfather')
X, Y, Z = m.variables('X', 'Y', 'Z') 
grandfather(X, Z) <= [father(X, Y), father(Y, Z)]
```

While in Datalog, you don't have to explicitly state the variables and the relation, as it is baked in to the language,  in our library in Python, we need to (e.g. X, Y, Z and father, grandfather)

Making a query in python has the following syntax
```python
m.run(facts, rules, query)
```
A concrete example would be:
```python
m.run(facts, rules, grandfather(X, Y)) # which gives [('Aks', 'Cad')]
m.run(facts, rules, father(X,_)) # ['Aks', 'Bob', 'Yan']
m.run(facts, rules,father(_,X)) # ['Bob', 'Cad', 'Zer']
m.run(facts, rules,father(X, Y)) # [('Aks', 'Bob'), ('Bob', 'Cad'), ('Yan', 'Zer')
m.run(facts, rules, granfather(X,_)) # ['Aks']
```

Creating this DSL in python gives us some unique benefits. For e.g if we had these two relations
```python
paternal_grandfather = m.relation('paternal_grandfather')
maternal_grandmother = m.relation('maternal_grandmother')
father = m.relation('father')
mother = m.relation('mother')

X, Y, Z = m.variables('X', 'Y', 'Z')
rules = [
    paternal_grandfather(X, Z) <= [father(X, Y), father(Y, Z)],
    maternal_grandmother(X, Z) <= [mother(X, Y), mother(Y, Z)]
] 

```
If you notice, the rule for `paternal_grandfather` and `maternal_grandmother` are very similar. I could perhaps encapsulate that into a function. I'll use the word `transitive` though I believe it is incorrect to use it.. but I don't know what to call this for now. Rewriting the above code:
```python

def transitive(head, clause):
    X, Y, Z = m.variables('X', 'Y', 'Z')
    return head(X, Z) <= [clause(X, Y), clause(Y, Z)]

paternal_grandfather = m.relation('paternal_grandfather')
maternal_grandmother = m.relation('maternal_grandmother')
father = m.relation('father')
mother = m.relation('mother')


rules = [
    transitive(paternal_grandfather, father),
    transitive(maternal_grandmother, mother)
]     
```

In this way, using Python, we have modularized a pattern using the `transitive` function.

RAJ: Add a recursive example.


Let's recap the benefits of Mercylog
- Simple Syntax. All you need to know is facts and rules. Because of such simplicity, it is also easy to build compilers for it.
- Expressive. Rules give a powerful mechanism 
- Declarative. Like SQL but more expressive. So we can optimize it's engines without affecting the code


I'll continue to update you with my future learnings!


# Additional Info
* Calling Datalog as `SQL + Recursion` is not true. SQL does support recursion. The intution I'm going for is that most of us are acquainted with the relational non recursive aspect of SQL.
