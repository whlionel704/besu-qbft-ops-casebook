# Besu QBFT Operations Casebook

This repository is a documentation-only casebook capturing my practical learning in Besu QBFT platform engineering. It records my local setup notes, platform experiments, failure-mode simulations, mock investigations, and troubleshooting observations across Kubernetes operations, validator networking, observability, and blockchain platform reliability.

---

## 1. Objective of This Repository

The objective of this repository is to document my learning journey in operating and troubleshooting a private Besu QBFT blockchain network on Kubernetes.

This repository focuses on:

- how a private QBFT validator network is deployed locally;
- how validator, bootnode, RPC, metrics, and P2P components fit together;
- how Kubernetes configuration changes affect blockchain platform behaviour;
- how to document failure-mode simulations and mock investigations;
- how to reason about platform health beyond basic Kubernetes pod status.

This repository is **not** the upstream Besu Kubernetes deployment itself. It is a structured record of my experiments, observations, and operational learning.

---

## 2. Upstream QBFT Network Source and Local Setup

The QBFT network used for this learning project comes from the open-source Consensys `quorum-kubernetes` repository.

```text
Upstream repository:
https://github.com/Consensys/quorum-kubernetes
```

I cloned the upstream repository into my local development machine and customized selected configuration files for learning purposes.

My local setup was:

```text
Local PC / Laptop
        |
        v
Minikube Kubernetes cluster
        |
        v
quorum-kubernetes Helm-based deployment
        |
        v
Private Besu QBFT network
```

The network was deployed into my local Minikube cluster using the Helm charts and configuration structure from the upstream repository. My changes were focused on adapting the local environment, understanding the Besu node configuration, and preparing the setup for platform-observability and troubleshooting experiments.

The local network contains the main components required for a private Besu QBFT environment:

```text
Bootnodes
Validator nodes
RPC endpoints
Metrics endpoints
Prometheus/Grafana monitoring components
```

---

## 3. Why I Used an Existing Open-Source Repo

I used an existing open-source repo because my goal was not to prove that I could build an entire Besu Kubernetes deployment from scratch.

My goal was to learn the platform engineering side of a private blockchain network:

```text
How do Besu validators discover each other?
How does P2P communication affect network health?
What does a healthy validator network look like?
What does an unhealthy validator network look like?
Which metrics and logs matter during troubleshooting?
How can incidents be reconstructed after the fact?
```

Using the upstream repo allowed me to start from a realistic deployment foundation and focus my effort on operating, modifying, observing, breaking, recovering, and documenting a Besu QBFT network.

This is closer to real platform engineering work, where engineers often operate and customize existing platforms instead of building every component from zero.

---

## 4. High-Level Architecture of My Local Private QBFT Network

The private QBFT network runs inside my local Minikube Kubernetes cluster.

```text
Developer Laptop
      |
      v
Local Minikube Cluster
      |
      v
besu namespace
      |
      +--> bootnode-1
      +--> bootnode-2
      |
      +--> validator-1
      +--> validator-2
      +--> validator-3
      +--> validator-4
      |
      +--> Prometheus

```

The validator nodes communicate with each other over Besu P2P networking.

```text
validator-1  <------ P2P ------>  validator-2
     |                                |
     |                                |
    P2P                              P2P
     |                                |
     v                                v
validator-3  <------ P2P ------>  validator-4
```

The important Besu ports in this local setup are:

| Port | Purpose |
|---:|---|
| `8545` | JSON-RPC HTTP |
| `8546` | WebSocket RPC |
| `8547` | GraphQL |
| `30303` | P2P / RLPx node-to-node communication |
| `9545` | Metrics endpoint for Prometheus |

---


## 5. Ethical Use of Open Source

This repository uses an open-source project as a learning foundation.

I do not claim that I created the original `quorum-kubernetes` project.

The upstream project belongs to its original authors and contributors.

This repository is my own documentation of:

```text
local setup decisions
configuration changes
platform observations
failure-mode simulations
mock investigations
troubleshooting workflows
lessons learned
```
