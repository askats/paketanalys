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

## 2026-09-29 – Steg 3: ARP och ICMP – ping från Kali till servern
- Tömde Kalis ARP-cache (sudo ip neigh flush all) för att tvinga fram en ny ARP-fråga, startade Wireshark på eth0 och pingade servern 4 gånger. Fångsten sparades som ~/fangster/steg3-ping.pcapng på Kali (publiceras inte).
- ARP: Kali frågade via broadcast (ff:ff:ff:ff:ff:ff, EtherType 0x0806) "Who has 10.10.10.3?".
- ICMP: 4 par echo request/reply med samma id och seq 1–4, ca 0,5 ms svarstid, TTL 64 (Linux standard, ingen router passerades).
- Avvikelse 1: okänd granne 10.10.10.2 (MAC 08:00:27:69:58:f3) i ARP-cachen. Hypotes: VirtualBox DHCP-server. Verifierad med nmcli (dhcp_server_identifier = 10.10.10.2). Kort lånetid (600 s) förklarar varför Kali ofta pratar med den.
- Avvikelse 2: två olika ARP-svar för 10.10.10.3, först 52:54:00:12:35:00 och 2,6 ms senare serverns riktiga 08:00:27:da:4a:bc. Wireshark Expert Information varnade: "Duplicate IP address configured".
- Utredning: Linux godtog det första svaret och ignorerade det andra. Kalis ARP-cache och Ethernet-huvudena i både request och reply visade att all trafik gick via 52:54:00:12:35:00 i båda riktningarna.
- Slutsats: 52:54:00:12:35:00 är VirtualBox NAT-motor (lokalt administrerad MAC). Ett känt beteende i VirtualBox NAT-nätverk, ofarligt här.
- Varför det spelar roll: exakt samma mönster (dubbla ARP-svar, trafik via en mellanhand) är kännetecknet för ARP-spoofing och man-in-the-middle. På IP-nivå såg allt normalt ut, avvikelsen syntes bara på lager 2.
- Lärdom: OUI (de tre första byten i MAC-adressen) avslöjar tillverkaren. 08:00:27 = VirtualBox virtuella nätverkskort. Lokalt administrerade adresser är tilldelade av mjukvara.
- Lärdom: IPv6 link-local-adressen byggs av MAC-adressen (EUI-64), vilket läcker hårdvaruidentitet.
- Arbetssätt: avvikelse → hypotes → verifiering med flera oberoende källor → slutsats.

## 2026-09-30 – Steg 4: DNS – uppslagningar från Kali
- Kalis DNS-server är 172.20.10.1 (utdelad av DHCP, vidarebefordrad från Macens nätverk av VirtualBox). Stängde av servern med sudo poweroff för att spara minne.
- Fångade två uppslagningar med dig på eth0: example.com och finnsinte.example. Fångsten sparades som ~/fangster/steg4-dns.pcapng på Kali (publiceras inte).
- Första fångsten saknade DNS-paket: dig hade körts utanför fångstfönstret och i flera terminaler. Felsökte, stängde överflödiga terminaler och Wireshark-instanser (jobs, kill %1) och gjorde om fångsten.
- DNS gick okrypterat över UDP port 53 via gatewayen (Ethernet-destination 52:54:00:12:35:00, NAT-motorn). Den som ser trafiken ser vilka namn som slås upp.
- Kopplade verktyg och paket: dig visade id 3573, Wireshark 0x0df5, samma värde i hex.
- Slumpad källport (38306) och transaktions-ID skyddar mot DNS cache poisoning: ett falskt svar måste gissa båda.
- example.com: två A-poster, 104.20.23.154 och 172.66.147.243 (redundans och lastbalansering), TTL 313 s. Avkodade svaret för hand ur hex (c0 0c = namnpekare, typ A, klass IN, TTL 0x139, 4 byte adress).
- Flaggor 0x8180: svar, ej authoritative, recursion desired och available. 172.20.10.1 är en rekursiv resolver, inte ägare av namnet.
- finnsinte.example: NXDOMAIN med SOA från rotservrarna. .example är en reserverad toppdomän.
- Verifierade med whois att 104.20.23.154 tillhör Cloudflare, Inc. (104.16.0.0/12). IP-adresser hos CDN säger lite om vem som ligger bakom tjänsten.
- SOC-koppling: många NXDOMAIN från en värd kan tyda på skadlig kod som provar slumpade domäner (DGA). Låga TTL kan användas för snabbt byte av IP (fast flux).
- Lärdom: en tom filtrering betyder inte att filtret är fel, utan att inget matchar. Felsök uppifrån: körde kommandot, vart gick trafiken, fångade jag vid rätt tid och på rätt gränssnitt?