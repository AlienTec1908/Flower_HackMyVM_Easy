# Flower - HackMyVM (Easy) - Writeup

![Flower Icon](Flower.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Flower" (Schwierigkeitsgrad: Easy).

## Metadaten

*   **Maschine:** Flower (HackMyVM - Easy)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Flower](https://hackmyvm.eu/machines/machine.php?vm=Flower)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 2. November 2022
*   **Original Writeup:** [https://alientec1908.github.io/Flower_HackMyVM_Easy/](https://alientec1908.github.io/Flower_HackMyVM_Easy/)

## Zusammenfassung des Angriffspfads

Die initiale Erkundung mit `nmap` offenbarte lediglich einen offenen Port 80, auf dem ein Apache-Webserver lief. Die Web-Enumeration mit `gobuster` fand unter anderem die Datei `index.php` und ein Skript `run.sh`, das auf PHP-Nutzung hindeutete.

Eine Schwachstellenanalyse der `index.php` (impliziert durch Logs, vermutlich mit Burp Suite) ergab eine PHP Code Injection-Schwachstelle. POST-Requests an `index.php` mit einem Parameter namens `petals`, der Base64-kodierten PHP-Code enthielt, wurden vom Server dekodiert und ausgeführt.

Durch Senden einer Base64-kodierten PHP-Reverse-Shell-Payload (`nc -e /bin/bash ...`) im `petals`-Parameter wurde eine Reverse Shell als Benutzer `www-data` mittels `nc` aufgebaut und anschließend stabilisiert (`python pty`).

Die Enumeration als `www-data` führte zum Home-Verzeichnis des Benutzers `rose` und einem Python-Skript `diary.py` im Unterverzeichnis `diary`. Die Prüfung der `sudo`-Rechte (`sudo -l`) zeigte, dass `www-data` dieses Skript (`/home/rose/diary/diary.py`) als Benutzer `rose` ohne Passwort ausführen durfte. Da `www-data` (oder `rose`) Schreibrechte auf das Skript oder dessen Verzeichnis hatte, wurde der Inhalt von `diary.py` mit einem Payload zum Starten einer Bash-Shell (`import os; os.system("/bin/bash")`) überschrieben. Die Ausführung des modifizierten Skripts via `sudo` gewährte eine Shell als Benutzer `rose`.

**Hinweis:** Der Schritt zur Eskalation von `rose` zu `root` ist im bereitgestellten Original-Writeup nicht dokumentiert. Es muss eine weitere Schwachstelle oder Fehlkonfiguration vorhanden gewesen sein.

Schließlich wurden die User-Flag als `rose` und die Root-Flag (nach dem undokumentierten Schritt zu `root`) gelesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `gobuster`
*   `wget`
*   `cat`
*   `wfuzz` (erfolglos)
*   `echo`
*   `base64`
*   Burp Suite / `curl`
*   `nc` (netcat)
*   `python3` (`pty`, `os` modules)
*   `stty`
*   `cd`
*   `ls`
*   `file`
*   `sudo`
*   `vi` (implied)
*   `id`
*   `bash`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Portscan (`nmap`) -> Nur Port 80 (HTTP, Apache).
2.  **Web Enumeration:** `nikto`, `gobuster` -> findet `/index.php`, `/run.sh`, `/flower.jpg`.
3.  **Vulnerability Assessment:** `wfuzz` (kein Erfolg mit GET). Manuelle Prüfung/Burp (impliziert) -> findet PHP Code Injection in `index.php` über POST-Parameter `petals` (akzeptiert Base64-Payload).
4.  **Vorbereitung:** PHP-Reverse-Shell (`nc -e ...`) Base64-kodieren (`echo ... | base64`).
5.  **Initial Access:** `nc`-Listener starten. Base64-Payload via POST-Request an `index.php` mit Parameter `petals` senden (`curl` / Burp) -> Shell als `www-data`.
6.  **Shell Stabilization:** `python3 -c 'import pty; ...'`, `export TERM=xterm`, `stty raw -echo; fg`.
7.  **Privilege Escalation (`www-data` -> `rose`):** Enumeration -> `/home/rose/diary/diary.py` gefunden. `sudo -l` -> `(rose) NOPASSWD: /usr/bin/python3 /home/rose/diary/diary.py`. Skript `/home/rose/diary/diary.py` mit `import os;os.system("/bin/bash")` überschreiben. `sudo -u rose /usr/bin/python3 /home/rose/diary/diary.py` ausführen -> Shell als `rose`.
8.  **User Flag:** `cat /home/rose/user.txt`
9.  **Privilege Escalation (`rose` -> `root`):** *Dieser Schritt fehlt in der Quelldokumentation.* Es muss ein weiterer Exploit angewendet worden sein.
10. **Root Flag:** `cat /root/root.txt` (als Root)

## Flags

*   **User Flag (`/home/rose/user.txt`):** `HMV{R0ses_are_R3d$}`
*   **Root Flag (`/root/root.txt`):** `HMV{R0ses_are_als0_black.}`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
