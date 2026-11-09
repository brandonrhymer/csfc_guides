# Run GP 2.0 troubleshooting checks

**Status:** Incomplete source. Product identification and lab validation required before use.

## Scope

The source contains three Linux-style diagnostic commands. It does not identify what “GP 2.0” means, supported versions, expected output, or recovery steps. Do not infer a product from the abbreviation.

## Prerequisites

Confirm the product, operating system, approved test destination, service unit, CLI executable, required privileges, and escalation owner. Source-specific identifiers have been replaced with placeholders.

## Checks

### 1 Test permitted network reachability

```bash
ping <APPROVED_TEST_ADDRESS>
```

Use only an approved destination. ICMP failure alone does not establish a service failure because network policy may block it. Stop the command after the approved sample has been collected.

### 2 Inspect the service

```bash
systemctl status <SERVICE_NAME>
```

Confirm that the named unit belongs to the intended product. The product owner must define which states and errors require escalation.

### 3 Inspect the product link state

```bash
sudo <SERVICE_CLI> link show
```

Run only after the executable and subcommand are verified for the installed product. Use elevated privileges only when required and authorized.

## Validate and interpret

There is no product-specific pass/fail result yet. Record the command result, time, and observed symptom without treating successful ICMP as proof of application health. Do not invent service states or link-state meanings. The product owner must supply expected results and escalation criteria before this becomes an operational runbook.

## Record and escalate

Record whether each check completed and the relevant interpreted result in the approved support system. Diagnostic output may reveal identifiers or operational details; keep raw output out of this guide. Escalate under the product-specific runbook. The source provides no repair or restart authorization.

[Return to guide index](../README.md)
