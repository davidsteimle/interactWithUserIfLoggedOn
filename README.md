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

## General Logic

In this case, no user is required for the installation. This is pseudocode:

```powershell
$UserLogonState = Get-CimInstance Win32_... | Select-Object -Property UserName
if($UserLogonState){
    # A user is logged on; deal with that.
    # Get user idle time.
    # Check for screensaver?
} else {
    # No user is logged on; proceed.
}
```

If a user is logged on, and we need to interact, we need to do so in the User context. In this case, however, SCCM is behaving purely in the System context.

One way to interact with the user is to create a scheduled task run as them. We already have the user name, so it is easy to build a scheduled task in XML, programatically, and import it into the Task Scheduler. See [UserXmlBuild.ps1](UserXmlBuild.ps1). When we run that task, it will appear in the user context and allow them to interact.

We need to parse the user's response. I do not have this methodology right now, but the basic responses should yield:
* **Ok** &mdash; user acknowledges the need to install software, close programs, initiate reboot. Proceed with your script and appropriate exit code is supplied to SCCM.
* **Dismiss** &mdash; user is not prepared to allow you to interfere with their work. Maybe they are in a meeting, or about to be in one. Maybe they have an issue that takes precedence over you upgrading some software? This response should have your script exit with code *1618* (fast retry). Your script will exit and tell SCCM to try again later; this will generally happen in two hours, beginning the process all over again.

### Cleanup
It might be a good idea to disable the scheduled task after the response is gained by your script, regardless of its nature.

## References
* [Get-CimInstance by Microsoft](https://docs.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance?view=powershell-7)
* [Getting the user idle time with C#](https://www.codeproject.com/articles/13384/getting-the-user-idle-time-with-c)
