# Binary bomb lab write-up

Ce programme exécutable ouvre le fichier en argument, lit une ligne par phase (6 phases), et traite chaque ligne pour désamorcer chaque phase.

- Ligne 1 → désamorce la phase 1  
- Ligne 2 → désamorce la phase 2  
- etc...

**Prérequis :**  
- `defuse.txt` (mon fichier que je passe en argument)
- `gdbcfg`     (Pour se faciliter la vie avec gdb classique)

## Sommaire

- [Phase 1](#phase-1)
- [Phase 2](#phase-2)
- [Phase 3](#phase-3)
- [Phase 4](#phase-4)
- [Phase 5](#phase-5)
- [Phase 6](#phase-6)
- [Secret Phase](#secret-phase)

## Phase 1

Rien de spécial dans cette première phase qui sert d'exemple, la solution est en clair dans le binaire.

**Solution :** `I am just a renegade hockey mom.`

## Phase 2

Le programme fait appel à une fonction `read_six_numbers` qui place les 6 chiffres entrés dans un tableau nommé `local_38` (sous Ghidra).
Notez que les variable du type `piVarX` & `local_X` sont des variables très certainement générique de Ghidra. Je les utilise dans mes explications pour plus de simplicité. 
<br>
La même adresse est assignée à la variable `piVar1`. Donc `local_38` et `piVar1` pointent le même tableau d'entier.  

Petit rappel : `local_38[0]` est pareil que `*local_38`, les deux accèdent à la première valeur du tableau 
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
    - On incrémente `piVar1` pour traverser le tableau tant que nous ne sommes pas à la fin.
    - Chaque nombre suivant doit être le double du précédent (tableau à 6 chiffres).

**Solution :** `1 2 4 8 16 32`

## Phase 3

```
  switch(local_18) {
  case 0:
    iVar1 = 0x274;
    break;
  case 1:
    iVar1 = 0;
    break;
  case 2:
    iVar1 = 0;
    goto LAB_00101699;
  case 3:
    iVar1 = 0;
    goto LAB_0010169e;
  case 4:
    iVar1 = 0;
    goto LAB_001016a1;
  case 5:
    iVar1 = 0;
    goto LAB_001016a4;
  case 6:
    iVar1 = 0;
    goto LAB_001016a7;
  case 7:
    iVar1 = 0;
    goto LAB_001016aa;
  default:
    explode_bomb();
    iVar1 = 0;
    goto LAB_001016ad;
  }
  iVar1 = iVar1 + -0x24c;
LAB_00101699:
  iVar1 = iVar1 + 0x2b0;
LAB_0010169e:
  iVar1 = iVar1 + -0x7e;
LAB_001016a1:
  iVar1 = iVar1 + 0x7e;
LAB_001016a4:
  iVar1 = iVar1 + -0x7e;
LAB_001016a7:
  iVar1 = iVar1 + 0x7e;
LAB_001016aa:
  iVar1 = iVar1 + -0x7e;
LAB_001016ad:
  if ((5 < (int)local_18) || (local_14 != iVar1)) {
    explode_bomb();
  }
```

- 2 arguments : uint, int
- Utilise un switch/case sur le premier élément (uint)
- Conditions : notre premier argument doit être inférieur à 5 `(uint < 5)` et notre deuxième argument doit être le résultat du calcul final `(int == calcul fait par le switch/case)`

Explication :
- Il y a 7 switch/case et nous ne pouvons pas commencé à l'étape 5 ou plus.
- Si on prend 4 comme premier argument, on tombe dans le case 4, comme les case sont sans break, on continue à travers les autres 4 → 5 → 6 → 7.
- Cela donne `eax = 0 + 0x7e - 0x7e + 0x7e - 0x7e = 0`.

**Solution :** `4 0`

## Phase 4

**fait avec seulement GDB**

premier argument est passé à la fonction func4 (passé par edi -> rdi)

RAX commence toujours à 0xe(=14)
ICI j'avais 8 en premier argument
```
	0x55555555571a <func4+5>:	mov    %edx,%eax			met la valeur de edx dans eax (=14)	
	0x55555555571c <func4+7>:	sub    %esi,%eax			soustrait esi à eax (14 - 0 = 14)
	0x55555555571e <func4+9>:	mov    %eax,%ebx			met eax dans ebx (=14)
	0x555555555720 <func4+11>:	shr    $0x1f,%ebx			décale ebx de 31 bits (0x1f = 31) -> ebx = 0
	0x555555555723 <func4+14>:	add    %eax,%ebx			ajoute eax à ebx (0 + 14 = 14)
	0x555555555725 <func4+16>:	sar    %ebx				divise ebx par 2 (14 / 2 = 7)
	0x555555555727 <func4+18>:	add    %esi,%ebx			ajoute esi à ebx (7 + 0 = 7)
	0x555555555729 <func4+20>:	cmp    %edi,%ebx			compare ebx par rapport à edi (ebx = 7 > edi = 8) [edi est notre argument donc c'est ici qu'il va falloir savoir si on veut jump à +30 ou +42]
	0x55555555572b <func4+22>:	jg     0x555555555733 <func4+30>	jg = jump if greater	
	0x55555555572d <func4+24>:	jl     0x55555555573f <func4+42>	jl = jump if less	DANS CE CAS LA on jump ici, on va à +42
```
Voici le contenu du programme à +42 :
```	
 	0x55555555573f <func4+42>:	lea    0x1(%rbx),%esi			calcul rbx + 1 (7 + 1) et l'assigne à esi (incrémente esi) [n'oublions pas que rsi est le deuxième argument passé pour les fonctions, on dirait qu'il incrémente un compteur]
	0x555555555742 <func4+45>:	call   0x555555555715 <func4>		appel récursif à lui même donc on recommence ce qui se passe au dessus mais esi vaut 1 maintenant
   	0x555555555747 <func4+50>:	add    %eax,%ebx			/*standby*/
   	0x555555555749 <func4+52>:	jmp    0x55555555572f <func4+26>

   	0x55555555571a <func4+5>:	mov    %edx,%eax			met la valeur de edx dans eax (=14)	
   	0x55555555571c <func4+7>:	sub    %esi,%eax			soustrait esi à eax (14 - 8 = 6)
   	0x55555555571e <func4+9>:	mov    %eax,%ebx			met eax dans ebx (=6)
   	0x555555555720 <func4+11>:	shr    $0x1f,%ebx			décale ebx de 31 bits (0x1f = 31) -> ebx = 0
   	0x555555555723 <func4+14>:	add    %eax,%ebx			ajoute eax à ebx (0 + 6 = 6)
   	0x555555555725 <func4+16>:	sar    %ebx				divise ebx par 2 (6 / 2 = 3)
   	0x555555555727 <func4+18>:	add    %esi,%ebx			ajoute esi à ebx (8 + 3 = 11)
   	0x555555555729 <func4+20>:	cmp    %edi,%ebx			compare ebx par rapport à edi (ebx = 11 > edi = 8) [edi est notre argument donc c'est ici qu'il va falloir savoir si on veut jump à +30 ou +42]
   	0x55555555572b <func4+22>:	jg     0x555555555733 <func4+30>	jg = jump if greater	DANS CE CAS LA on jump ici, on va à +30
   	0x55555555572d <func4+24>:	jl     0x55555555573f <func4+42>	jl = jump if less	
```
Voici le contenu du programme à +30 :
```
   	0x555555555733 <func4+30>:	lea    -0x1(%rbx),%edx			rbx -1 dans edx (décrémente rbx)
   	0x555555555736 <func4+33>:	call   0x555555555715 <func4>		appel récursif
```
/!\ Si ebx = edi -> return /!\

après notre func4 on compare directement eax à 10 (eax 32 bits, rax 64 bits, la valeur de retour)
si eax != 10 on explose
donc on veut que eax = 10
si c'est bon :
on check si notre deuxième argument vaut 0xa(=10)
```	
 	0x55555555579e <phase_4+83>:	cmpl   $0xa,0x4(%rsp)	cette ligne compare la valeur contenue à l'adresse rsp+4 avec 10 (on comprends assez logiquement que c'est notre deuxième argument, suffit de la changer pour voir que rsp+4 change. Surtout qu'un int fait 4 bits et que la on accède donc tout pile au deuxième int)
```
si oui GG !

Maintenant il faut trouver comment avoir 10 dans rax et c'est GG !
On ne peut pas avoir plus de 14 en premier argument sinon on explose !
comme c'est un int -> on peut mettre [0;14]

**Solution :** `3 10`

**pourquoi 3** : La première étape de func4 est globalement d'avoir 14 dans `edx`, de le diviser par 2 avant de le comparer avec notre argument. Donc en observant le code si on peut rapidement trouvé que si on met 3, nous allons tombé dans
<br>
`jg     0x555555555733 <func4+30>	jg = jump if greater`
<br>
qui va simplement décrémenté `ebx` (7 - 1) et faire un appel récursif qui va prendre `edx` à 6, le diviser par 2 et sortir de notre func4 car 3 == 3.
Enfin, il faut aussi comprendre que le résultat de la fonction (dans RAX) est l'addition de chaque retour récursif de cette fonction.
C'est pour cela que 3 rempli son travail, car il crée deux passage dans func4, un premier qui renvoie 7 et un deuxième qui renvoie 3.
7 + 3 = 10. Comme nous avions besoin de rax == 10, GG !

Dans ghidra, on observe directement le calcul, c'est bien plus simple, mais moins intéressant pour s'habituer à manipuler de l'assembleur dans un débugger comme GDB.

## Phase 5

Nous avons besoin de 2 int en arguments sinon on explose.
Nous nommerons nos deux argument respectivement `a` et `b`

un ET logique est fait entre `a` et *0xf* puis stocker à la place de ce dernier (seulement les 4 bits de poids faible sont gardés = on fait `a` modulo 16). Ensuite on voit que si `a` vaut 0xf, on explose.
Si on veut ne pas exploser il faut que `a` contienent au moins un 0 dans les 4 bits de poid faible de sa valeur en binaire donc on exclu simplement 15, il nous reste donc [0;14] comme possibilités.

Deux variables sont initialisées à 0 on les nommera respectivement `x` et `y`.
ensuite on boucle tant que `a` ne vaut pas *0xf*.

La boucle se décrit comme tel :
- `y` semble être un compteur qui s'incrémente à chaque passage de boucle
- `a= tab[a]` tab est un tableau de int dans le binaire à l'adresse de rsi. 
- enfin, `x = x + a` -> est la somme de chaque valeur de `a` à chaque tour de boucle.

Pour désomorcer cette phase il faut que `y == Oxf` = 15 (15 tour de boucle) et `b == x` [après la boucle]

Donc pour flag cette 5ème phase, il faut :
faire exactement 15 tour de boucle ET connaitre à l'avance la somme de chaque `a` à chaque passage de boucle.

Voici le contenu du tableau à chaque index :
<img width="372" height="825" alt="image" src="https://github.com/user-attachments/assets/36ac8623-3826-43ef-ae11-1e36f774293a" />

Maintenant il faut réfléchir à comment tombé sur 0xf au 15e tour de boucle.
Il suffit de faire 15 fois la boucle à l'envers en partant de 0xf pour savoir à quelle valeur démarrer : 
5 (0x12) -> 12 (0x3) -> 3 (0x7) -> 7 (0x11) -> 11 (0x13) -> 13 (0x9) -> 9 (0x4) -> 4 (0x8) -> 8 (0x0) -> 0 (0xa) -> 10 (0x1) -> 1 (0x2) -> 2 (0xe) -> 14 (0x6) -> 6 (0xf)

Il est important de noté que la première valeur (ici 5) n'est pas ajouter à la somme finale, elle sert seulement de points de départ. Donc on additionne en partant de 12.

12 + 3 + 7 + 11 + 13 + 9 + 4 + 8 + 0 + 10 + 1 + 2 + 14 + 6 + 15 = 115.

**Solution :** `5 115`

*Note : la première valeur (ici 5) n’est pas ajoutée à la somme finale, elle sert seulement de point de départ.*

## Phase 6

On fait refait appel à `read_six_numbers` pour récupérer 6 int qu'on passe depuis notre fichier de solver.
Nos 6 arguments sont dans R13.

La première étape sert à vérifier qu'on ne met pas deux fois le même nombre dans nos 6 arguments et toujours des nombres < 7.

Deuxièmement, le programme va aller chercher la valeur du noeud associé à notre premier argument -> récupérer la valeur de ce noeud -> aller chercher le noeud associé à notre 2eme argument -> récup la valeur de ce noeud, s'assurer que notre deuxième noeud à une valeur inférieure au premier, ainsi de suite...

Dans l'image suivante on peut observer pour chaque node sa valeur, son numéro puis l'adresse du noeud suivant.

<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/a644c6a3-b315-46ac-9c28-083bb7f61c0e" />
Il faut donc comprendre qu'il va falloir donner en arguments les noeuds dans l'ordre décroissant de leur valeur. 

**Solution :** 5 4 3 1 6 2

## Secret Phase

Nous devons ajouter `DrEvil` à la fin du solve de la phase 4 pour arriver dans la phase secrète. (Nous le voyons dans la function `phase_defuse` qui fait un appel à la fonction fscanf à une adresse précise). 

Secret phase attends une strings qui sera transformée en un int.
Ce int doit être < 1000 sinon on explose.

une fonction `fun7` est appellée, il faudra que son retour soit = 5 pour résoudre cette phase.
`fun7` prends l'adresse d'un tableau de int en 1er argument et notre argument en 2eme.
```
        00101987 48 85 ff        TEST       param_1,param_1 ## Check si le tableau param_1 est bien initialisé
```
Pourquoi regarder qu'a chaque passage notre paramètre 1 est bien initialisé ? car dans la suite nous allons voir que l'adresse du tableau passé en argument n'est jamais le même.

Il existe 3 CAS :
Nous appelerons `tab` la tableau passé en 1er argument à la fonction `fun7`. 

notre argument est inférieur à `tab[0]` -> appel récursif de `fun7` avec tab[2] en 1er argument.
iVar1 est doublé.

```
	if tab[0] > notre argument :
 		iVar1 = call fun7 avec l'adresse qui nous donne 2 éléments plus loin notre noeud actuel et toujours notre argument
		iVar1 = iVar1 * 2	
```
Ivar1 est initialisé à 0
notre argument est supérieur à `tab[0]` -> appel récursif de `fun7` avec `tab[4]` en 1er argument.
iVar1 est doublé puis incrémenté.

```
	Ivar1 = 0;
 	if tab[0] != notre argument
		iVar1 = call fun7 avec l'adresse qui nous donne 4 éléments plus loin notre noeud actuel et toujours notre argument
		iVar1 = iVar1 * 2 + 1
```

notre argument vaut `tab[0]`.
On renvoie 0.

```
	iVar1 = 0;
```

Il y a un tableau de tableau, d'ou le (int **)(param_1 + 2) dans les appels récursifs. <br>
J'ai trouvé 7 tableaux différents : `n1, n21, n22, n31, n32, n33, n34, n41, n42, n43, n44, n45, n46, n47, n48.` <br>
C'est une chaine car dans ce dumb que j'ai réussi à trouver en me baladant autour de l'adresse de `n1` (le premier arguments passé par défault à fun7) :
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/c85d2fbd-3c80-415a-97dc-011884bf4142" />
On retrouve un peu la même idée que dans la phase 6. On navigue comme cela : 
<br>
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/fcb2f67e-cdef-44d3-9974-31713101b13e" />
<br>
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/c9ec4318-b31e-412d-ab7f-97cd7ac42eca" />
<br>

> Légende :  
> - ROUGE → noeud  
> - BLEUE → valeur du noeud  
> - ORANGE → adresse du noeud n+2  
> - JAUNE → adresse du noeud n+4

L'objectif est donc d'avoir la fonction `fun7` qui renvoie la valeur 5, pour ce faire nous devons obtenir le calcul suivant avec les récursions : 
<br> `0 -> 0 * 2 + 1 (=1) -> 1 * 2 (=2) -> 2 * 2 + 1 (=5)`.<br>
Donc il nous faut trouver un chiffre à passer en argument qui est supérieur, inférieur ou égale à la valeur du noeud dans lequel nous sommes à chaque étape pour s'assurer de prendre le bon chemin.
Attention ! certains noeud n'ont pas de suivant et si c'est le cas, un appel récursif renvoie -1 ! (d'oû la vérification que le tableau est bien initialisé au début de `fun7`.

**Solution :** `47`
