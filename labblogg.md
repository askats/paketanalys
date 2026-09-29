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

## 2026-09-29 – Steg 2: Wireshark på Kali utan root
- Kontrollerade att Wireshark (4.6.6) och dumpcap är installerade på Kali.
- labbkali är redan medlem i gruppen wireshark, så jag kan fånga trafik utan sudo.
- dumpcap ägs av root:wireshark med behörigheten -rwxr-xr--: bara root och gruppen wireshark får köra programmet.
- dumpcap har bara två capabilities: cap_net_raw och cap_net_admin (getcap). Den har alltså exakt de rättigheter som behövs för att fånga paket, inte full root-makt.
- Startade Wireshark som vanlig användare: eth0 syns med trafikkurva och inget behörighetsfel.
- Varför: Wiresharks protokolltolkar har haft sårbarheter. Körs Wireshark som root kan skadlig trafik i värsta fall ge en angripare full kontroll. Att bara låta ett litet program fånga med minimala rättigheter är principen om minsta privilegium.
- Lärdom: extcap-verktyget sshdump kan fånga trafik på en annan maskin via SSH, användbart för servern som saknar GUI.
- Ingen ändring gjordes i Kali, så ingen ny ögonblicksbild behövdes.