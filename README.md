# Work

Executable ouvre le fichier en argument

NEED -> defuse.txt + gdbCfg

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

premier argument est passé à la fonction func4 (passé par edi -> rdi)

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


rsp > 14



