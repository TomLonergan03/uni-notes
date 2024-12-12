An important part of [[W7N3 - Privilege separation|privilege separation]] is to authenticate users, to ensure that getting around the separation is not as simple as just logging in as root with no further verification.

The standard authentication mechanism is to require a username and corresponding password. The difference between a [[W4N2 - Public key encryption|cryptographic key]] and a password is that a password must be **hard to guess**, but still **easy to remember** as a user must enter it themselves.

# Attacks on passwords
- **Phishing:** fraudulent emails/websites/phonecalls/etc. which attempt to trick a user into entering their login details
	- The risk of this can be reduced by:
		- **Making it difficult for attackers to reach users** e.g. by implementing anti-spoofing controls on organisational email addresses or by filtering incoming emails
		- **Helping users identify and report phishing** e.g. training programs, warnings on emails originating outside the organisation, and ensuring it is easy to report phishing attacks
		- **Protecting the organisation from the effects of undetected phishing emails** e.g. using 2 factor authentication, using up to date browsers and proxy servers, and protecting devices from malware
		- **Responding quickly to incidents** e.g. encouraging users to report suspicious activity, and by having a planned and rehearsed response plan
- **Malware:** if e.g. a keylogger is installed then an attacker can record login details as they are entered
	- This can be mitigated using:
		- **2 factor authentication**
- **Online guessing attacks:** often, an attacker will know a target username, and can then try to brute-force all possible passwords until a password works
	- This can be limited by:
		- **Using a good password:** ensure it is long enough, with a good mix of symbols
		- **Rate limiting:** impose a limit on the number of failed attempts before the system is locked for a set period
		- **Captchas:** include a [captcha](https://en.wikipedia.org/wiki/CAPTCHA) to be solved along side a password entry, making it harder for an automated system to enter passwords
- **Offline guessing attacks:** if a server's password database is leaked, then an attacker can simply look up a username and find their password
	- This can be reduced by:
		- **Salting and hashing the passwords:** store passwords and a random salt value, as well as the hash of the salt ORed with their password. This means identical passwords do not have the same hash, preventing frequency analysis as well as using precomputed hash tables. This means every password must be independently brute-forced.

Additionally, using a password manager means that a user memorises one long, strong password and uses that to unlock the password manager, which is then able to generate unique strong passwords for each account the user holds.