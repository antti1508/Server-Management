# h4 Pizza Fantasia

## x) Lue ja tiivistä.

### Tiivistelmä: Karvinen 2023: Configuration Management of Distributed Systems over Unreliable and Hostile Networks, vaaditut kohdat eli s.112-117 (https://westminsterresearch.westminster.ac.uk/download/4cc417566aa9af60fe3826d690719e390abdb7a3c8672f3d51b1eb4ca75e7624/1427236/karvinen-2023-configuration-management-of-distributed-systems.pdf)

### 4.12.1 Size and Complexity of Some DSLs s. 112

- Karvinen analysoi Salt-työkalun (versio 2017.7.4 "Nitrogen") lähdekoodista generoitua dokumentaatiota. Saltin DSL sisältää 510 tilafunktiota (state functions), joiden dokumentaatio on yli 20 000 riviä ja yli 75 000 sanaa.
- Lukuun eivät sisälly ehtolauseet (if-else) tai silmukat (for). Salt toteuttaa nämä generoimalla YAML-koodia Jinja2-mallipohjilla (templates), mikä luo ylläpitäjälle jälleen uuden matalan tason abstraktiokerroksen ymmärrettäväksi.
- Puppet tarjoaa suoraan 113 funktiota. Sen kontrollirakenteet eroavat huomattavasti yleiskäyttöisistä kielistä (kuten C, Go tai Python): esimerkiksi toistuvia tehtäviä varten funktioiden sijaan käytetään omia erikoiskäsitteitä, kuten virtuaalisia resursseja (virtual resources).

### 4.12.2 Use of DSL Functions in Case Configuration s. 112–115

- Karvinen tutki kahta julkista tuotantotason Puppet-konfiguraatiota (Mozilla Release Engineering ja Yhdysvaltain hallinnon USGCB) selvittääkseen, mitä ominaisuuksia todellisuudessa käytetään.
- Mozillan manifesteissa yli 1 %:n osuuteen yltävät komennot kattoivat 87 % kaikesta käytöstä. Käytetyimmät funktiot olivat `file` (19 %), `package` (10 %), `exec` (6 %) ja `service` (3 %), kun taas kontrollirakenteista korostuivat `case` (20 %), `if` (3 %), `class` (3 %) sekä resurssiriippuvuuksien hallintaan käytetty `anchor` (2 %).
- USGCB-konfiguraatiossa toistui sama kaava: `augeas` (22,5 %, olemassa olevien tiedostojen muokkaus), `file` (16,7 %), `service` (14,7 %), `exec` (14,7 %) ja `package` (12,7 %).
- Vaikka työkaluissa on satoja funktioita, käytännön tarpeet katetaan muutamalla perusfunktiolla (taustaprosesseille `package-file-service` ja työasemaohjelmille `package-file`).

### 4.12.3.1 Dependencies Between Main Functions s. 115–117

- Luvussa tarkastellaan funktioiden riippuvuuksia järjestelmän kehittäjän näkökulmasta. Yleisimmät käyttötapaukset jakautuvat neljään ryhmään: daemon-asetukset (`package`, `file`, `service`), sovellusasetukset (`package`, `file`), käyttäjähallinta (`user`, `group`) ja tiedostojen käsittely (`file`, `directory`, `symlink`).
- Sisäisesti kaikki toiminnot rakentuvat vain kahden suoraan kohdekoneeseen vaikuttavan perusmekanismin varaan: tiedostojen hallinnan (`file`, `dir`, `symlink`) ja ulkoisten komentojen suorituksen (`exec`).
- Tiedostojen hallinta kannattaa toteuttaa suoraan alusta asti tarkistamalla olemassaolo ja sisältö (`if not exists`), koska asetukset ovat pääosin tekstitiedostoja. Ulkoisten komentojen (`exec`) idempotenssi varmistetaan joko komennon omalla luonteella (esim. `apt-get install`) tai tarkistamalla komennon tuottama lopputulos.
- Muut keskeiset resurssit (`package`, `service`, `user`, `group`) sekä korkeamman tason toiminnot (kuten `cron`) voidaan toteuttaa näiden kahden perusmekanismin avulla ilman monimutkaisia lisäabstraktioita.

## a) Räpylä.

