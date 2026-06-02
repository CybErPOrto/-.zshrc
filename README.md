# -.zshrc
 Elle te donne un petit workflow recon avec ports, services, urls et creds, en s’appuyant sur les sorties Nmap et sur des recherches de mots-clés utiles comme password, secret, token ou key



Ce que fait chaque fonction
ports extrait les ports ouverts et te sort un CSV prêt à réutiliser avec nmap -p.

services sort une vue lisible avec port/proto service version.

urls génère automatiquement les URLs web probables à partir des ports détectés.

creds cherche des indices de secrets ou de credentials dans un fichier ou un dossier.

Petite remarque utile
Nmap indique que le -sV sert à faire la détection de version, donc ta fonction services devient bien plus utile si tu pars d’un scan avec version detection. Et pour les recherches de creds, des motifs simples comme password, secret, token ou api_key sont une bonne base de départ.
