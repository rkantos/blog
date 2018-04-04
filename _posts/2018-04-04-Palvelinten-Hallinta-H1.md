---
title: Palvelinten Hallinta ICT4TN022-4 2018 H1
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Palvelinten hallinta-kurssia. http://terokarvinen.com/2018/aikataulu-%E2%80%93-palvelinten-hallinta-ict4tn022-4-ti-5-ke-5-loppukevat-2018-5p

> 	a) Lue virallisesta Salt Getting Started Guide -kirjasta luvut Understanding SaltStack (noin 8 alasivua) ja SaltStack Fundamentals (6 alasivua, ei tarvitse asentaa demoympäristöä) ja SaltStack Configuration Management: Functions (1 alasivu). (Tätä lukutehtävää ei tarvitse raportoida).
> 
> 	b) Silmäile Laineen 2017 varastossa olevia salt -asetuksia. (Tätä lukutehtävää ei tarvitse raportoida).
> 	
> 	c) Asenna Salt Master ja Slave pull-arkkitehtuurilla (eli master on server). Voit laittaa herran ja orjan myös samalle koneelle. Kokeile suorittamalla salt:illa komentoja etänä.
> 	
> 	d) Kokeile jotain Laineen esimerkistä lainattua tilaa tai tee jostain tilasta oma muunnelma. Muista testata lopputuloksen toimivuus. Huomaa, että varastossa on myös keskeneräisiä esimerkkejä, kuten Battlenet-asennus Windowsille.
> 	
> 	e) Kerää laitetietoja koneilta saltin grains-mekanismilla.
> 	
> 	f) Oikeaa elämää. Säädä Saltilla jotain pientä, mutta oikeaa esimerkiksi omalta koneeltasi tai omalta virtuaalipalvelimelta. (Kannattaa kokeilla Saltia oikeassa elämässä, mutta jos se ei onnistu, rakenna jotain oikeaa konettasi vastaava virtuaaliympäristö ja tee asetus siinä).
> 	
> 	g) Vapaaehtoinen: asenna ja konfiugroi jokin palvelin Saltilla. (package-file-server)

# C) Palvelinten hallintatyökalun eli Salt Stackin asennus


####  Salt Master & Minion| [1]

Salt Masterin ja Minionin asennus Debianissa ja Ubuntussa tapahtuu kuten muiden yleisten ohjelmapakettien kanssa, eli suorittamalla komento: `sudo apt-get update && sudo apt-get install -y salt-master salt-minion` . 


#### Salt ohjelmakirjastot Debianilla| [4]
Koska omassa käytössä on kautta linjan sekä Debian että Ubuntu-pohjaisia virtuaalikoneita, huomasin heti alussa että Debianissa on käytössä huomattavasti vanhempi versio Saltista. Komennolla `salt --version` saimme versioksi Debian 9:llä "salt 2014.1.13 (Hydrogen)" 

Päätin tarkistaa löytyykö Saltille uudempi versio suoraan vaihtoehtoisesta ohjelmakirjastosta. Saltilta löytyykin omat ohjelmakirjastot Debianille, Red Hatille, Ubuntulle, SUSE:lle, Fedoralle, Windowsille, Mac:lle ja muutamalla muulle Debianin tai Ubuntun johdannaisille. 

Debianilla `repo.saltstack.com` käyttöönotto onnistuu seuraavasti:

Lisätään Saltin ohjelmakirjaston public key- avain:
```
wget -O - https://repo.saltstack.com/apt/debian/9/amd64/latest/SALTSTACK-GPG-KEY.pub | sudo apt-key add -
```
Lisätään ohjelmakirjaston osoite uuteen apt-listaan /etc/apt/sources.list.d/saltstack.list
```
echo "deb http://repo.saltstack.com/apt/debian/9/amd64/latest stretch main" | sudo tee /etc/apt/sources.list.d/saltstack.list
```

Suoritetaan normaalit komennot ohjelmakirjastojen lisäyksen jälkeen ja asennetaan uusin versio Saltista.

```
sudo apt-get update && sudo apt-get upgrade
#seuraavaa ei tarvita, jos Salt asennettiin jo
sudo apt-get update && sudo apt-get install -y salt-master salt-minion
```

####  Palomuuriasetukset Ubuntulla ja iptablesilla| [2]

Salt Master tarvii toimiakseen sisääntulevat TCP-portit numerolla 4505 ja 4506. Viimeisimmällä Ubuntun LTS-versiolla 16.04.4 Salt:n asennuksen mukana tulevat palomuurisäännöt ufw:lle, joten oikeat palomuuriasetukset Saltille saa käyttöön suorittamalla komennon `sudo ufw allow salt`. 

