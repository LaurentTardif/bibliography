# elk 

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


## Monitoring 


### Common trap 

 A part ça les qq piège habituels de ES (ulimit à augmenter pour le user qui lance ES)
 - bien sizer sa mémoire 
 - augmenter le refreshInterval 
 - si on veut plusieurs instance faire attention que celles ci tournent bien sur des disques différent 
 - droits d’accès aux fichier de logs) 
 + bien gérer la rotation des index (en général un index par jour et on garde que les X derniers index ouverts)
 Dans les grosses volumétries : kafka + ELK => beaucoup de problématiques du à l’énorme volumétrie qu’ils gèrent
donc à prendre en compte : criticité des logs / volumétrie / rétention (durée, taille) / HA

- bien faire son mapping : bien souvent on n’a pas besoin de faire de l’analyse full text sur des logs (en tout cas pas sur tous les champs)
- étudier la possibilité d’utiliser beats / ingest node plutôt que logstash si on a des cas d’utilisation très simples (comme le tien) : ça évite de maintenir un logstash.
- plutôt que d’utiliser grok, il faut tenter le filter ‘dissect’, disponible dans logstash et en tant que pipeline processor, ça évite d’utiliser des regex et ça peut permettre d’atteindre des gains de perf d’un facteur proche de 10.

