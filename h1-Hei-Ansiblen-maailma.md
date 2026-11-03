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

Aloitetaan tässä kohtaa harjoitukset käynnistämällä Debian 13 virtuaalikone. Koneen käynnistyttä ajoin komennot ```sudo apt update -y && sudo apt upgrade -y``` tämän jälkeen asensin SSH:n komennolla ```sudo apt install ssh -y```, viimeisimmän komennon jälkeen sain ilmoituksen että SSH asennetaan Debian koneelleni: 

<img width="1071" height="347" alt="kuva" src="https://github.com/user-attachments/assets/90597379-2b7c-4666-8fb2-35e06c3fe8e6" />

Kuva 1. SSH:n asennus.

Seuraavaksi laitoin SSH demonin päälle komennolla ```sudo systemctl enable --now ssh```, tämän jälkeen käynnistin virtuaalikoneen uudelleen ja tarkistin SSH-demonin statuksen komennolla ```sudo systemctl status ssh``` (Karvinen, T. 2026d) :

<img width="1285" height="613" alt="kuva" src="https://github.com/user-attachments/assets/8ed8b353-540d-45af-8788-3183a0caf133" />

Kuva 2. SSH-demonin statuksen tarkastaminen.

Sitten on aika kokeilla SSH:n toimivuus lokaalisti, ensin vain salasanalla. Käytin tähän testiin komentoa ```ssh s200963@localhost```, alempana kuva testituloksesta:

<img width="1271" height="794" alt="kuva" src="https://github.com/user-attachments/assets/5ba0ab60-e1e0-4124-bec5-64d7e54657c5" />

Kuva 3. SSH:n toimivuus testattu lokaalisti.

# c) Pubkey

Tämä vaihe aloitetaan ssh-avaimien luomisella. Käytin komentoa ```ssh-keygen``` luodakseni uudet SSH-avaimet Debian virtuaalikoneelleni. Seuraavaksi kopion julisen avaimen lokaalille clientille, jolle pääsen jo kirjautumaan salasanalla. Käytin avaimen kopiointiin komentoa ```ssh-copy-id localhost``` (Karvinen, T. 2026d), tämän jälkeen pääte kysyi viimeisen kerran salasanaa osoitteeseen s200963@localhost. Salasanan syöttämisen jälkeen sain ilmoituksen, jossa kerrottiin että julkinen avain on lisätty kohteesta "/home/s200963/.ssh/id_ed25519.pub/" localhostiin. Pääte vielä kehottaa kokeilemaan kirjautumista, jotta nähdään että avain toimii oikein: 

<img width="1281" height="787" alt="kuva" src="https://github.com/user-attachments/assets/45bd6721-35b6-47da-891a-715eba23cb4c" />

Kuva 4. Ilmoitus avainten kopioinnista.

Seuraavaksi kokeillaan kirjautuminen localhostille ilman salasanaa komennolla ```ssh s200963@localhost```, jonka jälkeen voidaan todeta että kaikki näyttäisi toimivan. Alla kuva tuloksesta:

<img width="1261" height="776" alt="kuva" src="https://github.com/user-attachments/assets/ed050f0e-6bf1-475f-a161-efff14094d8b" />

Kuva 5. Testattu SSH-kirjautuminen ilman salasanaa.

# d) Hei Ansible.

Aloitetaan tehtävä asentamalla Ansible. Ajetaan komento ```sudo apt update && sudo apt install ansible -y```. Tämän jälkeen tarkistetaan, että asennus onnistui komennolla ```ansible --version```.

Seuraavaksi loin kansion nimeltä ansible komennolla: ```mkdir ansible```, jonka jälkeen lisäsin kansion sisällä hosts.ini tiedoston, käytin tässä tekstieditorina nanoa ja komentoa ```nano hosts.ini```. Hosts.ini-tiedoston luotuani lisäsin tiedostoon ainoan koneeni "localhost" ja tulostin hosts.ini tiedoston sisällön komennolla ```cat hosts.ini```, tämän jälkeen poistetaan vielä python-höpötys lisäämällä seuraavanlainen pätkä hosts.ini-tiedostoon (Karvinen, T. 2026a):
```bash
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```
Alla vielä käyttämäni komennot :

<img width="208" height="74" alt="kuva" src="https://github.com/user-attachments/assets/97016ee5-8dbd-4ff8-8db6-dc853eb56840" />

Kuva 6. Käytetyt komennot kansion ja hosts.ini-tiedoston luontiin.

Seuraavaksi kokeilin ansiblen ja hosts.ini tiedoston toimivuutta komennolla ```ansible all -a 'uptime' -i hosts.ini```: 

<img width="1283" height="201" alt="kuva" src="https://github.com/user-attachments/assets/accf3022-a924-47a6-ab86-5f767786c613" />

Kuva 7. Ansiblen testikomennon tulos.

Seuraavaksi luodaan ansiblelle pakollinen kansiorakenne ja ansible.cfg, site.yml & main.yml tiedostot. Kansioiden luomiseen käytin komentoa ```mkdir -p roles/hello/tasks/``` (Karvinen, T. 2026a). Alempana kuva hakemistopuusta:

<img width="395" height="323" alt="kuva" src="https://github.com/user-attachments/assets/7276b7a0-2591-4c4e-b1ea-7a1b9b994666" />

Kuva 8. Hakemistopuu.

Seuraavaksi aloitin komentojen lisäämisen tiedostoihin, tässä kohdassa seurasin Teron ohjeita artikkelista: https://terokarvinen.com/hello-ansible/

## Tiedostojen sisällöt ovat nyt seuraavanlaiset:

## hosts.ini
```bash
localhost
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

## ansible.cfg
```bash
[defaults]
inventory = hosts.ini
```

## site.yml
```bash
- hosts: all
  roles:
    - hello
```

## ~/ansible/roles/hello/tasks/main.yml
```yaml
- copy:
    dest: /tmp/hello-ansible
    content: "Hei Ansiblen maailma"
```

Seuraavaksi kokeillaan ansiblen toimivuutta komennolla ```ansible-playbook site.yml```, komennon jälkeen ansible väittää tehneensä muutoksia, mutta tarkistetaan asia vielä. Alla kuva edellisen komennon tuloksesta: 

<img width="1168" height="359" alt="kuva" src="https://github.com/user-attachments/assets/b7ab339c-d0a1-45cb-8f05-bea62babcc72" />

Kuva 9. Ansible-playbookin tuloste.

Seuraavaksi tarkistetaan onko muutokset tehty localhostille. Tarkistetaan localhostin tmp kansio ja muutokset näyttäisi menneen läpi, alla kuva:

<img width="1274" height="561" alt="kuva" src="https://github.com/user-attachments/assets/6a02c9b0-977f-43fb-8439-f091cc921c2a" />

Kuva 10. Kokonaisuuden tulos.

## Lähteet
Karvinen, T. 2026a. Hello Ansible. Luettu 3.11.2026. https://terokarvinen.com/hello-ansible/

Karvinen, T. 2026b. Palvelinten hallinta. Luento 29.10.2026. Haaga-Helia ammattikorkeakoulu.

Karvinen, T. 2026c. Palvelinten hallinta. Kurssisivusto. Luettu 2.11.2026. https://terokarvinen.com/palvelinten-hallinta/

Karvinen, T. 2026d. SSH Public Key Login without Password. Luettu 2.11.2026. https://terokarvinen.com/ssh-public-key-login-without-password/