Debianilla tai iptablesilla oikeat palomuuriasetukset saa käyttöön suorittamalla seuraavat komennot seuraavasti:
```
sudo iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4505 -j ACCEPT
sudo iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4506 -j ACCEPT
```
Jonka jälkeen `iptables -L` komennolla pitäisi saada seuraava listaus:

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:4505
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:4506

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

```


# D) Arctic CCM Saltin esimerkkitila

####  Saltin tila käyttäen Arctic CCM esimerkkitiloja | [3]

Päätin käyttää tässä esimerkkitilana yksinkertaista tilaa user.sls, joka luo uuden käyttäjän ja varmistaa että käyttäjän tiedot pysyy jatkossa samanlaisina.

Luodaan tiedosto /srv/salt/user.sls -tiedosto tilaa varten. Salasana on tässäkin kopioitu suoraan esimerkistä, vaikka sen luonti onnistuu normaalisti komennolla seuraavan stackexchange -sivuston ratkaisulla https://unix.stackexchange.com/a/210146 `mkpasswd -m sha-512 `, olettaen että paketti whois on asennettuna.

```
 opiskelija:
   user.present:
     - fullname: opiskelija
     - shell: /bin/bash
     - home: /home/opiskelija
     - password: luo oma salasana tähän
     - enforce_password: True
```

Luodaan samalla myös top.sls -tiedosto, jotta jatkossa voidaan käyttää `sudo salt '*' state.highstate` -komentoa. top.sls tiedostossa määritellään käyttäjän luonti vain tietokoneelle, jonka salt id (hostanameen perustuva) on s001.

```
base:
   's001'
       - user
```

![Onnistunut tilan käyttöönotto](https://i.imgur.com/hWxYxXL.png)

Käyttäjätunnuksen toimivuus voidaan testata seuraavasti:

```
#ei toimi käyttäjällä opiskelijat

$ sudo -i -u opiskelijat whoami
sudo: unknown user: opiskelijat
sudo: unable to initialize policy plugin

#toimii

$ sudo -i -u opiskelija whoami
opiskelija
```


# E) Laitetietojen kerääminen Saltin Grains -moduleilla 

Koska etenkin aikaisemmin itsellä on ollut tapana etsiä paras ja samalla myös halvin vaihtoehto virtuaalipalvelimeksi, olen tullut käyttäneeksi useiden eri virtualisointitapoja käyttäviä virtuaalipalvelimia. Usein OpenVZ:lla tai LXC:llä toteutetut ratkaisut ovat kaikista halvimpia resurssiyksikköä kohden. Perinteisesti näissä virtualisointitavoissa ei ole ollut suuria ongelmia, mutta nykyään raudan kehityksen, yleisen hintojen halventumisen ja viimeisimpänä Intelin rautatason haavoittuvuuksien (Spectre & Meltdown) vuoksi KVM -pohjaisia virtuaalipalvelimia on järkevämpi pitää. OpenVZ:n avulla toimivat virtuaalipalvelimet eivät esimerkiksi tällä hetkellä tue 2.6.32 versiota uudempaa Kerneliä. [9] Tämä asettaa rajoitteita monien uudempien distrojen kanssa.. Uusimmat systemd:n versiot tukevat käytännössä 3.13 uudempia versioita. [10]

#### Virtualisoinmetodin selvittäminen Grainsin avulla | [5] [6] 

Koska virtualisointimetodi on yksi merkittävä tekijä joidenkin sovellusten konfiguroinnissa, selvitetään grainsin avulla millä virtualisointialustoilla minionit ovat. Suorittamalla komennon `sudo salt '*' grains.get virtual` saamme kaikkien minioneiden virtual -osan grainseista, joka kertoo virtualisointimetodin. Virtual -osa näyttäisi tukevan ainakin seuraavien virtuaalisointimetodien tunnistamista: Parallels, VMware, VirtualBox, KVM, Xen HVM, Xen PV, LXC, HyperV, OpenVZ
[11]

```
$ sudo salt '*' grains.get virtual
s101:
    physical
s201:
    kvm
s001:
    kvm
```

Kaikki virtuaalipalvelimet toimivat kvm:n avulla, joten yllättävää on että Salt Masterilla toimiva Minion tulkitaan fyysiseksi koneeksi.  Omalla tulkinnalla Saltin grains -python tiedostosta en päässyt kuin seuraavaan kohtaan, jonka oletan tulkitsevan Masterilla olevan Minionin fyysiseksi laitteeksi. [12]

```
        elif command == 'virt-what':
            # if 'virt-what' returns nothing, it's either an undetected platform
            # so we default just as virt-what to 'physical', otherwise use the
            # platform detected/returned by virt-what
            if output:
                grains['virtual'] = output.lower()
            break
