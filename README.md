# Un compilateur brain-fuck ?

Brainf*** est un language de programmation quelque peu ésotérique désigné par **Urban Müller** en 1992. Les programmes brainfuck ne sont composés que des caractères **+**, **-**, **<**, **>**, **[**, **]**, **.** et **,**.

Voici un exemple du fameux "hello world" en bf :

```brainfuck
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```
Inutile donc de préciser que la syntaxe de ce langage est très peu adaptée à l'écriture de véritables programmes. En revanche, c'est un excellent joujou pour les curieux que nous sommes et il peut s'avérer très utiles dans certaines circonstances. Laissez nous vous expliquez pourquoi...

## Un histoire de complétude ?

Lorsque l'on parle des langages de programmation, il est souvent question de la *Turing Completeness* (caractère Turing complet) de ces derniers.
Derrière cette expression un peu barbare se cache une idée simple : il s'agit de montrer qu'un langage de programmation est suffisamment riche pour pouvoir exprimer n'importe quel calcul. Nous ne rentrerons pas dans les détails formels qui se cachent derrière cette notion car il faudrait des centaines de lignes juste pour définir proprement le sujet (Des générations de chercheurs y travaillent encore, c'est dire !).

Maintenant que cette terminologie est introduite, pourquoi diable nous intéressons nous à ce sujet dans un article sur Brainfuck ? Eh bien figurez vous que ce langage de programmation a le bon goût d'être complet au sens de Turing. Malgré son aspect ultra élémentaire, il est théoriquement suffisamment riche pour exprimer à peu près ce qu'on veut.

Or, il se trouve que prouver la complétude d'un langage de programmation peut être une tâche assez ardue. L'une des manière simple et de procéder par équivalence avec un langage dont on connaît déjà le caractère Turing Complet. Dans notre cas, nous voulions démontrer la complétude d'un langage d'assemblage en cours d'élaboration : le langage Eva. Pour se faire, nous avons implémenté un compilateur Brainfuck vers Eva.

## Un bref aperçu du langage EVA ...

Le langage Eva est un langage développé par l'association [CodeAnon](https://github.com/codeanonorg). Il s'agit d'un langage d'assemblage destiné à être utilisé pour programmer une machine virtuelle. Trés peu d'instructions sont disponibles en Eva.

+ `ADD/ADDC/SUB/SUBC` : opérations arithmétiques
+ `MOV` : chargement de valeur dans les registres
+ `LDR/STR` : opération sur la mémoire
+ `PUSH/POP` : opération sur la pile
+ `IN/OUT` : Entrées / Sorties
+ `CMP` : comparaison
+ `BEQ/BNEQ/BLE/BLT` : branchement conditionnels

## ... et du langage BF

Le langage brainfuck est prévu pour s'éxecuter sur une machine contenant des "cellules" de mémoire. On pourra également manipuler une tête de lecture écriture permettant de mettre à jour le contenu de ces cellules. Cette tête est appellée *data pointer*.

+ `+` : Incrémente de 1 la valeur dans la cellule pointé par le *data pointer*
+ `-` : décrémente de 1 la valeur dans la cellule pointé par le *data pointer*
+ `>` : décale le *data pointer* d'une cellule vers la droite.
+ `<` : décale le *data pointer* d'une cellule vers la gauche.
+ `[` : début d'une boucle **while** qui s'arrête lorsque le *data pointer* pointe une cellule de valeur 0.
+ `]` : fin d'une boucle **while**.
+ `,` : saisie d'un caractère dans la cellule pointée par *data pointer*
+ `.` : affichage du caractère dans la cellule pointée par *data pointer*

Nous constatons déjà que les instructions de Eva ou de Brainfuck sont à peu près du même ordre. On ne peut manipuler que des structures très simples : des cases de mémoire, et de opérations arithmétiques élémentaires (addition, soustraction).

## Le coeur du compilateur

