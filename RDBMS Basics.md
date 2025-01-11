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

## Relational Data Model

Difference between entity relationship model (ERM) and relational data model:

![](https://github.com/Magistus-Ninaruru/PostgreSQL/blob/main/images/erm_relational_diff.png)

**Terminologies**

A **relation schema** (denoted R,S,T,...) has:
 - a name (unique within a given database)
 - a set of attributes (which can be viewed as column headings)
 - e.g., STUDENT(Name, Ssn, Home_phone, Address, Office_phone, Age, Gpa)
 - or generally, R (A1, A2, …, An)
 - Sometimes R can be written as: R(A1:D1, A2:D2, ... An:Dn), where Ai denotes an attribute in R, Di denotes the domain of Ai

Each **attribute** (denoted A,B,... or a1,a2,...) has:
 - a name (unique within a given relation)
 - an associated domain (set of allowed values)
 - e.g., STUDENT(Name: string, Ssn: string, …, Age: integer, Gpa: real)

A **relation** r of the relation schema R(A1, A2, ... , An) is denoted by r(R) where r(R) is a set of n-tuples, i.e., r = {t1, t2, ... , tm}. Each tuple t is an ordered list of values t = <v1, v2, v3 ..., vn> where each vi is an element
of dom(Ai).

### Integrity Constraints

**Constraints Applying to Single Relations**
 - **Domain Constraints**
    - “No employee is younger than 15 or older than 80” -> dom(Age) = > 15 or < 80
 - **Superkeys and keys**
    - “Employees with the same tax code are identical”, in other words, the values of any given two employees’ tax code attribute are different
    - t1[taxCode] ≠ t2[taxCode]
 - **NULL value constraint**
    - ”Employee name cannot be NULL” -> Add *NOT NULL* in table definition

**Constraints Applying to Many Relations**
 - **Entity Integrity Constraint**
    - No primary key value can be NULL
    - Applies to a single relation, but important in the context of multiple relations
 - **Referential Integrity Constraint**
    - Between two relations, a tuple in one relation that refer to another relation must refer to an existing tuple in that relation
    - For example, Account.branchName must refer to an existing tuple (i.e., branch name) in Branch.branchName

**Integrity Violations of Database Operations and How to Solve Them**

<ins>Primary Key Constraint</ins>

Violation: Attempting to insert or update a tuple with a duplicate primary key value or a NULL primary key.

Actions:
 - Ensure the primary key value is unique and not NULL before the operation.
 - Modify the incoming data to use a unique value or leave the primary key unchanged during an update.
 - For batch inserts, identify and remove duplicates in the input data.

<ins>Foreign Key Constraint</ins>

Violation:
 - Inserting a tuple with a foreign key value that does not match a primary key in the referenced table.
 - Deleting a tuple that is referenced by a foreign key in another table.

Actions:

For insertion: Insert the corresponding primary key value into the referenced table before performing the insert.

For deletion:
 - Use ON DELETE CASCADE if cascading deletion is acceptable.
 - Use ON DELETE SET NULL or ON DELETE SET DEFAULT if allowed by the schema.
 - Prevent deletion if the referenced tuple cannot be modified or removed.

For updates: Similar to deletions, ensure the foreign key in dependent tuples is updated accordingly.

**ON DELETE**: In relational databases, ON DELETE actions are used to define the behavior of a foreign key when a referenced row in the parent table is deleted.

```
ON DELETE CASCADE
-- When a row in the parent table is deleted,
-- all rows in the child table referencing that row are also automatically deleted.

ON DELETE SET NULL
-- When a row in the parent table is deleted,
-- all foreign key references in the child table are set to NULL.

ON DELETE SET DEFAULT
-- When a row in the parent table is deleted,
-- all foreign key references in the child table are set to a default value specified in the column definition.
```

Example usage:

```
CREATE TABLE Department (
    Dept_ID INT PRIMARY KEY,
    Name VARCHAR(50)
);

CREATE TABLE Employee (
    Emp_ID INT PRIMARY KEY,
    Name VARCHAR(50),
    Dept_ID INT,
    FOREIGN KEY (Dept_ID) REFERENCES Department(Dept_ID)
    ON DELETE CASCADE
);
```

In the above case, if all apartments with Dept_ID = 1 are deleted from Department, then all employees with Dept_ID = 1 are also deleted accordingly from table Employee.

<ins>Not Null Constraint</ins>

Violation: Inserting or updating a tuple with a NULL value in a column that has a NOT NULL constraint.

Actions:
 - Provide a valid, non-NULL value for the constrained column.
 - Update the schema to allow NULL values if appropriate (not recommended without careful consideration).

<ins>Referential Integrity Constraint</ins>

Violation: Performing an operation that leaves related data inconsistent.

Actions:
 - Use cascading operations like ON UPDATE CASCADE or ON DELETE CASCADE to maintain consistency.
 - Modify the dependent tuples to align with changes in the referenced tuples.





















































 
