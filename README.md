# Un compilateur brain-fuck ?


# Introduction

Brainfuck est un language de programmation quelque peu ésotérique désigné par **Urban **Müller**** en 1992. Les programmes brainfuck ne sont composés que des caractères **+**, **-**, **<**, **>**, **[**, **]**, **.** et **,**.

Voici un exemple du fameux "hello world" en bf :

```brainfuck
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```
Inutile donc de préciser que la syntaxe de ce langage est très peu adaptée à l'écriture de véritables programmes. En revanche, c'est un excellent joujou pour les curieux que nous sommes et il peut s'avérer très utiles dans certaines circonstances. Laissez nous vous expliquez pourquoi...

## Un histoire de complétude ?

Lorsque l'on parle des langages de programmation, il est souvent question de la *Turing Completeness* (caractère Turing complet) de ces derniers.
Derrière cette expression un peu barbare se cache une idée simple : il s'agit de montrer qu'un langage de programmation est suffisamment riche pour pouvoir exprimer n'importe quel calcul. Nous ne rentrerons pas dans les détails formels qui se cachent derrière cette notion car il faudrait des centaines de lignes juste pour définir proprement le sujet (Des générations de chercheurs y travaillent encore, c'est dire !).

Maintenant que cette terminologie est introduite, pourquoi diable nous intéressons nous à ce sujet dans un article sur Brainfuck ? Eh bien figurez vous que ce langage de programmation a le bon goût d'être complet au sens de Turing. Malgré son aspect très rudimentaire, il est théoriquement suffisamment riche pour exprimer à peu près ce qu'on veut.

Or, il se trouve que prouver la complétude d'un langage de programmation peut être une tâche assez ardue. L'une des manières simples et de procéder par équivalence avec un langage dont on connaît déjà le caractère Turing Complet. Dans notre cas, nous voulions démontrer la complétude d'un langage d'assemblage en cours d'élaboration : le langage Eva. Pour se faire, nous avons implémenté un compilateur Brainfuck vers Eva.

## Un bref aperçu du langage EVA ...

Le langage Eva est un langage développé par l'association [CodeAnon](https://github.com/codeanonorg). Il s'agit d'un langage d'assemblage destiné à être utilisé pour programmer une machine virtuelle. Il met à disposition du programmeur un jeu d'instruction très concis :

+ `ADD/ADDC/SUB/SUBC` : opérations arithmétiques
+ `MOV` : chargement de valeur dans les registres
+ `LDR/STR` : opération sur la mémoire
+ `PUSH/POP` : opération sur la pile
+ `IN/OUT` : Entrées / Sorties
+ `CMP` : comparaison
+ `BEQ/BNEQ/BLE/BLT` : branchement conditionnels

## ... et du langage Brainfuck

Le langage brainfuck est prévu pour s'éxecuter sur une machine contenant des "cellules" de mémoire. On pourra également manipuler une tête de lecture écriture permettant de mettre à jour le contenu de ces cellules. Cette tête est appellée *data pointer*.

+ `+` : Incrémente de 1 la valeur dans la cellule pointé par le *data pointer*
+ `-` : décrémente de 1 la valeur dans la cellule pointé par le *data pointer*
+ `>` : décale le *data pointer* d'une cellule vers la droite.
+ `<` : décale le *data pointer* d'une cellule vers la gauche.
+ `[` : début d'une boucle **while** qui s'arrête lorsque le *data pointer* pointe une cellule de valeur 0.
+ `]` : fin d'une boucle **while**.
+ `,` : saisie d'un caractère dans la cellule pointée par *data pointer*
+ `.` : affichage du caractère dans la cellule pointée par *data pointer*

Nous constatons déjà que les instructions de Eva ou de Brainfuck sont à peu près du même ordre de proximité avec la machine. On ne peut manipuler que des structures très simples : des unités de mémoire (cellules, registres), et des opérations arithmétiques élémentaires (addition, soustraction).

# Méthode

## Choix d'implémentation

Pour exécuter des programmes Brainfuck sur la machine virtuelle Eva, deux solutions s'offraient à nous. La première aurait été d'écrire une deuxième machine virtuelle en langage Eva et capable d'interpréter le langage brainfuck. On aurait donc fait tourner une VM brainfuck dans une VM Eva.

Une deuxième option étant d'écrire un compilateur capable de traduire le langage Brainfuck en langage Eva. Nous avons décidé de choisir cette alternative, un peu moins dépendante des éventuels défauts du langage Eva encore en cours de conception. Le compilateur est donc implémenté dans un troisième langage (Rust).

## Traduction des programmes

La traduction à proprement parler des programmes brainfuck se décompose en plusieurs étapes. La première étape consiste à faire une analyse syntaxique du programme. On extrait ensuite une représentation abstraite de ce programme sur laquelle le compilateur va pouvoir travailler. Pour illustrer cette idée, voici un exemple de code brainfuck et son équivalent après analyse :

```brainfuck
+++[->+<]
```

```
Program(Incr, Incr, Incr, Loop(Decr, Shift_right, Incr, Shift_left))
```

On remarque que l'analyse est un peu plus raffiné qu'une simple lecture instruction par instruction. Dans le cas des boucles, on construit une instruction abstraite `Loop` dans laquelle on place la séquence d'instruction à répéter. Ceci nous amène droit à une première étape dans l'écriture du compilateur : la définition des types pour représenter les programmes. La structure générale d'un programme brainfuck est la suivante :

+ Un programme est une suite d'instructions
+ `Incr` `Decr` `Shift_left` `Shift_right` `Input` `Output` sont des instructions élémentaires
+ `Loop` est une instruction composée d'une liste d'autres instructions

Pour aller un peu plus loin, nous pouvons traduire cette première intuition de manière plus formelle en proposant une grammaire du langage brainfuck :

```
Alphabet = {"+", "-", "<", ">", ".", ","}

Incr -> "+"
Decr -> "-"
Shift_left -> "<"
Shift_right -> ">"
Input   -> ","
Output  -> "."

instruction -> Incr
instruction -> Decr
instruction -> Shift_left
instruction -> Shift_right
instruction -> Input
instruction -> Output
instruction -> "[" program "]"

program -> instruction
program -> instruction program
```

Les instructions se construisent donc de manière inductive (notons qu'en omettant l'instruction de boucle, la structure des programmes perd son aspect inductif). En **Rust** cette grammaire se traduit par la déclaration de type énuméré suivante :

```rust
// type pour les commandes brainfuck
pub enum Command {
	Inc,
	Dec,
	Shift,
	Unshift,
	Loop(Vec<Command>)
	Input,
	Output,
}

// alias pour le type "Program"
pub Program = Vec<Command>
```

On accompagnera cette définition de type d'un analyseur syntaxique construit à l'aider du module spécialisé [rust-peg](https://github.com/kevinmehall/rust-peg) :


```rust
peg::parser! {
	grammar brainfuck() for str {
		use super::{Command};
		rule ws() = quiet!{(" " / "\t" / "\n")*} // caractères blancs
		rule inc()      -> Command = v:$("+"+) {Command::Inc(v.len())}
		rule dec()      -> Command = v:$("-"+) {Command::Dec(v.len())}
		rule shift_r()  -> Command = v:$(">"+) {Command::Shift_left(v.len())}
		rule shift_l()  -> Command = v:$("<"+) {Command::Shift_right(v.len())}
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


# Sources

+ [Brainfuck - Wikipédia](https://en.wikipedia.org/wiki/Brainfuck)

+ [Formal Grammar - Wikipédia](https://en.wikipedia.org/wiki/Formal_grammar)
