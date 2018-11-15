*******************************
``import py2neo`` -- Quickstart
*******************************

Hopefully by this stage you have a little familiarity with both `python` and `neo4j`//`cypher`.

If you're familiar with the example provided by default when you select **"Jump into code: Movie Graph"**.

This is a smaller and similar dataset to the one provided here:
https://neo4j.com/developer/movie-database/

Let's replicate

``:play movie-graph``

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

Note: This looks great but **YOUR DB OBJECTS DO NOT EXIST YET!**.

They need to committed to the database per the next step.

Full ``Node`` and ``Relationship`` reference: https://py2neo.org/v4/data.html


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

First thing we need to connect to the database:

See reference here: https://py2neo.org/v4/matching.html

.. code-block:: python

    from py2neo import Graph, NodeMatcher
    graph = Graph(password='[yoursekretpasswordhere]')
    matcher = NodeMatcher(graph)

**Find the actor named "Tom Hanks"...**

`cypher`:

.. code-block:: cypher

    MATCH (tom {name: "Tom Hanks"}) RETURN tom

`python`:

.. code-block:: python

    >>> tom = matcher.match(name="Tom Hanks").first()
    >>> print(tom)
    (_69:Person {born: 1956, name: 'Tom Hanks'})


**Find the movie with title "Cloud Atlas"...**

`cypher`:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"}) RETURN cloudAtlas

`python`:

.. code-block:: python

    >>> cloudAtlas = matcher.match(title="Cloud Atlas").first()
    >>> print(cloudAtlas)
    (_105:Movie {released: 2012, tagline: 'Everything is connected', title: 'Cloud Atlas'})


**Find 10 people...**

`cypher`:

.. code-block:: cypher

    MATCH (people:Person) RETURN people.name LIMIT 10

`python`:

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

`cypher`:

.. code-block:: cypher

    MATCH (nineties:Movie) WHERE nineties.released >= 1990 AND nineties.released < 2000 RETURN nineties.title

`python`:

Note: watch the prefix **`"_."`** in the ``where`` statement.

.. code-block:: python

    >>> nineties = matcher.match("Movie").where('_.released >= 1990', '_.released < 2000')
    >>> print(list(nineties))
    [(_9:Movie {released: 1997, tagline: 'Evil has its winning ways', title: "The Devil's Advocate"}),
     (_13:Movie {released: 1992, tagline: "In the heart of the nation's capital, in a courthouse of the U.S. government, one man will stop at nothing to keep his honor, and one will stop at nothing to find the truth.", title: 'A Few Good Men'}),
     (_50:Movie {released: 1997, tagline: 'A comedy from the heart that goes for the throat.', title: 'As Good as It Gets'}),
     (_54:Movie {released: 1998, tagline: 'After life there is more. The end is just the beginning.', title: 'What Dreams May Come'}),
     (_60:Movie {released: 1999, tagline: 'First loves last. Forever.', title: 'Snow Falling on Cedars'}),
     (_65:Movie {released: 1998, tagline: 'At odds in life... in love on-line.', title: "You've Got Mail"}),
     (_71:Movie {released: 1993, tagline: 'What if someone you never met, someone you never saw, someone you never knew was the only someone for you?', title: 'Sleepless in Seattle'}),
     (_76:Movie {released: 1990, tagline: 'A story of love, lava and burning desire.', title: 'Joe Versus the Volcano'}),
     (_78:Movie {released: 1999, tagline: 'Welcome to the Real World', title: 'The Matrix'}),
     (_81:Movie {released: 1998, tagline: 'At odds in life... in love on-line.', title: 'When Harry Met Sally'}),
     (_85:Movie {released: 1996, tagline: 'In every life there comes a time when that thing you dream becomes that thing you do', title: 'That Thing You Do'}),
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

First thing we need to connect to the database:

See reference here: https://py2neo.org/v4/matching.html

.. code-block:: python

    from py2neo import Graph, RelationshipMatcher
    graph = Graph(password='[yoursekretpasswordhere]')
    r_matcher = RelationshipMatcher(graph)


**List all Tom Hanks movies...**

`cypher`:

.. code-block:: cypher

    MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies) RETURN tom,tomHanksMovies

`python`:

