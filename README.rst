*******************************
``import py2neo`` -- Quickstart
*******************************

As everyone knows:

``python`` == best

``neo4j`` == amazing

.. image:: both.gif

``python`` + ``neo4j`` = ``Py2Neo`` <3

We are so grateful to https://github.com/technige for combining them as https://github.com/technige/py2neo.

Hopefully by this stage you have a little familiarity with both ``python`` and ``neo4j``\/``cypher``.

If you'd like to know more about ``Python`` there is a whole internet of resources, we like https://realpython.com/start-here/ or https://docs.python-guide.org/intro/learning/

If you'd like to know more about ``Neo4J``, most people will become familiar with the materials we use here by the simple browser interface that is easy to install using the instructions here: https://neo4j.com/developer/guide-neo4j-browser/

If this url works for you to access Neo4J: http://127.0.0.1:7474/browser/ , you ready for this quickstart! The code used below come from **"Jump into code: Movie Graph"**.

This is a smaller and similar dataset to the one provided here:
https://neo4j.com/developer/movie-database/

Let's replicate: ``:play movie-graph``

Create
++++++

Here's the ``cypher`` from this tutorial:

.. code-block:: cypher

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
    CREATE
      (Keanu)-[:ACTED_IN {roles:['Neo']}]->(TheMatrix),
      (Carrie)-[:ACTED_IN {roles:['Trinity']}]->(TheMatrix),
      (Laurence)-[:ACTED_IN {roles:['Morpheus']}]->(TheMatrix),
      (Hugo)-[:ACTED_IN {roles:['Agent Smith']}]->(TheMatrix),
      (LillyW)-[:DIRECTED]->(TheMatrix),
      (LanaW)-[:DIRECTED]->(TheMatrix),
      (JoelS)-[:PRODUCED]->(TheMatrix)

This is the same in ``py2neo``. There are 3 steps.

# Step 1: Connect to your GraphDB

# Step 2: Create your ``Node`` and ``Relationship`` objects

# Step 3: Commit your Subgraphs (https://py2neo.org/v4/data.html#subgraph-objects)


Step 1: Connect
---------------

.. code-block:: python

    from py2neo import Graph

    graph = Graph(password='[yoursekretpasswordhere]')


There are plenty of options for connecting to your database if this implementation doesn't work for you. See the reference here: https://py2neo.org/v4/database.html#py2neo.database.Graph


Step 2: Create ``Node`` and ``Relationship`` Subgraphs
------------------------------------------------------

Full ``Node`` and ``Relationship`` reference: https://py2neo.org/v4/data.html

.. code-block:: python

    from py2neo import Node, Relationship

    # Movie
    TheMatrix = Node("Movie", title='The Matrix', released=1999,
                     tagline='Welcome to the Real World')

    # Persons
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

    # Relationships with roles property
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

They need to committed to the database per the next step.


Step 3: Commit
--------------

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


The gist of the full dataset can be found here: https://gist.github.com/elena/733275bd55fba0a48cd885fe0427e5d4


Find
++++

Example queries for finding individual nodes.

First thing we need to connect to the database:

See reference here: https://py2neo.org/v4/matching.html

.. code-block:: python

    from py2neo import Graph, NodeMatcher
    graph = Graph(password='[yoursekretpasswordhere]')
    matcher = NodeMatcher(graph)

**Find the actor named "Tom Hanks"...**

``cypher``:

.. code-block:: cypher

    MATCH (tom {name: "Tom Hanks"}) RETURN tom

``python``:

.. code-block:: python

    >>> tom = matcher.match(name="Tom Hanks").first()
    >>> print(tom)
    (_69:Person {born: 1956, name: 'Tom Hanks'})


**Find the movie with title "Cloud Atlas"...**

``cypher``:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"}) RETURN cloudAtlas

``python``:

.. code-block:: python

    >>> cloudAtlas = matcher.match(title="Cloud Atlas").first()
    >>> print(cloudAtlas)
    (_105:Movie {released: 2012, tagline: 'Everything is connected', title: 'Cloud Atlas'})


**Find 10 people...**

``cypher``:

.. code-block:: cypher

    MATCH (people:Person) RETURN people.name LIMIT 10

``python``:

