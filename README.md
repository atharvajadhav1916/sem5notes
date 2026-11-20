# sem5notes

**EXP 2**
```sql
-- your Exp 02 SQL code here


-- 1. Create Dimension Tables (Descriptive Context)
CREATE TABLE Dim_Product (
    prod_id INT PRIMARY KEY,
    prod_name VARCHAR(50),
    category VARCHAR(30)
);

CREATE TABLE Dim_Customer (
    cust_id INT PRIMARY KEY,
    cust_name VARCHAR(50),
    region VARCHAR(30)
);

CREATE TABLE Dim_Store (
    store_id INT PRIMARY KEY,
    store_name VARCHAR(50),
    city VARCHAR(30)
);

CREATE TABLE Dim_Time (
    time_id INT PRIMARY KEY,
    month VARCHAR(15),
    quarter VARCHAR(10),
    year INT
);

-- 2. Create Fact Table (Numeric Measures + Foreign Keys)
CREATE TABLE Fact_Sales (
    sales_id INT PRIMARY KEY,
    prod_id INT,
    cust_id INT,
    store_id INT,
    time_id INT,
    quantity INT,
    revenue DECIMAL(10, 2),
    FOREIGN KEY (prod_id) REFERENCES Dim_Product(prod_id),
    FOREIGN KEY (cust_id) REFERENCES Dim_Customer(cust_id),
    FOREIGN KEY (store_id) REFERENCES Dim_Store(store_id),
    FOREIGN KEY (time_id) REFERENCES Dim_Time(time_id)
);

-- 3. Insert Sample Data into Dimensions
INSERT INTO Dim_Product VALUES 
(1, 'Laptop', 'Electronics'),
(2, 'Headphones', 'Accessories'),
(3, 'Desk Chair', 'Furniture');

INSERT INTO Dim_Customer VALUES 
(101, 'Rohan', 'West'),
(102, 'Priya', 'North'),
(103, 'Amit', 'West');

INSERT INTO Dim_Store VALUES 
(301, 'Reliance Digital', 'Mumbai'),
(302, 'Croma', 'Delhi');

INSERT INTO Dim_Time VALUES 
(201, 'Jan', 'Q1', 2026),
(202, 'Feb', 'Q1', 2026),
(203, 'Apr', 'Q2', 2026);

-- 4. Insert Sample Data into Fact Table
INSERT INTO Fact_Sales VALUES 
(1, 1, 101, 301, 201, 2, 120000.00),
(2, 2, 102, 302, 202, 5, 15000.00),
(3, 3, 103, 301, 203, 1, 8500.00),
(4, 1, 102, 302, 203, 1, 60000.00);

-- 5. Display Table Structures (Shows PK and FK links)
DESC Fact_Sales;

-- 6. Verify Star Schema with a Multi-Table Join
SELECT f.sales_id, c.cust_name, c.region, s.city, p.prod_name, t.month, t.year, f.quantity, f.revenue
FROM Fact_Sales f
JOIN Dim_Customer c ON f.cust_id = c.cust_id
JOIN Dim_Store s ON f.store_id = s.store_id
JOIN Dim_Product p ON f.prod_id = p.prod_id
JOIN Dim_Time t ON f.time_id = t.time_id;
```


