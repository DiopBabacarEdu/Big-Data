
# Introduction à MapReduce en Python

### Introduction

Dans le domaine de la science des données, nous serons amenés à travailler souvent avec d'énormes quantités de données. Dans ces cas, les approches traditionnelles peuvent s'avérer inadaptées ou tout simplement pas faisables.&#x20;

La technique MapReduce exploite les avantages du parallélisme afin de traiter de larges volumes de données. Plusieurs implémentations existent, comme Apache Hadoop.&#x20;

Dans cette partie nous allons présenter les concepts de MapReduce de manière intuitive, avec des exemples pratiques.

---

## Exercice : Trouver la plus longue chaîne de caractères

Imaginons que nous ayons une liste de chaînes de caractères et que nous devions retourner la plus longue.

### Solution Simple

```python
# Trouver la plus longue chaîne
def find_longest_string(list_of_strings):
    longest_string = None
    longest_string_len = 0
    for s in list_of_strings:
        if len(s) > longest_string_len:
            longest_string_len = len(s)
            longest_string = s
    return longest_string

list_of_strings = ['abc', 'python', 'dima']
print(find_longest_string(list_of_strings))
```

#### Temps d'exécution pour une liste simple :

- Liste avec 3 éléments : quelques microsecondes.
- Liste avec 3 000 éléments : fonctionne toujours efficacement.

#### Problème avec 300 millions d'éléments

```python
large_list_of_strings = list_of_strings * 100000000
%time max_length = max(large_list_of_strings, key=len)
```

Temps d'exécution : 20+ secondes. Ce n'est pas acceptable dans un système réel.

---

### Approche : MapReduce

Pour résoudre ce problème à grande échelle, nous pouvons utiliser une approche parallèle. Cela nécessite :

1. De diviser les données en plusieurs blocs.
2. D'exécuter la recherche de la plus longue chaîne sur chaque bloc.
3. De réunir les résultats pour trouver la chaîne finale la plus longue.

#### Etapes :

1. **Mapper** : Calculer la longueur de chaque chaîne.
2. **Reducer** : Trouver la chaîne la plus longue.

Voici comment ces étapes sont mises en œuvre :

```python
# Mapper : Calculer la longueur des chaînes
mapper = len

def reducer(p, c):
    if p[1] > c[1]:
        return p
    return c

# Exécution MapReduce
mapped = map(mapper, list_of_strings)
mapped = zip(list_of_strings, mapped)
reduced = reduce(reducer, mapped)
print(reduced)
```

Temps d'exécution : plus rapide qu'une exécution linéaire sur une grande liste.

---

### Parallélisation avec des Blocs de Données

Nous divisons les données en plusieurs blocs :

```python
data_chunks = chunkify(list_of_strings, number_of_chunks=30)

# Mapper et Reducer sur chaque bloc
reduced_all = []
for chunk in data_chunks:
    mapped_chunk = map(mapper, chunk)
    mapped_chunk = zip(chunk, mapped_chunk)
    reduced_chunk = reduce(reducer, mapped_chunk)
    reduced_all.append(reduced_chunk)

# Reducer final
reduced = reduce(reducer, reduced_all)
print(reduced)
```

Cette approche est scalable et permet d'être exécutée sur plusieurs processeurs ou machines.

---

### Exemple : Comptage de mots

Imaginons que nous voulions compter les mots les plus fréquemment utilisés dans un grand jeu de données tout en excluant les mots courants (stop words).

#### Implémentation du MapReduce pour le comptage de mots :

```python
from collections import Counter

def mapper(text):
    tokens_in_text = text.split()
    tokens_in_text = map(clean_word, tokens_in_text)
    tokens_in_text = filter(word_not_in_stopwords, tokens_in_text)
    return Counter(tokens_in_text)

def reducer(cnt1, cnt2):
    cnt1.update(cnt2)
    return cnt1

def chunk_mapper(chunk):
    mapped = map(mapper, chunk)
    reduced = reduce(reducer, mapped)
    return reduced
```

#### Exécution avec parallélisation :

```python
%%time
data_chunks = chunkify(data, number_of_chunks=36)
mapped = pool.map(chunk_mapper, data_chunks)
reduced = reduce(reducer, mapped)
print(reduced.most_common(10))
```

Temps d'exécution : Réduction significative du temps grâce à la parallélisation.

---

### Conclusion

MapReduce est une technique puissante pour traiter des données massives de manière évolutive et efficace. Elle est adaptée à de nombreuses tâches comme le comptage, la recherche et les algorithmes d'apprentissage supervisé et non supervisé. Comprendre ses concepts de base est essentiel pour optimiser le traitement des données dans des environnements complexes.
