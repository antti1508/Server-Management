# h3 Demoni

# x) Lue ja tiivistä

### Tiivistelmä: Karvinen 2026: Apache installed with Ansible - quick notes (https://terokarvinen.com/apache-ansible/)

* Apache 2 -web-palvelin asennetaan ja määritetään automaattisesti Ansiblen avulla kolmessa vaiheessa: asenna paketti (```apt```), kopioi/linkitä asetustiedostot (```copy```, ```file```) ja käynnistä palvelu uudelleen (```systemd```).
* ```roles/apache2/``` koodi on jaettu loogisesti kolmeen kansioon: ```tasks/``` (asennus- ja tiedostotoiminnot), ```files/``` (valmiit asetustiedostot) ja ```handlers/``` (palvelun uudelleenkäynnistys).
* Virtuaalipalvelimen asetustiedosto (```example.com.conf```) ohjaa sivuston juurihakemiston (```/home/tero/publicsite/```) tavallisen käyttäjän kotikansioon, joten verkkosivuja voi muokata ilman root- tai sudo-oikeuksia.
* Asetustiedostojen muokkaus laukaisee ```notify```-toiminnon kautta handlerin, joka käynnistää ```apache2```-palvelun uudelleen vasta, kun asetuksiin on tehty muutoksia.

### Tiivistelmä: Ansible Community Documentation: Handlers: running operations on change (https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html)

* Handlerit ovat Ansiblen tehtäviä, jotka suoritetaan vain silloin, kun jokin toinen tehtävä tekee kohteeseen muutoksen ja lähettää niille ilmoituksen (notify). Tyypillinen esimerkki on palvelun uudelleenkäynnistys vain silloin, kun sen asetustiedosto on muuttunut – jos muutoksia ei ole, palvelua ei käynnistetä turhaan uudelleen.
* Tehtävät käynnistävät handlereita ```notify```-avainsanalla, jolle annetaan joko yksittäisen handlerin nimi tai lista nimiä.
* Handlerit ajetaan siinä järjestyksessä kuin ne on määritelty ```handlers```-osiossa (ei ```notify```-listan järjestyksessä). Lisäksi vaikka useampi tehtävä ilmoittaisi samalle handlerille saman ajon aikana, Ansible suorittaa sen lopuksi vain kerran välttääkseen turhia palvelukatkoja.
* Oma huomio: On todella kätevää, että Ansible deduplikoi ilmoitukset automaattisesti eli ajaa saman handlerin vain kerran, vaikka viisi eri asetustiedostoa muuttuisi.

### Tiivistelmä: ansible-doc service (https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_module.html#ansible-collections-ansible-builtin-service-module)

* ```ansible.builtin.service``` -moduulilla hallitaan palveluita (daemoneja) kohdekoneilla. Se toimii yleiskäyttöisenä rajapintana taustajärjestelmän varsinaiselle palvelunhallinnalle (kuten ```systemd```, ```sysvinit```, ```upstart``` tai ```openrc```) tunnistaen oikean järjestelmän automaattisesti.
* enabled (valinnainen, boolean): Määrittää, käynnistyykö palvelu automaattisesti, kun järjestelmä käynnistetään (boot). Vaatii käytännössä joko tämän tai ```state```-parametrin määrittelyn.
* name (pakollinen, string): Hallittavan palvelun nimi (esimerkiksi ```httpd``` tai ```apache2```)
* state (valinnainen, string): Määrittää palvelun halutun tilan (```started```, ```stopped```, ```restarted``` tai ```reloaded```). Tiloista ```started``` ja ```stopped``` ovat idempotentteja (eli ne tekevät muutoksen vain, jos palvelu ei jo ole kyseisessä tilassa), kun taas ```restarted``` käynnistää palvelun aina uudelleen.
* EXAMPLES: Käyttöesimerkeissä näytetään tyypillisimmät komennot: yksittäisen palvelun käynnistys (```state: started```), pysäytys (```state: stopped```), uudelleenkäynnistys (```state: restarted```), asetusten uudelleenlataus (```state: reloaded```), automaattisen käynnistyksen kytkentä päälle (```enabled: true```) sekä usean palvelun käynnistäminen silmukassa (```loop```).

