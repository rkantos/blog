---
title: Palvelinten Hallinta ICT4TN022-4 2018 H4
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Palvelinten hallinta-kurssia. http://terokarvinen.com/2018/aikataulu-%E2%80%93-palvelinten-hallinta-ict4tn022-4-ti-5-ke-5-loppukevat-2018-5p

> a) Opiskele yllä aikataulussa olevat artikkelit. Noissa artikkeleissa opetetaan ne asiat, joilla läksyt saa tehtyä. Tätä lukutehtävää ei tarvitse raportoida. Luettava materiaali on kunkin tapaamiskerran kohdalla.
> 
> b) Tee kahdella orjalla esimerkki, jossa orjat saavat eri muuttujan pilarista. Tarkista ‘pillars.items’, että kummalekin orjalle mene eri tieto. Tee jokin muu kuin tunnilla tehty sshd-esimerkki.
> 
> c) Tee kahdella orjalla esimerkki, jossa toinen orja saa muuttujan pilarista ja toinen käyttää oletusarvoa (pillar.get). Tee jokin muu kuin tunnilla tehty sshd-esimerkki.




# A) Kahden Salt-orjan muuttujat pilarista 

Tehdään yksinkertainen kahden orjan esimerkki, jossa tallennetaan kuvitteelliset palvelinkohtaiset MySQL-palvelimen salasanat kahdelle eri palvelimelle (minionille).

#### Saltin 'pillars' konfigurointi | [1]

Luodaan `/srv/pillar/top.sls` tiedosto Salt pilarien hallitsemiseksi ja lisätään seuraava sisältö:

```
$cd /srv/pillar/
$sudoedit	top.sls

base:
  's101':
    - mysqlconfs101
  's001':
    - mysqlconfs001
```

Luodaan mysqlconfs101.sls ja mysqlconfs001.sls tiedostot, johon laitetaan Mysql-palvelinten salasanat. 

Tiedostojen sisällöt:

```
$ tail -vn +1 mysqlconfs*
==> mysqlconfs001.sls <==
mysqlbackendpw: yiavRugBynUtDyRudCecbackeuGlyp

==> mysqlconfs101.sls <==
mysqlbackendpw: .OkyeushdebPerWantIfyecgietCac
```

#### Saltin jinja ja muuttujien konfigurointi | [2]

Tallennetaan MySQL -salasanat palvelimilla, eli minioneilla s101 ja s001 sijaintiin `/etc/mysql/mysqlsecret.conf`

Luodaan `/srv/salt/mysqlpw.sls` ja `/srv/salt/config/mysqlsecret.conf` tiedostot. Tallennetaan tiedostojen sisällöksi:



```
$ tail -vn +1 /srv/salt/mysqlpw.sls 
==> mysqlpw.sls <==
#mariadb-server:
#  pkg.installed

/etc/mysql/mysqlsecret.conf:
  file.managed:
    - source: salt://config/mysqlsecret.conf
    - template: jinja
    - context:
      mysqlpw: {{ pillar['mysqlbackendpw'] }}

#mariadb-server:
#  service.running:
#    - watch:
#      - file: /etc/mysql/mysqlsecret.conf


$ tail -vn +1 config/mysqlsecret.conf 
==> config/mysqlsecret.conf <==
Mysql PW: {{ mysqlpw }}
```

Koska Mysql-serveriä ei ole nyt tarkoitus asentaa, on nämä kohdat kommentoitu pois.



#### testaus & pillars.items |

Testataan Salt tilaa ja pilareita suorittamalla komento `sudo salt '*' 	mysqlpw`

