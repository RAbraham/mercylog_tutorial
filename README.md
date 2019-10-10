### TLDR:

Datalog is like SQL + Recursion. It's derivatives have reduced the code base by 50% or more.

Today, I would like to explore a constrained  language called Datalog. It's a constrained form of Prolog and may not be as expressive as C++ or Python. But it's derivatives have known to reduce the numbers of lines of code down by 50% or more([Overlog](https://dl.acm.org/citation.cfm?id=1755913.1755937), [Yedalog](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43462.pdf)). 

Let's get started:


Datalog has a *minimalist syntax* which I love. Let's take an example.
Suppose our data is about fathers and sons. If we had an excel sheet, we would enter the data like:
```
Father  Son
'Aks'   'Bob'
'Bob'   'Cad'
'Yan'   'Zer'
```
In Datalog, we express the same data as:

```
father('Aks', 'Bob')
father('Bob', 'Cad')
father('Yan', 'Zer')
```
Here we are trying to say Aks is the father of 'Bob' and 'Bob' is the father of 'Cad'. The datum father('Aks', 'Bob') is called a fact i.e. it is true.

So Datalog can be used to express data. Not very interesting so far but a building block. These facts above can also be viewed as the existing state of the system, like we store data in files, or databases.

But that's not enough. What about code? Datalog sees code as rules to be applied declaratively.

Let's say our program needs to find out who's a grandfather. We could write a rule like:
'A person X is the grandfather of Z if X is the father of Y and Y is the father of Z'. In Datalog, this rule is written as:
 
```datalog
grandfather(X, Z) :- father(X,Y), father(Y, Z)
```
The LHS (i.e. grandfather) is known as the `head` and the RHS after the `:-` is known as the body

X, Y, Z are special variables called logic variables. They are different from regular variables. They are more used to represent a pattern or to link the head and the body.

To further understand logical variables, consider these two rules:


```bash
grandfather(X, Z) :- father(X,Y), father(Y, Z)
grandmother(X, Z) :- mother(X,Y), mother(Y, Z)
```
 Here the `X`, `Z` and `Y` used in `grandfather` are completely different from the `X`, `Y` and `Z` in `grandmother`. In rules, the variables only make sense `in` that single rule. So we can reuse the same logic variables in different rules without worrying that they have some logical connection.


The next concept is 'queries'. How do we feed input and get back some output. 
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

query # result
```
father(X,_) # ['Aks', 'Bob', 'Yan']
father(_,X) # ['Bob', 'Cad', 'Zer']
father(X, Y) # [('Aks', 'Bob'), ('Bob', 'Cad'), ('Yan', 'Zer')
father(X, 'Zer'), father('Zer', Y) # [] as there are no facts that match the query
grandfather(X, Y) # [('Aks', 'Cad')]
grandfather(X,_) # ['Aks']
```
Here '_' is a special variable indicating that you don't care for the result.


I was always interested in the Datalog syntax and it's power. I kept delaying it until I met Bashlog. Because, the *syntax of datalog is so simple*, it makes it *easy to write interpeters* for different targets. What Bashlog did was take Datalog syntax and convert it to bash scripts! Because, it used awk(* mawk actually), sed, grep, which are tuned for high performance on Unix like platforms, it was incredibly fast in parsing big text files comparable with all the specialized databases out there. Just Bash Scripts. It blew my mind. So if you are interested in pure Datalog, check out Bashlog(Raj: Link)

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

While in Datalog, you don't have to explicitly state the variables and the relation, as it is inbaked in to the language,  in our library in Python, we need to (e.g. X, Y, Z and father, grandfather)

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
If you notice, the rule for `paternal_grandfather` and `maternal_grandmother` are very similar. I could perhaps encapsulate that into a function. I'll use the word `transitive` though I believe it is wrong in this case but I don't know what to call this. Rewriting the above code:
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


Let's recap the benefits of Mercylog
- Simple Syntax. All you need to know is facts and rules. Because of such simplicity, it is also easy to build compilers for it.
- Expressive. Rules give a powerful mechanism 
- Declarative. Like SQL but more expressive.


I'll continue to update you with my future learnings!