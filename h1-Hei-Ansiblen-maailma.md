# h1 Hei Ansiblen maailma

# x) Lue ja tiivistä

## Tiivistelmä: Karvinen 2026: SSH public key - Login without password (https://terokarvinen.com/ssh-public-key-login-without-password/)

* SSH korvaa salasanakyselyn asymmetrisellä avainparilla. Julkinen avain (.pub) voidaan jakaa kohdekoneille vapaasti, kun taas salainen avain pidetään omalla koneella.
* ```sudo apt-get -y install ssh``` asentaa OpenSSH-palvelimen ja ```sudo systemctl enable --now ssh``` käynnistää taustapalvelun välittömästi sekä asettaa sen käynnistymään koneen käynnistyessä.
* ```ssh-keygen``` luo avainparin hallintakoneelle (~/.ssh/id_...). Olemassa olevaa avainta ei pidä ylikirjoittaa.
* ```ssh-copy-id käyttäjä@kohde``` vie julkisen avaimen kohdekoneen ~/.ssh/authorized_keys -tiedostoon, jolloin salasana syötetään viimeisen kerran.
* Yhteysongelmia selvitetään asiakaskoneella komennolla ```ssh -v localhost``` (verbose-tila) ja palvelinpäässä komennoilla ```sudo systemctl status ssh``` & ```sudo journalctl --follow```.

## Tiivistelmä Karvinen 2026: Hello Ansible (https://terokarvinen.com/hello-ansible/)

* Ansible on keskitetty konfiguraationhallintatyökalu (Infrastructure as Code), jolla kuvataan haluttu lopputila ja Ansible tekee vain tarpeelliset muutokset.
* ```sudo apt-get install ansible``` asentaa työkalun hallintakoneelle; kohdekoneille ei asenneta erillistä taustaohjelmaa, vaan niille riittää pelkkä SSH-palvelin ja Python.
* Yhteyden toimivuus kohdekoneisiin testataan ajamalla yksittäinen komento ```ansible all -a "uptime"```.
* ```site.yml``` & ```roles/hello/tasks/main.yml``` muodostavat kokonaisuuden, jossa playbook kytkee koneet rooleihin ja tehtävätiedosto määrittää suoritettavat toimenpiteet
* Määritykset suoritetaan komennolla ```ansible-playbook site.yml```, ja pääkäyttäjäoikeuksia (become: true) vaativissa tehtävissä sudo-salasana kysytään valitsimella ```--ask-become-pass```.

# a) Paikallista. Asenna Debian 13 paikallisesti (Jos raportoit tämän jo h0 yhteydessä, pelkkä linkki raporttiin riittää.

Linkki Debian 13 asennukseeni (h0): https://github.com/antti1508/Server-Management/blob/main/h0-Oikeasti-netiss%C3%A4.md

# b) Sshecrets

Aloitetaan tässä kohtaa harjoitukset käynnistämällä Debian 13 virtuaalikone. Koneen käynnistyttä ajoin komennot ```sudo apt update -y && sudo apt upgrade -y``` tämän jälkeen asensin SSH:n komennolla ```sudo apt install ssh -y```. Asennuksen jälkeen sain ilmoituksen onnistuneesta asennuksesta:

# c) Pubkey

# d) Hei Ansible.

# e) Vapaaehtoinen bonus

## Lähteet
Karvinen, T. Palvelinten hallinta https://terokarvinen.com/palvelinten-hallinta/
https://terokarvinen.com/ssh-public-key-login-without-password/
