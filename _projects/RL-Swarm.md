---
layout: page
title: RL-Swarm
description: Multi-Agent Reinforcement Learning Framework with Pheromone-Based Coordination
img: assets/img/projects/RLSwarm/RLSwarm.png
importance: 4
category: Master's
related_publications: false
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/rl_swarm_simulation.jpg" title="RL-Swarm Simulation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Simulation of swarm agents coordinating through pheromone-based communication in a grid environment.
</div>

## Abstract

**RL-Swarm** is a research-oriented Multi-Agent Reinforcement Learning (MARL) framework designed to investigate emergent cooperative behaviors in decentralized systems. Extending the foundational `RL-swarms` library, this project introduces a biologically inspired **pheromone-based perception mechanism**, allowing agents to communicate indirectly via the environment (stigmergy). The framework supports comparative analysis between centralized and decentralized learning paradigms using algorithms such as SARSA, Q-Learning, and Deep Q-Networks (DQN), providing a testbed for optimizing swarm intelligence in dynamic environments.

## System Architecture

The framework is built upon a modular architecture that separates the simulation environment from the learning agents.

### Environment Dynamics
The simulation, based on the **Slime** model, features:
*   **Pheromone Field**: A dynamic grid where agents deposit chemical trails that diffuse and evaporate over time, creating a temporal memory of collective activity.
*   **Local Perception**: Agents possess a limited field of view, sensing pheromone concentrations and neighbor positions only within a defined radius.
*   **Scalability**: Optimized to support simulations ranging from 10 to over 100 concurrent agents.

### Learning Algorithms
The framework implements a suite of RL algorithms to solve the coordination problem:
*   **SARSA (On-Policy)**: Learns the value of the policy being followed, promoting safer exploration.
*   **Q-Learning (Off-Policy)**: Learns the optimal policy independently of the agent's actions.
*   **Deep Q-Networks (DQN)**: Utilizes neural networks to approximate the Q-function, enabling the handling of high-dimensional state spaces (e.g., large pheromone grids).

## Implementation Details

### State Representation
The agent's observation space is constructed to facilitate spatial awareness:
```python
state = {
    'position': (x, y),                    # Absolute position (normalized)
    'pheromone_map': local_radius_grid,    # Convolutional view of local pheromones
    'agent_neighbors': nearby_agents,      # Relative coordinates of peers
    'velocity': (vx, vy),                  # Current heading
    'positional_encoding': pos_enc         # Sinusoidal encoding for grid localization
}
```

### Reward Structure
To induce cooperative behavior, the reward function balances individual and collective goals:
$$ R_t = \alpha \cdot R_{coverage} + \beta \cdot R_{pheromone} + \gamma \cdot R_{coordination} - \delta \cdot R_{collision} $$

### Neural Network Architecture (DQN)
For the Deep Learning approach, the agent utilizes a custom architecture:
*   **Input Heads**: Separate processing streams for scalar (position, velocity) and grid (pheromone map) data.
*   **CNN Encoder**: Processes the local pheromone grid to extract spatial features.
*   **Fusion Layer**: Concatenates processed features before passing them to the decision-making MLP.

## Key Features

*   **Stigmergic Coordination**: Demonstrates how complex group behaviors can emerge from simple, local interactions without direct communication.
*   **Ablation Studies**: Includes tools for evaluating the impact of specific features like positional encoding and normalization.
*   **Gymnasium Integration**: Fully compatible with the modern `gymnasium` API for seamless integration with standard RL libraries like `stable-baselines3`.

## Resources

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
    <div class="repo p-2 text-center">
        <a href="https://github.com/DanielRossi1/RL-Swarm" class="btn btn-primary z-depth-1">
            <i class="fab fa-github"></i> View Code
        </a>
    </div>
</div>
