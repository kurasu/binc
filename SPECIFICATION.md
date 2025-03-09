# Specification binc v1 (draft)

## Data Types

### Varint (Variable length values)

Lengths are stored with a custom variable-length integer encoding scheme that efficiently stores integers using fewer bytes for smaller values.
It's loosely based on the SQLite4 varint format but allow larger values in the 2-byte representation and omits the 5-7 byte representations.

Variable length values are noted to take 1+ bytes of size in the specification and referred to as the type _varint_. They are always unsigned.

For Operation IDs, the encoded varint value is inverted with XOR 0xFF for each byte to make it easier to identify when a new Operation starts in a binary stream.

### Encoding

Depending on the value (a), the varint is encoded as follows:

| a                | Bytes | b₀                 | b₁             | b₂              | b₃ | b₄ | b₅ | b₆ | b₇ | b₈ | b₉ |
|------------------|-------|--------------------|----------------|-----------------|----|----|----|----|----|----|----|
| < 220            | 1     | a                  |                |                 |    |    |    |    |    |    |    |
| < 8411           | 2     | (a-219) >> 8 + 220 | (a-219) & 0xFF |                 |    |    |    |    |    |    |    |
| < 73947          | 3     | 252                | (a-8411) >> 8  | (a-8411) & 0xFF |    |    |    |    |    |    |    |
| < 2<sup>24</sup> | 4     | 253                | a₀             | a₁              | a₂ |    |    |    |    |    |    |
| < 2<sup>32</sup> | 5     | 254                | a₀             | a₁              | a₂ | a₃ |    |    |    |    |    |
| < 2<sup>64</sup> | 9     | 255                | a₀             | a₁              | a₂ | a₃ | a₄ | a₅ | a₆ | a₇ | a₈ |

### Decoding

Depending on the first byte (a₀), the varint is decoded as follows:

| a₀    | Value                    |
|-------|--------------------------|
| < 220 | a₀                       |
| < 252 | (a₀-220) << 8 + a₁ + 219 |
| 252   | (a₁ << 8) + a₂ + 8411    |
| 253   | a<sub>1-4</sub>          |
| 254   | a<sub>1-5</sub>          |
| 255   | a<sub>1-9</sub>          |

### Strings

Strings are stored as UTF-8 encoded bytes prefixed with the string length in bytes as a varint. 
Strings are not null-terminated.

### Node ID

A Node ID is a varint value which is unique within the container. Node IDs within the file should generally be
contiguous and start from 1, but this is not a strict requirement. However as an implementation may opt for a flat
vector storage they shouldn't be sparse. 

## Concepts

### Nodes

A _Node_ is a generic data structure which can store other _Nodes_ and _Attributes_. Child Nodes are index-based
and must be contiguous. Nodes can have a _type_ and a _name_.

There is a pre-existing root node with ID 0.

### Attributes

_Attributes_ are stored in _Nodes_ using an ID and can be of various types. The ID is a varint which acts as a key. The name of an Attribute ID can be defined globally (optional).

## File format

A binc file consists of a _File Header_ followed by any number of _Operations_.

The Operation Header includes the data size, making it possible skip or store unknown Operations as bytes.

### File Header

| Bytes | Payload        |
|-------|----------------|
| 4     | format: 'binc' |
| 4     | version: 1     |

### Operation Header + Data

| Bytes       | Type            | Payload      |
|-------------|-----------------|--------------|
| 1+          | inverted varint | Operation ID |
| 1+          | varint          | data size    |
| <data size> |                 | <data>       |

Note: the Operation ID uses an inverted varint representation.

## Operations

Each Operation has an ID and a data payload. Each operation represents a modification of the in-memory data structure.

### Add Node

Operation ID = 1

Adds a new node to the container.

The new node is added as a child of the specified parent node. As child nodes are index-based, the new node is inserted
at the specified index and existing nodes with the same or higher index are shifted one step.

| Bytes | Type    | Payload                         |
|-------|---------|---------------------------------|
| 1+    | node-id | id for new node                 |
| 1+    | node-id | parent Node ID (root node is 0) |
| 1+    | length  | insertion index in parent node  |

### Remove Node

Operation ID = 2

Removes a node and all its children from the container.

As the node is removed, the indices of all nodes with a higher index are shifted one step.

| Bytes | Type    | Payload              |
|-------|---------|----------------------|
| 1+    | node-id | id of node to remove |

### Move Node

Operation ID = 3

Moves a node to a new parent node. The node is inserted at the specified index in the new parent node and removed from
the old parent node.

| Bytes | Type    | Payload                            |
|-------|---------|------------------------------------|
| 1+    | Node ID | id of the node to move             |
| 1+    | Node ID | id of the new parent               |
| 1+    | length  | insertion index in new parent node |

### Set Type

Operation ID = 4

Sets the type of a node. A type is a user-defined integer value which can be used define different types (classes) of
nodes.

| Bytes | Type    | Payload               |
|-------|---------|-----------------------|
| 1+    | Node ID | node                  |
| 1+    | Type ID | new type for the node |

### Define Type Name

Operation ID = 5

Defines a global user-readable name for a type.

| Bytes | Type    | Payload                        |
|-------|---------|--------------------------------|
| 1+    | Type ID | node type which to define name |
| 1+    | string  | name                           |

### Set Name

Operation ID = 6

Sets the name of a node.

| Bytes | Type    | Payload      |
|-------|---------|--------------|
| 1+    | Node ID | node to name |
| 1+    | string  | name         |

### Define Attribute Name

Operation ID = 7

Defines a name for an attribute id.

| Bytes | Type    | Payload                  |
|-------|---------|--------------------------|
| 1+    | Attr ID | id for attribute to name |
| 1+    | string  | name                     |

### Set Bool

Operation ID = 8

Sets a boolean attribute.

| Bytes | Type    | Payload   |
|-------|---------|-----------|
| 1+    | Node ID | node      |
| 1+    | Attr ID | attribute |
| 1     | bool    | new value |

### Set String

Operation ID = 9

Sets a string attribute.

| Bytes | Type    | Payload   |
|-------|---------|-----------|
| 1+    | Node ID | node      |
| 1+    | Attr ID | attribute |
| 1+    | string  | new value |

## Conventions

### Metadata

Metadata is stored as attributes in the root node. The root node has the ID 0.

Ideally metadata should be stored before adding the first node so that the metadata can be scanned without parsing the whole file.