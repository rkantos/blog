---
title: Palvelinten Hallinta ICT4TN022-4 2018 H5
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Palvelinten hallinta-kurssia. http://terokarvinen.com/2018/aikataulu-%E2%80%93-palvelinten-hallinta-ict4tn022-4-ti-5-ke-5-loppukevat-2018-5p

> a) Valitse aihe omaksi kurssityöksi ja varaa se kommenttina aikataulusivun perään.
> 
> b) Julkaise raportti MarkDownilla. Jos käytät GitHub:ia, se tekee muotoilun automaattisesti “.md”-päätteisiin dokumentteihin.
> 
> c) Aja oma Salt-tila suoraa git-varastosta. Voit joko tehdä tilan alusta lähtien itse tai forkata sirottimen.


# A) Kurssityön aihe

Kurssilla esiteltävän moduulin aiheena viritän Salt:n asentamaan Softether/Openvpn sisäverkon, niin että Saltilla voisi ohjata uudet palvelimet yhdistämään heti jo luotuun verkkoon. Yritän lisäksi saada Baculan konfiguroitua niin, että se alkaa automaattisesti tekemään varmuuskopioita näistä palvelimista.

 
# B) Raportin julkaisu Markdownilla

Kaikki kotitehtävät on julkaistu Githubin gh-pages ominaisuutta ja Jekylliä apuna käyttäen ja näin myös Markdown on ollut kaikissa tehtävissä käytössä.


# C) Salt-tilan suorittaminen Git-varastosta

#### Git-varaston luominen | [1]

Tässä tehtävässä teemme koko versiohallinnan vain paikallisen Git-varaston avulla. Aloitetaan perustamalla itse paikallinen Git-varasto. Suoritetaan komento `mkdir localsalt && cd localsalt`. Suoritetaan git:in alustuskomento `git init.` Luodaan oletus README komennolla `echo "Tämä on pääasiassa paikallisesti suoritettavaa Salt-tilaa varten luotu Git-varasto" >> README `

#### Ensimmäinen Git-commit "Git-varastomerkinnän luonti" |

```
$ git add . # Komento lisää kaikki uudet tiedostot ja muutokset Git-varaston juuressa seuraavaksi tehtävää commit, eli varsinaista tallentamista varten
$ git commit -m "Ensimmäinen muutos, readme" #lisää tehdyt muutokset Git-varastoon uniikin tunnisteen kanssa 

```

Tyypillisesti näiden komentojen jälkeen Git -varasto työnnettäisiin johonkin päävarastoon, jossa usean henkilön yhteen projektiin luoma sisältö lisättäisiin. Tämä tapahtuisi käyttämällä komentoja `git remote add origin *git-palvelimen tiedot*` , `git push origin master` ja `git pull && git push` . Näiden komentojen jälkeen Git muistaa palvelimen konfiguraation ja uuden commitin jälkeen muutokset voi siirtää pääasiallisen Git-varaston palvelimelle vain käyttämällä komentoa `git push`.

Kuten aikaisemmin mainittu, tässä esimerkkitehtävässä käytetään kuitenkin vain paikallista Git-varastoa.

#### Saltin alustus Git-varastoon |

Luodaan Saltin vaatima perusrakenne. Luodaan `srv/salt/top.sls` -tiedosto seuraavalla sisällöllä:

```
base:
  '*':
    - programs
    - ssh-geoip
```


####  Esimerkkioletusohjelmien määritys Saltin asennettavaksi |

Luodaan `srv/salt/programs.sls` -tiedosto, jossa määritellään tavalliset paketit, jotka halutaan Saltin asentavan.

```
packages:
  pkg.installed:
    - pkgs:
      - dnsutils
      - htop
      - traceroute
```


#### SSH-yhteyksien salliminen maakohtaisesti |

Luodaan tämän tehtävän yhteydessä uusi Salt-tila pohjautuen erinomaiseen esimerkkiin [2]

Salt-tilaa varten tarvitsemme kahden paketin asennuksen ja kolmen konfigurointitiedoston luomisen.

Luodaan oma kansio Salt-tilaa varten localsalt Git-varastoon. 

```
$ mkdir ssh-geoip && cd ssh-geoip
```

Pääasiallinen konfigurointitiedosto `/usr/local/bin/sshfilter.sh` näyttää tältä:

```
#!/bin/bash

# UPPERCASE space-separated country codes to ACCEPT
ALLOW_COUNTRIES="FI" #Muutettu yhteyksien sallimiseksi Suomesta

if [ $# -ne 1 ]; then
  echo "Usage:  `basename $0` <ip>" 1>&2
  exit 0 # return true in case of config issue
fi

COUNTRY=`/usr/bin/geoiplookup $1 | awk -F ": " '{ print $2 }' | awk -F "," '{ print $1 }' | head -n 1`

[[ $COUNTRY = "IP Address not found" || $ALLOW_COUNTRIES =~ $COUNTRY ]] && RESPONSE="ALLOW" || RESPONSE="DENY"

if [ $RESPONSE = "ALLOW" ]
then
  exit 0
else
  logger "$RESPONSE sshd connection from $1 ($COUNTRY)"
  exit 1
fi
```

