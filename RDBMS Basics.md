# Data Modelling

Kinds of data models:
 - **Conceptual**: Represents a high-level, abstract view of the organizational data. It focuses on what data is needed and how it should be organized. E.G. A diagram showing entities like "Customer," "Order," and "Product" with their relationships (e.g., "Customer places Order").
 - **Logical**: Refines the conceptual model to include more detail while remaining independent of any specific database technology. E.G. A diagram showing entities "Customer" with attributes like "CustomerID," "Name," and "Email," and relationships with entities like "Order" and "Product."
 - **Physical**: Internal file storage within a specific RDBMS.

## Entity Relationship Model (ERM) & Entity Relationship Diagram (ERD)

ER has three major modelling constructs:
 - **entity**: objects (“things”) in your world that you are interested, e.g. Student, Course, Class, ...
 - **attribute**: data item describing a property of interest, e.g. Student (StudentID, FirstName, LastName, ...)
 - **relationship**: association between entities (objects), e.g. students take courses

 Example ERD:

 ![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/erd_example.png)

Attribute types:
 - **Composite attribute**: An attribute that can be broken down into smaller, meaningful sub-parts (components).
 - **Derived attribute**: An attribute whose value can be calculated or derived from other attributes.
 - **Multivalued attribute**: An attribute that can have multiple values for a single entity instance.
 - **Simple attribute**: An attribute that cannot be broken down further; it has a single, atomic value. 

ERD representation:

 ![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/attribute_type.png)

Entity set:

Definition: An **entity set** is a collection of similar types of entities that share the same properties or attributes in an Entity-Relationship (ER) model. It is essentially a group of entities that represent real-world objects or concepts within a database. For example:
 - Entity set (Student) is consisted of many individual students in real world (Alice, Bob, Cindy, ...) who share the same attributes (FirstName, LastName, StudentID, ...)

## Keys

### Primary Key (PK)

Definition: A key that uniquely identifies each record in a table or entity set.

Features:
 - Must be unique for every record.
 - Cannot contain NULL values.
 - There can only be one primary key for a table.

Example: In a Student table, StudentID can be a primary key.

### Candidate Key

Definition: A set of attributes that can uniquely identify records in a table. **A candidate key is eligible to be chosen as the primary key**.

Features:
 - There can be multiple candidate keys in a table.
 - Each candidate key must be unique and not contain NULL values.
 - Minimal: Contains only the attributes necessary to uniquely identify a record (no extra attributes)

Example: In an Employee table, both EmployeeID and SocialSecurityNumber (SSN) could be candidate keys. We can choose either ID or SSN as the PK, but not both).

### Alternate Key

Definition: A candidate key that is not chosen as the primary key.

### Foreign Key (FK)

Definition: An attribute (or a set of attributes) in one table that refers to the primary key in another table.

Purpose: Ensures referential integrity between tables.

Example: In an Orders table, CustomerID could be a foreign key referencing CustomerID in the Customer table.

### Super Key

Definition: A set of attributes that can uniquely identify records in a table. A super key may include additional attributes beyond what is necessary for uniqueness.

Features:
 - Every primary key is a super key, but not all super keys are primary keys.
 - Not minimal: it can contain many extra attributes, this is how super key is different from candidate key.

Example:
In a Student table, {StudentID, Name} is a super key, though only StudentID is needed to uniquely identify a record.

## Reltionship & Relationship Set

**Relationship**: an association among several entities.

**Relationship set**: collection of relationships of the same type. Some properties:
 - **Degree** = # entities involved in the relationship (in ER model, ≥ 2)
 - **Cardinality** = # associated entities on each side of the relationship (1: 1, 1: M, M: N)
 - **Participation** = must every entity be in the relationship (total participation; partial participation)

Cardinality example:

For State (S) and City, one state contains many cities but one city is only contained in a single state. So the cardinality is 1: M. 1 is represented by a single arrow.

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/cardinality_example.png)

We use thick line to represent total participation:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/relationship_participation.png)

Sometimes, we can have recursive (or self-referencing) relationships. For example:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/self-referencing.png)

## Weak Entity

A **weak entity set** is an entity set that:
 - Depends on a Strong Entity: A weak entity set cannot exist on its own. It relies on a strong entity set (also called the "owner" entity) for identification.
 - Lacks a Primary Key: Weak entities do not have enough attributes to form a unique primary key on their own. Instead, they use the primary key of the associated strong entity combined with their own discriminator (partial key) for uniqueness.
 - Must have total participation in the relationship with the owner entity.

For example:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/weak_entity.png)

In the above example, each family member is uniquely identified by a combination of EmployeeName (the PK from its owner identity) and Name (its discriminator).

## Specialisation/Inheritance

A **subclass** of an entity set A is a set of entities:
 - with all attributes of A, plus (usually) its own attributes
 - that is involved in all of A's relationships, plus its own
 - i.e., subclass inherits attributes and relationships from its parent

Properties of subclasses:
 - **overlapping** or **disjoint** (can an entity be in multiple subclasses?)
 - **total** or **partial** (does every entity have to also be in a subclass?)
 - A class/subclass relationship is often called an IS-A (or IS-AN) relationship because of the way we refer to the concept. We say a DOCTOR is a PERSON ...

Example:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/subclass.png)


































































































 