**EXP3**
```sql
-- your Exp 03 SQL code here


-- Setup Tables & Data (Same as Exp 02)
CREATE TABLE Dim_Product (prod_id INT PRIMARY KEY, prod_name VARCHAR(50), category VARCHAR(30));
CREATE TABLE Dim_Customer (cust_id INT PRIMARY KEY, cust_name VARCHAR(50), region VARCHAR(30));
CREATE TABLE Dim_Store (store_id INT PRIMARY KEY, store_name VARCHAR(50), city VARCHAR(30));
CREATE TABLE Dim_Time (time_id INT PRIMARY KEY, month VARCHAR(15), quarter VARCHAR(10), year INT);

CREATE TABLE Fact_Sales (
    sales_id INT PRIMARY KEY, prod_id INT, cust_id INT, store_id INT, time_id INT,
    quantity INT, revenue DECIMAL(10, 2),
    FOREIGN KEY (prod_id) REFERENCES Dim_Product(prod_id),
    FOREIGN KEY (cust_id) REFERENCES Dim_Customer(cust_id),
    FOREIGN KEY (store_id) REFERENCES Dim_Store(store_id),
    FOREIGN KEY (time_id) REFERENCES Dim_Time(time_id)
);

INSERT INTO Dim_Product VALUES (1, 'Laptop', 'Electronics'), (2, 'Headphones', 'Accessories'), (3, 'Desk Chair', 'Furniture');
INSERT INTO Dim_Customer VALUES (101, 'Rohan', 'West'), (102, 'Priya', 'North'), (103, 'Amit', 'West');
INSERT INTO Dim_Store VALUES (301, 'Reliance Digital', 'Mumbai'), (302, 'Croma', 'Delhi');
INSERT INTO Dim_Time VALUES (201, 'Jan', 'Q1', 2026), (202, 'Feb', 'Q1', 2026), (203, 'Apr', 'Q2', 2026);

INSERT INTO Fact_Sales VALUES 
(1, 1, 101, 301, 201, 2, 120000.00),
(2, 2, 102, 302, 202, 5, 15000.00),
(3, 3, 103, 301, 203, 1, 8500.00),
(4, 1, 102, 302, 203, 1, 60000.00);

-- ==========================================
-- OLAP OPERATIONS
-- ==========================================

-- 1. SLICE (Filter on ONE dimension: Region = 'West')
SELECT '1. SLICE (West Region)' AS Operation;
SELECT c.region, p.prod_name, t.month, f.revenue
FROM Fact_Sales f
JOIN Dim_Customer c ON f.cust_id = c.cust_id
JOIN Dim_Product p ON f.prod_id = p.prod_id
JOIN Dim_Time t ON f.time_id = t.time_id
WHERE c.region = 'West';

-- 2. DICE (Filter on MULTIPLE dimensions: Region = 'West' AND Quarter = 'Q1')
SELECT '2. DICE (West + Q1)' AS Operation;
SELECT c.region, t.quarter, p.prod_name, f.revenue
FROM Fact_Sales f
JOIN Dim_Customer c ON f.cust_id = c.cust_id
JOIN Dim_Product p ON f.prod_id = p.prod_id
JOIN Dim_Time t ON f.time_id = t.time_id
WHERE c.region = 'West' AND t.quarter = 'Q1';

-- 3. ROLL-UP (Aggregate upward to grand totals using WITH ROLLUP)
SELECT '3. ROLL-UP (Region -> Category Subtotals)' AS Operation;
SELECT c.region, p.category, SUM(f.revenue) AS Total_Revenue
FROM Fact_Sales f
JOIN Dim_Customer c ON f.cust_id = c.cust_id
JOIN Dim_Product p ON f.prod_id = p.prod_id
GROUP BY c.region, p.category WITH ROLLUP;

-- 4. DRILL-DOWN (Go deeper: Year -> Quarter -> Month -> Product)
SELECT '4. DRILL-DOWN (Year -> Quarter -> Month -> Product)' AS Operation;
SELECT t.year, t.quarter, t.month, p.prod_name, SUM(f.revenue) AS Revenue
FROM Fact_Sales f
JOIN Dim_Time t ON f.time_id = t.time_id
JOIN Dim_Product p ON f.prod_id = p.prod_id
GROUP BY t.year, t.quarter, t.month, p.prod_name
ORDER BY t.year, t.quarter, t.month;

-- 5. PIVOT (Rotate Regions into Columns by Category)
SELECT '5. PIVOT (Category vs Region)' AS Operation;
SELECT p.category,
    SUM(CASE WHEN c.region = 'West' THEN f.revenue ELSE 0 END) AS West_Sales,
    SUM(CASE WHEN c.region = 'North' THEN f.revenue ELSE 0 END) AS North_Sales
FROM Fact_Sales f
JOIN Dim_Product p ON f.prod_id = p.prod_id
JOIN Dim_Customer c ON f.cust_id = c.cust_id
GROUP BY p.category;
```


