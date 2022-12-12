# Miniprojekti

## Johdanto

Päätin lähteä rakentamaan moduulia, jonka avulla voidaan asentaa LAMP. LAMP tulee sanoista Linux, Apache, MySQL ja PHP/Perl/Python (Linux.fi). Tietokantaohjelmista 
PostrgreSQL on edelliseltä kurssilta tutumpi kuin MySQL ja ohjelmointikielistä Python oli tutuin, joten valitsin ne tähän. Inspiraatiota sain tehtävässä H4 moduulin kokeilussa testaamastani Santtu Hurrin moduulista. Halusin itsekin kokeilla tehdä moduulin, jonka käyttämiselle olisi selkeät ohjeet ja sen voisi kuka tahansa helposti niiden avulla ottaa käyttöönsä. 

Testaamista nopettaakseni käytin Vagrantia
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

Lisäämällä public_html kotihakemistoon, kotisivut päivittyvät niin että näkyviin tulee index of vagrant. 

## Skriptitiedosto

Nyt kun sain ensimmäisen salt-tilan rakennettua aloitin tekemään tiedostoa, jonka avulla moduulin asennus toimii. Tiedosto ladataan wgetin avulla ja ajamalla skripti moduuli hoitaa asennukset. Ihan ensin kokeilin tiedoston avulla pakettilistan päivitystä ja yksinkertaisia tekstejä. Tähän katsoin apuja Santtu Hurrin vastaavasta tiedostosta run.sh. Aluksi tiedosto näytti tältä:

