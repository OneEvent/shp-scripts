# YAML Script Structure

## Overview
- ProtocolWorkbench uses YAML scripts for two related workflows:
  - test scripts in `shp-scripts/test`
  - manufacturing scripts in `shp-scripts/manufacturing`
- The scripts are intentionally simple so they can be read by people, the app, and AI agents.
- Both runners load the YAML into a top-level document with ordered steps.
- Both runners support variable substitution with `${variable_name}` inside step arguments.

## Shared Top-Level Shape
Most scripts follow this general layout:

```yaml
name: Human-readable script title
description: Optional summary of what the script does
stop_on_fail: true
steps:
  - name: First step
    kind: rpc
    call: Some Method
    args:
      some-arg: some-value
    expect:
      status: "0"
```

### Top-Level Fields
- `name`: required, human-readable title.
- `description`: optional summary shown in UI where supported.
- `stop_on_fail`: optional, defaults to `true`.
- `steps`: required ordered list of step definitions.

### Common Step Fields
- `name`: required label for the step.
- `purpose`: optional explanation of why the step exists.
- `kind`: optional execution type, defaults to `rpc`.
- `call`: required operation name or method name.
- `args`: optional key/value map of inputs to the call.
- `expect`: optional block used to decide whether the step passed.
- `save_as`: optional output mapping.

### Variable Substitution
- Any argument value may reference a previously saved variable with the form `${variable_name}`.
- The runner resolves those values at execution time.
- If a variable is missing, the step fails.

## Test Script Format
Test scripts are loaded by the test runner and usually live in `shp-scripts/test`.

### Required Test-Script Fields
- `key`: canonical script identifier used for linking and lookup.
- `name`: script title.
- `steps`: ordered list of actions.

### Optional Test-Script Fields
- `description`: script summary.
- `stop_on_fail`: defaults to `true`.
- `azure_devops`: linkage metadata for Azure Test Plans.

### Azure DevOps Block
The test runner understands this optional block:

```yaml
azure_devops:
  plan_id: 1300
  suite_id: 1301
  test_case_id: 1302
  point_ids:
    - 1303
```

- `plan_id`: Azure test plan ID.
- `suite_id`: Azure suite ID.
- `test_case_id`: Azure test case ID.
- `point_ids`: optional list of point IDs.

### Test Step Expectations
- Test steps use `expect.status`.
- The status value should be the numeric RPC status code as a string.
- Common values include `"0"` for success and `"6"` for `InternalError`.
- The default is `"0"` when no status is supplied.

### Test Step Save-As
- `save_as` stores one output value under one variable name.
- Example:

```yaml
save_as: firmware_version
```

## Manufacturing Script Format
Manufacturing scripts are loaded by the manufacturing runner and usually live in `shp-scripts/manufacturing`.

### Required Manufacturing Fields
- `name`: required script title.
- `steps`: ordered list of actions.

### Optional Manufacturing Fields
- `description`: script summary.
- `stop_on_fail`: defaults to `true`.

### Manufacturing Step Kinds
The manufacturing runner supports these step kinds:
- `rpc`: send an OpenRPC method to the device.
- `api`: call a provisioning or trust-center API.
- `local`: run an internal helper action.
- `programming`: trigger LinkServer firmware programming.

### Manufacturing Step Expectations
- Manufacturing steps use numeric status strings, usually `"0"` for success.
- Some API steps also check `expect.outcome`.
- Example:

```yaml
expect:
  status: "0"
  outcome: "Issued"
```

### Manufacturing Save-As
- `save_as` is a field-to-variable map.
- The runner looks for a response field name on the left and stores it under the variable name on the right.
- Example:

```yaml
save_as:
  certificateB64: uart_device_cert_b64
  trustPolicyB64: uart_trust_policy_b64
```

## Common Patterns
- Keep step names short and descriptive.
- Use `purpose` when the step would not be obvious from `call`.
- Save only the values needed by later steps.
- Prefer stable keys and IDs so scripts remain portable across environments.
- Leave commented-out template steps in place when they are useful to future operators.

## Current Reference Files
- Test example: `shp-scripts/test/get-firmware-version.yaml`
- Manufacturing example: `shp-scripts/manufacturing/commission-fresh-device.yaml`
