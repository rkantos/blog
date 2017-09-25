---
title: Linux palvelimet ICT4TN021 2017 H4
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

H4 Virtuaalipalvelimet (VPS) & Wordpress
========================

# Hx
========================

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Linux-kurssia. http://terokarvinen.com/2017/aikataulu-linux-palvelimet-ict4tn021-4-ti-ja-5-to-alkusyksy-2017-5-op

> r) Kokeile julkista virtuaalipalvelinta (VPS). Voit vuokrata palvelimen esimerkiksi Linodelta, Amazonilta, DigitalOceanilta, OVH:lta tai monista muista paikoista. Edullisinta on käyttää GitHub Education -paketista DigitalOceanin palveluita.
> 
> Vaihtoehto: jos et jostain syystä halua vuokrata virtuaalipalvelinta, voit kokeilla tehdä testipalvelimen vagrantilla, mutta tämä ei ole yhtä jännittävää.
> 
> x) Laita julkinen domain-nimi osoittamaan koneeseesi. NameCheap ja Gandi ovat tunnettuja nimien vuokraajia. GitHub Education -paketista saa NameCheapilta .me domainin ilmaiseksi vuodeksi.
> 
> s) Laita julkiselle palvelimellesi käyttäjän kotihakemistoon tallennettu sivu näkymään Apachen oletussivuna.
> 
> y) Etsi julkisen palvelimesi lokeista esimerkkejä murtautumisyrityksistä. Voit etsiä lisätietoa IP-osoitteista ottamatta niihin yhteyttä esimerkiksi komennoilla geoiplookup tai whois.
> 
> Vapaaehtoisia lisätehtäviä:
> 
> v) Laita monta DNS-nimeä samaan IP-osoitteeseen. Apache Name Based Virtual Hosting.t) Asenna WordPress. Se on maailman suosituin sisällönhallintajärjestelmä (CMS). Samalla opit asentamaan kolmannen osapuolen valmiita PHP-ohjelmia. WordPress kannattaa asentaa wordpress.org:sta löytyvästä tervapallosta (.tar.gz).
> 
> u) Kokeile WordPressia kirjoittamalla esimerkkisisältöä.
> 
> WordPress vapaaehtoisia:
> 
> c) Ota järkevät URLit (permalinks) käyttöön
> d) Vaihda teema
> e) Varmuuskopioi sisältö
> f) Palauta varmuuskopioitu sisältö puhtaaseen WordPress-asennukseen
> g) Tee WordPressiin oma teema
> h) Asenna WordPressiin plugin (esim Dofollow)
> i) Tee WordPressiin oma plugin
> j) Lisää kuvia WordPressiin (ja laita tämä toimimaan)
> k) Laita WordPress nimipohjaiseen virtuaalipalvelimeen (http://thello.foo tms)
> 
> Muita vapaaehtoisia:
> 
> l) Asenna Drupal ja kokeile sitä
> m) Asenna Joomla ja kokeile sitä
> n) Hanki virallinen, selainten hyväksymä TLS-sertifikaatti Let’s Encryptistä
> o) Vaikea: Tee esimerkkisivu Python Flaskilla
> p) Vaikea: Tee esimerkkisivu Ruby on Rails (tuotantotyyppinen, ei pelkkä yhden käyttäjän testipalvelin)
> q) Vaikea: Tee esimerkkisivu Python Django:lla (tuotantotyyppinen, ei pelkkä yhden käyttäjän testipalvelin)




# R) Virtuaalipalvelimet & Vagrant

