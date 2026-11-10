# h2 Voileipä

## x) Lue ja tiivistä

### Tiivistelmä: Karvinen 2026: Sudo without password (https://terokarvinen.com/passwordless-sudo/)

* Ansible vaatii usein orjakoneella pääkäyttäjän (root) oikeudet tehtävien suorittamiseen. Yksi suoraviivainen ratkaisu on sallia sudo-komennon ajo ilman salasanakyselyä.
* Luodaan uusi käyttäjä (esim. anantti), oma ryhmä (esim. sudoless) ja lisätään käyttäjä tähän ryhmään (sudo adduser anantti sudoless).
* Ennen sudoers-tiedostojen muokkausta avataan toiseen ikkunaan valmiiksi pääkäyttäjäistunto (sudo -i). Jos sudoers-tiedosto rikkoutuu, se voidaan korjata valmiiksi auki olevasta root-kuoresta, vaikka sudo lakkaisi tavalliselta käyttäjältä toimimasta.
* Toimivuus testataan kirjautumalla sisään uudelleen ryhmäjäsenyyden aktivoimiseksi ja ajamalla ```sudo -k```, sekä ```sudo echo "testi"```, jonka tulee tulostaa teksti ilman salasanaa.

### Tiivistelmä: Munroe 2006: xkcd 149: Sandwitch (https://xkcd.com/149/)

* Sarjakuvassa hahmo kieltäytyy toisen tavallisesta käskystä tehdä voileipä ("Make me a sandwich." -> "What? Make it yourself.").
* Kun pyyntö toistetaan pääkäyttäjän etuliitteellä "Sudo make me a sandwich", toinen tottelee välittömästi ("Okay.").
* Sarjakuva havainnollistaa Linuxin käyttöoikeusmallia, järjestelmä ei arvioi komennon mielekkyyttä, vaan suorittaa annetun käskyn kyselemättä, jos käyttäjällä on tarvittavat pääkäyttäjäoikeudet (sudo).

### Tiivistelmä: Karvinen 2026: Passwordless Sudo with Ansible (https://terokarvinen.com/passwordless-sudo-with-ansible/) 

* IaC-periaate "Manual before auto": toimenpide kannattaa testata kertaalleen käsin ennen automatisointia
* Rooli määritellään tiedostossa roles/käyttäjänimi/tasks/main.yml, joka suorittaa järjestyksessä: ryhmän luonnin (group), käyttäjän luonnin ja ryhmäjäsenyydet (user), SSH-avaimen viennin (authorized_key) ja sudoers-säännön tallennuksen (copy).
* Sudoers-sääntö kirjoitetaan suoraan copy-moduulilla tiedostoon /etc/sudoers.d/sudoless käyttäen parametria content: "%sudoless ALL = (ALL) NOPASSWD: ALL\n" ja oikeuksia mode: "0644".
* Ensimmäisellä ansiblen ajokerralla tarvitaan vielä salasanaa, koska salasanatonta sudoa ei vielä ole, tällöin playbook ajetaan kysymällä salasana: ```ansible-playbook site.yml --ask-become-password``` (tai -K).

### Tiivistelmä: Ansiblen sisäänrakennettu dokumentaatio (https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html)

**ansible-doc copy:**

Kopioi tiedoston paikalliselta hallintakoneelta tai etäkoneelta toiseen hakemistoon etäkoneella. Tiedostojen noutamiseen etäkoneelta paikalliskoneelle käytetään vastaavasti fetch-moduulia.

Esimerkki:

```yaml
- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'
```

**ansible-doc apt:**

Asentaa, päivittää ja poistaa ohjelmistopaketteja esimerkiksi Debian- ja Ubuntu-pohjaisilla koneilla

Esimerkki: 

```yaml
- name: Install apache httpd (state=present is optional)
  ansible.builtin.apt:
    name: apache2
    state: present
```

**ansible-doc file:**

Luo kansioita, tiedostoja ja linkkejä sekä säätää niiden oikeuksia (ei koske tiedostojen sisältöön).

