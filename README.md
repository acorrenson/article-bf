<div style="text-align:center">
	<img src="./banner.jpg"></img>
</div>

# Un compilateur Brainfuck ?

Arthur Correnson <arthur.correnson@univ-tlse3.fr> & Nathan Graule <solarliner@gmail.com>

# Introduction

Brainfuck est un langage de programmation ésotérique désigné par **Urban **Müller**** en 1992. Les programmes brainfuck ne sont composés que des caractères **+**, **-**, **<**, **>**, **[**, **]**, **.** et **,**.

Voici un exemple du fameux "hello world" en bf :

```brainfuck
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```
Inutile donc de préciser que la syntaxe de ce langage est très peu adaptée à l'écriture de véritables programmes. En revanche, c'est un excellent joujou pour les curieux que nous sommes et il peut s'avérer très utiles dans certaines circonstances. Laissez-nous vous expliquez pourquoi...

## Un histoire de complétude ?

Lorsque l'on parle des langages de programmation, il est souvent question de la *Turing Completeness* (caractère Turing complet) de ces derniers.
Derrière cette expression un peu barbare se cache une idée simple : il s'agit de montrer qu'un langage de programmation est suffisamment riche pour pouvoir exprimer n'importe quel calcul. Nous ne rentrerons pas dans les détails formels qui se cachent derrière cette notion car il faudrait des centaines de lignes juste pour définir proprement le sujet (Des générations de chercheurs y travaillent encore, c'est dire !).

Maintenant que cette terminologie est introduite, pourquoi diable nous intéressons nous à ce sujet dans un article sur Brainfuck ? Eh bien figurez-vous que ce langage de programmation a le bon goût d'être complet au sens de Turing. Malgré son aspect très rudimentaire, il est théoriquement suffisamment riche pour exprimer à peu près ce que l'on veut.

Or, il se trouve que prouver la complétude d'un langage de programmation peut être une tâche assez ardue. L'une des manières simples et de procéder par équivalence avec un langage dont on connaît déjà le caractère Turing Complet. Dans notre cas, nous voulions démontrer la complétude d'un langage d'assemblage en cours d'élaboration : le langage Eva. Pour se faire, nous avons implémenté un compilateur Brainfuck vers Eva.

## Un bref aperçu du langage EVA ...

Le langage Eva est un langage développé par l'association [CodeAnon](https://github.com/codeanonorg). Il s'agit d'un langage d'assemblage destiné à être utilisé pour programmer une machine virtuelle. Il met à disposition du programmeur un jeu d'instruction très succinct :

+ `ADD/ADDC/SUB/SUBC` : opérations arithmétiques
+ `MOV` : chargement de valeur dans les registres
+ `LDR/STR` : opération sur la mémoire
+ `PUSH/POP` : opération sur la pile
+ `IN/OUT` : Entrées / Sorties
+ `CMP` : comparaison
+ `BEQ/BNEQ/BLE/BLT` : branchement conditionnel

## ... et du langage Brainfuck

Le langage brainfuck est prévu pour s'exécuter sur une machine contenant des "cellules" de mémoire. On pourra également manipuler une tête de lecture écriture permettant de mettre à jour le contenu de ces cellules. Cette tête est appelée *data pointer*.

+ `+` : Incrémente de 1 la valeur dans la cellule pointée par le *data pointer*
+ `-` : décrémente de 1 la valeur dans la cellule pointée par le *data pointer*
+ `>` : décale le *data pointer* d'une cellule vers la droite.
+ `<` : décale le *data pointer* d'une cellule vers la gauche.
+ `[` : début d'une boucle **while** qui s'arrête lorsque le *data pointer* pointe une cellule de valeur 0.
+ `]` : fin d'une boucle **while**.
+ `,` : saisie d'un caractère dans la cellule pointée par *data pointer*
+ `.` : affichage du caractère dans la cellule pointée par *data pointer*

Nous constatons déjà que les instructions de Eva ou de Brainfuck sont à peu près du même ordre de proximité avec la machine. On ne peut manipuler que des structures très simples : des unités de mémoire (cellules, registres), et des opérations arithmétiques élémentaires (addition, soustraction).

# Méthode

## Choix d'implémentation

Pour exécuter des programmes Brainfuck sur la machine virtuelle Eva, deux solutions s'offraient à nous. La première aurait été d'écrire une deuxième machine virtuelle en langage Eva et capable d'interpréter le langage brainfuck. Nous aurions donc fait tourner une machine virtuelle brainfuck dans une machine virtuelle Eva.

