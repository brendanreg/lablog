my writeup for the noxious sherlock from hack the box -

Sherlock Scenario:
**The IDS device alerted us to a possible rogue device in the internal Active Directory network. The Intrusion Detection System also indicated signs of LLMNR traffic, which is unusual. It is suspected that an LLMNR poisoning attack occurred. The LLMNR traffic was directed towards Forela-WKstn002, which has the IP address 172.17.79.136. A limited packet capture from the surrounding time is provided to you, our Network Forensics expert. Since this occurred in the Active Directory VLAN, it is suggested that we perform network threat hunting with the Active Directory attack vector in mind, specifically focusing on LLMNR poisoning.**

q1:
**Its suspected by the security team that there was a rogue device in Forela's internal network running responder tool to perform an LLMNR Poisoning attack. Please find the malicious IP Address of the machine.**

they've provided us with a single artifact for this one; a pcap file. we'll be using wireshark to analyze the traffic.
to begin, we know the ip address of the victim machine: 172.17.79.136. to identify the attacker's ip, we can search the traffic for the llmnr protocol, which use port 5355.
we can also exclude ipv6 traffic as we know the responses were directed towards an ipv4 address.
after applying the filter "llmnr && not ipv6", we're left with a manageable list of packets. 

<img width="509" alt="image" src="https://github.com/user-attachments/assets/4edc4d56-4f60-42a3-901f-8704844580ff" />

let's check the endpoints tab in statistics to familiarize ourselves with all of the players here:

<img width="581" alt="image" src="https://github.com/user-attachments/assets/bf2d99ab-58d3-47a5-afb4-4a6b288a9098" />


we can see 4 addresses involved in our filter. one is a multicast address and another is the legitimate machine mentioned in the scenario, so let's ignore those. that leaves us with two potential attackers: 172.17.79.129 & 172.17.79.135.
let's take a closer look by filtering our traffic further to only include these two potential targets by applying the filter "llmnr && not ipv6 && (ip.src==172.17.79.129 || ip.src==172.17.79.135)":

<img width="833" alt="image" src="https://github.com/user-attachments/assets/490815f6-c30c-4660-8a02-00fc27975920" />

here we can see a plethora of llmnr responses eminating from 135 to 136 (our victim), and 129 (our other suspect). this indicates to me we've found our culprit.
Answer: 172.17.79.135

q2:
**What is the hostname of the rogue machine?**

this one should be simple. the device is likely getting a dhcp address, and it must send a request to do so. dhcp requests will include device hostnames. let's try the filter "ip.src==172.17.79.135 && dhcp" to see if that's the case:

<img width="481" alt="image" src="https://github.com/user-attachments/assets/61b19436-a6db-4b26-9c79-dcd608d4a730" />

Answer: kali

q3:
**Now we need to confirm whether the attacker captured the user's hash and it is crackable!! What is the username whose hash was captured?**

we can search for ntlmssb to see if authentication was attempted, and check for the acct name in the smb header:

<img width="761" alt="image" src="https://github.com/user-attachments/assets/6e03e244-a78e-4794-8f60-91134b434b62" />

Answer: john.deacon

q4:
**In NTLM traffic we can see that the victim credentials were relayed multiple times to the attacker's machine. When were the hashes captured the First time?**

let's check the time on the earliest packet caught by the above filter

Answer: UTC Arrival Time: Jun 24, 2024 11:18:30.919816000 UTC

q5:
**What was the typo made by the victim when navigating to the file share that caused his credentials to be leaked?**

let's go back to our llmnr filter with ip.src set to the victim's ip and see if we can inspect his llmnr request packet:

<img width="470" alt="image" src="https://github.com/user-attachments/assets/010ad281-7cad-4f84-9132-1e05cc162dc0" />

Answer: DCC01

q6:
**To get the actual credentials of the victim user we need to stitch together multiple values from the ntlm negotiation packets. What is the NTLM server challenge value?**

in the initial session setup response packet from the adversary, we can find the challenge value:

<img width="476" alt="image" src="https://github.com/user-attachments/assets/694747a4-dffb-40d6-b0e6-2debc04b2cbc" />

Answer: NTLM Server Challenge: 601019d191f054f1

q7:
**Now doing something similar find the NTProofStr value.**

assuming this has to do with the response to the challenge, let's look at the next packet originating from the victim:

<img width="479" alt="image" src="https://github.com/user-attachments/assets/af6345e2-d9c8-424e-9630-f86d26df7373" />

Answer: NTProofStr: c0cc803a6d9fb5a9082253a04dbd4cd4

q8:
**To test the password complexity, try recovering the password from the information found from packet capture. This is a crucial step as this way we can find whether the attacker was able to crack this and how quickly.**

at this point i'm a fish out of water, so it's time to do some learning.
we already know that ntlmssp uses a challenge-response authentication process. after some research, i've learned the following about ntlmv2:
simply put: client requests authentication by sending username and domain -> server responds by issuing an arbitrary challenge phrase -> client sends back a response based on the username, domain, challenge phrase, and password hash -> server also computes the hash and compares to the response.
hashcat uses bruteforce with predefined lists of password hashes. it basically goes down the list and attempts to compute the same response that the client generated by applying known password hashes with the information it already knows (username, domain, challenge phrase). when we get a match, we know we've cracked the password.

hashcat knows how to crack ntlmv2, so we just need to supply it with the information it requires to do so. after extracting the necessary information from the packets and normalizing it in a way hashcat can read, we're left with the following: 
john.deacon::FORELA:601019d191f054f1:c0cc803a6d9fb5a9082253a04dbd4cd4:010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000

let's feed it to hashcat and see if we get a crack:

Answer: NotMyPassword0K?

q9:
**Just to get more context surrounding the incident, what is the actual file share that the victim was trying to navigate to?**

let's return to wireshark and filter for smb2 traffic. if anyone connected to a share, we'd see it:

<img width="815" alt="image" src="https://github.com/user-attachments/assets/a0b1d6ef-a274-4aeb-b8da-7b7981520e48" />

there's a couple of packets showing the victim machine connecting to different shares, but one of them occurs less than a minute after the typo, so let's go with that one.

Answer: \\DC01\DC-Confidential





