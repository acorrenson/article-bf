# Un compilateur brain-fuck ?

Brainf*** est un langage de programmation quelque peu ésotérique designé par **Urban Müller** en 1992. Le programmes brainfuck ne sont composé que des caractères **+**, **-**, **<**, **>**, **[**, *]**, **.** et **,**.
Inutile donc de préciser que la syntaxe de ce langage est très peu adaptée, à l'écriture de véritables programmes. En revanche, c'est un excelent joujou pour les curieux que nous sommes. Laissez nous vous expliquez pourquoi nous avons écris un compilateur brainfuck.
Voici un exemple du fameux "hello world" en bf :

```brainfuck
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

# Un histoire de complétude ?

Lorsqu'on parle des langages de programmation, il est souvent question de la *Turing Completeness* (caractère Turing complet) de ces derniers.
Derrière cette expression un peu barbare se cache une idée simple : il s'agit de montrer qu'un langage de programmation est suffisament riche pour pouvoir exprimer n'importe quel calcul. Nous ne renterons pas dans les détails formels qui se cachent derrière cette notion car il faudrait des centaines de lignes juste pour définir proprement le sujet (Des générations de chercheurs y travaillent encore, c'est dire !).
Maintenant que cette terminologie est introduite, pourquoi diable nous interessons nous à ce sujet dans un article sur Brainfuck ? Eh bien figurez vous que ce langage de programmation a le bon goût d'être complet au sens de Turing. Malgré son aspect ultra élémentaire, il est théoriquement suffisament riche pour exprimer à peu près ce qu'on veut.

Or, il se trouve que prouver la complétude d'un langage de programmation peut être une tâche assez ardue. L'une des manière simple et de procéder par équivalence avec un langage dont on connait déjà le caractère Turing Complet.