Une deuxième option étant d'écrire un compilateur capable de traduire le langage Brainfuck en langage Eva. Nous avons décidé de choisir cette alternative, un peu moins dépendante des éventuels défauts du langage Eva encore en cours de conception. Le compilateur est donc implémenté dans un troisième langage (Rust).

## Traduction des programmes

### Analyse syntaxique

La traduction à proprement parler des programmes brainfuck se décompose en plusieurs étapes. La première étape consiste à faire une analyse syntaxique. Au cours de cette étape, on extrait une représentation abstraite du programme sur laquelle le compilateur va pouvoir travailler. Cette étape nécessite d'abord de connaître la [grammaire formelle]() du langage afin d'identifier correctement les différents symboles qui compose un programme ainsi que leur agencement. Dans le cas de Brainfuck, la structure des programmes peut se définir informellement comme suit :

+ Un programme est une suite d'instructions
+ `+` `-` `>` `<` `.` `,` `[` `]` sont des symboles élémentaires
+ `+` `-` `>` `<` `.` `,` peuvent être interprétés comme des instructions
+ Une boucle est une instruction qui commence par le symbole `[`, termine par le symbol `]` et contient une suite d'instructions arbitraires (non vide).

La grammaire associée est la suivante :

```
Alphabet = {"+", "-", "<", ">", ".", ","}

# nommage des symboles élémentaires
Incr        -> "+"
Decr        -> "-"
Shift_left  -> "<"
Shift_right -> ">"
Input       -> ","
Output      -> "."
Loop_beg    -> "["
Loop_end    -> "]"

# règle de production d'une boucle
loop -> Loop_begin program Loop_end

# règle de production d'une instruction
# une instruction est soit un symbol élémentaire du langage
# soit une boucle while
instruction -> Incr
instruction -> Decr
instruction -> Shift_left
instruction -> Shift_right
instruction -> Input
instruction -> Output
instruction -> loop

# un programme peut se définir inductivement :
# Ou bien c'est une instruction seule,
# Ou c'est une instruction suivie d'un programme plus petit.
program -> instruction
program -> instruction program
```

**Remarque** : On a ici nommé les symboles élémentaires du langage Brainfuck pour garder une cohérence avec les [déclarations de types]() qui suivront.

Effectuons l'analyse syntaxique du programme suivant selon cette grammaire :

```brainfuck
+++[->+<]
```

```
program ( Incr, Incr, Incr, loop( Decr, Shift_right, Incr, Shift_left ) )
```

On pourra aussi remarquer que cette grammaire ne permet pas a priori de décrire des programmes brainfuck invalides :

```brainfuck
Programme invalide :
[++
```

En effet, ce programme ne satisfait aucune de nos règles de production.

En **Rust** la structure des programmes se traduit par la déclaration de type énuméré suivante :

```rust
// type pour les commandes brainfuck
pub enum Command {
	Inc,
	Dec,
	ShiftLeft,
	ShiftRight,
	Loop(Vec<Command>)
	Input,
	Output,
}

// alias pour le type "Program"
pub Program = Vec<Command>
```

