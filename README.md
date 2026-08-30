# CLASP

CLASP is an experimental serverless runtime built on top of [Faasm](https://github.com/faasm/faasm).
It keeps Faasm's WebAssembly-based isolation and shared-memory model, and extends its scheduler and
planner to support elastic stream processing: it decides how many workers to use, where to place
each operator, and how to move operator state when that placement changes.

CLASP is the reference implementation for the paper *CLASP: Chained-Request-Aware Scaling and
Operator Placement for Serverless Stream Processing*.

## What CLASP adds on top of Faasm

**Distributed scheduling.** Each worker runs its own scheduling engine, so a function instance
dispatches its chained requests locally instead of routing them through the planner. Requests
destined for the same worker are passed through shared memory.

**Incoming request queues.** The planner and every worker maintain a queue of pending requests.
Queue growth is the signal CLASP uses to detect worker saturation, and queued requests are
migrated along with operator state when the placement changes.

**Per-worker metrics monitor.** Each worker records its queue length, per-operator execution
counts and latencies, per-edge chained-request rates, and the number of distinct remote workers
its requests reach. These are reported periodically to the planner.

**Chained-request-aware capacity model.** The planner estimates worker capacity from execution
cost together with the cost of dispatching chained requests locally and remotely, and the fan-out
cost of reaching additional workers. The model coefficients are fitted online by recursive least
squares over observations from saturated workers, so no offline profiling is required.

**Scaling and operator placement.** When the input rate changes, the planner projects the observed
load onto the target rate and bin-packs operators onto the fewest workers that can sustain it,
visiting operators in reverse-topological order so that the chained-request cost of each placement
is known when the decision is made. Operator parallelism is derived from the resulting placement
rather than tuned independently.

**Coordinated state migration.** A single migration plan is derived from the difference between the
old and the new placement. Workers exchange state and queued requests directly with each other and
resume as soon as their own inbound transfers complete, with no global barrier and no coordination
through the planner.

## Quick start

The build workflow follows Faasm's original process. Update submodules and activate the virtual environment:

```bash
git submodule update --init --recursive
source ./bin/workon.sh
```

```bash
myfaasmctl deploy.compose
```

```bash
myfaasmctl cli.cpp

# Compile the demo function
inv func demo hello
```

## Experiment
Please follows [experiment](https://github.com/PancakeTY/experiment-faabric).

```bash
inv stream.run-wc.test
```

## Acknowledgements
This project builds on [Faasm](https://github.com/faasm/faasm). Thanks to the
Faasm team for open-sourcing their work. The original Faasm paper was
published at Usenix ATC '20 and can be found
[here](https://www.usenix.org/conference/atc20/presentation/shillaker).
