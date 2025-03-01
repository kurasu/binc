# Specification for binc v1

## File format

A binc file consists of a _File Header_ followed by any number of _Changes_.

The change header includes the data size, making it possible skip or store unknown change types as bytes.

### File Header

| Bytes | Payload        |
|-------|----------------|
| 4     | format: 'binc' |
| 4     | version: 1     |

### Change Header + Data

| Bytes    | Payload        |
|----------|----------------|
| 1+       | change type ID |
| 1+       | datasize       |
| datasize | <change data>  |

## Data Types

### Variable length values

Lengths are stored with _variable length encoding_ in big-endian order.

Hence: MSB of a byte is used to indicate if another byte will follow, the remaining 7-bits are used for the value.

Examples:

| Value | Encoded (Hex) |
|-------|---------------|
| 20    | 0x14          |
| 127   | 0x7x          |
| 128   | 0x8100        |

Variable length values are noted to take 1+ bytes of size in the specification and referred to as the type _length_. It
is always unsigned.

### Strings

Strings are stored as UTF-8 encoded bytes prefixed with the string length in bytes as a variable length value. Strings
are not null-terminated.

### Node ID

A node-id is a length value which is unique within the container. Node ids within the file should generally be
contiguous and start from 1, but this is not a strict requirement. However as an implementation may opt for a flat
vector storage they shouldn't be sparse.

## Concepts

### Nodes

A _Node_ is a generic data structure which can store other _Nodes_ and _Attributes_. Child nodes are index-based
and must be contiguous. Nodes can have a type and a name.

There is a pre-existing root node with ID 0.

### Attributes

_Attributes_ are stored in _Nodes_ using an ID and can be of various types. The ID is a (variable-length) integer value
which acts as a key. The name of an attribute-id can be defined globally (optional).

## Changes

Each change has a type ID and a data payload. Each change represents a modification to the in-memory data structure.

### Add Node

change type ID = 1

Adds a new node to the container.

The new node is added as a child of the specified parent node. As child nodes are index-based, the new node is inserted
at the specified index and existing nodes with the same or higher index are shifted one step.

| Bytes | Type    | Payload                         |
|-------|---------|---------------------------------|
| 1+    | node-id | id for new node                 |
| 1+    | node-id | parent node id (root node is 0) |
| 1+    | length  | insertion index in parent node  |

### Remove Node

change type ID = 2

Removes a node and all its children from the container.

As the node is removed, the indices of all nodes with a higher index are shifted one step.

| Bytes | Type    | Payload              |
|-------|---------|----------------------|
| 1+    | node-id | id of node to remove |

### Move Node

change type ID = 3

Moves a node to a new parent node. The node is inserted at the specified index in the new parent node and removed from
the old parent node.

| Bytes | Type    | Payload                            |
|-------|---------|------------------------------------|
| 1+    | node-id | id of the node to move             |
| 1+    | node-id | id of the new parent               |
| 1+    | length  | insertion index in new parent node |

### Set Type

change type ID = 4

Sets the type of a node. A type is a user-defined integer value which can be used define different types (classes) of
nodes.

| Bytes | Type    | Payload               |
|-------|---------|-----------------------|
| 1+    | node id | node                  |
| 1+    | type id | new type for the node |

### Define Type Name

change type ID = 5

Defines a global user-readable name for a type.

| Bytes | Type    | Payload                        |
|-------|---------|--------------------------------|
| 1+    | type id | node type which to define name |
| 1+    | string  | name                           |

### Set Name

change type ID = 6

Sets the name of a node.

| Bytes | Type    | Payload      |
|-------|---------|--------------|
| 1+    | node id | node to name |
| 1+    | string  | name         |

### Define Attribute Name

change type ID = 7

Defines a name for an attribute id.

| Bytes | Type    | Payload                  |
|-------|---------|--------------------------|
| 1+    | attr id | id for attribute to name |
| 1+    | string  | name                     |

### Set Bool

change type ID = 8

Sets a boolean attribute.

| Bytes | Type    | Payload   |
|-------|---------|-----------|
| 1+    | node id | node      |
| 1+    | attr id | attribute |
| 1     | bool    | new value |

### Set String

change type ID = 9

Sets a string attribute.

| Bytes | Type    | Payload   |
|-------|---------|-----------|
| 1+    | node id | node      |
| 1+    | attr id | attribute |
| 1+    | string  | new value |
