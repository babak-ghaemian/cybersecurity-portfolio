# wireshark Network Analysis Lab

Lab blog

Sniffing and Analyzing Network Traffic with Wireshark
Today’s Server+ lab was all about packet sniffing — capturing live network traffic with Wireshark and picking apart plaintext protocols to see exactly what an attacker (or a curious sysadmin) could see. The lab had three main phases:
	•	Configure a Kali Linux interface to passively capture traffic
	•	Generate real traffic using FTP, Telnet, and email (SMTP/POP3)
	•	Analyze the capture in Wireshark to pull out credentials and messages sent in the clear
Here’s how it went.
Setting Up the Sniffer (Kali Linux)
I logged into the Kali 2 box and opened a terminal to check my interface:

ifconfig

Then, to put the interface into a clean listening state without an assigned IP, I ran:

ifconfig eth0 0.0.0.0 up

A quick ifconfig confirmed eth0 had no IPv4 address — exactly what you want for a passive sniffer, since it shouldn’t be generating its own traffic on the network.
From there I launched Wireshark, clicked through the Lua error and root warning prompts, went to Capture > Interfaces, selected eth0, and hit Start. Packets started rolling in almost immediately once other machines on the network became active.
Generating Traffic (Windows 10)
With the capture running, I switched to the Windows 10 machine to generate some traffic for Wireshark to catch.
FTP: I connected to the Windows Server —

ftp 192.168.1.10

— logged in as user ftp with password zombie, listed the directory with ls, switched to binary mode with bin, and pulled down a JPEG with get 2008.jpg. FTP sends credentials in plaintext, so this password would be fully visible in the capture later.
Telnet: Next I telnetted into the same server:

telnet 192.168.1.10

I hit a snag here — my first login attempt as administrator kept getting rejected, even though I was sure I had the right password. Turns out I’d left Caps Lock on from switching windows earlier, and P@ssw0rd isn’t the same as p@ssw0rd (or worse, P@SSW0RD). Once I turned Caps Lock off and retyped it carefully, the login went through. A good reminder that plaintext protocols like Telnet are unforgiving about small mistakes — and even more unforgiving about security, since none of this was encrypted either way.
Once in, I created a new user and added them to a privileged group:

net user creeper P@ssw0rd /add
net group "domain admins" creeper /add

I also checked the existing superman account with net user superman to grab its details for one of the lab’s challenge flags.
Email: Finally, I used Opera Mail to compose and send a message to student@campus.edu with the subject “minecraft,” then checked mail to confirm it came through. POP3 and SMTP are both plaintext protocols, so this message — and the login used to retrieve it — would also show up unencrypted in the capture.
Analyzing the Capture (Wireshark)
Back on Kali, I stopped the capture and started digging through it with display filters:

frame contains zombie

This pulled up the FTP session and confirmed the password zombie was sitting right there in plaintext.

pop

Filtering on POP traffic exposed the student account’s password, P@ssw0rd, in the same unencrypted way.

frame contains buy

This located the email frame containing “buy” from the minecraft message. Right-clicking the POP frame and choosing Follow TCP Stream let me read the entire email body in plain text.

telnet

Filtering on Telnet and following that TCP stream showed the entire session — including the exact commands used to create the creeper account and add it to Domain Admins, all fully readable.

What I Learned
	•	How to configure a Linux interface for passive sniffing without assigning it an IP
	•	How FTP, Telnet, and POP3/SMTP all transmit credentials and data in plaintext
	•	How to use Wireshark display filters to isolate specific protocols or keywords
	•	How Follow TCP Stream reconstructs an entire conversation for easy reading
	•	Why case sensitivity matters even in “simple” login troubleshooting

Final Thoughts
This lab was a clear, hands-on demonstration of why plaintext protocols are considered legacy and insecure by modern standards — watching an admin password appear in cleartext in Wireshark makes the risk real in a way that reading about it never quite does. It’s also a solid reminder that network sniffing isn’t just an attacker’s tool; it’s just as valuable for admins auditing their own traffic for weak spots.​​​​​​​​​​​​​​​​
