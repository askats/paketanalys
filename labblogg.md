# Labblogg – Paketanalys

## 2026-09-29 – Steg 1: Planering och nytt repo
- Planerade projektet: egen trafik i labbet (ICMP, DNS, SSH, brandvägg) följt av SOC-analys av övningsfiler med skadlig trafik.
- Beslut: eget repo i stället för undermapp i hemmalabb, eftersom projektet visar en annan kompetens (analys) än hemmalabb (bygga och härda).
- Skapade repot på GitHub helt tomt (ingen README, .gitignore eller licens) så att historiken startar lokalt och första push inte krockar.
- Skapade .gitignore från start som blockerar .pcap, .pcapng, .zip och mappen ovningsfiler/.
- Varför: fångstfiler kan innehålla känsliga uppgifter och övningsfilerna innehåller skadligt material. Sådant ska aldrig hamna i ett publikt repo av misstag.
- Lärdom: ls -la visar dolda filer (punkt först) och filbehörigheter, ett grundkommando vid undersökningar eftersom angripare ofta gömmer filer så.
- Lärdom: säkerhet börjar i dokumentationen, och det är lättare att förhindra en läcka än att städa upp efteråt (en pushad fil finns kvar i Git-historiken).

- Felsökning: repot fick av misstag namnet Paketanalys och GitHub varnade för omdirigering vid push. Döpte om till paketanalys och uppdaterade adressen med git remote set-url. Lärdom: remote-adressen ska peka exakt rätt, inte via en omväg.