Päätin asentaa uutena demonina vsftpd:n, joka on tiedostosiirtopalvelin. Kokeilin samalla myös, että vsftpd asennettiin onnistuneesti. Käytin seuraavanlaisia komentoja:

```command
sudo apt update && sudo apt install -y vsftpd
systemctl status vsftpd
sudo vsftpd -v
```

<img width="1467" height="555" alt="kuva" src="https://github.com/user-attachments/assets/408ae87b-b636-4777-85f1-4649a11fb89f" />

Kuva 1. vsftpd:n asennuksen tarkistus.

Lopuksi poistin vsftpd:n, jotta Ansible asentaa sen seuraavassa kohdassa. Käytin tähän komentoja: 

```command
sudo apt purge -y vsftpd
sudo rm -f /etc/vsftpd.conf
```

## b) Automaatti.

Aloitin luomalla kansiorakenteen komennolla (`pwd=/home/user/ansible`): `mkdir -p roles/vsftpd/{tasks,handlers,files}`.

Sitten loin `main.yml`-tiedoston `tasks`-kansioon komennolla `nano roles/vsftpd/tasks/main.yml` ja muokkasin tiedostoa niin, että se asentaa vsftpd:n ja tarkistaa, että se on käynnissä (Ansible Documentation, 2026).

Tiedostosta `main.yml` tuli seuraavanlainen:

```yaml
- name: Asenna vsftpd
  apt:
    name: vsftpd
    state: present
    update_cache: true

- name: Varmista että vsftpd on käynnissä
  service:
    name: vsftpd
    state: started
    enabled: true
```

Sitten lisäsin roolin vielä `site.yml`-tiedostoon, ja tämän jälkeen ajoin Ansiblen komennolla `ansible-playbook site.yml`. Alla on tulos ja kokeilu siitä, että vsftpd asentui ja on käynnissä:

<img width="630" height="265" alt="kuva" src="https://github.com/user-attachments/assets/c319aa8a-e465-4add-869a-87c69f4471c9" />

Kuva 2. Ansiblen ajo.

<img width="1812" height="526" alt="kuva" src="https://github.com/user-attachments/assets/c031ef81-dee6-465a-b15f-8cf7edce6212" />

Kuva 3. vsftpd-asennus onnistui.

## c) Asetus.

Aloitustilanne vsftpd:hen yhdistettäessä näyttää seuraavanlaiselta:

<img width="842" height="153" alt="kuva" src="https://github.com/user-attachments/assets/cbda9c16-6190-4bf6-bb37-89136fccc8bc" />

Kuva 4. vsftpd alussa.

Seuraavaksi kopioin konfiguraatiotiedoston Ansiblen alle komennolla `cp /etc/vsftpd.conf roles/vsftpd/files/vsftpd.conf`. Näin saadaan oletusasetukset tiedostoon, ettei tarvitse kaikkea kirjoittaa itse. Sitten muokkailin tiedostoa ohjeiden (Debian Manpages, 2026) avulla niin, että `ftpd_banner`-asetus muuttuu seuraavanlaiseksi: `ftpd_banner=Ansiblen hallitsema vsftpd.`

<img width="1462" height="219" alt="kuva" src="https://github.com/user-attachments/assets/1df8a799-7e5f-4676-8d91-3199a0704637" />

Kuva 5. conf-tiedoston muutos.

Sitten muokkasin `main.yml`-tiedostoa `vsftpd`-kansion alla siten, että muokkaamani conf-tiedosto asetetaan vsftpd:n oletusasetuspolkuun. Lisäksi loin handlerin, joka käynnistää vsftpd:n uudelleen (Ansible Documentation, 2026). Nyt `main.yml` näyttää tältä:

```yaml
- name: Asenna vsftpd
  apt:
    name: vsftpd
    state: present
    update_cache: true

- name: Kopioi vsftpd asetustiedosto
  copy:
    src: vsftpd.conf
    dest: /etc/vsftpd.conf
    owner: root
    group: root
    mode: "0644"
  notify: Käynnistä vsftpd uudelleen

- name: Varmista että vsftpd on käynnissä
  service:
    name: vsftpd
    state: started
    enabled: true
```

`handlers`-kansion `main.yml` näyttää seuraavanlaiselta:

```yaml
- name: Käynnistä vsftpd uudelleen
  service:
    name: vsftpd
    state: restarted
```

