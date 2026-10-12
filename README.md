# visnetwork-go

A Go port of the [vis-network](https://github.com/visjs/vis-network) physics
engine — Barnes-Hut repulsion, spring attraction, central gravity, and the
stabilization loop that drives them.

The point of a port rather than a fresh implementation is **fidelity**. If you
already have a graph laid out by vis-network in a browser and you want the same
picture from Go — server-side, in a wasm canvas, or in a test — a different
force model gives you a different picture. This one matches vis-network's
defaults and its arithmetic, down to the places where the original does
something slightly odd (there is a comment in `gravity.go` noting where
vis-network assigns instead of accumulating; the port keeps the original
behaviour rather than "fixing" it).

## Install

```
go get github.com/0magnet/visnetwork-go
```

## Use

```go
import "github.com/0magnet/visnetwork-go/physics"

e := physics.NewPhysicsEngine(nodes, edges) // map[string]PhysicsNode, map[string]PhysicsEdge
e.SetOptions(physics.DefaultPhysicsOptions())

for !e.IsStabilized() {
    e.PhysicsTick()
}
```

`PhysicsNode` and `PhysicsEdge` are interfaces, so your own graph types
implement them directly and the engine writes positions back through
`SetPosition` — no copying in and out of a parallel structure.

```go
type PhysicsNode interface {
    GetID() string
    GetPosition() (float64, float64)
    SetPosition(x, y float64)
    GetMass() float64
    GetSize() float64
    IsFixed() (bool, bool)
}
```

## Interactive vs batch

The engine is **steppable**, which is the reason it exists in this shape:

- `PhysicsStep()` / `PhysicsTick()` advance the simulation one frame, so you can
  render between frames and let a user drag nodes mid-flight.
- `IsStabilized()`, `SetStabilized()`, `GetStabilizationIterations()` and
  `ResetStabilization()` expose the convergence state.
- `Stabilize(maxIterations int) int` runs to convergence and reports how many
  iterations it took, for when you just want final positions.
- `RunSingleStep() bool` is the one-frame primitive.

Per-node **mass** and **size** feed the force calculation, and `IsFixed()` pins a
node on either axis independently — which is what makes drag-to-pin work.

## Compared to gonum

[`gonum.org/v1/gonum/graph/layout`](https://pkg.go.dev/gonum.org/v1/gonum/graph/layout)
has an Eades force-directed layout with a Barnes-Hut theta constant, and is
steppable via `Update()`. Prefer it if you want a general-purpose layout and
gonum is already in your build.

Use this one if you need vis-network's specific force model and defaults, or any
of: pinned nodes, per-node mass and size, or an explicit stabilization check —
none of which gonum's layout exposes. It also has no dependencies outside the
standard library (testify for tests only), which matters for a wasm binary.

## Notes

`BarnesHutOptions` and `PhysicsOptions` mirror vis-network's option names and
default values, so a configuration copied from a vis-network page works here.

Extracted from [skywire](https://github.com/skycoin/skywire).

## Dependency Graph

Made with [goda](https://github.com/loov/goda):

```
go run github.com/loov/goda@latest graph github.com/0magnet/visnetwork-go/... | dot -Tsvg -o docs/visnetwork-go-goda-graph.svg
```

![Dependency Graph](docs/visnetwork-go-goda-graph.svg "github.com/0magnet/visnetwork-go Dependency Graph")

## Lines of Code

Made with [gocloc](https://github.com/hhatto/gocloc) (excludes `vendor/`, `node_modules/`, `.git/`):

```
gocloc --not-match-d='(vendor|node_modules|\.git)' .
```

```
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Go                               6            213            213           1040
Makefile                         1             22             52            111
YAML                             1              0              7             98
Markdown                         1             27              0             86
Bourne Shell                     1              8             17             31
-------------------------------------------------------------------------------
TOTAL                           10            270            289           1366
-------------------------------------------------------------------------------
```
