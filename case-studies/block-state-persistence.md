## Blockchain Platform Engineering Case Study

### Resolving Besu QBFT Validators Restarting from Block 0 After Minikube Restart

## 1. Problem Statement

A Hyperledger Besu QBFT network deployed on Kubernetes with Helm was expected to resume from the latest block after 'minikube stop' and 'minikube start'. Instead, validator nodes repeatedly restarted from block 0 / genesis after Minikube restarts, even though Kubernetes PVC/PV objects existed. The objective was to identify the root cause, fix the storage design, and prove that validators could resume from the last persisted block height.

## 2. Describe the Problem

The network successfully produced blocks before restart. At around block 104, the Besu database under /data/besu was approximately 75 MB. After Minikube restarted, /data/besu became almost empty and block production restarted from block 0 / block 1. This showed that the blockchain state was not truly durable at the underlying storage layer.

| Expected behaviour | Actual behaviour |
|---|---|
| Latest block = 104 -> Minikube restart -> validators resume from block 104+ | Latest block = 104 -> Minikube restart -> validators restart from block 0 / genesis |

## 3. What was done

- Verified Besu logs and confirmed block production restarted from block 0 after Minikube restarts.
- Checked Besu config.toml and confirmed data-path=/data/besu.
- Inspected pod mounts and confirmed /data/besu was backed by a Kubernetes PVC.
- Measured database size before and after restart using du -sh /data/besu.
- Inspected PV YAML and found the backing hostPath was /tmp/besu-node-validator-*.
- SSHed into Minikube and verified /data was a persistent ext4 mount.
- Updated the Helm storage template to make hostPath configurable through storage.hostPathBase.
- Reinstalled genesis, bootnodes and validators together to ensure consistent QBFT network configuration.
- Validated that Besu detected the existing database and resumed from a later block height instead of block 0.

![Investigation flow: layered platform debugging](assets/block-state-persistence/image1.png)

## 4. Challenges / Blockers and Resolutions

| Challenge | Investigation | Resolution | Outcome |
|---|---|---|---|
| Validators restarted from block 0 after Minikube restart | Compared block height and /data/besu size before/after restart; observed that database size dropped from ~75 MB to almost 0 MB. | Moved investigation below Besu into Kubernetes PV/PVC and host filesystem. | Confirmed it was definitely a storage persistence issue |
| PVC/PV existed but data still vanished | Inspected PV YAML and found hostPath=/tmp/besu-node-validator-* despite PVC being Bound. | Identified /tmp as unsuitable for long-lived blockchain state. | Root cause isolated to local PV backing path. |
| Helm chart hardcoded /tmp | Used grep -R "hostPath:" . and found that charts/besu-node/templates/node-storage.yaml contained /tmp/… as the database file directory | Changed hostPath to {{ .Values.storage.hostPathBase }}/{{ include "besu-node.fullname" . }} and set hostPathBase=/data/besu. | New PVs used durable /data/besu paths. |
| Old genesis with re-installed nodes prevented QBFT consensus | Pods started but stayed in sync/peer evaluation logs, e.g. Unable to find sync target, and did not produce blocks. | Reinstalled genesis, bootnodes and validators together as one consistent network set. | Validator set, keys, static peers and genesis aligned; QBFT block production resumed. |

## 5. Storage Architecture Before / After

![Storage architecture before and after](case-studies/assets/block-state-persistence/image2.png)

## 6. Key Helm Changes Made

The original template created local PersistentVolumes under /tmp. The fix parameterized the base path and configured it to use /data/besu.

![Key Helm changes made](qbft_assets/image3.png)

## 7. Learning Points

- A PVC being bound to a PV does not guarantee durable blockchain state; the PV backing storage location must also be durable.
- persistentVolumeReclaimPolicy: Retain does not protect data if the hostPath itself is temporary.
- For stateful blockchain systems, validation must include restart testing and block-height continuity.
- Helm templates can hide infrastructure assumptions; hardcoded paths should be environment-configurable.
- Genesis, validator keys, static peers, bootnodes and validators must be deployed consistently for QBFT consensus.

## 8. Business Value

This fix improves platform reliability by ensuring local Besu validator state survives cluster restarts. It reduces false diagnosis of Besu/QBFT issues, shortens recovery time during local testing, and makes the Helm chart safer across environments by replacing a hardcoded temporary path with a configurable persistent path.

## 9. Future Work

- Run a 24/7 block-production soak test to prove the QBFT network continuously produces blocks without manual intervention.
- Add Prometheus/Grafana checks for latest block height, validator peer count, pod restarts, RPC availability, disk usage and block-production stalls.
- Create alerts for: no new block in 60 seconds, peer count below threshold, validator pod restart, RPC unavailable and disk usage above threshold.

Besu QBFT Persistent Storage Case Study | Platform Engineering Portfolio
