*******************************
``import py2neo`` -- Quickstart
*******************************

As everyone knows:

``python`` == best

``neo4j`` == amazing

.. image:: both.gif

``python`` + ``neo4j`` = ``Py2Neo`` <3

We are so grateful to https://github.com/technige for combining them as
https://github.com/technige/py2neo.

---

    This guide exists to demonstrate how to run the ``cypher`` in the built-in
    `Neo4j` tutorial **"movie demo database"** using
    technige_'s
    Py2Neo_.

.. _technige: https://github.com/technige
.. _Py2Neo: https://github.com/technige/py2neo

---

Both the python driver for neo4j and py2neo are actively being developed. This guide was created
based on versions:

* Py2neo v5
* Neo4j 4.0

This is **not** a beginner-beginner tutorial, but it's
pretty close to the beginning. We'll be assuming that you know what both ``python``
and ``Neo4j``\/``cypher`` are and have them both running and have played with
them a bit and are maybe still learning.

*... hey, that's a bit too fast ...*

**If you'd like to know more about ``Python``**: the programming language: there
is a whole internet of resources, this is the official we like guide to kicking
off: https://wiki.python.org/moin/BeginnersGuide/Download.

**If you'd like to know more about ``Neo4j``**: the awesome-sauce property graph
database. Take your first steps using the docs here:
https://neo4j.com/developer/get-started/.

Going forward we're going to use jargon and assume you are already familiar with these <3

*... hey, that's a bit too slow ...*

This document is to replay the basic cypher tutorial "Movie Graph", as Py2Neo.
This would potentially be useful to understand *interacting* with a graph using python.
This is just some demo code, not a whole project.

If you'd like something heavier, say to *deploy* this graph in a
project, there's the official python-flask demo project here:
https://github.com/neo4j-examples/movies-python-bolt

---

If you're cool with this, you're ready for this quickstart!

**Prologue**

The built-in `Neo4j` tutorial "movie demo database" is one of the first things
you see when you first open the `Neo4j` browser interface. The instructions to
install this are here: https://neo4j.com/developer/guide-neo4j-browser/

Once you install and start Neo4j the defaul neo4j-browser url is:
http://localhost:7474/browser/. From here you can see the big friendly
**"Jump into code: Write Code"** ``:play write-code``.

  The Movie Graph is a mini graph application containing actors and directors that are related through the movies they've collaborated on.

  This guide will show you how to:

  1. Create: insert movie data into the graph
  2. Find: retrieve individual movies and actors
  3. Query: discover related actors and directors
  4. Solve: the Bacon Path

In this document we going to replicate this ``:play movie-graph`` using Py2Neo.

All code examples presented are also here: `Movie Graph as Py2Neo Jupyter Notebook <https://github.com/elena/py2neo-quickstart/blob/main/py2neo-quickstart.ipynb>`_


---

.. contents::


Create
++++++

This is a sample of the ``cypher`` from the tutorial, hopefully this looks familiar.
This is what we'll convert to python.

``cypher``:

.. code-block:: cypher

    // Nodes
    CREATE (TheMatrix:Movie {title:'The Matrix', released:1999, tagline:'Welcome to the Real World'})
    CREATE (Keanu:Person {name:'Keanu Reeves', born:1964})
    CREATE (Carrie:Person {name:'Carrie-Anne Moss', born:1967})
    CREATE (Laurence:Person {name:'Laurence Fishburne', born:1961})
    CREATE (Hugo:Person {name:'Hugo Weaving', born:1960})
    CREATE (LillyW:Person {name:'Lilly Wachowski', born:1967})
    CREATE (LanaW:Person {name:'Lana Wachowski', born:1965})
    CREATE (JoelS:Person {name:'Joel Silver', born:1952})
    CREATE (Emil:Person {name:"Emil Eifrem", born:1978})
    CREATE (Emil)-[:ACTED_IN {roles:["Emil"]}]->(TheMatrix)

    // Relationships
    CREATE
      (Keanu)-[:ACTED_IN {roles:['Neo']}]->(TheMatrix),
      (Carrie)-[:ACTED_IN {roles:['Trinity']}]->(TheMatrix),
      (Laurence)-[:ACTED_IN {roles:['Morpheus']}]->(TheMatrix),
      (Hugo)-[:ACTED_IN {roles:['Agent Smith']}]->(TheMatrix),
      (LillyW)-[:DIRECTED]->(TheMatrix),
      (LanaW)-[:DIRECTED]->(TheMatrix),
      (JoelS)-[:PRODUCED]->(TheMatrix)


