# Work

Executable ouvre le fichier en argument
il lit une ligne par phase (6 phases)
ligne 1 -> désamorce phase 1
ligne 2 -> désamorce phase 2
etc...

NEED -> defuse.txt + gdbcfg

## Phase 1
Prends la ligne 1
Ligne 1 : "I am just a renegade hockey mom."

## Phase 2
Prends la ligne 2
Tableau de 6 int : 1 2 4 8 16 32

Le programme fait appel à une fonction "read_six_numbers" qui met nos 6 chiffres entrées dans un tableau nommé "local_38" (sous ghidra) mais on assigne la même adresse à "piVar1" donc qu'on traite l'un ou l'autre, dans tous les cas on vérifie juste les valeurs du tableau que nous avons mis en entrée (2ème ligne)

#### Première valeur du tableau doit valoir 1 :
c'est assez littéral, "si la première valeur du tableau est différent de 1" -> explode_bomb()
	
	if (local_38[0] != 1) {
		explode_bomb();
	}
	
#### Deuxième valeur du tableau doit valoir 2 :
(rbx) piVar1[1] -> deuxième valeur du tableau    
(eax) *piVar1 * 2 -> valeur de l'entier pointé par piVar1 multiplié par 2 -> Donc 4
 
(On trouve sa valeur grâce à GDB)
Suffit de diviser 4 par 2 pour connaitre la valeur que nous devons avoir dans notre tableau en deuxième position
	
	if (piVar1[1] != *piVar1 * 2) {
		explode_bomb();
	}
	
Après cela, piVar1 est incrémenté
	
Puis on boucle tant que piVar1 n'est pas la dernière valeur du tableau. (sur cette étape seulement)
	
l'important est de comprendre que *piVar1 et Pivar1[0] c'est la même chose. Donc on a besoin d'un tableau qui commence par 1 (première étape) et ensuite "piVar1[1] != *piVar1 * 2" la deuxième valeur du tableau doit être égale au double de la première. Ensuite on incrémente donc la deuxième valeur devient la première et la troisième la deuxième. Ainsi de suite jusqu’à la fin du tableau. 
	
En résumé, il nous faut un tableau qui commence par 1, puis chaque nombre qui suit devra être le double du précédent, dans la limite d'un tableau de 6 chiffre.

## Phase 3

4 0

prends la 3ème ligne -> 2 arguments -> uint, int -> sinon explosion

switch/case en fonction du premier elements (uint)

