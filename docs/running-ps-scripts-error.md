---
tags:
- powershell
- windows
---

# How to fix "running scripts is disabled on this system"?

1. First, Open PowerShell with Run as Administrator.
2. Then, run this command in PowerShell.
3. Type Y  and press Enter to confirm.
   
``` 
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Params

#### `-ExecutionPolicy`

The acceptable `-ExecutionPolicy` values are as follows:

- <b class="malachite">AllSigned.</b> Requires that all scripts and configuration files are signed by a trusted publisher, including scripts written on the local computer.
- <b class="malachite">Bypass.</b> Nothing is blocked and there are no warnings or prompts.
- <b class="malachite">Default.</b> Sets the default execution policy. Restricted for Windows clients or RemoteSigned for Windows servers.
- <b class="malachite">RemoteSigned.</b> Requires that all scripts and configuration files downloaded from the Internet are signed by a trusted publisher. The default execution policy for Windows server computers.
- <b class="malachite">Restricted.</b> Doesn't load configuration files or run scripts. The default execution policy for Windows client computers.
- <b class="malachite">Undefined.</b> No execution policy is set for the scope. Removes an assigned execution policy from a scope that is not set by a Group Policy. If the execution policy in all scopes is Undefined, the effective execution policy is Restricted.
- <b class="malachite">Unrestricted.</b> Beginning in PowerShell 6.0, this is the default execution policy for non-Windows computers and can't be changed. Loads all configuration files and runs all scripts. If you run an unsigned script that was downloaded from the internet, you're prompted for permission before it runs.

#### `-Scope`

Specifies the scope that is affected by an execution policy. The default scope is **LocalMachine**.

The effective execution policy is determined by the order of precedence as follows:

- <b class="malachite">MachinePolicy.</b> **Set by a Group Policy** for all users of the computer.
- <b class="malachite">UserPolicy.</b> **Set by a Group Policy** for the current user of the computer.
- <b class="malachite">Process.</b> Affects only the current PowerShell session.
- <b class="malachite">CurrentUser.</b> Affects only the current user.
- <b class="malachite">LocalMachine.</b> Default scope that affects all users of the computer.

The `Process` scope only affects the current PowerShell session. The execution policy is saved in the environment variable `$env:PSExecutionPolicyPreference`, rather than the registry. When the PowerShell session is closed, the variable and value are deleted.


### Notes

Execution policies for the **CurrentUser** scope are written to the registry hive `HKEY_LOCAL_USER`.

Execution policies for the **LocalMachine** scope are written to the registry hive `HKEY_LOCAL_MACHINE`.

`Set-ExecutionPolicy` <i>**doesn't**</i>: 

- change the **MachinePolicy** and **UserPolicy** scopes because they are set by Group Policies.
- override a Group Policy, even if the user preference is more restrictive than the policy.

### Reference

[MS Docs for Set-ExecutionPolicy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-7.3)