.. code-block:: python

   >>> r_matcher = RelationshipMatcher(graph)
   >>> tom = matcher.match(name="Tom Hanks").first()
   >>> tomHanksMovies = r_matcher.match(nodes=[tom], r_type="ACTED_IN")
   >>> print(list(tomHanksMovies))
   [(Tom Hanks)-[:ACTED_IN {roles: ['Jimmy Dugan']}]->(_181),
    (Tom Hanks)-[:ACTED_IN {roles: ['Rep. Charlie Wilson']}]->(_169),
    (Tom Hanks)-[:ACTED_IN {roles: ['Hero Boy', 'Father', 'Conductor', 'Hobo', 'Scrooge', 'Santa Claus']}]->(_180),
    (Tom Hanks)-[:ACTED_IN {roles: ['Chuck Noland']}]->(_160),
    (Tom Hanks)-[:ACTED_IN {roles: ['Jim Lovell']}]->(_154),
    (Tom Hanks)-[:ACTED_IN {roles: ['Paul Edgecomb']}]->(_140),
    (Tom Hanks)-[:ACTED_IN {roles: ['Dr. Robert Langdon']}]->(_121),
    (Tom Hanks)-[:ACTED_IN {roles: ['Zachry', 'Dr. Henry Goose', 'Isaac Sachs', 'Dermot Hoggins']}]->(_105),
    (Tom Hanks)-[:ACTED_IN {roles: ['Mr. White']}]->(_85),
    (Tom Hanks)-[:ACTED_IN {roles: ['Joe Banks']}]->(_76),
    (Tom Hanks)-[:ACTED_IN {roles: ['Sam Baldwin']}]->(_71),
    (Tom Hanks)-[:ACTED_IN {roles: ['Joe Fox']}]->(_65)]


**Who directed "Cloud Atlas"?**

`cypher`:

.. code-block:: cypher

    MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name

This is possible, but getting out of the scope of ``py2neo``, the following are all cases where falling back to native cypher is probably best.

`python`:

.. code-block:: python

    >>> results = graph.run('MATCH (cloudAtlas {title: "Cloud Atlas"})<-[:DIRECTED]-(directors) RETURN directors.name')
    >>> print(list(results))
    [<Record directors.name='Tom Tykwer'>,
     <Record directors.name='Lilly Wachowski'>,
     <Record directors.name='Lana Wachowski'>]

The following will produce the same result, although is less elegant:

`python`:

.. code-block:: python

    >>> cloudAtlas = matcher.match(title="Cloud Atlas").first()
    >>> directors = r_matcher.match(r_type="DIRECTED", nodes=(None, cloudAtlas))
    >>> for director in directors:
    >>>     print(director.nodes[0]['name'])
    Tom Tykwer
    Lilly Wachowski
    Lana Wachowski


**Tom Hanks' co-actors...**

`cypher`:

.. code-block:: cypher

   MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name

`python`:

.. code-block:: python

    >>> results = graph.run('MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors) RETURN coActors.name')
    >>> print(list(results))
    [<Record coActors.name='Bill Paxton'>,
     <Record coActors.name='Madonna'>,
     <Record coActors.name='Geena Davis'>,
     <Record coActors.name="Rosie O'Donnell">,
     <Record coActors.name='Lori Petty'>,
     <Record coActors.name='Philip Seymour Hoffman'>,
     <Record coActors.name='Julia Roberts'>,
     <Record coActors.name='Helen Hunt'>,
     <Record coActors.name='Bill Paxton'>,
     <Record coActors.name='Gary Sinise'>,
     <Record coActors.name='Ed Harris'>,
     <Record coActors.name='Kevin Bacon'>,
     <Record coActors.name='Patricia Clarkson'>,
     <Record coActors.name='Michael Clarke Duncan'>,
     <Record coActors.name='David Morse'>,
     <Record coActors.name='Sam Rockwell'>,
     <Record coActors.name='Gary Sinise'>,
     <Record coActors.name='Bonnie Hunt'>,
     <Record coActors.name='James Cromwell'>,
     <Record coActors.name='Ian McKellen'>,
     <Record coActors.name='Audrey Tautou'>,
     <Record coActors.name='Paul Bettany'>,
     <Record coActors.name='Jim Broadbent'>,
     <Record coActors.name='Hugo Weaving'>,
     <Record coActors.name='Halle Berry'>,
     <Record coActors.name='Liv Tyler'>,
     <Record coActors.name='Charlize Theron'>,
     <Record coActors.name='Meg Ryan'>,
     <Record coActors.name='Nathan Lane'>,
     <Record coActors.name='Victor Garber'>,
     <Record coActors.name="Rosie O'Donnell">,
     <Record coActors.name='Rita Wilson'>,
     <Record coActors.name='Bill Pullman'>,
     <Record coActors.name='Meg Ryan'>,
     <Record coActors.name='Steve Zahn'>,
     <Record coActors.name='Parker Posey'>,
     <Record coActors.name='Dave Chappelle'>,
     <Record coActors.name='Greg Kinnear'>,
     <Record coActors.name='Meg Ryan'>]


**How people are related to "Cloud Atlas"...**

`cypher`:

.. code-block:: cypher

    MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo

`python`:

.. code-block:: python

   >>> results = graph.run('MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"}) RETURN people.name, Type(relatedTo), relatedTo')
   >>> print(list(results))
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