il faut (utin < 5) et (int = calcul fait par le switch/case à partir du nombre d'arguments lu par sscanf)
	(utin < 5) et (int = calcul fait par le switch/case à partir du nombre 2)

 uint pour switch/case doit être > 5 donc switch/case de 6 et 7 inutilisables -> reste que 0 à 5

Si on prends 4 en premier argument, on tombe dans le case 4, comme il n'y à pas de break on continue de passer dans chaque case suivante, 4 -> 5 -> 6 -> 7
Cela donne eax = 0 + 0x7e - 0x7e + 0x7e - 0x7e = 0
Donc notre 3eme ligne doit être "4 0"
 
## Phase 4
**fait avec seulement GDB**

premier argument est passé à la fonction func4 (passé par edi -> rdi)

RAX commence toujours à 0xe(=14)
ICI j'avais 8 en premier argument

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

Voici le contenu du programme à +42 :
	
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

Voici le contenu du programme à +30 :

   	0x555555555733 <func4+30>:	lea    -0x1(%rbx),%edx			rbx -1 dans edx (décrémente rbx)
   	0x555555555736 <func4+33>:	call   0x555555555715 <func4>		appel récursif

/!\ Si ebx = edi -> return /!\

après notre func4 on compare directement eax à 10 (eax 32 bits, rax 64 bits, la valeur de retour)
si eax != 10 on explose
donc on veut que eax = 10
si c'est bon :
on check si notre deuxième argument vaut 0xa(=10)
	
 	0x55555555579e <phase_4+83>:	cmpl   $0xa,0x4(%rsp)	cette ligne compare la valeur contenue à l'adresse rsp+4 avec 10 (on comprends assez logiquement que c'est notre deuxième argument, suffit de la changer pour voir que rsp+4 change. Surtout qu'un int fait 4 bits et que la on accède donc tout pile au deuxième int)
si oui GG !

Maintenant il faut trouver comment avoir 10 dans rax et c'est GG !
On ne peut pas avoir plus de 14 en premier argument sinon on explose !
comme c'est un int -> on peut mettre [0;14]

la solution est "3 10"

pourquoi 3 : La première étape de func4 est globalement d'avoir 14 dans edx, de le diviser par 2 avant de le comparer avec notre argument. Donc en observant le code si on peut rapidement trouvé que si on met 3, nous allons tombé dans *jg     0x555555555733 <func4+30>	jg = jump if greater* qui va simplement décrémenté ebx (7 - 1) et faire un appel récursif qui va prendre edx à 6, le diviser par 2 et sortir de notre func4 car 3 == 3.
Enfin, il faut aussi comprendre que le résultat de la fonction (dans RAX) est l'addition de chaque retour récursif de cette fonction.
C'est pour cela que 3 rempli son travail, car il crée deux passage dans func4, un premier qui renvoie 7 et un deuxième qui renvoie 3.
7 + 3 = 10. Comme nous avions besoin de rax == 10, GG !

Dans ghidra, on observe directement le calcul, c'est bien plus simple, mais moins intéressant pour s'habituer à manipuler de l'assembleur dans un débugger comme GDB.

## Phase 5

2 int en arguments sinon on explose
nous nommerons nos deux argument respectivement *a* et *b*

un ET logique est fait entre *a* et 0xf, puis stocker à la place de ce dernier (seulement les 4 bits de poids faible sont gardés = on fait *a* modulo 16). Ensuite on voit que si *a* vaut 0xf, on explose.
Si on veut ne pas exploser il faut que *a* contienent au moins un 0 dans les 4 bits de poid faible de sa valeur en binaire donc on exclu simplement 15, il nous reste donc [0;14] comme possibilités.

Deux variables sont initialisées à 0 on les nommera respectivement *x* et *y*.
ensuite on boucle tant que *a* qui ne devait pas valoir 0xf vale 0xf.

La boucle se décrit comme tel :
*y* semble être un compteur qui s'incrémente à chaque passage de boucle
*a = tab[a]*. tab est un tableau de int dans le binaire à l'adresse de rsi. 
enfin, *x = x + a* -> c'est la somme de chaque valeur de *a* à chaque tour de boucle.

Pour désomorcer cette phase il faut que *y* == Oxf = 15 (15 tour de boucle) et *b* == x [après la boucle]

Donc pour flag cette 5ème phase, il faut :
faire exactement 15 tour de boucle ET connaitre à l'avance la somme de chaque *a* à chaque passage de boucle.

Voici le contenu du tableau à chaque index :
<img width="372" height="825" alt="image" src="https://github.com/user-attachments/assets/36ac8623-3826-43ef-ae11-1e36f774293a" />

Maintenant il faut réfléchir à comment tombé sur 0xf au 15e tour de boucle.
Il suffit de faire 15 fois la boucle à l'envers en partant de 0xf pour savoir à quelle valeur démarrer : 
5 (0x12) -> 12 (0x3) -> 3 (0x7) -> 7 (0x11) -> 11 (0x13) -> 13 (0x9) -> 9 (0x4) -> 4 (0x8) -> 8 (0x0) -> 0 (0xa) -> 10 (0x1) -> 1 (0x2) -> 2 (0xe) -> 14 (0x6) -> 6 (0xf)

Enfin, la solution est "5 115". 
Il est important de noté que la première valeur (ici 5) n'est pas ajouter à la somme finale, elle sert seulement de points de départ. Donc on additionne en partant de 12.

12 + 3 + 7 + 11 + 13 + 9 + 4 + 8 + 0 + 10 + 1 + 2 + 14 + 6 + 15 = 115.

## Phase 6

mes 6 arguments sont dans R13

La première étape sert à vérifier qu'on ne met pas deux fois le même numéro dans nos 6 arguments et toujours de numéros < 7

deuxièmement, le programme va aller chercher la valeur du noeud associé à notre premier argument -> récup la valeur de ce noeud -> aller chercher le noeud associé à notre 2eme argument -> récup la valeur de ce noeud, s'assurer que notre deuxième noeud à une valeur inférieure au premier, ainsi de suite...

Dans l'image suivante on peut observer pour chaque node sa valeur, son numéro puis la suivante.

<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/a644c6a3-b315-46ac-9c28-083bb7f61c0e" />



Il faut donc comprendre qu'il va falloir donner en arguments les noeuds dans l'ordre décroissant, c'est à dire :
5 4 3 1 6 2

## Secret phase

On doit rajouter DrEvil à la fin du solve de la phase 4 pour arriver dans la phase secrète.

Secret phase attends une strings qui sera transformée en int.
Ce int doit être < 1000

une fonction fun7 est appellée, il faudra que son retour soit = 5 pour defuse cette phase
fun7 prends un tableau de int en 1er argument et notre argument en 2eme

        00101987 48 85 ff        TEST       param_1,param_1 ## Check si le tableau param_1 est bien initialisé


2 CAS :
```
	if tab[0] > notre argument :
 		iVar1 = call fun7 avec l'élement 2 index plus loin que l'actuel
		iVar1 = iVar1 * 2	
 	sinon iVar1 = 0;
 	puis if tab[0] != notre argument
 		iVar1 = call fun7 avec l'élement 4 index plus loin que l'actuel
	iVar1 = iVar1 * 2 + 1
```

Il y a un tableau de tableau, d'ou le (int **)(param_1 + 2) dans les appels récursifs. <br>
J'ai trouvé 7 tableaux différents : n1, n21, n22, n31, n32, n33, n34, n41, n42, n43, n44, n45, n46, n47, n48. <br>
C'est une chaine car dans ce dumb que j'ai réussi à trouver en me baladant autour de l'adresse de n1 (le premier arguments passé par défault à fun7) :
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/c85d2fbd-3c80-415a-97dc-011884bf4142" />
On retrouve un peu la même idée que dans la phase 6. On navigue comme cela : 
<br>
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/fcb2f67e-cdef-44d3-9974-31713101b13e" />
<br>
<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/c9ec4318-b31e-412d-ab7f-97cd7ac42eca" />
<br>
Légende : ROUGE -> noeud, BLEUE -> valeur du noeud, ORANGE -> adresse du noeud n+2, JAUNE -> adresse du noeud n+4.

Le solve est 47 !