You'll notice that this is effectivley **1 step**, where you create:

    Step 1. ``CREATE`` ``nodes``, then ``CREATE`` ``relationships``.

You don't really think about committing this transaction.

---

Using the ``python`` driver/``py2neo`` you must specifically think about both:

* **Connecting** to your Graph DB
* **Committing** the transaction

So using ``py2neo`` there are **3 steps**.

    Step 0: Connect to your GraphDB

    Step 1: Create your ``Node`` and ``Relationship`` objects

    Step 2: Commit your Subgraphs (https://py2neo.org/v5/data.html#subgraph-objects)

--

Step 0: Connect to Graph using Py2Neo
-------------------------------------

.. code-block:: python

    from py2neo import Graph

    my_graph = Graph(password='[mysekretpasswordhere]')


There are plenty of options for connecting to your database if this implementation
doesn't work for you.

For example, the following are all functionally **equivalent**:

.. code-block:: python

    my_graph0 = Graph()
    my_graph1 = Graph(host="localhost")
    my_graph2 = Graph("bolt://localhost:7687")

    my_graph0 == my_graph1 == my_graph2


See the reference here: https://py2neo.org/v5/database.html#py2neo.database.Graph

Note that as of Neo4j version 4: if you have **multiple graphs databases**, you
can choose which database you connect to using the ``name`` argument, see the docs above.
Multi-database support is in active development at the Neo4j level in versions 4 add 5.
https://neo4j.com/developer/manage-multiple-databases/

A full list of database ``names`` can be shown through the Cypher:

``cypher``:

.. code-block:: cypher

    // switch to system database
    :use system

.. code-block:: cypher

    SHOW DATABASES

---

Step 1: Create ``Node`` and ``Relationship`` Subgraphs using Py2Neo
-------------------------------------------------------------------

Full ``Node`` and ``Relationship`` reference: https://py2neo.org/v5/data.html

``python``:

.. code-block:: python

    from py2neo import Node, Relationship

    # Nodes
    TheMatrix = Node("Movie", title='The Matrix', released=1999, tagline='Welcome to the Real World')
    Keanu = Node("Person", name='Keanu Reeves', born=1964)
    Carrie = Node("Person", name='Carrie-Anne Moss', born=1967)
    Laurence = Node("Person", name='Laurence Fishburne', born=1961)
    Hugo = Node("Person", name='Hugo Weaving', born=1960)
    LillyW = Node("Person", name='Lilly Wachowski', born=1967)
    LanaW = Node("Person", name='Lana Wachowski', born=1965)
    JoelS = Node("Person", name='Joel Silver', born=1952)
    Emil = Node("Person", name="Emil Eifrem", born=1978)

    # Relationships
    LillyWTheMatrix = Relationship(LillyW, "DIRECTED", TheMatrix)
    LanaWTheMatrix = Relationship(LanaW, "DIRECTED", TheMatrix)
    JoelSTheMatrix = Relationship(JoelS, "PRODUCED", TheMatrix)
    KeanuTheMatrix = Relationship(Keanu, "ACTED_IN", TheMatrix)
    KeanuTheMatrix['roles'] = ['Neo']
    CarrieTheMatrix = Relationship(Carrie, "ACTED_IN", TheMatrix)
    CarrieTheMatrix['roles'] = ['Trinity']
    LaurenceTheMatrix = Relationship(Laurence, "ACTED_IN", TheMatrix)
    LaurenceTheMatrix['roles'] = ['Morpheus']
    HugoTheMatrix = Relationship(Hugo, "ACTED_IN", TheMatrix)
    HugoTheMatrix['roles'] = ['Agent Smith']
    EmilTheMatrix = Relationship(Emil, "ACTED_IN", TheMatrix)
    EmilTheMatrix['roles'] = ['Emil']

Note: This looks great but **YOUR DB OBJECTS DO NOT EXIST YET!**

They need to committed to the database.


Step 2: Commit using Py2Neo
---------------------------

``python``:

.. code-block:: python

    # Commit the transactions

    tx = graph.begin()
    tx.create(TheMatrix)
    tx.create(Keanu)
    tx.create(Carrie)
    tx.create(Laurence)
    tx.create(Hugo)
    tx.create(LillyW)
    tx.create(LanaW)
    tx.create(JoelS)
    tx.create(Emil)
    tx.create(KeanuTheMatrix)
    tx.create(CarrieTheMatrix)
    tx.create(LaurenceTheMatrix)
    tx.create(HugoTheMatrix)
    tx.create(LillyWTheMatrix)
    tx.create(LanaWTheMatrix)
    tx.create(JoelSTheMatrix)
    tx.create(EmilTheMatrix)
    tx.commit()


This is just a sample from the more detailed example database provided at:
https://neo4j.com/developer/movie-database/. The gist of the full dataset can be
found here: https://gist.github.com/elena/733275bd55fba0a48cd885fe0427e5d4

The full set is also with the code examples that go along with this here: `Movie Graph as Py2Neo Jupyter Notebook <https://github.com/elena/py2neo-quickstart/blob/main/py2neo-quickstart.ipynb>`_

---

Find
++++

    Example queries for finding individual nodes.


Connect to the database:

.. code-block:: python

    from py2neo import Graph
    graph = Graph(password='[yoursekretpasswordhere]')


There are **multiple methods** of instantiating ``NodeMatcher``.

.. code-block:: python

   nodes_matcher = NodeMatcher(graph)
   nodes_matcher.match()

   # this is the same as:

   graph.nodes.match()


https://py2neo.org/v5/matching.html#py2neo.matching.NodeMatcher
https://py2neo.org/v5/database.html#py2neo.database.Graph.nodes


Quick demo:

.. code-block:: python

    keanu = graph nodes.match("Person", name="Keanu Reeves").first()

    Out[]: Node('Person', born=1964, name='Keanu Reeves')


.. code-block:: python

    match_using_matcher = node_matcher.match(name="Keanu Reeves").first()
    match_using_graphnodes = graph.nodes.match(name="Keanu Reeves").first()

    match_using_matcher == match_using_graphnodes

    Out[]: True

---

Demo from https://py2neo.org/v5/database.html#py2neo.database.Graph.nodes:

.. code-block:: python

    keanu0 = graph.nodes[1]
    keanu1 = graph.nodes.get(1)
    keanu2 = graph.nodes.match("Person", name="Keanu Reeves").first()

    keanu0 == keanu1 == keanu2

    Out[]: True


.. code-block:: python

    len(graph.nodes.match("Person"))

    Out[]: 145


Note, the full set of data has been loaded, you can see this:

* https://github.com/elena/py2neo-quickstart/blob/main/py2neo-movie-graph-data.ipynb
* https://gist.github.com/elena/733275bd55fba0a48cd885fe0427e5d4
* https://neo4j.com/developer/movie-database/


---

Find the actor named "Tom Hanks"...
-----------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (tom {name: "Tom Hanks"}) RETURN tom

``python``:

.. code-block:: python

    node_matcher.match(name="Tom Hanks").first()

    Out[]: Node('Person', born=1956, name='Tom Hanks')


Note: don't forget the **``.first()``**. Without it you get a ``NodeMatch``
object, which is probably not what you want.


There may be performance differences based upon your use case. As a general
rule it's better to be specific in queries (in this case using the label
"Person" would assist performance).


