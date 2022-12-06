# Miniprojekti

## Johdanto

Päätin lähteä rakentamaan moduulia, jonka avulla voidaan asentaa LAMP. LAMP tulee sanoista Linux, Apache, MySQL ja PHP/Perl/Python (Linux.fi). Tietokantaohjelmista 
PostrgreSQL on edelliseltä kurssilta tutumpi kuin MySQL ja ohjelmointikielistä Python oli tutuin, joten valitsin ne tähän. Testaamista nopettaakseni käytin Vagrantia
virtuaalikoneiden luomiseen. Vagrant luo virtuaalikoneet ilman graafista käyttöliittymää, mutta se riittää testaukseen.

## Virtuaalikoneiden asennus ja yhdistäminen

Vagrantin olin jo asentanut aiemmin. Kahden koneen luomiseen käytin Tero Karvisen ohjetta: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/.
Nimesin koneet herra ja orja, ja yhdistin ne Saltin avulla toisiinsa. Orjan voi helposti tuhota ja luoda uuden tyhjän koneen. Herrakoneeseen asensin salt-masterin ja 
orjalle salt-minionin, jonka jälkeen lisäsin orjan minion-asetustiedostoon herran osoitteen ja käynnistin minion-demonin uudestaan. Herrakoneella orja näkyi 
hyväksymättömien avaimien listassa, josta sen hyväksyin. Tarkistin vielä että orja vastasi ja yhteys toimii:

    vagrant@herra:~$ sudo salt '*' cmd.run 'hostname -I'
    orja:
        10.0.2.15 192.168.88.102
    
## Git

Seuraavaksi asensin herralle gitin ja kloonasin sille Githubiin tekemäni LAMP-varaston.
