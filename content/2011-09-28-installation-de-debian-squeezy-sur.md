---

title: "Installation de debian squeezy sur eeepc 901 ok"
date: 2011-09-28
comments: true
---

<div class='post'>
<div dir="ltr" style="text-align: left;" trbidi="on">Je me suis amusé à installer la dernière debian sur mon eeePc dans la journée de hier. J'étais encore sur une version EasyPeasy qui commencait à se faire un peu vieille. J'ai donc décidé d'installer THE distribution sur la bête. Aucun soucis si vous avez une connexion internet via le câble ethernet.&nbsp; Pour ma part, j'ai utilisé unetbootin avec le premier cd d'installation. Tout se déroule bien sauf que le wifi ne fonctionne pas, malgrès toute les tentatives de configuration de wpa_supplicant (eh oui j'avais rien d'autre sous la main). Il faut donc trouver une connexion ethernet, rajouter les dépôts <b>non-free</b>, faire un petit <b>apt-get update</b> et installer le paquet <b>firmware-ralink</b>. Pour les non amateurs de&nbsp; la ligne de commande, je vous conseille d'installer un gestionnaire de connexion wifi (j'ai pris le <b>network-gnome-manager</b>). On redémarre la bête et hop le wifi fonctionne. Pour gérer l'hibernation et les spécificités du eeepc, installer <b>eeepc-acpi-scripts</b> et <b>cpufreq </b>pour la gestion du processeur depuis les applets gnome. </div></div>