![image](https://user-images.githubusercontent.com/113497086/206383803-dc670f80-8ef7-4087-9cbf-9d797445d073.png)

Käytin tähän kokeiluun tyhjää vagrantilla luotua linux-konetta. Siihen asensin wgetin `$ sudo apt-get install wget`, jonka jälkeen wgetilla hain tuon tekemäni tiedoston githubista `$ wget https://raw.githubusercontent.com/SariLinsen/LAMP/main/script.sh`. Se onnistui ja seuraavaksi ajoin tiedoston `$ bash script.sh`. Siinä määrittelemäni toiminnot suoritettiin ja Githubin LAMP-varasto kloonattiin. Kokeillakseni luomani apache-tilan toimivuutta, tein /srv/salt-kansion jonne sen kopioin ja asensin salt-minionin. Tämän jälkeen ajoin tilan `$ sudo salt-call --local state.apply apache`.

Tila ajettiin onnistuneesti. Apache asentui, asetustiedostojen userdir-aktivointi suoritettiin ja apache uudelleenkäynnistettiin. Asensin Tälle koneelle vielä curlin, jotta sain tarkistettua että localhostiin tulee näkyviin sinne asettamani hello world ja käyttäjän kotisivut tulivat näkyviin.

![image](https://user-images.githubusercontent.com/113497086/206386339-aec427e3-615e-4675-8bd9-7e3c2de4fa76.png)

Testasin vielä tekemällä public_html-kansioon index.html-tiedoston, johon kirjoitin "vagrantin kotisivut" ja se päivittyi. Nyt käyttäjä voi siis itse jatkaa omien kotisivujensa rakentamista tänne.

![image](https://user-images.githubusercontent.com/113497086/206387075-0c5ca986-6329-4443-bf79-af7504dbf5af.png)

Jatkoin script.sh -tiedoston muokkausta ja lisäilin siihen noita vaiheita, joita piti tehdä saadakseen apache-tila ajettavaksi. Lisäsin saltin asennuksen, /srv/salt-hakemiston luomisen ja sinne apachen kopioimisen. Loppuun laitoin myös tilan ajamisen. Ensimmäinen kokeilu epäonnistui kirjoitusvirheen takia, olin laittanut kopiointiin cp:n tilalle epähuomiossa cd. Virheilmoitus viimeisellä rivillä onneksi kertoi mitä komentoa ei tunnistettu. Kirjoitusvirheen korjauksen jälkeen tiedoston ajaminen onnistui, apache asentui ja localhostiin tuli hello world sekä kotisivuille index of -teksti. 

![image](https://user-images.githubusercontent.com/113497086/206393610-6c8ec056-972c-413a-bfdc-306cc7528257.png)

## Mariadb

Yritin ensin tehdä salt-tilan postgreSQL:n asennuksesta, mutta se osoittautui itselleni liian haastavaksi. Tein ensin ainoastaan paketin asennuksen hoitavan tilan. Sen ajaminen aiheutti alla olevan virheilmoituksen. Yritin korjata tiedostoa muutamienkin ohjeiden avulla, mutta niissä tuli aina lisää virheilmoituksia, joita en enää osannutkaan korjata yksin.

    vagrant@herra:/srv/salt/postgresql$ sudo salt-call --local state.apply postgresql
    [ERROR   ] Command 'apt-get' failed with return code: 100
    [ERROR   ] stdout: Get:1 https://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
    Hit:2 https://deb.debian.org/debian bullseye InRelease
    Get:3 https://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
    Get:4 https://deb.debian.org/debian bullseye-backports InRelease [49.0 kB]
    Reading package lists...
    [ERROR   ] stderr: E: Release file for https://deb.debian.org/debian/dists/bullseye-updates/InRelease is not valid yet (invalid for another 1h 19min 47s). Updates for this repository will not be applied.
    E: Release file for https://deb.debian.org/debian/dists/bullseye-backports/InRelease is not valid yet (invalid for another 1h 19min 46s). Updates for this repository will not be applied.
    [ERROR   ] retcode: 100
    [ERROR   ] An error was encountered while installing package(s): E: Release file for https://deb.debian.org/debian/dists/bullseye-updates/InRelease is not valid yet (invalid for another 1h 19min 47s). Updates for this repository will not be applied.
    E: Release file for https://deb.debian.org/debian/dists/bullseye-backports/InRelease is not valid yet (invalid for another 1h 19min 46s). Updates for this repository will not be applied.
    local:
    ----------
              ID: postgresql
        Function: pkg.installed
          Result: False
         Comment: An error was encountered while installing package(s): E: Release file for https://deb.debian.org/debian/dists/bullseye-updates/InRelease is not valid yet (invalid for another 1h 19min 47s). Updates for this repository will not be applied.
                  E: Release file for https://deb.debian.org/debian/dists/bullseye-backports/InRelease is not valid yet (invalid for another 1h 19min 46s). Updates for this repository will not be applied.
         Started: 01:02:23.745499
        Duration: 3072.886 ms
         Changes:

    Summary for local
    ------------
    Succeeded: 0
    Failed:    1
    ------------
    Total states run:     1
    Total run time:   3.073 s

Sain kuitenkin näin asennettua postgresql:n orjalle onnistuneesti. Asennuksessa näyttäisi tulevan samalla useampi paketti, joten ehkä se voisi aiheuttaa ongelmat ja tilassa pitäisi tehdä tarkempia määrityksiä onnistuneen asennuksen suorittamiseksi.

![image](https://user-images.githubusercontent.com/113497086/206427049-31015d28-4d4c-4d9c-8dee-fb425317ad53.png)

Näiden ongelmien ja ajan rajallisuuden takia päätin vaihtaa tietokantaohjelmaksi mariadb:n. Tein sille vain paketin asentavan tilan.

![image](https://user-images.githubusercontent.com/113497086/206429214-5c48dabf-0741-489d-accf-59b9cbfc442f.png)

Lisäsin sen kopioinnin script.sh-tiedostoon ja tilan ajamisen.

    #!/bin/bash

    echo "Päivitetään pakettilistaus ajantasalle, asennetaan git ja salt.."
    sudo apt-get update
    sudo apt-get -y install git salt-minion

    echo "Luodaan kansio kotisivujasi varten.."
    mkdir public_html

    echo "Haetaan tiedostot Githubista.."
    git clone https://github.com/SariLinsen/LAMP.git
    cd LAMP/

    echo "Luodaan kansio /srv/salt/ ja kopioidaan tiedostot.."
    sudo mkdir /srv/salt/
    sudo cp -R apache /srv/salt/
    sudo cp -R mariadb /srv/salt/

    sudo salt-call --local state.apply apache
    sudo salt-call --local state.apply mariadb

Tämän jälkeen tein taas uuden koneen vagrantilla ja testasin moduulin käyttöä. Mariadb asentui ja kokeilin jatkaa sen käyttämistä Tero Karvisen "Install MariaDB on Ubuntu 18.04 – Database Management System, the New MySQL" ohjeiden mukaisesti. Sain Mariadb:n toimimaan ja luotua sinne testisisältöä. 

![image](https://user-images.githubusercontent.com/113497086/206432344-18c77a22-73bd-4d3f-9d67-d5169dcee273.png)

Seuraavaksi aloin tehdä top.sls-tiedostoa, jotta yhdellä `sudo salt-call --local state.apply` komennolla saadaan ajettua kaikki top-tiedoston listassa olevat tilat. 

![image](https://user-images.githubusercontent.com/113497086/206439475-bde17504-d7cb-415c-b08d-398050931e24.png)

Poistin script.sh tiedostosta  erilliset apache ja mariadb -tilojen ajamiset ja korvasin sen `sudo salt-call --local state.apply`-rivillä. Tein taas uuden virtuaalikoneen vagrantilla ja testasin. Tilan ajamisessa tuli virhetilanne ja ilmoitus että top-tiedostoa ei löydy. Tajusin että en ollut lisännyt sitä /srv/salt/-kansioon kopioitavaksi, joten eihän salt sitä tietenkään löytänyt. Lisäsin script.sh tilojen kopioinnin yhteyteen `sudo cp top.sls /srv/salt/` -rivin, jonka jälkeen top.sls oli saltin käytössä ja tilan ajaminen onnistui seuraavassa testissä.

## Python

Tein ensin python3-paketin asentavan tilan ja testasin että se toimii. Seuraavaksi kirjoitin yksinkertaisen skriptin, joka tulostaa päivämäärän (Geeks for geeks). Muokkasin oikeuksia niin että se toimii kaikilla käyttäjillä kaikkialla hakemistossa. Tämän onnistuttua lisäsin sen pythonin tilaan. 

![image](https://user-images.githubusercontent.com/113497086/206997041-a037cda9-e69f-4b67-82bb-126ad542956f.png)


Tein uuden virtuaalikoneen ja testasin moduulia siinä. Kaikki asennukset meni läpi ongelmitta ja tekemäni skriptikin toimi kuten sen oli tarkoituskin. Python-paketin asennuksesta tuli ilmoitus että se on jo asennettu ja se onkin ilmeisesti näissä jo valmiina. Testasin myös että apachen asennukset edelleen toimivat ja mariadb oli myös asentunut. 

![image](https://user-images.githubusercontent.com/113497086/206995689-ba1fad5c-31c2-464f-88e1-e463533be40a.png)

Tehtävä on kesken ja päivittyy edetessään.

## Lähteet

Karvinen, T. 2021. Two Machine Virtual Network With Debian 11 Bullseye and Vagrant. Luettavissa: https://terokarvinen.com/2021/two-machine-virtual-network-with-debian-11-bullseye-and-vagrant/. Luettu: 6.12.2022.

Karvinen, T. 2018. Apache User Homepages Automatically – Salt Package-File-Service Example. Luettavissa: https://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example/?fromSearch=apache%20user%20homepages. Luettu: 6.12.2022.

Hurri, S. 2022. Lemphelper. Luettavissa: https://github.com/santtuhurri/lemphelper. Luettu: 8.12.2022.

Karvinen, T. 2016. PostgreSQL Install and One Table Database – SQL CRUD tutorial for Ubuntu. Luettavissa: https://terokarvinen.com/2016/postgresql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu/?fromSearch=postgresql. Luettu: 8.12.2022.

Karvinen, T. 2018. Install MariaDB on Ubuntu 18.04 – Database Management System, the New MySQL. Luettavissa: https://terokarvinen.com/2018/install-mariadb-on-ubuntu-18-04-database-management-system-the-new-mysql/?fromSearch=mariadb. Luettu: 8.12.2022.

Karvinen, T. 2018. Salt States – I Want My Computers Like This. Luettavissa: https://terokarvinen.com/2018/salt-states-i-want-my-computers-like-this/?fromSearch=salt%20states. Luettu: 8.12.2022.

Geeks for geeks 2022. Get current date using Python. Luettavissa: https://www.geeksforgeeks.org/get-current-date-using-python/. Luettu: 12.12.2022.
