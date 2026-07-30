# Golden Ticket Attacks

#### **Golden Ticket Attacks: What Is It?**

A **Golden Ticket** attack is one of the most powerful types of attacks in a **Kerberos**-based authentication system, like those used in Active Directory (AD). In a Golden Ticket attack, an attacker **compromises the `krbtgt` account** (the Kerberos Ticket Granting Ticket service account) and forges a **Kerberos Ticket Granting Ticket (TGT)**. This forged TGT allows the attacker to gain **unrestricted access to all resources and systems** within the domain.

#### **Golden Ticket Attack: Process Overview**

1. **Compromise the `krbtgt` Account**:

   * The attacker first needs to **obtain the NTLM hash** of the `krbtgt` account, which is the Kerberos Ticket Granting Ticket (TGT) service account in an Active Directory (AD) domain. This can be done using tools like **Mimikatz**.

   Using **lsadump** in Mimikatz, the attacker can dump the NTLM hash of the `krbtgt` account.

```bash
lsadump::lsa /inject /name=krbtgt
```

<figure><img src="/files/cYlH4OLGO8J0N1xh7dTy" alt=""><figcaption><p>copy the <code>krbtgt user</code> SID and NTLM hash</p></figcaption></figure>

**Forge a Golden Ticket**:

* With the **NTLM hash** and the **SID of the domain**, the attacker can create a **Golden Ticket** using the `krbtgt` account. The `kerberos::golden` command in Mimikatz is used to forge a TGT, which allows the attacker to impersonate any user, including an admin (e.g., `admin`).

  ```bash
  kerberos::golden /USER:admin /domain:marvel.local /sid:23412341234 /krbtgt:43gro9gro2qer /id:500 /ptt
  ```

<figure><img src="/files/7AjKNvhuZYtwGUo8d2Tc" alt=""><figcaption></figcaption></figure>

**Access Domain Resources**:

* Once the Golden Ticket is forged, the attacker has **unrestricted access** to the domain. Using tools like **Impacket**'s **`cdm`** (Command Line Client) or **PsExec**, the attacker can interact with machines within the domain.

For example, using **`cdm`**, the attacker can access a remote machine (e.g., `THEPUNSIER`) and run commands, like viewing the contents of the C$ share:

```bash
misc::cdm
dir //THEPUNSIER/c$
```

<figure><img src="/files/fzq9cy2S6CekDYIoBURC" alt=""><figcaption><p>it opened the new cmd for the machine</p></figcaption></figure>

This allows the attacker to see the contents of the `C$` administrative share on the `THEPUNSIER` machine.

**Using PsExec for Remote Command Execution**:

* if it The attacker can use **PsExec** to remotely execute commands on the compromised machine.
* If PsExec is available on the target system, the attacker can execute a command like:

  ```bash
  psexec.py //THEPUNSIER cmd.exe
  ```
* This will open a **remote shell** on the `THEPUNSIER` machine, allowing the attacker to execute commands as an administrator.