Esimerkki:

```yaml
- name: Create a directory if it does not exist
  ansible.builtin.file:
    path: /etc/some_directory
    state: directory
    mode: '0755'
```

**ansible-doc user:**

Luo, muokkaa ja poistaa paikallisia käyttäjätilejä ja niiden ryhmäjäsenyyksiä.

Esimerkki:


```yaml
- name: Add the user 'james' with a bash shell, appending the group 'admins' and 'developers' to the user's groups
  ansible.builtin.user:
    name: james
    shell: /bin/bash
    groups: admins,developers
    append: yes
```

**ansible-doc authorized_key:**

Lisää tai poistaa käyttäjän SSH-julkisia avaimia ~/.ssh/authorized_keys -tiedostosta salasanatonta kirjautumista varten. 

Esimerkki:

```yaml
- name: Set authorized key taken from file
  ansible.builtin.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
```

## a) Sudoless

Aloitin tehtävän käynnistämällä Debian 13 virtuaalikoneeni, jossa ansible onkin asennettu jo aikaisemmista tehtävistä. Tässä kohdassa oli tarkoituksena luoda käyttäjä, luoda ryhmä sudoless ja lisätä käyttäjä sudoless ryhmään, käytin tässä Teron antamia komentoja:

```console
sudo adduser anantti
sudo groupadd sudoless
sudo adduser anantti sudoless
```
(Karvinen. 2026d)

Tämän jälkeen avasin uuden terminaali-ikkunan jossa pidin root sessiota auki komennolla ```sudo -i``` sen aikaa kun muutellaan sudoers.d kansiossa sudoless tiedostoa.

Sitten ajoin komennon ```sudo visudo /etc/sudoers.d/sudoless``` ja lisäsin tiedostoon onelinerin: %sudoless ALL = (ALL) NOPASSWD: ALL.

Sitten olikin testien aika, vaihdoin käyttäjän ananttiin ja join komennot ```sudo -k``` (sudo -k mitätöi käyttäjän välimuistiin tallennetut pääsyoikeudet) & ```sudo echo "Sudon testaus"```. (Karvinen. 2026d) Alla kuva terminaalista: 

<img width="771" height="135" alt="kuva" src="https://github.com/user-attachments/assets/268da94f-60af-4770-9e52-c58e0054e09a" />

Kuva 1. Sudon testaus.

## b) Antero

Seuraavaksi oli aika kokeilla salasanatonta sudo-käyttäjää automaattisesti Ansiblella. Aloitin tämän vaiheen luomalla kansiorakenteeseen uuden kansion anantti roles-osion alle. Tämän jälkeen loin tasks-hakemiston ja sinne ```main.yml```-tiedoston. Muokkasin ```main.yml```-tiedoston Teron ohjeiden mukaisesti seuraavanlaiseksi:

```yaml
- group:
    name: "sudoless"
    state: present
- user:
    name: "anantti"
    state: present
    groups: ["sudoless", "sudo", "adm"]
- authorized_key:
    user: "anantti"
    key: "oma ssh-avain"
- copy:
    dest: "/etc/sudoers.d/sudoless"
    content: "%sudoless ALL = (ALL) NOPASSWD: ALL\n"
    owner: "root"
    group: "root"
    mode: "0644"
```
(Karvinen. 2026c)

Tämän jälkeen lisäsin ```site.yml``` tiedostoon become: true:n jotta ansiblea ajetaan sudona, sekä lisäsin rooleihin anantti:n jotta ansible ajaa myös sen taskit:

```yaml
- hosts: all
  become: true
  roles:
    - hello
    - anantti
```

Sitten olikin aika ajaa ansible ensimmäisen kerran jotta salasanaton sudo-käyttäjä luodaan:

<img width="1728" height="670" alt="kuva" src="https://github.com/user-attachments/assets/59dc33b2-8c8c-45ef-a64d-22885a37df40" />

Kuva 2. Ensimmäinen ajo ansiblella

