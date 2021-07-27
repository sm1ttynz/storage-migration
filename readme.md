# VCD VM Storage Profile Migration

## Overview

This project aspires to provide our customers a tool to aid with simple storage migration tasks.

### Use-Case

As a customer of Datacom Cloud,  I want to be able to migrate VCD bound VM storage from one profile/policy to another so that I can manage storage performance and storage costs.

As a customer of Datacom Cloud,  I want to be able to identify VM's which have hard disks that have not been migrated so that I can plan manual actions to remove remaining VM storage.

### Constraints

This script is designed to handle simple VM storage implementations e.g. a single storage profile configured at the VM level for all VM disks.

**Note:** If a storage profile/policy is set at the VM hard disk level, these will not be updated via the script.
#### Migration skip conditions

These conditions will cause VM's to be skipped and warnings logged.  This simply means the script will not handle the conditions, not necessarily that the VM storage profile/policy cant be migrated.

- Multiple storage polices are found per VM
- Media is attached
- VM's are not in a complaint state

## Pre-requisites

- Python 3.8 - Tested on 3.8

### Planning

- Ensure there is sufficient capacity in the target storage profiles to meet the needs to any planned migrations.
- Confirm all backup actions have completed successfully.
- Eject attached media where possible
- Complete testing migrations in a dev/test/non-prod environment starting small and ramping up with confidence.  Targeting vapps may help to focus on specific applications/business services.

## Script Setup

1. Copy the contents of this repo a server (testing on windows) which has access to the vCD api.
1. From you bash or cmd terminal set your working directory to the same level as the *storage-migration.py* file.

## Python Installation

Ensure your have Python for Windows 3.8.x installed, if not download and install from the following:

https://www.python.org/downloads/windows/

## Virtual Environment Usage - (No Internet Access)
If your system doesn't have internet access then activate the virtual environment packaged with this repository.
First though, you need to add the Python installation path to the python environment config file. 

From the script working directory:

1. Update the python configuration file path : 
    ```bash
        python.exe -c "import os, sys; print(os.path.dirname(sys.executable))"
    ```
    Edit the python config file and with the Python Install Path
        \venv\pyenv.cfg

1. Activate the Python Virtual Environment

```bash
    "venv\Scripts\activate"
```

or

```bash
    source "venv\Scripts\activate"
```

### Required Python libraries check

1. test your environment has the right libraries installed.

Virtual Environment
```bash
    "venv\Scripts\python.exe" -m pip list
```

Standard Python install with internet access
```bash
    python -m pip list
```

As long as *pyvcloud* is in the list then the dependant libraries will also be installed.

## Virtual Environment Deactivation

1. Either close your CMD terminal window or simply run the following command:

```bash
    "venv\Scripts\deactivate"
```

### Parameters

| Parameters | Description | Defaults | Required |
| :-- | :-- | :-- | --- |
| --vcd_host | FQDN or IP of VMware Cloud Director | None | true |
| --org | Name of the target Organisation | None | true |
| --vdc | Name of Virtual DataCenter | None | true |
| --vapp | Name of target vapp | None | false |
| --user | Username to authenticate to VCD with permissions to manage the VDC | None | true |
| --source_storage_profile |  Storage profile name which VM's should be migrated from | None | true |
| --target_storage_profile | Storage profile name which VM's should be migrated to | None | true |
| --no_checkmode | Turns off checkmode to take required action to update the desired Storage Profile | false | false |
| --silent | Silences user prompts | false | false |
| --ignore_errors | Continue on VM update errors | false | false |
| --exclude_txt | text string used to skip vApps or VM's if string found in the vApp or VM name - ** note use the equals ='name'. Exclude overrides include  | None | false |
| --include_txt | text string used to include VM's if string found in the VM name. include_text is processed before exclude_text ** note use the equals ='name'  | None | false |
| --max_size | Exclude VM's based on max diskspace | None | false

### Storage Profile naming table

| Portal | vCD |
| :-- | :-- |
| Storage Tier 1 | Tier1 |
| Storage Tier 2 | Tier2 |
| All Flash Array - Gold | Tier7 |
| All Flash Array - Diamond | Tier6 |

## Examples

1: Execute the script in dry-run/checkmode with user prompts.

  *Optionally* add `--vapp <vapp name>` to target a specific vapp.

```bash
    "venv\Scripts\python.exe" storage-migration.py --vcd_host <vcd.demo.com> --org <org_name> --vdc <vdc_name> --user <username> --source_storage_profile "<storage profile name>" --target_storage_profile "<target_storage_profile>"
```

2: Execute the script in without user prompts and ignore vm errors

```bash
    "venv\Scripts\python.exe" storage-migration.py --vcd_host <vcd.demo.com> --org <org_name> --vdc <vdc_name> --user <username> --source_storage_profile "<storage profile name>" --target_storage_profile "<target_storage_profile>" --no_checkmode --silent --ignore_errors
```

3: Execute the script in without user prompts and ignore vm errors and exlude VM's based on text found in the VM name and exclude based on max_size

```bash
    "venv\Scripts\python.exe" storage-migration.py --vcd_host <vcd.demo.com> --org <org_name> --vdc <vdc_name> --user <username> --source_storage_profile "<storage profile name>" --target_storage_profile "<target_storage_profile>" --no_checkmode --silent --ignore_errors --exclude_txt=test --max_size=1000
```

**Notes:**

- Ensure all backups are complete
- VM's are in a compliant state
- Media has been ejected

## Test cases

- A VM with a single storage profile can be update to a new available storage profile
- A VM with the following conditions will be skipped and a log entry will be added to the warning log.

  - multiple storage profiles
  - media attached
  - non-compliant state
  - snapshots

- Scripts run without `--no-checkmode` should not make changes
- When `--silent` is not used only 'y' or 'Y' will cause the script to continue
