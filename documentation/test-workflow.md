Test Workflow

Overview
- ProtocolWorkbench (PWB) runs scripted hardware tests over UART.
- The script lives in YAML.
- Azure DevOps Test Plans is used for test organization and result tracking.
- PWB executes the test and publishes the outcome back to the linked Azure test point.
- PWB can also import test-to-script linkage from Azure DevOps by reading a custom work item field from test cases in a configured suite catalog.

What must be set up in Azure DevOps
1. Create or choose a Test Plan.
2. Create or choose a Test Suite inside that plan.
3. Add a Test Case to that suite.
4. Create a custom work item field for PWB script linkage, for example `Custom.PwbScriptKey`.
5. On each Azure Test Case, set that field to the script key that PWB should run.
6. Make sure your Azure DevOps PAT, Organization, and Project are configured in PWB user secrets.

What must be set up in PWB
1. Open Connection Settings.
2. Select the correct UART / COM port.
3. Select the baud rate and RTS/CTS setting needed by the device.
4. Connect to the device.
5. Make sure the sibling repo `shp-scripts` is cloned next to `ProtocolWorkbench`.
6. Add the Azure suite catalog entry in `shp-scripts/test/suites.yaml`.
7. Confirm the Test page shows the correct Plan ID, Suite ID, and Case ID for the imported or linked script.

How the YAML script is used
- The YAML script defines:
  - script key
  - script name and description
  - Azure DevOps linkage metadata
  - ordered test steps
  - API method calls
  - arguments
  - expected numeric status values used for pass / fail
- The `key` field is the canonical machine-readable script identity.
- The `name` field is the human-readable title shown in the UI.
- During development, PWB prefers the source YAML file from the repo when it is available.
- PWB reads scripts directly from the shared `shp-scripts` repo.
- The Test page shows the active script file path.
- Use Open Script to edit the file.
- Use Reload Script after changes so PWB re-reads the YAML without a rebuild.
- For the shared YAML layout used by both test and manufacturing scripts, see [yaml-script-structure.md](yaml-script-structure.md).

How Azure linkage works
- There are two linkage layers:
  - Suite catalog YAML in PWB
  - Script key field on the Azure Test Case
- The suite catalog lives in `shp-scripts/test/suites.yaml`.
- Each suite catalog entry contains:
  - `plan_id`
  - `suite_id`
  - `script_field_name`
- PWB loads the configured suite entries from that repo YAML.
- PWB queries Azure DevOps for the test cases in those suites.
- PWB reads the custom field named by `script_field_name` from each Azure Test Case.
- That field should normally contain a stable script key, such as `get-firmware-version`.
- That Azure key should match the YAML script's `key:` field.
- PWB resolves that key to the YAML file in `shp-scripts/test`.
- For compatibility, PWB also accepts a repo-relative or absolute script path.
- After PWB loads the script, it publishes results back to the Azure test point for that plan, suite, and case.
- The script YAML can still include an `azure_devops` block, and those IDs are used when the script is run directly without suite import.

How to run a test from PWB
1. Confirm the device is connected on the Test page.
2. Confirm the script key resolved to the correct script file and the Azure IDs look correct.
3. Press Run in ProtocolWorkbench.
4. Review each row in the results table.
5. If Azure publishing is configured, use Open Azure Run to jump to the Azure result details.

What to look for in Azure DevOps
- Execute tab:
  - the test point outcome should update to Passed or Failed
- Runs page:
  - automated runs should appear in the run list
- Run details:
  - comments, failure text, and stack trace details should be visible

Pass / fail behavior
- Tests are allowed to fail.
- A failed device response should publish as a failed Azure test result.
- The current sample script intentionally includes a failing step to demonstrate failed reporting.

Recommended workflow
1. Create the Azure Test Case.
2. Add or confirm the YAML script `key:` value.
3. Set the PWB script key custom field on that Azure test case to the same value.
4. Add the suite entry once in `shp-scripts/test/suites.yaml`.
5. Open the Test page in PWB.
6. Verify that PWB imported the script and is showing the Azure IDs on screen.
7. Run the test from PWB and open the Azure run for details.

Current limitations
- The current runner is still a first-pass prototype.
- Help content is displayed as plain text for now rather than rendered Markdown.
- The suite catalog is repo-managed, so adding a new Azure suite still requires a matching YAML catalog entry in `shp-scripts`.
