---
title: Linux palvelimet ICT4TN021 2017 H3
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Linux-kurssia. http://terokarvinen.com/2017/aikataulu-linux-palvelimet-ict4tn021-4-ti-ja-5-to-alkusyksy-2017-5-op

> a) Asenna Apache, laita käyttäjien kotisivut (http://example.com/~tero) toimimaan. Testaa esimerkkikotisivulla. Tämä a-kohta on vaihdettu.
> 
> b) Surffaa oman palvelimesi weppisivuja. Etsi Apachen lokista esimerkki onnistuneesta (200 ok) sivulatauksesta ja epäonnistuneesta (esim 404 not found) sivulatauksesta. Analysoi rivit.Tee jokin seuraavista (yksi riittää, useampi vapaaehtoisena lisätehtävänä):
> 
> c) Tee virhe weppipalvelimella ajettavaan koodiin (esim PHP tai Python), etsi se lokista ja analysoi tuo lokirivi
> 
> d) Tee virhe johonkin Apachen asetustiedostoon, etsi ja analysoi tuo rivi. Etsimiseen sopivat esimerkiksi Apachen omat lokit, syslog sekä ‘apache2ctl configtest’.
> 
> e) Asenna ja kokeile PhpMyAdmin:a tai jotain muuta valmista weppiliittymää tietokantojen hallinnointiin.
> 
> f) Tee palvelimella ajettava weppiohjelma, joka tekee käyttäjälle jonkin yksinkertaisen laskun (esim. painoindeksi BMI)
> 
> g) Tee palvelimella ajettava weppiohjelma, joka käyttää tietokantaa. Voit tehdä jonkin yksinkertaisen CRUD-ohjelman, esimerkiksi TODO-listan
> 
> h) Tee Apachelle uusi sivu, joka näkyy suoraan palvelimen pääsivulla, mutta jonka sivuja voi muokata normaalin käyttäjän oikeuksilla (name based virtual host, DocumentRoot käyttäjän kotihakemistoon).
> 
> i) Kuinka monta eri HTTP Status:ta (200, 404, 500…) saat aiheutettua lokeihin? Selitä, miten aiheutit tilanteet ja analysoi yksi rivi kustakin statuksesta.
> 
> j) Asenna LAMP (Linux, Apache, MySQL, PHP). Testaa kunkin komponentin toiminta. Testaa lopuksi kokonaisuus. (Voit aloittaa tilanteesta, jossa Linux-käyttöjärjestelmä on jo asennettu, mutta ei muita (AMP) osia.
> 
> k) Kokeile jotain Flaskin uutta ominaisuutta flask-testipalvelimessa. Voit kokeilla esim. muotteja (templates), tietokantaa tai syötteiden ottamista lomakkeilta (forms).
> 
> l) Asenna Python Flask + PostgreSQL + Apache mod WSGI. Testaa kunkin komponentin toiminta. Testaa lopuksi kokonaisuus. (vaikea)

# A) Apache ja käyttäjäkohtaiset kotisivut |

#### Apache2 | Asennus

Apachen asennuspaketti löytyy suurimpien distrojen oletusohjelmavarastoista ja sen asentaminen onnistuu komennolla `sudo apt-get install apache2`

Käyttämässäni Ubuntu 16.04.3 -versiossa Apachen oletuskonfigurointitiedostot ovat valmiina näyttämään sivun kaikissa kyseiseen tietokoneeseen osoittavissa osoitteissa. Tämä selviää oletus virtuaali-isännän konfigurointitiedostosta, joka löytyy polusta `/etc/apache2/sites-available/000-default.conf`Ubuntulla voit muokata tiedostoa komennolla *sudoedit*, joka avaa oletuksena helppokäyttöisen Nano -editorin.

Tiedostosta löytyy rivi `<VirtualHost *:80>`,  jossa tähteä käytetään villikorttina kuvaamaan mitä tahansa osoitetta joka koneeseen osoittaa. Kaksoispiste ja numero kuvaa sitä porttia missä Apache2 -palvelin kuuntelee pyyntöjä. Tässä on kyseessä portti 80, joka on normaali HTTP-portti. Voit halutessasi estää oletussivuston näkymisen kaikkiin osoitteisiin, joko muokkaamalla VirtualHost -riviä näin, jolloin palvelin kuuntelee vain paikallisia pyyntöjä:

`<VirtualHost localhost:80>`

Tai poistamalla koko oletussivun konfiguroinnin käytöstä komennolla:

`a2dissite 000-default  && sudo service apache2 reload`



#### Apache2 |  Käyttäjien kotisivut

Käyttäjien kotisivut saadaan toimintaan lisäämällä käyttäjän, jolle kotisivut halutaan lisätä, kotihakemistoon kansio `public_html`. Ubuntulla Apachen mukana tulevat konfigurointiskriptit sisältävät konfigurointityökalun käyttäjäkohtaisten kotihakemistojen lisäämiseen. 

Otetaan siis käyttäjien omat kotisivut käyttöön komennolla `sudo a2enmod userdir`

Luodaan siis sisäänkirjautuneell käyttäjälle kotihakemistoon public_html -kansio komennolla `mkdir ~/public_html_`. Tilde, eli "~"-merkki tarkoittaa tässä siis kirjautuneen käyttäjän kotihakemistoa. Myöhemmin käyttäjien sivuja selattaessa tilde tarkoittaa, että ollaan jonkin käyttäjän kotisivuilla, eli käyttäjälle tehdyn public_html -kansion sisällä. Esimerkiksi "http://localhost/~käyttäjä"


#### Apache2 | Muutoksien käyttöönotto ja toiminnan varmistus

Käynnistetään ensin Apache2 -palvelin uudelleen, jotta kaikki muutokset tulevat voimaan. Palvelimen voi käynnistää uudelleen komennolla`sudo service apache2 restart`

Varmistetaan Apachen -testisivun toiminta menemällä osoitteeseen http://localhost ja kotisivujen toiminta menemällä osoitteeseen http://localhost/~käyttäjänimi , omassa tapauksessani http://localhost/~k

Tässä koko asennus giffinä!
![Apache2](/img/apache2-asennus.gif "apt-get")



# B) Apachen ilmoittamat statukset

Apachelle lähetettyihin pyyntöihin liittyvät lokimerkinnät löytyvät /var/log/apache2/access.log -polusta. Apache kirjoittaa lokiin Ubuntun oletuskonfiguraatiolla kaikki palvelimelle tehdyt pyynnöt ja niihin liittyvät metatiedot. [6]

#### HTTP tilannekoodi 200 | onnistunut sivulataus

Onnistuneen sivulatauksen lokimerkintään kuuluu, kuten kaikkiin access.log -tiedostoon oletuksena merkittäviin riveihin, sivua pyytäneen asiakkaan IP-osoite, aika, päivämäärä, palvelimen aikavyöhyke, pyynnön tyyppi, hakemisto, protokolla, HTTP-tilannekoodi, pyynnön koko ja tiedot käyttäjän selaimesta (User Agent).


`127.0.0.1 - - [12/Sep/2017:00:07:01 +0300] "GET /~k/ HTTP/1.1" 200 696 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:55.0) Gecko/20100101 Firefox/55.0"`

Oheisessa lokimerkinnässä kohdat tarkoittavat seuraavaa:

Asiakkaan IP-osoite: 127.0.0.1, pyyntö tuli siis localhostilta eli samalta tietokoneelta.

Päivämäärä, aika ja aikavyöhyke: [12/Sep/2017:00:07:01 +0300]

Pyynnön sisältö: "GET /~k/ HTTP/1.1", pyyntö käytti HTTP1.1 -protokollaa ja pyysi GET-pyynnöllä palvelimelta hakemistoa /~k/

Pyynnön koko ja HTTP-tilannekoodi: 200 696, HTTP-tilannekoodi 200 kertoo onnistuneesta latauksesta ja luku 696 pyynnön koosta tavuina.

