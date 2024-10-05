# linux_ajout_memoire_a_chaud
Déroulé prise en compte d'ajout de mémoire virtuelle à chaud - serveur Linux

POC sur Debian 11

Consulter l'état des slot de mémoire actifs
```bash
cat /sys/devices/system/memory/memory*/state
  # output = liste uniquement l'état online/offline pour chaque fichier des dossiers parents memory1, memory2, memory39...
```
> Ex pour 4Go de RAM il y a environ 39 dossiers mémoire (à reconfirmer)

Ajout mémoire depuis superviseur (vsphere, ...)	

Consulter de nouveau l'état des slot de mémoire
```bash
cat /sys/devices/system/memory/memory*/state
  # output attendu = plus dossiers et donc des fichiers /state à offline, correspondant aux slots de mémoire virtuelle ajoutés
```
>	Ex pour 8Go de RAM, memory1 à memory71

Pour allouer manuellement l'espace mémoire au système (modifier fichier par fichier)
```bash
echo online > /sys/devices/system/memory/memoryXX/state
  # XX étant le numéro du slot mémoire actuellement à offline
```
Pour allouer tout l'espace mémoire disponible (modifier tous les fichiers memoryXX)

Créer un script .sh avec l'autorisation d'être exécuté
```bash
#!/bin/bash

# dossier cible
base_dir="/sys/devices/system/memory"

# boucle qui selectionne les fichiers avec l'arborescence all dossiers "memoryX" avec fichier "state"
for file in "$base_dir"/memory*/state; do

# si le fichier state contient "offline", alors on saisit "online" pour activer l'espace memoire
if grep -q "offline" "$file"; then
	echo online > "$file"
	echo "Fichier $file mis a jour"
fi

done
```
Exécuter le script avec root (owner des fichiers concernés)

Consulter de nouveau l'état des slot de mémoire

Si le swap était saturé avant l'ajout de mémoire, vider les fichiers
```bash
swapoff
swapon
```
