# Et si ça devenait agréable de faire du CSS ?

Pour beaucoup de développeurs, le CSS a une image d'un langage qu'il faut sans arrêt bricoler pour arriver à nos fins.
Une des raisons principales est que le CSS est très permissif et ne propose pas d'architecture en lui-même, c'est un grand bac a sable.
En conséquences, si un projet grossi, la structure CSS choisie (ou construite de manière empirique) finie par être un cauchemar à maintenir et à étendre tant il y a de collisions de règles.
Un indicateur d'un projet ayant grossi sans avoir fait évoluer sa structure CSS est le nombre de `!important` dans le code. Cela montre des conflits n'ayant pas été résolus, mais masqués (Je reviendrais sur ce point).

Je vais donc ici vous proposer une architecture CSS qui va vous permettre d'organiser votre code, de le documenter, d'éviter les conflits mais aussi d'améliorer la maintenabilité et faciliter l'extension.

Rapidement pour que tout le monde soit d'accord sur les termes que j'utilise:
```
regle {
    propriété: valeur;
    propriété: valeur !important;
}
```

## La précision

Afin de comprendre et d'éviter les conflits dans le code, il faut se préoccuper de la notion de 'précision' d'une règle CSS.
Si il y a conflit sur une propriété, c'est la règle avec la plus grande précision qui l'emporte, sauf si il y a la directive `!important`.
Si deux propriétés sont en conflit avec la même précision, c'est la règle déclarée en dernier qui l'emporte (l'ordre a son importance !).

La précision d'une règle augmente avec le nombre de membres qu'elle contient.
```
body { // précision: 1
    [...]
}

body p { // précision: 2
    [...]
}

body > p { // précision: 2
    [...]
}

.container { // précision: 1
    [...]
}

.container.disabled { // précision: 2
    [...]
}
```

Attention à l'utilisation de préprocesseurs CSS, toujours penser à la précision des règles qui seront génerées:
```
body {
    [...]

    p {
        [...]

        &.selected {
            [...]
        }
    }
}
```
Compilera en:
```
body { // précision: 1
    [...]
}

body p { // précision: 2
    [...]
}

body p.selected { // précision: 3
    [...]
}
```

La précision d'une règle contenant un identifiant est infinie, elle écrasera n'importe quelle propriété lors d'un conflit.
Cela rend l'extension du code impossible, c'est donc à proscrire dans vos bases de code CSS.

```
#page { // précision infinie
    [...]
}
```

De manière générale, il faut essayer de toujours garder la précision la plus basse possible.


## Structure HTML

Il arrive régulierement que lors de corrections ou d'extensions des fonctionnalités, qu'il faille changer la structure HTML d'une page.
Si les règles CSS se basent exclusivement sur cette dernière, les propriétés ne cibleront peut-être plus les bons blocs HTML.
Cela implique plus de maintenance et de la résilience à l'extension.
Il faut donc éviter au maximum de se baser sur la structure HTML afin de définir du style.
De manière générale il ne faut pas définir un élément en fonction de où il est, mais ce qu'il est.
Il faut donc préférer définir des classes spécifiques qui concernent le cas d'application précis de l'élément.

```
nav button {
    [...]
}
```

Pourrait être:

```
.button--nav {
    [...]
}
```

Cela réduit la dépendence à la structure HTML et cela a aussi réduit la précision, parfait.

## Découpage en composants + convention de nommage

Voici un point qui va grandement (et je pèse mes mots) améliorer la facilité de lecture et donc de maintenance de votre code.
C'est la suite logique du point précédent où il va falloir définir des composants et les isoler de leur environnement.
Je vous propose la convention de nommage BEM pour (Bloc, Element, Modifier)
Voici un lien d'un article en Anglais qui décrit très bien le fonctionnement de cette convention:
[https://css-tricks.com/bem-101/](https://css-tricks.com/bem-101/)

Pour rester simple voici ce que propose BEM:
  * Créer des 'Blocs' séparés de manière logique
  * Pour chaque Bloc, nous pouvons y ajouter:
    * Soit des Modificateurs, aussi appelés 'spécialisations', avec deux tirets
    * Soit des Elements enfants, avec deux underscores

Cela va permettre de standardiser le nommage de vos classes et faciliter la lecture du code.

Prenons l'exemple d'un simple article:
  * définition de la racine du bloc: `article`
  * des éléments qui le composent: `titre`, `contenu`
  * et enfin des spécialisations du bloc: `informatique`, `general`
  * dans son propre fichier du nom du composant: `_article.scss`

```
.article {
    [...]
}

.article--menuiserie {
    [...]
}

.article--informatique {
    [...]
}

.article__titre {
    [...]
}

.article__contenu {
    [...]
}
```

La même chose avec Sass:

```
.article {
    [...]

    &--menuiserie {
        [...]
    }

    &--informatique {
        [...]
    }

    &__titre {
        [...]
    }

    &__contenu {
        [...]
    }
}
```

Et appliqué à du HTML:

```
<body>
    <div class="article article--informatique">
        <div class="article__titre">
            Et si ça devenait agréable de faire du CSS ?
        </div>
        <div class="article__contenu">
            [...]
        </div>
    </div>
    <div class="article article--menuiserie">
        <div class="article__titre">
            L'utilisation de la scie sauteuse en milieu urbain
        </div>
        <div class="article__contenu">
            [...]
        </div>
    </div>
</body>
```

C'est en effet plus verbeux qu'une méthode moins structurée, mais à grande échelle, cela compartimente correctement les comportements attendus.
Attention toutefois au nommage de vos spécialisations ! Il ne faut pas nommer une spécialisation par une valeur qu'elle comporte, car si cette valeur change, le nom de la classe n'aura plus de sens.
Par exemple, ne pas utiliser `.article--rouge` mais plutôt spécifier la sémantique, le sens qu'il porte : `.article--important`.

## !important

Maintenant que nous avons aborder comment éviter les conflits, pourquoi avoir besoin de la directive `!important` ? Pourquoi existe-t-elle si nous pouvons nous en passer ?
Simplement car cette directive doit-être utilisée de manière proactive et non réactive.
L'utiliser en réponse à un conflit que difficile à résoudre soulève un problème de précision dans l'architecture.
Par contre, il est possible de l'utiliser dans un cas où l'on sait déjà en amont qu'une propriété DOIT surpasser toute autre règle, par exemple dans le cas d'un plein écran:

```
.article {
    [...]

    &--fullscreen {
        position: absolute !important;
        width: 100vw !important;
        height: 100vh !important;
    }
}
```

Si ces propriétés ne sont plus désirées, c'est qu'il faut retirer la classe du HTML.

## Les dépendances externes

Les dépendances externes peuvent parfois avoir des précisions plus grandes que les règles de notre code.
Au lieu de placer la directive `!important` afin de résoudre le problème et passer à la suite, ou encore de se baser sur la structure du HTML pour augmenter la précision, voici une petite astuce:

```
.classe-externe.classe-externe { // Précision: 2
    [...]
}
```

Cela augmente la précision de la règle, ce qui est obligatoire si l'on veut changer le comportement du code, mais sans aucune dépendance de classe et sans détruire l'arbre de précision avec un `!important`.

## Documentation

Et enfin le meilleur pour la fin, la documentation.
Après avoir refondu toute la structure et appliqué tous ces superbes conseils, il nous manque encore de la documentation afin de se rappeler du comportement d'un bloc, d'un élément ou d'une spécialisation.
Je vous propose une documentation simple en entête de chaque fichier de composant.
Grouper chaque élément et ses spécialisations et séparer les éléments entre eux afin de bien visualiser les différents groupes de règles :

```
/*
.article
    Composant représentant un article de blog
.article--menuiserie
    Spécialisation d'un article concernant la menuiserie
.article--informatique
    Spécialisation d'un article concernant l'informatique

.article__titre
    Élement titre d'un article

.article__contenu
    Élement de contenu d'un article

Exemple:
<div class="article article--informatique">
    <div class="article__titre">
        [...]
    </div>
    <div class="article__contenu">
        [...]
    </div>
</div>
*/

.article {
    [...]

    &--menuiserie {
        [...]
    }

    &--informatique {
        [...]
    }

    &__titre {
        [...]
    }

    &__contenu {
        [...]
    }
}
```

## Conclusion

Nous y voilà, nous avons vu qu'il faut décrire des composants pour ce qu'ils sont et non pour leur emplacement dans la page.
Nous savons maintenant qu'il ne faut pas utiliser d'identifiant dans le CSS mais aussi qu'il faut utiliser `!important` avec parcimonie.
Nous avons aussi vu qu'utiliser une convention de nommage en CSS permet d'ajouter du sens dans notre code sans augmenter la précision.
Et enfin nous avons vu comment documenter nos classes CSS pour s'y retrouver et que chaque composant soit indépendant.

Si vous étiez fachés avec le CSS et que vous êtes arrivés jusqu'ici, déjà bravo, et j'espère vous avoir au moins un peu réconcilié avec ce language.
Sinon j'espère vous avoir quand même apporté quelque chose au travers de cet article.
