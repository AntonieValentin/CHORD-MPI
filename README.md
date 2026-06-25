# MPI Chord

A C/MPI project that simulates a simplified version of the CHORD protocol. Each MPI process acts as a node in a distributed hash table, placed on a circular identifier space, and cooperates with the other nodes to resolve lookup requests.

The project focuses on the core idea behind CHORD: finding the node responsible for a key without using a central coordinator, by routing requests through a finger table.

## Overview

The system uses a static CHORD ring. All nodes exist from the start of the program, each with its own identifier, and the ring does not change while the program is running.

Each process reads its local input file, learns its own CHORD ID and the keys it has to search for, then exchanges node IDs with the other MPI processes. After that, every node builds its local view of the ring, including its successor, predecessor, and finger table.

## Main Features

* one MPI process represents one CHORD node;
* static identifier space with IDs in the range `0..15`;
* local construction of successor and predecessor information;
* finger table generation using the CHORD formula;
* distributed lookup routing through MPI messages;
* `closest_preceding_finger` logic for efficient forwarding;
* lookup paths printed only by the node that started the request;
* distributed termination using explicit completion messages.

## Lookup Flow

For each key, the initiating node creates a lookup message and sends it into the MPI service loop. Every node that receives the request adds itself to the path and decides what to do next.

If its successor is responsible for the key, the node adds that successor as the final target and sends the result back to the initiator. Otherwise, it forwards the request to the best next hop found in its finger table.

A final output line looks like this:

```text id="v3sfb7"
Lookup 12: 3 -> 9 -> 1
```

This means that the lookup for key `12` started at node `3`, was forwarded through node `9`, and ended at node `1`, which is responsible for the key.

## Input Format

Each MPI process reads a file named after its rank:

```text id="f6suf7"
in0.txt
in1.txt
in2.txt
...
```

Each file contains:

```text id="s47ywd"
<node_id>
<number_of_lookups>
<key_1>
<key_2>
...
```

The node ID gives the position of that process in the CHORD ring, while the remaining values are the keys for which the node must start lookups.

## Building and Running

The project is built with MPI tools:

```bash id="bmd4yr"
make build
```

It can be run with `mpirun`, using one process for each input node:

```bash id="ek6c3q"
mpirun -np <number_of_nodes> ./tema2
```

The number of MPI processes should match the number of `inR.txt` input files.

## Notes

This is not a full dynamic CHORD implementation. Nodes do not join or leave during execution, and the finger tables are computed once at startup.

The important part is the distributed routing itself: messages must travel from node to node, each hop must be justified by the CHORD rules, and the final path must not contain cycles. The implementation keeps the structure simple while preserving the main mechanism that makes CHORD efficient.
