# h0 Oikeasti netissä

## x) Lue ja tiivistä

Aloitin tehtävän lukemalla tehtävänannossa annetut artikkelit, minkä jälkeen tiivistin ne.

Tiiviste Niemi 2025: H4 Tehtävät, linkki: https://github.com/AONCODES/linux-course/blob/main/h4.md

* Tulimuurin saa asennettua komennolla: `sudo apt install ufw`. Tulimuurista saa avattua portteja komennolla, esimerkki: `sudo ufw allow 22/tcp` ja sen saa laitettua päälle komennolla: `sudo ufw enable`.
* Pilvipalveluita vuokrattaessa tulee ottaa huomioon palveluntarjoajan sijainti ja lait, joita heidän tulee noudattaa.
* Jotta Principle of Least Privilege (PoLP) toteutuu, tulisi palvelimella käyttää normaalia käyttäjää pääkäyttäjän sijaan, normaali käyttäjä tulee lisätä sudo-ryhmään ja varmistaa käyttäjälle pääsy palvelimelle.
* SSH estää avainten käytön, jos niiden oikeudet ovat liian avoimet. Hakemisto ~/.ssh ja sen avaintiedostot on suojattava muiden käyttäjien luku- ja kirjoitusoikeuksilta komannoilla `chmod 700 ~/.ssh` ja `chmod 600 ~/.ssh/*`.
* Weppipalvelimia on useita, ja esimerkiksi Apache-palvelin asennetaan komennolla: `sudo apt install apache2`.

Tiiviste Partanen 2025: Harjoitus 5: Nimekäs: a) Nimi, linkki: https://partanennn.github.io/linux-servers/practices/fifth-week.html

* Namecheap tarjoaa verkkotunnuksia vuosipohjaisella maksulla.
* Halvimmillaan verkkotunnuksen voi saada ensimmäiseksi vuodeksi tarjouskampanjasta alle eurolla.
* Verkkotunnus kohdistetaan omaan palvelimeen Namecheapin DNS-hallinnassa lisäämällä palvelimen julkinen IPv4-osoite A-tietueeseen (A record).


## a) Paikallista

