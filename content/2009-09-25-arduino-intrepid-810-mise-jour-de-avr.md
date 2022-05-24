---

title: "Arduino Intrepid 8.10 - Mise à jour de avr-gcc"
date: 2009-09-25
comments: true
categories:
 - Non classé
 - Linux
 - Bug
 - Arduino
 - ubuntu
---

<div class='post'>
Sur le site d'Arduino , on nous spécifis que la version de avr-gcc livré avec la 8.10 de ubuntu est buggé et on vous explique comment corriger le tir en utilisant les paquets de la version 9.04. Sur la version française, une erreur apparaît (en tout cas chez moi)&nbsp; et elle semblerait lié au traducteur en français. La manipulation décrite ici peut engendrer des problèmes (mise à jour de la libc6).<br /><br />Dans un terminal :<br /><pre><span style="font-size: xx-small;">$&gt; LANG=en_GB</span></pre><span style="font-size: xx-small;"><br /></span><br /><pre><span style="font-size: xx-small;">$&gt; sudo synaptic</span></pre><br /><br />Ajouter les deux  dépots suivants :<br /><pre><span style="font-size: xx-small;">deb http://us.archive.ubuntu.com/ubuntu/ jaunty main restricted</span></pre><span style="font-size: xx-small;"><br /></span><br /><pre><span style="font-size: xx-small;">deb http://us.archive.ubuntu.com/ubuntu/ jaunty universe </span></pre><br /><br />Rechargez les paquets et installez les paquets gcc-avr et avr-libc  et supprimez les dépots après l'installation.</div>
