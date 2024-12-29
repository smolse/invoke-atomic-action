# invoke-atomic-action

This action executes [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) tests and
[adversary emulation](https://www.atomicredteam.io/invoke-atomicredteam/docs/adversary-emulation) schedules directly
on GitHub Actions runners with the help of the
[Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam) PowerShell framework. Linux, macOS, and
Windows runners are supported.

## Usage

Currently, the action exposes minimal configuration options for the `Invoke-AtomicTest` and `Invoke-AtomicRunner`
cmdlets used to execute tests and schedules. It primarily relies on the default settings and the default atomics
location (`C:\AtomicRedTeam\atomics` on Windows and `~/AtomicRedTeam/atomics` on Linux/macOS). The action installs
`Invoke-AtomicRedTeam` from the PowerShell Gallery and downloads atomics from the Atomic Red Team GitHub repository if
they are not already present on the runner.

It's possible to customize the behavior of `Invoke-AtomicRunner` by placing a `privateConfig.ps1` script in the
directory where the `Invoke-AtomicRedTeam` module is installed, as described in the
[Invoke-AtomicRedTeam documentation](https://www.atomicredteam.io/invoke-atomicredteam/docs/continuous-atomic-testing).

Execution logs are uploaded to the GitHub Actions workflow run as artifacts upon successful completion.

### Inputs

- `technique`

  ID of the MITRE ATT&CK technique to execute. For example:

  ```yaml
  with:
    technique: T1033
  ```

  - Type: string
  - Optional (must be provided if the `adversary-emulation` input parameter is set to "false")

- `test-names`

  Comma-separated list of Atomic Red Team test names to execute. For example:

  ```yaml
  with:
    technique: T1033
    test-names: User Discovery With Env Vars PowerShell Script,GetCurrent User with PowerShell Script
  ```

  - Type: string
  - Optional

- `test-numbers`

  Comma-separated list of Atomic Red Team test numbers to execute. For example:

  ```yaml
  with:
    technique: T1033
    test-numbers: 4,5
  ```

  - Type: string
  - Optional

- `test-guids`

  Comma-separated list of Atomic Red Team test GUIDs to execute. For example:

  ```yaml
  with:
    technique: T1033
    test-guids: dcb6cdee-1fb0-4087-8bf8-88cfd136ba51,1392bd0f-5d5a-429e-81d9-eb9d4d4d5b3b
  ```

  - Type: string
  - Optional

- `test-input-args`

  Stringified JSON object containing the input arguments for the Atomic Red Team test. For example:

  ```yaml
  with:
    technique: T1140
    test-guids: 356dc0e8-684f-4428-bb94-9313998ad608
    test-input-args: '{"message":"hello world"}'
  ```

  - Type: string
  - Optional

- `logging-module`

  Name of the logging module to use for the Atomic Red Team test execution. For example:
    
  ```yaml
  with:
    technique: T1033
    logging-module: Attire-ExecutionLogger
  ```
    
  - Type: string
  - Optional
  - Default: `Default-ExecutionLogger`

- `get-prereqs`

  Flag to indicate whether to try to install the prerequisites for the Atomic Red Team tests. For example:

  ```yaml
  with:
    technique: T1033
    get-prereqs: false
  ```

  - Type: string
  - Optional
  - Default: `true`

- `cleanup`

  Flag to indicate whether to clean up the Atomic Red Team test artifacts after execution. For example:

  ```yaml
  with:
    technique: T1033
    cleanup: false
  ```

  - Type: string
  - Optional
  - Default: `true`

- `adversary-emulation`

  Flag to indicate whether to execute an adversary emulation schedule instead of a single Atomic Red Team test.

  - Type: string
  - Optional
  - Default: `false`

- `list-of-atomics`

  Path to the CSV schedule file containing the list of Atomic Red Team tests to execute. For example:

  ```yaml
  with:
    adversary-emulation: true
    list-of-atomics: ./IcedID.csv
  ```

  - Type: string
  - Optional (must be provided if the `adversary-emulation` input parameter is set to "true")

### Example Workflows

#### Execute Atomic Red Team Test

```yaml
name: Run Atomic Red Team Test
on:
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - uses: smolse/invoke-atomic-action@main
        with:
          technique: T1033
          test-numbers: 2
```

#### Execute Atomic Red Team Adversary Emulation

```yaml
name: Run Adversary Emulation
on:
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  adversary-emulation:
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - uses: smolse/invoke-atomic-action@main
        with:
          adversary-emulation: true
          list-of-atomics: ./IcedID.csv
```