**EXP 4**
```python
# your Exp 04 Python code here

import pandas as pd
import numpy as np
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import LabelEncoder, StandardScaler
from sklearn.model_selection import train_test_split

# 1. Create a Raw Messy Dataset
data = {
    'City': ['Mumbai', 'Pune', 'Delhi', 'Mumbai', 'Pune', 'Delhi'],
    'Age': [22, np.nan, 35, 28, 42, np.nan],
    'Salary': [35000, 48000, np.nan, 52000, 75000, 60000],
    'Purchased': ['No', 'Yes', 'Yes', 'No', 'Yes', 'No']
}
df = pd.DataFrame(data)
print("--- 1. RAW DATA ---")
print(df, "\n")

# 2. Handle Missing Values (Replace NaN with Column Mean)
imputer = SimpleImputer(strategy='mean')
df[['Age', 'Salary']] = imputer.fit_transform(df[['Age', 'Salary']])
print("--- 2. AFTER HANDLING MISSING VALUES ---")
print(df, "\n")

# 3. Encode Categorical Data (Text -> Numbers)
le_city = LabelEncoder()
df['City'] = le_city.fit_transform(df['City'])

le_target = LabelEncoder()
df['Purchased'] = le_target.fit_transform(df['Purchased'])
print("--- 3. AFTER ENCODING CATEGORICAL COLUMNS ---")
print(df, "\n")

# 4. Feature Scaling (Standardize Age & Salary)
scaler = StandardScaler()
df[['Age', 'Salary']] = scaler.fit_transform(df[['Age', 'Salary']])
print("--- 4. AFTER FEATURE SCALING ---")
print(df, "\n")

# 5. Split into Training and Testing Sets (80% Train, 20% Test)
X = df[['City', 'Age', 'Salary']]
y = df['Purchased']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
print("--- 5. TRAIN / TEST SPLIT SHAPES ---")
print("X_train shape:", X_train.shape)
print("X_test shape:", X_test.shape)
```

**EXP 5**
```python
# your Exp 05 Python code here

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 1. Sample Student Performance Dataset
df = pd.DataFrame({
    'Study_Hours': [2, 3, 4, 5, 6, 7, 8, 9, 10, 3],
    'Attendance':  [60, 65, 70, 75, 80, 85, 88, 92, 95, 62],
    'Marks':       [45, 52, 58, 65, 72, 78, 84, 90, 96, 50],
    'Department':  ['CS', 'IT', 'CS', 'IT', 'CS', 'IT', 'CS', 'IT', 'CS', 'IT']
})

# 2. Data Exploration (Printed in Terminal)
print("--- 1. FIRST 5 ROWS ---")
print(df.head(), "\n")

print("--- 2. STATISTICAL SUMMARY ---")
print(df.describe(), "\n")

print("--- 3. CORRELATION MATRIX ---")
print(df[['Study_Hours', 'Attendance', 'Marks']].corr(), "\n")

# 3. Data Visualization (4 Charts in One Window)
fig, axes = plt.subplots(2, 2, figsize=(10, 7))
fig.suptitle('Exp 05: Data Exploration & Visualization', fontsize=14, fontweight='bold')

# Plot 1: Histogram
sns.histplot(df['Marks'], bins=5, kde=True, ax=axes[0, 0], color='skyblue')
axes[0, 0].set_title('1. Marks Distribution (Histogram)')

# Plot 2: Scatter Plot
sns.scatterplot(x='Study_Hours', y='Marks', hue='Department', data=df, ax=axes[0, 1], s=80)
axes[0, 1].set_title('2. Study Hours vs Marks (Scatter)')

# Plot 3: Box Plot
sns.boxplot(x='Department', y='Marks', data=df, ax=axes[1, 0], palette='Set2')
axes[1, 0].set_title('3. Marks by Department (Box Plot)')

# Plot 4: Correlation Heatmap
corr = df[['Study_Hours', 'Attendance', 'Marks']].corr()
sns.heatmap(corr, annot=True, cmap='coolwarm', ax=axes[1, 1])
axes[1, 1].set_title('4. Feature Correlation (Heatmap)')

plt.tight_layout()
plt.show()
```

