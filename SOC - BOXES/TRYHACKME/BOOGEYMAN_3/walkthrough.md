**Boogeyman 3**
---

W boxie tym naszym zadaniem jest przeanalizowanie TTPs'a wykonanego na maszynie.
Całe zadanie jest kontynuacją wcześniejszego boxa Boogeyman 2,
Najtrudniejszy z wszystkich boogeymanów z naciskiem na kibane i sysmona

<br>

**TASK 1**
---
**Q:** What is the PID of the process that executed the initial stage 1 payload? <br>
>**W:** Przechodzimy na podany adres serwera kibany, przechodzimy do zakładki discovery i odszukujemy date incydentu czyli August 29 2023 do August 30 2023
Odszukujemy pierwszy dokument wspomniany na początku czyli ProjectFinancialSummary_Q3
Otrzymujemy 4 wyniki. Na samym początku możemy dostrzec PID z którego został pobrany payload

**A:** 6392

---

**Q:** The stage 1 payload attempted to implant a file to another location. What is the full command-line value of this execution? <br>
>**W:** W tym samym miejscu w następnym wydarzeniu możemy dostrzec komende która posłużyła do pobrania payloada

**A:** "C:\Windows\System32\xcopy.exe" /s /i /e /h D:\review.dat C:\Users\EVAN~1.HUT\AppData\Local\Temp\review.dat <br>

---

**Q:** The implanted file was eventually used and executed by the stage 1 payload. What is the full command-line value of this execution? <br>
>**W:** W trzecim wpisie możemy zauważyć kolejną komende która posłużyła do wykonania payloada

**A:** "C:\Windows\System32\rundll32.exe" D:\review.dat,DllRegisterServer <br>

---

**Q:** The stage 1 payload established a persistence mechanism. What is the name of the scheduled task created by the malicious script? <br>
>**W:** W ostatnim czwartym wpisie możemy dostrzec komende która posłużyła do utworzenia shedule taska o następującej nazwie

**A:** Review <br>

---

**Q:** The execution of the implanted file inside the machine has initiated a potential C2 connection. What is the IP and port used by this connection? (format: IP:port) <br>
>**W:** Z poprzednich pytań wiemy, że atakujący utworzył skrpyt powershella, odszukujemy dane poprzez następujący filtr
powershell.exe and event.provider : "Microsoft-Windows-Sysmon" and event.code : "3"
Możemy przefiltrować po destination.ip oraz destination.port

**A:** 165.232.170.151:80 <br>

---

**Q:** The attacker has discovered that the current access is a local administrator. What is the name of the process used by the attacker to execute a UAC bypass? <br>
>**W:** Operując na danych pozyskanych z poprzednich pytań wiemy, że atakujący utworzył xcopy pliku na maszynie, odszukujemy wystąpienia tego pliku czyli review.dat, możemy filtrować po process.executable. Zauważamy wiele procesów takich jak powershell czy whoami wśród których znajdują się wpisy o interesującym nas procesie

**A:** fodhelper.exe <br>

---

**Q:** Having a high privilege machine access, the attacker attempted to dump the credentials inside the machine. What is the GitHub link used by the attacker to download a tool for credential dumping? <br>
>**W:** W celu odnalezienia połączeń na github, możemy przeszukać logi po fazie wskazującej na to czyli github.com. Otrzymujemy 56 wpisów, filtrujemy po process.command_line, znajdujemy interesujące nas informacje

**A:** https://github.com/gentilkiwi/mimikatz/releases/download/2.2.0-20220919/mimikatz_trunk.zip <br>

---


**Q:** After successfully dumping the credentials inside the machine, the attacker used the credentials to gain access to another machine. What is the username and hash of the new credential pair? (format: username:hash) <br>
>**W:** Wiemy, że atakujący wykonał skrypt mimikatz.exe, wyszukujemy po ten frazie i szukamy po filtrze command_line

**A:** itadmin:F84769D250EB95EB2D7D8B4A1C5613F2<br>
---

**Q:** Using the new credentials, the attacker attempted to enumerate accessible file shares. What is the name of the file accessed by the attacker from a remote share? <br>
>**W:** Z poprzednich część wiemy jaki hostname oraz jakim skryptem było wykonywane większość komend, dokładamy do tego polecenie cat często występujące podczas enumeracji plików w celu znalezienia interesującego nas pliku. host.name : "WKSTN-0051.quicklogistics.org" and powershell.exe and cat

**A:** IT_Automation.ps1 <br>
---

**Q:** After getting the contents of the remote file, the attacker used the new credentials to move laterally. What is the new set of credentials discovered by the attacker? (format: username:password) <br>
>**W:** W tej samej część odszukujemy informacje o nowych użytkownikach, odnajdujemy informacje o allan.smith

**A:** QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987 <br>

---

**Q:** What is the hostname of the attacker's target machine for its lateral movement attempt? <br>
>**W:** W tym samym miejscu dostrzegamy nazwę urządzenia

**A:** WKSTN-1327 <br>

---

**Q:** Using the malicious command executed by the attacker from the first machine to move laterally, what is the parent process name of the malicious command executed on the second compromised machine? <br>
>**W:** Odszukujemy informacje na temat maszyny host.hostname : "WKSTN-1327"
Otrzymujemy za dużo informacji, musimy przefiltrować bardziej, możemy dokonać filtowania po nowoutworzonych procesach w sysmonie czyli code 1
host.hostname : "WKSTN-1327" and event.provider : "Microsoft-Windows-Sysmon" and event.code : "1"
Otrzymujemy 591 wyników. Przeszukując wyniki odnajdujemy interesujący nas plik

**A:** wsmprovhost.exe <br>

---

**Q:** The attacker then dumped the hashes in this second machine. What is the username and hash of the newly dumped credentials? (format: username:hash) <br>
>**W:** Informacje o tym odnajdujemy w tym samym miejscu w wpisie zawierajacym mimikatz.exe

**A:** administrator:00f80f2538dcb54e7adc715c0e7091ec <br>

---

**Q:** After gaining access to the domain controller, the attacker attempted to dump the hashes via a DCSync attack. Aside from the administrator account, what account did the attacker dump?<br>
>**W:** Z poprzednich wpisów wiemy, że atakujący użył innej maszymy w celu wyciągnięcia danych, odczytujemy nazwę maszyny, DC01, następnie modyfikujemy zapytanie
host.hostname : "DC01"  and event.provider : "Microsoft-Windows-Sysmon" and event.code : "1"
Otrzymujac tym samym 54 wyniki, sprawdzamy tylko wyniki z poleceniem lsadump, dostrzegamy innego uzytkownika

**A:** backupda <br>

---

**Q:** After dumping the hashes, the attacker attempted to download another remote file to execute ransomware. What is the link used by the attacker to download the ransomware binary? <br>
>**W:** Idąc dalej po incydencie zachowując te same filtry, możemy zauważyć kolejny plik który został pobrany

**A:** http://ff.sillytechninja.io/ransomboogey.exe <br>