# h2 Voileipä

# x) Lue ja tiivistä

## Tiivistelmä: Karvinen 2026: Sudo without password (https://terokarvinen.com/passwordless-sudo/)

* Ansible vaatii usein orjakoneella pääkäyttäjän (root) oikeudet tehtävien suorittamiseen. Yksi suoraviivainen ratkaisu on sallia sudo-komennon ajo ilman salasanakyselyä.
* Luodaan uusi käyttäjä (esim. anantti), oma ryhmä (esim. sudoless) ja lisätään käyttäjä tähän ryhmään (sudo adduser anantti sudoless).
* Ennen sudoers-tiedostojen muokkausta avataan toiseen ikkunaan valmiiksi pääkäyttäjäistunto (sudo -i). Jos sudoers-tiedosto rikkoutuu, se voidaan korjata valmiiksi auki olevasta root-kuoresta, vaikka sudo lakkaisi tavalliselta käyttäjältä toimimasta.
* 

## Tiivistelmä: Munroe 2006: xkcd 149: Sandwitch (https://xkcd.com/149/)

## Tiivistelmä: Karvinen 2026: Passwordless Sudo with Ansible (https://terokarvinen.com/passwordless-sudo-with-ansible/) 

## Tiivistelmä: Ansiblen sisäänrakennettu dokumentaatio (https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html)

# a) Sudoless

# b) Antero

# c) Package

# d) File

# e) Jotain muuta

## Lähteet