Find the movie with title "Cloud Atlas"...
------------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"}) RETURN cloudAtlas

``python``:

.. code-block:: python

    node_matcher.match(title="Cloud Atlas").first()

    Out[]: Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas')


Find 10 people...
-----------------

``cypher``:

.. code-block:: cypher

    MATCH (people:Person) RETURN people.name LIMIT 10

``python``:

.. code-block:: python

    node_matcher.match("Person").limit(10).all()

    Out[]: [Node('Person', born=1964, name='Keanu Reeves'),
            Node('Person', born=1967, name='Carrie-Anne Moss'),
            Node('Person', born=1961, name='Laurence Fishburne'),
            Node('Person', born=1960, name='Hugo Weaving'),
            Node('Person', born=1967, name='Lilly Wachowski'),
            Node('Person', born=1965, name='Lana Wachowski'),
            Node('Person', born=1952, name='Joel Silver'),
            Node('Person', born=1978, name='Emil Eifrem'),
            Node('Person', born=1964, name='Keanu Reeves'),
            Node('Person', born=1967, name='Carrie-Anne Moss')]


Note: don't forget the **``.all()``**. Without it you get a ``NodeMatch``
object, which is probably not what you want.



Find movies released in the 1990s...
------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title

``python``:

There are a list of standard operators available such as ``=``, ``<>``, etc.
See the full list here: https://py2neo.org/v5/matching.html#node-matching