**EXP 6**
```python
# your Exp 06 Python code here

from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

# 1. Load Built-in Dataset (No external CSV needed)
iris = load_iris()
X, y = iris.data, iris.target

# 2. Split into 80% Train and 20% Test
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 3. Model 1: Decision Tree Classifier
dt_model = DecisionTreeClassifier(random_state=42)
dt_model.fit(X_train, y_train)
dt_preds = dt_model.predict(X_test)

print("--- 1. DECISION TREE CLASSIFIER ---")
print("Accuracy:", accuracy_score(y_test, dt_preds))
print("Confusion Matrix:\n", confusion_matrix(y_test, dt_preds), "\n")

# 4. Model 2: Gaussian Naive Bayes Classifier
nb_model = GaussianNB()
nb_model.fit(X_train, y_train)
nb_preds = nb_model.predict(X_test)

print("--- 2. NAIVE BAYES CLASSIFIER ---")
print("Accuracy:", accuracy_score(y_test, nb_preds))
print("Classification Report:\n", classification_report(y_test, nb_preds, target_names=iris.target_names))

# 5. Predict on a Brand-New Unseen Flower Sample
new_sample = [[5.1, 3.5, 1.4, 0.2]]
pred_class = iris.target_names[dt_model.predict(new_sample)[0]]
print(f"New Sample {new_sample} Predicted Class: {pred_class}")
```

**EXP 7**

```python
# your Exp 07 Python code here
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans

# 1. Sample Mall Customer Dataset (Income in $k, Spending Score 1-100)
df = pd.DataFrame({
    'CustomerID': range(1, 13),
    'Annual_Income':  [15, 16, 17, 18, 45, 48, 50, 52, 85, 88, 90, 95],
    'Spending_Score': [39, 81, 35, 77, 50, 55, 48, 52, 88, 92, 15, 85]
})

X = df[['Annual_Income', 'Spending_Score']]

# 2. Apply K-Means Clustering (K = 3 Customer Segments)
kmeans = KMeans(n_clusters=3, n_init=10, random_state=42)
df['Cluster'] = kmeans.fit_predict(X)

print("--- 1. CUSTOMER SEGMENTS ---")
print(df, "\n")

print("--- 2. CLUSTER CENTROIDS (Income, Spending) ---")
print(kmeans.cluster_centers_, "\n")

# 3. Visualize the Customer Clusters & Centroids
plt.figure(figsize=(8, 5))
plt.scatter(df['Annual_Income'], df['Spending_Score'], c=df['Cluster'], cmap='viridis', s=100, label='Customers')
plt.scatter(kmeans.cluster_centers_[:, 0], kmeans.cluster_centers_[:, 1], 
            c='red', marker='X', s=200, label='Centroids')

plt.title('Exp 07: Customer Segmentation using K-Means')
plt.xlabel('Annual Income (k$)')
plt.ylabel('Spending Score (1-100)')
plt.legend()
plt.grid(True)
plt.show()
```

**EXP 8**
```python
# your Exp 08 Python code here


import pandas as pd
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import dendrogram, linkage
from sklearn.cluster import AgglomerativeClustering

# 1. Sample Pattern Analysis Dataset (e.g., Customer Spend vs Visit Frequency)
df = pd.DataFrame({
    'Customer': ['C1', 'C2', 'C3', 'C4', 'C5', 'C6', 'C7', 'C8', 'C9', 'C10'],
    'Annual_Spend': [15, 18, 16, 48, 52, 50, 85, 88, 92, 90],
    'Visit_Freq':   [2,  3,  2,  8,  9,  7,  15, 16, 14, 17]
})

X = df[['Annual_Spend', 'Visit_Freq']]

# 2. Apply Agglomerative (Bottom-Up) Hierarchical Clustering
hc = AgglomerativeClustering(n_clusters=3, linkage='ward')
df['Cluster'] = hc.fit_predict(X)

print("--- 1. HIERARCHICAL CLUSTERING ASSIGNMENTS ---")
print(df, "\n")

# 3. Plot Dendrogram & Cluster Scatter Side-by-Side
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
fig.suptitle('Exp 08: Hierarchical Clustering & Dendrogram', fontsize=13, fontweight='bold')

# Plot 1: Dendrogram (Shows how points merge step-by-step)
linked = linkage(X, method='ward')
dendrogram(linked, labels=df['Customer'].values, ax=axes[0])
axes[0].set_title('1. Dendrogram (Ward Linkage)')
axes[0].set_xlabel('Customers')
axes[0].set_ylabel('Euclidean Distance')

# Plot 2: Resulting Clusters
axes[1].scatter(df['Annual_Spend'], df['Visit_Freq'], c=df['Cluster'], cmap='viridis', s=100)
for i, txt in enumerate(df['Customer']):
    axes[1].annotate(txt, (df['Annual_Spend'][i] + 1, df['Visit_Freq'][i]))
axes[1].set_title('2. Agglomerative Clusters (K=3)')
axes[1].set_xlabel('Annual Spend ($k)')
axes[1].set_ylabel('Visit Frequency')
axes[1].grid(True)

plt.tight_layout()
plt.show()
```