Välittäjätieto tulisi "-" kohdassa väliviivan tilalle, mutta koska tässä tapauksessa asiakas pyysi suoraan hakemistoa /~k/ eikä tullut esimerkiksi linkin kautta ei lokimerkintä ole viivaa rikkaampi.

Viimeisenä kerrotaan käyttäjän tietoja User Agentin muodossa: "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:55.0) Gecko/20100101 Firefox/55.0", Käyttäjä tuli sivulle siis käyttäen selaimella joka mainostaa itseään Mozilla/5.0 yhteensopivana. Käyttäjällä oli käytössä 64 bittinen todennäköisesti graafisella ympäristöllä toimiva (X11) Ubuntu. User Agent kertoo vielä myös Firefoxin tarkan version (Firefox/55.0) sekä sen käyttämän selainmoottorin version (Gecko/20100101).


#### HTTP tilannekoodi 404 | sivua ei löytynyt

`::1 - - [12/Sep/2017:00:10:40 +0300] "GET /~k/blog/ HTTP/1.1" 404 498 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/62.0.3202.9 Chrome/62.0.3202.9 Safari/537.36"`

404 -tilannekoodin kanssa suurimpana erona oli käyttämäni selain Chromium.

Asiakkaan IP-osoite: ::1, pyyntö tuli siis localhostilta (IPv6) eli samalta tietokoneelta.

Päivämäärä, aika ja aikavyöhyke: [13/Sep/2017:00:10:40 +0300]

Pyynnön sisältö: "GET /~k/blog/ HTTP/1.1", pyyntö käytti HTTP1.1 -protokollaa ja pyysi GET-pyynnöllä palvelimelta hakemistoa /~k/blog/

Pyynnön koko ja HTTP-tilannekoodi: 404 498, HTTP-tilannekoodi 404 kertoo sivulatauksesta, jota ei löytynyt asiakkaan pyytämästä hakemistosta ja luku 498 pyynnön koosta tavuina.

Välittäjätieto tulisi "-" kohdassa väliviivan tilalle, mutta koska tässä tapauksessa asiakas pyysi suoraan hakemistoa /~k/blog/ eikä tullut esimerkiksi linkin kautta ei lokimerkintä ole viivaa rikkaampi.

Viimeisenä kerrotaan käyttäjän tietoja User Agentin muodossa: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/62.0.3202.9 Chrome/62.0.3202.9 Safari/537.36", Käyttäjä tuli sivulle siis käyttäen selaimella joka mainostaa itseään Mozilla/5.0 yhteensopivana. Käyttäjällä oli käytössä 64 bittinen todennäköisesti graafisella ympäristöllä toimiva (X11) Linux. User Agent kertoo vielä myös Chromiumin tarkan version (Ubuntu Chromium/62.0.3202.9) sekä sen käyttämän selainmoottorin version (AppleWebKit/537.36 (KHTML, like Gecko)).




# D) Apachen konfigurointivirhe | Open-Xchange

Olen noin vuoden ajan ajanut omalla palvelimellani Open-Xchange-nimistä groupware-sovellusta. [2] Sovellus pyrkii etenkin nykypäivänä olemaan vastine Googlen ja Microsoftin tarjoamille yhdistelmäpalveluille, jotka ovat saaneet alun hyvin nettiselaimessa toimivan sähköpostiohjelman luomisesta. Open-Xchange on vahvasti sähköpostiin perustuva, eikä se sellaisenaan toimi ilman sähköpostipalvelinta. Suurinosa Open-Xchangen komponenteista on avoimeen lähdekoodiin perustuvia, mutta sovellukseen on kuitenkin saatavilla monia lisäosia, joita varten tarvitaan ainakin maksullisia lisenssejä. Olen myös ottanut omalla palvelimellani käyttöön Nextcloudin, jossa on hyvin monia samankaltaisia ominaisuuksia kuin Open-Xchangessa. Kummassakin sovelluksessa on kuitenkin hyvät ja huonot puolensa ja etenkin sähköposti- ja toimistokäytössä Open-Xchange vaikuttaa tällä hetkellä paremmalta.

