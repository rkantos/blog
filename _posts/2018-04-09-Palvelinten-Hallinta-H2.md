---
title: Palvelinten Hallinta ICT4TN022-4 2018 H2
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Palvelinten hallinta-kurssia. http://terokarvinen.com/2018/aikataulu-%E2%80%93-palvelinten-hallinta-ict4tn022-4-ti-5-ke-5-loppukevat-2018-5p

> a) Opiskele yllä aikataulussa olevat artikkelit. (Tätä lukutehtävää ei tarvitse raportoida). Luettava materiaali on kunkin tapaamiskerran kohdalla, esimerkiksi nyt “2. Package-file-server. [...] Luettavaa: Karvinen 2018: Pkg-File-Service – Control Daemons with Salt..”
> 
> b) Laita käyttäjien kotisivut toimimaan Apachella.
> 
> c) Laita PHP toimimaan käyttäjien kotisivuilla. (Huomaa, että PHP toimii oletuksena kaikkialla muualla kuin käyttäjien public_html-kotisivuilla.)
> 
> d) Rakenna tila (state), joka tekee Apachelle uuden nimipohjaisen virtuaalipalvelimen (name based virtual hosting). Voit simuloida nimipalvelun toimintaa hosts-tiedoston avulla.
> 
> e) Tee tila, joka laittaa esimerkkikotisivun uusille käyttäjille. Voit laittaa esimerkkikotisivu /etc/skel/:iin, niin se tulee automaattisesti ‘adduser tero’ komennolla käyttäjiä luodessa.
> 
> f) Eri asetukset. Tee Package-File-Service tilalla eri asetuksia kuin ne, mitä tehtiin tunnilla; ja eri kuin mitä teit/teet h2 muissa kohdissa. Voit muuttaa jotain toista asetusta samoista demoneista tai valita kokonaan eri demonit.

# B) Käyttäjien kotisivut Apachella Salt-tilan avulla


#### Käyttäjien kotisivut Apachella  |

Jotta käyttäjien kotisivut saadaan toimimaan alusta asti Salt-tilalla, täytyy ensiksi asentaa itse Apache. Apachen asennus Debianilla ja Ubuntulla tapahtuu komennolla `sudo apt-get install apache2` Käyttäjien kotisivut alkavat toimimaan tämän jälkeen komennolla `sudo a2enmod userdir`

Testataan ensiksi Apache2:n ja käyttäjäkohtaisten kotisivujen toiminta ajamalla edellä mainitut komennot ja tarkistamalla testikäyttäjän testi kotisivujen toimivuus kun public_html -kansio on luotu käyttäjälle.

