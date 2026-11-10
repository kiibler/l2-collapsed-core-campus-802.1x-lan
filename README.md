# l2-collapsed-core-campus-802.1x-lan
Kahden rakennuksen L2 Collapsed-Core kampusverkon toteutus, jossa hyödynnetään porttikohtaista 802.1X-autentikaatiota, dynaamista VLAN-määritystä ja OSPF Unnumbered runkoverkkoa. Toteutin tämän projekin Kaakkois-Suomen ammattikorkeakoulun VirtualLab-alustalla ja kytkiminä käytin Ciscon vIOSl2-kuvia.

## Verkon topologia
![Verkon topologia](topologia/L2-collapsed-core-network.png)

## VLAN-segmentointi ja IPv4-osoitteet

Verkko muodostuu kahdesta rakennuksesta (Rakennus 1, Rakennus 2). Yrityksen sisäinen IPv4-osoitteisto on suunniteltu siten, että pohjana käytettiin IPv4 aluetta: `172.16.0.0/16`. Rakennukselle 1 annettiin IPv4 osoitealue: `172.16.1.0/24` ja Rakennukselle 2: `172.16.2.0/24`. Rakennuskohtaiset verkot on jaettu pienempiin osiin siten, että jokaiselle VLAN:lle on oma `/26` aliverkko. Käytännössä tämän kokoisten aliverkkojen kanssa IPv4 osoitteet loppuisivat nopeasti kesken, mutta ne sopivat laboratorioympäristöön.

- **VLAN 10 (STAFF):** `172.16.X.0/26`
- **VLAN 20 (GUEST):** `172.16.X.64/26`
- **VLAN 30 (MANAGEMENT)**: `172.16.X.128/26`
- **VLAN 40 (SERVICES)**: `172.16.X.192/26`
- **VLAN 99 (UNUSED)**: Ei IPv4 osoitealuetta. Käytetään Trunk-linkkien native VLANina estämään VLAN Hopping -hyökkäykset VLAN 1:stä.
- **VLAN 999 (UNAUTH)**: Ei IPv4 osoitealuetta. Käytetään 802.1X-porttien oletus VLANina ja karanteeni VLANina, mikäli päätelaitteen autentikointi epäonnistuu.

## Vikasietoisuus, kaistanleveys ja EtherChannel

Vikasietoisuuden ja kaistanleveyden parantamiseksi, Access- ja Distribution-tason kytkimen ovat yhdistetty toisiinsa L2 **EtherChannel**-liitännällä. Jokaiseen loogiseen EtherChannel-liitäntään liitettiin kaksi Etherent-kaapelia. EtherChannel-liitännän muodostamiseen käytettiin **LACP**:tä.

## Spanning Tree Protocol

Kytkimillä käytetään Ciscon **Rapid PVST+** -protokollaa. Rakennusten sisäiseen verkkoon ei muodostu L2-silmukoita, koska Distribution-kytkimet on yhdistetty toisiinsa L3-porteilla. Tästä huolimatta **Root Bridge** -roolit on jaettu prioriteetti arvojen avulla siten, että rakennusten sisällä kumpikin Distribution-kytkin on kahden VLANin **Root Bridge**. Esimerkiksi: Rakennuksen 1 DSW1-laite toimii **Root Bridge** -roolissa VLAN:lle 10 ja 20.

## Päätelaitteiden redundanttinen Default Gateway ja HSRP

Päätelaitteiden Default Gatewayn vikasietoisuus on toteutettu ottamalla käyttöön **HSRP**. Kuten **STP Root Bridge** -roolit, myös **HSRP Active** -roolit on jaettu prioriteetti arvojen avulla Distribution-kytkimille siten, että kumpikin kytkin toimii *HSRP Active* -roolissa kahdelle VLAN:lle. Esimerkiksi: Rakennuksen 1 DSW1-laite toimii VLAN 10:n ja 20:n **HSRP Active** -laitteena. STP ja HSRP roolit on jaettu siten, että L2-tason ja L3-tason optimaaliset reitit täsmäävät toistensa kanssa. Näin myös saavutetaan liikenteen kuormanjakausta verkon sisällä, koska eri VLANien liikenteet jakaantuvat Distribution-tason kytkimien välillä. **HSRP Active** -roolissa toimivat Distribution-kytkimet hyödyntävät **preempt** toiminnallisuutta, jolla varmistetaan **HSRP Active** -roolin tarkoituksenmukainen jako.

