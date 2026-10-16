# Klapstuk

Een platformvechter voor 2 tot 4 spelers in de browser: drieëntwintig vechters, zeven podia, items die je stuk voor stuk aan of uit zet, een Klapbal waarmee elke vechter een eigen final smash doet, en CPU-tegenstanders van niveau 1 tot 12. Speel lokaal op één toetsenbord of gamepad, of online met vrienden.

De hele site is één bestand: `index.html`. De WebRTC-bibliotheek zit erin en het spel gebruikt alleen standaardlettertypen (Impact en het systeemlettertype); de pagina laadt niets van andere websites. Er is geen bouwstap en geen eigen server nodig.

## Online zetten

Kies één van deze manieren. Alle drie zijn gratis.

**Netlify Drop (snelst, zonder account)**
1. Ga naar https://app.netlify.com/drop
2. Sleep de map `klapstuk-web` in het venster.
3. Je krijgt meteen een adres zoals `https://iets-willekeurigs.netlify.app`.

**GitHub Pages**
1. Maak een nieuwe, openbare repository en zet `index.html` erin.
2. Ga naar *Settings → Pages*, kies de branch `main` en de map `/ (root)`.
3. Na een minuut staat de site op `https://<gebruikersnaam>.github.io/<repository>/`.

**Cloudflare Pages**: maak een project via *Upload assets* en upload de map.

Lokaal testen kan ook. Open `index.html` niet als bestand, maar start een kleine server in deze map:

```bash
python -m http.server 8000
```

Ga daarna naar http://localhost:8000.

## Training en toetsen

- **Training: bokszak** (in het menu) zet je eerste menselijke speler tegenover een bokszak die niets doet, met onbeperkt levens. Linksboven staan je combo, de beste combo, de laatste klap en welke aanvallen verzwakt zijn door herhaling. Met **R** zet je de zak terug en met **H** toon je raakvlakken (ook via het pauzemenu).
- Onder **Besturing en technieken** kun je de toetsen van beide toetsenbord-indelingen aanpassen: klik op een toets en druk de nieuwe. Je instellingen blijven bewaard in je browser.

## Online spelen

1. Zet de schakelaar bovenaan het menu op **Online**, vul je naam in en kies **Nieuw spel hosten**.
2. Stuur de **uitnodigingslink** (of de code van 4 tekens) naar je vrienden. Wie de link opent, doet meteen mee.
3. Iedereen kiest een vechter in hetzelfde rooster als bij lokaal spelen, en een besturing op de eigen spelerkaart. De host kiest het podium, de levens, de items en of lege plekken met CPU's worden gevuld; de anderen zien die keuzes meteen.
4. De host drukt op **Start!**.

Er spelen maximaal 4 mensen. Wie later komt, kijkt mee. Niemand heeft een account nodig.

## Hoe de verbinding werkt

- De browsers verbinden rechtstreeks met elkaar via **WebRTC**, met de bibliotheek [PeerJS](https://peerjs.com) 1.5.4 (MIT-licentie, ingebouwd in `index.html`).
- Om elkaar te vinden gebruiken ze de **gratis openbare signaleringsserver van PeerJS** (`0.peerjs.com`). Daarna loopt het spelverkeer direct tussen de spelers.
- De **host** draait het echte spel en stuurt ongeveer 30 keer per seconde de stand naar de gasten. Gasten sturen alleen hun knoppen. Houd als host het tabblad op de voorgrond: browsers vertragen tabbladen op de achtergrond.
- Voor het vinden van een route door routers gebruikt het spel de STUN-servers van Google en Cloudflare.
- Is er een **TURN-server** ingesteld (zie hieronder), dan stuurt die het verkeer door als een directe verbinding niet lukt.

## TURN-server instellen

Sommige bedrijfs-, school- of mobiele netwerken blokkeren directe WebRTC-verbindingen. Een TURN-server lost dat op door het verkeer door te sturen. De vroegere gratis publieke TURN-servers bestaan niet meer: die van PeerJS (`eu-0`/`us-0.turn.peerjs.com`) hebben geen DNS-record meer, en Open Relay (`openrelay.metered.ca`) is uitgezet. Een TURN-server vraagt dus een eigen (gratis) account.

De instellingen staan bovenaan het script `window.KlapP2P` in `index.html`. Zoek naar `TURN-instellingen` en vul **één** van de twee in.

**Optie 1: Metered (gratis basisabonnement)**
1. Maak een account op https://www.metered.ca/stun-turn en maak een app aan, bijvoorbeeld `klapstuk`.
2. Maak onder *TURN Server → Credentials* een API-sleutel aan.
3. Vul het adres in dat Metered geeft:
   ```js
   const TURN_CREDENTIALS_URL = 'https://klapstuk.metered.live/api/v1/turn/credentials?apiKey=JOUW_SLEUTEL';
   ```

**Optie 2: vaste gegevens** (Metered, ExpressTURN, je eigen coturn-server, …)
```js
const TURN_SERVERS = [{
  urls: ['turn:HOST:80', 'turn:HOST:443?transport=tcp', 'turns:HOST:443?transport=tcp'],
  username: 'NAAM',
  credential: 'WACHTWOORD',
}];
```

Let op: deze gegevens staan leesbaar in een openbare pagina. Iedereen die de bron bekijkt, kan je TURN-tegoed gebruiken. Zet bij je TURN-aanbieder daarom een limiet op het verbruik, en waar dat kan een beperking tot je eigen domein.

## Bekende beperkingen

- **Gratis signaleringsserver.** De openbare PeerJS-server heeft geen garantie op beschikbaarheid. Ligt hij plat, dan kun je wel lokaal spelen maar geen nieuwe online spellen starten. Je kunt een eigen [PeerServer](https://github.com/peers/peerjs-server) draaien en `host`, `port` en `path` toevoegen aan `PEER_OPTS` in het script `window.KlapP2P`.
- **Vertraging.** Gasten zien het spel een paar frames later dan de host, en hun invoer reist eerst naar de host. Bij veel afstand of een trage verbinding voelt een gast dat.