# y) Infra-uutiset

Infisical-blogissa julkaistu artikkeli tarkastelee perinteisen keskitetyn identiteetin- ja pääsynhallintajärjestelmän, Microsoft Active Directoryn (AD), soveltuvuutta nykyaikaiseen pilvi-infrastruktuuriin. Kirjoittajan mukaan AD on alun perin suunniteltu staattisille konesaliympäristöille, joissa ylläpitäjä hyväksyy manuaalisesti pitkäikäiset fyysiset palvelimet ja työasemat osaksi toimialuetta (Lobsien, 2026).

**Artikkelin keskeiset väitteet ovat:**
* **Staattinen vs. dynaaminen infrastruktuuri:** Nykyaikaisessa pilvi- ja konttiympäristössä (esim. Kubernetes-podit, CI/CD-ajurit ja testitietokannat) koneidentiteettejä luodaan ja tuhotaan automaattisesti tuhansia kertoja päivässä. Perinteinen ihmisen hyväksyntään nojaava keskitetty hakemisto ei skaalaudu tällaiseen vauhtiin (Lobsien, 2026).
* **Tietoturvaongelmat automaatiossa:** Kun vanhoja AD-ympäristöjä yritetään pakottaa osaksi modernia automaatiota, organisaatiot päätyvät usein kovakoodaamaan ylläpitäjätunnuksia skripteihin uusien koneiden liittämiseksi. Tämä kumoaa keskitetyn valvonnan alkuperäisen idean ja luo vakavia tietoturva-aukkoja (Lobsien, 2026).
* **Uusi toimintamalli:** Kirjoittaja esittää, että keskitetyn hakemistopalvelun sijaan moderni infrastruktuuri siirtyy kohti identiteettien liittouttamista (federation, kuten OIDC) ja salaisuuksien välitystä (credential brokering), joissa palvelut todentavat itsensä dynaamisesti ilman raskasta keskitettyä toimialuerakennetta (Lobsien, 2026).

Artikkeli herätti Hacker Newsissa laajan ja kriittisen keskustelun pitkän linjan järjestelmäasiantuntijoiden keskuudessa. Kommentoijat haastoivat artikkelin väitteen siitä, että keskitetyn hallinnan ja Active Directoryn aikakausi olisi ohi (Hacker News, 2026):
* **Ryhmäkäytäntöjen (Group Policy, GPO) ylivoimaisuus:** Useat asiantuntijat korostivat, että AD:n keskitetty ryhmäkäytäntömoottori on edelleen ylivertainen työasemien ja palvelimien hallinnassa. Sen avulla tuhansia asetuksia, verkko-asemia, tulostimia ja tietoturvasääntöjä voidaan kohdentaa dynaamisesti käyttäjä- ja konetasolla ilman monimutkaisia skriptejä – ominaisuus, jota modernit pilvipohjaiset MDM-ratkaisut eivät ole vieläkään täysin korvanneet (Hacker News, 2026).
* **AD:n ominaisuuksien väärinymmärrys:** Kommentoijat huomauttivat artikkelin sisältävän teknisiä epätarkkuuksia. Esimerkiksi koneiden liittäminen toimialueeseen ei vaadi ylläpitäjän manuaalista hyväksyntää (ms-DS-MachineAccountQuota), ja palvelutilien salasanojen hallintaan on jo olemassa automatisoitu, salasanaton gMSA-ratkaisu (Group Managed Service Accounts), jota jopa AWS suosittelee Windows-pohjaisille konteille (Hacker News, 2026).
* **Paikallisten (on-premises) ympäristöjen välttämättömyys:** Keskustelussa muistutettiin, että kriittisillä toimialoilla – kuten terveydenhuollossa, viranomaisissa ja teollisuudessa – laitteita ei voida tai haluta kytkeä pilvipalveluihin tietoturva- ja toimintavarmuussyistä. Näissä ympäristöissä keskitetty paikallinen hallinta on edelleen ehdoton vaatimus (Hacker News, 2026).
* **Liika hallinta hidastaa kehitystä:** Toisaalta kommentoijat tunnistivat artikkelin kuvaaman byrokratiaongelman. Kun keskitetty hallinta ja tietoturvahallinto paisuvat liian raskaiksi, kehittäjät alkavat kiertää sääntöjä esimerkiksi tilaamalla omia pilvipalvelimia luottokortilla sen sijaan, että odottaisivat viikkoja sisäisen IT-osaston hyväksyntää (Hacker News, 2026).

