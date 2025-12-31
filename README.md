# DAT
Predict compound bioactivity against the dopamine transporter (DAT) with traditional Machine Learning models and Graphic Neural Network

# Research Question Definition
1. Determine the target = DAT b/o my interest
   
# Select and Download the Data
1. Go to ChEMBL to download the data of the Bioactivity data for target CHEMBL238 (Sodium-dependent dopamine transporter)
2. Select the "Organism" as "homo sapiens"
3. Standard Type --> IC50 to convert it to an active label
4. Download the CSV
   
# Stage I - Data Cleaning
1. Keep the data with those in 'unit' nM  
2. Keep the ['Standard Value'] is numeric
3. The data now is having 3008 rows (remove 3955-3008 = 947 rows)

# Remove the Salt
In computer simulation, Na+ or Cl-s are not attending the combination of protein targeting
It is essential to remove them to prevent further noises in the next step - feature engineering.
1. Use SaltRemover to remove those unqualified data (with multiple ions)
2. The data now is having **3008 rows **, same as the previous data.

# Remove duplicated Smile
1. If the SMILE is same, take the 'median' across the data, and choose the first molecole ID to be the ID
2. The data now is having **2361 rows**

# Define the threshold b/o the concentration of the standard value (Here we use 1000)
1. We use 1000 as the threshold: >1000 = 1 (Active), <1000 = 0 (Inactive)
2. Observe the distribution of the data: 1332 for label 1 (active), and 1029 for label 0 (inactive)

# Sanity Check
1. Check if the reference drug: Methylphenidate (MPH) is standard in the new dataset
2. The corresponding ID for MPH within ChEMBL is CHEMBL796.
3. Conclusion: label - 1 found for CHEMBL796. Successful.

# Stage II - Feature Engineering - Prepare Data-type Input(2048-bit Morgan Fingerprints for RF/XGBoost)
1. Computer couldn't read the string from the SMILES column; we need to convert it to the numeric vectors.
2. Morgan Fingerprint (ECFP4)- scan each chemical environment of the atoms in each molecule
3. Encode a 'SMILES' string into a fixed length array (2048 bit)

# Stage II - Feature Engineering - PyG Graph Objects (For GNN)
1. 原子特徵提取函數
2. SMILES --> Graph
3. 封裝成PyG Data

# Stage III - Training Phase: Random Forest, XGBoost
1. Split the dataset into training one and test one in order to be trained in the future
2. Baseline model: **Random Forest, XGB**
3. Print their performance b/o AUC-ROC score, F1 Score, Precision, Recall
4. RF's performance: AUC:0.92, F1:0.85
5. XGBoost performance: AUC-ROC:0.92, **F1:0.87** 小勝RF

# 生成MPH(796)的指紋+看哪些bit是active
1. 結果: 亮起36個Bits

基準點測試 (Methylphenidate): 兩個模型預測MPH為Active的機率
RF - 0.616 - Effective
XGB - 0.649 - Effective

**0.6真的比0.5好一點而已，我們來看看GNN可以做到多少!**

# Stage III - Training Phase: GNN model!
1. GNN 的圖卷積結構確實捕捉到了 Morgan Fingerprints（位元陣列）所遺失的分子拓樸資訊。
2. Final Test AUC = **0.7885**

# Stage IV - 模型可解釋性分析 (XAI)
1. 在 RDKit 中，Morgan Fingerprint 是透過掃描原子周圍半徑（Radius）內的環境生成的。
2. 我們可以透過 bitInfo 參數來「逆向工程」，找出特定 Bit 對應的分子片段。
3. 模型優劣對比：為什麼 GNN 更具潛力？

# Conclusion
We compared traditional models including RF, XGBoost and GNN for their performance 
to predict compound bioactivities against the dopamine transporter (DAT).

1) RF 的限制： 雖然 Random Forest (AUC 0.65*) 表現尚可，
但 Morgan Fingerprint 本質上是將分子「打碎」成多個子結構片段，並統計其存在與否。
它丟失了片段之間的相對空間位置與全局拓樸結構 (Topology)。
例如，它知道分子有「苯環」和「胺基」，但不知道它們是否相隔合適的距離以嵌入 DAT 蛋白的結合口袋。

2)GNN 的優勢： Graph Neural Network (AUC 0.78*) 將分子視為圖結構 (Graph)，
直接在原子 (Nodes) 與化學鍵 (Edges) 上進行訊息傳遞 (Message Passing)。

GNN 不僅識別原子特徵，還能學習原子在分子整體中的上下文 (Context)。
這使得 GNN 能區分出**「同樣擁有苯環與胺基，但空間排列不同」的立體異構物或結構類似物** ，
因此在決定 DAT 抑制能力時，GNN 提供了比單純的結構指紋更深層的判斷依據。