Toista ajoa varten korvataan localhost ```hosts.ini``` tiedostoon tekstillä anantti@localhost ja ajetaan ansible ilman -K parametriä. ```hosts.ini``` näyttää sitten seuraavanlaiselle:

```yaml
anantti@localhost
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

Alempana kuva toisesta ansiblen ajosta:

<img width="1897" height="658" alt="kuva" src="https://github.com/user-attachments/assets/2a39245f-14a5-438b-a2eb-917f8742695d" />

Kuva 3. Toinen ajo ansiblella uudella salasanattomalla sudo-käyttäjällä.

## c) Package

Seuraavaksi minun pitäisi asentaa 2 ohjelmaa ansiblella, aloitin luomalla uuden roolin ja taskin, sekä main.yml tiedoston. Tässä kohdassa käytin tiivistelmääni kohdasta ansible-doc apt apuna **(HUOM task tulee lisätä ```site.yml``` tiedostoon)**. ```Main.yml``` tiedostosta tuli seuraavanlaien:

```yaml
- name: "kahden paketin asennus"
  apt:
    - curl
    - tree
  state: present
```

Seuraavkasi kokeillaan ettei ohjelmat ole jo valmiiksi asennettu:

<img width="665" height="148" alt="kuva" src="https://github.com/user-attachments/assets/f1e41012-779e-403d-b930-5e3fc83a3284" />

Kuva 4. Testaus ettei ohjelmat ole järjestelmässä.

Seuraavaksi ajetaan ansible uudelleen ja katsotaan asentuuko ohjelmat curl & tree.

Alempana kuva tuloksesta:

<img width="1903" height="723" alt="kuva" src="https://github.com/user-attachments/assets/9e507bbb-99b2-4458-a65a-0421ae741db8" />

Kuva 5. Ansible asentaa.

Muutoksia ainakin tapahtui, seuraavaksi kokeillaan toimiiko curl & tree koneella. Alla kuva tuloksesta:

<img width="1897" height="268" alt="kuva" src="https://github.com/user-attachments/assets/869ea0e6-ea3a-425a-81a7-a64577947ff0" />

Kuva 6. curl ja tree komennot.

## d) File

Tässä tehtävässä oli tarkoitus luoda monirivinen tiedosto. Aloitin tehtävän luomalla uuden roolin ja tein sille tasks kansion ja ```main.yml``` tiedoston **(HUOM task tulee taas lisätä site.yml tiedostoon)**. Tässä taas ```main.yml``` sisältöineen:

```yaml
- name "Luodaan monirivinen tiedosto"
  copy:
  dest: /ect/secret.txt
  content: |
    Tämä on ansiblella
    luoto monirivinen
    tiedosto secret.txt
  owner: anantti
  group: sudoless
  mode: "0640"
```

Oikeudet 0640 symbolisessa muodossa ovat -rw-r----- (4 = luku, 2 = kirjoitus, 0 = ei oikeutta).
* Omistaja (anantti / rw-): Saa lukea ja muokata tiedostoa.
* Ryhmä (sudoless / r--): Saa vain lukea tiedoston.
* Muut (others / ---): Eivät pääse tiedostoon lainkaan käsiksi.

Ajetaan ansible-playbook ja katsotaan mitä tapahtuu. Saatiin taas muutos, kuva alla:

<img width="1899" height="774" alt="kuva" src="https://github.com/user-attachments/assets/22d282bb-329e-4f73-a0e5-d5b0052df346" />

Kuva 7. Ansiblen tulos.

Tämän jälkeen tarkastetaan toimiiko tiedosto ja oikeudet, käytin ensin komentoa ```cat /etc/secret.txt``` käyttäjällä s200963 jolloin saatiin vastaus:

<img width="634" height="79" alt="kuva" src="https://github.com/user-attachments/assets/bc856cd4-7751-4ce8-a93c-b1c02f4713df" />

Kuva 8. Oikeuksien testaus

Seuraavaksi kokeillaan käyttäjällä anantti jolle annoin oikeudet tiedostoon:

<img width="817" height="119" alt="kuva" src="https://github.com/user-attachments/assets/bb4c107c-8e05-4d14-becc-824ef209cd7e" />

Kuva 9. Oikeudet toimii

Kaikki näyttäisi toimivan juuri kuten pitääkin.

## e) Jotain muuta

Päätin tässä kohdassa toteuttaa Ansiblella tauon (pause) ja varmistaa käyttäjältä, haluaako tämä asentaa Curl- ja Tree-ohjelmat. Loin uuden roolin ja sille tasks-kansion sekä ```main.yml```-tiedoston. Lisäsin luodun roolin myös ```site.yml```-tiedostoon. Tiedosto ```roles/varmistus/tasks/main.yml``` näytti tältä:

```yaml
- name: "Varmistus ohjelmien asennuksesta"
  pause:
    prompt: "Haluatko varmasti asentaa Curl & Tree ohjelmat? Paina enter jatkaaksesi tai Ctrl+C peruaksesi"