Päätin ottaa esimerkiksi Open-Xchangen, sillä juuri pariviikkoa aikaisemmin se oli taas lopettanut toimintansa. Yleensä päivitän kaikki palvelimen paketit lähes heti kun ne tulevat saataville. Open-Xchange:lla on tapana päivittyä usein ja yleensä päivityksillä on tapana aiheuttaa ongelmia Open-Xchangen toimivuuteen Apachen kanssa. Ongelmat liittyvät Open-Xchangen Apacheen tarkoitettuun proxy_http -konfigurointitiedostoon, joka on siis  Apachelle tarkoitettu reverse proxy*, joka välittä Open-Xchangen käyttöliittymän eteenpäin. [1]

* Reverse proxy tarkoittaa välityspalvelinta, joka piilottaa yleensä palvelimella sijaitsevan sovelluksen taustapalvelut taakseen ja välittää näin tietoa palvelimen tai verkon sisältä ikään kuin se tulisi samalta palvelimelta.



#### Virheilmoitus | Open-Xchange

Apache käynnistyy Open-Xchangen oletusasennusohjeiden mukaisella konfiguraatiolla ongelmitta. Kuitenkin viimeisimmän Open-Xchangen päivityksen jälkeen sain seuraavanlaista virhettä Apachen error.log -lokitiedostoon, jonka löytää polusta /var/log/apache2/error.log

Ensimmäisellä rivillä Apache kertoo klo 23:06 tapahtuneesta välityspalvelin (proxy) modulin virheestä Apachen prosessissa 17653 . *(111)Connection refused: AH00957:* , kertoo kuinka Apachen proxy-liitännäinen worker ei onnistunut yhdistämään konfiguroinnin (proxy_http.conf)  mukaiseen osoitteeseen, koska yhteys kiellettiin. AH00957 on ilmeisimmin proxy-liitännäisen virhekoodi *connection refused* -sanomalle. Viimeinen osa lokimerkintää kertoo protokollan (HTTP) ja portin (IPv4/IPv6 portti 8009), jolla proxy-moduuli yritti yhdistää paikalliseen (localhost*) palvelimeen.*HTTP: attempt to connect to [::1]:8009 (localhost) failed*

`[Wed Sep 11 23:06:01.257436 2017] [proxy:error] [pid 17653] (111)Connection refused: AH00957: HTTP: attempt to connect to [::1]:8009 (localhost) failed`

Seuraavalla rivillä Apachen saman proxy-moduulin funktio *ap_proxy_connect_backend* kertoo poistavansa modulin workerin käytöstä konfigurointitiedostossa määritellylle osoitteelle *localhost*. Myös 60s "katkaisuaika" on määritelty konfigurointitedostoon. (proxy_http.conf) 

`[Wed Sep 11 23:06:01.297297 2017] [proxy:error] [pid 17653] AH00959: ap_proxy_connect_backend disabling worker for (localhost) for 60s`

Viimeisellä rivillä kerrotaan koko tapahtuman lopputulema pyynnön aloittaneen asiakkaan *84.248.1.11* osoitetta myöden. Lokimerkinnässä näkyy myös HTTP-asiakasportti 44508. Jälleen AH -alkuinen AH01114 -virhekoodi vaikuttaa olevan Apachen mod_proxy -liitännäisen virhekoodi *failed to make connection to backend* -sanomalle. mod_proxy -liitännäinen ei siis onnistunut yhdistämään localhostilla sijaitsevaan taustapalveluun "backend". Viimeisimpänä lokirivillä kerrotaan mistä osoitteesta virhe on saanut alkunsa. Kyseessä on juuri Open-Xchange -ympäristön etusivu, joka tässä merkinnässä oli osoitteessa *https://example.com/appsuite/*

`[Wed Sep 11 23:06:01.297388 2017] [proxy_http:error] [pid 17653] [client 84.248.1.11:44508] AH01114: HTTP: failed to make connection to backend: localhost, referer: https://example.com/appsuite/`




# E)  phpMyAdmin