Paikalliseen virtuaalikoneeseen tarvitaan ISO-levykuva, jonka avulla Debian-käyttöjärjestelmä asennetaan VirtualBox-alustalle. Uusin ISO-tiedosto ladataan Debianin virallisilta verkkosivuilta (https://www.debian.org/distrib). Koska VirtualBox on jo asennettu tietokoneelle, sen asennusta ei käsitellä tässä yhteydessä. Virtuaalikoneen luonti aloitetaan valitsemalla asennuslevykuva ja nimeämällä virtuaalikone. Tämän jälkeen koneelle määritellään suorituskykyasetukset, kuten keskusmuisti, tallennustila ja prosessoriytimien määrä.

<img width="633" height="689" alt="kuva" src="https://github.com/user-attachments/assets/e6348d13-9eba-4e18-8153-b890d90e8c07" />

Kuva 1. Debian 13 suorityskykyyn liittyvät asetukset.

Seuraavaksi käynnistetään virtuaalikone ensimmäisen kerran ja asennetaan käyttöjärjestelmä. Valitsin asennustyypiksi *Graphical install*.

<img width="629" height="553" alt="kuva" src="https://github.com/user-attachments/assets/3ddf2ce4-a1aa-4ac1-bf99-a074fd30d0b9" />

Kuva 2. Asennusvalikko.

Käyttöjärjestelmän asennuksen aikana vastataan useisiin asennuskysymyksiin, joiden valinnat on tiivistetty alle:
* Käyttöjärjestelmän kieli: Englanti
* Näppäimistön kieli, aikavyöhyke ja sijainti: Suomi (no dead keys)
* Hostname: Debian
* Domain name: blank
* Uusi käyttäjä: Debian
* Asetetaan käyttäjien salasanat
* Levyjaon asetukset: Guided
* Package manager: deb.debian.org
* Software selection: Otetaan Debian desktop,  GNOME & standard system utilities (tarvittaessa voi lisätä weppipalvelimen ja ssh-palvelimen)

Muutaman latausikkunan jälkeen saadaan ilmoitus "Installation is complet, so it is time to boot into your new system" painetaan continue ja katsotaan miten käy. Kone käynnistyy uudelleen ja kirjautumisikkuna aukee, syötän tunnukset ja pääsin koneelle sisään.

<img width="1265" height="881" alt="kuva" src="https://github.com/user-attachments/assets/4429cfd8-4832-4b41-ba34-af979fedeb2b" />

Kuva 3. Debian virtuaalikoneen työpöytä asennuksen jälkeen.


## b) VPS

Teron luennolla avattiin eri pilvipalveluntarjoajista ja niiden eroista. Valitsin itse UpCloudin tätä kurssia varten sillä heiltä sai 30 päiväksi 500 euroa krediittejä ja niin sanotun prepaid maksumallin. Lisäsin omat tietoni ja maksutavan verkkosivulle, jonka jälkeen valitsin palvelinkoneen ja siihen käyttöjärjestelmän sekä tekniset tiedot. Tein nämä asiat luennolla 22.10.2026 kun meillä oli siihen sen verran aikaa että sain palvelimen hommattua ja kokeiltua sen toiminnan. Loin myös SSH-avaimet palvelimelle ja kokeilin niiden toiminnan. Alempana kuva palvelimen konfiguraatioista:

<img width="840" height="195" alt="kuva" src="https://github.com/user-attachments/assets/9e718fe2-adbc-49ce-9fec-50f7bc17b6b4" />

Kuva 4. UpCloudin palvelimen konfiguraatio

## c) Alkutoimet

Alkutoimet aloitin lataamalla tulimuurin, avaamalla portin 22 ja laittamalla tulimuurin päälle, käytin komentoja: `sudo apt install ufw`, `sudo ufw allow 22/tcp` & `sudo ufw enable`. Tämän jälkeen loin käyttäjän "antti" ja lisäsen sen sudo ryhmään, käytin komentoa `sudo adduser antti && sudo adduser antti sudo`. 

Seuraavaksi kopioin .ssh kansion käyttäjän antti .ssh hakemistoon ja lisäsin käyttäjälle oikeudet, käytin komentoja: `sudo cp -R .ssh /home/antti/.ssh`, `chmod 700 /home/antti/.ssh`, `chmod 600 /home/antti/.ssh/authorized_keys` & `chown -R antti:antti /home/antti/.ssh` 

Tämän jälkeen kokeilin, että lisätty käyttäjä toimii: `ssh antti@destination` ja lukitsin root-käyttäjän kirjautumisen salasanalla käyttäen komentoa: `sudo usermod --lock root` sekä disabloin ssh-kirjautumisen root käyttäjältä.

<img width="955" height="766" alt="kuva" src="https://github.com/user-attachments/assets/6d45b2f5-8ce8-4132-aae9-0873acd0805d" />

Kuva 5. Disabloitu ssh-kirjautuminen käyttäjältä root.

Jotta asetus tulee voimaan käynnistetään ssh-palvelu vielä uudelleen komennolla: `sudo service ssh restart`

## d) Weppipalvelin

Valitsin weppipalvelimeksini Apachen, asensin Apachen komennolla `sudo apt install apache2`. 

Ensimmäisellä yrityksellä sain virheilmoituksen "Error: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?", joten ajetaan komento `sudo apt update && sudo apt upgrade` ja kokeillaan uudelleen. 

Ajetaan nyt komento `sudo apt install apache2` -> onnistui.

Avataan weppisivua varten portti tulimuurista komennolla: `sudo ufw allow 80/tcp`

Tarkistetaan Apache:n versio komennolla: `apachectl -v`

Kokeillaan selaimella apachen toimivuus:

<img width="1268" height="976" alt="kuva" src="https://github.com/user-attachments/assets/d96781b6-8691-4a9c-bca2-f242769d6ceb" />

Kuva 6. Apachen oletussivu.

## e) Vapaaehtoinen: Nimi

Ostin domainin Namecheap-palvelun kautta, koska tarkoituksena on tehdä tämä sivu vielä harjoituksena niin ostin halvimman domainin joka oli saatavilla. Hintaa domainille tuli lopuksi 1.38€ / ensimmäinen vuosi.

<img width="1070" height="801" alt="kuva" src="https://github.com/user-attachments/assets/2b58ad72-0969-4aa3-ba60-2b6abd890dc8" />

Kuva 7. Namecheap ostoskori.

Lisäsin A-recordin domainille ja laiton siihen palvelimeni IP-osoittee, jossa weppipalvelin on.

<img width="1129" height="384" alt="kuva" src="https://github.com/user-attachments/assets/fd0949e5-6dbd-4690-9c9b-53e251868469" />

Kuva 8. DNS-asetukset.

Tämän jälkeen kaikki toimi ja verkkosivuun pääsi domain-nimellä.

## Lähteet 
Karvinen, T. 2017. First Steps on a New Virtual Private Server – An Example on DigitalOcean. Blogikirjoitus. Luettavissa: https://terokarvinen.com/2017/first-steps-on-a-new-virtual-private-server-an-example-on-digitalocean/. Luettu: 27.10.2026.   

Karvinen, T. 2026a. Palvelinten hallinta. Verkkosivusto / kurssisivu. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 27.10.2026.   

Karvinen, T. 2026b. Palvelinten hallinta. Luento. Haaga-Helia ammattikorkeakoulu. 22.10.2026.   

OpenBSD 2024. SSH(1) – OpenBSD General Commands Manual. Luettavissa: https://man.openbsd.org/ssh.1. Luettu: 27.10.2026.
