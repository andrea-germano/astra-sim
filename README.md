# ASTRA-sim (MLSynth-extended — QoS & multi-flow)

This fork includes additional functionality we implemented to aid in our evaluation of **disaggregated LLM inference** workloads synthesised with [MLSynth](https://github.com/NetMLSim/MLSynth). It extends the ns-3 network frontend and the system/workload layers so that per-flow priority (QoS), concurrent point-to-point transfers, and detailed per-operator statistics can be driven directly from the Chakra Execution Traces produced by MLSynth.

All additions are opt-in and controlled from the system configuration file, so default runs behave exactly like upstream ASTRA-sim.

## Current features

### Multiple concurrent flows
By default, ASTRA-sim allows at most one in-flight GPU communication (point-to-point) operation per NPU. The `unlimited-in-flight-comm-ops` flag in the sys config lifts this limit, so a rank can issue several `SEND` operations concurrently (needed to model simultaneous KV-cache transfers). The single-operation assertions in `HardwareResource` become no-ops when the flag is set, while `COMM_RECV` nodes remain always schedulable.

### Per-flow QoS tagging
When `qos-enabled` is set, `issue_send_comm` reads the Chakra attribute `comm_qos_pg` from each `SEND` node and forwards it to the ns-3 backend as the RDMA priority group (`pg`, valid range 1–7, default 3). This lets MLSynth assign different priorities to different traffic classes (e.g. KV transfer vs. tensor-parallel collectives) on a per-flow basis. Invalid or missing tags fall back to the default priority with a warning.

### Per-operator CSV statistics
The `csv-output-enabled` flag makes each system dump a `stats_sys<id>.csv` file at the end of the run. Each row describes one operator (compute or communication) with node id, name, type, comm size, start/end tick, duration, achieved bandwidth, and the roofline metrics (operational intensity, compute/memory utilisation, memory-bound flag). Operator names are now propagated into the statistics records.

### Configurable log/output folder
A new `--logging-folder` command-line argument sets the directory for all log and CSV output; nested paths are created automatically. `LoggerFactory` exposes the active path via `log_path()`, and the spdlog configuration is resolved relative to it.

### Richer callback trace
When `trace-enabled` is on, the workload callback log lines additionally report the operator `comm_size` and achieved `bw`.

> The ns-3 submodule is repointed to the companion fork [`andrea-germano/astra-network-ns3`](https://github.com/andrea-germano/astra-network-ns3), which implements the switch/end-host priority scheduling that consumes the `pg` set here.

---

<details>
<summary>ASTRA-sim</summary>

# ASTRA-sim
[ASTRA-sim](https://astra-sim.github.io/) is a distributed AI system simulator. It models the end-to-end software and hardware stack of modern AI systems - encompassing workload scheduling, collective communication algorithms, and hardware architectures (compute/memory/network). Through a suite of APIs, it enables plug-and-play of external open/proprietary components for modeling different parts of the AI system. This provides end-to-end multi-fidelity simulation capabilities for aiding in design and deployment of next-generation distributed AI systems. 


### Overview and Documentation
Here is a concise visual summary of ASTRA-sim, showing its layers and APIs:
![alt text](https://github.com/astra-sim/astra-sim/blob/master/docs/images/astrasim_overview_codesign.png)

For a comprehensive understanding of the tool, and to gain insights into its capabilities, please visit our [website](https://astra-sim.github.io/).

For information on how to use ASTRA-sim, please visit our [Wiki](https://astra-sim.github.io/astra-sim-docs/index.html).

ASTRA-sim accepts MLCommons Chakra Execution Traces as workload-layer inputs. For details, please visit [Chakra Github](https://github.com/mlcommons/chakra).


### Releases and Contributions

ASTRA-sim is currently at **version 2.0.**
The previous version, ASTRA-sim 1.0, is available in the `ASTRA-sim-1.0` [branch](https://github.com/astra-sim/astra-sim/tree/ASTRA-sim-1.0).

We encourage community contributions to ASTRA-sim via PRs.


## Contact Us
For any questions about using ASTRA-sim, you can email the ASTRA-sim User Mailing List: astrasim-users@googlegroups.com

To join the mailing list, please fill out the following form: https://forms.gle/18KVS99SG3k9CGXm6


We appreciate your interest and support in ASTRA-sim!

</details>