Tämä tehtävä on tehty LAMP-asennuksen MySQL:n ja PHP:n testaus -kohdassa [PhpMyAdmin](#mysql-ja-php-testaus--phpmyadmin)


# J) LAMP -asennus

Ubuntulla LAMP-asennus on helppo tehdä, sillä kaikki vaadittavat paketit löytyvät oletusohjelmavarastoista. Kaikki ohjelmat ovat myös lisensoitu avoimen lähdekoodin määrityksen mukaisilla (FSF[]) lisensseillä, joskin vain MySQL on GPL2 lisensoitu. Apache [4] ja PHP [5] käyttävät kummatkin omia lisenssejään. Sekä Apache että PHP -lisenssit ovat BSD tai MIT -tyyppisiä lisenssejä, jotka eivät sisällä rajoituksia koodin käyttämiselle suljetun lähdekoodin sovellutuksissa.

LAMP-asennuksen voi suorittaa komennolla `sudo apt-get install lamp-server^` (`^` on tärkeä)

Komento on sama, joka ajetaan jos LAMP-palvelin valitaan asennettavaksi Ubuntun asennuksen yhteydessä. [3]

Ainoa käyttäjältä vaadittava toimenpide on vahvan pääkäyttäjän salasanan valitseminen MySQL-palvelimelle. Yksi luotettavimmista tavoista luoda vahva salasana on käyttää APG:ta. [7] APG:n asentaminen onnistuu komennolla `sudo apt-get install apg` , jonka jälkeen vahvoja salasanoja voi luoda vain suorittamalla apg-komennon.

Valittuasi vahvan salasanan ja kirjoitettuasi (tai kopioituasi sen asennusikkunaan ja talteen) asennus valmistuu.


#### Apache2 testaus |

Apache voidaan testata menemällä paikalliseen osoitteeseen http://localhost/

#### MySQL ja PHP testaus | phpMyadmin

Ehkä helpoin tapa testata kummatkin on asentaa phpMyAdmin. phpMyAdmin käyttää sekä php:ta että MySQL-tietokantaa toimiakseen. 

Myös phpMyAdmin voidaan asentaa Ubuntussa oletuksena löytyvistä ohjelmavarastoista komennolla `sudo apt-get install phpmyadmin`
Asennus pyytää antamaan MySQL:lle pääkäyttäjän salasanan, joka on määritelty asennettaessa mysql-server -pakettia LAMP:n asennuksen yhteydessä.

Ubuntun ohjelmavarastoista löytyvä phpMyAdmin -paketti asentaa itselleen automaattisesti oman virtualhost -konfiguroinnin, jonka seurauksena phpMyAdminin hallintapaneeliin pääsee osoitteesta http://localhost/phpmyadmin

Ohessa jälleen phpMyAdminin asennus ja siten myös MySQL:n ja PHP:n testaus giffinä.

![PHP & MySQL & phpMyAdmin](/img/phpmyadmin.gif "apt-get")


### Lähteet |




[1]: https://oxpedia.org/wiki/index.php?title=AppSuite:Open-Xchange_Installation_Guide_for_Debian_8.0#Configure_services
[2]: https://fi.wikipedia.org/wiki/Ryhm%C3%A4ty%C3%B6sovellus
[3]: https://help.ubuntu.com/community/ApacheMySQLPHP
[4]: https://github.com/apache/httpd/blob/trunk/LICENSE
[5]: http://php.net/license/
[6]: https://httpd.apache.org/docs/2.4/logs.html
[7]: https://help.ubuntu.com/community/StrongPasswords#Generating_Strong_Passwords_in_Ubuntu

[1: https://oxpedia.org/wiki/index.php?title=AppSuite:Open-Xchange_Installation_Guide_for_Debian_8.0#Configure_services

[2: https://fi.wikipedia.org/wiki/Ryhm%C3%A4ty%C3%B6sovellus

[3: https://help.ubuntu.com/community/ApacheMySQLPHP

[4: https://github.com/apache/httpd/blob/trunk/LICENSE

[5: http://php.net/license/

[6: https://httpd.apache.org/docs/2.4/logs.html

[7: https://help.ubuntu.com/community/StrongPasswords#Generating_Strong_Passwords_in_Ubuntu