- **VLAN 10 (STAFF) HSRP VIP:** `172.16.X.1/26`
- **VLAN 20 (GUEST) HSRP VIP:** `172.16.X.65/26`
- **VLAN 30 (MANAGEMENT) HSRP VIP**: `172.16.X.129/26`
- **VLAN 40 (SERVICES) HSRP VIP**: `172.16.X.193/26`

## Rakennusten välinen runkoverkko, OSPF ja IP Unnumbered

Rakennukset on yhdistetty toisiinsa **Full Mesh** -verkkotopologialla, jonka yli ajetaan dynaamista reititysprotokollaa **OSPFv2**. Runkoverkko on konfiguroitu ilman linkkien omia IPv4 osoitteita. Sen sijaan jokainen linkki hyödyntää `ip unnumbered loopback 0` -menetelmää ja lainaa **loopback 0** -interfacen IPv4 osoitetta. Jotta menetelmä toimisi, linkkien **OSPF**-verkkotyypiksi on konfiguroitu **point-to-point**. Tällä tavoin ei tarvitse suunnitella jokaiselle linkille transit-verkkoa, ja Distribution-laitteiden reitityspöydät säilyvät yksinkertaisina.

## 802.1X ja FreeRadius

Verkon porttikohtainen tietoturva on toteutettu hyödyntämällä 802.1X-standardia yhdessä kahden Debian-pohjaisen FreeRadius-palvelimen kanssa.

### 1. 802.1X-portin elinkaari ja karanteeni VLAN
- Ennen autentikoitumisprosessin alkua, portti on sijoitettu karanteeni VLAN:iin 999 (UNAUTH). Tässä tilassa kytkin sallii ainoastaan **EAPOL**-liikenteen.

### 2. Autentikoitumisprosessi ja dynaaminen VLAN-määritys
- Kun Windows 11 -päätelaite kytkeytyy porttiin, autentikoitumisprosessi aloitetaan, joko kytkimen **EAP-Request/Identity**-viestillä tai Windows 11 -päätelaitteen **EAPOL Start**-viestillä. Prosessi on konfiguroitu hyödyntämään **PEAP/MSCHAPv2** protokollaa.
- Kytkin toimii **Authenticator**-roolissa päätelaitteen ja FreeRadius-palvelimen kanssa, joka on sijoitettu VLANiin 40 (SERVICES).
- Onnistuneen autentikaation jälkeen FreeRadius-palvelin palauttaa **Access-Accept**-viestin mukana tarvittavat tiedot, jotka ohjaavat portin oikeaan VLANiin:
    - *Tunnel-Type = VLAN*
    - *Tunnel-Medium-Type = IEEE-802*
    - *Tunnel-Private-Group-Id = 10*
- Kytkin siirtää portin oikeaan VLANiin ja liikenne portin läpi sallitaan.

### 3. GUEST VLAN
- Jos päätelaite ei tue autentikointia eikä vastaa kytkimen lähettämiin **EAP**-viesteihin, portti siirretään VLANiin 20 (GUEST). Tämän VLANin tarkoitus on antaa päätelaitteille mahdollisuus rajattuun pääsyyn verkon resursseihin, esimerkiksi mahdollisuus ladata 802.1X-asiakasohjelmisto.

### 4. Epäonnistunut autentikaatio
- Jos päätelaitteen autentikoitumisprosessi epäonnistuu, kytkin siirtää portin VLANiin 999 (UNAUTH). Autentikoitumisprosessi alkaa alusta, kun päätelaite lähettää **EAPOL Start** -viestin kytkimelle.

## Validaatiot

Verkon toimivuuden, porttikohtaisen autentikaation ja reitityksen validointiin käytetyt komennot ja tulosteet löytyvät `validaatiot/`-kansiosta:
1. [802.1X ja Radius](validaatiot/802.1X-validaatiot.md)
2. [STP ja HSRP](validaatiot/stp-hsrp-validaatiot.md)
3. [OSPF ja reititys](validaatiot/ospf-validaatiot.md)
