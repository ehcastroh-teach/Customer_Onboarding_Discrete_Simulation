# Discrete Event Simulation of Customer Onboarding

This repository teaches discrete event simulation (DES) using SimPy through a real-world scenario: modeling a multi-stage customer onboarding pipeline. Two notebooks and accompanying code walk you through building simulations, running independent replications to control variance, and sweeping parameter space to find optimal staffing allocations.

## Learning Objectives

By the end of this material, you will be able to:

1. Explain what discrete event simulation is and identify systems where it is the right tool (vs. analytical queueing theory or continuous simulation).
2. Use SimPy's `Environment`, `Resource`, and process generator pattern to model multi-stage service networks with realistic constraints.
3. Instrument a simulation to collect queue state, per-customer wait times, and server utilization metrics.
4. Run independent replications and aggregate results to account for stochastic variability and compute confidence.
5. Sweep staffing configurations to identify optimal allocations that minimize customer wait time or other performance metrics.
6. Connect simulation-derived metrics to analytical M/M/c queueing theory and understand when each tool is appropriate.

## Data / File Dictionary

| File | Type | Purpose |
|------|------|---------|
| `01_help_desk_simulation.ipynb` | Notebook | Two-stage help desk using arbitrary service distributions; sweep staffing space to minimize mean wait time. |
| `02_tenant_onboarding_simulation.ipynb` | Notebook | M/M/c queueing model (Poisson arrivals, exponential service); extract standard metrics (utilization, queue delay, customers in system). |
| `images/onboarding.png` | Image | System diagram showing the two-stage help desk pipeline. |
| `images/GIGs.png` | Image | Visual representation of the GI/G/s queue structure (notebook 1). |
| `images/MMc.png` | Image | Visual representation of the M/M/c queue structure (notebook 2). |

## Workflow Diagram

```
Customer Arrival
       |
       v
  [Stage 1: Triage]
    (1-3 days)
       |
       v
  [Stage 2: Requirements]
   (1-7 days, repeats 50%)
       |
       v
   Depart System
```

In the second notebook, Stage 2 models deployment using Poisson arrivals and exponential service - the assumptions behind M/M/c queueing theory. Queue delays, utilization, and Little's Law emerge from the simulation.

## Step-by-Step Walkthrough

### Notebook 1: Help Desk Simulation (GI/G/s)

Start here if you are new to SimPy. This notebook models a help desk with two sequential service stages, each staffed by a separate pool of engineers. Service times are uniform (not exponential), and 50 percent of customers require a follow-up round at stage 2.

**Why this matters:** Real service networks rarely follow the Poisson/exponential assumptions of M/M/c theory. By relaxing those assumptions in this first notebook, we learn the *mechanics* of DES without getting distracted by how closely it matches analytical theory. By the time you reach notebook 2, you will understand what Poisson and exponential *mean* in code and why they matter.

**What you build:**
- A `HelpDesk` class bundling two SimPy `Resource` objects (the engineer pools).
- A `go_to_help_desk` process generator describing one customer's end-to-end journey.
- An arrival process that spawns customers every 3 days for a simulated 183-day horizon.
- A configuration sweeper that tries all allocations up to a maximum headcount and ranks them by mean wait time.

**Key insight:** Diminishing returns kick in fast. Adding engineers to an already-adequate stage barely moves the needle; identifying the *binding bottleneck* and staffing it preferentially yields much larger gains.

### Notebook 2: Tenant Onboarding (M/M/c)

This notebook grounds simulation in analytical queueing theory. We model a customer onboarding pipeline as a single-stage M/M/c queue: customers arrive according to a Poisson process, service times are exponential, and c servers draw from a shared FIFO queue.

**Why this matters:** M/M/c has closed-form solutions for wait time, queue length, and utilization. Running the simulation and confirming that the output *matches* the analytical result is a powerful sanity check on both. It also lets you see exactly where the assumptions show up in code - `np.random.exponential()` for service times, Poisson arrivals - so that when you later *violate* those assumptions, you understand what changes.

**What you build:**
- A `MonitoredResource` subclass that records queue state transitions and per-customer timings.
- An `arrival` process that spawns customers at random intervals.
- A `serve` process describing what one customer experiences (wait, then service, then depart).
- Computation of four headline metrics: mean wait time, queue delay, mean customers in system, and server utilization.

