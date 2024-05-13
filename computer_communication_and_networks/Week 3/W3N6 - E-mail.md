Email systems have 3 major components: **user agents**, **mail servers**, and the **simple mail transfer protocol (STMP)**. User agents allow users to read, reply to, forward, save, and compose emails. Examples of user agents include Gmail and Outlook. 
# Mail servers
Mail servers are the core of email infrastructure. Every user has a mailbox in their mail server, and when they wish to read their emails the emails are retrieved from that mailbox. When a user sends an email, it is added to the mail server's outgoing message queue, and is then sent to the recipient's mail server. If an email fails to be delivered, it is periodically retransmitted, and if delivery isn't successful within a certain timeout then the email is removed from the queue and the server notifies the sender with another email message.
# Simple mail transfer protocol (STMP)
STMP is the main [[W3N4 - Application layer protocols|application layer protocol]] for Internet email. It uses [[W3N3 - Transportation services#Transmission control protocol (TCP)|TCP]] to for reliable data transfer between mail servers. STMP consists of a client and server process, both of which are running on each mail server. When a mail server is sending mail, it uses the client, and when it is receiving, it uses the server.

Despite being common, the STMP protocol has some archaic restrictions, primarily that the body of the email must all be in 7 bit ASCII. This was fine when it was created, as bandwidth was too low for anyone to want to transfer images or other large files by email, but it requires modern emails to ASCII encode multimedia data before being send, and decode the ASCII at the other end. STMP does not normally use intermediate servers, with an email being sent directly from the sending to receiving server no matter the distance, and an email that fails to deliver is stored on the sender's server only. 

STMP uses [[W3N5 - The Web and HTTP#Non-persistent and persistent connections|persistent connections]], so if a mail server is sending many emails to the same mail server it does all of them in one TCP session.
## Differences between STMP and HTTP
STMP and [[W3N5 - The Web and HTTP#HyperText transfer protocol (HTTP)|HTTP]] are both used to transfer files from one host to another, but in different contexts. HTTP involves a client requesting files from a web server (a **pull protocol**), whereas STMP has the client sending a file to a mail server (a **push protocol**)
## STMP headers
A typical email header looks like this:
```
From: alice@aServer.com
To: bob@anotherServer.com
Subject: Hello
```
# Mail access protocols
When an email is sent from a user agent, it is first transferred by STMP to the sender's mail server, which then sends it using STMP to the recipient's mail server. The recipient must then use a pull protocol to get the email from their mail server. Common mail access protocols include **post office protocol (POP3)**, **internet mail access protocol (IMAP)**, or HTTP.
## Post office protocol (POP3)
POP3 consists of 3 phases, authorisation, transaction and update. The authorisation phase consists of a user logging in using `user <username>` and `pass <password>`. During the transaction phase, a user agent issues commands to download emails (using `retr`, mark (`dele`) and unmark emails for deletion, and get various usage statistics. User agents can operate in 2 modes: download-and-delete (where an email is deleted from the mail server as soon as it is downloaded), or download-and-keep (where the email remains on the mail server). The update phase occurs when a user ends the session, with any emails marked for deletion being deleted.
## Internet mail access protocol (IMAP)
Once an email is requested using POP3, the user agent can create mail folders and organise emails. This doesn't affect the mail server, but for a user which wishes to keep their emails similarly organised across many machines this introduces a lot of extra work. The IMAP protocol instead associates each message with a folder on the server, so it is consistent wherever a client accesses it from. When an email arrives, IMAP automatically puts it in the `INBOX` folder, and offers commands to allow a user to create new folders and move emails between folders.
## Web based email
Today, the standard email client will often be accessed through a web browser. This means that emails are transferred from the sender to their sever using HTTP, and from the receiving mail server to the recipient over HTTP. The email is still moved between the mail servers using STMP though.