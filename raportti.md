#IRC -serveri Torin takana

###Tavoite
Projektin tavoite oli asentaa Banana-pi:lle käyttöjärjestelmä, tor hidden service ja asentaa unrealircd kuuntelemaan sen porttia.

###Bananian konfigurointi

Ensiksi ajoimme Banian interaktiivisen alkukonfiguroinnin.

```
Bananian-config
```
Alkuun vaihdoimme Root -salasanan ja valitsimme timezoneksi Helsingin.

Näppäimistöksi valitsimme Package Configuration - fi_FI.UTF-8 UTF-8 eli suomenkielisen näppäimistön.

Järjestelmän kieleksi valitsimme englannin en_US.UTF-8.

Hostnameksi määritimme bananapi.

Jätimme hardware configuraationin BananaPi:ksi.

Lisäksi laajensimme Rootin tiedostojärjestelmää.

Lopulta käynnistimme tietokoneen uudestaan.

```
shutdown -r now
```

###Torin asennus

Aloitimme ajamalla komennot:
```
apt-get update ja apt-get upgrade
```
Tämän jälkeene asensimme Torin apt-get komennolla.
```
apt-get install tor 
```
Aloitimme ensiksi asentamaan irkkiservua root-käyttäjälle, mutta tajusimme ettei se ole järkevä ratkaisu. Jos irc-serverissä olisi ollut jotain aukkoja ja se pyörisi rootilla,
Hyökkääjä pystyisi esim jo pelkällä puskurin ylivuodolla ajamaan omaa koodiansa ja ottamaan koneen haltuun. Sen takia teimme admin nimisen käyttäkän irc-serverin ajamista varten.

```
root@bananapi ~ # adduser admin
Adding user `admin' ...
Adding new group `admin' (1000) ...
Adding new user `admin' (1000) with group `admin' ...
Creating home directory `/home/admin' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for admin
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] Y
```

Tämän jälkeen siirsimme unrealircd-paketin adminin homekansioon ja annoimme kaikille oikeudet siihen.
```
root@bananapi ~ # mv unrealircd-4.0.3.1.tar.gz /home/admin/
root@bananapi ~ # chmod 777 /home/admin/unrealircd-4.0.3.1.tar.gz
root@bananapi ~ # su admin
admin@bananapi:/root$
```

Loimme kansion jonne torin konfiguraatioiden pitäisi mennä ja määrittelimme torin konffifiluun että se kuuntelee irkkiporttia 6667. Konffifiluun määriteltiin myös juuri luotu kansio.
```
#mkdir /home/admin/torri    
#nano /etc/tor/ torrc
#HiddenServiceDir /home/admin/torri    
#HiddenServicePort 6667 127.0.0.1:6667 (irkkiportit)
```

Torin buuttaus, jotta uudet konfiguraatiot otettaisiin käyttöön. Ei onnistu.

/etc/init.d/tor restart

[FAIL] Stopping tor daemon...failed (/usr/bin/tor died: process 17496 not running; or permission denied).
[ ok ] Starting tor daemon...done.


Katsotaan logeista mistä mättää. Ilmeisesti juuri luotu kansio ei kelpaa, koska se ei ole debian-torin omistuksessa.

less /var/log/tor/log

Apr 26 19:07:55.000 [warn] /home/admin/torri is not owned by this user (debian-tor, 106) but by root (0). Perhaps you are running Tor as the wrong user?


Luodaan siis homeen kansio debian-tor ja laitetaan se käyttäjän debian-torin omistukseen ja ryhmään.


root@bananapi:/home# mkdir debian-tor
root@bananapi:/home# chown debian-tor debian-tor
root@bananapi:/home# chgrp debian-tor debian-tor


Konffataan hiddenservice-kansioksi tuo juuri tehty debian-tor kansio.

root@bananapi:/home# nano /etc/tor/torrc

HiddenServiceDir /home/debian-tor/
vaihdetaan konffeihin kansioksi debian-tor


Buutataan tori uudestaan ja tällä kertaa se onnistuu hyvin. Käydään katsomassa uudesta hostname tiedostosta torverkon hostnamemme joka on vihk2qce7xi4lhc7.onion 

