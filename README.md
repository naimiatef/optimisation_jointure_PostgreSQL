# optimisation_jointure_PostgreSQL

### Cas de Test des Jointures

**Données Exemple :**  
```sql
create table emp(deptid int, empid int);
create table dept(deptid int, salary int);

insert into emp(deptid, empid)
select n, random()*1000
from generate_series(1, 50000) n;

insert into dept(deptid, salary)
select n, random()*1000
from generate_series(1, 20000) n;

create index idx_dept1 on emp(deptid);
create index idx_dept2 on dept(deptid);
```

---

### Types de Jointures

#### 1. **Nested Loop (Boucle imbriquée) :**
   - **Description :**  
     La jointure **nested loop** joint deux tables en récupérant les résultats d'une table et en interrogeant l'autre table pour chaque ligne de la première table.
   - **Caractéristiques :**
     - Moins performant des types de jointure.
     - Rapide pour produire le premier enregistrement.
     - Peut avoir des performances négatives si le second enfant est lent.
     - Seul type de jointure capable d'exécuter un **CROSS JOIN**.
     - Seul type de jointure capable de traiter des conditions de jointure inégales.

   - **Exemple de requête :**
     ```sql
     explain analyze select * from emp e, dept d where e.deptid < d.deptid;
     ```

---

#### 2. **Hash Joins (Jointure par hachage) :**
   - **Description :**  
     La **hash join** charge les enregistrements candidats d'une des tables dans une table de hachage, puis elle est interrogée pour chaque enregistrement de l'autre table.
   - **Caractéristiques :**
     - Ne peut être utilisé que pour des conditions de jointure d'égalité.
     - Plus performant pour joindre une grande table avec une petite table.
     - Uniquement pour les types de données pouvant être hachés.
     - Démarrage lent en raison du hachage de la table la plus petite.
     - Les performances peuvent être impactées négativement si les statistiques des tables sont obsolètes ou incorrectes.

   - **Exemple de requête :**
     ```sql
     explain analyze select * from emp e, dept d where e.deptid = d.deptid;
     ```

---

#### 3. **Merge Joins (Jointure par fusion) :**
   - **Description :**  
     La **merge join** combine deux listes triées comme une fermeture éclair. Les deux côtés de la jointure doivent être triés au préalable.
   - **Caractéristiques :**
     - Ne peut être utilisé que pour des conditions de jointure d'égalité.
     - Généralement, le plus performant pour les grands ensembles de données.
     - Nécessite des entrées triées, ce qui peut entraîner des tris lents ou des scans d'index.
     - Démarrage lent, car tous les tuples d'index sont lus et triés.

   - **Exemple de requête :**
     ```sql
     explain analyze select * from emp e, dept d where e.deptid = d.deptid;
     ```

   - **Création de l'index :**
     ```sql
     create index idx_dept1 on emp(deptid);
     create index idx_dept2 on dept(deptid);
     ```

---

### Stratégies de Jointures dans PostgreSQL

| Stratégie de Jointure  | **Nested Loop Join**                       | **Hash Join**                                   | **Merge Join**                                    |
|------------------------|--------------------------------------------|------------------------------------------------|---------------------------------------------------|
| **Algorithme**          | Pour chaque ligne de la table extérieure, scanne la table intérieure | Crée un hash de la table intérieure, scanne la table extérieure, sonde le hash | Trie les deux relations et fusionne les lignes     |
| **Indexes utiles**      | Index sur les clés de jointure de la table intérieure | Aucun                                         | Index sur les clés de jointure des deux relations |
| **Bonne stratégie si**  | La table extérieure est petite             | La table de hachage tient dans **work_mem**     | Les deux tables sont grandes                      |

---

### Conclusion
Les jointures sont essentielles pour combiner des données provenant de plusieurs tables. Le choix du type de jointure (nested loop, hash join, merge join) dépend de plusieurs facteurs, comme la taille des tables, la présence d'index, et le type de condition de jointure.