Tallennetaan konfigurointitiedosto Salt Git-varastoon tiedostoon `srv/salt/ssh-geoip/sshfilter.sh`


Koska kahden konfigurointitiedoston muutokset ovat hyvin yksinkertaisia, voimme käyttää Saltin `file.prepend` -toimintoa, joka tarkistaa tiedoston sisällön ja lisää rivin tiedoston alkuun vain jos sitä ei löydy ennestään. [3] hosts.deny ja hosts.allow -tiedostoihin lisättävien rivien sijainti on tärkeä, sillä se määrittää missä järjestyksessä sääntöjä noudatetaan. Tässä tapauksessa halutaan sääntöjen olevan ensimmäisenä "suodattamassa".

```
/etc/hosts.deny:
  file.prepend:
    - text: |
        sshd: ALL

#hosts.allow
/etc/hosts.allow:
  file.prepend:
    - text: |
       sshd: ALL: aclexec /usr/local/bin/sshfilter.sh %a'

```

Kootaan koko Salt -tila `srv/salt/ssh-geoip/init.sls` -tiedostoon. Koska ssh-geoip -konfiguraatio ei luota mihinkään palveluun, ei tässä ole tarvetta`watch.file` -kohdalle. Käytössä on siis vain ns. package-file -rakenne.

```
#vaaditut paketit
programs:
  pkg.installed:
    - pkgs:
      - ssh
      - geoip-bin
      - geoip-database

#pääkonfigurointitiedosto
/usr/local/bin/sshfilter.sh:
   file.managed
     - source: salt://ssh-geoip/sshfilter.sh

#hosts.deny
/etc/hosts.deny:
  file.prepend:
    - text: |
        sshd: ALL


#hosts.allow
/etc/hosts.allow:
  file.prepend:
    - text: |
       sshd: ALL: aclexec /usr/local/bin/sshfilter.sh %a'

```


Tässä kohtaa on hyvä myös tehdä uusi commit Git-varastoon `git add . && git commit -m "ssh-geoip perusrakenne"`


Luodaan localsalt Git-varastoon tarvittavat bash-skriptit, saltin ensimmäistä suorittuskertaa varten:


```
#!/bin/bash
#
# high.sh
#

#Varmistetaan että salt ja git on asennettu

sudo apt-get update
sudo apt-get install git salt-minion

#Koska Salt-tila asenetaan käyttäen vain paikallista Git-varastoa, tässä kohtaa ei tarvita seuraavaa komentoa:
#git clone https://github.com/**/**

#Määritellään Git:n perusasetukset
git config --global credential.helper "cache --timeout=3600"

#Suoritetaan Salt-tila paikallisesti
sudo salt-call --local --file-root srv/salt/ state.highstate --state-output filter -l warning
```

`--state-output filter -l warning` lähde: [4]

Annetaan high.sh bash-skriptitiedostolle suoritusoikeudet `chmod +x high.sh`

Tässä vaiheessa on jälleen hyvä suorittaa uusi commit Gittiin: `git add . && git commit -m "valmis Salt-git -tila"`

![alt text](/www/img/ "apt-get")

####  Salt-tilan testaus paikallisesta varastosta paikallisesti | 

Poistetaan kaikki salt tilan ja bash-skriptin asentamat tiedostot: `sudo apt-get purge geoip-bin geoip-database htop dnsutils traceroute git salt-minion`

Testataan Salt-tilan suorittaminen suorittamalla `high.sh` Bash-skripti.


![high.sh Bash-skriptin suoritus](https://i.imgur.com/rMaTfuY.png)
![high.sh Bash-skriptin suoritus](https://i.imgur.com/zmgUcDU.png)
![high.sh Bash-skriptin suoritus](https://i.imgur.com/gQDIHfT.png)

Lisätään varmuuden vuoksi Gittiin vielä viimeinen tila, jos jotain muutoksia on vielä tullut testaamisen jälkeen:

`git add . && git commit -m "valmis & testattu Salt-git -tila"`


### Lähteet |

[1]: http://terokarvinen.com/2012/git-from-offline-to-network
[2]: https://www.axllent.org/docs/view/ssh-geoip/
[3]:  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html#salt.states.file.prepend
[4]: https://docs.saltstack.com/en/latest/ref/output/all/salt.output.highstate.html
[5]: 
[6]: 
[7]: 


[1 : http://terokarvinen.com/2012/git-from-offline-to-network
[2 : https://www.axllent.org/docs/view/ssh-geoip/
[3 :  https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html#salt.states.file.prepend
[4 : https://docs.saltstack.com/en/latest/ref/output/all/salt.output.highstate.html

