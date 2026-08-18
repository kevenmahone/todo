ARCHITECTURE TAILS + OPNSENSE + VPN

OBJECTIF
Créer un ordinateur dédié exclusivement à Tails, avec le minimum de matériel et de stockage permanent possible, connecté à Internet uniquement à travers un pare-feu/routeur VPN.

1. ORDINATEUR DÉDIÉ À TAILS

- Ordinateur utilisé uniquement pour Tails.
- Pas de disque dur/SSD interne si le matériel le permet.
- Aucune donnée personnelle stockée sur l'ordinateur.
- Webcam retirée.
- Microphone retiré.
- Carte Wi-Fi/WLAN retirée.
- Bluetooth retiré.
- Haut-parleurs retirés.
- Connexion réseau uniquement par Ethernet.
- Si l'ordinateur n'a pas de port Ethernet, utiliser un adaptateur USB-Ethernet compatible avec Tails.
- Ne pas utiliser de périphériques USB inconnus ou non fiables.

2. TAILS SUR DVD-R

- Utiliser l'image officielle de Tails.
- Vérifier l'authenticité de l'image Tails avant de la graver.
- Graver l'image sur un DVD-R.
- Un DVD-R finalisé est un support en lecture seule : Tails ne peut normalement pas modifier son contenu.
- Le DVD-R permet donc d'avoir un support système qui ne peut pas être modifié pendant l'utilisation.

IMPORTANT :

- La vérification cryptographique officielle de Tails est préférable à une simple comparaison SHA-256 du DVD.
- Un SHA-256 identique indique seulement que les données lues sont identiques ; il ne prouve pas à lui seul que l'image originale était authentique.

3. VÉRIFICATION SHA-256 DU DVD

Pour identifier le périphérique :

lsblk

ou :

lsblk -f

Le lecteur DVD peut par exemple apparaître comme :

/dev/sr0

Pour calculer le SHA-256 du contenu lu depuis le DVD :

sudo dd if=/dev/sr0 bs=4M status=progress | sha256sum

Pour enregistrer le résultat dans un fichier :

sudo dd if=/dev/sr0 bs=4M status=progress | sha256sum | tee dvd-sha256.txt

Pour vérifier que le fichier existe :

ls -l dvd-sha256.txt

Pour afficher son contenu :

cat dvd-sha256.txt

Lors d'une utilisation ultérieure, recalculer le SHA-256 et comparer les deux valeurs.

IMPORTANT :
Le fichier dvd-sha256.txt doit lui-même être conservé sur un support fiable si l'on veut pouvoir comparer le résultat plus tard. Pour Tails, utiliser de préférence la procédure officielle de vérification cryptographique de Tails.

4. MÉMOIRE RAM ET BATTERIE

- Retirer la batterie peut réduire certains risques liés à un ordinateur portable hors tension.
- Cependant, retirer la batterie ne garantit PAS l'effacement instantané de la RAM.
- La DRAM perd normalement son contenu lorsqu'elle n'est plus alimentée, mais certaines données peuvent persister pendant un certain temps.
- Un retrait de batterie n'est donc pas une protection absolue contre les attaques de type cold-boot.
- Tails utilise également des mécanismes destinés à réduire les données sensibles laissées en mémoire lors de l'arrêt.

5. ROUTEUR / PARE-FEU

Utiliser un appareil dédié comme :

Protectli Vault FW2B

Configuration envisagée :

Internet
|
v
[ Routeur/ONT Internet ]
|
v
[ Protectli FW2B ]
|
|-- OPNsense
|-- VPN WireGuard
|-- Pare-feu
|-- Kill switch réseau
|-- DNS sécurisé
|
v
Ethernet
|
v
[ Ordinateur Tails ]
|
v
[ Tails + Tor Browser ]

6. PROTECTLI FW2B

Configuration matérielle souhaitée :

- Protectli Vault FW2B.
- Pas de Wi-Fi/Bluetooth intégré ou périphériques radio inutiles.
- Installer OPNsense comme système d'exploitation du pare-feu.
- Utiliser uniquement les fonctions réseau nécessaires.
- Le FW2B possède normalement un stockage interne : ce n'est donc PAS un appareil totalement sans stockage.
- Le stockage interne du FW2B doit être considéré séparément du stockage de l'ordinateur Tails.

Site officiel :
https://protectli.com/product/fw2b/

7. OPNsense

Installer OPNsense sur le Protectli FW2B.

Objectif :

Internet
|
v
OPNsense
|
+--> VPN WireGuard
|
+--> Firewall
|
+--> DNS
|
v
Tails

Configurer le pare-feu pour empêcher le trafic direct vers Internet si le tunnel VPN est indisponible.

Principe :

VPN UP
-> trafic autorisé

VPN DOWN
-> trafic Internet bloqué

C'est le principe du kill switch.

8. IPV6

Si IPv6 n'est pas correctement configuré dans toute l'architecture :

- Désactiver IPv6 sur OPNsense.
- Vérifier également qu'aucun périphérique ne peut contourner le VPN via IPv6.
- Tester depuis Tails qu'aucune adresse IPv6 publique n'est exposée.

IMPORTANT :
Il est préférable de comprendre et tester la configuration IPv6 plutôt que de simplement la désactiver sans vérification.

9. VPN

Configurer le VPN directement sur OPNsense, par exemple avec WireGuard.

Architecture :

