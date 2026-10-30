# lem-in

> Bring a whole ant colony from `##start` to `##end` in the fewest possible turns.

`lem-in` reads a colony file — rooms, tunnels, start, end and the number of ants — and prints
the shortest sequence of moves that delivers every ant to the exit. Because a room can hold
only one ant at a time, the answer is not simply "the shortest path": it is the best
combination of vertex-disjoint paths.

Built with the Go standard library only.

## How it works

1. **Parse & validate** the colony file and reject malformed input with an `ERROR:` message.
2. **Build a flow network** — every room is split into `room_in → room_out` with a capacity of `1`
   (start and end get a capacity equal to the number of ants), and every tunnel becomes
   `room_out → room_in` with a capacity of `1`. A room can therefore hold one ant per turn.
3. **Edmonds–Karp max flow** — BFS-based augmenting paths find the maximum number of
   vertex-disjoint routes.
4. **Flow decomposition** — the resulting flow is turned back into real paths.
5. **Pick the best subset** — for `k = 1..N` paths the total number of turns is counted and the
   shortest prefix that finishes earliest is kept.
6. **Simulate** — each ant is assigned to the path that will deliver it soonest, and the moves
   are printed turn by turn.

## Input format

```
<number of ants>
<room name> <x> <y>
...
##start
<start room> <x> <y>
##end
<end room> <x> <y>
...
<room1>-<room2>
```

- The first non-comment line is the number of ants.
- `##start` / `##end` mark the room declared on the following line.
- Tunnels are written `room1-room2` and are bidirectional.
- Lines starting with `#` are comments and are ignored.
- Rooms are `name x y` with integer coordinates.

### Validation

The parser rejects, among others:

- a missing or non-positive number of ants;
- duplicate room names or duplicate coordinates;
- rooms declared after the tunnels;
- unknown rooms in a tunnel, or the same tunnel twice;
- a missing, duplicated or misplaced `##start` / `##end`;
- room names starting with `L` or `#`, or non-integer coordinates;
- colonies with no path between start and end.

## Usage

```bash
go run . example01
```

Or build it once:

```bash
go build -o lem-in
./lem-in example00
```

The colony is echoed first, then an empty line, then one line per turn:

```
L1-t L2-0
L1-E L2-o L3-t
L2-E L3-o L4-t
```

`L<ant>-<room>` means ant number `<ant>` moved into `<room>` on that turn.

## Testing

```bash
for f in example*; do ./lem-in "$f" > /dev/null || echo "FAIL: $f"; done
for f in badexample*; do ./lem-in "$f" 2>&1 | grep -q ERROR || echo "FAIL: $f"; done
```

## Project structure

```
lem-in/
├── main.go              # CLI entry point
├── utils/
│   ├── parser.go        # input parsing and validation
│   ├── findPaths.go     # max-flow path selection
│   ├── helpers.go       # residual graph, BFS, Edmonds-Karp, flow decomposition
│   ├── simulator.go     # ant assignment and turn-by-turn output
│   ├── nodes.go         # room_in / room_out helpers
│   └── structures.go    # Colony type
├── example00 … example07
└── badexample00, badexample01
```

## Built with

- Go 1.22.3
- Standard library only (`fmt`, `os`, `strconv`, `strings`)
