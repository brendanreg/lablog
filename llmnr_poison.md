Goal: Simulate an LLMNR poison attack to steal credentials from user attempting to access an SMB file share in AD. Analyze with wireshark, pfsense, and windows event logs.
Prerequisites: An active directory domain including a DC & w11 client machine, along with a kali machine residing on the same internal network. We'll also set up a legitimate 
               fileshare on the domain and attempt to authenticate to it with our stolen credentials.
Resources: https://www.cynet.com/attack-techniques-hands-on/llmnr-nbt-ns-poisoning-and-credential-access-using-responder/#heading-4 , https://app.hackthebox.com/sherlocks/Noxious

What is LLMNR poisoning? What are responder attacks?
 -When a windows device attempts to access a lan resource, it sends out a dns query for the hostname of the network resource. if dns fails to respond, 
  windows fallsback on link local multicast name resolution. llmnr essentially attempts to ask other devices on the lan if it knows where the resource is.
  this is where the attack begins; a malicious machine residing on the network uses a tool called a responder to provide a false location to the requesting machine.
  the requesting machine then attempts to authenticate to the false location by providing their domain credentials. and voila, we've stolen the username and 
  password hash of an unsuspecting user.

<img width="345" alt="network diagram" src="https://github.com/user-attachments/assets/cc477f55-d432-4727-af12-a605bf202d59" />