Sitten ajoin taas Ansiblen komennolla `ansible-playbook site.yml` ja sain muutoksia. Kokeilin vaihtuiko banneri komennolla `ftp localhost`, ja tässä on tulos:

<img width="870" height="454" alt="kuva" src="https://github.com/user-attachments/assets/b21f9a71-6668-4504-a94d-0e1731c4cc98" />

Kuva 6. Ansiblen ajo.

<img width="779" height="180" alt="kuva" src="https://github.com/user-attachments/assets/b1fbb013-ac36-465b-9f16-0a1c3607d994" />

Kuva 7. Muutokset tehty Ansiblella.

Tässä siis käytin Ansiblea vaihtamaan vsftpd:n bannerin. Käyttömahdollisuuksiahan on nyt monia, kun todettiin, että oletusarvoinen conf-tiedosto saatiin korvattua ja asetuksia muutettua Ansiblella.

## d) Paikka remonttiin.

Seuraavaksi rikoin ympäristön poistamalla `vsftpd`-demonin asetuksineen komennolla `sudo apt purge -y vsftpd` sekä varmistamalla komennolla `sudo rm -f /etc/vsftpd.conf`, että myös Ansiblen kopioima konfiguraatiotiedosto poistui. Tämän jälkeen varmistin komennoilla `systemctl status vsftpd` ja `ftp localhost`, ettei palvelua enää löytynyt järjestelmästä eikä yhteys toiminut:

<img width="1099" height="362" alt="kuva" src="https://github.com/user-attachments/assets/e55043e6-a61a-4f43-9b35-314274a83ffa" />

Kuva 8. Vsftpd poistettu ja yhteys estynyt.

Kun tilanne oli todettu rikkinäiseksi, ajoin Ansible-roolin uudestaan komennolla `ansible-playbook site.yml`. Ansible havaitsi puuttuvan paketin ja asetustiedoston, asensi `vsftpd`:n uudelleen, palautti muokatun `vsftpd.conf`-tiedoston sekä käynnisti demonin:

<img width="2524" height="493" alt="kuva" src="https://github.com/user-attachments/assets/be575b96-aff6-4099-ace4-cd58169725ac" />

Kuva 9. Ansiblen ajo palauttaa tilanteen.

Lopuksi testasin yhteyden uudestaan komennolla `ftp localhost` ja totesin, että palvelin oli jälleen käynnissä ja c-kohdassa asetettu oma `ftpd_banner` oli palautunut käyttöön:

<img width="770" height="277" alt="kuva" src="https://github.com/user-attachments/assets/e7c8369f-33fe-451f-a516-d61541ce4ee3" />

Kuva 10. Palvelin toimii jälleen mukautetuilla asetuksilla.

## e) Idempotentti.

Osoitin tilan idempotenssin ajamalla saman Ansible-playbookin uudestaan komennolla `ansible-playbook site.yml` tekemättä mitään muutoksia rooliin tai kohdejärjestelmään:

<img width="2531" height="983" alt="kuva" src="https://github.com/user-attachments/assets/e1f53e26-a459-4e6a-9b99-ecee7e838172" />

Kuva 11. Playbookin uudelleenajo tuottaa tuloksen changed=0.

Tulosteesta nähdään, että kaikki tehtävät palauttivat tilan `ok` ja muutosten määrä oli nolla (`changed=0`). Myöskään palvelun uudelleenkäynnistävää handleria ei suoritettu uudelleen, koska paketti oli jo asennettuna, asetustiedosto vastasi tavoitetilaa ja demoni oli käynnissä. Tämä todistaa toteutuksen olevan idempotentti.

## f) (Vain Pasilan ryhmä -3015)

Pari on ilmoitettu Moodleen.

## Lähteet
Ansible Documentation 2026. ansible.builtin collection. Luettavissa: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/

Debian Manpages 2026. vsftpd.conf(5) — config file for vsftpd. Luettavissa: https://manpages.debian.org/bookworm/vsftpd/vsftpd.conf.5.en.html

Karvinen, T. 2026a. Palvelinten hallinta. Luento 19.11.2026. Haaga-Helia ammattikorkeakoulu.

Karvinen, T. 2026b. Palvelinten hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu: 24.11.2026.