.. code-block:: python

    >>> people = matcher.match("Person").limit(10)
    >>> print(people)
    <py2neo.matching.NodeMatch object at 0x7fc00046ac18>
    >>> print(list(people))
    [(_0:Person {born: 1967, name: 'Carrie-Anne Moss'}),
     (_1:Person {born: 1961, name: 'Laurence Fishburne'}),
     (_2:Person {born: 1960, name: 'Hugo Weaving'}),
     (_3:Person {born: 1967, name: 'Lilly Wachowski'}),
     (_4:Person {born: 1965, name: 'Lana Wachowski'}),
     (_5:Person {born: 1952, name: 'Joel Silver'}),
     (_6:Person {born: 1978, name: 'Emil Eifrem'}),
     (_10:Person {born: 1975, name: 'Charlize Theron'}),
     (_11:Person {born: 1940, name: 'Al Pacino'}),
     (_12:Person {born: 1944, name: 'Taylor Hackford'})]


**Find movies released in the 1990s...**

``cypher``:

.. code-block:: cypher

    MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title

``python``:

Note: watch the prefix **`"_."`** in the ``where`` statement.

.. code-block:: python

    >>> nineties = matcher.match("Movie").where('_.released >= 1990', '_.released < 2000')
    >>> print(list(nineties))
    [(_9:Movie {released: 1997, tagline: 'Evil has its winning ways', title: "The Devil's Advocate"}),
     (_13:Movie {released: 1992, tagline: "In the heart of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title: 'A Few Good Men'}),
     (_50:Movie {released: 1997, tagline: 'A comedy from the heart that goes for the throat.', title: 'As Good as It Gets'}),
     ...
     (_95:Movie {released: 1996, tagline: 'Come as you are', title: 'The Birdcage'}),
     (_97:Movie {released: 1992, tagline: "It's a hell of a thing, killing a man", title: 'Unforgiven'}),
     (_100:Movie {released: 1995, tagline: 'The hottest data on earth. In the coolest head in town', title: 'Johnny Mnemonic'}),
     (_140:Movie {released: 1999, tagline: "Walk a mile you'll never forget.", title: 'The Green Mile'}),
     (_151:Movie {released: 1992, tagline: "He didn't want law. He wanted justice.", title: 'Hoffa'}),
     (_154:Movie {released: 1995, tagline: 'Houston, we have a problem.', title: 'Apollo 13'}),
     (_157:Movie {released: 1996, tagline: "Don't Breathe. Don't Look Back.", title: 'Twister'}),
     (_167:Movie {released: 1999, tagline: "One robot's 200 year journey to become an ordinary man.", title: 'Bicentennial Man'}),
     (_181:Movie {released: 1992, tagline: 'Once in a lifetime you get a chance to do something different.', title: 'A League of Their Own'})]

See full reference here: https://py2neo.org/v4/matching.html


Query
+++++

Finding patterns within the graph.

1. Actors are people who acted in movies
2. Directors are people who directed a movie
3. What other relationships exist?

See reference here: https://py2neo.org/v4/matching.html

``RelationshipMatcher`` needs to be imported and instantiated:

.. code-block:: python

    from py2neo import Graph, RelationshipMatcher
    graph = Graph(password='[yoursekretpasswordhere]')
    r_matcher = RelationshipMatcher(graph)


**List all Tom Hanks movies...**

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies

``python``:

.. code-block:: python

   >>> r_matcher = RelationshipMatcher(graph)
   >>> tom = matcher.match(name="Tom Hanks").first()
   >>> tomHanksMovies = r_matcher.match(nodes=[tom], r_type="ACTED_IN")
   >>> print(list(tomHanksMovies))
   [(Tom Hanks)-[:ACTED_IN {roles: ['Jimmy Dugan']}]->(_181),
    (Tom Hanks)-[:ACTED_IN {roles: ['Rep. Charlie Wilson']}]->(_169),
    (Tom Hanks)-[:ACTED_IN {roles: ['Hero Boy', 'Father', 'Conductor', 'Hobo', 'Scrooge', 'Santa Claus']}]->(_180),
    (Tom Hanks)-[:ACTED_IN {roles: ['Chuck Noland']}]->(_160),
    ...
    (Tom Hanks)-[:ACTED_IN {roles: ['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins']}]->(_105),
    (Tom Hanks)-[:ACTED_IN {roles: ['Mr. White']}]->(_85),
    (Tom Hanks)-[:ACTED_IN {roles: ['Joe Banks']}]->(_76),
    (Tom Hanks)-[:ACTED_IN {roles: ['Joe Fox']}]->(_65)]


**Who directed "Cloud Atlas"?**

``cypher``:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name

This is possible, but getting out of the scope of ``py2neo``, the following are all cases where falling back to native cypher is probably best.

``python``:

.. code-block:: python

    >>> results = graph.run('MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name')
    >>> results.data()
    [{'directors.name': 'Tom Tykwer'},
     {'directors.name': 'Lilly Wachowski'},
     {'directors.name': 'Lana Wachowski'}]

The following will produce the same result, although is less elegant:

``python``:

.. code-block:: python

    >>> cloudAtlas = matcher.match(title="Cloud Atlas").first()
    >>> directors = r_matcher.match(r_type="DIRECTED", nodes=(None, cloudAtlas))
    >>> for director in directors:
    >>>     print(director.nodes[0]['name'])
    Tom Tykwer
    Lilly Wachowski
    Lana Wachowski


**Tom Hanks' co-actors...**

``cypher``:

.. code-block:: cypher

   MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name

``python``:

.. code-block:: python

    >>> results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name')
    >>> results.data()
     [{'coActors.name': 'Bill Paxton'},
      {'coActors.name': 'Madonna'},
      {'coActors.name': 'Geena Davis'},
      {'coActors.name': "Rosie O'Donnell"},
      {'coActors.name': 'Lori Petty'},
      {'coActors.name': 'Philip Seymour Hoffman'},
      ...
      {'coActors.name': 'Meg Ryan'},
      {'coActors.name': 'Steve Zahn'},
      {'coActors.name': 'Parker Posey'},
      {'coActors.name': 'Dave Chappelle'},
      {'coActors.name': 'Greg Kinnear'},
      {'coActors.name': 'Meg Ryan'}]


**How people are related to "Cloud Atlas"...**

``cypher``:

.. code-block:: cypher

    MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo

``python``:

.. code-block:: python

   >>> results = graph.run('MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo')
   >>> results.data()
   [<Record people.name='Jessica Thompson' Type(relatedTo)='REVIEWED' relatedTo=(Jessica Thompson)-[:REVIEWED {rating: 95, summary: 'An amazing journey'}]->(_105)>,
    <Record people.name='Stefan Arndt' Type(relatedTo)='PRODUCED' relatedTo=(Stefan Arndt)-[:PRODUCED {}]->(_105)>,
    <Record people.name='Tom Tykwer' Type(relatedTo)='DIRECTED' relatedTo=(Tom Tykwer)-[:DIRECTED {}]->(_105)>,
    <Record people.name='Lilly Wachowski' Type(relatedTo)='DIRECTED' relatedTo=(Lilly Wachowski)-[:DIRECTED {}]->(_105)>,
    <Record people.name='Lana Wachowski' Type(relatedTo)='DIRECTED' relatedTo=(Lana Wachowski)-[:DIRECTED {}]->(_105)>,
    <Record people.name='David Mitchell' Type(relatedTo)='WROTE' relatedTo=(David Mitchell)-[:WROTE {}]->(_105)>,
    <Record people.name='Jim Broadbent' Type(relatedTo)='ACTED_IN' relatedTo=(Jim Broadbent)-[:ACTED_IN {roles: ['Vyvyan Ayrs', 'Captain Molyneux', 'Timothy Cavendish']}]->(_105)>,
    <Record people.name='Hugo Weaving' Type(relatedTo)='ACTED_IN' relatedTo=(Hugo Weaving)-[:ACTED_IN {roles: ['Bill Smoke', 'Haskell Moore', 'Tadeusz Kesselring', 'Nurse Noakes', 'Boardman Mephi', 'Old Georgie']}]->(_105)>,
    <Record people.name='Halle Berry' Type(relatedTo)='ACTED_IN' relatedTo=(Halle Berry)-[:ACTED_IN {roles: ['Luisa Rey', 'Jocasta Ayrs', 'Ovid', 'Meronym']}]->(_105)>,
    <Record people.name='Tom Hanks' Type(relatedTo)='ACTED_IN' relatedTo=(Tom Hanks)-[:ACTED_IN {roles: ['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins']}]->(_105)>]



Solve
+++++


You've heard of the classic "Six Degrees of Kevin Bacon"? That is simply a shortest path query called the "Bacon Path".

1. Variable length patterns
2. Built-in shortestPath() algorithm

**Movies and actors up to 4 "hops" away from Kevin Bacon**

``cypher``:

.. code-block:: cypher

    MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood)
    RETURN DISTINCT hollywood

``python``:

.. code-block:: python

    >>> results = graph.run('MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..4]-(hollywood) RETURN DISTINCT hollywood')
    >>> results.data()
    [<Record hollywood=(_149:Person {born: 1969, name: 'Michael Sheen'})>, ... <Record hollywood=(_29:Person {born: 1959, name: 'Val Kilmer'})>, <Record hollywood=(_28:Person {born: 1957, name: 'Kelly McGillis'})>, <Record hollywood=(_27:Movie {released: 1986, tagline: 'I feel the need, the need for speed.', title: 'Top Gun'})>]
    >>> len(results.data())
    135

**Bacon path, the shortest path of any relationships to Meg Ryan**

``cypher``:

.. code-block:: cypher

    MATCH p=shortestPath(
      (bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
    )
    RETURN p

``python``:

.. code-block:: python

    >>> results = graph.run('MATCH p=shortestPath((bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})) RETURN p')
    >>> results.data()
    [{'p': (Kevin Bacon)-[:ACTED_IN {roles: ['Jack Swigert']}]->(_154)<-[:ACTED_IN {roles: ['Jim Lovell']}]-(Tom Hanks)-[:ACTED_IN {roles: ['Joe Banks']}]->(_76)<-[:ACTED_IN {roles: ['DeDe', 'Angelica Graynamore', 'Patricia Graynamore']}]-(Meg Ryan)}]



For more information on shortest path:

https://neo4j.com/docs/developer-manual/current/cypher/clauses/match/#query-shortest-path

https://neo4j.com/docs/graph-algorithms/current/algorithms/shortest-path/


Recommend
+++++++++

Let's recommend new co-actors for Tom Hanks. A basic recommendation approach is to find connections past an immediate neighborhood which are themselves well connected.

For Tom Hanks, that means:

1. Find actors that Tom Hanks hasn't yet worked with, but his co-actors have.
2. Find someone who can introduce Tom to his potential co-actor.

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
          (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors)
    WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors
    RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC

``python``:

.. code-block:: python


    >>> results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors), (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cocoActors) WHERE NOT (tom)-[:ACTED_IN]->()<-[:ACTED_IN]-(cocoActors) AND tom <> cocoActors RETURN cocoActors.name AS Recommended, count(*) AS Strength ORDER BY Strength DESC')
    >>> results.data()
    [{'Recommended': 'Tom Cruise', 'Strength': 5},
     {'Recommended': 'Zach Grenier', 'Strength': 5},
     {'Recommended': 'Cuba Gooding Jr.', 'Strength': 4},
     {'Recommended': 'Keanu Reeves', 'Strength': 4},
     {'Recommended': 'Carrie Fisher', 'Strength': 3},
     {'Recommended': 'Carrie-Anne Moss', 'Strength': 3},
     {'Recommended': 'Kelly McGillis', 'Strength': 3},
     {'Recommended': 'Val Kilmer', 'Strength': 3},
     {'Recommended': 'Anthony Edwards', 'Strength': 3},
     {'Recommended': 'Laurence Fishburne', 'Strength': 3},
     {'Recommended': 'Jack Nicholson', 'Strength': 3},
     ...
     {'Recommended': 'Emil Eifrem', 'Strength': 1},
     {'Recommended': 'Stephen Rea', 'Strength': 1},
     {'Recommended': 'John Hurt', 'Strength': 1},
     {'Recommended': 'Christian Bale', 'Strength': 1},
     {'Recommended': 'Robin Williams', 'Strength': 1},
     {'Recommended': 'Demi Moore', 'Strength': 1},
     {'Recommended': 'Aaron Sorkin', 'Strength': 1},
     {'Recommended': 'Jonathan Lipnicki', 'Strength': 1},
     {'Recommended': 'Jay Mohr', 'Strength': 1},
     {'Recommended': 'Regina King', 'Strength': 1},
     {'Recommended': 'Natalie Portman', 'Strength': 1}]

``cypher``:

.. code-block:: cypher

    MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors),
      (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"})
    RETURN tom, m, coActors, m2, cruise

``python``:

.. code-block:: python

    >>> results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors), (coActors)-[:ACTED_IN]->(m2)<-[:ACTED_IN]-(cruise:Person {name:"Tom Cruise"}) RETURN tom, m, coActors, m2, cruise')
    >>> results.data()
    [{'tom': (_69:Person {born: 1956, name: 'Tom Hanks'}),
    'm': (_154:Movie {released: 1995, tagline: 'Houston, we have a problem.', title: 'Apollo 13'}),
    'coActors': (_17:Person {born: 1958, name: 'Kevin Bacon'}),
    'm2': (_13:Movie {released: 1992, tagline: "In the heart of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title: 'A Few Good Men'}),
    'cruise': (_14:Person {born: 1962, name: 'Tom Cruise'})},
    {'tom': (_69:Person {born: 1956, name: 'Tom Hanks'}),
    'm': (_140:Movie {released: 1999, tagline: "Walk a mile you'll never forget.", title: 'The Green Mile'}),
    'coActors': (_40:Person {born: 1961, name: 'Bonnie Hunt'}),
    'm2': (_35:Movie {released: 2000, tagline: 'The rest of his life begins now.', title: 'Jerry Maguire'}),
    'cruise': (_14:Person {born: 1962, name: 'Tom Cruise'})},
    {'tom': (_69:Person {born: 1956, name: 'Tom Hanks'}),
    'm': (_76:Movie {released: 1990, tagline: 'A story of love, lava and burning desire.', title: 'Joe Versus the Volcano'}),
    'coActors': (_32:Person {born: 1961, name: 'Meg Ryan'}),
    'm2': (_27:Movie {released: 1986, tagline: 'I feel the need, the need for speed.', title: 'Top Gun'}),
    'cruise': (_14:Person {born: 1962, name: 'Tom Cruise'})},
    {'tom': (_69:Person {born: 1956, name: 'Tom Hanks'}),
    'm': (_71:Movie {released: 1993, tagline: 'What if someone you never met, someone you never saw, someone you never knew was the only someone for you?', title: 'Sleepless in Seattle'}),
    'coActors': (_32:Person {born: 1961, name: 'Meg Ryan'}),
    'm2': (_27:Movie {released: 1986, tagline: 'I feel the need, the need for speed.', title: 'Top Gun'}),
    'cruise': (_14:Person {born: 1962, name: 'Tom Cruise'})},
    {'tom': (_69:Person {born: 1956, name: 'Tom Hanks'}),
    'm': (_65:Movie {released: 1998, tagline: 'At odds in life... in love on-line.', title: "You've Got Mail"}),
    'coActors': (_32:Person {born: 1961, name: 'Meg Ryan'}),
    'm2': (_27:Movie {released: 1986, tagline: 'I feel the need, the need for speed.', title: 'Top Gun'}),
    'cruise': (_14:Person {born: 1962, name: 'Tom Cruise'})}]


---

Clean up
++++++++

When you're done experimenting, you can remove the movie data set.

Note:

1. Nodes can't be deleted if relationships exist
2. Delete both nodes and relationships together

*WARNING: This will remove all Person and Movie nodes!*


| **Delete all Movie and Person nodes, and their relationships**

``cypher``:

.. code-block:: cypher

    MATCH (n) DETACH DELETE n


``python``:

 https://py2neo.org/v4/database.html#py2neo.database.Transaction.delete

.. code-block:: python

    >>> graph = Graph(password='[yoursekretpasswordhere]')
    >>> len(graph.match())
    253
    >>> graph.delete_all()


| **Note you only need to compare property values like this when first creating relationships**
| **Prove that the Movie Graph is gone**

``cypher``:

.. code-block:: cypher

    MATCH (n) RETURN n

.. code-block:: python

    >>> len(graph.match())
    0

    >>> list(graph.match())
    []

