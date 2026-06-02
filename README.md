# ergodic-theory

[![crates.io](https://img.shields.io/badge/crates.io-0.1.0-orange)](https://crates.io/crates/ergodic-theory)
[![license](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](./LICENSE)
[![docs](https://docs.rs/ergodic-theory/badge.svg)](https://docs.rs/ergodic-theory)

**Ergodic theory in Rust.** Time averages equal space averages.

---

## What This Does

Ergodic theory studies what happens to dynamical systems in the long run. If you iterate a transformation over and over on a space equipped with a probability measure, does the system "explore everywhere"? Or does it get trapped in sub-regions?

This library answers that question computationally. It gives you:

- **Measure spaces** and **measure-preserving transformations** — the formal foundation
- **Birkhoff's Ergodic Theorem** — time averages converge to space averages (almost everywhere)
- **Ergodicity checking** — invariant sets have measure 0 or 1
- **Mixing conditions** — weak mixing and strong mixing as stronger forms of "well-behaved long-run behavior"
- **Kolmogorov-Sinai entropy** — how chaotic is the system, really?
- **Lyapunov exponents** — exponential divergence rates of nearby trajectories
- **Perron-Frobenius operator** — compute invariant measures via the transfer operator
- **Markov chains** — finite-state chains as measure-preserving systems
- **Shannon-McMillan-Breiman theorem** — entropy along orbits converges to conditional entropy
- **Behavioral prediction** — will an entity explore the full state space or get stuck?

---

## Key Idea

The central theorem of ergodic theory is deceptively simple:

> **Time average = Space average** (for ergodic systems, almost everywhere)

If you flip a fair coin infinitely many times, the fraction of heads converges to ½. Birkhoff's theorem generalizes this to *any* measure-preserving dynamical system. This library makes that generalization computational — you construct a measure space, define a transformation, and then verify ergodic properties numerically.

---

## Install

Add to `Cargo.toml`:

```toml
[dependencies]
ergodic-theory = "0.1"
```

Requires **Rust 2021 edition**. Dependencies: `nalgebra`, `serde`, `approx`.

---

## Quick Start

```rust
use ergodic_theory::measure::{Measure, MeasureSpace, Transformation};
use ergodic_theory::birkhoff;
use ergodic_theory::ergodicity::ErgodicChecker;

// 1. Create a measure space over 4 states with uniform measure
let measure = Measure::uniform(4);
let space = MeasureSpace::new(vec!["A", "B", "C", "D"], measure);

// 2. Define a transformation (permutation of states)
let transform = Transformation::permutation(vec![1, 2, 3, 0]);

// 3. Check ergodicity
let checker = ErgodicChecker::new(&space, &transform);
if checker.is_ergodic() {
    println!("System is ergodic — time averages = space averages!");
}

// 4. Verify Birkhoff's theorem: time average of an observable
let observable = |state: usize| -> f64 { (state + 1) as f64 };
let time_avg = birkhoff::time_average(&space, &transform, &observable, 10_000);
let space_avg = birkhoff::space_average(&space, &observable);
assert!((time_avg - space_avg).abs() < 0.01);
```

---

## API Reference

### `measure` — Measure Spaces and Transformations

| Type | Description |
|------|-------------|
| `Measure` | Probability measure over a finite state set. Supports `uniform(n)`, `dirichlet(weights)`, `point_mass(i, n)` |
| `MeasureSpace` | A measurable space: states + a probability measure |
| `Transformation` | A measurable map T: X → X. Construct via `permutation`, `stochastic_matrix`, or custom closures |

### `birkhoff` — Birkhoff's Ergodic Theorem

| Function | Description |
|----------|-------------|
| `time_average(space, transform, observable, iterations)` | Compute (1/N) Σ f(Tⁱx) — the Birkhoff average |
| `space_average(space, observable)` | Compute ∫ f dμ — the expected value under the measure |

### `ergodicity` — Ergodicity Verification

| Method | Description |
|--------|-------------|
| `ErgodicChecker::new(space, transform)` | Construct a checker |
| `is_ergodic()` | Returns `true` if every invariant set has measure 0 or 1 |
| `invariant_sets()` | Returns all invariant subsets of the state space |

### `mixing` — Weak and Strong Mixing

| Function | Description |
|----------|-------------|
| `is_weak_mixing(space, transform, tolerance)` | (1/N) Σ\|μ(T⁻ⁿ(A) ∩ B) − μ(A)μ(B)\| → 0? |
| `is_strong_mixing(space, transform, tolerance)` | μ(T⁻ⁿ(A) ∩ B) → μ(A)μ(B) as n → ∞? |

### `entropy` — Kolmogorov-Sinai Entropy

| Function | Description |
|----------|-------------|
| `kolmogorov_sinai_entropy(space, transform, partition)` | h(T) = sup_α h(T, α), the measure-theoretic entropy |
| `partition_entropy(space, partition)` | H(α) = −Σ μ(Aᵢ) log μ(Aᵢ) |
| `conditional_entropy(space, partition_a, partition_b)` | H(α \| β) = H(α ∨ β) − H(β) |

### `lyapunov` — Lyapunov Exponents

| Function | Description |
|----------|-------------|
| `compute_lyapunov_exponent(jacobian_fn, initial_state, iterations)` | λ = lim (1/n) Σ ln\|DT(xᵢ)\| — exponential divergence rate |
| `max_lyapunov_exponent(system, state, iterations)` | Largest Lyapunov exponent for multi-dimensional systems |

### `perron_frobenius` — Transfer Operator

| Function | Description |
|----------|-------------|
| `compute_invariant_measure(transition_matrix)` | Find the fixed point of the Perron-Frobenius operator via eigendecomposition |
| `perron_frobenius_matrix(transition_matrix)` | Construct the P-F operator matrix |

### `markov` — Markov Chains

| Type/Function | Description |
|---------------|-------------|
| `MarkovChain` | A finite-state Markov chain with transition matrix |
| `stationary_distribution()` | Compute the invariant measure |
| `is_ergodic()` | Check irreducibility + aperiodicity |
| `as_measure_preserving()` | Lift to a measure-preserving system on sequence space |

### `shannon_mcmillan` — Shannon-McMillan-Breiman Theorem

| Function | Description |
|----------|-------------|
| `empirical_entropy_along_orbit(space, transform, partition, orbit_length)` | Entropy rate along a single orbit — converges to h(T) a.e. |

### `agent_prediction` — Behavioral Prediction

| Function | Description |
|----------|-------------|
| `predict_exploration(agent_transitions, state_space_size)` | Will the entity explore the full state space? |
| `long_term_visit_frequency(agent_transitions, state_space_size)` | Predicted stationary distribution of visits |
| `mixing_time(agent_transitions, epsilon)` | How many steps until "close to stationary"? |

---

## The Math

### Measure-Preserving Systems

A **measure-preserving transformation** T on a probability space (X, 𝒜, μ) satisfies:

> μ(T⁻¹(A)) = μ(A) for all measurable A ∈ 𝒜

### Birkhoff's Ergodic Theorem

For an ergodic measure-preserving system (X, 𝒜, μ, T) and f ∈ L¹(μ):

> (1/N) Σᵢ₌₀ᴺ⁻¹ f(Tⁱx) → ∫ f dμ  almost everywhere as N → ∞

### Ergodicity

T is **ergodic** if every T-invariant set has measure 0 or 1:

> T⁻¹(A) = A ⟹ μ(A) ∈ {0, 1}

### Mixing

**Weak mixing**: (1/N) Σⁿ₌₀ᴺ⁻¹ |μ(T⁻ⁿ(A) ∩ B) − μ(A)μ(B)| → 0

**Strong mixing**: μ(T⁻ⁿ(A) ∩ B) → μ(A)μ(B) as n → ∞

Strong mixing ⟹ weak mixing ⟹ ergodic.

### Kolmogorov-Sinai Entropy

h(T) = sup_α h(T, α) where h(T, α) = lim (1/n) H(∨ᵢ₌₀ⁿ⁻¹ T⁻ⁱα)

### Lyapunov Exponents

λ = lim (1/n) Σ ln ‖DT(xᵢ)‖

Positive λ means nearby trajectories diverge exponentially — a hallmark of chaos.

### Shannon-McMillan-Breiman

For an ergodic system with partition α:

> −(1/n) log μ(αⁿ(x)) → h(T)  a.e.

---

## License

MIT OR Apache-2.0
