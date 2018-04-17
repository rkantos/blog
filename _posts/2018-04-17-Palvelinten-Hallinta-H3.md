---
title: Palvelinten Hallinta ICT4TN022-4 2018 H3
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
> b) Tiedosto muotista: tee yksinkertainen SLS-tilatiedosto, joka laittaa muuttujan tiedostoon. Käytä jinjan kontekstimuuttujaa (template: jinja, context: …).
> 
> c) SLS tilaa Jinjalla: tee yksinkertainen SLS-tilatiedosto, joka käyttää for-in -silmukaa. Voit esimerkiksi tehdä kolme tiedostoa silmukalla. (Tässä tehtävässä siis käytetään jinjaa vain SLS-tiedoston sisällä, älä sotke samaan esimerkkiin tekstitiedostojen sisällön muuttamista.)
> 
> d) SSH-demonin portti: tee tila, joka asentaa SSH-demonin valittuun porttiin. Käytä portin valintaan Jinjaa, siten että sshd_config:issa “Port:”-kohdan arvo tulee Jinjan muuttujasta.
> 
> e) Kokeile jonkun toisen opiskelijan tekemää Salt-tilaa. Kokeiltava tila voi olla mistä vain harjoituksesta. Opiskelijoiden raportteja ja koodeja löydät tämän sivun perästä kommenteista.




# B & C) Muuttujat jinjalla Salt-tilassa

Käyttäen apuna Tero Karvisen "Make a Million of Those - Jinja Templating Salt States" -artikkelia luodaan Salt-tila, joka luo muuttujan tiedostoon. [1]


#### Yksinkertainen SLS-tila, joka luo tiedostot muotista |

Luodaan tiedosto tilatiedosto `/srv/salt/monetfilut.sls` ja määritellään sisällöksi jinjan avulla seuraava sisältö:

```
{% for filut in ['yksfilu', 'kaksfilu', 'kolmfilu'] %}

/tmp/filut/{{ filut }}:
  file.managed:
    - source: salt://lähdefilu.txt
    - makedirs: True
    - template: jinja
    - context:
      filut: {{ filut }}
{% endfor %}
```

Tiedoston `/srv/salt/lähdefilu.txt`  sisältö:

```
Tällä tiedostolla voidaan luoda useampi tiedosto, jossa on sisällä muuttuja "{{ filut }}" jokaista tiedostoa kohden.
```

