🔋 SOLARCUBE LOKAAL IN HOME ASSISTANT — EIGEN INTEGRATIE + COMPLETE HANDLEIDING
Velen van jullie hebben de HA-EMS integratie van AEG (de nj.zip) al draaien. Goed nieuws: de lokale API van de Solarcube werkt inmiddels — en ik heb er een alternatieve, "schone" Home Assistant-integratie voor gebouwd: Solarcube Local. Hieronder wat het is, wat je eraan hebt, en een volledige installatiehandleiding — óók als je de AEG-versie al hebt draaien (lees dan zeker stap 1!).
═══════════════════════════
WAT IS HET / WAT HEB JE ERAAN
═══════════════════════════
✅ Volledig lokaal: rechtstreeks via je eigen netwerk (TCP poort 8080), geen cloud, geen account, geen internet nodig
✅ Echte HA-sensoren mét eenheden en device classes: batterij-SOC, laad-/ontlaadvermogen, netvermogen (meterdata), PV-vermogens — direct bruikbaar in automatiseringen, statistieken en het Energie-dashboard
✅ Snelheid instelbaar van 1 t/m 300 seconden (de cloud ververst maar eens per ±5 minuten; lokaal kan elke seconde)
✅ Twee apparaten in HA: het systeem (totalen) + je opslagmodule(s) per serienummer
✅ Filtert af en toe voorkomende onzin-meetwaarden van de firmware er automatisch uit
⚠️ Wat (nog) NIET kan: de batterij aansturen. De besturingscommando's van de lokale API worden door de huidige firmware nog niet beantwoord — dat geldt ook voor de AEG-integratie zelf. Dit komt vermoedelijk met een volgende firmware-/app-update (ligt als vraag bij AEG). Uitlezen werkt volledig.
⚠️ Disclaimer: hobby-project, niet van of namens AEG, gebruik op eigen risico. De integratie leest alleen uit, dus je kunt niets kapotmaken aan je Solarcube.
═══════════════════════════
STAP 1 — HEB JE DE AEG-INTEGRATIE (HA-EMS) AL DRAAIEN? EERST DIT LEZEN
═══════════════════════════
Belangrijk om te weten: de Solarcube accepteert maar een zeer beperkt aantal gelijktijdige lokale verbindingen. En de AEG-integratie maakt óók in Cloud Mode automatisch een lokale verbinding (via apparaat-discovery). Twee integraties die tegelijk lokaal verbinden = periodieke uitval bij één van beide of allebei. Kies daarom bewust:
OPTIE A — Alleen Solarcube Local (mijn advies als je vooral sensoren/automatiseringen wilt):
1. Instellingen → Apparaten & Services → HA-EMS/Aecc → drie puntjes → Verwijderen
2. Daarna verder met stap 2 hieronder
Let op: gebruik je template-sensoren die op de Aecc-entiteiten leunen (sensor.aecc_ha_ems_series_...)? Die moet je daarna ompunten naar de nieuwe sensoren.
OPTIE B — Beide naast elkaar (als je het AEG-paneel/de cloud-data wilt houden):
Dit werkt bij mij wisselend en kan uitval geven. Wil je het proberen: zet HA-EMS op Cloud Mode (verwijderen en opnieuw toevoegen, dan Cloud kiezen) en hou de Activiteit-log van beide in de gaten. Valt er regelmatig iets uit → terug naar optie A.
OPTIE C — Niets doen: heb je HA-EMS in Local Mode draaien en ben je tevreden? Dan hoef je niet te wisselen. Deze integratie is een alternatief, geen verplichting. 😉
═══════════════════════════
STAP 2 — BESTANDEN INSTALLEREN
═══════════════════════════
Nodig: de zip uit dit bericht (solarcube_local_v0.3.zip), uitgepakt op je PC.
Via de File Editor add-on (zit in HA):
1. Open File Editor → mapje linksboven → navigeer naar /config/custom_components
2. Maak daar een nieuwe map: solarcube_local
3. Maak dáárin een map: translations
4. Upload in solarcube_local deze bestanden: __init__.py, api.py, config_flow.py, const.py, coordinator.py, manifest.json, sensor.py, services.yaml, strings.json
5. Upload in translations: en.json en nl.json
6. Controleer: manifest.json moet DIRECT in solarcube_local staan, niet in een submap (klassieke fout: dubbel geneste map)
Via Samba/SSH kan natuurlijk ook: hele map solarcube_local uit de zip naar /config/custom_components/ kopiëren.
Daarna: Instellingen → Systeem → Herstarten (volledige herstart, geen snelle herlaad).
═══════════════════════════
STAP 3 — IP-ADRES VAN JE SOLARCUBE OPZOEKEN
═══════════════════════════
Kijk in je router bij de aangesloten apparaten; de Solarcube meldt zich met naam en serienummer (bijv. "AEG Solarcube A24CM..."). Noteer het IP-adres.
Tip: geef de Solarcube in je router een vast IP (DHCP-reservering), anders werkt de integratie niet meer zodra het adres ooit wisselt.
═══════════════════════════
STAP 4 — INTEGRATIE TOEVOEGEN
═══════════════════════════
1. Instellingen → Apparaten & Services → Integratie toevoegen
2. Zoek: Solarcube Local
3. Vul in: het IP-adres van je Solarcube en poort 8080
4. Verzenden — de setup test direct de verbinding
Bij succes verschijnen twee apparaten: "Solarcube (jouw IP)" met de systeemtotalen en "Solarcube opslag <serienummer>" met de waarden van je opslagmodule. De handigste sensor voor automatiseringen: "Meter totaal actief vermogen" — dat is je netto netvermogen, zoals je P1 hem ziet.
═══════════════════════════
STAP 5 — POLLING-SNELHEID INSTELLEN (OPTIONEEL)
═══════════════════════════
Standaard ververst alles elke 5 seconden. Sneller of langzamer:
Instellingen → Apparaten & Services → Solarcube Local → Configureren → interval aanpassen (1–300 s). Bij mij draait 1 seconde al de hele dag stabiel.
═══════════════════════════
PROBLEMEN OPLOSSEN
═══════════════════════════
❌ "Kan geen verbinding maken met het apparaat"
→ Meestal: een andere client houdt de verbinding bezet. Hád je HA-EMS draaien? Verwijder/deactiveer die, herstart HA en probeer opnieuw.
→ Check het IP-adres (verwar de Solarcube niet met je P1-meter!)
→ Poort is altijd 8080
❌ Integratie deed het, maar valt af en toe uit
→ Vrijwel altijd: er verbindt iets anders ook lokaal met de cube (HA-EMS, tweede HA-instantie, eigen scripts). Eén lokale client aanhouden.
❌ "Integratie niet gevonden" bij toevoegen
→ Bestanden staan in een verkeerde map (zie stap 2, punt 6), of HA is niet volledig herstart.
❓ Sensor toont heel even een absurde waarde
→ De firmware geeft soms direct na het verbinden één onzin-waarde terug; de integratie filtert die op de module-sensoren automatisch weg (alles boven 4000 W wordt genegeerd).
═══════════════════════════
Vragen of rare resultaten? Zet het in de comments, het liefst met een screenshot van je sensoren of de melding. En zodra AEG de besturingscommando's in de firmware activeert, volgt hier een update — de integratie is daar al op voorbereid. ⚡
Nogmaals, dit is een eigen hobby project dat ik samen met mijn "vriend" Claude AI heb gemaakt. Op het gebied van HA ben ik eigenlijk nog maar een beginner, hoewel ik nu meestal wel het meeste weet te vinden dat Claude aangeeft.
