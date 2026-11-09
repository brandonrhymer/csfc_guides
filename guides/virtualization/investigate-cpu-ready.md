# Investigate high CPU Ready

**Status:** Technical review draft. Lab validation required. The source's proposed virtual-socket change is not a validated general fix.

## Scope

Inspect CPU Ready for a VM and determine whether an approved configuration change is justified. The source targets ESXi hosts with one physical CPU socket and proposes changing cores per virtual socket without changing total vCPU count.

CPU Ready can reflect several scheduling and workload conditions. A physical host with one socket does not, by itself, establish that the VM should use one virtual socket.

## Prerequisites

- Confirm the ESXi, vSphere, virtual hardware, and guest versions.
- Identify workload symptoms and collect a representative baseline.
- Confirm the chart's units, sampling interval, and whether values are aggregated across vCPUs.
- Record the VM's total vCPU count, cores per socket, CPU limit, and current host configuration.
- Obtain an approved maintenance, rollback, and validation plan before changing topology.

## 1 Inspect CPU Ready

1. Open vSphere and select **Menu > Hosts and Clusters**.
2. Select the VM.
3. Open **Monitor > Performance > Advanced**.
4. Select the CPU chart and the CPU Ready counter.
5. Review a representative time window. The source uses the last hour.
6. Correlate the readings with the reported workload symptoms.

**Threshold review:** The source uses 5% as an action threshold and 0–2% as an ideal range. These are unvalidated source targets, not universal limits. Do not compare a millisecond summation, a VM aggregate, and a per-vCPU percentage as if they were equivalent. Confirm the metric interpretation and deployment-specific threshold before deciding to act.

### Convert a summation counter

For a CPU Ready summation value measured in milliseconds, divide it by the sample duration in milliseconds and multiply by 100. Use the actual interval for the selected chart.

For example, 800 ms over 20 seconds is `800 / (20 * 1000) * 100 = 4%`. If that value is the sum across four vCPUs, the average is 1% per vCPU. Do not divide a per-vCPU counter again, and inspect individual vCPUs because an average can hide a busy vCPU.

## 2 Record the current settings

1. Open **Actions > Edit Settings > CPU** for the VM.
2. Record total vCPU count, cores per socket, and CPU limit.
3. Open the host's **Summary** and record its physical CPU socket count.
4. If the host has more than one physical socket, the source's proposed procedure is outside its stated scope. Escalate for a platform-specific assessment.

## 3 Evaluate the source proposal

The source proposes setting **Cores per socket** equal to total vCPU count, while leaving total vCPU unchanged. For example, a VM with four vCPUs would be presented as one virtual socket with four cores.

Do not apply this merely because CPU Ready is high. The technical owner must first confirm how virtual socket topology, vNUMA, guest behavior, workload sizing, and licensing interact in this configuration. Changing the displayed socket count is not a universal scheduler fix, and unchanged vCPU count does not prove that all software licensing is unaffected.

If the proposal is explicitly approved and supported:

1. Follow the product's required power-state and change procedure. A guest reboot alone may not satisfy a required VM power-off.
2. Open the VM's CPU settings.
3. Keep total vCPU count unchanged.
4. Set the approved cores-per-socket value.
5. Apply the change and return the VM to service under the approved procedure.

## 4 Compare results

1. Allow a comparable post-change measurement interval to elapse.
2. Review CPU Ready with the same counter interpretation and a comparable workload.
3. Confirm application health and performance.
4. If the change does not meet the agreed acceptance criteria, use the approved rollback or escalation plan.

Do not use a one-hour chart containing mostly pre-change data as immediate proof that the change helped or failed.

## 5 Inspect other constraints

- Review the VM CPU limit. The source expects **Unlimited**, but do not change an intentional limit without approval.
- Review host CPU utilization. The source suggests escalation for sustained 85–90% usage; treat that as an unvalidated source heuristic until the technical owner confirms a threshold.
- Have the platform owner investigate contention, workload sizing, and other scheduler constraints as appropriate. This source is not a complete performance-diagnosis runbook.

## Completion record

Record the counter units and interval, baseline and post-change measurements, workload conditions, unchanged total vCPU count, old and new topology, power-state changes, application outcome, and any rollback. Keep real VM and host identifiers in the authorized ticket, not in this guide collection.

## References for technical review

- [Broadcom guidance on converting CPU summation and CPU percentage](https://knowledge.broadcom.com/external/article/306576/converting-between-cpu-summation-and-cpu.html) explains measurement conversion. Apply the correct sampling interval and distinguish VM aggregate from per-vCPU results.
- [Broadcom guidance on rightsizing VMs on ESXi 8](https://knowledge.broadcom.com/external/article/438023/rightsizing-virtual-machines-on-esxi-80.html) provides version-specific sizing and topology guidance. Modern virtual NUMA behavior is not determined solely by the displayed cores-per-socket value. Confirm applicability to the installed release and virtual hardware.

[Return to guide index](../README.md)