Näyttäisi toimivan
![alt text](https://i.imgur.com/eGb421j.png)






#### Salt tilan luominen Package-File-Service rakenteella | [1]

Salt -tilaa varten tarvitaan Apachen asennus, joka onnistuu käyttämällä Salt-tilassa seuraavaa pkg-tilamoduulia:
```
apache2:
 pkg.installed
```


Koska Salt-tilat voi lajitella voidaan kätevästi lajitella eri kansioihin, on nyt aika tehdä Apache-tilalle oma kansio `/srv/salt/apache`. Luodaan kansioon init.sls -tiedosto, joka on ensimmäinen tiedosto minkä Salt	lukee ja suorittaa top.sls tiedoston lukemisen jälkeen tai suoritettaessa Apache-tilaa suoraan komennolla `sudo salt '*' apache` [2] 



#### Salt & Apachen kotihakemistoasetus  | [1]

Normaalisti Apache2:n käyttäjien kotihakemistot saadaan toimimaan edellä mainitulla `sudo a2enmod userdir` komennolla. Saltilla voidaan ajaa suoraan kyseinen komento, mutta tällöin konfiguraatio ei vielä ole idempotentti. Idempotentin tilan saavuttamista varten joudutaan Saltin pitää olla tietoinen tiedostosta missä konfiguraatiomuutokset otetaan käyttöön. Kuten Tero Karvinen blogissaan selvittää, tekee `a2enmod userdir` -komento `/etc/apache2/mods-available/userdir.conf` tiedostosta symbolisen linkin paikkaan `/etc/apache2/mods-enabled/userdir.conf`. Täten Saltin täytyy vähintään pystyä seuraamaan tuota mods-enabled -kansioon luotua tiedostoa, jotta se voi tulkita konfiguraatiomuutoksen olevan onnistunut. 

Koska nämä seikat on tiedossa Apache2:n userdir -komennosta, voidaan suoraan ohittaa komennon käyttö ja käsitellä `/mods-available/`  ja `/mods-enabled/`-kansioissa olevia konfiguraatiota. Näin syntyy luotettavasti toimiva Salt-tila.


Määritellään siis Tero Karvisen löytämien tiedostojen perusteella, jotka tekevät oikeat muutokset, `/etc/apache2/mods-available/userdir.conf` ja `/etc/apache2/mods-available/userdir.load` niin, että Salt tekee symboliset linkit Apachen `mods-enabled` -kansioon. Lopuksi laitetaan Salt käynnistämään Apache uudelleen kun näihin tiedostoihin tehdään muutoksia `mods-enabled` -kansiossa.

Kaikkien käyttäjien tulee edelleen luoda itselleen public_html -kansio erikseen.

```
/etc/apache2/mods-enabled/userdir.conf:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.conf

/etc/apache2/mods-enabled/userdir.load:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.load


apache2:
 service.running:
   - name: apache2
   - watch:
     - file: /etc/apache2/mods-enabled/userdir.conf
     - file: /etc/apache2/mods-enabled/userdir.load
```


#### Saltin valmis Apache2 -tila |
Lisätään /srv/salt/apache/init.sls tiedostoon siis:
```
apache2:
 pkg.installed

/etc/apache2/mods-enabled/userdir.conf:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.conf

/etc/apache2/mods-enabled/userdir.load:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.load


apache2:
 service.running:
   - name: apache2
   - watch:
     - file: /etc/apache2/mods-enabled/userdir.conf
     - file: /etc/apache2/mods-enabled/userdir.load
```

`/srv/salt/top.sls` tiedoston sisältö:

```
base:
   '*':
       - resolv-cloudflare
   's101':
       - apache
```


Toimii:
![alt text](https://i.imgur.com/ayHFDe2.png)


testi -käyttäjän kotihakemistokin näyttäisi edelleen olevan näkyvillä:

![alt text](https://i.imgur.com/eGb421j.png)

# C) Salt: PHP käyttäjien kotisivuille

Kun toimiva Apache -tila on luotu, voidaan ottaa käyttöön PHP käyttäjien omissa kotihakemistoissa. Luodaan uusi sls-tiedosto samaan apache-kansioon `/srv/salt/apache/php.sls` . Kuten Apachen kanssa tehtiin, asennetaan PHP samantapaisesti käyttämällä Saltin pkg-tilamoduulia [3]:
```
php:
 pkg.installed
   - pkgs:
     - php5
     - libapache2-mod-php5
```

PHP:n saattaminen toimintaan kotihakemistoissa onnistuu Ubuntussa kommentoimalla `/etc/apache2/mods-enabled/php5.conf` tiedoston lopusta viisi riviä:


```
<FilesMatch ".+\.ph(p[345]?|t|tml)$">
    SetHandler application/x-httpd-php
</FilesMatch>
<FilesMatch ".+\.phps$">
    SetHandler application/x-httpd-php-source
    # Deny access to raw php sources by default
    # To re-enable it's recommended to enable access to the files
    # only in specific virtual host or directory
    Require all denied
</FilesMatch>
# Deny access to files without filename (e.g. '.php')
<FilesMatch "^\.ph(p[345]?|t|tml|ps)$">
    Require all denied
</FilesMatch>

# Running PHP scripts in user directories is disabled by default
#
# To re-enable PHP in user directories comment the following lines
# (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it
# prevents .htaccess files from disabling it.
#<IfModule mod_userdir.c>
#    <Directory /home/*/public_html>
#        php_admin_flag engine Off
#    </Directory>
#</IfModule>
```

PHP toimii käyttäjän testi public_html kansiossa:
![alt text](https://i.imgur.com/bLVzSat.png)


Jotta Salt voi hallita konfigurointimuutosta voidaan käyttää jälleen `file.managed` funktiota. Luodaan ensin konfigurointitiedosto `/srv/apache/php/php5.conf`, johon laitetaan edellämainittu konfiguraatio niin, että PHP todellakin on käytössä käyttäjien kotihakemistoissa:

```
/etc/apache2/mods-available/php5.conf:
  file.managed:
    - source: salt://apache/php/php5.conf

/etc/apache2/mods-enabled/userdir.conf:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.conf

/etc/apache2/mods-enabled/userdir.load:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.load

```
Koska Ubuntun libapache2-mod-php huolehtii jo php-moduulin käyttöönotosta Apachessa, jää Saltin tehtäväksi vain hallita kyseistä konfiguraatiota. Voidaan kuitenkin varmuuden vuoksi seurata kumpaakin php:n käyttöönottoon liittyvistä tiedostoista. Huolitaan lopuksi kuitenkin jälleen siitä, että Apache käynnistetään uudelleen, kun konfiguraatioon on tehty muutoksia.

```
apache2:
 service.running:
   - name: apache2
   - watch:
     - file: /etc/apache2/mods-enabled/php5.conf
     - file: /etc/apache2/mods-enabled/php5.load
```


Koko Salt-tilan `/srv/salt/apache/php.sls` sisällöksi tulee siis: 

```
php:
 pkg.installed
   - pkgs:
     - php5
     - libapache2-mod-php5

/etc/apache2/mods-available/php5.conf:
  file.managed:
    - source: salt://apache/php/php5.conf

/etc/apache2/mods-enabled/userdir.conf:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.conf

/etc/apache2/mods-enabled/userdir.load:
 file.symlink:
   - target: /etc/apache2/mods-available/userdir.load

apache2:
 service.running:
   - name: apache2
   - watch:
     - file: /etc/apache2/mods-enabled/php5.conf
     - file: /etc/apache2/mods-enabled/php5.load


```

![salt state.highstate](https://i.imgur.com/fIuIWIG.png)


`/srv/salt/top.sls` tiedoston sisältö:

```
base:
   '*':
       - resolv-cloudflare
   's101':
       - apache
       - apache.php
```

Toimii edelleen vaikka komennon `sudo apt-get purge apache2 php5 libapache2-mod-php5` jälkeen:
![alt text](https://i.imgur.com/bLVzSat.png)


# D) Salt: Apache nimipohjainen virtuaalipalvelin 

Toimivan Apache-asennuksen päälle on helppo luoda uusia nimipohjaisia virtuaalipalvelimia `SALT.STATES.APACHE` -moduulin avulla.

#### SALT.STATES.APACHE  | [4]

Salt:n dokumentaatiosta löytyvä esimerkki sisältää oleelliset osat uuden nimipohjaisen virtuaalipalvelimen luontiin Apachelle. Luodaan siitä tälle palvelimelle sopiva konfiguraatio, jossa vastatataan kahden eri IP-osoitteen porttiin 80. Kummassakin osoitteessa ovat myös eri osoitteet s101.4e.fi ja s102.4e.fi. Laitetaan toinen niistä aliakseksi.

```
/etc/apache2/sites-available/s101.4e.fi.conf:
  apache.configfile:
    - config:
      - VirtualHost:
          this: '*:80'
          ServerName:
            - s101.4e.fi
          ServerAlias:
            - s102.4e.fi
          ErrorLog: /var/www/logs/s101.4e.fi-error_log
          CustomLog: /var/www/logs/s101.4e.fi-access_log combined
          DocumentRoot: /var/www/vhosts/s101.4e.fi
          Directory:
            this: /var/www/vhosts/s101.4e.fi
            Options:
              - Indexes
              - FollowSymlinks
            AllowOverride: All
```

Testataan ensin nimipohjaisen virtuaalipalvelimen toiminta  luomalla vhost-tiedosto `/etc/apache2/sites-enabled/s101.4e.fi.conf`:

```
<VirtualHost *:80>
		ServerName s101.4e.fi
		ServerAlias s102.4e.fi
		ErrorLog /home/testi/logs/s101.4e.fi-error_log
		CustomLog /home/testi/logs/s101.4e.fi-access_log combined
		DocumentRoot /home/testi/public_html/
	<Directory /home/testi/public_html/>
		Options Indexes FollowSymlinks
		AllowOverride All
	</Directory>
</VirtualHost>
```
Apachen uudelleenkäynnistyksen jälkeen palvelin näyttää vastaavan hyvin:

![s102.4e.fi test](https://i.imgur.com/St61oeN.png)

Luodaan uutta nimipohjaista virtuaalipalvelinta varten tiedosto `/srv/salt/apache/vhost.sls`

Luodaan myös konfiguraation mukainen logikansio `/home/testi/logs/` [5]

```
/home/testi/logs/:
  file.directory:
    - user: testi
    - group: testi
    - makedirs: True

/etc/apache2/sites-available/s101.4e.fi.conf:
  apache.configfile:
    - config:
      - VirtualHost:
          this: '*:80'
          ServerName:
            - s101.4e.fi
          ServerAlias:
            - s102.4e.fi
          ErrorLog: /home/testi/logs/s101.4e.fi-error_log
          CustomLog: /home/testi/logs/s101.4e.fi-access_log combined
          DocumentRoot: /home/testi/public_html/
          Directory:
            this: /home/testi/public_html/
            Options:
              - Indexes
              - FollowSymlinks
            AllowOverride: All

/etc/apache2/sites-enabled/s101.4e.fi.conf:
 file.symlink:
   - target: /etc/apache2/sites-available/s101.4e.fi.conf

apacherestart:
 service.running:
   - name: apache2
   - watch:
     - file: /etc/apache2/sites-enabled/s101.4e.fi.conf
```


`/srv/salt/top.sls` tiedoston sisältö:

```
base:
   '*':
       - resolv-cloudflare
   's101':
       - apache
       - php
       - apache.vhost
```

Testataan lopuksi `apache.vhost` -tila poistamalla apache ja sen konfigurointitiedostot `sudo apt-get purge apache2` ja suoritetaan tämän jälkeen `state.highstate`, `sudo salt 's101' state.highstate`

![sudo salt 's101' state.highstate](https://i.imgur.com/jQkneYE.png)

Toimiva s101.4e.fi: 
![s101.4e.fi test](https://i.imgur.com/yg0KFTt.png)




# E) Salt: Esimerkkikotisivu uusille käyttäjille /etc/skel

Luodaan Saltille tila `/srv/salt/apache/skelpublic.sls`, joka luo public_html kansion ja tiedostot index.html ja test.php samaan kansioon. Tähän voidaan käyttää Saltin `file.recurse`  funktiota, joka kopio kansion ja tiedostot rekursiivisesti [5]

Luodaan `/srv/salt/apache/var/public_html/index.html`:
```
Hello World
```

 ja `/srv/salt/apache/var/public_html/test.php` -tiedostot:

```
Hello World, Hello Hypertext Preprocessor. What's 3+3?

<?php
	$calc = 3 + 3;
   echo "it is $calc";
?>
```

Salt -tila `/srv/salt/apache/skelpublic.sls`:
```
/etc/skel/public_html:
  file.recurse:
    - source: salt://apache/var/public_html
    - include_empty: True
```

`/srv/salt/top.sls` tiedoston sisältö:

```
base:
   '*':
       - resolv-cloudflare
   's101':
       - apache
       - php
       - apache.vhost
       - apache.skelpublic
```

![sudo salt 's101' state.highstate](https://i.imgur.com/RsfFrOl.png)



Testaus voidaan tehdä luomalla toinen testikäyttäjä Saltilla. Luodaan tila `/srv/salt/testuser2.sls`

```
test2:
  user.present:
    - fullname: Test two
    - shell: /bin/bash
    - home: /home/test2
```
![sudo salt 's101' state.apply testuser2](https://i.imgur.com/W49GDHu.png)



Suoritetaan Saltilla pelkkä testuser2: `sudo salt 's101' state.apply testuser2`

![http://s102.4e.fi/~test2/](https://i.imgur.com/W49GDHu.png)

Kotisivut toimivat heti!

Myös PHP toimii!

![http://s102.4e.fi/~test2/test.php](https://i.imgur.com/zvZrke7.png)




# F) Salt: Nginx:n nimipohjaiset virtuaalipalvelimet

Nginxille ei löydy oletuksena samanlaisia moduuleita nimipohjaisten virtuaalipalvelimien luomiseksi, kuin Apachelle, joten seurataan täten normaalia package-file-service -kaavaa.



#### Nginx Package-File-Service | [7]

Nginx:in tiedostorakenne konfigurointitiedostoille on samankaltainen Apachen kanssa, vaikka itse konfigurointitiedostojen syntaksi onkin hieman erilainen. Tyypillisesti Nginxin virtuaalipalvelin konfiguraatiot sijaitsevat `/etc/nginx/sites-available/` ja `/etc/nginx/sites-enabled/` -kansiossa. Voidaan rakentaa siten hyvin samantyyppinen Salt-konfiguraatio, kuin Salt-Apachen kotihakemistojen käyttöönotossa. 


Luodaan ensimmäiseksi haluttavan nimipohjaisen virtuaalipalvelimen konfiguraatio `/srv/salt/nginx/conf/stest.4e.fi.vhost`:

```
server {
	listen 8080 default_server;
	listen [::]:8080 default_server;
	root /home/test2/public_html;
	index index.html index.htm index.nginx-debian.html;
	server_name stest.4e.fi;
	location / {
		try_files $uri $uri/ =404;
	}
}
```

Laitetaan Nginx porttiin 8080, koska olemassaoleva Apachen asennus pyörii normaalisti portissa 80. Poistetaan myös oletussivusto "default" käytöstä [6]:

```
/etc/nginx/sites-enabled/default:
  file.absent
```

Testataan ensimmäiseksi konfiguraatiota manuaalisesti laittamalla konfiguraatio suoraan `/etc/nginx/sites-enabled/stest.4e.fi.vhost` -tiedostoon:

Näyttää toimivan:

![http://stest.4e.fi/](https://i.imgur.com/6Q2r1Lp.png)


Luoda itse Salt-tila tiedostoon `/srv/salt/nginx/init.sls`


```
nginx:
  pkg.installed

/etc/nginx/sites-enabled/default:
  file.absent

/etc/nginx/sites-available/stest.4e.fi.vhost:
  file.managed:
    - source: salt://nginx/conf/stest.4e.fi.vhost

/etc/nginx/sites-enabled/stest.4e.fi.vhost:
 file.symlink:
   - target: /etc/nginx/sites-available/stest.4e.fi.vhost

nginxservice:
 service.running:
   - name: nginx
   - watch:
     - file: /etc/nginx/sites-enabled/stest.4e.fi.vhost
```

Testataan ensin pelkkä `sudo salt 's101' state.apply nginx`

![sudo salt 's101' state.apply nginx](https://i.imgur.com/qnIUAT2.png)

Ilmeisesti Salt ei voi luoda symbolisia linkkejä tiedostojen päälle. Poistetaan siis poistamaan aiemmin luotu konfiguraation `/etc/nginx/sites-enabled/stest.4e.fi.vhost`


Debian 8:lla on jokin erikoinen ongelma nginxin asennuksen kanssa. Nginx asentuu normaalisti käyttäen `sudo apt-get install nginx`, kuten myös Saltin tilalla, joka sisältää vain
```
nginx:
  pkg.installed
```

Kuitenkin Suoritettaessa koko nginx.init -tila ensimmäisen kerran saadaan virheilmoitus. 

![dpkg](https://i.imgur.com/4ySoT9s.png)


```
----------
          ID: nginxinstall
    Function: pkg.installed
      Result: False
     Comment: Problem encountered installing package(s). Additional info follows:
              
              errors:
                  - Running as unit run-26865.scope.
                    E: Sub-process /usr/bin/dpkg returned an error code (1)
     Started: 00:51:45.519293
    Duration: 11533.104 ms
     Changes:   
              ----------
              libgd3:
                  ----------
                  new:
                      2.1.0-5+deb8u11
                  old:
              libvpx1:
                  ----------
                  new:
                      1.3.0-3+deb8u1
                  old:
              libxpm4:
                  ----------
                  new:
                      1:3.5.12-0+deb8u1
                  old:
              libxslt1.1:
                  ----------
                  new:
                      1.1.28-2+deb8u3
                  old:
              nginx-common:
                  ----------
                  new:
                      1.6.2-5+deb8u5
                  old:
----------
```


Seuraavalla suorituskerralla virheilmoitusta ei kuitenkaan tule ja Salt raportoi Nginx:n asentuneen onnistuneesti.


`/srv/salt/top.sls` tiedoston sisältö:

```
base:
   '*':
       - resolv-cloudflare
   's101':
       - apache
       - php
       - apache.vhost
       - apache.skelpublic
       - nginx
```

Testataan lopuksi koko konfiguraatio. Poistetaan kaikki paketit ja konfiguraatiot `sudo apt purge -y nginx-full apache2 php5 nginx-common nginx && sudo apt-get autoremove -y && sudo rm -r /etc/nginx` Suoritetaan sen jälkeen `sudo salt 's101' state.highstate`

Huolimatta Salt:n virheilmoituksesta, kaikki toimii toisella suorituskerralla kuten pitää.

![sudo salt '*' state.highstate](https://i.imgur.com/7VLkjbY.png)

http://stest.4e.fi:8080/ toimii nyt:

![http://stest.4e.fi:8080/](https://i.imgur.com/6Q2r1Lp.png)

Lopuksi testataan vielä test.php toimivuus osoitteessa http://stest.4e.fi:8080/test.php 
Koska PHP:ta ei ole vielä laitettu toimimaan Nginxin kanssa, ei sivu vielä toimi ja tuloksena saamme vain test.php -tiedoston ladattua raakamuodossa.



Päivitetty 9.4.2018; https://github.com/rkantos/blog/blob/gh-pages/_posts/2018-04-04-Palvelinten-Hallinta-H2.md

### Lähteet |

[1]: http://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example
[2]:  https://docs.saltstack.com/en/latest/topics/best_practices.html
[3]:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.pkg.html#installation-of-packages-using-os-package-managers-such-as-yum-or-apt-get
[4]:  https://docs.saltstack.com/en/2017.7/ref/states/all/salt.states.apache.html
[5]:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html
[6]:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html#salt.states.file.absent
[7]:  https://www.digitalocean.com/community/tutorials/saltstack-infrastructure-creating-salt-states-for-nginx-web-servers
[8]: 
[9]: 
[10]: 
[11]: 
[12]: 
[13]: 
[14]: 
[15]: 

[1: http://terokarvinen.com/2018/apache-user-homepages-automatically-salt-package-file-service-example

[2:  https://docs.saltstack.com/en/latest/topics/best_practices.html

[3:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.pkg.html#installation-of-packages-using-os-package-managers-such-as-yum-or-apt-get

[4:  https://docs.saltstack.com/en/2017.7/ref/states/all/salt.states.apache.html

[5:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html

[6:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html#salt.states.file.absent

[7:  https://www.digitalocean.com/community/tutorials/saltstack-infrastructure-creating-salt-states-for-nginx-web-servers
