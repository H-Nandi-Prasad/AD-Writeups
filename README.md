**CHECKPOINT-LAB ATTACK CHAIN** 

- **PHASE 1**
	- Here i already did the Basic Scannings of nmap .
		![Screenshot](images/Pasted%20image%2020260919191147.png)
- Here you might be wondering about why should we know that we should check the domain Accounts and their privileges and how should we know that we should approach like this itself.
- So basically while doing the enumeration we need to check all the possible resources we can, here there is no shortcut for the enumeration of the attack.	
- **bloodyAD**
	- We are using the bloodyAD which is the Active Directory enumeration tools , which is very handy.
	- Just try it out with all the options you will be available and get the good grip on the tool.
	- Here we should focus on the methodology not the tools.
	- Here is what you need to know while doing the enumeration.
	- **ACLs - (Access Control Lists)**
		- This is represents the permission for each associated user of the domain.
		- An *Access Control Entry* (**ACE**) is a single permission rule inside an Access Control List (ACL) that grants, denies, or audits a user or group's access to an Active Directory (AD) object.
		- `Mark Davies\0ADEL:<GUID>` means *Mark Davies used to be an AD object, but it was deleted*. When AD deletes an object, it moves it into the hidden `Deleted Objects` container and changes the name to include `DEL:<GUID>` so the deleted object's DN stays unique.
		- So here the most important thing while enumerating is that checking who has the right access to modify objects.
		![Screenshot](images/Pasted%20image%2020260911125757.png)
		- Here we can use the flag **--detail** to get in-detailed information and for enumeration we need to check what is it , where it was belonged to, before deleting and where the object is now by looking at these parameters:
			1. *msDS-LastKnownRDN* → What was its name?
			2. *lastKnownParent*   → Where did it live?
			3. *isDeleted*         → Is it deleted?
			4. *DN*                → Where is the object now?
		- Since the account alex.turner has the write privelege we need to check we can be able to recover the mark.davis account 
		- Here before enumerating further we actually need to understand that the privelege of the alex.turner here
			- The Account privilege says **permission:WRITE** , it doesn't mean that the account can do all the write operation over the domain. It means alex in our scenario can do some extent of the write operation on himself and on the OU of which he is currently in , OU=Employee.
		- Back to the recovery of the account, we can use the information of the deleted account information like GUID, CN, DC etc.
		![Screenshot](images/Pasted%20image%2020260911135050.png)
		- After the execution the Account is recovered.
		- Since we know only one password we can do pasword spraying to find out whether the password same as the current user(alex.turner).
		![Screenshot](images/Pasted%20image%2020260911135812.png)
		- We can see the permission as the write which was not there for the alex.turner.
		- Since the DevDrop has something interesting information related to the vs-code so we will be enumerating that share.

- **PHASE-2**

	- So here we will be authenticating with the smb to enumerate the shares and uploading the payload to the DevDrop share.
	- Here we are using the reverse shell to deliver the payload.
	- since the share DevDrop is based on the VS Code so we already have the automated script that is vis_shell.py
	- Uploading the reverse shell using the smbclient.
![Screenshot](images/Pasted%20image%2020260911185812.png)
	 
 listening on the other terminal so that it will connect from that side.
	 
![Screenshot](images/Pasted%20image%2020260911185725.png)

- Now get the user.txt in the Desktop 
- To Escalate Further we are using the rubeus.exe 

![Screenshot](images/Pasted%20image%2020260911200025.png)

![Screenshot](images/Pasted%20image%2020260911195939.png)

Now we are getting the delegated account user krbtgt from the kerberos using the rubeus.exe

![Screenshot](images/Pasted%20image%2020260911201218.png)

![Screenshot](images/Pasted%20image%2020260911201312.png)

- So here the *.kirbi* is the format used in the windows to present the ticket to the kerberos . Here we are conveting the ticket to the *.ccache* format which is used in the Linux Systems and also in the Impacket tools.

![Screenshot](images/Pasted%20image%2020260911202208.png)

- Now we are executing this command to know the privileges about the ryan.brooks

![Screenshot](images/Pasted%20image%2020260911223041.png)

- The Interpretation of the Output is that it shows us on which accounts the user ryan brooks has the privelege access to create/modify.
- From the Output we can see that svc_deploy is interesting one.
- So here we are exploiting one of the feature that is **BadSuccessor**.
	- ***BadSuccessor***
		- It abuses the Windows Server 2025 **delegated Managed Service Account (dMSA)** feature.
		- BadSuccessor can let you create a **new dMSA account** that the KDC treats as the successor of another account, such as `svc_deploy`
		- The clever part is that we are **not directly changing `svc_deploy`**. we are creating another AD object and manipulating the dMSA relationship around it.
		- So here after KDC acknowledges the relationship of the newly created account DMSA6 then the kerberos will create the unique tickets for that account seperately.
		- In the nutShell we can get the idea as that the newly created account inherits the privileges of the target account - svc_deploy.
		- Here the OU=**`DMSAHolder`** is the vulnerable Object.
- Usage of the BadSuccessor 
![Screenshot](images/Pasted%20image%2020260912064328.png)

- So here we got the .kirbi which is not compatible in the linux, we will be converting the kirbi to the ccache by decoding it and then passing it to the ticket-Converter.
- We can make use of the KRB5CCNAME name represents the system variable in which authentication takes place by default for kerberos.

![Screenshot](images/Pasted%20image%2020260912065021.png)

- Verify with the help of the *klist*

![Screenshot](images/Pasted%20image%2020260912065143.png)

- using the saved ccache credentials to authenticate to the smb server.

![Screenshot](images/Pasted%20image%2020260912065331.png)

- Further Enumerating the VMBackups share.

![Screenshot](images/Pasted%20image%2020260912065533.png)

- Since the vmem file contains all the necessary information like RAM,SAM,Security,etc so we did actually choose that one.

![Screenshot](images/Pasted%20image%2020260912075945.png)

![Screenshot](images/Pasted%20image%2020260912075858.png)

![Screenshot](images/Pasted%20image%2020260912080146.png)

- Changing the file Permissions:

![Screenshot](images/Pasted%20image%2020260912080240.png)

**Key Points:**
- Deeper Dive into the OU Permissions.
- Account Recovery with privilege
- Payload Uploads like Rubeus.exe 
- Deeper Dive into the Kerberos architecture
- Delegation of the account tgt
- Bad Successor - evilDMSA exploit
- memory Foreignsics - Volatality 3 
- File Permissions.
