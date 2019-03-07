HTTP Headers - The Hidden Champions
======

Talk Slides: [https://schepp.github.io/HTTP-headers/](https://schepp.github.io/HTTP-headers/) 
Video: [Microsoft Technical Summit 2016 | HTTP/2 - die Zukunft beginnt jetzt](https://channel9.msdn.com/Events/microsoft-techncial-summit/Technical-Summit-2016/HTTP2-die-Zukunft-beginnt-jetzt)

---

Every call we send to and receive from a web server is accompanied by more or less metadata, also known as the HTTP headers. Usually, those headers stay hidden in the shadows and we barely notice them - and we underestimate their powers. But ignoring them is a mistake! There is a lot happening around them lately. During The last couple of years a lot of new and powerful headers have emerged which, when applied correctly, help us harden our web applications against attacks and make our sites load a lot faster. That's why in this talk, I'm gonna give you the modern take on how to make the web more secure & fast.

> Christian Schaefer (https://twitter.com/derSchepp), known as "Schepp", is a freelance frontend developer from Düsseldorf, Germany. Instead of hacking around with JS-Frameworks as almost every other frontend developer currently does, he works on traditional server-rendered component-based systems, uses bleeding edge CSS, has an eye on accessibility as well as the loading and runtime performance of a site. And then he also organizes a meetup (https://www.meetup.com/Webworker-NRW/) and co-hosts a podcast (https://workingdraft.de/).

---

Jeder HTTP-Request, den wir an einen Web Server senden oder den wir von einem solchen empfangen, wird begleitet von Metadaten, den HTTP Headern. Üblicherweise bekommen wir von diesen Headern gar nicht viel mit, weil sie sich unseren Blicken entziehen - und deshalb unterschätzen wir ihre Bedeutung im Web. Das allerdings ist ein großer Fehler! Tatsächlich entwickelt sich bei den HTTP Headern in den letzten Jahren mindestens so viel Neues wie im restlichen Web-Stack. Vornehmlich sind es dabei Security- und Performance-bezogene Header, die in den letzten Jahren neu hinzugekommen sind. Entsprechend wird es in meinem Talk um den Einsatz brandmoderner HTTP Headern gehen, mit denen man eine Webseite über das Normalmaß hinweg beschleunigt und sie gegen Angriffe härtet. 

> Christian Schaefer (https://twitter.com/derSchepp), auch "Schepp" genannt, ist freiberuflicher Frontend-Entwickler aus Düsseldorf. Anstatt mit hippen JavaScript-Frameworks herumzuspielen wie gefühlt sonst fast jeder, arbeitet er an traditionellen, serverseitig gerenderten Komponenten-Bibliotheken und nutzt dabei modernstes CSS, achtet auf Barrierefreiheit und eine rasend schnelle Ladezeit. Außerdem organisiert er ein Meetup (https://www.meetup.com/Webworker-NRW/) und podcastet über Frontend-Themen (https://workingdraft.de/).

---

![Avatar Picture](https://s.gravatar.com/avatar/7096dcb1690ef7418c4e94518f2fed31?s=200) 
 
