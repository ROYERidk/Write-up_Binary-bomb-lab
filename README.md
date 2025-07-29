# Work

Executable ouvre le fichier en argument

NEED -> defuse.txt + gdbCfg

Phase 1
Prends la ligne 1
Ligne 1 : "I am just a renegade hockey mom."

Phase 2
Prends la ligne 2
Tableau de 6 int : 1 2 4 8 16 32

Le programme fait appel à une fonction "read_six_numbers" qui met nos 6 chiffres entrées dans un tableau nommé "local_38" (sous ghidra) mais on assigne la même adresse à "piVar1" donc qu'on traite l'un ou l'autre, dans tous les cas on vérifie juste les valeurs du tableau que nous avons mis en entrée (2ème ligne)

Première valeur du tableau doit valoir 1 :
	c'est assez littéral, "si la première valeur du tableau est différent de 1" -> explode_bomb()
	
	if (local_38[0] != 1) {
	    explode_bomb();
	  }
	
Deuxième valeur du tableau doit valoir 2 :
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
