**Boogeyman 2**
---

W boxie tym naszym zadaniem jest przeanalizowanie TTPs'a wykonanego na maszynie.
Całe zadanie jest kontynuacją wcześniejszego boxa Boogeyman 1,
pełni role uzupełnienia i pokazania jak możemy uprościć dane
procesy poprzez zastosowanie odpowiednich narzędzi

<br>

**TASK 1**
---
**Q:** I am now ready for round 2 with the Boogeyman! <br>
>**W:**

**A:** No answer needed

<br><br><br>

**TASK 2** 
---
**Q:** What email was used to send the phishing email? <br>
>**W:** Odczytujemy dane z pliku

**A:** westaylor23@outlook.com

---

**Q:** What is the email of the victim employee? <br>
>**W:** Odczytujemy dane z pliku

**A:** maxine.beck@quicklogisticsorg.onmicrosoft.com <br>

---

**Q:** What is the name of the attached malicious document? <br>
>**W:** Odczytujemy dane z pliku

**A:** Resume_WesleyTaylor.doc <br>

---

**Q:** What is the MD5 hash of the malicious attachment? <br>
>**W:** md5sum Resume_WesleyTaylor.doc
52c4384a0b9e248b95804352ebec6c5b  Resume_WesleyTaylor.doc

**A:** 52c4384a0b9e248b95804352ebec6c5b <br>

---

**Q:** What URL is used to download the stage 2 payload based on the document's macro? <br>
>**W:** Znajdujemy w virustotal po podaniu hasha w zakladce behavior w network communication

**A:** https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png <br>

---

**Q:** What is the name of the process that executed the newly downloaded stage 2 payload? <br>
>**W:** Wykonujemy polecenie olevba Resume_WesleyTaylor.doc
Na samym dole widzimy IOC wscript.exe

**A:** wscript.exe <br>

---

**Q:** What is the full file path of the malicious stage 2 payload? <br>
>**W:** Znajdujemy po środku logu shell_object.Exec ("wscript.exe C:\ProgramData\update.js")

**A:** C:\ProgramData\update.js <br>

---


**Q:** What is the PID of the process that executed the stage 2 payload? <br>
>**W:** W celu znalezienia PID procesu możemy zastosować komende volatility3:
vol -f WKSTN-2961.raw windows.psscan

**A:** 4260<br>
---

**Q:** What is the parent PID of the process that executed the stage 2 payload? <br>
>**W:** Znajdujemy po zastosowaniu tej samej komendy co wyzej

**A:** 1124 <br>
---

**Q:** What URL is used to download the malicious binary executed by the stage 2 payload? <br>
>**W:** Z poprzedniej komendy czyli olevba Resume_WesleyTaylor.doc 
Wiemy ze szukany plik to

**A:** https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe <br>

---

**Q:** What is the PID of the malicious process used to establish the C2 connection? <br>
>**W:** sprawdzamy czy proces utworzyl jakis proces potomny komenda:
vol -f WKSTN-2961.raw windows.pstree | grep "4260"
Otrzymujemy updater.exe na PID 6216

**A:** 6216 <br>

---

**Q:** What is the full file path of the malicious process used to establish the C2 connection? <br>
>**W:** W celu znalezienia pelnej scieżki mozemy zastosować znowu komende vol na procesie z pytania wyżej
vol -f WKSTN-2961.raw windows.cmdline --pid 6216

**A:** C:\Windows\Tasks\updater.exe <br>

---

**Q:** What is the IP address and port of the C2 connection initiated by the malicious binary? (Format: IP address:port) <br>
>**W:** Uruchamiamy netscan dla procesu powyżej
vol -f WKSTN-2961.raw windows.netscan |grep -i "6216"

**A:** 128.199.95.189:8080 <br><br><br>

---

**Q:** What is the full file path of the malicious email attachment based on the memory dump? <br>
>**W:** Uruchamiamy filescan dla pliku 
vol -f WKSTN-2961.raw windows.filescan |grep "Resume"

**A:** C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc <br>

---

**Q:** The attacker implanted a scheduled task right after establishing the c2 callback. What is the full command used by the attacker to maintain persistent access? <br>
>**W:** W celu odnalezienia informacji o tym, przeszukujemy rawa, szukajac plikow shedule czyli taskschd albo schtasks
strings  WKSTN-2961.raw |grep -i "schtasks"

**A:** schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"' <br>
