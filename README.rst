*******************************
``import py2neo`` -- Quickstart
*******************************

Hopefully by this stage you have a little familiarity with both `python` and `neo4j`//`cypher`.

If you're familiar with the example provided by default when you select **"Jump into code: Movie Graph"**, also provided at: https://neo4j.com/developer/example-project/


Let's replicate

``:play movie-graph``

Here's the `cypher` from this tutorial:

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

This is the same in `py2neo`. There are 3 steps.

# Step 1: Connect to your GraphDB

# Step 2: Create your `Node` and `Relationship` objects

# Step 3: Commit your subgraphs ()


Step 1:
-------

.. code-block:: python

    from py2neo import Graph

    graph = Graph(password='[yoursekretpasswordhere]')


See further details about `graph`: https://py2neo.org/v4/database.html#py2neo.database.Graph


Step 2:
-------

`Node` and `Relationship` reference: https://py2neo.org/v4/data.html

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

Note: This looks great but **YOUR DB OBJECTS DO NOT EXIST YET!**. You need to commit them to the database.


Step 3:
-------

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