```

Seuraavaksi ajetaan taas ansible ja katsotaan kysyykö se asennusta ennen halutaanko jatkaa. Alla kuva ansible-playbookin tulosteesta:

<img width="1896" height="864" alt="kuva" src="https://github.com/user-attachments/assets/aaf884fb-f032-47f7-bcce-77764364e570" />

Kuva 10. Asennuksen varmistus.

Kokeillaan vielä painaa Ctrl+C ennen asennusta ja varmistetaan että asennus keskeytyy:

<img width="1574" height="848" alt="kuva" src="https://github.com/user-attachments/assets/6073ad03-6237-485f-b0c4-c2ed19735d83" />

Kuva 11. Ansible playbook keskeytetty.

Asennus keskeytyi, mutta tulosteeseen jäi punainen virheilmoitus, mikä ei näyttänyt siistiltä. Aloin selvittää parempaa toteutustapaa ja huomasin, että vahvistuskyselyn voi toteuttaa lukemalla käyttäjän syötteen (register) ja lopettamalla suorituksen tarvittaessa hallitusti Ansiblen meta-moduulilla (Ansible s.a.). Muokkasin varmistustehtävää siten, että se kysyy erikseen lupaa asennukselle.

Muokkasin tiedoston ```roles/varmistus/tasks/main.yml``` seuraavanlaiseksi:

```yaml
- name: Pyydä vahvistus käyttäjältä
  pause:
    prompt: "Kirjoita 'y' jatkaaksesi asennusta, tai paina vain Enter peruaksesi"
  register: kayttajan_valinta

- name: Lopeta pelikirjan ajo siististi ilman virhettä
  meta: end_play
  when: kayttajan_valinta.user_input | lower != 'y'
```

Nyt playbookin lopetus on siisti, eikä virheilmoitusta tule näkyviin:

<img width="1895" height="628" alt="kuva" src="https://github.com/user-attachments/assets/74320e0c-5395-4352-8bad-92886db894cc" />

Kuva 12. Siisti lopetus.

Kokeillaan vielä että playbook ajaa läpi jos valitsee 'y':

<img width="1898" height="788" alt="kuva" src="https://github.com/user-attachments/assets/7dec7d6d-d1b0-4152-8f86-cb5458a921c7" />

Kuva 13. Playbookin ajo loppuun.

Kaikki näyttäisi toimivan nyt oikein ja ilman virheitä.

### Lähteet:
Ansible s.a. Index of all Modules in the Ansible.builtin Collection. Luettavissa: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html. Luettu 9.11.2026.

Karvinen, T. 2026a. Palvelinten hallinta. Luento 5.11.2026. Haaga-Helia ammattikorkeakoulu.

Karvinen, T. 2026b. Palvelinten hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu 9.11.2026.

Karvinen, T. 2026c. Passwordless Sudo with Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo-with-ansible/. Luettu 9.11.2026.

Karvinen, T. 2026d. Sudo without password. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu 9.11.2026.
