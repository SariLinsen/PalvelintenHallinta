# Miniprojekti

## Johdanto

Päätin lähteä rakentamaan moduulia, jonka avulla voidaan asentaa LAMP. LAMP tulee sanoista Linux, Apache, MySQL ja PHP/Perl/Python (Linux.fi). Tietokantaohjelmista 
PostrgreSQL on edelliseltä kurssilta tutumpi kuin MySQL ja ohjelmointikielistä Python oli tutuin, joten valitsin ne tähän. Testaamista nopettaakseni käytin Vagrantia
virtuaalikoneiden luomiseen. Vagrant luo virtuaalikoneet ilman graafista käyttöliittymää, mutta se riittää testaukseen.

## Virtuaalikoneiden asennus ja yhdistäminen

Vagrantin olin jo asentanut aiemmin. Kahden koneen luomiseen käytin Tero Karvisen ohjetta "Two Machine Virtual Network With Debian 11 Bullseye and Vagrant".
Nimesin koneet herra ja orja, ja yhdistin ne Saltin avulla toisiinsa. Orjan voi helposti tuhota ja luoda uuden tyhjän koneen. Herrakoneeseen asensin salt-masterin ja 
orjalle salt-minionin, jonka jälkeen lisäsin orjan minion-asetustiedostoon herran osoitteen ja käynnistin minion-demonin uudestaan. Herrakoneella orja näkyi 
hyväksymättömien avaimien listassa, josta sen hyväksyin. Tarkistin vielä että orja vastasi ja yhteys toimii:

    vagrant@herra:~$ sudo salt '*' cmd.run 'hostname -I'
    orja:
        10.0.2.15 192.168.88.102
    
## Git

Seuraavaksi asensin herralle gitin ja kloonasin sille Githubiin tekemäni LAMP-varaston.

## Apache

Aloitin asentamalla ensin Apachen käsin. Tarkistin curlilla että Apachen default-sivu tulee näkyviin ja muokkasin siihen helloworldin. Laitoin myös käyttäjän kotivut 
toimimaan ja sinne tekstin homepages.

![image](https://user-images.githubusercontent.com/113497086/205917862-aaeaf26f-ce9a-48e1-9405-9ca9f72fd4ca.png)

Kun tämä onnistui, siirryin automatisointiin. Tein sudona hakemiston /srv/salt/apache, jonne init.sls tiedostoon asennettavan paketin nimen ja default-sivun päivityksen. Init.sls-tiedoston kirjoittamiseen käytin apuna Tero Karvisen ohjetta "Apache User Homepages Automatically – Salt Package-File-Service Example". Kokeilin ensin ajaa tilan paikallisesti herralle. Apache oli jo asennettuna ja /var/www/html/index.html päivittyi tilan mukaisesti. Testasin myös että localhostissa näkyi asettamani teksti.

![image](https://user-images.githubusercontent.com/113497086/205923106-860b0238-f317-4a5d-b980-5fe4e0a5829b.png)

Tämän jälkeen ajoin tilan orjalle, joka vastasi että apache asennettiin ja defaultsivu päivitettiin. Koitin curlilla sitä testata, mutta sitähän ei orjalle oltu asennettu. Asensin sen ensin, jonka jälkeen localhostin vastaus tuli näkyviin odotetunlaisesti.

![image](https://user-images.githubusercontent.com/113497086/205927373-6443cc45-be80-4f69-af09-6ad28c1e12e0.png)

Kopioin /var/www/html/ -kansiossa olevan index.html tiedoston saltiin apachen kansioon ja lisäsin sen polun init.sls. Kokeilin muokata sen sisältöä saltissa ja ajoin taas apache-tilan. Default-sivu päivittyi. Seuraavaksi aloin muokata init-tiedostoa niin että käyttäjän kotisivut saataisiin toimimaan. Sisältö on suoraan Tero Karvisen yllä mainitusta ohjeesta. Findilla hain apachen päivitetyt tiedostot ja kirjoitin ne tähän.

![image](https://user-images.githubusercontent.com/113497086/205941570-0ec9d830-dbd2-431a-9e9a-93802f770458.png)

Ennen tilan ajamista orjalle kokeilin että siellä localhost/~vagrant (orjan käyttäjänimi on myös vagrant) tuottaa virheilmoituksen 404 not found. Tilan ajamisen jälkeen virheilmoitus muuttui 403 forbidden. 

![image](https://user-images.githubusercontent.com/113497086/205943347-43e759c4-180c-4958-979c-54d8f963bd8b.png)

Tehtävä on kesken ja päivittyy edetessään.

## Lähteet

Karvinen, T. 2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/. Luettu: 6.12.2022.

Karvinen, T. 2018. Apache User Homepages Automatically – Salt Package-File-Service Example. Luettavissa: https://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example/?fromSearch=apache%20user%20homepages. Luettu: 6.12.2022.
