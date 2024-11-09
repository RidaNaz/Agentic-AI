# Graph Query Language GQL

- A graph query language is a specialized tool designed to interact with ***graph databases***, allowing users to **query** and **manipulate** data in a *graph-like structure*.
- At its core, it enables direct interaction with ***nodes*** and ***edges***, the *fundamental components of graph data models*.
- This approach provides a more intuitive way to work with **highly connected data** *compared to traditional relational database queries*.

## Fundamentals of graph databases

### 1. Nodes and edges

- **Nodes** are the fundamental entities within a graph, representing ***people, places, objects***, etc.
- **Edges** capture the ***relationships between these nodes***, adding context and meaning to the data.

### 2. Properties:

- **Both nodes and edges** can have properties associated with them.
- These properties ***store additional information*** about the entities and relationships they represent.
- For instance,
    - **Person node** might have properties like ***name, age, and location***,
    - while a **friendship edge** might have a property like ***startDate***.

### 3. Labels:

- **Nodes and edges** can also be assigned labels, ***categorizing them into different types***.
- This allows for more ***granular querying*** and ***filtering*** based on **specific node or edge types**.
- For example, you might have labels like ***Person, City, and FRIEND*** to distinguish different entities and relationships in your graph.

***In GQL, a graph is set of zero or more nodes and edges where:***

**A Node has:**
- A **label set** with zero or more ***unique label names***,
- A **property set** with zero or more ***name-value pairs with unique names***.

**An Edge has:**
- A **label set** with zero or more ***unique label names***,
- A **property set** with zero or more ***name-value pairs with unique names***,
- ***Two endpoint nodes*** in the same graph,
- An indication of *whether* the edge is **directed** or **undirected** ,
- When a **directed edge**, ***one endpoint node is the source*** and ***the other node is the destination***.

## Graph Pattern Matching (GPM)

- GQL uses a powerful GPM language to simplify complex data analysis.
- For instance, to find all nodes with a ***one-hop relationship*** to a node named **'Avery'**, the query would be:

```gql
MATCH (a {firstname: 'Avery'})-[b]->(c) RETURN a, b, c
```

This query allows the user to retrieve relationships without needing to specify them.

## Quantified Path Patterns (QPP)

- GQL supports complex queries like QPP, which can find paths of ***varying lengths***.
- An example query to find paths ***up to five hops long*** is:

```gql
MATCH ((a)-[r]->(b)){1,5} RETURN a, r, b
```

## Data Manipulation

- Nodes and edges can be ***added*** using the **INSERT** statement.
- For example, to insert a node labeled as **'Pet'** with ***properties***:

```gql
INSERT (:Pet {name: 'Unique', pettype: 'Dog'})
```
- To insert **two nodes** and **an edge** between them, the query could be:

```gql
INSERT (:Person {firstname: 'Avery', lastname: 'Stare', joined: date("2022-08-23")})
-[:LivesIn {since: date("2022-07-15")}]->
(:City {name: 'Granville', state: 'OH', country: 'USA'})
```

## Modifying and Deleting Data

- GQL allows **modification** of data ***by setting or removing properties***.
- For example:

```gql
MATCH (d:Pet) WHERE d.name="Unique" SET d.weight_kg=26
MATCH (e {lastname: 'Stare'}) REMOVE e.joined
```

- Deletion is done by ***detaching*** and ***then removing nodes and relationships***:

```gql
MATCH (a {firstname: 'Avery'})-[b]->(c) DETACH DELETE a, c
```

## Transactions
GQL supports serializable transactions with *explicit* or *implicit*  **START TRANSACTION** statements, and can be terminated with ***COMMIT*** or ***ROLLBACK***.

## Schema-free and Fixed-schema Graphs

In GQL, both schema-free and fixed-schema graphs are supported:

### 1. Schema-Free Graphs:
Allow any data insertion without restrictions, offering flexibility and quick setup but relying on developers to maintain data consistency.

### 2. Fixed-Schema Graphs:
Use a graph type to define allowed node and edge types, enforcing a structured schema. This ensures that any data entered must align with the specified structure, ideal for controlled, consistent data handling.

### Example

***1. Define a Graph Type (`fraud_TYPE`):***

- This `GRAPH TYPE` defines two node types (`Customer` and `Account`) with specified attributes:
    - `Customer` node has `id` and `name` (both strings).
    - `Account` node has `no` (account number) and `acct_type`.
- Two edge types are defined:
    - `HOLDS` connects `Customer` to `Account`.
    - `TRANSFER`, an edge with an `amount` attribute (integer), represents a transfer between accounts.

```gql
CREATE GRAPH TYPE /MyFolder/control/fraud_TYPE
  (customer:Customer => {id::STRING, name::STRING}),
  (account:Account => {no::STRING, acct_type::STRING }),
  (customer)-[:HOLDS]->(account),
  (account)-[:TRANSFER {amount::INTEGER}]->(account)
```

***2. Create a Graph Using the Graph Type:***

- A new graph named `fraud` is created, constrained by the structure defined in `fraud_TYPE`.

```gql
CREATE GRAPH /MyFolder/control/fraud   // graph name is `fraud`
  TYPED /MyFolder/control/fraud_TYPE   // graph type is `fraud_TYPE`
```

***3. Insert Data:***

- Using the ***Data Manipulation Language (DML)***, nodes and edges are added to the `fraud` graph.
- Data entries must conform to `fraud_TYPE`:
    - Two customers (`Doe` and `Reiss`) and two accounts are added.
    - Relationships are established:
        - `Doe` holds account `25673890`, and `Reiss` holds account `05663981`.
        - A transfer of 5000 units from `Doe`'s account to `Reiss`'s account is represented by the `TRANSFER` edge.

```gql
USE GRAPH fraud
INSERT
  (d:Customer {id: 'AB23', name: 'Doe'}),
  (r:Customer {id: 'CH45', name: 'Reiss'}),
  (a1:Account {no: '25673890', type: 'C'}),
  (a2:Account {no: '05663981', type: 'C'}),
  (d)-[:HOLDS]->(a1),
  (r)-[:HOLDS]->(a2),
  (a1)-[:TRANSFER {amount: 5000}]->(a2)
```

***4. Query Data:***

- A query retrieves information about customers and accounts involved in a transfer, displaying the customer names, account numbers, and transfer amount.

```gql
USE GRAPH fraud
MATCH
  (c1:Customer)-[:HOLDS]->(a1:Account)
  -[t:TRANSFER]->
  (a2:Account)<-[:HOLDS]-(c2:Customer)
RETURN c1.name, a1.no, t.amount, c2.name, a2.no
```

***This structure allows GQL to offer flexibility with schema-free graphs and data integrity with fixed-schema graphs.***

[Use Editor for GQL](https://opengql.github.io/editor/)