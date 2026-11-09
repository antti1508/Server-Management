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


## b) Antero

## c) Package

## d) File

## e) Jotain muuta

### Lähteet:
Ansible s.a. Index of all Modules in the Ansible.builtin Collection. Luettavissa: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html. Luettu 9.11.2026.

Karvinen, T. 2026a. Palvelinten hallinta. Luento 5.11.2026. Haaga-Helia ammattikorkeakoulu.

Karvinen, T. 2026b. Palvelinten hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu 9.11.2026.

Karvinen, T. 2026c. Passwordless Sudo with Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo-with-ansible/. Luettu 9.11.2026.

Karvinen, T. 2026d. Sudo without password. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu 9.11.2026.
