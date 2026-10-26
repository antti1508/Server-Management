# h0 Oikeasti netissä

## x) Lue ja tiivistä

Aloitin tehtävän lukemalla tehtävänannossa annetut artikkelit, minkä jälkeen tiivistin ne.

Tiiviste Niemi 2025: H4 Tehtävät, linkki: https://github.com/AONCODES/linux-course/blob/main/h4.md

* Tulimuurin saa asennettua komennolla: `sudo apt install ufw`. Tulimuurista saa avattua portteja komennolla, esimerkki: `sudo ufw allow 22/tcp` ja sen saa laitettua päälle komennolla: `sudo ufw enable`.
* Pilvipalveluita vuokrattaessa tulee ottaa huomioon palveluntarjoajan sijainti ja lait, joita heidän tulee noudattaa.
* Jotta Principle of Least Privilege (PoLP) toteutuu, tulisi palvelimella käyttää normaalia käyttäjää pääkäyttäjän sijaan, normaali käyttäjä tulee lisätä sudo-ryhmään ja varmistaa käyttäjälle pääsy palvelimelle.
* SSH estää avainten käytön, jos niiden oikeudet ovat liian avoimet. Hakemisto ~/.ssh ja sen avaintiedostot on suojattava muiden käyttäjien luku- ja kirjoitusoikeuksilta komannoilla `chmod 700 ~/.ssh` ja `chmod 600 ~/.ssh/*`.
* Weppipalvelimia on useita, ja esimerkiksi Apache-palvelin asennetaan komennolla `sudo apt install apache2`.

Tiiviste Partanen 2025: Harjoitus 5: Nimekäs: a) Nimi, linkki: https://partanennn.github.io/linux-servers/practices/fifth-week.html

* Namecheap tarjoaa verkkotunnuksia vuosipohjaisella rekisteröintimaksulla.
* Halvimmillaan verkkotunnuksen voi saada ensimmäiseksi vuodeksi tarjouskampanjasta alle eurolla.
* Verkkotunnus kohdistetaan omaan palvelimeen Namecheapin DNS-hallinnassa lisäämällä palvelimen julkinen IPv4-osoite A-tietueeseen (A record).


## a) Paikallista

Paikalliseen virtuaalikoneeseen tarvitaan ISO-levykuva, jonka avulla Debian-käyttöjärjestelmä asennetaan VirtualBox-alustalle. Uusin ISO-tiedosto ladataan Debianin virallisilta verkkosivuilta (https://www.debian.org/distrib). Koska VirtualBox on jo asennettu tietokoneelle, sen asennusta ei käsitellä tässä yhteydessä. Virtuaalikoneen luonti aloitetaan valitsemalla asennuslevykuva ja nimeämällä virtuaalikone. Tämän jälkeen koneelle määritellään suorituskykyasetukset, kuten keskusmuisti, tallennustila ja prosessoriytimien määrä.

<img width="633" height="689" alt="kuva" src="https://github.com/user-attachments/assets/e6348d13-9eba-4e18-8153-b890d90e8c07" />

Kuva 1. Debian 13 suorityskykyyn liittyvät asetukset

Seuraavaksi käynnistetään virtuaalikone ensimmäisen kerran ja asennetaan käyttöjärjestelmä. Valitsin asennustyypiksi *Graphical install*.

<img width="629" height="553" alt="kuva" src="https://github.com/user-attachments/assets/3ddf2ce4-a1aa-4ac1-bf99-a074fd30d0b9" />

Kuva 2. Asennusvalikko

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


## b) VPS

## c) Alkutoimet

## d) Weppipalvelin

## e) Vapaaehtoinen: Nimi

## Lähteet 
https://man.openbsd.org/ssh.1

Karvinen, T. 2026. Palvelinten-hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/

Karvinen, T. 2026. Palvelinten-hallinta. Luento. Haaga-Helia. 22.10.2026
