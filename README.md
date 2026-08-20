# Windows Security Fundamentals — Reconnaissance Windows (HTB Academy)

## Contexte
Ce module fait partie du parcours "Windows Fundamentals" sur HTB Academy. 
Il couvre deux mécanismes essentiels pour un pentester ou un défenseur : 
les identifiants de sécurité (SID) qui gèrent les droits d'accès, et le 
Registre Windows qui contrôle notamment ce qui démarre automatiquement 
avec le système — un point d'entrée classique pour la persistance des 
attaquants.

## 1. Comprendre les SID (Security Identifier)

Chaque utilisateur et groupe Windows possède un SID unique, structuré ainsi :
`S-1-5-21-<SID du domaine/machine>-<RID>`

Le RID (dernier segment) est particulièrement utile en reconnaissance : 
`500` désigne toujours l'Administrateur, `501` l'invité, et `100x` les 
comptes utilisateurs classiques. Un attaquant qui repère un RID `500` 
sait immédiatement qu'il cible un compte à privilèges élevés.

**Commande utilisée (PowerShell) :**
​```powershell
Get-LocalUser -Name "bob.smith" | Select-Object Name, SID
​```

**Alternative en CMD (via WQL) :**
​```cmd
wmic useraccount where name='bob.smith' get name,sid
​```

J'ai testé les deux pour comparer : PowerShell est plus lisible en sortie, 
tandis que WMIC reste utile sur des systèmes plus anciens où PowerShell 
peut être restreint.

**Résultat :** SID de bob.smith → `S-1-5-21-2614195641-1726409526-3792725429-1002`
Le RID `1002` confirme qu'il s'agit d'un compte utilisateur standard, 
pas d'un administrateur.

## 2. Explorer le Registre pour la persistance

Le Registre stocke les clés de démarrage automatique dans :
`HKLM\Software\Microsoft\Windows\CurrentVersion\Run` (toute la machine)
`HKCU\Software\Microsoft\Windows\CurrentVersion\Run` (utilisateur courant)

C'est un des premiers endroits à vérifier en analyse forensique ou en 
recherche de malware, car de nombreux logiciels (légitimes ou non) 
s'y inscrivent pour démarrer avec Windows.

**Commande :**
​```cmd
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
​```

**Ce que j'ai trouvé :** une application de sécurité tierce (NordVPN) 
désactivée au démarrage pour l'utilisateur courant — un exemple concret 
de la façon dont on peut auditer rapidement ce qui tourne (ou ne tourne 
plus) au démarrage d'une machine.

![Résultat de la requête registre](https://github.com/user-attachments/assets/17425012-8195-47fa-a515-b2dd35e1dc3f)

## Ce que je retiens
Les SID et le Registre sont deux briques de base en reconnaissance 
Windows : les SID pour identifier rapidement le niveau de privilège 
d'un compte, le Registre pour comprendre ce qui persiste sur une 
machine. Ces deux réflexes reviennent constamment en pentest interne 
comme en réponse à incident.
