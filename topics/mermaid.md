- [Table of contents](#table-of-contents)
- [Flowchart and basics](#flowchart-and-basics)
  - [Class diagrams](#class-diagrams)
- [Styling](#styling)
 
## Table of contents

- [Table of contents](#table-of-contents)
- [Flowchart and basics](#flowchart-and-basics)
  - [Class diagrams](#class-diagrams)
- [Styling](#styling)

## Flowchart and basics

The orientation is where the edges point

- `BT`, `TB`/`TD` Bottom/Down, Top
- `LR`, `RL` Left, Right

Sample format for rectangular nodes, with labeled edges: `<node_id>[<node_text>] -- <edge_text> --> <node_2_id>`. Unlabeled edge format: `-->`.

Nodes can be declared explicitly, or implicitly (in a connection).

If a node is defined multiple times with a property (ie. label), the last definition one prevails (although this is not correct).

Use `<br>` for a newline, without closing tag (see #Styling for formatting/styling).

```mermaid
graph BT

%% "C-style" declaration; can look cleaner, but it's not required.

pt[positive_ticket<br>quantity: 1]
pb[positive_booking]
at[amending_ticket]
ab[amending_booking]

pb -- source_is --> pt
at -- amends --> pt
ab -- amends --> pb
ab -- source_is --> pt
```

The `&` operator defines a group of nodes, which are all connected with another node (or group of).

```mermaid
graph BT

%% two edges
amending_ticket_1 & amending_ticket_2 --> positive_ticket

%% four edges!
patient1 & patient2 --> doctor1 & doctor2
```

### Class diagrams

```mermaid
classDiagram

%% comments can't be inside a class (bug!, https://git.io/JuX5t), and they must be in their own line
%% blank lines are ignored
class ClassA {
  field
  Type field

  method()
  method(param) ReturnType
  static_method()$
}

class ClassC

%% inheritance
ClassA <|-- ClassB

%% generic association, with label on the edge
%% a label can span multiple lines via `\n`; leading/trailing spaces are ignored; colon is not allowed
ClassC <-- ClassB : mylabel

%% annotations; displayed inside, at the top
class Color {
  <<enum>>
  RED
  BLUE
}
```

## Styling

Styling must be defined separately.

It seems that the `style` directive can't apply to multiple nodes, so a class is used:

```mermaid
graph LR

start(Start) --> pause1(Pause1) & pause2(Pause2) --> stop(Stop)

classDef pauseClass fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray: 5, 5;

style start fill:#f9f,stroke:#333,stroke-width:4px;
class pause1,pause2 pauseClass;
```

Edges use `linkStyle` (with a 0-based index), which can instead be applied to multiple edges, but classes can't be defined:

```mermaid
graph LR

node0 --> node1 --> node2 --> node3 --> node4

%% Applies to the second and third edges

linkStyle 1,2 stroke:#ff3,stroke-width:4px;
```
