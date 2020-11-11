# Interact with User if Logged On
When deploying software to remote systems, it is often necessary to interact with the user in order to close applications, or reboot their system. SCCM disallows interaction with user unless a user is logged on. Find a way around this.

## SCCM Deployment
_User Experience_ should be 'whether or not user is logged on' and do not check 'allow user to interact'.

## Check for User
There are several states to consider. Deployments can be delayed greatly by user logons, or machine sitting idly in an office somewhere. The main cases to consider are:
* **No user logged on** &mdash; if you do not need the user logged on, this is the best case
* **User logged on (active)** &mdash; user action must be negotiated, or if you need a user logged on, this is a best case
* **User logged on (inactive)** &mdash; though logged on, the system idle time is greater than your determined threshold (say, one hour)

### Determining User State
``Get-CimInstance`` will help us divine if a user is logged on.

## References
* [Get-CimInstance by Microsoft](https://docs.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance?view=powershell-7)