Komennon `sudo salt 's101' state.apply monetfilut` suorittamisen jälkeen:
![sudo salt 's101' state.apply monetfilut](https://i.imgur.com/GcypBvm.png)

```
$ tail -vn +1 /tmp/filut/*
==> /tmp/filut/kaksfilu <==
Tällä tiedostolla voidaan luoda useampi tiedosto, jossa on sisällä muuttuja "kaksfilu" jokaista tiedostoa kohden.

==> /tmp/filut/kolmfilu <==
Tällä tiedostolla voidaan luoda useampi tiedosto, jossa on sisällä muuttuja "kolmfilu" jokaista tiedostoa kohden.

==> /tmp/filut/yksfilu <==
Tällä tiedostolla voidaan luoda useampi tiedosto, jossa on sisällä muuttuja "yksfilu" jokaista tiedostoa kohden.
```

#### Lisätila Apachen vhost useammalla Aliaksella jinjan avulla | [2] [3]

#Heti Jinja-muottien kokeilemisen jälkeen halusin saada tehtyä muutoksia sls-tilojen avulla, joissa yhteen kohtaan tiedostoa tulisi useampi lisätty tieto. Tässä esimerkissä halusin lisätä Apachen vhostille useampia `ServerAlias` -määrittelyjä.

Loin siis uuden `/srv/salt/apache/vhost-jinja.sls` -tilatiedoston:

```
{%set aliases = [['s103.4e.fi', 's104.4e.fi', 's105.4e.fi']] %}


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
          {% for alias in aliases %}
            {% for elem in alias %}
            - {{ elem }}
            {% endfor %}
          {% endfor %}
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

Komennon `sudo salt 's101' state.apply apache.vhost-jinja` suorittamisen jälkeen:
![sudo salt 's101' state.apply apache.vhost-jinja](https://i.imgur.com/8bwJ6u2.png)


Tuloksena saadaan `/etc/apache2/sites-available/s101.4e.fi.conf` -tiedoston, jonka sisällöksi tulee:

```
<VirtualHost *:80>
ServerName s101.4e.fi
ServerAlias s102.4e.fi s103.4e.fi s104.4e.fi s105.4e.fi
ErrorLog /home/testi/logs/s101.4e.fi-error_log
CustomLog /home/testi/logs/s101.4e.fi-access_log combined
DocumentRoot /home/testi/public_html/
<Directory /home/testi/public_html/>
Options Indexes FollowSymlinks
AllowOverride All
</Directory>
</VirtualHost>
```

Myös osoite http://s102.4e.fi/ toimii edelleen, vaikka muilla ServerAliaksilla ei vielä olekkaan toimivia DNS-tietoja.


# D) SSH-demonin asennus ja portin määritys Jinja

Soveltaen edellisiä tapoja käyttää jinjaa ja muuttujia sekä Package-File-Service rakennetta, tehdään ssh-demonin asetuksia varten tiedosto `/srv/salt/ssh-port-jinja.sls` ja konfigurointitiedosto `config/ssh-port-jinja.conf`  Tässä kohtaa lisään SSH-demonille toisen portin, jossa se vastaanottaa pyyntöjä asiakkailta.

Tiedoston `/srv/salt/ssh-port-jinja.sls` sisältö:

```
openssh-server:
  pkg.installed

/etc/ssh/sshd_config:
  file.managed:
    - source: salt://config/ssh-port-jinja.conf
    - template: jinja
    - context:
      port: {{ 9988 }}

sshd:
  service.running:
    - watch:
      - file: /etc/ssh/sshd_config
```

Tiedoston `config/ssh-port-jinja.conf`` sisältö:

```
Port 22
Port {{ port }}
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
UsePrivilegeSeparation yes

KeyRegenerationInterval 3600
ServerKeyBits 1024

SyslogFacility AUTH
LogLevel INFO

LoginGraceTime 120
PermitRootLogin yes
StrictModes yes

RSAAuthentication yes
PubkeyAuthentication yes

IgnoreRhosts yes
RhostsRSAAuthentication no
HostbasedAuthentication no

PermitEmptyPasswords no

ChallengeResponseAuthentication no


X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes


AcceptEnv LANG LC_*

Subsystem sftp /usr/lib/openssh/sftp-server

UsePAM yes
```

Porttimuutos saatiin tehtyä onnistuneesti ja testi toimii:

![sudo salt 's101' state.apply ssh-port-jinja](https://i.imgur.com/kop0npD.png)

```
$ telnet s101 9988
Trying ::1...
Connected to localhost.
Escape character is '^]'.
SSH-2.0-OpenSSH_6.7p1 Debian-5+deb8u4
```



# E) Satunnaisen kanssaopiskelijan Salt-tilakokeilu

Päätin tässä kohtaa käyttää Ville Tourosen ohjeita Salt-tilan luomiseen sysstatille [4]

#### Sysstatin manuaalinen asennus |

```
$sudo apt-get install systat

#Sysstatin konfiguraation muuttaminen false -> true
sudoedit /etc/default/systat

#Käynnistetään sysstat
sudo systemctl start sysstat
```

Tavallisen asennuksen jälkeen Sysstat toimii kuten pitää:

![Sysstat iostat ](https://i.imgur.com/jyGrVk3.png)

#### Sysstat Salt-tilan luominen |

Luodaan `/srv/salt/sysstat/init.sls` ja `/srv/salt/sysstat/config` -tiedostot seuraavilla sisällöillä:

`/srv/salt/sysstat/init.sls`:

```
sysstat:
  pkg.installed

/etc/default/sysstat:
  file.managed:
    - source: salt://sysstat/config

startsysstat:
  service.running:
     - name: sysstat
     - watch: /etc/default/sysstat
```

`/srv/salt/sysstat/config`:

```
ENABLED="true"
```

Testataan Sysstat Salt-tila poistamalla Sysstat komennolla `sudo apt-get purge sysstat` ja suorittamalla Sysstat -tila komennolla `sudo salt 's101' state.apply sysstat`


Tila näyttää toimivan ensiyrittämällä [5]:

![sudo salt 's101' state.apply sysstat && ](https://i.imgur.com/X85CbAr.png)


### Lähteet |

[1]: http://terokarvinen.com/2018/make-a-million-of-those-jinja-templating-salt-states
[2]: https://stackoverflow.com/questions/30056622/how-to-iterate-over-a-list-of-list-in-jinja
[3]: https://stackoverflow.com/questions/33616695/how-do-i-append-to-each-item-in-a-list-of-strings
[4]: https://villetouronen.wordpress.com/2018/04/08/palvelin-hallinta-viikko-2-harjoitus-2-kayttajien-ja-heidan-kotisivujen-lisaaminen-kayttaen-saltia/
[5]: https://www.tecmint.com/sysstat-commands-to-monitor-linux/
[6]: 
[7]: 
[8]: 
[9]: 
[10]: 
[11]: 
[12]: 
[13]: 
[14]: 
[15]: 

[1: http://terokarvinen.com/2018/make-a-million-of-those-jinja-templating-salt-states
[2: https://stackoverflow.com/questions/30056622/how-to-iterate-over-a-list-of-list-in-jinja
[3: https://stackoverflow.com/questions/33616695/how-do-i-append-to-each-item-in-a-list-of-strings
[4: https://villetouronen.wordpress.com/2018/04/08/palvelin-hallinta-viikko-2-harjoitus-2-kayttajien-ja-heidan-kotisivujen-lisaaminen-kayttaen-saltia/
[5: https://www.tecmint.com/sysstat-commands-to-monitor-linux/