**Key insight:** Server utilization and wait time are not the same thing. A system can have low utilization but still high wait times if service-time variance is high. Conversely, a system can be heavily utilized and still meet wait-time SLAs if the bottleneck is well-staffed. The metrics together tell the story; no single number does.

## How to Run

### From a clean clone:

```bash
# Install dependencies (if not already present)
pip install simpy numpy jupyter

# Start Jupyter
jupyter notebook

# Open 01_help_desk_simulation.ipynb or 02_tenant_onboarding_simulation.ipynb
# Run cells top-to-bottom (clean kernel restart between notebooks)
```

### Dependencies:

- **SimPy** (4.x): discrete event simulation framework
- **NumPy** (2.x): numerical operations and random sampling
- **Python** (3.8+): generators, dataclasses, standard library

### Running simulations:

All random seeds are fixed at the top of each notebook, so results are reproducible. To explore variance across runs, change the seed value.

```python
random.seed(2022)  # Fixed seed for reproducibility
wait_times = run_simulation(num_help_desk_eng=2, num_on_call_eng=3)
```

To run multiple independent replications:

```python
all_wait_times = []
for _ in range(10):  # 10 independent runs
    all_wait_times += run_simulation(num_help_desk_eng=2, num_on_call_eng=3)
mean_wait = statistics.mean(all_wait_times)
```

## Key Concepts Glossary

| Term | Definition |
|------|-----------|
| **Discrete Event Simulation (DES)** | A simulation technique that advances virtual time in jumps to the next scheduled event, rather than stepping through fixed time intervals. Efficient for systems where behavior concentrates at discrete moments. |
| **SimPy** | A Python library for discrete event simulation built on generator functions. Processes `yield` events; the simulator handles scheduling and concurrency. |
| **Resource** | A SimPy object modeling a limited capacity (e.g., a pool of servers). Requests queue in FIFO order if all capacity is busy. |
| **Process Generator** | A Python generator function that `yield`s events. Each customer, arrival stream, or workflow is a separate process running concurrently. |
| **Poisson Process** | A stochastic arrival process where inter-arrival times are exponentially distributed and arrivals in disjoint time windows are independent. Models random walk-ins or unscheduled events. |
| **Exponential Distribution** | A memoryless probability distribution; the future evolution depends only on the current state, not on how long you have been waiting. Used for service times in M/M/c theory. |
| **M/M/c Queue** | Kendall notation for a queue with Poisson arrivals (M), exponential service (M), and c parallel servers. Has closed-form analytical solutions. |
| **Queue Delay** | Time spent waiting for a server to become available. Excludes service time. |
| **Server Utilization** | Fraction of available server capacity that is in use. Values near 100% mean the system is at saturation; values near 0% mean servers are idle. |
| **Little's Law** | A fundamental result: mean customers in system = arrival rate × mean time in system. Connects throughput, wait time, and queue length. |
| **Independent Replication** | Running a stochastic simulation multiple times with different random seeds to reduce variance in aggregate estimates. Precision improves at rate 1/√R where R is the number of replications. |
| **Traffic Intensity (ρ)** | In M/M/c, ρ = λ / (c × μ), where λ is arrival rate and μ is service rate per server. Must be < 1 for stability. |

## Further Reading

- SimPy documentation - Overview and examples: https://simpy.readthedocs.io/
- Kleinrock, L. *Queueing Systems, Volume 1: Theory*. Wiley, 1975. (Canonical reference for M/M/c theory and Little's Law.)
- Law, A. M. *Simulation Modeling and Analysis*, 5th ed. McGraw-Hill, 2015. (Chapters 1-3 on DES fundamentals; Chapter 9 on replication strategy.)
- Ross, S. M. *Introduction to Probability Models*, 12th ed. Elsevier, 2019. (Chapter 8 on queueing.)

## Credits and Acknowledgements

This material was developed as a learning exercise combining:
- SimPy's official examples and documentation
- Classical queueing theory (Kleinrock, Law, Ross)
- Real-world customer onboarding scenarios from service platforms

The two-notebook structure mirrors how practitioners move from ad-hoc simulation to grounded analytical theory.
