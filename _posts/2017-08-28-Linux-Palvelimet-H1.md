---
title: Linux palvelimet ICT4TN021 2017 H1
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Publish H1 22.8.2017 'Livetikku'
{:.no_toc}

**Sivu on julkaistu ensimmäisen kerran 28.8.2017 ja** 
**päivitetty 30.8.2017**


# Sisältö
{:.no_toc}

* Table of contents placeholder
{:toc}

### Tehtävänanto



Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Linux-kurssia. http://terokarvinen.com/2017/aikataulu-linux-palvelimet-ict4tn021-4-ti-ja-5-to-alkusyksy-2017-5-op

> a) Tee oma Linux-livetikku. Kokeile sitä jossain muussa kuin koulun koneessa. Tämä kohta ei edellytä asentamista – jos kuitenkin asennat, ota ensin varmuuskopiot.
> 
> b) Listaa testaamasi koneen rauta (‘sudo lshw -short -sanitize’).
> 
> c) Asenna kolme itsellesi uutta ohjelmaa. Kokeile kutakin ohjelmaa sen pääasiallisessa käyttötarkoituksessa.
> 
> d) Mitä lisenssiä kukin näistä ohjelmista käyttää? Selitä lyhyesti, mitä oikeuksia ja velvolisuuksia tuosta lisenssistä seuraa.
> 
> d->e) Listaa käyttämäsi ohjelmat (esim. MS Word), kunkin ohjelman käyttötarkoitus (esim. Tekstinkäsittely) ja vastaava vapaa Linux-ohjelma (esim. LibreOffice Writer). Jos johonkin tarkoitukseen ei löydy vapaata Linux-ohjelmaa, listaa sekin.
> 
> f) Vapaaehtoinen lisätehtävä: varmuuskopioi tiedostosi (voit käyttää esimerkiksi ulkoista USB-levyä)

# A) Boottaavan Linux USB-tikun luominen #

#### Kohde | Asus QM1 & x5-Z8300 SoC
Päätin ottaa tässä kohtaa luonnollisesti yhden haastavimmista kohteista Linuxin pyörittämiseen. Käytössä on siis viimeisin (2017) versio QM1 tikusta x5-z8300 SoC:lla. Olen aikaisemminkin yrittänyt saada Asuksen QM1 - 'PC-Tikkuun' Linuxin asennettua.  En kuitenkaan tähän asti ole siinä onnistunut. Laitteessa on ilmeisesti samankaltainen UEFI BIOS muiden vastaavien Asuksen Atom ja Windows -pohjaisten halpojen kannattavien, tablettien ja jopa puhelimien kanssa. Kaikissa näissä laitteissa tulee mukana oletusarvoisesti Windows 8 tai Windows 10. Kyseinen PC-tikku pohjautuu ja muistuttaa hyvin vahvasti Intelin vastaavaa laitetta, mutta hieman ominaisuuksista riisuttuna. Näistä merkitsevimpinä toisen USB -liitännän (USB3) puuttuminen. [1]

#### Asus QM1 -laitteen käynnistys live cd:ltä |
Laitteen pitäisi käynnistyä UEFI-valikkoon ESC-näppäimellä, missään ei kuitenkaan varsinaisesti kerrottu missä tai miten tätä nappia pitäisi painaa, joten yleensä helpoin tapa päästä laitteen UEFI-valikkoon on käynnistää Windows uudestaan painamalla shift näppäintä valitessa "käynnistä uudelleen" tai "restart" vaihtoehtoa. Näin päästään tämän näköiseen valikkoon:

![Windows 10 Recovery -valikko](/img/w8uefirecovery.jpg)

 Myöhemmin myös itselle selvisi että ESC-nappia voi painaa pohjassa. Kaikki valinnat vievät lopulta samaan Asuksen UEFI-valikkoon, joka näyttää samalta kuin kohdassa [USB-tikun luonti &#124; Easy2Boot](#usb-tikun-luonti--easy2boot)


#### USB-tikun luonti | Easy2Boot  ####

Yleensä boottaavien USB-tikkujen asentamiseen olen tottunut käyttämään YUMI:a https://www.pendrivelinux.com/yumi-multiboot-usb-creator/ Aikaisempien huonojen kokemuksien johdosta etsin Googlella seuraavia vaihtoehtoja. Jotkut väittivät kaiken toimineen ilman ongelmia 'tavallisin' keinoin. Ainakin yksi [2] oli maininnut Easy2Boot:n käytön. Olen joskus aikasemminkin käyttänyt samaa ohjelmaa kokeillessa erilaisia Windowsilla USB-tikkujen luontiin soveltuvia ohjelmia. Koko sovelluksen kätevin ominaisuus onkin varmaan se että siinä on eräänlainen oma bootloader, joka mahdollistaa pelkkien iso-tiedostojen lataamisen muistikortille. Etenkin hitailla muistikorteilla tämä säästää aikaa, kun kaikkia pieniä tiedostoja ei tarvitse purkaa erikseen.

Asensin E2B:n ensin USB-tikulle... Sitten USB-hubin kanssa säätäessä muistin juuri sen toisen USB-paikan puuttumisen laitteestani. USB-hubin läpi en luonnollisesti saanut USB-tikkua boottaamaan. Suoraan USB-tikun liittäminen laitteeseen ei myöskään paljon edistä asiaa, sillä sitten laitteeseen ei voi enää liittää näppäimistöä.

Seuraavaksi asensin E2B:n siis MicroSD-kortille ja kokeilin bootata laitteen sen kautta. -- Ei löytynyt sen paremmin mitään boottivalikosta. 
![Asus QM-1 tyhjä boot-valikko](/img/emptybootmanager.jpg)

![Asus QM-1 UEFI-valikko](/img/qm1bootmenu.jpg)

![Asus QM-1 UEFI-valikko](/img/bootx64.efi.jpg)
Viimeisenä vaihtoehtona laitteen UEFI -valikossa on vielä "Boot from file".. Tätä kautta voi selata Windowsin BOOT-osiolle josta löytyy .efi -päätteinen tiedosto, jonka kautta saa laitteelle esiasennetun Windowsin päälle. Tätä kautta muistinkin että vastaavanlainen tiedosto pitäisi löytyä myös Ubuntun live-cd isosta. Pienen Googletuksen jälkeen löysin mahdollisesti oikeille jäljille vievän ohjeen. https://askubuntu.com/questions/395879/how-to-create-uefi-only-bootable-usb-live-media 

Koska olin jo samasta microsd-kortista tehnyt boottaavan, ei seuraava vaihe vaatinut kuin ISO-tiedoston tiedostojen purkamisen ja siirtämisen sille. Huomiona tosin se, että E2B formatoi muistitikun oletuksena NTFS -formaattiin ja Asuksen UEFI-bios lukee FAT32 -tiedostojärjestelmiä.

Kuten kuitenkin aikaisemminkin, ei laite suostu boottaamaan tälläkään vaihtoehdolla vaan saan virheet \EFI\BOOT\BOOTx64.EFI boot failed. tai \EFI\BOOT\grubx64.efi boot failed. , riippuen kumpaa tiedostoa \BOOT -kansiossa yritän käyttää. 

![Tyhjää täys](/img/bootx64.efi-boot-failed.jpg)

#### USB-tikun luonti | Linuxium [15]

En osaa varsinaisesti sanoa miksi en aikaisemmin ollut törmännyt "ainoaan viimeiseen vaihtoehtoon", eli Linuxiumin isorespin.sh:n https://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html Isorespinin ohjeissa tiivistetään QM1 -kaltaisten laitteen ongelmat, joka on siis 64-bittisten käyttöjärjestelmien boottaus ja asennus käyttäen 32-bittistä bootloaderia.. Mutta eipä onnistanut isorespinillä käyttäen Ubuntu Maten 16.04.3 -versiota. Tällä kertaa kuitenkin aikaisemmista kerroista eroavasti, tuli UEFI:n boottivalikkoon "Windows Boot Managerin" rinnalle uusi vaihtoehto "EFI SD/MMC Card". Myös microSD-kortille puretun Ubuntu-asennuksen GRUB-valikkoon päästiin tälläkertaa. Nyt saatiin myös selvät virheilmoitukset! 

![Kernelin virheilmoitukset](/img/initramfs-pci-error.jpg)

Edit: Tässä päästiin siis lataamaan Ubuntua muistiin, sillä kyseessähän ovat Kernelin virheilmoitukset.

*intel_ish_ipc 0000:00:0a.0: [ishtp-ish]: Timed out waiting for FW*

*intel_ish_ipc 0000:00:0a.0: [ishtp-ish]: ISH: hw start failed.*

*intel_ish_ipc 0000:00:0a.0: [ishtp-ish]: ISH: PCI driver initialization failed*

Tämän jälkeen Busyboxin perus ash shelli käynnistyy ja tulee vielä virhe: *(initramfs) Unable to find a medium containing a live file system.*


Käytännössä Isorespinin GUI:n valinnoilla tehty komento, jolla ISO-tiedosto saatiin ohjelmalla tehtyä on samanlainen kuin esimerkissä: 
> **isorespin.sh -i ubuntu-17.04-desktop-amd64.iso -u -l rtl8723bs_4.12.0_amd64.deb -f linuxium-install-UCM-files.sh -f wrapper-linuxium-install-UCM-files.sh-c wrapper-linuxium-install-UCM-files.sh**


# B) Koneen rauta sudo lshw -short -sanitize
Koska en keksinyt Asuksen laitteen jälkeen mielenkiintoisempaakaan laitetta, niin kaivoin kaapin pohjalta yhden taannoin parhaiten Linux-tuetuista tietokoneista. Vuoden 2005 Thinkpad T43:n.
Komentoa **sudo lshw -short -sanitize** -käyttäen saatiin seuraavat tiedot: ... ja eipäs saatukkaan, sillä T43 ei käynnistynytkään enää uusimmalla Ubuntun versiolla vaan stoppasi ilmeisesti ennen GRUB:n avautumista kohtaan *Cannot find grldr*

Eli raudan paikalla on nyt siis X220, eli koulukoneeni tiedot: https://github.com/rkantos/rkantos.github.io/blob/master/Linux%20Palvelimet%202017-3/X220%20lshw


# C) Uusien ohjelmien testaus

#### NeoFetch [9] | Yksinkertainen järjestelmä ja käyttöjärjestelmätietojen näyttö
Tuleekin todennäköisesti tarpeeseen usein.. Useamman virtuaalipalvelimen kanssa ilman lxc:tä, Dockereita tai Kubernetejä tai mitään muuta monen palvelimen hallintaan tarkoitettua sovellusta unohtaa aina välillä mikä tarkka versio Ubuntusta tai Debianista olikaan käytössä. **Uname -a** & **lsb_release -a** komennoilla selviää Kernel ja distron versio. **lscpu** komennolla selvittää prosessorin mallin ja tiedot, **top:lla** ja **htop:lla** selviää muistinkäyttö. NeoFetchillä selviää kuitenkin nämä kaikki ja vähän lisää.
NeoFetchin voi asentaa lisäämällä PPA:n:
sudo add-apt-repository ppa:dawidd0811/neofetch
sudo apt update
sudo apt install neofetch

#### Peek [10] | Tehokas ohjelma GIF-animoitujen ruutukappausvideoiden tekoon
Peek on erittäin helppokäyttöinen ja tehokas ohjelma pienien ruutukaappausvideoiden tekoon. Videot voi tehdä GIF, WebM tai mp4 -muotoon. Ensimmäinen on yhteensopiva melkein minkä tahansa selaimen kanssa (paitsi lynx tai links jne) ja WebM sekä mp4-videot toistuvat uusimmilla HTML5:sta tukevilla selaimilla ilman ongelmia.
Peekin voi asentaa Ubuntuun lisäämällä PPA:n:
Peekillä tälläisen luominen onnistui alle kahdessa minuutissa sovelluksen asentaminen mukaanlukien:
![Peek:n nopea testaus](/img/peek-testi.gif "apt-get")


sudo add-apt-repository ppa:peek-developers/stable
sudo apt update
sudo apt install peek

#### undistract-me [11] | Ilmoitus pitkien komentojen loppumisesta
undistract-me:n tarkoitus on toimia komentokehoitteessa niin, että se havaitsee pitkään juoksevat komennot. Oletuksena undistract-me menee päälle kun jokin komento on pyörinyt kymmenen sekunnin ajan. Sovelluksen tarkoituksena on tehostaa työaikaa, sillä se ilmoittaa komennon suorituksen loppuneen. Näin voi käyttää muun ajan tehokkaasti vaikka seuraavan komennon etsimiseen ja jatkaa komentojen antamista heti edellisen loppumisen jälkeen, hukkaamatta aikaa komentokehoitteen tuijottamiseen.

undistract-me löytyy suoraan Ubuntun ohjelmavarastosta ja sen voi asentaa suoraan komennolla:
sudo apt install undistract-me

# D) Ohjelmistojen lisenssit
NeoFetch käyttää MIT -lisenssiä, joka sallii sen muokkaamisen ja käyttämisen muualla, kunhan lisenssi on muokatussa lähdekoodissa mukana. MIT-lisenssiä voidaan käyttää myös suljetun lähdekoodin ohjelmistoissa. [12]
Peek on lisensoitu GPL:llä. GPL-lisenssi on Free Software Foundationin ylläpitämä lisenssi ja on täten täysin avoimien ohjelmistojen määritelmien mukainen. [13]
undistract-me käyttää Expat/MIT-lisenssiä, joka on yhteensopiva GPL:n kanssa. [14]

# E) Linuxilla käyttämäni ohjelmat

#### Nettiselain |
Windowsilla käytän luonnollisesti Chromea, kuten kuka tahansa Googlen palveluita käyttävä. Google vain tuntuu johtaneen kehitystä selaimissa jo lähes vuosikymmenen! Ubuntulle ja muille Distroille on toki saatavissa Chrome, mutta Chromium ainakin "yrittää" olla avoimeen lähdekoodiin perustuva. Suurin osa Chromiumin lähdekoodista on BSD- ja MIT-lisensseihin perustuvaa. Chromium on siinä mielessä hyvä että se sopii ainakin jonkinlaiseen "foliohattuiluun", sillä oletuksena monet Googlen palveluihin nojaavat toiminnot eivät (2013 jälkeen [3] ) ilman kikkailua enää toimi [4]

#### Tekstinkäsittely & toimistosovellukset |
Omalla dualboot Ubuntu-kannettavalla käytän hyvin harvoin MS Wordiin verrattavissa olevia tekstinkäsittelyohjelmia. Pääosin käytänkin Nextcloudin kanssa hyvin yhteen pelaavaa **QOwnNotesia, Google Docsia Chromiumilla, Katea** (KDE:n ylläpitämä Notepad++ tyyppinen tekstieditori) ja tietenkin komentokehotteessa **Nanoa**. 

Kuvakaappauksien ottamiseen en ole Linuxille löytänyt vielä yhtä sulavasti ja hyvin toimivaa kuvakaappaus-ohjelmaa kuin **Greenshot**. Greenshotilla voi nopeasti ottaa kuvan haluamastaan näytön kohdasta ja tallentaa sen haluammillaan ennalta määritellyillä asetuksilla. Greenshotissa on myös suoraan hyvin toimiva Imgur-latausvaihtoehto. Linuxilla käytän **imgur-screenshot** -skriptiä [6], joka hoitaa ainakin Imgur-latauksen sulavasti, mutta pyörii muuten vain taustalla.

#### Media, musiikki, videot |
**Spotify** on yleensä päällä ja pyörii suoraan Spotifyn omasta asiakasohjelmasta. [5] Muut mediatiedostot toistan VLC:llä, oli ne sitten paikallisia tai verkosta striimattavia. Myös Windowsilla kaikki hoituu samoilla ohjelmilla, joskin Spotifysta on vielä käytössä se "ainoa oikea" versio eli 0.9.7 [6]. Ks. myös muutaman muun täydellisesti kuvaamat tunteeni Spotifyn huonoimmasta uudistuksesta: https://www.reddit.com/r/spotify/comments/2y6a68/spotify_update_1011060gc75ebdfd_mega_discussion/cp9ozqf/ Ubuntulla Spotifyn ohjelmavaraston lisänneenä voi lähimmäksi pääsee Spotify:n versiolla 0.9.17. Version voi asentaa komennolla **sudo apt-get install spotify-client-0.9.17**

#### Pelaaminen |
Pääasiassa pelaamani pelit eivät toimi Linuxeilla edes Winellä. Uusimmat pelit toimivat käytännössä vain DirectX 10 tai 11 turvin. Winen tukema viimeisin DirectX versio on tänään (Elokuu 2017) DirectX 9.0c [7]. Olen toki unelmoinut pitkään että koko työpöytäympäristön voisi siirtää virtualisoituun ympäristöön ja täten vain pelejä voisi ajaa omissa Windows-virtuaalikoneissa. Tämäkin on ainakin vielä 2017 loputon suo (ajallisesti ja taloudellisesti), sillä pelaamani pelit kuten Battlefield 3,4 tai 1 vaativat käytännössä kaikki käytettävissä olevat resurssit. IOMMU / PCIe passthrough:n pohjautuvat ratkaisut ovat kyllä mahdollistaneet tälläiset ratkaisut jo useamman vuoden, mutta sekä ratkaisujen testaamattomuuden ja tuen puutteen takia en ole niihin vielä ehtinyt. Ks. Toimiviksi väitetyt GPU passthroughta käyttävät ratkaisut: https://www.reddit.com/r/pcmasterrace/comments/2z0evz/gpu_passthrough_or_how_to_play_any_game_at_near/ ja https://www.reddit.com/r/linux_gaming/comments/5yhxqf/quick_gpu_passthrough_guide/ Täällä myös hieman suorituskykytestejä: https://forum.level1techs.com/t/how-fast-is-kvm-host-vs-virtual-machine-performance/110192 
Ongelmaksi jää myös Windows-lisensointi, johon periaatteessa jokaiselle virtualisoidulle Windowsille täytyisi hommata retail -lisenssi.

Joitakin harvemmin pelaamiani pelejä löytyy Linuxillekkin natiivina, kuten lapsuusvuosien suossikkini Transport Tycoon Deluxe. Pelistä on tehty käytännössä avoimen lähdekoodin (GPL) kopio, OpenTTD, joka vastaa vanhaa alunperin DOS:lla pyörinyttä Transport Tycoonin 'Deluxe' -versiota. Avoimen lähdekoodin versioon on lisätty paljon ominaisuuksia, modeja ja tietenkin suorituskykyparannuksia. Peli toimii natiivisti myös Windowsilla. Myös monet Half Life 2:n pelimoottoria käyttävät pelit, Half Life 1, HL2, Left 4 Dead, Team Fortress 2 ja Portal ovat toimineet Linuxilla vuodesta 2013.

#### Työpöydän etäohjaus (järjestelmäriippumaton) |
Windowsilla ja Linuxilla käytän Teamvieweriä, joka on ehdottomasti tämän hetken paras työkalu sekä Windows ja Linux työpöytien etäkäyttöön tai ohjaukseen. Ei-kaupallisessa käytössä Teamviewer on ilmainen. Teamvieweristä on saatavilla omat versiot käytännössä kaikille Linux-distroille. Kaikella hyvällä on kuitenkin huonot puolensa... Teamviewerin lähdekoodi on luonnollisesti täysin suljettua. 

Silloin tällöin käytän myös eri VNC-ohjelmia ja palvelimia, kuten TightVNC:tä ja TigerVNC:tä. Viimeaikoina olen kokeillut myös RDP:tä Linuxilla xRDP:llä. Teamviewer on kuitenkin edelleen kätevin ja tuskattomin, johtuen Teamviewerin palomuurien läpäisykyvystä. Samanlaista avoimen lähdekoodin versiota RDP:stä tai vaikka VNC:stä odotellessa...

### Lähteet |

[1]: https://www.asus.com/fi/Stick-PCs/QM1/specifications/
[2]: https://www.linuxquestions.org/questions/linux-hardware-18/asus-stick-pc-qm1-4175574279/#post5656074
[3]: https://www.chromium.org/developers/how-tos/api-keys
[4]: https://askubuntu.com/questions/225930/how-do-i-get-the-latest-beta-and-development-version-of-chromium
[5]: https://github.com/jomo/imgur-screenshot
[6]: https://www.spotify.com/fi/download/linux/
[7]: https://archive.is/20161008213028/https://wiki.winehq.org/FAQ%23Does_Wine_support_.NET.3F_Should_I_install_native_.NET_in_Wine.3F
[8]: http://www.javiersantos.me/post/82199053318/download-for-win-os-x-turn-back-to-the-previous-spotify
[9]: https://github.com/dylanaraps/neofetch
[10]: https://github.com/phw/peek
[11]: https://github.com/jml/undistract-me
[12]: https://www.sofokus.com/blogi/avoin-lahdekoodi/
[13]: https://www.gnu.org/philosophy/free-sw.html
[14]: https://www.gnu.org/licenses/license-list.html#Expat
[15]: https://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html

[1: https://www.asus.com/fi/Stick-PCs/QM1/specifications/

[2: https://www.linuxquestions.org/questions/linux-hardware-18/asus-stick-pc-qm1-4175574279/#post5656074

[3: https://www.chromium.org/developers/how-tos/api-keys

[4: https://askubuntu.com/questions/225930/how-do-i-get-the-latest-beta-and-development-version-of-chromium

[5: https://github.com/jomo/imgur-screenshot

[6: https://www.spotify.com/fi/download/linux/

[7: https://archive.is/20161008213028/https://wiki.winehq.org/FAQ%23Does_Wine_support_.NET.3F_Should_I_install_native_.NET_in_Wine.3F

[8: http://www.javiersantos.me/post/82199053318/download-for-win-os-x-turn-back-to-the-previous-spotify

[9: https://github.com/dylanaraps/neofetch

[10: https://github.com/phw/peek

[11: https://github.com/jml/undistract-me

[12: https://www.sofokus.com/blogi/avoin-lahdekoodi/

[13: https://www.gnu.org/philosophy/free-sw.html

[14: https://www.gnu.org/licenses/license-list.html#Expat

[15: https://linuxiumcomau.blogspot.com/2017/06/customizing-ubuntu-isos-documentation.html