Tällä hetkellä itsellä ei ole tarvetta virtuaaliapalvelimelle, ja koska en ollut aikaisemmin kuullut Vagrantista, päätin kokeilla sitä. Olen vuokrannut virtuaaliapalvelimia jo vuodesta 2010. Jopa ennen ensimmäisen virtuaalipalvelimen vuokraamistani suunittelin oman virtuaalipalvelinpalvelun perustamista Suomeen. 2010 virtuaalipalvelimien hinnoittelu oli vielä aika kovaa, ja etenkin suomessa näin raon pienemmälle halvempia palvelimia tarjoavalle palveluntarjoajalle. Periaatteessa vieläkin Suomessa olisi hyvin rakoa etenkin hallituille palvelinpalveluntarjoajille. 2010 vuoden jälkeen kuitenkin monta asiaa on muuttunut ja vuosi 2017 taitaa olla kulminoitumispiste sille, että suomestakin alkaa saamaan jo "LEB" (ks. https://lowendbox.com/about/) hintaluokkaan mahtuvia palvelimia.


Edesvaikuttajina tähän ovat olleet erityisesti C-lion1 tietoliikennemerikaapeli, joka valmistui 2016 palvelemaan erityisesti yrityksiä. [1] Suomessa suuria palvelinkeskuksia on vuoden 2014 jälkeen tullut kuin sieniä sateella. Alla listaa palvelinkeskuksista joita suomessa on otettu käyttöön hieman ennen merikaapelin valmistumista ja sen jälkeen.

*Google, Hamina, 2011, jonka jälkeen laajennuksia vuonna 2012 ja 2015 [2] [3]*
*Yandex, Mäntsälä, 2014 [5]*
*Hetzner, Tuusula, 2017 [6]*


Ja suomalaiset jälkijunassa, kuten aina:
*Telia, Helsinki, 2018 [7] *
*Ficolo, Ulvila, 2018 -2019[13]*

Edellä mainituista vain oikeastaan Google ja Hetzner ovat kunnostautuneet virtuaalipalvelimien myymisessä suoraan kuluttajille. Googlelta ei Haminasta virtuaalipalvelimia ole kuitenkaan ainakaan vielä ennen vuotta 2017 saanut. Hetzneriltä LEB-kategoriaan mahtuvia virtuaalipalvelimia saanee todennäköisesti heti kun he saavat ensimmäisen palvelinkeskuksensa tulille. Hetzneristä on mainittava vielä se, että he ovat ostaneet kapasiteettiä ns. riittävästi. [12] Tälläisen virtuaalipalvelimen kanssa voi olla mahdollista esimerkiksi alentaa keskieurooppaan suuntautuville yhteyksille aiheutuvaa latenssia jopa yli 10ms verran nykyisiin pääsääntöisesti ruotsin kautta kulkeviin yhteyksiin verrattuna! 


#### LowEndBox & LowEndTalk |

LowEndBox -sivu on tarkoitettu erittäin halpojen virtuaaliapalvelimien löytämiseen. Sivu listaakin sivuillaan https://lowendbox.com/about/ ehtoja, joita tarjouksien pitää täyttää. Useimmiten sivuilta löytyy kuitenkin huomattavasti halvempiakin palvelimia. Alle dollarilla kuukaudessa voi löytää luotettavia palveluntarjoajia, joilta saa jaetun CPU-ytimen ja jopa 512MB muistia! Näistä omasta mielestäni ehdottomasti parhain on tällä hetkellä Time4VPS:n kahden vuoden (24€) sopimus , jolla saa n. eurolla kuukaudessa 512MB -muistilla varustetun OpenVZ:n perustuvan virtuaalipalvelimen.

Tällä hetkellä hyviä tarjouksia löytyy parhaiten LowEndTalk, eli LowEndBox-sivuston foorumin puolelta. Parempia tarjouksiakin kuin Time4VPS on saatavilla, joskin palveluntarjojien luotettavuus ei välttämättä ole Time4VPS tai Openvz.io -tasoa. Tämän takia halvimpien palveluntarjoajien kanssa toimiessa, suosittelen maksamaan palvelimista kuukausiperustaisesti, tai maksimissaan 

Muutama mainittava tarjous 2017 Syyskuulta:

https://www.lowendtalk.com/discussion/124916/gullo-s-hosting-2-year-vps-hosting-is-back#latest
2 $ / vuosi maksava virtuaalipalvelin 128MB-muistilla varustettuna. Palvelimet ovat NAT:n takana, joten palvelujen saattamiseksi näkyviin julkiseen internetiin voi joutua tekemään hieman enemmän konfigurointia. Sopii hyvin esimerkiksi sellaisiin tarkoituksiin, joissa suurinosa yhteyksistä muodostetaan palvelimen suunnalta, kuten esimerkiksi uptime-seurantaan tai irc-shellin pitoon. Näin pieniin hintaluokkiin mennessä on mainittava myös, että usein rajoitteena näissä toimii IPv4-osoitteiden "hinnat". Yleinen lisä-ip-osoitteiden hinta virtuaalipalvelimille on 0.5-2€/kk/ip. Täten on tavallista että virtuaalipalvelin, jolla on oma julkinen IPv4-osoite maksaa vähintään 10€/vuosi. Kannattaa siis olla varuillaan halvemmalla palveluja tarjoavien palveluntarjoajien kanssa, sillä joko palveluja ylläpidetään 
a. liikevaihdon kasvattamiseksi, 
b. ilmaisella sähköllä, 
c. ilmaisilla ip-osoitteilla, tai,
d. palveluntarjoaja yrittää kahmita mahdollisimman paljon rahaa asiakkailta kadotakseen yht'äkkiä...

https://www.lowendtalk.com/discussion/105291/let-storage-i-m-back#latest
n. 50€/2v tai n. 100€/3v maksava 1TB-tilalla varustettu "varastointivirtuaaliapalvelin". Tälläiselle palvelimelle, tai parille, jos haluaa huomioida varmuuskopiot voisi pystyttää esimerkiksi oman Nextcloud-palvelimen. Näin ei tarvitsisi luottaa isoihin toimijoihin. Pelkällä varastointikoolla mitattuna hinta on hyvin kilpailukykyinen Googlen ja Microsoftin pilvipalveluiden hinnoittelun kanssa. Tämä siis Googlen 1TB tilan maksaessa 100€/1v ja Microsoftin O365:n 1TB tilan maksaessa 70€/1v. Kumpaankin palveluun kuuluu luonnollisesti erinomaiset sähköpostipalvelut, dokumenttienkäsittely ja jako-ominaisuudet. Microsoftin palveluun sisältyy toki myös Office-paketin viimeisimmät työpöytäsovellukset. Omille linux-palvelimille on toki tarjolla monia avoimen lähdekoodin vaihtoehtoja, mutta niiden konfigurointikin toki vaatii oman aikansa.

https://www.lowendtalk.com/discussion/124610/hostens-2-40ghz-768mb-15gb-1tb-from-0-99-eur-mo-instant-setup#latest
OpenVZ:hen perustuvalla vuoden sopimuksella reilut 12€/1v maksava 768MB muistia varustettu virtuaalipalvelin. Vaikka palveluntarjoaja vaikuttaa luotettevalta viitaten 13 vuoden kokemukseen, ovat archive.org ja whois -hakujen tekeminen suositeltavaa. Näin ainakin joitakin faktoja pystyy tarkastamaan. Tällä palveluntarjoajalla ovat käytössä myöskin tavalliset pyörivät levyt. Samaan hintaan löytyy todennäköisesti myös täysin SSD-levyihin nojaavia virtuaaliapalvelimia.

#### ServerBear ja korvaaja ServerScope |

Eri LEB-palvelimien vertailuun on muutaman vuoden ajan ollut erilaisia suorituskykytesteihin perustuvia vertailusivustoja. Näistä tunnetuin oli ServerBear, joka kuitenkin lopetti toimintansa oikeasti 2014, mutta virallisesti vasta viimevuonna. [8] ServerBearin tilalle näyttää tällä hetkellä tulleen ainakin ServerScope [9]. ServerScope testaa ServerBearin tapaan avoimeen lähdekoodiin perustuvalla skriptillä palvelimien eri ominaisuuksia verkon nopeudesta massamuistin I/O (operaatioita sekunnissa) nopeuteen. [10] Muistutan että jos haluat ajaa mitä tahansa ServerScopen kaltaisia skriptejä omalla (virtuaali)palvelimellasi, suosittelen että teet sen heti kun olet ostanut ja asentanut palvelimen. Skripti käyttää luonnollisesti pääkäyttäjän oikeuksia, ja täten sillä on pääsy kaikkeen mitä palvelimellasi on. Tietoturvan säilyttämiseksi kannattaa virtuaalipalvelimen asennus suorittaa tyhjästä heti skirptin suorittamisen jälkeen uudelleen.

Kuva https://i.imgur.com/NEj56T4.png
![ServerScope Syyskuu 2017](/www/img/serverscope.png)


ServerScope:n etusivu ja top10 hinta-laatusuhteen omaavat virtuaalipalvelimet. Kärjessä mainitsemisen arvoisia tarjoajia ainakin BuyVM [11] Time4VPS tarjoamat palvelimet ovat tähän asti halvimmat alle 700km etäisyydellä (Helsinki) suomesta löytyvät palvelimet. Kyseessä on usein myös yksi halvimmista Eurooppalaisista palveluntarjoajista hinta-laatusuhteella mitattuna. Vielä lähempänä OpenVZ.io tarjoaa Tukholmasta halpoja virtuaalipalvelimia "Extreme" -luokassaan. Extreme -luokka OpenVZ.io:lla tarkoittaa virtuaalipalvelintarjontaa sellaisista sijanneista, joissa palveluntarjoajalla on mahdollisesti paikallisesti vain yksi tai muutama fyysinen palvelin. 


#### Eri virtualisointitavat |

Eri palveluntarjoajat käyttävät erilaisia tapoja virtualisoidakseen asiakaspalvelimia. Suosituimpia näistä ovat ehkä KVM, Xen ja OpenVZ. Lisäksi palveluntarjoajat käyttävät näiden virtualisointiteknologioiden lisäksi erilaisia hallintapaneeliratkaisuja. Näistä käytetyimpiä LEB-palveluntarjoajilla ovat SolusVM ja Virtualizor sekä WHCMS:n erilaiset lisäosat. WHCMS on ehdottomasti laajin käytössä oleva CRM, jota eri palveluntarjoajat käyttävät asiakkaiden 'rajapintana'. WHCMS hoitaa yleensä siis laskutuksen, sähköpostituksen, ticket-pohjaisen viestityksen ja kaiken muun palvelujen myynnin automatisointiin tarvittavan työn.

Valitessasi omaa virtualisointialustaa pystyy yleensä nopeasti päättämään mitä alustaa kannattaa käyttää. LEB-palvelimien hinnan takia on mielestäni myös järkevää käyttää useampaa palvelinta eri tarkoituksiin. 

Omaa kerneliä tarvittaessa kannattaa halvimman vaihtoehdon etsiminen aloittaa palveluntarjoajista jotka käyttävät KVM ja Xen-pohjaisia ratkaisuja, koska ne ovat ylivoimaisesti halvimpia tapoja palveluntarjoajille toteuttaa ympäristöjä, joissa asiakas voi käyttää kokonaan käyttöjärjestelmän kernelistä erillään olevia virtuaalipalvelmia. Muita tälläisiä ratkaisuja tarjoavat mm. Proxmox VE, Vmware, ja Hyper-V (Windows). 

Yleensä suurimpaan osaan palveluita, joita linux-palvelimella voi pyörittää riittää jokin ohjelmapohjainen tai osittain-ohjelmapohjainen virtualisointiteknologia, kuten OpenVZ tai LXC. Ylivoimaisesti suosituin näistä lienee puhtaassa LEB-virtualisoinnissa OpenVZ. OpenVZ:lla pärjäät todennäköisesti niin kauan kun sinun ei tarvitse tehdä muutoksia mihinkään, mikä vaatii uutta moduulia lisättäväksi kerneliin. Tälläisiä muutoksia ovat esimerkiksi monet verkkoihin liittyvät putkitukset, kuten 6to4 reitittimet, VPN:t jne. Kaikki OpenVZ-palveluntarjoajat joita olen tähän mennessä käyttänyt ovat kuitenkin tarjonneet tärkeimpiä moduleita, kuten OpenVPN:n käyttöön vaadittavan tun/tap -modulin. Myös libfuse-kernelmoduli, joka vaaditaan verkon yli toimivien tiedostojärjestelmien käyttöön, on saatavilla useimmilta OpenVZ palveluntarjoajilta. 

Lisää eri virtualisointitavoista, niiden eduista ja hyödyistä lisää täältä: 

https://veesp.com/en/blog/virtualization-technologies-comparison

#### virtuaalipalvelintarjoajien TOP3 |

Omaa tiukan budjetin virtuaalipalvelintarjoajaa hakiessa lähtisin liikkeelle näistä palveluntarjoajista:

1. Openvz.io, Nimensä mukaisesti OpenVZ, jolloin käytössä isäntäkoneen kernel
2. Time4vps, Ehdottomasti ykkönen jos tarvitaan oma kernel
3. Scaleway, tarjonnassa luotettavia virtuaalipalvelimia, mutta palveluntarjoajan ehdoton hienous on "virtuaalipalvelimen hinnoin" tuotetut dedikoidut palvelimet. Palveluntarjoaja on tietääkseni yksi ainoita pienempiä tarjoajia isojen tarjoajien kuten Googlen ja Amazonin ulkopuolella, jotka ovat kehittäneet täysin omia palvelinkehikoita ja niihin sopivia suurtiheystietokonekortteja: https://www.youtube.com/watch?v=XFhgSKNJP2s

### Vagrant |

Käytin tässä Tero Karvisen esimerkkiä [4].

> Install VirtualBox and Vagrant
> 
> $ sudo apt-get update
> $ sudo apt-get -y install virtualbox vagrant
> Create a New Virtual Machine & Connect
> 
> $ vagrant init bento/ubuntu-16.04
> $ vagrant up
> $ vagrant ssh

Ensin asensin Vagrantin komennolla `sudo apt-get -y install virtualbox vagrant`

Tämän jälkeen komennolla `vagrant init bento/ubuntu-16.04` saimme ladattua Vagrantin "levykuvan", josta virtualisoitu Ubuntun asennus käynnistetään.

![Vagrant-A / Lataus](/www/img/Vagrant-A.gif)

Lopuksi käynnistetään Vagrant ja yhdistetään siihen SSH:lla. `vagrant up` ja `vagrant ssh`

![Vagrant-B / Käynnistys](/www/img/Vagrant-B.gif)


# X) Oma domain

#### Cloudflare |

Käytän nimipalvelintarjoajani Cloudflarea, jolla on globaalisti toteutettu nimipalvelin-järjestelmä. Cloudflaren ilmaisella tunnuksella saa käyttöön käsittääkseni loputtomasti nimipalvelin palveluita eri domaineille. Viimeisimmän domainin rekisteröin Domainhotelli Oy:n kautta, sillä jostain syystä Ficoran palveluntarjoajien verkkotunnuspalveluun tekemäni tunnukset olivat poistettu taas käytöstä. Domainhotellista domainin hankkiminen on yhtä triviaalia kuin kaikista muistakin domainin -hankintapalveluista. Sieltä saa .fi -domainin kuitenkin nettohintaan 9€/vuosi. Domainhotellillta saa käyttöön myös oman DNS-nimipalvelun, joka vaaditaan, että Cloudflaren nimipalvelimia voi alkaa käyttämään. Tässäkin välissä voidaan tosin käyttää omalle virtuaalipalvelimelle asennettavaa nimipalvelinta, ja laittaa nimipalvelinasetukset osoittamaan Domainhotellin konfigurointipaneelista oman (bind) palvelimen dns-osoitteisiin.

En ole täysin varma "aktivoituiko" Domainhotellista hankkimani domaini viimeksi käyttämällä pelkkiä Cloudflaren DNS-palvelimia, vai auttoiko siihen se että konfiguroin ensin Domainhotellin DNS-palvelimet domainille. Suoraan Ficoralta domaineja hankittaessa vaaditaan jokin nimipalvelin ennen kun domain "aktivoituu". Aktivoituminen vaaditaan domainin lisäämiseksi Cloudflarelle.

https://i.imgur.com/oDXr0CZ.png
![Domainhotelli DNS-asetukset](/www/img/domainhotelli.png)
kuva hallintapaneelista
![Cloudflaren asetukset](/www/img/cloudflare-a-tietue.gif)


# S) Käyttäjän public_html -hakemisto domainin pääsivulle

Yksinkertaisimmin saamme yhden käyttäjän public_html -hakemiston näkymään rekisteröimämme domainin -etusivulla muokkaamalla kyseiselle käyttäjälle luotua virtual hostia, tässä tapauksessa /etc/apache2/sites-available/rkantos.conf lisäämällä ensimmäisen rivin `<VirtualHost *:80>`jälkeen `ServerName rekisteröimämmedomain.fi`

Tällöin konfiguraatiotiedoston alun tulisi näyttää tältä:

```
<VirtualHost *:80>
        ServerName rekisteröimämmedomain.fi

        ServerAdmin webmaster@localhost
        DocumentRoot /home/rkantos/public_html/
```




# Y) Murtautumisyritykset SSH-lokeilla

Omilla virtuaaliapalvelimillani olen estänyt ylimääräiset ssh-kirjautumis tai murtautumisyritykset Fail2ban:lla. Fail2ban estää ip-osoitteet määrätyksi aikaa liian monen yrityskerran jälkeen. Vähän ajan kuluttua Fail2ban käyttöönoton jälkeen murtautumisyritykset vähenivät selvästi vaikka kaikki aikaisemmin estetyt ip-osoitteet voivat omalla konfiguraatiolla yrittää kirjautumista muutamien minuuttien päästä uudelleen. 

Olin kuitenkin juuri ottanut toimintaan toisen SSH-palvelimen samalla virtuaalipalvelimella, mutta eri portissa. Tämäkin osa palvelimesta saa näköjään oman osansa kirjautumisyrityksistä. Yhdistävänä tekijänä vaikuttaa olevan käyttäjätunnus "admin" tai "ftp"/"ftpuser"

Jokaisessa lokiviestissä toistuu sama ssh-daemonin (sshd) raportoima viesti epäonnistuneesta kirjautumisyrityksestä joko väärällä käyttäjätunnuksella tai salasanalla.

Ensimmäisessä viestissä sshd kertoo että konfiguroinnin mukaisen autentikointimetodin kautta (PAM), ei löydy käyttäjää ftpuser. Kirjautumisyritys tulee IP-osoitteesta 50.62.56.171 ja asiakasportista 27657. Ensimmäisen salasanayrityskerran jälkeen käyttäjä, tai todennäköisemmin botti, yrittää salasanaa samalle käyttäjälle vielä kolmeen kertaan. Tämän jälkeen ssh-daemon konfiguroinnin mukaisesti tiputtaa yhteyden.
Luonnollisesti IP-osoite löytyy AbuseIPDB -sivustolta: https://www.abuseipdb.com/check/50.62.56.171

```
Sep 20 18:36:20 d sshd[22046]: Invalid user ftpuser from 50.62.56.171 port 27657
Sep 20 18:36:20 d sshd[22046]: input_userauth_request: invalid user ftpuser [preauth]
Sep 20 18:36:20 d sshd[22046]: pam_unix(sshd:auth): check pass; user unknown
Sep 20 18:36:20 d sshd[22046]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=50.62.56.171
Sep 20 18:36:22 d sshd[22046]: Failed password for invalid user ftpuser from 50.62.56.171 port 27657 ssh2
Sep 20 18:36:23 d sshd[22046]: pam_unix(sshd:auth): check pass; user unknown
Sep 20 18:36:24 d sshd[22046]: Failed password for invalid user ftpuser from 50.62.56.171 port 27657 ssh2
Sep 20 18:36:24 d sshd[22046]: pam_unix(sshd:auth): check pass; user unknown
Sep 20 18:36:26 d sshd[22046]: Failed password for invalid user ftpuser from 50.62.56.171 port 27657 ssh2
Sep 20 18:36:26 d sshd[22046]: Connection closed by 50.62.56.171 port 27657 [preauth]
Sep 20 18:36:26 d sshd[22046]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=50.62.56.171

```
Tämä kirjautumisyritys on muuten samanlainen edellisen kanssa, paitsi että IP-osoite on eri. Tälläkin kertaa sama IP-osoite on raportoitu AbuseIPDB -sivustolla jo useampaan otteeseen: https://www.abuseipdb.com/check/58.242.83.19
Koska toisen palvelimen SSH-konfiguroinnissa oli vielä merkintä `PermitRootLogin prohibit-password` - oli täten root -käyttäjätunnus käytettävissä, mutta vain ilman salasanaa. [14] Käytännössä "prohibit-password" tarkoittaa ei suinkaan sitä että root-tunnukselle pystyy kirjautumaan ilman salasanaa, vaan sitä, että salasanalla tunnukselle kirjautumiseen vaaditaan ssh-avain (privatekey). Tässä kohtaa en täysin ymärrä miksi PAM ylipäätään ensimmäisen kirjautumisyrityksen jälkeen tekee lokimerkintöjä, sillä ssh-daemonin ei pitäisi hyväksyä root-tunnukselle mitään muita kirjautumistapoja kuin ssh-avain.

Tässä lokimerkinnässä näkyy myös nähtävästi PAM:n raportoima 5 kirjautumisyrityksen raja, jolloin ssh-yhteys tiputetaan. `PAM 5 more authentication failures;`
Vaikkakin PAM on käytössä, mutta root-käyttäjän salasanakirjautuminen ei, näyttää SSH-daemonin PAM-moduuli silti liittyvän jotenkin SSH-daemonin oletuskonfiguroinnista johtuvaan liiallaisten kirjautumisyrityskertojen estoon.
```
Sep 21 21:13:10 d sshd[5969]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=58.242.83.19  user=root
Sep 21 21:13:11 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:14 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:18 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:21 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:24 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:27 d sshd[5969]: Failed password for root from 58.242.83.19 port 62393 ssh2
Sep 21 21:13:27 d sshd[5969]: error: maximum authentication attempts exceeded for root from 58.242.83.19 port 62393 ssh2 [preauth]
Sep 21 21:13:27 d sshd[5969]: Disconnecting: Too many authentication failures [preauth]
Sep 21 21:13:27 d sshd[5969]: PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=58.242.83.19  user=root
Sep 21 21:13:27 d sshd[5969]: PAM service(sshd) ignoring max retries; 6 > 3
```
Viimeinen merkintä on jälleen lähes identtinen ensimmäisen kanssa. IP löytyy jälleen AbuseIPDB:stä: https://www.abuseipdb.com/check/64.50.176.226 Käyttäjätunnusta ei taaskaan ole olemassa palvelimellani, mutta todennäköinen botti yrittää silti kirjautua salasanan avulla siihen 5 kertaa.
```
Sep 21 21:02:36 d sshd[23742]: Invalid user admin from 64.50.176.226 port 33974
Sep 21 21:02:36 d sshd[23742]: input_userauth_request: invalid user admin [preauth]
Sep 21 21:02:36 d sshd[23742]: pam_unix(sshd:auth): check pass; user unknown
Sep 21 21:02:36 d sshd[23742]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=64.50.176.226
Sep 21 21:02:38 d sshd[23742]: Failed password for invalid user admin from 64.50.176.226 port 33974 ssh2
Sep 21 21:02:38 d sshd[23742]: pam_unix(sshd:auth): check pass; user unknown
Sep 21 21:02:40 d sshd[23742]: Failed password for invalid user admin from 64.50.176.226 port 33974 ssh2
Sep 21 21:02:40 d sshd[23742]: pam_unix(sshd:auth): check pass; user unknown
Sep 21 21:02:42 d sshd[23742]: Failed password for invalid user admin from 64.50.176.226 port 33974 ssh2
Sep 21 21:02:42 d sshd[23742]: Connection closed by 64.50.176.226 port 33974 [preauth]
Sep 21 21:02:42 d sshd[23742]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=64.50.176.226
```


### Lähteet |

[1]: https://www.cinia.fi/palvelut/kansainvaliset-yhteyspalvelut/c-lion1-merikaapeli.html
[2]: https://yle.fi/uutiset/3-6915349
[3]: http://www.tivi.fi/Kaikki_uutiset/2015-03-11/Yle-Google-laajentaa-rajusti-Haminassa-tilat-moninkertaisiksi-3217212.html
[4]: http://terokarvinen.com/2017/virtual-machine-in-two-minutes-install-vagrant-and-virtualbox-on-ubuntu-16-04-3-live-usb
[5]: https://yle.fi/uutiset/3-7550739
[6]: http://www.is.fi/taloussanomat/art-2000001898010.html
[7]: https://www.telia.fi/yrityksille/tuotteet/tietoliikenne/telia-data-center/datakeskus-pitajanmakeen
[8]: https://twitter.com/ServerBear/status/765034545703813121
[9]: https://www.lowendtalk.com/discussion/85392/building-a-serverbear-alternative/p1
[10]: https://github.com/serverscope/serverscope-benchmark
[11]: https://lowendbox.com/?s=buyvm&searchsubmit=Find
[12]: http://www.zdnet.com/article/finlands-baltic-undersea-cable-gets-a-10m-to-help-boost-datacentre-connections/
[13]: https://www.rakennuslehti.fi/2017/09/paakaupunkiseudulle-suunnitellaan-uutta-datakeskusta/
[14]: https://askubuntu.com/questions/449364/what-does-without-password-mean-in-sshd-config-file

[1: https://www.cinia.fi/palvelut/kansainvaliset-yhteyspalvelut/c-lion1-merikaapeli.html
[2: https://yle.fi/uutiset/3-6915349
[3: http://www.tivi.fi/Kaikki_uutiset/2015-03-11/Yle-Google-laajentaa-rajusti-Haminassa-tilat-moninkertaisiksi-3217212.html
[4: http://terokarvinen.com/2017/virtual-machine-in-two-minutes-install-vagrant-and-virtualbox-on-ubuntu-16-04-3-live-usb
[5: https://yle.fi/uutiset/3-7550739
[6: http://www.is.fi/taloussanomat/art-2000001898010.html
[7: https://www.telia.fi/yrityksille/tuotteet/tietoliikenne/telia-data-center/datakeskus-pitajanmakeen
[8: https://twitter.com/ServerBear/status/765034545703813121
[9: https://www.lowendtalk.com/discussion/85392/building-a-serverbear-alternative/p1
[10: https://github.com/serverscope/serverscope-benchmark
[11: https://lowendbox.com/?s=buyvm&searchsubmit=Find
[12: http://www.zdnet.com/article/finlands-baltic-undersea-cable-gets-a-10m-to-help-boost-datacentre-connections/
[13: https://www.rakennuslehti.fi/2017/09/paakaupunkiseudulle-suunnitellaan-uutta-datakeskusta/
[14: https://askubuntu.com/questions/449364/what-does-without-password-mean-in-sshd-config-file