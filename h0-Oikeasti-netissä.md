# h0 Oikeasti netissä

## x) Lue ja tiivistä

Aloitin tehtävän lukemalla tehtävänannossa annetut artikkelit, minkä jälkeen tiivistin ne.

Tiiviste Niemi 2025: H4 Tehtävät, linkki: https://github.com/AONCODES/linux-course/blob/main/h4.md

* Tulimuurista saa avattua portteja komennolla, esimerkki: `sudo ufw allow 22/tcp` ja sen saa laitettua päälle komennolla: `sudo ufw enable`.
* Pilvipalveluita vuokrattaessa tulee ottaa huomioon palveluntarjoajan sijainti ja lait, joita heidän tulee noudattaa.
* Jotta Principle of Least Privilege (PoLP) toteutuu, tulisi palvelimella käyttää normaalia käyttäjää pääkäyttäjän sijaan, normaali käyttäjä tulee lisätä sudo-ryhmään ja varmistaa käyttäjälle pääsy palvelimelle.
* SSH estää avainten käytön, jos niiden oikeudet ovat liian avoimet. Hakemisto ~/.ssh ja sen avaintiedostot on suojattava muiden käyttäjien luku- ja kirjoitusoikeuksilta komannoilla `chmod 700 ~/.ssh` ja `chmod 600 ~/.ssh/*`.
* Weppipalvelimia on useita, ja esimerkiksi Apache-palvelin asennetaan komennolla `sudo apt install apache2`.

Tiiviste Partanen 2025: Harjoitus 5: Nimekäs: a) Nimi, linkki: https://partanennn.github.io/linux-servers/practices/fifth-week.html

* Namecheap tarjoaa verkkotunnuksia vuosipohjaisella rekisteröintimaksulla.
* Halvimmillaan verkkotunnuksen voi saada ensimmäiseksi vuodeksi tarjouskampanjasta alle eurolla.
* Verkkotunnus kohdistetaan omaan palvelimeen Namecheapin DNS-hallinnassa lisäämällä palvelimen julkinen IPv4-osoite A-tietueeseen (A record).


## a) Paikallista

## b) VPS

## c) Alkutoimet

## d) Weppipalvelin

## e) Vapaaehtoinen: Nimi

## Lähteet 
https://man.openbsd.org/ssh.1
