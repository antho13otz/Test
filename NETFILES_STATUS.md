# Etat du module reseau GC-MP3

## Ce qui est present dans cette revision

- `source/netfiles.h` / `source/netfiles.c` : API backend FTP RFC 959, incluse dans la compilation de `source/main.c`.
- FTP : connexion par IPv4 numerique, login USER/PASS, mode binaire, liste `LIST` au format Unix, changement de dossier, telechargement RETR, envoi STOR, renommage RNFR/RNTO, suppression DELE, creation MKD et suppression de dossier RMD.
- Mode passif PASV, transfert par blocs, controles des arguments CR/LF et messages d'erreur.
- `tests/run_netfiles_host_test.sh` : test hote de compilation du module et du contrat de disponibilite des protocoles.

## Limites importantes (ne pas confondre avec une implementation complete)

- Cette revision n'integre pas encore un ecran GX/Indigo permettant de configurer et piloter les sessions depuis l'onglet FICHIERS. Le backend FTP est compile dans l'unite de traduction principale, mais ses fonctions ne sont pas encore appelees par l'interface.
- FSP, SMB et SFTP ne sont pas implementes : `nf_protocol_available()` retourne volontairement faux pour ces protocoles. Aucun de ces protocoles n'est presente comme fonctionnel.
- FTP envoie les identifiants en clair. Utiliser uniquement sur un reseau de confiance. Pour du chiffrement, il faut FTPS explicite ou SFTP, qui n'est pas fourni ici.
- Le parseur LIST vise les sorties Unix courantes; les serveurs FTP qui fournissent un format DOS ou non standard peuvent necessiter MLSD ou un parseur additionnel.
- Les appels sont synchrones et bloquants. Ils doivent etre executes depuis un thread de travail avant toute integration dans la boucle GX/audio.

## Tests executes dans l'environnement disponible

- `./tests/run_netfiles_host_test.sh` : passe (compilation hote avec une interface reseau simulee, assertions sur les protocoles disponibles).
- `make` cible GameCube : non execute jusqu'a la compilation, car l'environnement actuel ne contient pas `DEVKITPPC` / devkitPPC. Le Makefile s'arrete explicitement sur cette dependance.

Ce commit est une base backend FTP testee, pas une livraison des quatre protocoles et de l'interface complete.


## Serveur local FTP (ajout)

- `source/netserver.c` / `source/netserver.h` : serveur FTP local démarré après `if_config()` si la carte SD est montée.
- Port contrôle TCP 2121, mode passif 50000–50007. Identifiants par défaut `gc-mp3` / `indigo`; FTP non chiffré, réseau local uniquement.
- Opérations implémentées : USER/PASS, PWD/CWD/CDUP, PASV, LIST/NLST, RETR/STOR, SIZE, RNFR/RNTO, DELE, MKD/RMD, QUIT/NOOP/SYST/TYPE. La racine est `sd:/`; les chemins `..`, les changements de volume et les antislashs sont refusés.
- `tests/run_ftp_integration_test.py` démarre une version hôte avec adaptateur sockets POSIX et vérifie une session FTP réelle : login, liste, upload/download binaire, renommage, suppression, création/suppression de dossier et rejet des mauvais identifiants.
- FSP et SMB ne sont pas implémentés côté serveur. Ne pas présenter ces protocoles comme disponibles.
- La cible GameCube n'a pas été compilée dans cet environnement : `DEVKITPPC` / devkitPPC est absent. Le test hôte ne remplace pas une compilation GameCube ni un essai sur BBA + carte SD réels.
