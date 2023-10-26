# PATHOS DATABASE DESIGN DOC

Pathos DataBase is another layer to PathosCloud absorbing the complexity of building data structures only with nodes and paths. It creates data structures that can be understood as tables in order to get information and assign indexes in a more intuitive way.

Nodes and paths are made out of nodes and paths. Each table is a path made out of paths representing each record. The records may represent single nodes or may be indexed to another record from any table within the ownership of the table owner.

The table analogy works by creating a tag structure for relational database management that only the library tool is able to interpret. This early version of PathosDB allows unique and nullable fields and JSON database translation.

`summoning pathos-db :: pdb`

## Creation


**pdb.createTable(__owner__, __name__)**
It creates a path tagged as “table” owned by the given entity hash (the system (the pioneer) if nothing is given)

The name of the table holds a unique key on the dictionary for the table owner. If the table name already exists, it is not possible to create it.


**pdb.addFieldToTable(__owner__, __table__, __field_name__, __type__, __unique__)**
It adds a field to the specified table. The field can be either a node or a path.


**pdb.setUniqueField(__owner__, __table__, __field_name__)**
If an existing field from an existing table exists and is not unique, this function tags it as unique, but if it is already unique, it cannot tag it as not unique.


**pdb.setNullableField(__owner__, __table__, ___field_name__)**
If an existing field from an existing table exists and is not nullable, this function tags it as nullable, but if it is already nullable, it tags it as not nullable.

Internally, the tables are just six paths, one with nodes representing the keys, another one with the values of the keys (a string or a hash), a third one with the type of value ( record, node, hash), a fourth one with nodes that have either true or false values for hash uniqueness (unique fields, default not unique), a fifth one, with nodes that have either true or false values for empty values (nullable fields, default nullable) and finally, a sixth one with a field pointing the ancestor record whenever the field on a record is updated (default ancestor pointer points to the record itself). The paths representing the keys hold unique keys on the dictionary for the table owner AND the table they’re associated with. The table may belong to the system (the pioneer), a user, a user ramification, or an organization.


**pdb.createTableRecord(__owner__, __table__, __record__)**
It receives the name of the table, and the record for the table as an array with the following structure:
```
{
    “the_field_name”: “the_content”,
    “the_second_field_name”: “the_content”,
}
```

#### Nullability
If any field is tagged as not null, the function starts by finding non-existing fields that should not be null. If one of those is found, it returns `-1`.

#### Uniqueness
If any field is tagged as unique, the function then looks for the given value on the existing records. If it is found, it returns `-2`.

#### Referencing system hashes
If the field is a random hash, the hash is searched on the global site and in the table owner’s domain. If it is not found, it returns `-3`.

#### Referencing other records
If the field is a record, the path of the record is searched on the global site and in the table owner’s domain and indexed properly (it works like a primary key index). If it is not found, it returns `-4`.

#### Nodes
If the field is signaled as a node, the content is assumed as the content of the node. A node is created and added as the new record.


## Mutability

**pdb.editRecord(__owner__, __table__, __record__)**

It receives the name of the table, and the record for the table as an array with the following structure (it can contain from all the table’s fields to a single one):
```
{
    “the_field_name”: “the_content”,
    “the_second_field_name”: “the_content”,
}
```

#### Nullability
The existing record already respects the nullability structure. If any field is missing on the updated array, the existing field is taken as default. (no possible error code)


#### Uniqueness
If any field is tagged as unique, the function then looks for the given updated values on the existing records. If it is found, it returns `-2`.


#### Referencing system hashes
If any updated field is a random hash, the hash is searched on the global site and in the table owner’s domain. If it is not found, it returns `-3` and the name of the field.


#### Referencing other records
If any updated field is a record, the path of the record is searched on the global site and in the table owner’s domain and indexed properly (it works like a primary key index). If it is not found, it returns `-4` and the name of the field.


#### Nodes
If the field is signaled as a node, the content is assumed as the content of the node. A node is created and added as the new record.


Whenever a field is updated, a “update record” is created on a special table on the system and the ancestor pointer of the field is updated to its ancestor record hash.

## Retrieving

**pdb.getTable(__owner__, __table__, __n__, __depth__)**
This function returns a path as a json object with the table content for the specified number of records. (false = all of them). The 'depth' variable is set to the content of the table by default. The depth in retrieving the layers inside the tree of records can be specified, but it may grow unexpectedly fast.

**pdb.getRecord(__owner__, __table__, __record__)**
This function returns a path as a json object with the record content for the specified record.

## Deleting
**pdb.deleteTable(__owner__, __table__)**
This function deletes the specified table by moving the pointer of the table to a table of deleted tables. It can be accessed later by its owner.

**pdb.deleteRecord(__owner__, __table__, __record__)**
The record is removed form the path table, but it is kept on a mirror table of deleted records

## Deleting (DANGEROUS MODE)

**pdb.recursivelyDeleteTable(__owner__, __table__)**
If the owner is specified, this function deletes the specified table by moving ERASING every appearance of its hashes on the system that can be tracked.If the table is public, the owner is changed to the pioneer’s domain, but it never disappears.

**pdb.recursivleyDeleteRecord(__owner__, __table__, __record__)**
If the owner is specified, the record is ERASED from the file system, including every appearance that can be tracked. If the table is public, the owner is changed to the pioneer’s domain, but it never disappears.

