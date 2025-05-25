Goal: Simulate an LLMNR poison attack to steal credentials from user attempting to access an SMB file share in AD. Analyze with wireshark packet capture.
Prerequisites: An active directory domain including a DC & w11 client machine, along with a kali machine residing on the same internal network.
Resources: https://www.cynet.com/attack-techniques-hands-on/llmnr-nbt-ns-poisoning-and-credential-access-using-responder/#heading-4 , https://app.hackthebox.com/sherlocks/Noxious, https://www.kali.org/tools/responder/

What is LLMNR poisoning? What are responder attacks?
 -When a windows device attempts to access a lan resource, it sends out a dns query for the hostname of the network resource. if dns fails to respond, 
  windows fallsback on link local multicast name resolution. llmnr essentially attempts to ask other devices on the lan if it knows where the resource is.
  this is where the attack begins; a malicious machine residing on the network uses a tool called a responder to provide a false location to the requesting machine.
  the requesting machine then attempts to authenticate to the false location by providing their domain credentials. and voila, we've stolen the username and 
  password hash of an unsuspecting user.

For a more ridgid definition, let's consult the MITRE ATT&CK framework. We can find LLMNR Poisoning and SMB relay under Adversary-In-The-Middle. ID T1557.001:

<img width="661" alt="image" src="https://github.com/user-attachments/assets/c4957ea8-d5c3-4196-bc83-3c0bad8d90e6" />

Let's set the stage:

<img width="345" alt="network diagram" src="https://github.com/user-attachments/assets/cc477f55-d432-4727-af12-a605bf202d59" />

We've got an ad domain comprised of a dc and a client machine, which we will target for this attack. We have kali ready with responder to carry out the attack. finally, we've converted our private switch to an internal one in hyper-v, which allows our host os to recieve multicast traffic from the simulation environment.
So, let's kick off the capture in wireshark and start up responder on kali:

-
<img width="427" alt="image" src="https://github.com/user-attachments/assets/6bfdd0a2-f1a1-4667-85b4-69084cbfa569" />

-
<img width="479" alt="image" src="https://github.com/user-attachments/assets/aa4911ca-2f07-4692-8659-39005d820531" />

-
<img width="333" alt="image" src="https://github.com/user-attachments/assets/5b94299c-6782-4674-b387-9aceff15f45f" />

Now, let's browse to the legitimate fileshare just to see what kind of traffic we get, and we can compare it to the llmnr traffic we'll prompt later:


And now, let's simulate an accidental typo on the part of the victim when attempting to browse to that same share:
<img width="454" alt="image" src="https://github.com/user-attachments/assets/442b37ef-8a77-45a2-9ef1-2f773b2e87fa" />

and let's see if we generated any llmnr traffic. if we did, our attack was probably successful!

<img width="678" alt="image" src="https://github.com/user-attachments/assets/5bacdde9-313d-45dd-bb99-997bca14d6e1" />

nice, our victim sent out some llmnr requests, let's see if our attacker at 10.0.1.11 grabbed those credentials...

<img width="326" alt="image" src="https://github.com/user-attachments/assets/a16a4714-a9dc-4688-8f9a-c4694bb00516" />

that was easy! 
that long string we recieved is the ntlmv2 hash. it's an amalgamation of the information exchanged between the server and client when authenticating credentials. ntlmv2 uses this information and the user's password to create a hash, which is the very long string we see at the end. in a legitimate ntlmssp negotiation, both the server and client calculate this hash, and if they match, the user is authenticated.
let's see if we can crack this hash and reveal wvictim's password. all we need to do is feed the string we got from responder into hashcat. let's create a text file so hashcat can read it:

<img width="325" alt="image" src="https://github.com/user-attachments/assets/d20f0803-407e-4e3e-9987-d093468e1f2a" />

and feed it to hashcat in a format it understands:

<img width="475" alt="image" src="https://github.com/user-attachments/assets/617b5137-e4b5-46dc-9154-204ff1f98da0" />

ntlmv2 is the file we created with the hash harvested from our responder attack. rockyou.txt is a wordlist comprised of common passwords. it will attempt to apply these passwords to the prerequisite info (username, domain, challenge phrase, etc.) and recreate the hash that our victim sent. when it does, it knows the applied password must be our user, wvictim's password.

<img width="475" alt="image" src="https://github.com/user-attachments/assets/dfd0ea51-2584-43a7-8ab0-418cd27a25f2" />

And in less than 10 seconds we've found a match.