```


# F) Saltin käyttö omassa oikeassa virtuaaliympäristössä


#### Cloudflare DNS oletus DNS-palvelimeksi | [7]

Koska mielestäni parhaiten 2018 aprillipäivästä mieleen jäänyt asia oli Cloudflaren 1.1.1.1 ja 1.0.0.1 osoitteisiin perustama julkinen DNS-palvelu yhteistyössä APNIC:n [8] kanssa, päätin tehdän resolv.conf -tiedoston sisällön vaihtavan Salt -tilan. Ajattelin siis seuraavanlaisen tiedostosisällön sisällyttämistä /etc/resolv.conf -tiedostoon kaikille palvelimilleni.

```
#/etc/resolv.conf
#Make only hostname work without fqdn, specify search domain
search 4e.fi
#Use new Cloudflare 1.1.1.1 DNS-servers instead of Google's "dont be evil DNS'"
nameserver 1.1.1.1
nameserver 1.0.0.1
```

Saltilla luodaan tila käyttäen file.managed -tilaa, joka varmistaa että /etc/resolv.conf tiedoston sisältö on aina määritellyn mukainen. Konfigurointitiedosto tallennetaan kansioon /srv/salt/config/resolv-cloudflare_config . Saltin sls-tiedostolle annetaan nimeksi resolv-cloudflare.sls . resolv-cloudflare.sls -tilan sisällöksi tulee seuraava:

```
/etc/resolv.conf:
  file.managed:
    - source: salt://config/resolv-cloudflare_config
```

Lisätään luotu tila myös top.sls -tiedostoon '*' -kohdan alle, jossa tila määritellään kaikkien minioneiden käytettäväksi:

```
base:
   '*':
       - resolv-cloudflare
   's001'
       - user

```

Testataan tilan toimivuus suorittamalla `sudo salt '*' state.highstate`:

![Onnistuneesti muokatut resolv.conf -tiedostot](https://i.imgur.com/XbQw2t2.png)

Voimme testata nimipalvelimien toimivuutta `dig *domainnimi*` avulla:

```
$ dig 4e.fi

; <<>> DiG 9.9.5-9+deb8u15-Debian <<>> 4e.fi
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56516
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1536
;; QUESTION SECTION:
;4e.fi.				IN	A

;; ANSWER SECTION:
4e.fi.			300	IN	A	185.83.218.186

;; Query time: 3 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Wed Apr 04 03:37:28 BST 2018
;; MSG SIZE  rcvd: 50
```

Rivi `;; SERVER: 1.1.1.1#53(1.1.1.1)` kertoo että DNS-palvelimena todellakin toimii ensimmäinen osoite /etc/resolv.conf -tiedostosta.


### Lähteet |

[1]: http://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux
[2]: https://docs.saltstack.com/en/latest/topics/tutorials/firewall.html#iptables
[3]: https://github.com/joonaleppalahti/CCM/tree/master/salt
[4]: https://repo.saltstack.com/#debian
[5]:  https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.grains.html
[6]: https://docs.saltstack.com/en/latest/topics/grains/
[7]: https://1.1.1.1/
[8]: https://www.apnic.net/about-apnic/
[9]:  https://openvz.org/Download/kernel
[10]: https://github.com/systemd/systemd/blob/master/README
[11]: https://fossies.org/linux/salt/salt/grains/core.py
[12]: https://github.com/saltstack/salt/blob/17ad2c42786490cdde829f5656f909469d527384/salt/grains/core.py

[1: http://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux

[2: https://docs.saltstack.com/en/latest/topics/tutorials/firewall.html#iptables

[3: https://github.com/joonaleppalahti/CCM/tree/master/salt

[4: https://repo.saltstack.com/#debian

[5:  https://docs.saltstack.com/en/latest/ref/modules/all/salt.modules.grains.html

[6: https://docs.saltstack.com/en/latest/topics/grains/

[7: https://1.1.1.1/

[8: https://www.apnic.net/about-apnic/

[9: https://openvz.org/Download/kernel

[10: https://github.com/systemd/systemd/blob/master/README

[11: https://fossies.org/linux/salt/salt/grains/core.py

[12: https://github.com/saltstack/salt/blob/17ad2c42786490cdde829f5656f909469d527384/salt/grains/core.py

[13: 

[14: 

[15: 
