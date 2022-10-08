# UnitedCTF-2022 (track Desjardins)

Write-up for Desjardins Track at United CTF 2022.

D'abord, merci a tous les organisateurs du CTF ⛳🙌👍 Cette année encore fut une excellente année.

Je dois dire que je suis particulièrement fan des Tracks Desjardins dans les derniers CTF que j'ai fait. Ils nous proposent souvent des vrais challenges de Réponse aux Incidents pour lesquels on apprend beaucoup. Les Blue-teams sont parfois oubliés lors des CTFs... Mais pas ce coup-ci!

Cette année j'ai été servi sur le sujet. On avait un malware qu'on doit décortiquer pour comprendre et s'en protéger. On ira même jusqu'a se faire passer pour le serveur de commande et lui envoyer des fausses commandes de contrôle.

Évidemment, faire le Reverse Engineering complet du dit malware (avec IDA ou Ghidra, par exemple) n'était pas recommandé. 
On doit faire ça mode BlueTeam, avec Wireshark pis les logs de Firewall, a l'ancienne!!!

Voici la musique qui m'a accompagné pendant certaines étapes du défi :

Spiritbox - Circle With me https://www.youtube.com/watch?v=I0WzT0OJ-E0

Within Temptation (feat. AnnIsOkay) - Shed My Skin https://www.youtube.com/watch?v=mP0TVptLDQE



