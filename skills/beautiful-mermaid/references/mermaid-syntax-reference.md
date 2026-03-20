# Mermaid Syntax Reference

beautiful-mermaidでサポートされるMermaid構文のリファレンス。
generateモードおよびcreateモードでMermaidコードを生成する際に参照する。

## Flowchart shapes

```
A[Rectangle]  B(Rounded)  C{Diamond}  D([Stadium])
E((Circle))  F[[Subroutine]]  G(((Double Circle)))
H{{Hexagon}}  I[(Database)]  J>Flag]
K[/Trapezoid\]  L[\Inverse/]
```

## Edge types

```
A --> B        solid arrow
A --- B        solid line (no arrow)
A -.-> B       dotted arrow
A ==> B        thick arrow
A --text--> B  labeled edge
A <--> B       bidirectional
```

## Directions

`TD` (top-down), `LR` (left-right), `BT` (bottom-top), `RL` (right-left)