.. code-block:: python

    node_matcher.match("Movie").where('_.released >= 1990', '_.released < 2000')

    Out[] = [Node('Movie', released=1999, tagline='Welcome to the Real World', title='The Matrix'),
             Node('Movie', released=1992, tagline="In the heart of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title='A Few Good Men'),
             Node('Movie', released=1992, tagline='Once in a lifetime you get a chance to do something different.', title='A League of Their Own'),
             Node('Movie', released=1999, tagline='First loves last. Forever.', title='Snow Falling on Cedars'),
             Node('Movie', released=1996, tagline='In every life there comes a time when that thing you dream becomes that thing you do', title='That Thing You Do'),
             Node('Movie', released=1998, tagline='After life there is more. The end is just the beginning.', title='What Dreams May Come'),
             ...
             Node('Movie', released=1998, tagline='At odds in life... in love on-line.', title='When Harry Met Sally'),

Watch the prefix **`"_."`** in the ``where`` statement.

https://py2neo.org/v5/matching.html#py2neo.matching.NodeMatch.where

---

Query
+++++

  Finding patterns within the graph.

  1. Actors are people who acted in movies
  2. Directors are people who directed a movie
  3. What other relationships exist?


Connect to the database:

.. code-block:: python

    from py2neo import Graph
    graph = Graph(password='[yoursekretpasswordhere]')


There are **multiple methods** of instantiating ``RelationshipMatcher``.

.. code-block:: python

   relationship_matcher = RelationshipMatcher(graph)
   relationship_matcher.match()

   # this is the same as:

   graph.relationships.match()


https://py2neo.org/v5/matching.html#py2neo.matching.RelationshipMatch
https://py2neo.org/v5/database.html#py2neo.database.Graph.match


---

List all Tom Hanks movies...
----------------------------

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies

``python``:

.. code-block:: python

    graph.nodes.match(name="Tom Hanks").first()
    graph.match(nodes=[tom], r_type="ACTED_IN").all()

    Out[]: [ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=2006, tagline='Break The Codes', title='The Da Vinci Code'), roles=['Dr. Robert Langdon']),
            ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=1990, tagline='A story of love, lava and burning desire.', title='Joe Versus the Volcano'), roles=['Joe Banks']),
            ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=1999, tagline="Walk a mile you'll never forget.", title='The Green Mile'), roles=['Paul Edgecomb']),
            ...
            ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'), roles=['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins']),
            ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=2004, tagline='This Holiday Season… Believe', title='The Polar Express'), roles=['Hero Boy', 'Father', 'Conductor', 'Hobo', 'Scrooge', 'Santa Claus']),
            ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=1996, tagline='In every life there comes a time when that thing you dream becomes that thing you do', title='That Thing You Do'), roles=['Mr. White'])]


Who directed "Cloud Atlas"?
---------------------------

``cypher``:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name

This is possible, but getting out of the scope of ``py2neo``, the following are all cases where falling back to native cypher is probably best.

``python``:

.. code-block:: python

    results = graph.run('MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name')
    results.data()

    Out[]: [{'directors.name': 'Tom Tykwer'},
            {'directors.name': 'Lilly Wachowski'},
            {'directors.name': 'Lana Wachowski'}]