**EXP 9**
```python
# your Exp 09 Python code here

import pandas as pd
from mlxtend.preprocessing import TransactionEncoder
from mlxtend.frequent_patterns import apriori, association_rules

# 1. Sample Store Transaction Receipts
transactions = [
    ['Milk', 'Bread', 'Butter'],
    ['Milk', 'Bread'],
    ['Milk', 'Butter', 'Eggs'],
    ['Bread', 'Butter', 'Jam'],
    ['Milk', 'Bread', 'Butter', 'Eggs'],
    ['Milk', 'Bread', 'Jam'],
    ['Bread', 'Butter'],
    ['Milk', 'Bread', 'Butter']
]

# 2. One-Hot Encode Transactions into True/False Matrix
te = TransactionEncoder()
te_array = te.fit(transactions).transform(transactions)
df = pd.DataFrame(te_array, columns=te.columns_)

print("--- 1. ONE-HOT ENCODED TRANSACTIONS ---")
print(df, "\n")

# 3. Find Frequent Itemsets using Apriori (min_support = 40%)
frequent_itemsets = apriori(df, min_support=0.4, use_colnames=True)
print("--- 2. FREQUENT ITEMSETS (Support >= 0.4) ---")
print(frequent_itemsets, "\n")

# 4. Generate Association Rules (min_confidence = 70%)
rules = association_rules(frequent_itemsets, metric="confidence", min_threshold=0.7)
print("--- 3. ASSOCIATION RULES (Confidence >= 0.7) ---")
print(rules[['antecedents', 'consequents', 'support', 'confidence', 'lift']])
```

**EXP 10**
```python
# your Exp 10 Python code here

import numpy as np
import pandas as pd

# 1. Define Directed Web Graph (4 Web Pages: A, B, C, D)
# Adjacency Matrix: rows = FROM page, cols = TO page
pages = ['Page A', 'Page B', 'Page C', 'Page D']
adj_matrix = np.array([
    [0, 1, 1, 0],  # Page A links to B, C
    [0, 0, 1, 0],  # Page B links to C
    [1, 0, 0, 1],  # Page C links to A, D
    [0, 0, 1, 0]   # Page D links to C
], dtype=float)

N = len(pages)
d = 0.85          # Damping factor
tol = 1e-6        # Convergence threshold

# 2. Build Column-Stochastic Transition Matrix (M)
out_degrees = adj_matrix.sum(axis=1)
M = np.zeros((N, N))
for i in range(N):
    if out_degrees[i] > 0:
        M[:, i] = adj_matrix[i] / out_degrees[i]
    else:
        M[:, i] = 1.0 / N  # Handle dangling nodes with no outgoing links

print("--- 1. TRANSITION PROBABILITY MATRIX ---")
print(pd.DataFrame(M, index=pages, columns=pages).round(3), "\n")

# 3. Power Iteration to Compute PageRank
ranks = np.ones(N) / N  # Initial equal rank (0.25 each)

for iteration in range(1, 101):
    new_ranks = (1 - d) / N + d * M.dot(ranks)
    if np.linalg.norm(new_ranks - ranks, ord=1) < tol:
        print(f"Converged in {iteration} iterations!\n")
        ranks = new_ranks
        break
    ranks = new_ranks

# 4. Display Final Ranked Results
results = pd.DataFrame({
    'Web_Page': pages,
    'PageRank_Score': ranks.round(4)
}).sort_values(by='PageRank_Score', ascending=False)

print("--- 2. FINAL PAGERANK SCORES (HIGHEST TO LOWEST) ---")
print(results.to_string(index=False))
```