## Étape 1 : Un malware qui fait rien ?!
![chall-1](https://user-images.githubusercontent.com/16509773/194682871-b9bad6fd-0d38-4a47-bc24-d8f7747a86cc.jpg)

Lorsqu'on clique sur Download, on obtient un fichier "Malware.exe".
![chall-1-exe](https://user-images.githubusercontent.com/16509773/194682903-afede08f-5eba-4c60-9365-e92d11653250.jpg)

Lorsque le lance, on obtient une invite de commande pour laquelle il ne se passe rien. Elle dure environ 5 secondes et puis disparaît.
![chall-1-exe-malwarecmd](https://user-images.githubusercontent.com/16509773/194682911-20b751da-f002-4c8b-aea5-34240c101df7.jpg)

A ce moment, j'ai ouvert Process Monitor en pensant en voir plus. On voit dans celui-ci que Malware.exe lance un autre processus "Main.exe", mais rien qui me permet d'aboutir a un résultat concret n'est découvert. Il faut dire que j'ai eu des problèmes avec les appels réseaux dans Process Monitor, qui m'auraient potentiellement placé sur la bonne piste.

Je me suis dit, tôt ou tard, ce fichier malveillant devra parler a un serveur de Commande et Contrôle (C&C), donc lançons Wireshark...
Rien n'est trouvé sur wireshark au premier abord, j'ai beau virer les communications dans tous les sens. Rien n'y est.

Dans un moment de désespoir, je suis allé voir le reste du challenge (entre autre la partie 2). Dans cette section, il mentionné qu'avant de sortir sur Internet, le logiciel malveillant tente de "tester sa connexion". C'est alors que j'ai eu une illumination, en me disant qu'il pouvait tester aussi certaines connexions sur le poste lui-même ou, par exemple, vérifier le réseau du poste dans une phase d'identification.

C'est donc a ce moment que je décide d'ouvrir la "loopback Interface" de Wireshark. 
Celle-ci permet de voir des connexions sur 127.0.0.1 et nous sommes en mesure de trouver la réponse avec ces paquets.


## Étape 2 : Un test de connectivité, rien de plus facile
![Chall2](https://user-images.githubusercontent.com/16509773/194683427-64aef80c-23b6-4b02-8a33-690a797dc6ea.jpg)

Je dois avouer que cette étape a été la plus facile du parcours. Quoi de mieux pour trouver une connexion que BurpSuite ?
Je lance donc Burp Proxy et remarque que l'invite de commande "Malware.exe" reste désormais ouverte tant que je n'ai pas autorisé la connexion, un bel indice que je suis au bon endroit!!!


![burp80](https://user-images.githubusercontent.com/16509773/194683716-7e2e874d-10df-4373-bd19-05e75a24c1bc.jpg)

Nous avons donc découvert notre second Flag.


## Étape 3 : Une histoire de confiance

![Chall3](https://user-images.githubusercontent.com/16509773/194683774-50544d52-da95-45f4-93da-60d5e408479c.jpg)

Je dois dire que cette étape m'a fait galéré aussi. Je croyais qu'on devait instaurer le certificat de Google.ca comme certificat de confiance au départ. Je crois après quelques lectures sur le sujet que c'est probablement impossible et surtout que ça ne m'avançait a rien dans le contexte local.

Par contre, le certificat de BurpSuite lui peut être ajouté aux certificats machines en tant que "Root CA" pour permettre une confiance aveugle envers le proxy.
Je suis loin d'être un professionnel des certificats, mais en ayant ceci en place, ça me permettait aussi de décrypter le reste des communications et je savais que j'allais en avoir besoin.

Voir article : https://portswigger.net/burp/documentation/desktop/external-browser-config/certificate

En effectuant cette étape on voit désormais une nouvelle demande sur Google.ca, sur le port 443 et tel que décrit dans la partie #3 un nouveau URI 😈
Nous obtenons donc notre 3ième ⛳.


## Étape 4 : On est toujours proche du but quand on découvre qui est notre interlocuteur.

![Chall4](https://user-images.githubusercontent.com/16509773/194683927-528a39da-a56e-445d-a273-b09b04ee3343.jpg)

Les gens de sécurité défensive le savent, lorsqu'on découvre le domaine ou l'IP d'un serveur de commande et contrôle on connaît déja beaucoup d'information sur notre adversaire, on est désormais capable de le bloquer. 

Après le test de connexion précèdent, notre malware commence a être bavard... on a donc une autre connexion qui apparaît dans Burp.

![burpc2](https://user-images.githubusercontent.com/16509773/194684104-d2b85719-f8ea-4be3-9014-d818b8bf2c02.jpg)


## Étape 5 : Une lourde tâche...🪓 De lourds pouvoirs.
L'étape 5 de cette track était ma foi, surprenante, hilarante et surtout intéressante.

![Chall5](https://user-images.githubusercontent.com/16509773/194684394-15feddee-d0e8-4950-bb84-f27360a9701c.jpg)

De ce que j'en comprends nous devrons donc prendre contrôle du logiciel malveillant en se faisant passer par le serveur de C&C.
Je n'avais jamais pensé faire ça auparavant, mais c'est le genre d'un truc qu'on peut faire avec des proxy dans un Honeypot pour comprendre le fonctionnement d'un logiciel malveillant. J'ai trouvé cette partie plutôt intéressante, car en bout de ligne a la fin de cette étape on a monté un lab complet pour l'analyse dynamique.

Donc, j'ai commencé par lister ce qu'on avait et ce qui nous manquait : 
1) On possède l'URL nécessaire au fonctionnement. L'adresse IP derrière celle-ci importe peu.
                On peut donc potentiellement se faire passer pour ce domaine, en utilisant 127.0.0.1 et en changeant notre fichier 'hosts'.

Il nous manque : 
Un serveur web qui répond a ce domaine la réponse "None".

Étant donné que mon Lab est sur Windows, j'ai donc monté un serveur IIS et j'ai changé mon fichier hosts.
Changement du fichier Hosts : 
127.0.0.1 doit mener sur : "totallylegitdomain-notevil (.) cn"

Ouvrons donc un IIS qui répoond sur celle-ci. Le résultat devrait ressembler : 

![IIS1](https://user-images.githubusercontent.com/16509773/194684648-154f6614-ff6f-4b0c-91df-073770aab29c.jpg)

Un des défis que j'ai eu était de changer la page "HANDSHAKETEST.html" dans "wwwroot" pour que le .HTML disparaîsse.

Ma page s'appelait : HANDSHAKETEST.HTML, mais devait devenir HANDSHAKETEST pour que l'URL complet soit le suivant : totallylegitdomain-notevil (.) cn/handshaketest.

La réponse de cette page devait donner "None". Texte que j'ai seulement mis sur la page, dans "WWWROOT". 
Pour ce faire, j'ai ajouté une règle de Réécriture URL.
Plusieurs documentations sont disponibles sur Internet sur le sujet.

Lorsque toutes ces étapes sont une réussite, nous obtenons (enfin!) le flag final :
![finalflag](https://user-images.githubusercontent.com/16509773/194684865-acaad4aa-d508-434f-8067-7d783a793f5b.jpg)


## Small Recap & Quick Final Words

J'ai appris :

-Qu'avec un proxy et quelques manipulations faciles il serait plus facile que je pensais de monter un Honeypot ou sandbox d'analyse dynamique de malware.

-Qu'installer un IIS complet était probablement une solution trop complexe. Beaucoup d'autres solutions intelligentes étaient a portée de main.

Ce que j'ai préféré : 

-Je recommende les tracks Desjardins dans les CTF, car ils focus sur le côté défensif, mais je dois avouer que la dernière partie qui dans un monde réel nous permettrait de faire passer de fausses commandes a un malware a été particulièrement riche en émotions et un bon apprentissage pour moi.


Merci a tous les organisateurs du UnitedCTF 2022 encore un succès cette année🚩👌 (c'était super, en plus l'évènement est gratuit!)

Spécial thanks @Res260 pour le design de la track😍

Temps investi : + de 4h.

Vrai solution (plus rapide et géniale que la mienne) : https://github.com/UnitedCTF/UnitedCTF-2022/tree/master/challenges/desjardins/dynamic-malware-analysis 

-PeLouZe✌




