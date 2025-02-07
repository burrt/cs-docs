# Database Notes

* [ACID](database-notes.md#acid)
* [BASE](database-notes.md#base)
* [NoSQL vs SQL](database-notes.md#nosql-vs-sql)
* [Partitioning](database-notes.md#database-partitioning)
* [Indexing](database-notes.md#indexing)
* [DB Relationships](database-notes.md#relationships)

## ACID

* Atomic
  * "All or nothing" - no partial failure and the state is unchanged if the transaction fails
  * All operations in the statement will be applied if the transaction is successful, else they will all be rolled back
* Consistency
  * Changes to one valid state to another - must satisfy any defined consistency rules
* Isolation
  * Concurrent execution of transactions are isolated from each other and operate as if they were sequentially executed
  * This can be achieved with two-phase locking, timestamping, multi-versioning etc.
* Durability
  * Once committed, remain so even if power loss, crashes etc.

SQL databases often adhere to the ACID principles and therefore are well suited for systems that require strict state consistency.

## BASE

Basically Available, Soft state, Eventual consistency.

From [Stack Overflow](https://stackoverflow.com/questions/3342497/explanation-of-base-terminology):

> The CAP theorem states that a distributed computer system cannot guarantee all of the following three properties at the same time:
>
> * Consistency
> * Availability
> * Partition tolerance
>
> A BASE system gives up on consistency.
>
> Basically available indicates that the system does **guarantee availability**, in terms of the CAP theorem. Soft state indicates that the state of the system may change over time, even without input. This is because of the eventual consistency model. Eventual consistency indicates that the system will become consistent over time, given that the system doesn't receive input during that time.

## NoSQL vs SQL

From the [Azure article](https://azure.microsoft.com/en-au/overview/nosql-database/):

### NoSQL

* Gives up on Consistency in the CAP theorem
* Handling large, unrelated, indeterminate or rapidly changing data
* Performance and availability is more important than strict consistency
* Schema agnostic data, doesn't have to be strongly typed and the schema can evolve
* IoT, real-time analytics, content management
* Scales horizontally by sharding
* Well optimised for **reads** due to key-value based design and it is designed for horizontal scaling
* **Write** performance is very good due to weaker consistency
* e.g. social media (storing likes, comments), real-time analytics (various sources like IoT, fast writes)

### SQL

* Gives up on Availability in the CAP theorem
* Relational data that has logical requirements that can be identified in advance
* Schema of both app and database must be in-sync
* Complex querying of data
* Scales vertically
* **Reads** can be very optimised with indexes and the relationships know in advance however foreign keys and joins can degrade this quickly **Write** performance generally suffers due to ACID
* e.g. transactional/financial systems, inventory management

## Database Partitioning

| Type                | Description                                                                                                                                                                                                                                                                                                                      |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Vertical            | <p>By feature/domain objects. Usually a table is a domain object representation, instead, this will be a database.<br><strong>Cons</strong>: If the database grows too big, another partitioning scheme may need to be applied.</p>                                                                                              |
| Key-based (hashing) | <p>Could be split by an ID/value and that will be used for the server name/number. For example, if you have <code>n</code> servers, <code>mod(key, n)</code> will tell you where the data is stored.<br><strong>Cons</strong>: <code>n</code> is effectively fixed, if it changes, then the data migrations could be costly.</p> |
| Directory-based     | <p>Use a lookup table to determine where the data is stored.<br><strong>Cons</strong>: single point of failure and bottleneck, costly lookup at scale.</p>                                                                                                                                                                       |

## Indexing

A big topic so I will ony cover parts of it. [Old MS link about the indexes](https://technet.microsoft.com/en-us/library/jj835095\(v=sql.110\).aspx). For small tables, sometimes it can be better to not set an index to avoid the overhead of index performance cost and rely on the database to perform a simple table scan!

### Clustered vs non-clustered indexes

#### Clustered indexes

* There can be only one clustered index per table because the data rows themselves can only be sorted in one order. Usually this is the PK by default.
  * Sometimes it is better to change the default clustered index!
* A clustered index sorts and **stores** the data rows of the table based on the index key values.
* Useful for columns that are frequently used in **range** (e.g. `BETWEEN` or `IN`) queries or when retrieving large result sets or when relying on the _sorted values_.
* Suitable for tables where data is read more often than it is written, as clustered indexes can slow down insertions, updates, and deletions.

#### Non-clustered indexes

* A non-clustered index is a **separate structure** from the data rows. It maintains a copy of the indexed columns along with pointers to the actual data rows.
* Multiple non-clustered indexes can exist on a single table.
* The physical order of the rows on disk is not affected by the non-clustered index.
* Can be used on columns that are frequently searched but not sorted, such as columns used in `WHERE` clauses.
* Suitable for columns frequently used in search conditions, joins, and for improving the performance of read operations that don't involve sorting.

## Normalisation theory

### Relationships

These O'Reilly chapters cover the basics, including the symbols that are often used:

* [Learning MySQL - The Entity Relationship Model](https://learning.oreilly.com/library/view/learning-mysql/0596008643/ch04s03.html)
* [Learning MySQL - Examples](https://learning.oreilly.com/library/view/learning-mysql/0596008643/ch04s04.html)

![ERD symbols](https://i.imgur.com/eFPqJxlm.png)

#### 1:1

* One country, one capital city
* One person, one passport

A 1:1 relationship can be enforced at the database level using foreign keys with a unique constraint.

#### 1:M

* One building, many apartments
* One apartment only relates to one building

The latter defines it as a 1:M. Other examples:

* One company has many employees, and one employee is related to one company
* Authors to books

#### M:M

* One tenant can own many apartments, **and** one apartment can be owned by many tenants
* Therefore many tenants can own many apartments

A M:M relationship would be to have an table with columns of the two entities with the relation. Other examples:

* Customer and products - one customer is related/buys to many products, and one product is bought/related to many customers. The unique key would be the order ID.

### Superkey

* A superkey `SK` specifies a **uniqueness** constraint that **no two** distinct tuples in any state `r` of `R` can have the **same** value for SK.
* Every relation has at least **one default** superkey — the set of **all** its attributes.
* A superkey can have redundant attributes, however, so a more useful concept is that of a **key**, which has **no** redundancy.

### Key

* A key `K` of a relation schema `R` is a superkey of `R` with the additional property that **removing** any attribute `A` from `K` leaves a set of attributes `K'` that is **not** a superkey of `R` any more.

Hence, a key satisfies **two** properties:

1. **Two distinct** tuples in any state of the relation **cannot** have identical values for (all) the attributes in the key. This first property also applies to a superkey.
2. It is a **minimal** superkey i.e. a superkey from which we **cannot remove** any attributes and **still** have the uniqueness constraint in **condition 1 hold**. This property is **not** required by a superkey.

### Candidate key

* A relation can have more than one key, these are called candidate keys e.g. If a relation contains 2 unique serials.

### Terms

* **DBMS**: Database Management System is a collection of programs that enables users to create and maintain a database.
* **Domain** `D` is an atomic value. Each domain is specified a data _type_ or _format_.
* **Relation Schema** `R(A1, A2, Ai...)`:
  * `Ai`: **attribute** is the name of a role played by some domain `D`.
  * `R`: the name of the **schema**.
  * It is used to describe a relation called `R`.
* **Relation** or Relation state `r` or `r(R)` of the relation schema `R(A1, A2, Ai...)` is a set of tuples:
  * `r = {t1, t2, t3..}`, a set of **n-tuples**.
  * Each n-tuple `t` is an **ordered list** of **n** values `t = <v1, v2, ..., vn>`, where each value vi, 1 ≤ i ≤ n, is an element of `dom(Ai)` or is a special `NULL` value.
  * The **ith** value in tuple `t`, which corresponds to the attribute `Ai`, is referred to as `t[Ai]`.

Example:

```
Relation name = Student
Attributes = Name, Ssn, Address
Tuples = row

+---------+---------+---------+
| Name    | Ssn     | Address | // attributes with dom(Ai)
+---------+---------+---------+
| Bob     | 123     | Sydney  | // tuple
+---------+---------+---------+
| Alice   | 321     | Coona   |
+---------+---------+---------+
...

t[Name] = <'Bob'>           // first tuple only
t[Name, Ssn] = <'Bob, 123'>
```