# a) Apassi. Asenna Apache 2 käsin.

Asennus aloitettiin päivittämällä paketinhallinnan paikallinen pakettilista ja asentamalla ```apache2```-ohjelmistopaketti (Debian Wiki 2024). Palvelu asetettiin käynnistymään automaattisesti järjestelmän käynnistyksen yhteydessä (The Apache Software Foundation 2026). Tähän käytin komentoja:

```console
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable --now apache2
```
Asennuksen onnistuminen varmistettiin tarkistamalla palvelun tila komennolla ```systemctl status apache2```, joka vahvisti palvelun olevan aktiivinen (```active (running)```). Lisäksi yhteys testattiin paikallisesti hakemalla etusivu osoitteesta ```http://localhost```, jolloin palvelin vastasi Apachen oletussivulla (Debian Wiki 2024). Alempana kuvat testauksista:

<img width="1466" height="457" alt="kuva" src="https://github.com/user-attachments/assets/fbf10d56-0c8f-4991-97d6-e46698150468" />

Kuva 1. Apachen status.

<img width="2549" height="1305" alt="kuva" src="https://github.com/user-attachments/assets/8666132f-788e-409b-83fe-d15f1034cd79" />

Kuva 2. Apachen etusivu.

Oletusarvoisesti Apachen verkkosivujen juurihakemisto (```/var/www/html```) ja sen sisällä olevat tiedostot ovat ```root```-käyttäjän omistuksessa, mikä estää tavallista käyttäjää muokkaamasta sivustoa ilman korotettuja oikeuksia (Debian Wiki 2024).

Ongelma ratkaistiin siirtämällä hakemiston omistajuus suoraan tavalliselle käyttäjälle (```s200963```) ja säilyttämällä ryhmäomistajuus verkkopalvelimen omalla ```www-data```-ryhmällä (Debian Wiki 2024; Kerrisk 2026b). Hakemiston oikeudet asetettiin tasolle ```775```, jotta omistajalla ja ryhmällä on täydet luku-, kirjoitus- ja suoritusoikeudet, kun taas muilla käyttäjillä on vain luku- ja suoritusoikeudet sivujen näyttämistä varten (Kerrisk 2026a).

```console
sudo chown -R s200963:www-data /var/www/html
sudo chmod -R 775 /var/www/html
```

Oikeuksien muutos varmistettiin komennolla ```ls -ld /var/www/html```, jonka tuloste (```drwxrwxr-x```) vahvisti tavallisen käyttäjän omistajuuden.

Oikeusmuutosten jälkeen etusivun tiedosto (```/var/www/html/index.html```) avattiin muokattavaksi tavallisena käyttäjänä ilman sudo-komentoa: ```nano /var/www/html/index.html```

Tiedoston oletussisältö korvattiin seuraavalla HTML-rakenteella:

```html
<!DOCTYPE html>
<html lang="fi">
<head>
    <meta charset="UTF-8">
    <title>Apache-testisivu</title>
</head>
<body>
    <h1>Apache2 toimii!</h1>
    <p>Tätä etusivua on muokattu tavallisena käyttäjänä ilman sudo-oikeuksia.</p>
</body>
</html>
```

Alempana kuva apachen muokatusta verkosivusta:

<img width="2537" height="584" alt="kuva" src="https://github.com/user-attachments/assets/3e82cd29-bb1f-4833-9222-7731e944b017" />

Kuva 3. Apachen muokattu verkkosivu.

# b) Moottorix. Asenna Nginx käsin.