The following will produce the same result, although is less elegant:

``python``:

.. code-block:: python

    cloudAtlas = matcher.match(title="Cloud Atlas").first()
    directors = graph.match(r_type="DIRECTED", nodes=(None, cloudAtlas)) # << see notes about use of nodes=() here
    for director in directors:
         print(director.nodes[0]['name'])

.. code-block::

    Tom Tykwer
    Lilly Wachowski
    Lana Wachowski


There are several important things to note here:

- ``r_type`` is a kwarg to ``.match()``
- ``nodes`` is a **set**, of: ``(NodeTo, NodeFrom)`` -- in this case, the "from" Node is ``None``, because that's the undefined data that we want to find.

In the "List all Tom Hanks movies..." example above only one of the ``nodes`` set is defined -- we were less e
xplicit with our requirements. For this kwarg the correct number of inputs in the set is *one* or *two*, in a **particular order**.


Tom Hanks' co-actors...
-----------------------

``cypher``:

.. code-block:: cypher

   MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name

``python``:

.. code-block:: python

    results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name')
    results.data()

    Out[]: [{'coActors.name': 'Bill Paxton'},
            {'coActors.name': 'Madonna'},
            {'coActors.name': 'Geena Davis'},
            {'coActors.name': 'Lori Petty'},
            {'coActors.name': 'Philip Seymour Hoffman'},
            ...
            {'coActors.name': 'Meg Ryan'},
            {'coActors.name': 'Parker Posey'},
            {'coActors.name': 'Dave Chappelle'},
            {'coActors.name': 'Greg Kinnear'},
            {'coActors.name': 'Meg Ryan'}]


| **How people are related to "Cloud Atlas"...**

``cypher``:

.. code-block:: cypher

    MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo

``python``:

