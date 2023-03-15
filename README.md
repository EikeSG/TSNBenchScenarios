# TSN Scheduler Benchmarking Scenarios
This dataset provides scenarios for testing Time-Sensitive Networking (TSN) schedulers originally developed as part of the following publication:
- E. Schweissguth, S. Mehner, D. Hellmanns, J. Falk, H. Parzyjegla, P. Danielis, O. Hohlfeld, G. Muehl, and D. Timmermann, "**TSN Scheduler Benchmarking.**" in *WFCS 2023*. IEEE.

Please refer to the paper for detailed background information. Refer to the linked "**TSN Scheduler Benchmarking: Results**" for a comparison six scheduler configurations based on this scenario dataset. Also check the linked GitHub repository for updates and for a good in-browser view (especially for markdown files).

The scenarios comprise the input files for our initial *factor screening*, and our [*benchmarking scenarios*](#benchmarking-scenarios-and-test-cases) intended for re-use by other researchers. The structure of the scenario dataset is described in the following. When using the dataset, please consider our notes on [*licensing and citation*](#license-and-citation).

# Dataset structure
The scenarios are structured as follows:

```
scenarios
├── benchmarking
│   ├── fat_tree_12
│   ├── fat_tree_24
│   ├── ...
│   ├── ring_8
│   └── filters.json
├── factor_screening
│   ├── fat_tree_10
│   ├── fat_tree_20
│   ├── line_10
│   └── line_100
└── format_specification
    ├── streams.json
    └── topology.json
```

`factor_screening` and `benchmarking` contain the input files (i.e., scenarios) for the initial factor screening test and the more detailed benchmarking test cases, respectively.
Therein, the scenarios are structured into subdirectories by network structure and size, i.e., fat_tree_10 contains fat-tree scenarios with 10 end stations in the topology.
Within each subdirectory, there is a single topology file and multiple corresponding stream set files, both encoded in a **json** format explained in detail in the files found in `format_specification`.

# Benchmarking Scenarios and Test Cases
The `benchmarking` subdirectory contains the re-usable scenarios for scheduler performance analysis and comparison.
As presented in paper Sec. V-D, the benchmarking scenarios are structured into six test cases (TC) with carefully selected scenario parameters.
The full parameter list of all TCs is stated in `benchmarking/filters.json` in the following format:

```json
{
    "TC-L" : [
        [["line"], [8], [70, 100, 130], [310, 390, 490, 610], [1500], [6]],
        [["ring"], [8], [57, 82, 107], [210, 250, 310, 390], [1500], [6]],
        [["mesh"], [9], [55, 79, 103], [210, 250, 310, 390], [1500], [6]],
        [["fat_tree"], [8], [61, 87, 113], [190, 210, 250, 310], [1500], [6]]
    ],
    "TC-LL" : [
        [["line"], [8], [100], [310, 390, 490, 610], [1500], [1.5, 3, 6]],
        [["ring"], [8], [82], [250, 310, 390, 490], [1500], [1.5, 3, 6]],
        [["mesh"], [9], [79], [250, 310, 390, 490], [1500], [1.5, 3, 6]],
        [["fat_tree"], [8], [87], [190, 210, 250, 310], [1500], [1.5, 3, 6]]
    ],
    ...
}
```

As apparent, each TC has multiple parameter specifications (usually one per network structure).
From each parameter specification, the cartesian product is constructed to get parameter tuples `(network structure, no. of hosts, stream count, cycle time [us], frame size [B], latency factor)`, see paper Sec. V-B for details.

For example:

- `[["line"], [8], [70, 100, 130], [310, 390, 490, 610], [1500], [6]]` yields:
    - `(line, 8,  70, 310, 1500, 6)`
    - `(line, 8,  70, 390, 1500, 6)`
    - `(line, 8,  70, 490, 1500, 6)`
    - `(line, 8,  70, 610, 1500, 6)`
    - `(line, 8, 100, 310, 1500, 6)`
    - ...

For each such parameter tuple, we created four random stream sets (different in stream placement, i.e., stream sources/destinations), each stored as 15 redundant file copies with equal content.
The 15 copies correspond to the 15 repeated computations per scheduler for the respective scenario (to accomodate runtime variance of many schedulers).
Although this redundancy could have been avoided, we used this approach to simplify our test procedure: Executing each scheduler exactly once per available stream set file.
As we store the network topology in a separate file (not as part of the stream set file) and due to structuring of the file storage into subdirectories by network structure and size (such as `line_8`, `line_12`, ...), we obtain the following files in a each subdirectory:

- a single topology file named `t[tid].json`
    - `tid` being a unique integer id for the topology
- multiple corresponding stream set files, with naming convention:
    - `t[tid]_ss[placement]_[repetition]_sss[stream count]_ct[cycle time [us]]_fs[frame size [B]]_a[latency factor].json`
    - e.g. `t12_ss077-11_sss113_ct0210_fs1500_a6.json`
    - 4 different placements (different stream sources/destinations) and 15 repetitions for each *[stream count]-[cycle time]-[frame size]-[latency factor]* combination
    - files for different repetitions are redundant (i.e., contain the same information)

Note that the benchmarking scenarios are intentionally not structured into subdirectories based on TCs:
Since the parameter sweeps of different TCs have partially overlapping parameter tuples (e.g., see TC-L vs TC-LL) multiple TCs may *share* the same scenario files for such parameter overlappings.
This is done to reduce the total runtime of the benchmarking.
When only scenarios or results of a specific TC are needed (e.g., when plotting results), the parameter specifications from `filters.json` can also be used to filter the subdirectories for the desired files (hence the filename `filters.json`).

# Factor Screening Scenarios
The scenarios from our initial factor screening test are included in this repository for completeness, but not used for scheduler comparison.
We do not recommend running these scenarios for scheduler comparison.
Refer to our [*benchmarking scenarios*](#benchmarking-scenarios-and-test-cases) instead.
Note the slightly different naming scheme of the factor screening scenarios:

- network topology is defined in `topology.json`
- stream set files follow the naming convention:
    - `streams_[stream count]_[placement]_[cycle time [ns]]_[frame size [B]]_[latency factor].json`
    - e.g. `streams_20_0_500000_100_1.2.json`
    - 25 different placements (different stream sources/destinations) for each *[stream count]-[cycle time]-[frame size]-[latency factor]* combination

# License and Citation
This scenarios dataset is made available under the Open Database License: http://opendatacommons.org/licenses/odbl/1.0/. Any rights in individual contents of the dataset are licensed under the Database Contents License: http://opendatacommons.org/licenses/dbcl/1.0/. See `LICENSE` for a copy of the license text and https://opendatacommons.org/licenses/odbl/summary/ for a brief description.

When using our scenarios, please cite:
- E. Schweissguth, S. Mehner, D. Hellmanns, J. Falk, H. Parzyjegla, P. Danielis, O. Hohlfeld, G. Muehl, and D. Timmermann, "TSN Scheduler Benchmarking." in *WFCS 2023*. IEEE.
- E. Schweissguth, S. Mehner, D. Hellmanns "TSN Scheduler Benchmarking: Scenarios." [Online]. Available: (insert zenodo url of this release)
