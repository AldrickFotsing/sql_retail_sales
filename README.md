# 📊 Analyse des Ventes de Détail - SQL & PostgreSQL

## 📝 Description du Projet
Ce projet consiste en une analyse de données de ventes au détail à l'aide de **PostgreSQL**. L'objectif est de nettoyer un jeu de données brut, de réaliser une exploration de données (EDA) et de répondre à des problématiques métiers clés (KPI) à l'aide de requêtes SQL intermédiaires.

Ce projet démontre des compétences clés en gestion de bases de données, notamment le nettoyage de données, les agrégations, les extractions temporelles et l'utilisation de structures conditionnelles (`CASE WHEN`).

---

## 🛠️ Structure de la Base de Données & Préparation

### 1. Création de la Table
La première étape consiste à définir la structure accueillant nos données de ventes.

```sql
CREATE DATABASE sql_project_p2;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date       DATE,
    sale_time       TIME,
    customer_id     INT,
    gender          VARCHAR(15),
    age             INT,
    category        VARCHAR(15),    
    quantiy        INT, 
    price_per_unit  FLOAT,
    cogs            FLOAT,
    total_sale      FLOAT
);
```

---

### 2. Nettoyage des Données (Data Cleaning)
Afin de garantir la fiabilité de nos analyses, les lignes contenant des valeurs manquantes (NULL) ont été identifiées puis supprimées.

```sql
-- Verifions l'existence de variable manquante dans la colonne transactions_id
SELECT * FROM retail_sales
WHERE transactions_id IS NULL;

-- Verifions l'existence de variable manquante dans la colonne sale_time
SELECT * FROM retail_sales
WHERE sale_time IS NULL;

-- Modifions quantiy en quantity
ALTER TABLE retail_sales RENAME COLUMN quantiy TO quantity; 

-- Verifions la presence de valeurs manquantes dans nos donnees
SELECT * FROM retail_sales
WHERE 
	transactions_id IS NULL
	OR 
	sale_time IS NULL
	OR 
	gender IS NULL
	OR 
	category IS NULL
	OR 
	quantity IS NULL
	OR
	cogs IS NULL
	OR 
	total_sale IS NULL;

-- Supprimons les lignes ayant des valeurs manquantes
DELETE FROM retail_sales
WHERE 
	transactions_id IS NULL
	OR 
	sale_time IS NULL
	OR 
	gender IS NULL
	OR 
	category IS NULL
	OR 
	quantity IS NULL
	OR
	cogs IS NULL
	OR 
	total_sale IS NULL;


```

---

### 3. 🔍 Exploration des Données (EDA)
Quelques requêtes pour comprendre la volumétrie et la diversité de nos données après nettoyage :

```sql
-- Combien de ventes avons nous fait ?
SELECT COUNT(*) "Total des ventes" FROM retail_sales;

-- Combien de clients avons nous ?
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;

-- Quels sont nos differentes categories de produits
SELECT DISTINCT category FROM retail_sales;


```

---

### 4. 📈 Analyses & Problématiques Métiers (Business Questions)


* Q.1 Écrire une requête SQL pour récupérer toutes les colonnes des ventes effectuées le '2022-11-05'
```sql
SELECT * FROM retail_sales
WHERE sale_date = '2022-11-05';
```

* Q.2 Écrire une requête SQL pour récupérer toutes les transactions dont la catégorie est 'Clothing' (Vêtements) et la quantité vendue est supérieure à 4 au cours du mois de novembre 2022
```sql
SELECT 
	category, 
	SUM(quantity)
FROM retail_sales 
WHERE category = 'Clothing'
	AND 
	sale_date BETWEEN '2022-11-01' AND '2022-11-30'
	AND
	quantity >= 4
	GROUP BY 1;
```

 * Q.3 Écrire une requête SQL pour calculer le montant total des ventes (total_sale) pour chaque catégorie
```sql
 SELECT 
 	DISTINCT category,
	SUM(total_sale) AS "Montant total des ventes"
FROM retail_sales
GROUP BY 1
ORDER BY category DESC
;
```

* Q.4 Écrire une requête SQL pour trouver l'âge moyen des clients ayant acheté des articles de la catégorie 'Beauté'
```sql
SELECT 
	ROUND(AVG(age), 2) AS "Age moyen"
FROM retail_sales
WHERE category = 'Beauty';
```

* Q.5 Écrire une requête SQL pour trouver toutes les transactions où le montant total des ventes est supérieur à 1000
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
ORDER BY total_sale DESC;
```

* Q.6 Écrire une requête SQL pour trouver le nombre total de transactions (transactions_id) effectuées par chaque genre dans chaque catégorie
```sql
SELECT 
	category,
	gender,
	COUNT(*) AS "Nombre total de transactions"
FROM retail_sales
GROUP
	BY 
	category,
	gender
ORDER 
	BY 
	3 DESC;
```

* Q.7 Écrire une requête SQL pour calculer le panier moyen (ou la vente moyenne) pour chaque mois. Déterminer quel a été le meilleur mois de vente (en termes de chiffre d'affaires) pour chaque année.
```sql
SELECT 
	EXTRACT(YEAR FROM sale_date) AS "Annee",
	EXTRACT(MONTH FROM sale_date) as "Mois",
	AVG(total_sale) AS "Panier moyen"
FROM retail_sales
GROUP BY 1, 2
ORDER BY 3 DESC;
```

* Q.8 Écrire une requête SQL pour trouver les 5 meilleurs clients en fonction du montant total des ventes le plus élevé
```sql
SELECT 
	customer_id,
	SUM(total_sale) AS "Montant total des ventes"
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

* Q.9 Écrire une requête SQL pour trouver le nombre de clients uniques ayant acheté des articles dans chaque catégorie
```sql
SELECT 
	category,
	COUNT(DISTINCT customer_id) AS "Nombre de clients uniques"
FROM retail_sales
GROUP BY 1;
```

* Q.10 Écrire une requête SQL pour définir chaque tranche horaire et le nombre de commandes associé. Exemple : Matin <= 12h, Après-midi Entre 12h et 17h, Soir > 17h
```sql
WITH hourly_sale AS (
    SELECT 
        CASE 
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Matin'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Après-midi'
            ELSE 'Soir'    
        END as shift
    FROM retail_sales
)
SELECT 
    shift, 
    COUNT(*) AS "Nombre de commandes"
FROM hourly_sale
GROUP BY shift
ORDER BY "Nombre de commandes" DESC;

-- Fin du projet
```

---

### 5. 🎯 Conclusions & Acquis Techniques

Ce projet m'a permis de valider des compétences clés en SQL intermédiaire :

- **Data Cleaning** : Traitement des valeurs `NULL` et modification de structures de tables (`ALTER TABLE`).

- **Calculs Statistiques** : Utilisation avancée des fonctions d'agrégation (`SUM`, `AVG`, `COUNT`, `DISTINCT`).

- **Logique Conditionnelle** : Création de segments personnalisés avec CASE WHEN.

---

## 👤 Auteur
**MOUAFO FOTSING FRANCK ALDRICK** 
*Junior Data Scientist & Analyst* 
[LinkedIn](https://www.linkedin.com/in/aldrick-fotsing-81a287370/)
