### SplitwiseSimCPUAgingAware: Extended SplitwiseSim for CPU Aging-aware LLM Serving

SplitwiseSimCPUAgingAware is a discreet event simulator that helps evaluating CPU aging-aware model serving in LLM
inference clusters. It was built by extending SplitwiseSim, an LLM serving cluster simulator.

SplitwiseSimCPUAgingAware also implements our research work on mitigating embodied carbon in LLM inference clusters.
It hosts implementation and evaluation of the aging-aware CPU management technique that we propose.

#### Setup

We designed _splitwise-sim-cpu-aging-aware_ repository to focus on the CPU aging-aware extension. You can follow the 
steps below to setup SplitwiseSimCPUAgingAware on top of the SplitWiseSim.

1. Checkout to the base SplitWiseSim code. For that, download the tag from 
[https://github.com/tharindu-b-hewage/splitwise-sim/releases/tag/base-cpu-aging-aware](https://github.com/tharindu-b-hewage/splitwise-sim/releases/tag/base-cpu-aging-aware),
or clone the forked repository [https://github.com/tharindu-b-hewage/splitwise-sim](https://github.com/tharindu-b-hewage/splitwise-sim) and checkout to the tag ```base-cpu-aging-aware```.
2. Follow SplitwiseSim instructions to download traces and run example simulation scenarios.
3. Apply the patch: [extension/splitwise-sim.patch](extension/splitwise-sim.patch) which extends the base to ```SplitwiseSimCPUAgingAware```.

#### Changes Introduced

Below is a high-level summary of the new or modified functionality introduced with the patch. These changes primarily 
focus on adding CPU modeling (including per-core allocation, C-State power/temperature modeling, and new scheduling
algorithms) to manage silicon aging in CPU during LLM model serving.

1. **CPU and Core Modeling**  
   - **Core Power & Residency Tracking**  
     - New modules (`core_power.py` and `core_residency.py`) provide a detailed CPU model, including:
       - Per-core frequency, temperature, and aging effects.
       - Idle-state (C-State) transitions and associated wake-up latencies.
   - **Core Objects and Sleep/Wake Mechanisms**  
     - Each `CPU` processor is divided into `Core` objects tracking frequency, temperature, current task, forced sleep, etc.
     - A “sleep manager” adjusts which cores are forced to sleep or woken up based on cluster load and a configurable reaction function.

2. **Extended Configuration and Hardware Definitions**  
   - **CPU-Based SKUs**  
     - YAML definitions added for servers that include both CPU(s) and GPU(s), e.g. `dgx-a100-with-cpu.yaml`, `dgx-h100-with-cpu.yaml`, etc.
     - Parallel “variants” (e.g., `dgx-h100-with-cpu-vm40`, `dgx-h100-with-cpu-vm80`) model different CPU core counts.
   - **New CPU Processor YAMLs**  
     - Added for Intel (dual-xeon-platinum) and AMD (dual-amd-rome) CPUs with multiple core-count configurations.

3. **Instance and Executor Adjustments**  
   - **CPU Overheads for Common Operations**  
     - Memory allocation, task dispatch, and general “executor” overhead now model CPU usage.  
     - Instances track a dedicated CPU object and factor in CPU overhead when scheduling tasks.
   - **CPU Task Scheduling**  
     - Task management techniques in LLM inference clusters, including ```Task-to-core Mapping``` algorithm in our proposed technique.
     - `task_schedule_linux`, `task_schedule_least_aged`, and `task_schedule_proposed` illustrate distinct policies for core selection.
     - A new `cpu_configs.properties` file lets you switch among scheduling algorithms (`linux`, `least-aged`, or `proposed`).

4. **Simulator Hooks**  
   - **Periodic Sleep-Management**  
     - When running the simulation with `proposed` CPU scheduling, the simulator periodically calls `cpu.adjust_sleeping_cores()` to execute ```Selective Core Idling``` algorithm in our proposed technique.
   - **CPU Core Usage Logs**  
     - End-of-simulation triggers a final “state update” to log each core’s final status (frequency, temperature, etc.).

5. **Plotting & Analysis Scripts**  
   - Several new Python files (`llm-ca_misc_plots.py`, `llm-ca_perf_metric_plots.py`, `llm-ca_plots_tasks-vs-time.py`) to:
     - Parse and plot CPU usage, tasks-per-core distributions, frequency aging, etc.
     - Generate visualizations for reaction functions, core availability, and frequency drop over time.

6. **New Experiment Scripts**  
   - **`run_cpu_experiments.sh`**  
     - Automates the simulation runs across multiple VM configurations (varying CPU core counts) and scheduling techniques.
   - **Refined “splitwise” Scripts**  
     - Extended scripts (`run_splitwise_ha_cpu.sh`, etc.) configure cluster servers to include CPUs alongside GPUs, enabling CPU-based overhead.

Overall, these modifications integrate a rudimentary but flexible CPU model into the simulator, allowing fine-grained exploration of CPU aging through CPU overhead, power states, core-level scheduling, and the interplay between CPU and GPU resources in LLM inference clusters.

#### Usage

The ```run_cpu_experiments.sh``` script execute various configurations of inference traces, CPU management techniques, 
and instance core counts to conduct multiple LLM service experiments. Make sure to change experiment data output folder 
accordingly. Upon execution, you can refer to plotting and analysis scripts we mentioned in the section above. Modify 
the script to point experiment data properly. These scripts generate CPU aging-related plotting in the ```results_cpu```
folder.

#### Reference

If you use SplitwiseSimCPUAgingAware in your work, please cite the accompanying paper:




