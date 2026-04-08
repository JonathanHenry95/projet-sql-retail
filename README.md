# 🛒 Analyse SQL — Dataset E-commerce Retail

## Contexte

Analyse exploratoire et métier sur un dataset de transactions e-commerce (type UCI Online Retail).  
Objectif : extraire des insights actionnables sur le comportement client, les performances produits et la saisonnalité.

---

## Ce que ce projet démontre

- Maîtrise de SQL : window fonction (`ROW_NUMBER`, `RANK`, `LAG`), CTEs, sous-requêtes
- Trame : segmentation RFM, analyse de cohortes, calcul de churn
- Capacité à formuler une question business et y répondre par une requête

---

## Structure

```
projet-sql-retail/
├── data/               # Dataset source (lien ou extrait)
├── queries/
│   ├── 01_exploration.sql       # Vue d'ensemble : volume, période, pays
│   ├── 02_top_produits.sql      # Top 10 produits par CA et par volume
│   ├── 03_segmentation_rfm.sql  # Score Récence / Fréquence / Montant
│   ├── 04_cohortes.sql          # Rétention par cohorte d'acquisition
│   └── 05_saisonnalite.sql      # CA mensuel et tendances
├── outputs/            # Résultats exportés (CSV / screenshots)
└── README.md
```

---

## Exemples de requêtes

### Segmentation RFM
```sql
WITH rfm AS (
  SELECT
    CustomerID,
    MAX(InvoiceDate)                          AS derniere_commande,
    COUNT(DISTINCT InvoiceNo)                 AS frequence,
    SUM(Quantity * UnitPrice)                 AS montant_total,
    DATEDIFF(CURRENT_DATE, MAX(InvoiceDate))  AS recence_jours
  FROM transactions
  WHERE CustomerID IS NOT NULL
  GROUP BY CustomerID
)
SELECT
  CustomerID,
  recence_jours,
  frequence,
  ROUND(montant_total, 2) AS CA,
  NTILE(4) OVER (ORDER BY recence_jours ASC)   AS score_R,
  NTILE(4) OVER (ORDER BY frequence DESC)       AS score_F,
  NTILE(4) OVER (ORDER BY montant_total DESC)   AS score_M
FROM rfm;
```

### Analyse de cohortes
```sql
WITH premiere_commande AS (
  SELECT CustomerID, DATE_TRUNC('month', MIN(InvoiceDate)) AS cohorte
  FROM transactions
  GROUP BY CustomerID
),
activite AS (
  SELECT
    t.CustomerID,
    DATE_TRUNC('month', t.InvoiceDate) AS mois_activite,
    p.cohorte
  FROM transactions t
  JOIN premiere_commande p ON t.CustomerID = p.CustomerID
)
SELECT
  cohorte,
  mois_activite,
  COUNT(DISTINCT CustomerID) AS clients_actifs
FROM activite
GROUP BY cohorte, mois_activite
ORDER BY cohorte, mois_activite;
```

---

## Dataset

[E-Commerce Churn Dataset 2025](https://www.kaggle.com/datasets/nabihazahid/e-commerce-customer-insights-and-churn-dataset)

---

## Stack

![SQL](https://img.shields.io/badge/SQL-PostgreSQL-336791?logo=postgresql&logoColor=white)
![Python](https://img.shields.io/badge/Python-Pandas-150458?logo=pandas)
