# Tutoriel : Introduction à MapReduce avec Python

### **Introduction**
En tant que Data Scientists, nous sommes souvent confrontés à des quantités massives de données. Lorsque ces quantités deviennent trop grandes, de nombreuses approches classiques deviennent impraticables. Cependant, disposer de ces masses de données est une opportunité que nous souhaitons exploiter pleinement.

La technique **MapReduce** est une approche efficace pour traiter d’énormes volumes de données. Bien qu’il existe de nombreuses implémentations comme **Apache Hadoop**, ce tutoriel se concentre sur la compréhension du concept en utilisant des exemples concrets et simples avec Python.

---

## **Exemple de base : Trouver la plus longue chaîne**

Supposons que vous disposez d'une liste de chaînes de caractères et que vous souhaitez trouver la plus longue. Voici comment cela peut être fait en Python :

```python
# Fonction pour trouver la plus longue chaîne
def find_longest_string(list_of_strings):
    longest_string = None
    longest_string_len = 0 
    for s in list_of_strings:
        if len(s) > longest_string_len:
            longest_string_len = len(s)
            longest_string = s
    return longest_string

# Exemple d'utilisation
list_of_strings = ['abc', 'python', 'dima']
print(find_longest_string(list_of_strings))  # Résultat : python
```

- Cette méthode fonctionne bien pour de petites listes ou même pour des listes moyennes.
- Cependant, avec des millions d'éléments, cette approche devient lente.

---

## **Le Problème avec de Grandes Données**

Avec une liste contenant 300 millions d'éléments :

```python
large_list_of_strings = list_of_strings * 100000000
%time max_length = find_longest_string(large_list_of_strings)
```

- **Temps d'exécution** : 21.8 secondes.

**Problème** : Ce temps de réponse est inacceptable dans de nombreuses applications.

---

## **Solution : Évolutivité Horizontale (Horizontal Scaling)**

Au lieu d'utiliser un processeur plus rapide (**scaling vertical**), nous pouvons paralléliser notre calcul en :
1. Fractionnant les données en morceaux.
2. Exécutant la fonction sur chaque morceau en parallèle.
3. Combinant les résultats pour obtenir le résultat final.

Cela est rendu possible en divisant notre code en deux phases :
- **Mapper** : Transforme les données en valeurs intermédiaires.
- **Reducer** : Combine ces valeurs pour obtenir le résultat final.

---

## **Implémentation avec MapReduce**

Voici une version utilisant le principe MapReduce :

```python
# Mapper : Calculer la longueur des chaînes
mapper = len

def reducer(p, c):
    if p[1] > c[1]:
        return p
    return c

# Exemple de calcul
mapped = map(mapper, list_of_strings)  # Longueurs des chaînes
mapped = zip(list_of_strings, mapped) # Associer les chaînes à leur longueur
reduced = reduce(reducer, mapped)     # Trouver la plus longue
print(reduced)  # Résultat : ('python', 6)
```

---

## **Parallélisation avec Multiprocessing**

Pour paralléliser la phase Mapper :

```python
from multiprocessing import Pool

def chunks_mapper(chunk):
    mapped_chunk = map(mapper, chunk) 
    mapped_chunk = zip(chunk, mapped_chunk)
    return reduce(reducer, mapped_chunk)

pool = Pool(8)
data_chunks = chunkify(large_list_of_strings, number_of_chunks=8)
mapped = pool.map(chunks_mapper, data_chunks)  # Exécution en parallèle
reduced = reduce(reducer, mapped)
print(reduced)  # Résultat : ('python', 6)
```

---

## **Exemple Avancé : Word Count**

Objectif : Trouver les 10 mots les plus utilisés dans un ensemble massif de documents.

### **Mapper et Reducer pour Word Count**

```python
from collections import Counter

def mapper(text):
    tokens = text.split()
    tokens = [token.lower() for token in tokens if token.isalpha()]
    return Counter(tokens)

def reducer(cnt1, cnt2):
    cnt1.update(cnt2)
    return cnt1
```

### **Exécution avec MapReduce**

```python
data_chunks = chunkify(data, number_of_chunks=36)
mapped = pool.map(mapper, data_chunks)
reduced = reduce(reducer, mapped)
print(reduced.most_common(10))  # Top 10 mots
```

---

## **Avantages de MapReduce**

1. **Scalabilité** : Ajoutez simplement plus de processeurs ou de machines.
2. **Généricité** : Fonctionne pour une grande variété de tâches (ex. comptage, recherche, apprentissage supervisé ou non supervisé).

MapReduce est une technique puissante pour traiter des données massives et constitue la base de nombreux systèmes de Big Data modernes comme Hadoop ou Spark.