Koska sekä Apache2 että Nginx käyttävät oletusarvoisesti samaa HTTP-porttia 80, aiemmin asennettu Apache2-palvelu pysäytettiin ja sen automaattinen käynnistyminen estettiin ennen Nginxin käyttöönottoa porttiristiriitojen välttämiseksi komennoilla (F5, Inc. 2026; The Apache Software Foundation 2026):

```console
sudo systemctl stop apache2
sudo systemctl disable apache2
```

Palvelun pysähtyminen varmistettiin komennolla ```systemctl status apache2```, jonka tuloste osoitti palvelun olevan tilassa ```inactive (dead)```, kuva alla:

<img width="1458" height="384" alt="kuva" src="https://github.com/user-attachments/assets/a1dd25d6-ce6c-40b6-ba69-03fe7c77c188" />

Kuva 4. Apache disabloituna.

Seuraavkasi asennettiin `nginx`-ohjelmistopaketti ja palvelu määritettiin käynnistymään automaattisesti järjestelmän käynnistyessä (F5, Inc. 2026):

```console
sudo apt install nginx -y
sudo systemctl enable --now nginx
```
Asennuksen onnistuminen todennettiin komennolla `systemctl status nginx`, joka vahvisti palvelun olevan aktiivinen (`active (running)`). Alempana kuva:

<img width="1469" height="421" alt="kuva" src="https://github.com/user-attachments/assets/5fa6d580-2344-410c-bc57-7cf79fa75700" />

Kuva 5. Nginx aktiivisena.

Debian-pohjaisissa järjestelmissä Nginx käyttää oletusarvoisesti samaa `/var/www/html`-juurihakemistoa kuin Apache2, ja sen asetuksissa (`index`-direktiivi) `index.html` on etusijalla ennen asennuksen luomaa `index.nginx-debian.html`-oletustiedostoa (F5, Inc. 2026). Koska hakemiston ja `index.html`-tiedoston muokkausoikeudet oli siirretty tavalliselle käyttäjälle (`s200963`) jo a-kohdassa, ylimääräinen `index.nginx-debian.html`-tiedosto poistettiin suoraan ilman `sudo`-komentoa:

```console
rm -f /var/www/html/index.nginx-debian.html
```

Tämän jälkeen etusivun tiedosto (`/var/www/html/index.html`) avattiin muokattavaksi tavallisena käyttäjänä ilman korotettuja oikeuksia: `nano /var/www/html/index.html`

Tiedostoon tallennettiin seuraava HTML-sisältö:

```html
<!DOCTYPE html>
<html lang="fi">
<head>
    <meta charset="UTF-8">
    <title>Moottorix - Nginx</title>
</head>
<body>
    <h1>Nginx toimii!</h1>
    <p>Tätä etusivua on muokattu tavallisena käyttäjänä ilman sudo-oikeuksia.</p>
</body>
</html>
```
Lopuksi palvelimen toiminta testattiin hakemalla etusivu komentoriviltä komennolla `curl -i http://localhost`

Alempana kuva nginxin muokatusta verkosivusta:

<img width="1446" height="708" alt="kuva" src="https://github.com/user-attachments/assets/301a9691-da3c-4ee9-82bd-55e21f837f5d" />

Kuva 6. Muokattu Nginx-sivu.

# c) Automoottorix. Automatisoi Nginx asennus Ansiblella.

Tehtävässä automatisoitiin Nginx-verkkopalvelimen ylläpitäjän toimenpiteet Ansible-työkalulla kurssimateriaalin ohjeiden mukaisesti (Karvinen 2026). Konfiguraatio jäsenneltiin Ansiblen suositusten mukaisesti käyttämällä `site.yml` tiedostoa, sekä erillistä `nginx`-roolia (`roles/nginx/tasks/main.yml`) (Red Hat, Inc. 2026).

Ensin luotiin tarvittava hakemistorakenne `~/ansible`-työhakemiston alle:

```console
mkdir -p ~/ansible/roles/nginx/tasks
```

<img width="1143" height="803" alt="kuva" src="https://github.com/user-attachments/assets/a8621bca-28da-4cf3-b357-9bbc022f1cc5" />

