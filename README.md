# TSN Scheduler Benchmarking Scenarios
This dataset provides scenarios for testing Time-Sensitive Networking (TSN) schedulers, originally developed as part of the following PhD thesis (to be published):
- E. Schweissguth, "**Routing and Scheduling of Time-Triggered Ethernet Networks: ILP Models and Scheduler Benchmarking.**". University of Rostock.

An earlier version of benchmarking scenarios was published alongside the following publication:
- E. Schweissguth, S. Mehner, D. Hellmanns, J. Falk, H. Parzyjegla, P. Danielis, O. Hohlfeld, G. Muehl, and D. Timmermann, "**TSN Scheduler Benchmarking.**" in *WFCS 2023*. IEEE.

Scenarios of this earlier version are considered deprecated, but may still be useful for testing schedulers which do not support heterogenous cycle times. They are still available in the history (see releases).

Please refer to the thesis for detailed background information. Refer to "**TSN Scheduler Benchmarking: Results**" (https://github.com/EikeSG/TSNBenchResults or linked projects on Zenodo) for a comprehensive comparison of ILP-based scheduling models based on this scenario dataset. When accessing this dataset through Zenodo, also check the corresponding GitHub repository (https://github.com/EikeSG/TSNBenchScenarios) for updates and for a good in-browser view (especially for markdown files).

The scenarios comprise an unicast dataset and a multicast dataset, of which the unicast dataset is slightly smaller and easier to solve. The structure of the scenario dataset is described in the following. When using the dataset, please consider our notes on [*licensing and citation*](#license-and-citation).

# Dataset Structure
The scenarios are structured as follows:

```
scenarios
├── unicast
│   ├── mesh_9
│   ├── mesh_12
│   ├── ...
│   ├── ring_96
│   └── filters.json
├── multicast
│   ├── merged
│   └── filters.json
├── topologies
│   ├── fattree16_1.0x1.0.svg
│   ├── ...
│   └── topologies.md
└── format_specification
    ├── streams.json
    └── topology.json
```

`unicast` and `multicast` contain the input files (i.e., scenarios) for the easier unicast test and the more detailed multicast test cases, respectively.
Note that the two datasets have slightly different structure and naming schemes, which is only due to historic reasons.
Specifically, unicast scenarios are sorted by topology and have topology type and size as directory name, whereas multicast scenarios are in a single directory and have topology type and size as part of the respective topology and pattern files.
In both cases, a scenario consists of a topology file (`*.top`) and a corresponding stream set (`*.pat`, from traffic pattern) file, both encoded in a **json** format explained in detail in the files found in `format_specification`.
A visualization of all used topologies can be found in `topologies`.

The large amount of scenarios for scheduler benchmarking is reasoned by the numerous relevant parameters (topology size, stream count, cycle times, etc.).
For evaluation, the scenarios are structured into testcases with carefully selected scenario parameters.
Full parameter lists of the testcases are given in `unicast/filters.json` and `multicast/filters.json`, respectively.
The reasoning for the testcases and parameter selection can be found in the thesis, whereas a brief overview is given in the following.
The parameter lists in the `filters.json`-files are given in the following format:

```json
{
	"TC-L" : [
		[["ring"], [8], [57, 82, 107], [100, 124, 156, 196], [1500], [6]],
		[["mesh"], [9], [55, 79, 103], [84, 100, 124, 156], [1500], [6]]
	],
	"TC-LL" : [
		[["ring"], [8], [82], [100, 124, 156, 196], [1500], [1.5, 3, 6]],
		[["mesh"], [9], [79], [84, 100, 124, 156], [1500], [1.5, 3, 6]]
	],
    ...
}
```

As apparent, each TC has multiple parameter specifications (usually one per network structure), which where used during scenario creation as follows.
From each parameter specification, the cartesian product is constructed to get parameter tuples `(network structure, no. of hosts, stream count, cycle time [us], frame size [B], latency factor)`, see thesis Sec. 4.2.2.1 for details.

For example:

- `[["ring"], [8], [57, 82, 107], [100, 124, 156, 196], [1500], [6]]` yields:
    - `(ring, 8,  57, 100, 1500, 6)`
    - `(ring, 8,  57, 124, 1500, 6)`
    - `(ring, 8,  57, 156, 1500, 6)`
    - `(ring, 8,  57, 196, 1500, 6)`
    - `(ring, 8,  82, 100, 1500, 6)`
    - ...

Each such parameter tuple describes the topology to be used in the first 2 parameters (here: ring with 8 end stations), which is created by predefined helper functions, assigned a topology ID `tid`, and stored in a file named accordingly, such as `t00.top` for the stated `unicast/ring_8` example.
The following 4 parameters are used for stream set creation and describe stream count, cycle time, frame size and latency factor.
Each scenario is created with a random stream placement (i.e., stream sources/destinations) as well as random stream property assignments.
Assigned stream properties are based on the given cycle time, frame size and latency factor given in the parameter tuple and associated scaling factors, see thesis Sec. 4.2.1 for details.
For each such parameter tuple, four stream sets were created, which differ due to the randomness in stream creation.
Each created stream set is stored in a file named by the associated topology, a pattern ID `pid`, and the parameters stated in the tuple that was used for creation, e.g., `t00_p008-00_fc057_ct0100_fs1500_lf6.pat` for the stated `unicast/ring_8` example.

Specifically, the structure and naming convention used for the unicast and multicast are as follows:
- unicast with files in different subdirectories:
    - topology file path:   `[network structure]_[topology size]/t[tid].top`
    - stream set file path: `[network structure]_[topology size]/t[tid]_p[pid]-[repetition]_fc[stream count]_ct[cycle time]_fs[frame size]_lf[latency factor].pat`
- multicast with a single directory for all files:
    - topology file path:   `merged/t[tid]_[network structure][topology size].top`
    - stream set file path: `merged/t[tid]_[network structure][topology size]_p[pid]-[repetition]_sss[stream count]_ct[cycle time]_fs[frame size]_lf[latency factor].pat`

Note that, in this repository, the `[repetition]` entry is always `00`. During benchmarking, it is used to indicate repeated computations of a single scheduler on the same scenario (to accomodate runtime variance of many schedulers), see linked result data for examples.

Note that the benchmarking scenarios are intentionally not structured into subdirectories based on the testcases defined in `filters.json`:
Since the parameter sweeps of different testcases have partially overlapping parameter tuples (e.g., see TC-L vs TC-LL) multiple testcases may *share* the same scenario files for such parameter overlappings.
This is done to reduce the total runtime of the benchmarking.
When only scenarios or results of a specific testcases are needed (e.g., when plotting results), the parameter specifications from `filters.json` can also be used to filter the subdirectories for the desired files (hence the filename `filters.json`).

# License and Citation
This scenarios dataset is made available under the Open Database License: http://opendatacommons.org/licenses/odbl/1.0/. Any rights in individual contents of the dataset are licensed under the Database Contents License: http://opendatacommons.org/licenses/dbcl/1.0/. See `LICENSE` for a copy of the license text and https://opendatacommons.org/licenses/odbl/summary/ for a brief description.

When using the scenarios, please cite:
- E. Schweissguth, "Routing and Scheduling of Time-Triggered Ethernet Networks: ILP Models and Scheduler Benchmarking." PhD thesis, University of Rostock.
- E. Schweissguth, "TSN Scheduler Benchmarking: Scenarios." [Online]. Available: (insert zenodo url of the specific release you used)
