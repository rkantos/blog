---
title: Linux palvelimet ICT4TN021 2017 H2
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---


# H2 29.8 Komennot & lokimerkinnät
========================


# Sisältö
{:.no_toc}

* Table of contents placeholder
{:toc}

### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Linux-kurssia. http://terokarvinen.com/2017/aikataulu-linux-palvelimet-ict4tn021-4-ti-ja-5-to-alkusyksy-2017-5-op

> a) Aiheuta lokiin kaksi eri tapahtumaa: yksi esimerkki onnistuneesta ja yksi esimerkki epäonnistuneesta tai kielletystä toimenpiteestä. Analysoi rivit yksityiskohtaisesti.
> 
> b) Vapaaehtoinen kohta, ei ole opetettu vielä: Asenna SSH-demoni. Kokeile omalla ssh-palvelimellasi jotain seuraavista: ssh-copy-id, sshfs, scp tai git. (Helpoin lienee scp: ‘scp foo.txt tero@example.com:’)
> 
> c) Tee unelmien apt-get -komento: yksi komentorivi, joka asentaa suosikkiohjelmasi.
> 
> d) Asenna komentokehotteen paketinhallinnasta kolme itsellesi uutta komentorivillä toimivaa ohjelmaa. Kokeile kutakin ohjelmaa sen pääasiallisessa käyttötarkoituksessa.
> 
> e) Opettele ulkoa ja harjoittele tärkeimmät komennot (tätä “opettele ulkoa” alakohtaa ei tarvitse raportoida):

# A) Lokitapahtumat | Fail2Ban

Päätin käyttää lokien analysointi-tehtävässä Fail2Ban -ohjelmaa. Fail2Ban on suuniteltu estämään verkkohyökkäyksiä perustuun erilaisiin triggereihin. Yksinkertaisimillaan Fail2Ban estää Linuxin palomuurin avulla (iptables) esimerkiksi liian monta kertaa SSH:n väärällä tunnuksella tai salasanalla yhdistäneen IP-osoitteen määrätyksi ajaksi. Näin voidaan tehokkaasti estää ja ainakin huomattavasti hidastaa hyökkäyksiä, joissa yritetään kokeilla mahdollisimman montaa käyttäjätunnusta tai salasanaa kirjautumiseen. 

#### Fail2Ban | Onnistunut uudelleenkäynnistys

Ubuntulle valmiina saatava Fail2Ban -paketti on hyvin helppo laittaa toimimaan SSH:n kanssa. Ohjelmapaketin mukana tulevassa ohjelmapaketissa on valmiina kaikki Fail2Banin SSH:n kanssa toimimiseen saattamiseksi vaadittavat konfiguroinnit. Fail2Ban lukee authlogia, josta se sitten määritellyn mukaisesti estää iptables:n avulla IP-osoitteen tai osoitteet.

Varmistin jo olemassa olevan Fail2Banin toimivuuden ensin käynnistämällä Fail2Ban -daemonin uudelleen komennolla **sudo service fail2ban restart**

Uudelleenkäynnistyksestä saamme lokiin (/var/syslog) seuraavat merkinnät:

Ensimmäisellä rivillä, joka on päivätty 4 Syyskuuta klo 02 systemd, eli käytössä olevan Debian 9:n palveluiden tai daemonien hallintaohjelma, kertoo että se on pysäyttämässä Fail2Ban -palvelua.

`Sep  4 02:01:56 d systemd[1]: Stopping Fail2Ban Service...`

Fail2Ban -palvelun asiakasohjelma Fail2Ban-client ilmoittaa sekuntia myöhemmin sammuneensa onnistuneesti.

`Sep  4 02:01:57 d fail2ban-client[27897]: Shutdown successful`

Systemd ilmoittaa suorittaneensa Fail2Ban -palvelun pysäytyksen onnistuneesti.

`Sep  4 02:01:57 d systemd[1]: Stopped Fail2Ban Service.`

Uudelleenkäynnistyksen seuraavassa vaiheessa Systemd kertoo seuraavaksi olevansa käynnistämässä Fail2Ban palvelua.

`Sep  4 02:01:57 d systemd[1]: Starting Fail2Ban Service...`

Fail2Ban-asiakasohjelma prosessinumerolla 27923 kertoo käynnistävänsä Fail2Ban palvelinta fail2ban.server -prosessinimellä sekä numerolla 27924.

`Sep  4 02:01:57 d fail2ban-client[27923]: 2017-09-04 02:01:57,477 fail2ban.server         [27924]: INFO    Starting Fail2ban v0.9.6`

Fail2Ban-asiakasohjelma kertoo Fail2Ban:in käynnistyvän daemon-tilassa.