Internet
|
v
OPNsense
|
v
WireGuard VPN
|
v
Tor
|
v
Internet

Le VPN et Tor ont des fonctions différentes :

VPN :

- chiffre la connexion entre le réseau local et le serveur VPN ;
- masque l'adresse IP du réseau local au serveur Tor d'entrée dans certaines architectures.

Tor :

- fournit l'anonymisation en plusieurs relais ;
- Tails est conçu pour faire passer les applications compatibles par Tor.

IMPORTANT :
Un VPN ne rend pas automatiquement anonyme. Il ajoute une couche réseau supplémentaire, mais il faut toujours considérer les métadonnées, la configuration et les erreurs d'utilisation.

10. TAILS + TOR

Sur l'ordinateur :

[ Tails ]
|
v
[ Tor ]
|
v
[ OPNsense ]
|
v
[ WireGuard VPN ]
|
v
[ Internet ]

Utiliser le Tor Browser fourni par Tails.

Pour une sécurité maximale dans Tor Browser :

- Utiliser le niveau de sécurité le plus élevé compatible avec les besoins.
- Le niveau de sécurité élevé peut désactiver JavaScript ou certaines fonctionnalités.
- Ne pas installer d'extensions inconnues.
- Ne pas ouvrir des fichiers provenant de sources non fiables.
- Ne pas mélanger des identités personnelles et anonymes.
- Ne pas révéler volontairement d'informations permettant de relier différentes identités.

11. SURICATA / IDS / IPS

OPNsense peut être configuré avec un système IDS/IPS comme Suricata.

Objectif :

- détecter certaines activités réseau suspectes ;
- générer des alertes ;
- éventuellement bloquer certains trafics.

Mais Suricata n'est pas obligatoire pour faire fonctionner Tails + Tor + VPN.

À retenir :

- Tor chiffre une grande partie du trafic entre les relais.
- Un IDS placé devant Tor ne peut donc pas forcément inspecter le contenu final du trafic Tor.
- Il peut néanmoins détecter certains comportements réseau, scans, connexions suspectes et signatures réseau.



13. ARCHITECTURE FINALE
    
                INTERNET
               |
               v
      [ Routeur / ONT ]
               |
               v
      [ Protectli FW2B ]
               |
         [ OPNsense ]
               |
      +--------+--------+
      |                 |
  Firewall         WireGuard VPN
      |                 |
      +--------+--------+
               |
            Ethernet
               |
               v
      [ Ordinateur Tails ]
               |
          [ Tails OS ]
               |
            [ Tor ]
               |
               v
           INTERNET

CARACTÉRISTIQUES DE L'ORDINATEUR TAILS :

- Pas de stockage interne si possible
- Pas de webcam
- Pas de microphone
- Pas de Wi-Fi
- Pas de Bluetooth
- Pas de haut-parleurs
- Ethernet uniquement
- Tails démarré depuis DVD-R
- Aucune donnée personnelle stockée localement

OBJECTIF GÉNÉRAL :

Réduire au maximum :

- les données persistantes ;
- les interfaces radio ;
- les périphériques inutiles ;
- les possibilités de modification du système Tails ;
- les connexions réseau directes ;
- les fuites IPv6/DNS ;
- les erreurs de configuration réseau.

LIMITES IMPORTANTES :

Cette architecture n'offre pas une « invisibilité » garantie.

Elle ne protège pas automatiquement contre :

- les erreurs humaines ;
- les comptes utilisés avec une identité réelle ;
- les logiciels malveillants exploitant une vulnérabilité inconnue ;
- les attaques matérielles avancées ;
- les compromissions du routeur ;
- les attaques contre le fournisseur VPN ;
- les corrélations de trafic ;
- les attaques physiques sophistiquées ;
- les informations volontairement révélées par l'utilisateur.

Le principe le plus important reste : Tails, Tor, un VPN et OPNsense sont des couches de sécurité différentes ; aucune de ces couches ne doit être considérée comme une garantie absolue d'anonymat.






usb kill to block usb to be plug or unplug in tails if os detect something its shutdown pc

https://github.com/hephaest0s/usbkill


or 

https://github.com/NateBrune/silk-guardian










For Looking a video (movie or hacking guide )
with potential virus inside

do like that 

you open tails fallow this tutorial here you download a few video you remove ethernet cable you look the video after 
shutdown tails correctly 
and wait 30 min the computer need to be unplug and no baterry.



if the virus hack the bios or router they will get persistence but is rare











clean btc to buy vpn like mulvad vpn

https://b1exch.to/

http://exchanger.rjocosf2mfgkrlsrr5k52nb4lbldhiazajqcjtr35w2ulumlbggnmmad.onion/


fairtrade website use swap to have monero
http://fairfffoxrgxgi6tkcaxhxre2hpwiuf6autt75ianjkvmcn65dxxydad.onion/



http://xmxmrjoqo63c5notr2ds2t3pdpsg4ysqqe6e6uu2pycecmjs4ekzpmyd.onion/


these link as been found in 

https://daunt.link/




anonymous reloadable cards with 3D secure
visa pre pay card pay with crypto
https://2fiat.com/




to try

https://orangefren.com/

> https://stealthex.io
> https://changenow.io
> https://godex.io






deepweb search engine
http://deeprecyrsonacndoosu3udqp7ziofjddoiq6grsfizp3m3mvbiinpad.onion/category/forums