.. code-block:: python

    results = graph.run('MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo')
    results.data()

    Out[]: [{'people.name': 'Halle Berry',
             'Type(relatedTo)': 'ACTED_IN',
             'relatedTo': ACTED_IN(Node('Person', born=1966, name='Halle Berry'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'), roles=['Luisa Rey', 'Jocasta Ayrs', 'Ovid', 'Meronym'])},
            {'people.name': 'Stefan Arndt',
             'Type(relatedTo)': 'PRODUCED',
             'relatedTo': PRODUCED(Node('Person', born=1961, name='Stefan Arndt'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'))},
            {'people.name': 'Hugo Weaving',
             'Type(relatedTo)': 'ACTED_IN',
             'relatedTo': ACTED_IN(Node('Person', born=1960, name='Hugo Weaving'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'), roles=['Bill Smoke', 'Haskell Moore', 'Tadeusz Kesselring', 'Nurse Noakes', 'Boardman Mephi', 'Old Georgie'])},
            {'people.name': 'Lilly Wachowski',
             'Type(relatedTo)': 'DIRECTED',
             'relatedTo': DIRECTED(Node('Person', born=1967, name='Lilly Wachowski'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'))},
            {'people.name': 'Tom Tykwer',
             'Type(relatedTo)': 'DIRECTED',
             'relatedTo': DIRECTED(Node('Person', born=1965, name='Tom Tykwer'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'))},
            {'people.name': 'Tom Hanks',
             'Type(relatedTo)': 'ACTED_IN',
             'relatedTo': ACTED_IN(Node('Person', born=1956, name='Tom Hanks'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'), roles=['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins'])},
            {'people.name': 'Jim Broadbent',
             'Type(relatedTo)': 'ACTED_IN',
             'relatedTo': ACTED_IN(Node('Person', born=1949, name='Jim Broadbent'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'), roles=['Vyvyan Ayrs', 'Captain Molyneux', 'Timothy Cavendish'])},
            {'people.name': 'Lana Wachowski',
             'Type(relatedTo)': 'DIRECTED',
             'relatedTo': DIRECTED(Node('Person', born=1965, name='Lana Wachowski'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'))},
            {'people.name': 'David Mitchell',
             'Type(relatedTo)': 'WROTE',
             'relatedTo': WROTE(Node('Person', born=1969, name='David Mitchell'), Node('Movie', released=2012, tagline='Everything is connected', title='Cloud Atlas'))}]


``Python`` has strengths far beyond ``cypher``, though ``cypher`` is also magically strong, so we're not too fussed by dropping back to native ``cypher`` here. We get the best of both worlds.

For example:

.. code-block:: python

    >>> results.to_table()
     people.name     | Type(relatedTo) | relatedTo
    -----------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------
     Halle Berry     | ACTED_IN        | (Halle Berry)-[:ACTED_IN {roles: ['Luisa Rey', 'Jocasta Ayrs', 'Ovid', 'Meronym']}]->(_64)
     Stefan Arndt    | PRODUCED        | (Stefan Arndt)-[:PRODUCED {}]->(_64)
     Hugo Weaving    | ACTED_IN        | (Hugo Weaving)-[:ACTED_IN {roles: ['Bill Smoke', 'Haskell Moore', 'Tadeusz Kesselring', 'Nurse Noakes', 'Boardman Mephi', 'Old Georgie']}]->(_64)
     Lilly Wachowski | DIRECTED        | (Lilly Wachowski)-[:DIRECTED {}]->(_64)
     Tom Tykwer      | DIRECTED        | (Tom Tykwer)-[:DIRECTED {}]->(_64)
     Tom Hanks       | ACTED_IN        | (Tom Hanks)-[:ACTED_IN {roles: ['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins']}]->(_64)
     Jim Broadbent   | ACTED_IN        | (Jim Broadbent)-[:ACTED_IN {roles: ['Vyvyan Ayrs', 'Captain Molyneux', 'Timothy Cavendish']}]->(_64)
     Lana Wachowski  | DIRECTED        | (Lana Wachowski)-[:DIRECTED {}]->(_64)
     David Mitchell  | WROTE           | (David Mitchell)-[:WROTE {}]->(_64)


Other possible completions are:

.. code-block:: python

    # pandas
    results.to_data_frame()
    results.to_series()

    # sympy
    results.to_matrix()

    # numpy
    results.to_ndarray()


.. image:: magic.gif

---

Solve
+++++


    You've heard of the classic "Six Degrees of Kevin Bacon"? That is simply a shortest path query called the "Bacon Path".

    1. Variable length patterns
    2. Built-in shortestPath() algorithm


Movies and actors up to 4 "hops" away from Kevin Bacon
------------------------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
    RETURN DISTINCT hollywood

``python``:

.. code-block:: python

    results = graph.run('MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood) RETURN DISTINCT hollywood')
    results.data()

    Out[]: [{'hollywood': Node('Person', born=1971, name='Paul Bettany')},
            {'hollywood': Node('Person', born=1956, name='Tom Hanks')},
            {'hollywood': Node('Person', born=1976, name='Audrey Tautou')},
            {'hollywood': Node('Person', born=1939, name='Ian McKellen')},
            ...
            {'hollywood': Node('Movie', released=2006, tagline='Break The Codes', title='The Da Vinci Code')},
            {'hollywood': Node('Person', born=1977, name='Liv Tyler')},
            {'hollywood': Node('Movie', released=1996, tagline='In every life there comes a time when that thing you dream becomes that thing you do', title='That Thing You Do')}]


Note that this return a lot of results:

.. code-block:: python

   results = graph.run('MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood) RETURN DISTINCT hollywood')
   len(results.data())

.. code-block::

    Out[]: 133


Bacon path, the shortest path of any relationships to Meg Ryan
--------------------------------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH p=shortestPath(
      (bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
    )
    RETURN p

``python``:

.. code-block:: python

    results = graph.run('MATCH p=shortestPath((bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})) RETURN p')
    results.data()

    Out[]: [{'p': Path(subgraph=Subgraph({Node('Person', born=1958, name='Kevin Bacon'), Node('Movie', released=1992, tagline="In the heart of the
           nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing
           to find the truth.", title='A Few Good Men'), Node('Person', born=1947, name='Rob Reiner'), Node('Movie', released=1998, tagline='At odds
           in life... in love on-line.', title='When Harry Met Sally'), Node('Person', born=1961, name='Meg Ryan')}, {ACTED_IN(Node('Person', born=1958,
           name='Kevin Bacon'), Node('Movie', released=1992, tagline="In the heart of the nation's capital, in a courthouse of the U.S. government, one
           man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title='A Few Good Men'), roles=['Capt. Jack
           Ross']), DIRECTED(Node('Person', born=1947, name='Rob Reiner'), Node('Movie', released=1992, tagline="In the heart of the nation's capital,
           in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.",
           title='A Few Good Men')), PRODUCED(Node('Person', born=1947, name='Rob Reiner'), Node('Movie', released=1998, tagline='At odds in life...
           in love on-line.', title='When Harry Met Sally')), ACTED_IN(Node('Person', born=1961, name='Meg Ryan'), Node('Movie', released=1998,
           tagline='At odds in life... in love on-line.', title='When Harry Met Sally'), roles=['Sally Albright'])}), sequence=(Node('Person',born=1958,
           name='Kevin Bacon'), ACTED_IN(Node('Person', born=1958, name='Kevin Bacon'), Node('Movie', released=1992, tagline="In the heart of the
           nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to
           find the truth.", title='A Few Good Men'), roles=['Capt. Jack Ross']), Node('Movie', released=1992, tagline="In the heart of the nation's
           capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the
           truth.", title='A Few Good Men'), DIRECTED(Node('Person', born=1947, name='Rob Reiner'), Node('Movie', released=1992, tagline="In the heart
           of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at
           nothing to find the truth.", title='A Few Good Men')), Node('Person', born=1947, name='Rob Reiner'), PRODUCED(Node('Person', born=1947,
           name='Rob Reiner'), Node('Movie', released=1998, tagline='At odds in life... in love on-line.', title='When Harry Met Sally')), Node('Movie',
           released=1998, tagline='At odds in life... in love on-line.', title='When Harry Met Sally'), ACTED_IN(Node('Person', born=1961, name='Meg
           Ryan'), Node('Movie', released=1998, tagline='At odds in life... in love on-line.', title='When Harry Met Sally'), roles=['Sally Albright']),
           Node('Person', born=1961, name='Meg Ryan')))}]


For more about shortest path:

https://neo4j.com/docs/developer-manual/current/cypher/clauses/match/#query-shortest-path

https://neo4j.com/docs/graph-algorithms/current/algorithms/shortest-path/

---

Recommend
+++++++++

    Let's recommend new co-actors for Tom Hanks. A basic recommendation approach is to find connections past an immediate neighborhood which are themselves well connected.

    For Tom Hanks, that means:

    1. Find actors that Tom Hanks hasn't yet worked with, but his co-actors have.
    2. Find someone who can introduce Tom to his potential co-actor.


Extend Tom Hanks co-actors, to find co-co-actors who haven't worked with Tom Hanks...
-------------------------------------------------------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
          (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors)
    WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors
    RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC

``python``:

.. code-block:: python

    results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors), (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors) WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC')
    results.data()

    Out[]: [{'Recommended': 'Tom Cruise', 'Strength': 5},
            {'Recommended': 'Cuba Gooding Jr.', 'Strength': 4},
            {'Recommended': 'Keanu Reeves', 'Strength': 4},
            {'Recommended': 'Carrie Fisher', 'Strength': 3},
            {'Recommended': 'Carrie-Anne Moss', 'Strength': 3},
            {'Recommended': 'Kelly McGillis', 'Strength': 3},
            {'Recommended': 'Val Kilmer', 'Strength': 3},
            {'Recommended': 'Laurence Fishburne', 'Strength': 3},
            {'Recommended': 'Jack Nicholson', 'Strength': 3},
            ...
            {'Recommended': 'Emil Eifrem', 'Strength': 1},
            {'Recommended': 'Christian Bale', 'Strength': 1},
            {'Recommended': 'Robin Williams', 'Strength': 1},
            {'Recommended': 'Demi Moore', 'Strength': 1},
            {'Recommended': 'Aaron Sorkin', 'Strength': 1},
            {'Recommended': 'Natalie Portman', 'Strength': 1}]


Find someone to introduce Tom Hanks to Tom Cruise
`````````````````````````````````````````````````

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
      (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"})
    RETURN tom, m, coActors, m2, cruise

``python``:

.. code-block:: python

    results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors), (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"}) RETURN tom, m, coActors, m2, cruise')
    results.data()

    Out[]: [{'tom': Node('Person', born=1956, name='Tom Hanks'),
             'm': Node('Movie', released=1990, tagline='A story of love, lava and burning desire.', title='Joe Versus the Volcano'),
             'coActors': Node('Person', born=1961, name='Meg Ryan'),
             'm2': Node('Movie', released=1986, tagline='I feel the need, the need for speed.', title='Top Gun'),
             'cruise': Node('Person', born=1962, name='Tom Cruise')},
            {'tom': Node('Person', born=1956, name='Tom Hanks'),
             'm': Node('Movie', released=1999, tagline="Walk a mile you'll never forget.", title='The Green Mile'),
             'coActors': Node('Person', born=1961, name='Bonnie Hunt'),
             'm2': Node('Movie', released=2000, tagline='The rest of his life begins now.', title='Jerry Maguire'),
             'cruise': Node('Person', born=1962, name='Tom Cruise')},
            {'tom': Node('Person', born=1956, name='Tom Hanks'),
             'm': Node('Movie', released=1998, tagline='At odds in life... in love on-line.', title="You've Got Mail"),
             'coActors': Node('Person', born=1961, name='Meg Ryan'),
             'm2': Node('Movie', released=1986, tagline='I feel the need, the need for speed.', title='Top Gun'),
             'cruise': Node('Person', born=1962, name='Tom Cruise')},
            {'tom': Node('Person', born=1956, name='Tom Hanks'),
             'm': Node('Movie', released=1995, tagline='Houston, we have a problem.', title='Apollo 13'),
             'coActors': Node('Person', born=1958, name='Kevin Bacon'),
             'm2': Node('Movie', released=1992, tagline="In the heart of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title='A Few Good Men'),
             'cruise': Node('Person', born=1962, name='Tom Cruise')},
            {'tom': Node('Person', born=1956, name='Tom Hanks'),
             'm': Node('Movie', released=1993, tagline='What if someone you never met, someone you never saw, someone you never knew was the only someone for you?', title='Sleepless in Seattle'),
             'coActors': Node('Person', born=1961, name='Meg Ryan'),
             'm2': Node('Movie', released=1986, tagline='I feel the need, the need for speed.', title='Top Gun'),
             'cruise': Node('Person', born=1962, name='Tom Cruise')}]

This allows you to use the full force of python on the results. That's pretty great.

---

Clean up
++++++++

When you're done experimenting, you can remove the movie data set.

Note:

1. Nodes can't be deleted if relationships exist
2. Delete both nodes and relationships together

*WARNING: This will remove all Person and Movie nodes!*


Delete all Movie and Person nodes, and their relationships
----------------------------------------------------------

``cypher``:

.. code-block:: cypher

    MATCH (n) DETACH DELETE n


``python``:

.. code-block:: python

   graph = Graph(password='[yoursekretpasswordhere]')
   len(graph.match())

   Out[]: 253

https://py2neo.org/v5/database.html#py2neo.database.Transaction.delete

.. code-block:: python

   # !! WARNING: This will remove all Person and Movie nodes !!

   graph.delete_all()

---

Confirm that the Movie Graph is gone
------------------------------------

``cypher``:

.. code-block:: cypher

   MATCH (n) RETURN n

``python``:

.. code-block:: python

   len(graph.match())

   Out[]: 0

.. code-block:: python

   graph.match().all()

   Out[]: []

---

This guide does not cover many interesting features of ``neo4j`` and ``py2neo`` such as the ability to ``update`` and ``merge``:

https://py2neo.org/v5/data.html

https://py2neo.org/v5/database.html

Importantly the ``ogm`` (**"Object Graph Mapper"**, analogous to the ``orm`` "Object Relational Mapper" used by many frameworks for traditional relational databases) feature is not covered here: https://py2neo.org/v5/ogm.html

~Fin