`Sep  4 02:01:57 d fail2ban-client[27923]: 2017-09-04 02:01:57,477 fail2ban.server         [27924]: INFO    Starting in daemon mode`

Systemd kertoo lopuksi Fail2Ban -palvelun käynnistyneen onnistuneesti.

`Sep  4 02:01:57 d systemd[1]: Started Fail2Ban Service.`


#### Fail2Ban & IpTables| Onnistunut kirjautumisten estäminen 

Fail2Ban:in aktivoimiseksi kirjauduin omalle palvelimelleni viisi kertaa virheellisillä tunnuksilla. Tämän jälkeen Fail2Ban estää seuraavat kirjautumisyritykset kyseisestä IP-osoitteesta kymmenen minuutin ajaksi.

Ensinäkin näemme /var/auth.log:ilta seuraavan viisi kertaa:
```
Sep  4 02:21:43 d sshd[15977]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=85.76.7.100  user=x
Sep  4 02:21:45 d sshd[15945]: error: PAM: Authentication failure for x from 85.76.7.100
```

Virheilmoituksessa sshd -daemon prosessinumerolla 15977 kertoo klo 02:21 epäonnistuneesta autentikoinnista käyttäjälle x, IP-osoitteesta 85.76.7.100. Autentikointi virhe on alunperin Linuxin PAM:n, eli Linuxin käyttäjien tunnistusjärjestelmän raportoima. SSH-palvelu yrittää siis kirjata käyttäjän sisään verraten käyttäjätunnusta ja salasanaa PAM:sta löytyviin tunnuksiin. Väärät tunnukset syötettäessä SSHd -daemon siis estää yhteyden avaamisen ja siitä syntyy lokimerkintä.

Fail2Ban lukee jatkuvasti tätä /var/auth.log -tiedostoa, ja pystyy sen perusteella tekemään muutoksia iptables-palomuuriin, estäen logeissa esiintyvän IP-osoitteen. Fail2Banin logitiedostosta /var/log/fail2ban.log näemme onnistuneen eston lokimerkintänä:
```
2017-09-04 02:21:53,948 fail2ban.actions        [27926]: NOTICE  [sshd] Ban 85.76.7.126

```
Fail2Ban:in oma logi siis kertoo fail2ban.actions -prosessinimellä ja numerolla 27926 toimineen prosessin estäneensä 'Ban' sshd-konfiguraatiolla määrätyn IP-osoitteen 85.76.7.126.


# B) SSH-Daemon

#### SSH | Github | Jekyll

SSH-yhteyksien käyttäminen on itselle jo suhteellisen tuttua. Aikaisemmin käytin tiedostojen varmuuskopiointiin virtuaalipalvelinta ja käytin tarkoitukseen Baculaa, joka SSHFS:n avulla siirsi tiedostot. 

Kurssin yhteydessä ja tehtävien suositellun julkisen julkaisun vuoksi etsin itselle siihen tarkoitukseen parhaiten sopivan tavan. Mietin eri vahtoehtoja ja budjetti ja tietoturva edellä päädyin Githubin hostaukseen maksuttomuudesta ja luonnollisesta tietoturvasta johtuen. Maksuttomuus tulee ilmaisista repositoryjen ilmaisuudesta ja tietoturva vain HTML-sivujen julkaisun sallimisesta. Halusin kuitenkin käyttää jonkinlaista CMS-järjestelmää, ja ilman mahdollisuutta käyttää tietokantoja ja monimutkaisempaa ohjelmointia, päädyin Jekylliin. Tässä kohdassa siis näytän lyhyesti vain miten julkaisen sivut Githubiin Git -ohjelman avulla.


![alt text](/img/git-jekyll.gif "apt-get")


# C) Kaikenkattava komento ohjelmien asentamiseen

#### Ohjelmat |

oracle-java7/8/9-installer adobe-flashplugin python python-pip git ruby-dev build-essential

#### Palvelinten ja tietokoneiden ylläpitoon tarkoitetut ohjelmat |

xrdp htop openssh-server ssh bwm-ng aptitude dtrx mtr ipcalc

##### Tarvittavat lisäohjelmavarastot |
Kaikki ohjelmat eivät löydy Ubuntun tai Debianin oletusohjelmavarastoista, joten joudut lisäämään niitä varten kolmansien osapuolien ylläpitämät ohjelmavarastot:

#### Työpöytäohjelmat |

chrome chromium skype spotify firefox teamviewer icaclient libreoffice qownnotes flux vlc transmission mosh filezilla openvpn

##### Tarvittavat lisäohjelmavarastot |

Kaikki ohjelmat eivät löydy Ubuntun tai Debianin oletusohjelmavarastoista, joten joudut lisäämään niitä varten kolmansien osapuolien ylläpitämät ohjelmavarastot:

```
sudo add-apt-repository ppa:pbek/qownnotes
sudo add-apt-repository ppa:saiarcot895/chromium-dev
sudo add-apt-repository ppa:nathan-renniewaldock/flux
sudo add-apt-repository ppa:nilarimogard/webupd8 
sudo add-apt-repository ppa:webupd8team/java
```


Dropboxin ohjelmavaraston lisäämiseksi on tehtävä seuraavaa: https://www.dropbox.com/help/desktop-web/linux-repository

Dropboxin ohjelmavaraston lisääminen /etc/apt/sources.list -tiedostoon: deb http://linux.dropbox.com/ubuntu natty main
Sekä Dropboxin ohjelmavaraston avaimen lisääminen apt:hen:

```
echo deb http://linux.dropbox.com/ubuntu natty main | sudo tee /etc/apt/sources.list.d/dropbox.list

sudo apt-key adv --keyserver pgp.mit.edu --recv-keys 1C61A2656FB57B7E4DE0F4C1FC918B335044912E
```



#### GUI:n vaativat muut kuin työpöytäohjelmat |
grub-customizer winehq-devel remmina teamspeak gimp blueman steam(multiverse tarvitaan) cheese obs-studio

##### Tarvittavat lisäohjelmavarastot |
Kaikki ohjelmat eivät löydy Ubuntun tai Debianin oletusohjelmavarastoista, joten joudut lisäämään niitä varten kolmansien osapuolien ylläpitämät ohjelmavarastot:

```
sudo add-apt-repository ppa:danielrichter2007/grub-customizer
sudo add-apt-repository ppa:materieller/teamspeak3
sudo add-apt-repository ppa:obsproject/obs-studio
```

####  Koko komento |

Lisätään ensin tarvittavat ohjelmavarastot:

```
sudo add-apt-repository ppa:pbek/qownnotes ppa:saiarcot895/chromium-dev ppa:nathan-renniewaldock/flux ppa:nilarimogard/webupd8 ppa:webupd8team/java ppa:danielrichter2007/grub-customizer ppa:materieller/teamspeak3 ppa:obsproject/obs-studio
```

Asennetaan kaikki ohjelmat: 
```
sudo apt-get install oracle-java9-installer oracle-java8-installer adobe-flashplugin python python-pip git ruby-dev build-essential xrdp htop openssh-server ssh bwm-ng aptitude dtrx mtr ipcalc chrome chromium skype spotify firefox teamviewer icaclient libreoffice qownnotes flux vlc transmission mosh filezilla openvpn grub-customizer winehq-devel remmina teamspeak gimp blueman steam cheese obs-studio
```


# D) Komentorivillä toimivat ohjelmat [1]

Ubuntulla seuraavia ohjelmia varten ei tarvita erillisiä ohjelmavarastoja.

#### dtrx [2] | "Do The Right Extraction"

Sovellus on tarkoitettu useiden eri pakkasuformaattien purkamiseen ilman kaikkien ohjelmien komentojen muistamista erikseen. Ohjelman luojan mukaan ohjelma purkaa tar, zip, cpio, deb, rpm, gem, 7z, cab, lzh, rar, gz, bz2, lzma, xz -tiedostot sekä useita exe -tiedostoja. Ohjelma pystyy purkamaan myös Microsoftin cab -tiedostoja, InstallShield -arkistoja ja itsestään purkautuvia zippejä.

![dtrx](/img/dtrx.gif "apt-get")

#### mtr / mtr-tiny [3] | Traceroute & ping "2.0"

mtr on erityisesti verkkossa esiintyvien ongelmien, mutta myös reitityksien selvittämiseen tehty ohjelma. Ohjelma näyttää kätevästi kaiken sekä traceroute:lla että ping:llä (vasteaika ja pakettihäviö) saatavan tiedon samassa ikkunassa.

![mtr](/img/mtr.gif "apt-get")

#### ipcalc [4] | Verkko-osoitelaskuri

Ohjelman nimen mukaisesti ipcalc on tarkoitettu laskemaan verkko-osoitteita esimerkiksi aliverkon peitteen perusteella.


![ipcalc](/img/ipcalc.gif "apt-get")

### Lähteet |

[1]: https://kkovacs.eu/cool-but-obscure-unix-tools/
[2]: https://brettcsmith.org/2007/dtrx/
[3]: https://github.com/traviscross/mtr
[4]: https://github.com/nmav/ipcalc


[1: https://kkovacs.eu/cool-but-obscure-unix-tools/

[2: https://brettcsmith.org/2007/dtrx/

[3: https://github.com/traviscross/mtr

[4: https://github.com/nmav/ipcalc