root@bananapi:/home# /etc/init.d/tor restart

[ ok ] Stopping tor daemon...done.
[ ok ] Starting tor daemon...done.

less /home/debian-tor/hostname

puretaan tarilla unrealircd paketti ja asennetaan libssl-dev ssl:ää vartem

tar -xzvf unrealircd-4.0.3.1.tar.gz
root@bananapi ~ # apt-get install libssl-dev       



Aloitetaan konffaaminen

./Config


Do you want to generate an SSL certificate for the IRCd?
Only answer No if you already have one.
[Yes] -> yes
Generating certificate request ..
/usr/bin/openssl req -new \
              -config src/ssl.cnf -sha256 -out server.req.pem \
              -keyout server.key.pem -nodes
Generating a 4096 bit RSA private key


Laitetaan ssl mukaan salakuuntelun ehkäisemiseksi.

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name [US]:.
State/Province [New York]:.
Locality Name (eg, city) []:.
Organization Name (eg, company) [IRC geeks]:.
Organizational Unit Name (eg, section) [IRCd]:.
Common Name (Full domain of your server) []:vihk2qce7xi4lhc7.onion 

Skipataan kaikki muu, mutta laitetaan domainiksi onion osoitteemme


Käännetään irkkiservu lähdekoodista

admin@bananapi:~/unrealircd-4.0.3.1$ make


Koitetaan käynnistää unrealircd.

admin@bananapi:~/unrealircd-4.0.3.1$ ./unrealircd start
Starting UnrealIRCd

The configuration file does not exist (/home/admin/unrealircd/conf/unrealircd.conf).
* Create one by following:
  https://www.unrealircd.org/docs/Installing_from_source#Creating_a_configuration_file
* Or if you are upgrading from version 3.2.x then read:
  https://www.unrealircd.org/docs/Upgrading_from_3.2.x and
  https://www.unrealircd.org/docs/UnrealIRCd_files_and_directories
admin@bananapi:~/unrealircd-4.0.3.1$

Konffifilua ei löydy, onneksi esimerkki konffifilu löytyi, joten siirsimme sen /unrealircd/conf kansioon nimellä unrealircd.conf 


admin@bananapi:~/unrealircd-4.0.3.1$ cp doc/conf/examples/example.conf ../unrealircd/conf/unrealircd.conf

Serverin käynnistys uudestaan, valittelee oletuskonfiguraatiosta.

admin@bananapi:~/unrealircd-4.0.3.1$ ./unrealircd start

Loading IRCd configuration..
config error: /home/admin/unrealircd/conf/unrealircd.conf:144: please change the the name and password of the default 'bobsmith' oper block
config error: /home/admin/unrealircd/conf/unrealircd.conf:378: set::cloak-keys: (key 2) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /home/admin/unrealircd/conf/unrealircd.conf:379: set::cloak-keys: (key 3) Keys should be mixed a-zA-Z0-9, like "a2JO6fh3Q6w4oN3s7"
config error: /home/admin/unrealircd/conf/unrealircd.conf:376: set::cloak-keys: All your 3 keys should be RANDOM, they should not be equal
config error: /home/admin/unrealircd/conf/unrealircd.conf:386: set::kline-address must be an e-mail or an URL
config error: 5 errors encountered
config error: IRCd configuration failed to pass testing

vaihdetaan nanolla nämä kohdilleen unrealircd.conffiin

oper testi {
        class opers;
        mask *@*;
        password "akjsdkasdjkasjdkakjsdjkasdjk!!!23@";
        /* Oper permissions are defined in an 'operclass' block.
         * See https://www.unrealircd.org/docs/Operclass_block
         * UnrealIRCd ships with a number of default blocks, see
         * the article for a full list. We choose 'netadmin' here.
         */
        operclass netadmin;
        swhois "is a Network Administrator";
        vhost netadmin.mynet.org;
};

3 muutakin kohtaa kohdilleen. Turhia juttuja, koska liittyivät servereiden linkkaamiseen keskenään

Testataan irssillä että se toimii connectaamalla localhostiin eli avattiin irssi ja /connect localhost

