---
title: Linux palvelimet ICT4TN021 2017 H6
tags: ["linux","haaga-helia","koulu","infra","such","wow","test","tags"]
---

# Sisältö |
{:.no_toc}

* Table of contents placeholder
{:toc}
### Tehtävänanto |

Tehtävänanto on osa Tero Karvisen Haaga-Heliassa suoritettavaa Linux-kurssia. http://terokarvinen.com/2017/aikataulu-linux-palvelimet-ict4tn021-4-ti-ja-5-to-alkusyksy-2017-5-op

> a) Kirjoita ja suorita “Hei maailma” kolmella kielellä. Asenna tarvittavat ympäristöt.
> 
> b) Palauta linkki sivuun, josta löytyvät kaikki kotitehtäväraporttisi. Arviointi tehdään ensisijaisesti tästä linkistä. Linkki voi olla esimerkiksi blogin etusivu (jos blogissa on vain kotitehtävät) tai sivuun, jossa on linkki kuhunkin kuuteen tehtävään.

# A) Hello World - Hei maailma Go:lla, Ruby:lla ja Nodejs:llä

Valitsin tähän tehtävään kolme ohjelmointikieltä, joilla en ole aikaisemmin koskenut varsinaiseen lähdekoodiin. Javascriptin, Javan, PHP:n, c++ ja Bash:lla kanssa olen muokannut ja luonut omaa lähdekoodia aikaisemmin. Kaikki kolme kieltä ovat  etenkin web-sovelluksien tekijöiden suosiossa. Go ja Nodejs ovat näistä uusimpia. 

Ubuntu 16.04.3 versiolle löytyy kaikille kokeiltavista kielistä omat ohjelmavarastot.


#### Go | [1] [2]

Go on Googlen kehittämä moderni ohjelmointikieli.

Go:n voi asentaa Ubuntu 16.04-versioon komennolla `sudo apt-get update && sudo apt-get install golang`

Seuraavaksi voimme kirjoittaa luoda uuden sovelluksen komennolla `nano helloworld.go`

lisätään `helloworld.go -tiedoston` sisällöksi seuraava:

```
package main

import "fmt"

func main() {
     fmt.Printf("Hello Worlds!!\n")
}
```

Ennen ohjelman suorittamista se täytyy kääntää suorittamalla `build go`, jonka jälkeen voimme suorittaa ohjelman sen nimellä:

```
#Huom käännöksen jälkeen tiedostopäätettä .go ei enää ole, sillä ohjelma on nyt binäärimuodossa!
./helloworld
$ Hello Worlds!!
```



#### Ruby | [3]

Ruby on perinteisiin ohjelmointikieliin, kuten c:hen ja Assemblyyn verrattuna suhteellisen uusi ohjelmointikieli. Ruby nousi suurempaan suosioon vasta 10v ensimmäisen julkaistun version jälkeen vuonna 2005. Silloin julkaistiin internet-kehitysympäristö Ruby on Rails. [6]

```
sudo apt-get update
sudo apt-get install ruby
```

Lisätään tiedostoon `helloworld.rb` seuraavat ohjelmakoodi, joka perustuu Stackoverflow:n vastaukseen https://stackoverflow.com/a/705754/5776626 [6]:

```
class HelloWorld
   def initialize(name)
      @name = name.capitalize
   end
   def sayHi
      puts "Hello #{@name}!"
   end
end

hello = HelloWorld.new("Worlds!!")
hello.sayHi
```

Ohjelmakoodi suoritetaan yksinkertaisesti suorittamalla tiedostonnimi `ruby` -komennolla.

```
ruby helloworld.rb
$ Hello World!
```




#### Nodejs | [4] [5]

Nodejs on yksi suosituimmista palvelin

```
#Seuraavaa riviä ei tarvita, ellemme halua lisätä uusimpia Nodejs:n ohjelmavarastoja Ubuntuun.
#curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

sudo apt-get install -y nodejs
```

Koska Nodejs perustuu Javascriptiin voimme tehdä Hello World -ohjelman hyvin yksinkertaisesti tekemällä tiedoston helloworld.js komennolla `nano helloworld.js `

Tiedoston sisällöksi tarvitsemme vain `console.log('Hello World!!');`

Suoritus tapahtuu Ubuntussa komennolla `nodejs helloworld.js`

### Lähteet |

[1]:http://terokarvinen.com/2017/hello-go-world-install-and-run-go-in-less-than-a-minute-on-ubuntu-16-04-lts
[2]:https://github.com/golang/go/wiki/Ubuntu
[3]: https://stackoverflow.com/questions/705729/how-do-i-create-a-ruby-hello-world
[4]: http://nodeguide.com/beginner.html
[5]:https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
[6]:https://stackoverflow.com/a/705754/5776626
[7]: 
[8]: 
[9]: 
[10]: 
[11]: 
[12]: 
[13]: 
[14]: 
[15]: 

[1:http://terokarvinen.com/2017/hello-go-world-install-and-run-go-in-less-than-a-minute-on-ubuntu-16-04-lts
[2:https://github.com/golang/go/wiki/Ubuntu
[3: https://stackoverflow.com/questions/705729/how-do-i-create-a-ruby-hello-world
[4: http://nodeguide.com/beginner.html
[5:https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
[6:https://stackoverflow.com/a/705754/5776626