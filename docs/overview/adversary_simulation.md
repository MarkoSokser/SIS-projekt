# Adversary Simulation s MITRE CALDERA

Pregled:
U ovom projektu, vaš tim će dizajnirati i izvršiti simulirani kibernetički napad na lažnu korporativnu
mrežu koristeći MITRE CALDERA. Postavit ćete virtualno enterprise okruženje, implementirati obrambene kontrole, simulirati tehnike napada usklađene s MITRE ATT&CK okvirom i procijeniti učinkovitost obrana.

Ciljevi:
- Steći praktično iskustvo u emulaciji protivnika i radnim tokovima red/blue timova.
- Naučiti kako postaviti i osigurati realistično enterprise okruženje.
- Mapirati CALDERA tehnike napada na MITRE ATT&CK okvir.
- Procijeniti učinkovitost sigurnosnih kontrola u otkrivanju ili sprječavanju pojedinih napada.
- Razviti timski rad, tehničku dokumentaciju i vještine izvještavanja o incidentima.

Uloge u timu (4 studenta):
1. Infrastructure Engineer — gradi virtualno mrežno okruženje (AD domena, Linux serveri, endpointi, logiranje).
2. Blue Team Engineer — implementira i konfigurira obrambene kontrole (firewall, IDS/IPS, EDR, SIEM).
3. Red Team Operator — planira i izvodi CALDERA napadačke scenarije koristeći ATT&CK tehnike.
4. Threat Hunter/Analyst — nadzire logove, korrelira događaje i izrađuje analizu incidenta.

Praktični zadaci:
- Dizajnirati lažnu kompanijsku mrežu s najmanje 3–4 stroja (npr. Windows AD kontroler, Linux server, radne stanice).
- Implementirati osnovne sigurnosne kontrole (npr. Snort, Suricata, Wazuh, Splunk, Windows Defender).
- Upotrijebiti CALDERA za provođenje višefazne napadačke kampanje (initial access → persistence → lateral movement → data exfiltration).
- Dokumentirati otkrivanja: koje su tehnike zaustavljene, detektirane ili su ostale neprimijećene.
- Poboljšati obrane i ponovno pokrenuti CALDERA kako bi se izmjerile promjene u otkrivenosti.

Isporuke:
- Setup vodič — arhitektura VM/mreže, konfiguracije, instalacija alata.
- Attack-Defense matrica — ATT&CK tehnike koje su korištene vs. rezultat detekcije.
- Kronologija incidenta — slijed događaja s dokazima (logovi, screenshotovi).
- Završno izvješće — uočene ranjivosti, učinkovitost kontrola, preporuke za poboljšanja.
- Prezentacija/Demo — demonstracija jednog napada i njegovog otkrivanja/mitigacije.

Proširenja (opcionalno, za bonus):
- Usporediti CALDERA s drugim alatima za simulaciju.
- Automatizirati CALDERA izvođenja koristeći njegov REST API.
- Procijeniti kako se rezultati razlikuju među različitim OS ili konfiguracijama sigurnosnih alata.