**EXP 11**
```python
# your Exp 11 Python code here

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import LabelEncoder, StandardScaler
from sklearn.cluster import KMeans
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, classification_report

# 1. Raw Telecom Industry Dataset
df = pd.DataFrame({
    'Tenure_Months':   [2, 4, 3, 24, 36, 48, 1, 5, 30, 42, 6, 52],
    'Monthly_Charges': [85, 90, np.nan, 55, 50, 45, 95, 88, 60, np.nan, 82, 40],
    'Support_Calls':   [5, 4, 6, 1, 0, 1, 7, 4, 1, 0, 5, 0],
    'Contract':        ['Monthly', 'Monthly', 'Monthly', 'Yearly', 'Yearly', 'Two-Year',
                        'Monthly', 'Monthly', 'Yearly', 'Two-Year', 'Monthly', 'Two-Year'],
    'Churn':           ['Yes', 'Yes', 'Yes', 'No', 'No', 'No',
                        'Yes', 'Yes', 'No', 'No', 'Yes', 'No']
})

print("--- 1. RAW INDUSTRY DATA ---")
print(df.head(), "\n")

# 2. Preprocessing Pipeline (Imputation + Encoding + Scaling)
imputer = SimpleImputer(strategy='mean')
df[['Monthly_Charges']] = imputer.fit_transform(df[['Monthly_Charges']])

le_contract = LabelEncoder()
df['Contract_Code'] = le_contract.fit_transform(df['Contract'])

le_churn = LabelEncoder()
df['Churn_Code'] = le_churn.fit_transform(df['Churn'])

features = ['Tenure_Months', 'Monthly_Charges', 'Support_Calls', 'Contract_Code']
scaler = StandardScaler()
X_scaled = scaler.fit_transform(df[features])

# 3. Unsupervised Customer Segmentation (K-Means)
kmeans = KMeans(n_clusters=2, n_init=10, random_state=42)
df['Segment'] = kmeans.fit_predict(X_scaled)
print("--- 2. CUSTOMER SEGMENTS ASSIGNED ---")
print(df[['Tenure_Months', 'Monthly_Charges', 'Support_Calls', 'Contract', 'Segment', 'Churn']], "\n")

# 4. Supervised Churn Prediction Model (Random Forest)
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, df['Churn_Code'], test_size=0.25, random_state=42
)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
preds = model.predict(X_test)

print("--- 3. CHURN PREDICTION MODEL EVALUATION ---")
print("Model Accuracy:", accuracy_score(y_test, preds))
print(classification_report(y_test, preds, target_names=le_churn.classes_, zero_division=0))

# 5. Predict Churn Risk for a New Incoming Customer
new_cust = pd.DataFrame([[3, 92.0, 5, le_contract.transform(['Monthly'])[0]]], columns=features)
new_cust_scaled = scaler.transform(new_cust)
pred_label = le_churn.inverse_transform(model.predict(new_cust_scaled))[0]
churn_prob = model.predict_proba(new_cust_scaled)[0][1] * 100
print(f"New Customer (3 mo tenure, $92/mo, 5 calls, Monthly) -> Predicted Churn: {pred_label} ({churn_prob:.1f}% risk)\n")

# 6. Executive Dashboard Visualization
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
fig.suptitle('Exp 11: Telecom Customer Churn & Segmentation Project', fontsize=13, fontweight='bold')

# Panel 1: Customer Segments
sns.scatterplot(
    x='Tenure_Months', y='Monthly_Charges', hue='Segment',
    style='Churn', data=df, s=120, palette='Set1', ax=axes[0]
)
axes[0].set_title('1. Customer Segments vs Actual Churn')
axes[0].grid(True)

# Panel 2: Feature Importance (What drives Churn?)
importances = pd.Series(model.feature_importances_, index=features).sort_values()
importances.plot(kind='barh', color='teal', ax=axes[1])
axes[1].set_title('2. Key Drivers of Customer Churn')
axes[1].set_xlabel('Importance Score')

plt.tight_layout()
plt.show()
```
