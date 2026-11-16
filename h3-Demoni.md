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

# a) Apassi. Asenna Apache 2 käsin.

# b) Moottorix. Asenna Nginx käsin.

# c) Automoottorix. Automatisoi Nginx asennus Ansiblella.

## Lähteet:
Karvinen, T. 2026. Palvelinten hallinta. Luettavissa: https://terokarvinen.com/palvelinten-hallinta/. Luettu 16.11.2026.
