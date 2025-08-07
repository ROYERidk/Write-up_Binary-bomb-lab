# Binary bomb lab write-up

Ce programme exécutable ouvre le fichier en argument, lit une ligne par phase (6 phases), et traite chaque ligne pour désamorcer chaque phase.

- Ligne 1 → désamorce la phase 1  
- Ligne 2 → désamorce la phase 2  
- etc...

**Prérequis :**  
- `defuse.txt`  
- `gdbcfg`  

---

## Sommaire

- [Phase 1](#phase-1)
- [Phase 2](#phase-2)
- [Phase 3](#phase-3)
- [Phase 4](#phase-4)
- [Phase 5](#phase-5)
- [Phase 6](#phase-6)
- [Secret Phase](#secret-phase)

---

## Phase 1

Prends la ligne 1 :  
`I am just a renegade hockey mom.`

---

## Phase 2

Prends la ligne 2 :  
Tableau de 6 int : `1 2 4 8 16 32`

Le programme fait appel à une fonction `read_six_numbers` qui met les 6 chiffres entrés dans un tableau nommé `local_38` (sous Ghidra). On assigne la même adresse à `piVar1`, donc on traite le même tableau.

- **Première valeur du tableau doit valoir 1 :**
    ```c
    if (local_38[0] != 1) { explode_bomb(); }
    ```
- **Deuxième valeur du tableau doit valoir 2 :**
    ```c
    if (piVar1[1] != *piVar1 * 2) { explode_bomb(); }
    ```
    (la deuxième valeur doit être le double de la première)

- **Suite de la logique :**
    - On incrémente `piVar1` et on continue tant qu'on n'est pas à la fin du tableau.
    - Chaque nombre suivant doit être le double du précédent (tableau à 6 chiffres).

---

## Phase 3

Ligne 3 : `4 0`

- Prend la 3ème ligne → 2 arguments : uint, int (sinon explosion)
- Utilise un switch/case sur le premier élément (uint)
- Conditions : `(uint < 5)` et `(int = calcul fait par le switch/case)`

Explication :
- Si on prend 4 comme premier argument, on tombe dans le case 4, et sans break, on continue 4 → 5 → 6 → 7.
- Cela donne `eax = 0 + 0x7e - 0x7e + 0x7e - 0x7e = 0`.
- Donc la troisième ligne doit être :  
  `4 0`

---

## Phase 4

**Fait avec seulement GDB**

- Premier argument passé à la fonction `func4` (dans edi → rdi)
- RAX commence toujours à `0xe (=14)`
- Exemple d'utilisation avec l'argument 8
- Plusieurs opérations, appels récursifs, et comparaisons sont faits.

**Objectif :**
- Après la fonction, on compare directement `eax` à 10. Si `eax != 10`, explosion.
- Ensuite, on vérifie si le deuxième argument vaut `0xa (=10)`.

**Solution :**  
`3 10`

---

## Phase 5

2 int en arguments sinon explosion.  
On nomme les deux arguments respectivement `a` et `b`.

- Un ET logique est fait entre `a` et `0xf` (modulo 16, seulement les 4 bits de poids faible).
- `a` ne doit pas être 15 (`0xf`), sinon explosion. Possibilités : `[0;14]`.
- Deux variables initialisées à 0 : `x` et `y`.
- Boucle : tant que `a != 0xf`, on incrémente `y`, `a = tab[a]`, puis `x = x + a`.
- Il faut que `y == 0xf (=15)` et `b == x` après la boucle.

**Valeurs du tableau à chaque index :**  
![image](https://github.com/user-attachments/assets/36ac8623-3826-43ef-ae11-1e36f774293a)

**Explication :**
- On doit faire 15 fois la boucle à l’envers en partant de 0xf pour savoir à quelle valeur démarrer :

```
5 (0x12) → 12 (0x3) → 3 (0x7) → 7 (0x11) → 11 (0x13) → 13 (0x9) → 9 (0x4) → 4 (0x8) → 8 (0x0) → 0 (0xa) → 10 (0x1) → 1 (0x2) → 2 (0xe) → 14 (0x6) → 6 (0xf)
```

- La solution est donc :  
  `5 115`

*Note : la première valeur (ici 5) n’est pas ajoutée à la somme finale, elle sert seulement de point de départ.*

---

## Phase 6

Les 6 arguments sont dans `R13`.

- Première étape : vérifier qu’on ne met pas deux fois le même numéro dans les 6 arguments, et qu’ils sont tous < 7.
- Ensuite, le programme va chercher la valeur du noeud associé à chaque argument dans l’ordre donné.

**Image des noeuds :**  
![image](https://github.com/user-attachments/assets/a644c6a3-b315-46ac-9c28-083bb7f61c0e)

**Solution (ordre décroissant des noeuds) :**  
`5 4 3 1 6 2`

---

## Secret Phase

- Il faut ajouter `DrEvil` à la fin du solve de la phase 4 pour accéder à la phase secrète.
- Elle attend une string qui sera transformée en int (< 1000).
- Une fonction `fun7` est appelée, il faut que son retour soit `5` pour désamorcer la phase.

**Explication du fonctionnement :**

- Si `tab[0] > argument` :  
   - appel récursif à `fun7` avec l’élément 2 index plus loin, et on multiplie le retour par 2.
- Sinon, on initialise `iVar1 = 0`, puis si `tab[0] != argument` :  
   - appel récursif à `fun7` avec l’élément 4 index plus loin, et on multiplie le retour par 2 puis ajoute 1.

- Il y a un tableau de tableaux (int **), avec plusieurs noeuds (`n1, n21, n22, n31, ...`).

**Images explicatives :**  
![image](https://github.com/user-attachments/assets/c85d2fbd-3c80-415a-97dc-011884bf4142)  
![image](https://github.com/user-attachments/assets/fcb2f67e-cdef-44d3-9974-31713101b13e)  
![image](https://github.com/user-attachments/assets/c9ec4318-b31e-412d-ab7f-97cd7ac42eca)

> Légende :  
> - ROUGE → noeud  
> - BLEUE → valeur du noeud  
> - ORANGE → adresse du noeud n+2  
> - JAUNE → adresse du noeud n+4

**Solution :**  
`47`

---
