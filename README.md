<!-- HEADER BADGES -->
![Cybersecurity](https://img.shields.io/badge/CYBERSECURITY-NETWORKWALKS-0066CC?style=for-the-badge)
![Week & Batch](https://img.shields.io/badge/WEEK%204-B082-702963?style=for-the-badge)
![Authorized](https://img.shields.io/badge/AUTHORIZED-YES-00C853?style=for-the-badge)
![Skill](https://img.shields.io/badge/Skill-Cybersecurity-gray?style=for-the-badge&logoColor=white&labelColor=red)
![Skill](https://img.shields.io/badge/Skill-Linux-gray?style=for-the-badge&logoColor=white&labelColor=red)
![Ethical Hacking](https://img.shields.io/badge/Ethical%20Hacking-orange?style=for-the-badge&logo=kalilinux&logoColor=white)
![Penetration Testing](https://img.shields.io/badge/Penetration%20Testing-red?style=for-the-badge&logo=kalilinux&logoColor=white)
![Author](https://img.shields.io/badge/AUTHOR-Amogbon%20Abimbola%20Esther-6A1B9A?style=for-the-badge)



Patient Portal Penetration Test — Mediroza General Hospital

Modules completed	W2-PM1 (Multiple Kali Tools), W2-PM5 (Zenmap Scanning), Week 4 Capstone — Black-Box Web Application Penetration Test

Client / Target	Mediroza General Hospital — https://medirozahospital.com (training environment)

Permission secured from client?	Yes

Phases covered	Phase 1: Reconnaissance & Footprinting · Phase 2: Attack Surface Mapping · Phase 3: Vulnerability Analysis · Phase 4: Exploitation · Phase 5: Post-Exploitation Analysis · Phase 6:

 Reporting
⚠️ Note on scope and disclosure: This engagement was performed against a purpose-built training target ("Mediroza General Hospital") as part of a cybersecurity certification program, under explicit written authorization. All patient names, records, and lab values shown below are synthetic data generated for the training lab — no real individuals, real medical data, or real production systems are represented.

1. Liability Disclaimer
I have performed these activities only against a purpose-built training environment (Mediroza General Hospital) under explicit written authorisation from Networkwalks, as part of my Cybersecurity & Ethical Hacking capstone project. All identifying details, patient records, and credentials referenced in this report are synthetic training data — no real individuals, real medical data, or real production systems are represented or exposed. These materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries, unauthorised access is a crime even when nothing is damaged.


2. Introduction
This report covers a three-day black-box penetration test against Mediroza General Hospital's public-facing patient portal at https://medirozahospital.com, performed as my Week 4 capstone project following on from earlier footprinting (W2-PM1) and network scanning (W2-PM5) modules completed during my ongoing internship program at Networkwalks. The objective was to simulate the perspective of an external, unauthenticated attacker: identify exploitable vulnerabilities, demonstrate real-world impact through controlled exploitation, and deliver a professional report with prioritised remediation guidance — the same deliverables expected in a real client engagement.



The engagement was limited strictly to the domain medirozahospital.com and its subpaths. No social engineering, denial-of-service testing, or testing of out-of-scope infrastructure was performed, in accordance with the rules of engagement provided by Networkwalks and Mediroza General Hospital.

Overall Risk Rating: CRITICAL — immediate remediation is strongly recommended before this application is exposed to real patient data.

3. Tools Used
whois	Domain registration lookup for the target hospital's web infrastructure
whatweb	Fingerprint web technologies (server, CMS, plugins, IP)
nslookup	Resolve the domain name to its IP address using DNS
curl -I / curl -s	Inspect HTTP response headers; retrieve files via direct HTTP requests
wafw00f	Detect whether a Web Application Firewall protects the site
dig	DNS record enumeration (substituted for dnsrecon due to a Python 3.12 compatibility issue)
theHarvester	OSINT / email and subdomain gathering
Manual SQL injection testing	Probe login forms for unsanitised input handling and authentication bypass
John the Ripper / Networkwalks Hash Calculator & Password Cracker	Extract and dictionary-attack PDF encryption hashes to recover document passwords
ExifTool / pdfinfo	Document metadata analysis on retrieved patient report PDFs


4. Activities Performed
   
4.1 Reconnaissance & Attack Surface Mapping
From passive and active reconnaissance against medirozahospital.com using whois, whatweb, nslookup, curl, wafw00f and dig to profile the domain, resolve its IP, fingerprint the technology stack, and check for a Web Application Firewall (none was detected). I then reviewed robots.txt and sitemap.xml, which explicitly listed /patient/, /staff/, and /old/ as disallowed paths — an unintentional roadmap to the site's most sensitive directories, since Disallow directives are not an access control.



Browsing directly to each disallowed path confirmed that directory listing (autoindexing) was enabled on all three, exposing internal application files (login.php, portal.php, download.php) and, critically, a legacy database backup sitting in /old/.

4.2 Vulnerability Analysis & Exploitation — SQL Injection
Manual testing of the patient and staff login forms revealed that user input was concatenated directly into SQL queries with no sanitisation. A single quote in the username field returned a raw MySQL syntax error, confirming the injection point. Submitting the classic comment-based payload admin'-- - then authenticated successfully with no valid credentials required — and the same payload also bypassed authentication on the staff portal, indicating a shared vulnerable code pattern across both login forms.

4.3 Insecure Direct Object Reference & Report Retrieval
Once authenticated via the SQL injection bypass, the report download endpoint (/patient/download.php?id=) accepted a simple sequential integer with no server-side check that the session was authorised to access that specific patient's report. Iterating id=1, id=2 and id=3 from a single bypassed session retrieved three different named patients' encrypted pathology reports.

Patient portal showing sequential, guessable download IDs for every patient's lab report Figure 1 — The patient portal's "My lab reports" view. Each report is fetched via download.php?id=1/2/3 with no per-patient authorisation check (visible in the browser status bar).

4.4 Password Cracking on Retrieved Reports
Each retrieved PDF was encrypted (Standard V2.3, 128-bit), so I extracted the crackable hash from each file and ran a dictionary attack using John the Ripper and the Networkwalks Hash Calculator / Password Cracker. All three passwords were recovered within seconds.

patient_report_1.pdf downloaded via the IDOR, still password-protected pending cracking Figure 2 — patient_report_1.pdf downloaded via the IDOR, still password-protected pending cracking.

Dictionary attack recovering the password 123456 for patient_report_1.pdf Figure 3 — Dictionary attack against the extracted $pdf$ hash recovers the password 123456 for patient_report_1.pdf.

patient_report_2.pdf still password-locked prior to the cracking attempt Figure 4 — patient_report_2.pdf still password-locked prior to the cracking attempt.

A second dictionary attack recovering the password "password" for patient_report_2.pdf Figure 5 — A second dictionary attack recovers the password password for patient_report_2.pdf.

patient_report_3.pdf awaiting its password before the same workflow was repeated Figure 6 — patient_report_3.pdf awaiting its password before the same workflow was repeated.

A third dictionary attack recovering the password for patient_report_3.pdf Figure 7 — A third dictionary attack recovers the password !@#$%^& for patient_report_3.pdf.

With all three passwords recovered, the decrypted reports opened to reveal full patient lab-result tables:

Decrypted contents of patient_report_2.pdf Figure 8 — Decrypted contents of patient_report_2.pdf, confirming the encryption provided no meaningful protection once the weak password was cracked.

Decrypted contents of patient_report_1.pdf Figure 9 — Decrypted contents of patient_report_1.pdf, a second patient's lab-result report.

4.5 Metadata Analysis & Legacy Backup Exposure
Running ExifTool against the decrypted PDFs surfaced an internal developer comment and staff username embedded in one report's metadata, confirming that the database backup exposure identified in /old/ was a known, undocumented risk rather than a pure accident. Separately, downloading the backup file itself (a single unauthenticated curl request) revealed a full staff table and a shareholders table — satisfying, on its own, the engagement's data-exposure objective without ever touching the login form.

5. Risk Analysis / Impact
The risks below reflect observations from the reconnaissance, exploitation and post-exploitation activities described above. Risk level key: Critical / High / Medium / Low / Informational, applied consistent with common industry practice (e.g. OWASP Risk Rating Methodology).



Risk / Finding	Evidence / Observation	Potential Impact	Level

1	SQL injection — full authentication bypass	admin'-- - authenticated with no valid credentials on both /patient/login.php and /staff/login.php	Unauthenticated attacker gains full portal access, unlocking every downstream finding in this report	Critical

2	Exposed legacy database backup	Unencrypted mediroza_db_backup_2019.sql retrieved from /old/ via a single unauthenticated GET request	Full exposure of staff PII (national ID, salary) and shareholder equity data — a severe data-protection breach	Critical

3	IDOR on download.php	Sequential id=1/2/3 parameter returned three different patients' reports from one bypassed session	Combined with #1, allows enumeration of every patient record on the system, not just one	High

4	Directory listing enabled	/patient/, /staff/, /old/ each returned a full auto-generated file index	Reveals internal application structure and is the root enabling cause of Finding #2	High

5	Weak / default PDF passwords	All three patient report passwords recovered in seconds via dictionary attack (123456, password, !@#$%^&)	Provides no meaningful protection for confidential medical data at rest, independent of access-control fixes	Medium

6	Sensitive paths disclosed via robots.txt	robots.txt listed /patient/, /staff/, /old/ in plain text	Gave a direct roadmap to the site's most sensitive directories, accelerating this assessment	Low

7	Metadata disclosure in report PDFs	ExifTool extracted a developer's username and an internal comment from one report's metadata	Confirms internal knowledge of the backup exposure and leaks operational/staff information	Low

8	User-Agent based bot filtering	Default tool UA blocked (403); spoofed browser UA succeeded (200)	Negligible security value — trivially bypassed and not a meaningful barrier to a real attacker	Informational

6. Recommendations
Immediate (Critical / High):

Rewrite all database queries in login.php (patient and staff) and download.php to use parameterised queries / prepared statements — never concatenate user input into SQL strings.
Remove the /old/ directory and its database backup from the public web root immediately; store backups outside the webroot or in access-controlled, encrypted storage.

Disable directory listing (autoindexing) server-wide, not only on sensitive paths.
Add server-side authorisation checks on download.php tied to session identity, and replace sequential integer IDs with non-guessable, per-report tokens (UUIDs).
Rotate any credentials or secrets that may have shared a database with the exposed backup.
Short-term (Medium):

Generate strong, unique, randomly-generated passwords per patient report, delivered via a secure channel rather than reused or predictable values.
Implement account lockout / rate limiting on login endpoints.
Deploy a Web Application Firewall with SQL-injection detection rules as a defence-in-depth measure (none was detected during this assessment).
Longer-term / best practice:

Strip or sanitise document metadata (author, comments) from all generated PDFs before distribution.
Avoid listing sensitive paths in robots.txt — rely on authentication and server-side access control instead.
Conduct a full source-code review of the custom CMS, since the same vulnerability class (unsanitised SQL input) appeared in two independent modules.
Establish a periodic external penetration-testing programme and a secure software development lifecycle (SSDLC) to catch these issues before deployment.

7. Conclusion
This capstone engagement built directly on the footprinting and scanning skills developed in Weeks 1–2 of the program, extending them into full vulnerability discovery and controlled exploitation. I successfully chained a SQL injection authentication bypass into an IDOR to retrieve every patient report on the system, cracked the weak passwords protecting those reports, and independently confirmed a critical, unauthenticated data exposure — a legacy database backup containing staff and shareholder records — with PDF metadata corroborating the root cause.

The exercise reinforced that information gathering and small misconfigurations compound: none of the individual weaknesses required advanced skill, but together they produced a complete, unauthenticated path to confidential medical data. I also practised documenting findings the way a real client engagement demands — proof of exploitation, business impact, severity rating, and prioritised remediation — rather than just a list of technical observations.

Finally, this project reinforced that reconnaissance, exploitation and reporting must always be performed within an authorised scope; every activity described here was carried out against a purpose-built training environment under written permission as part of the assigned Networkwalks capstone project.

8. Evidences Collected
Screenshots below are presented in the order the corresponding activity occurred during testing

![Screenshot 1](Screenshot%201.jpeg)

![Screenshot 2](Screenshot%202.jpeg)

![Screenshot 3](Screenshot%203.jpeg)

![Screenshot 4](Screenshot%204.jpeg)

![Screenshot 5](Screenshot%205.jpeg)

![Screenshot 6](Screenshot%206.jpeg)

![Screenshot 8](Screenshot%208.jpeg)

![Screenshot 10](Screenshot%2010.jpeg)

![Pentesting Report](Pentesting%20report.jpeg)