On accompagnera cette définition de type d'un analyseur syntaxique construit à l'aider du module spécialisé [rust-peg](https://github.com/kevinmehall/rust-peg). Nous ne rentrerons pas dans les détails du code, mais nous pouvons voir que - mis de côté les détails techniques liés au langage Rust - ce morceau de code est une traduction de la grammaire du langage Brainfuck décrite précédemment.


```rust
peg::parser! {
	grammar brainfuck() for str {
		use super::{Command};
		rule ws() = quiet!{(" " / "\t" / "\n")*} // caractères blancs
		rule inc()      -> Command = v:$("+"+) {Command::Inc}
		rule dec()      -> Command = v:$("-"+) {Command::Dec}
		rule shift_l()  -> Command = v:$(">"+) {Command::Shift_left}
		rule shift_r()  -> Command = v:$("<"+) {Command::Shift_right}
		rule input()    -> Command = "," {Command::Input}
		rule output()   -> Command = "." {Command::Output}

		// note: lop est écrit avec un seul o car loop est un mot-clé de Rust et n'est donc pas disponible
		rule lop()      -> Command
			= ws() "["  ws() l:(inc() / dec() / shift_l() / shift_r() / input()
							   / output() / lop())+ ws() "]" ws() {
			Command::Loop(Box::from(l))
		}

		rule command()  -> Command = c:(inc() / dec() / shift_l() / shift_r() / input() / output() / lop()) {
			c
		}

		pub rule program() -> Program = ws() p:(command() / lop())+ ws() { Program::Program(p) }
	}
}
```

### Génération de code Eva

Construire une représentation des programmes est une première étape. Cette étape terminée, il reste à générer le code Eva à partir de la représentation du programme Brainfuck. La génération du code se traduit par une fonction qui reçoit en entré la représentation du programme Brainfuck et donne en sortie un programme Eva sous forme de texte (ou d'un executable).

#### Brainfuck est la mémoire

Avant d'entamer les détails de l'implémentation d'une telle fonction, rappelons que le langage Brainfuck requiert l'usage d'un *data pointer* et d'un ensemble de cellules de mémoire. Il faut donc se poser la question de comment simuler ces deux éléments.

1. Nous fixons le *data pointer* comme étant la valeur contenue dans le registre n°0 de la machine Eva (R0)
2. La mémoire de la machine Eva est utilisée à la fois pour charger le programme, et comme zone de lecture /écriture pour les programmes brainfuck. Un mot mémoires de Eva a une taille de 32-bits, par soucis de simplicité, nous avons donc fixé la taille des cellules à 32 bits pour avoir une équivalence directe entre cellule mémoire au sens de brainfuck et mot mémoire au sens de Eva. Notons que les machines Brainfuck mettent généralement à disposition des cellules de 8 bits (juste assez pour contenir des caractères ASCII).

Un premier problème se pose déjà. Si la mémoire de la machine Eva est à la fois le support de lecture/écriture et le support de stockage des programmes, il faut assurer qu'aucune opération de modification de la mémoire ne sera réalisée sur la région contenant le programme lui-même. Typiquement, il faudrait pouvoir éviter que l'instruction `+` ou `-` du langage Brainfuck ne soit exécutée alors que le *data pointer* pointe sur une case mémoire contenant le programme en train d'être exécuté.

Ce problème s'adresse en utilisant un *offset*. On compte le nombre de mots mémoire nécessaires au stockage du programme avant de le compiler, et on initialise le *data pointer* en conséquence. En eva, une instruction est stockée sur 32 bits (soit 1 mot). De plus les spécifications du langage Brainfuck impose deux contraintes sur le nombre de cellules :

1. Le nombre de cellules est au moins de 30k.
2. Le *data pointer* peut prendre des valeurs négatives.

Le point 1. est fixé pour garantir que la quantité de mémoire disponible est suffisante. Le point 2. est une spécificité de brainfuck qui permet de jouer avec les opérations `>` et `<`. Par exemple, si nous souhaitons écrire la valeur `1` dans deux cellules consécutives dés le début de l'éxecution d'un programme, on peut écrire `<+>+` au lieu de `>+>+` : écrire à gauche de la cellule initiale est tout aussi correcte que d'écrire à sa droite.

Pour garantir ces deux propriétés, on calcul donc l'*offset* suivant : `offset = nb_instructions + 15 000 - 1`.
Le code Eva généré par notre compilateur commencera donc toujours par l'en tête suivant :

```arm
	MOV R0, #n
```

où `#n` désigne l'*offset* auquel on ajoute 1 pour prendre en compte la première instruction `MOV` que l'on ajoute au programme. On peut donc simplifier le calcul de l'*offset* par la formule `offset = nb_instructions + 15 000`. Notons que cette méthode de calcul d'*offset* peut se généraliser pour s'adapter aux caractéristiques de la machine virtuelle utilisée et au nombre de cellules souhaité. On peut également se poser la question de la vérification du *data pointer*, c'est à dire s'assurer que ce dernier ne prend pas pour valeur des adresses protégées. Ces questions sont traitées dans la partie [discussion](#Discussion).

**Remarque 1** : Le calcul de cet *offset* peut se généraliser :
+ Soit `M` la taille en mot d'une instruction Eva en mémoire
+ Soit `N` le nombre de cellules que l'on souhaite mettre à disposition des programmeurs Brainfuck
+ Soit `I` le nombre d'instructions d'initialisations
+ Soit `T` le nombre d'instructions du code source brainfuck traduit en Eva.

Alors l'*offset* se calcul comme suit = `offset = M.(T+I) + N - 1`

**Remarque 2** : On peut ajouter un niveau de fiabilité supplémentaire en générant des instructions pour la vérification du *data pointer*. Ainsi, le code Eva généré s'occupe de vérifier que la valeur du *data pointer* est valide à tout moment et ne pointe pas sur des région de la mémoire protégées. Nous discuterons plus en détails ce cette question dans la partie [discussion](#Discussion)


#### Les instructions arithmétiques

Les opérateurs sur le *data pointer* `>` et `<` sont très simples à traduire, il s'agit d'opérations arithmétique sur le register `R0` (qui par convention sert de *data pointer*).

```arm
	; >
	ADD R0, #1
	; <
	SUB R0, #1
```

Les instructions `+` et `-` de brainfuck sont sont à peine plus complexes, elles nécessite de  modifier une valeur en mémoire en passant par un registre :

```arm
	; +
	; chargement de la valeur pointée
	LDR R1, [R0]
	; incrémentation
	ADD R1, #1
	; stockage de la valeur modifiée
	STR R1, [R0]

	; -
	; chargement de la valeur pointée
	LDR R1, [R0]
	; décrémentation
	SUB R1, #1
	; stockage de la valeur modifiée
	STR R1, [R0]
```

#### Entrées et Sorties

Les opérations d'entrée/sortie sont traduisibles directement en Eva également. Rappelons que l'instruction `.` affiche le caractère ASCII stocké sous forme d'entier dans la cellule pointée par *data pointer*. L'instruction `,` lit un octet et le place dans la cellule pointée par *data pointer*. Cela donne lieu aux traductions suivantes :

```arm
	; .
	OUT R0
	; ,
	IN R0
```

**Remarque** : Ces traductions sont relativement naïves et peuvent mener à des programmes très inefficaces. sIl n'est pas rares en brainfuck d'avoir de longues séquences d'instructions `+`, `-`, `>` ou `<`. Générer systématiquement une instruction Eva pour chacune de ces instructions Brainfuck mène à des codes très longs que l'on peut facilement optimiser. Par exemple, la séquence `++++++++` serait naïvement traduite comme une suite de 8 instructions `ADD R0, #1`. On peut remplacer cette suite d'instructions par `ADD R0, #8` et diviser ainsi le nombre d'instructions Eva générée. On augmente de cette façon les performances du programme Eva produit.

#### Boucles

Nous avons vu comment générer le code Eva pour les instructions arithmétiques et d'entrée/sortie du langage Brainfuck, il reste à traiter le cas des boucles. Ces dernières sont un peu plus subtiles à traduire. Le comportement d'une boucle en brainfuck est le suivant : Tant que le *data pointer* pointe sur une valeur différente de 0, le code de la boucle est executé, si le *data pointer* pointe sur 0, alors on sort de la boucle. Pour pouvoir décrire ce comportement, il faut donc deux choses indispensables : d'une part pouvoir localiser la première instruction du corps de la boucle, d'autre part pouvoir localiser la première instruction directement après la boucle. Pour permettre cela, on utilise les labels du langage Eva. On marque par un premier label le debut de la boucle ainsi que la première instruction directement après.

```brainfuck
avant [sequence] après
```

```arm
	; instructions pour "avant"

label_sequence:
	; instructions pour "sequence"

label_apres:
	; instructions pour après
```

Il faut ensuite assurer le bouclage sur les instructions du corps de la boucle. On utilise l'instruction de comparaison `CMP` ainsi que des branchements conditionnels `BNEQ/BEQ`.

```arm
label_sequence:
	; on charge la valeur pointée par le data pointer dans un registre
	LDR R1, [R0]
	; on compare cette valeur à 0.
	CMP R0, #0
	; on charge l'adresse de l'instruction de debut de sequence (première instruction dans le while)
	; dans un registre
	MOV R1, label_sequence
	; si la valeur pointée est différente de 0, on saute à l'adresse de début de séquence
	BNEQ R1
	; sinon, on poursuit l'execution des instructions dans l'ordre

label_apres:
	...
```

Notons que si le *data pointer* pointe déjà sur 0 avant le début de la boucle, il faut sauter les instructions du corps de la boucle :

```arm
	; instructions avant

	; on charge la valeur pointée par le data pointer dans R1
	LDR R1, [R0]
	; on compare cette valeur à 0
	CMP R1, #0
	; on charge l'adresse de la première instruction après la boucle
	LDR R1, label_apres
	; si la valeur pointée est 0, on peut passer la boucle (et donc sauter sur label_apres)
	BEQ R1
	; sinon, commence à executer les instructions de la séquence

label_sequence:
	; instructions de la sequence
	; ...

label_apres:
	; instructions après
	; ...
```

# Résultats

# Discussion


# Sources

+ [Brainfuck - Wikipédia](https://en.wikipedia.org/wiki/Brainfuck)
+ [Formal Grammar - Wikipédia](https://en.wikipedia.org/wiki/Formal_grammar)
+ [BF is Turing-Complete](http://www.iwriteiam.nl/Ha_bf_Turing.html)
+ [Brainfuck Specification](https://github.com/brain-lang/brainfuck/blob/master/brainfuck.md)