Kuva 7. Kansiorakenne nyt ansiblessa.

Roolin tehtävätiedostoon (`roles/nginx/tasks/main.yml`) määriteltiin osuus: `nginx`-paketin asennus, mahdollisen Debianin oletussivun poisto, `/var/www/html`-julkaisuhakemiston omistajuuden ja oikeuksien (`0775`) asettaminen tavalliselle käyttäjälle (`s200963`) sekä palvelun käynnistäminen ja automaattisen käynnistyksen varmistaminen (Red Hat, Inc. 2026):

```yaml
- name: Asenna nginx-paketti
  apt:
    name: nginx
    state: present

- name: Poista nginx oletussivu
  file:
    path: /var/www/html/index.nginx-debian.html
    state: absent

- name: Aseta julkaisuhakemiston oikeudet tavalliselle käyttäjälle
  file:
    path: /var/www/html
    state: directory
    owner: s200963
    group: www-data
    mode: '0775'
    recurse: yes

- name: Varmista että nginx on käynnissä ja käytössä
  service:
    name: nginx
    state: started
    enabled: yes
```

Tämän jälkeen nginx-rooli piti vielä lisätä ```site.yml```-tiedostoon.

Playbook suoritettiin komennolla `ansible-playbook site.yml`. Tulos alla:

<img width="1561" height="1122" alt="kuva" src="https://github.com/user-attachments/assets/1214c74a-9348-4cae-a86b-122d86eab328" />

Kuva 8. Ansbilen tulos.

Ylläpitäjän osuuden jälkeen itse HTML-verkkosivu muokattiin käsin tavallisena käyttäjänä ilman korotettuja oikeuksia: `nano /var/www/html/index.html`

Tiedostoon tallennettiin seuraava HTML-sisältö:

```html
<!DOCTYPE html>
<html lang="fi">
<head>
    <meta charset="UTF-8">
    <title>Automoottorix - Ansible Nginx</title>
</head>
<body>
    <h1>Nginx asennettu Ansiblella!</h1>
    <p>Ylläpitäjän osuus on automatisoitu Ansiblella.</p>
</body>
</html>
```

Lopuksi palvelimen toiminta ja päivitetty etusivu varmistettiin komentoriviltä komennolla `curl -i http://localhost`. Kuva alempana:

<img width="847" height="515" alt="kuva" src="https://github.com/user-attachments/assets/e6cdcec9-eaaa-428b-8b75-f269e2645ff8" />

Kuva 9. Curl komennon tulos.

## Lähteet
Debian Wiki 2024. Apache. Luettavissa: https://wiki.debian.org/Apache. Luettu: 17.11.2026.

F5, Inc. 2026. NGINX Documentation: Beginner's Guide. Luettavissa: https://nginx.org/en/docs/beginners_guide.html. Luettu: 17.11.2026.

Hacker News 2026. Keskusteluketju: The Active Directory Era Is Finally Over. Y Combinator. Luettavissa: https://news.ycombinator.com/item?id=50517815. Luettu: 17.11.2026.

Karvinen, T. 2026. Palvelinten hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 16.11.2026.

Kerrisk, M. 2026a. chmod(1) — Linux manual page. Luettavissa: https://man7.org/linux/man-pages/man1/chmod.1.html. Luettu: 17.11.2026.

Kerrisk, M. 2026b. chown(1) — Linux manual page. Luettavissa: https://man7.org/linux/man-pages/man1/chown.1.html. Luettu: 17.11.2026.

Lobsien, F. 2026. The Active Directory Era Is Finally Over. Infisical Blog. Luettavissa: https://infisical.com/blog/the-active-directory-era-is-finally-over. Luettu: 17.11.2026.

Red Hat, Inc. 2026. Ansible Documentation: Using Ansible playbooks. Luettavissa: https://docs.ansible.com/ansible/latest/playbook_guide/index.html. Luettu: 17.11.2026.

The Apache Software Foundation 2026. Apache HTTP Server Version 2.4 Documentation: Starting Apache. Luettavissa: https://httpd.apache.org/docs/2.4/invoking.html. Luettu: 17.11.2026.