![parent directory not present](https://i.imgur.com/OiWwU1B.png)

Saamme yhden virheilmoituksen toiselta minionilta puuttuvasta `/etc/mysql` -kansiosta. Oletettavasti näin tapahtuu koska toiselle minionille ei ole asennettu Mysql-palvelinta ollenkaan. Tästä päästään eroon muokkaamalla Salt -tilaa lisäämällä siihen `- makedirs: True` [3]

```
$ tail -vn +1 mysqlpw.sls 
==> mysqlpw.sls <==
#mariadb-server:
#  pkg.installed

/etc/mysql/mysqlsecret.conf:
  file.managed:
    - source: salt://config/mysqlsecret.conf
    - makedirs: True
    - template: jinja
    - context:
      mysqlpw: {{ pillar['mysqlbackendpw'] }}

#mariadb-server:
#  service.running:
#    - watch:
#      - file: /etc/mysql/mysqlsecret.conf

```


Nyt tila toimii kummallakin minionilla

![makedirs: True](https://i.imgur.com/Jd30DbY.png)

Myös `sudo salt '*' pillar.items ` -komento toimii odotetusti.

```
$ sudo salt '*' pillar.items
s201:
    ----------
bnc:
    ----------
s001:
    ----------
    mysqlbackendpw:
        yiavRugBynUtDyRudCecbackeuGlyp
s101:
    ----------
    mysqlbackendpw:
        .OkyeushdebPerWantIfyecgietCac
```


# B) Salt pilarin muuttuja ja oletusarvo

Oletusarvon lisääminen onnistuu vain muuttamalla Salt-tilassa `/srv/salt/mysqlpw.sls` olevan jinjan kontekstimuuttujan funktiota.

Sensijaan että käytetään `{{ pillar['mysqlbackendpw'] }}`, käytetään pillar.get -funktiota näin, `{{ pillar.get('mysqlbackendpw', no mysql pw specified.)}}`

Täten Salt-tilan konfiguraatio muuttuu vähän:

```
$ tail -vn +1 /srv/salt/mysqlpw.sls 
==> mysqlpw.sls <==
#mariadb-server:
#  pkg.installed

/etc/mysql/mysqlsecret.conf:
  file.managed:
    - source: salt://config/mysqlsecret.conf
    - template: jinja
    - context:
      mysqlpw: {{ pillar.get('mysqlbackendpw', 'no mysql pw specified.')}}

#mariadb-server:
#  service.running:
#    - watch:
#      - file: /etc/mysql/mysqlsecret.conf
```

Toimivuus voidaan tarkistaa käyttämällä `sudo salt '*' pillar.items` -komentoa. Täytyy kuitenkin muistaa että ensin pitää määritellä jollekkin aikaisemmin määrittelemättömälle minionille merkintä pilaritiedostoon `/srv/pillar/top.sls`. Nyt ei kuitenkaan määritellä mitään muuta, sillä haluamme Salt-tilan pillar.get -funktioon määritellyn oletusarvon s201 -minionin `/etc/mysql/mysqlpw.conf` -konfiguraatioon.

```
$ tail -vn +1 ../pillar/top.sls
==> ../pillar/top.sls <==
base:
  's101':
    - mysqlconfs101
  's001':
    - mysqlconfs001
  's201':
    -
```


Näin saadaan tulokseksi uusi tiedosto s201 -minionilla:

```
Summary for s201
------------
Succeeded: 1
Failed:    0
------------
Total states run:     1
Total run time:  50.297 ms
s101:
----------
          ID: /etc/mysql/mysqlsecret.conf
    Function: file.managed
      Result: True
     Comment: File /etc/mysql/mysqlsecret.conf updated
     Started: 08:57:01.435882
    Duration: 50.854 ms
     Changes:   
              ----------
              diff:
                  New file
              mode:
                  0644
```

Tiedoston sisältö on myös oikein päivitetty oletusarvolla:

```
@s201:~$ tail -vn +1 /etc/mysql/mysqlsecret.conf 
==> /etc/mysql/mysqlsecret.conf <==
Mysql PW: no mysql pw specified.
```

Mielenkiintoista kyllä, käyttämällä `salt '*' pillar.items` -komentoa, oletusarvoja ei listata minioneille, pelkästään jo määritellyt pillar -arvot:

```
$ sudo salt '*' pillar.items
s201:
    ----------
bnc:
    ----------
s001:
    ----------
    mysqlbackendpw:
        yiavRugBynUtDyRudCecbackeuGlyp
s101:
    ----------
    mysqlbackendpw:
        .OkyeushdebPerWantIfyecgietCac

```


B2


### Lähteet |

[1]: http://terokarvinen.com/2018/simple-secrets-in-salt-pillars
[2]: http://terokarvinen.com/2018/make-a-million-of-those-jinja-templating-salt-states
[3]: https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html


[1: http://terokarvinen.com/2018/simple-secrets-in-salt-pillars

[2: http://terokarvinen.com/2018/make-a-million-of-those-jinja-templating-salt-states

[3: https://docs.saltstack.com/en/latest/ref/states/all/salt.states.file.html