Testattiin myös torin kautta toiselta koneelta ja toimii hyvin. Tori osaa reitittää liinketeen natin ohi ilman mitään portforwardingeja.

Ajattelimme että serveriä voi käyttää vain ssl:ällä, joten konffataan hiddenserviceportiksi ircserverin ssl portti

nano /etc/tor/torrc
HiddenServicePort 6697 127.0.0.1:6697





5 !irc.foonet.com *** Y muista vaihtaa conffifiluun

poistetaan turhat linkit unrealin conffeista ja lisäillään tarpeellisia konffeja

#link hub.mynet.org
#{
#       incoming {
#               mask *@something;
#       };
#
#       outgoing {
#               bind-ip *; /* or explicitly an IP */
#               hostname hub.mynet.org;
#               port 6900;
#               options { ssl; };
#       };
#
#       password "00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF"; /* SSL fingerprint of other server */
#
#       class servers;
#};

me {
        name "vihk2qce7xi4lhc7.onion";
        info "haaga-helia 2016";
        sid "001";
};
/* The admin { } block defines what users will see if they type /ADMIN.
 * It normally contains information on how to contact the administrator.
 */


admin {
        "oppilas";

};

pois koska emme ole linkkaamassa servua.

#class servers
#{
#       pingfreq 60;
#       connfreq 15; /* try to connect every 15 seconds */
#       maxclients 10; /* max servers */
#       sendq 5M;
#};



pois koska allowattu kaikki toisessa conffissa

/* Example of a special allow block on a specific IP:
 * Requires users on that IP to connect with a password. If the password
 * is correct then it permits 20 connections on that IP.
 */
#allow {
        ip *@192.0.2.1;
        class clients;
        password "somesecretpasswd";
        maxperip 20;
};


vaihdettiin maski siten että mistään muualta kuin localhostista ei voi olla oper vaikka salasana olisi tiedossa.
oper testi {
        class opers;
        mask *@localhost;
        password "akjsdkasdjkasjdkakjsdjkasdjk!!!23@";
        operclass netadmin;
        swhois "is a Network Administrator";
        vhost linuxprojekti;
};


pois koska vain ssl

/* Standard IRC port 6667 */
listen {
        ip *;
        port 6667;
};



pois koska ei linkkejä
/* Special SSL/TLS servers-only port for linking */
listen {
        ip *;
        port 6900;
        options { ssl; serversonly; };
};


pois ei haluta linkkejä

link hub.mynet.org
{
        incoming {
                mask *@something;
        };

        outgoing {
                bind-ip *; /* or explicitly an IP */
                hostname hub.mynet.org;
                port 6900;
                options { ssl; };
        };

        password "00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF"; /* SSL fing$

        class servers;
};






Vaihdettiin networknamet ja defaultserverit kuntoon.

set {
        network-name            "Linuxprojektiserver";
        default-server          "vihk2qce7xi4lhc7.onion";
        services-server         "vihk2qce7xi4lhc7.onion";
        stats-server            "vihk2qce7xi4lhc7.onion";
        help-channel            "#Help";
        hiddenhost-prefix       "Clk";
        prefix-quit             "Quit";








pois ei linkkejä


/* U-lines give other servers (even) more power/commands.
 * If you use services you must add them here.
 * NEVER put the name of a (normal) UnrealIRCd server here!!!
 * ( If you wonder what Services are then see
 *   https://www.unrealircd.org/docs/Services )
 */
ulines {
        services.mynet.org;
};

Tehdään message of the day tiedosto ja rehashataan konffit servuun

admin@bananapi:~/unrealircd/conf$ nano ircd.motd

admin@bananapi:~/unrealircd$ ./unrealircd rehash









Laitetaan vielä unrealircd ajamaan cloak moduulia, joka piilottaa käyttäjien hostnamet ja ip:t

admin@bananapi:~/unrealircd/conf$ nano unrealircd.conf

loadmodule "cloak";



Valitettavasti ssl, yhdistäminen ei toiminut irssin avulla, joten konffasimme torin ja unrealicd:n kuuntelemaan myös porttia 6667. Xchat osasi yhdistää ssl:ällä myös.

servun käynnistys admin käyttäjällä

 ../unrealircd restart




