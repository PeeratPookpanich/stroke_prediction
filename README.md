##  <font color="#8450B2">_Group: 3PM 👧🏻👩🏻👩🏼👦🏻_</font> <br>
Deep Learning เป็นการเรียนรู้เชิงลึกที่เลียนแบบการทำงานของโครงข่ายประสาทของมนุษย์ โดยมีการทำงานซ้อนกันหลายๆ ชั้นหรือเรียกว่า Layer จากข้อมูลตัวอย่างเพื่อหา Pattern ของข้อมูล 
โดยการศึกษาในครั้งนี้มีจุดมุ่งหมายเพื่อเปรียบเทียบประสิทธิภาพของ **`Traditional Machine Learning (ML)`** และ **`Multilayer Perceptron (MLP)`** รวมถึงการ tuning โมเดลโดยการปรับ Hyperparameter เพื่อความสมบูรณ์ของโมเดล ทางทีมคาดหวังว่าจะได้ประสบการณ์ในเรื่องนี้ และเป็นประโยชน์ต่อผู้ที่กำลังสนใจในเรียนนี้ 

## _Key Highlight_
- ผลจากการวัดประสิทธิภาพโมเดล พบว่าในฝั่งของ Traditional ML โมเดล RandomForestClassifier ให้ Accuracy สูงที่สุดอยู่ที่ 88.26% ในขณะที่ MLP ให้ Accuracy สูงที่สุดอยู่ที่ 84.93% ซึ่งได้ค่าน้อยกว่า
- จากการศึกษาพบว่าการใช้ ReLu เป็น activation function ทั้งใน hidden และ output layer จะทำให้เกิดปัญหา error ในการเทรนโมเดล เนื่องจากการสุ่มข้อมูลใน batch แต่ละครั้งอาจสุ่มเจอค่า 0 ทั้งหมดในรอบนั้นๆ ทำให้ไม่สามารถคำนวนค่า backpropagation และ error ได้ แต่หากใช้ sigmoid จะไม่พบปัญหาดังกล่าว
- Dataset ที่เหมาะกับการ predict ค่าด้วย MLP ควรเป็น Dataset ที่มีขนาดใหญ่กว่าชุดข้อมูลที่ใช้ทำการทดลองในครั้งนี้ ทั้งนี้เพื่อให้มีข้อมูลจำนวนมากพอในการเทรนและเรียนรู้เพื่อนำไปพยากรณ์ผลลัพท์ที่แม่นยำมากยิ่งขึ้น

## 1. Introduction
การศึกษาในครั้งนี้ใช้ข้อมูลเรื่อง Stroke Prediction โดยข้อมูลจะมีลักษณะเป็น **`Binary classification`** คือการทำนายว่าผู้ป่วยคนนั้นมีโอกาสเป็นผู้ป่วยเป็นโรคหลอดเลือดสมองหรือไม่ (1: หากผู้ป่วยเป็นโรคหลอดเลือดสมอง, 0: หากผู้ป่วยไม่เป็นโรคหลอดเลือดสมอง) ซึ่งพิจารณาจากรายละเอียดข้อมูลของผู้ป่วยเช่น อายุ, เพศ, โรคความดันโลหิตสูง, โรคหัวใจ และค่าเฉลี่ยน้ำตาลในเลือด เป็นต้น

## 2. Data

### Data source
**Dataset descrption:**: Stroke Prediction Dataset (Ref: [Stroke Prediction Dataset | Kaggle](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset))))<br>
**Total Patient:** 5,110 <br>
**Total Features:** 11 <br>
**Fields:**
1) id: รหัสประจำตัวผู้ป่วย 
2) gender: "Male", "Female" หรือ "Other"  
3) age: อายุของผู้ป่วย  
4) hypertension: 0 หากผู้ป่วยไม่เป็นโรคความดันโลหิตสูง, 1 หากผู้ป่วยไม่เป็นโรคความดันโลหิตสูง  
5) heart_disease: 0 หากผู้ป่วยไม่เป็นโรคหัวใจ, 1 หากผู้ป่วยเป็นโรคหัวใจ
6) ever_married: "No" หรือ "Yes"  
7) work_type: "children", "Govt_jov", "Never_worked", "Private" หรือ "Self-employed"  
8) Residence_type: "Rural" หรือ "Urban"  
9) avg_glucose_level: ค่าเฉลี่ยน้ำตาลในเลือด  
10) bmi: ดัชนีมวลกาย
11) smoking_status: "formerly smoked", "never smoked", "smokes" หรือ "Unknown"*  
*Note: "Unknown" ในคอลัมน์ smoking_status หมายถึงไม่มีข้อมูลสำหรับผู้ป่วยรายนี้* <br>

### EDA
![image](https://user-images.githubusercontent.com/101736826/189482360-ce111321-fce8-4679-952c-d6f820168e2c.png)
- จากการสำรวจ dataset พบว่าปัจจัยที่ส่งผลต่อการเป็นโรคหลอดเลือดสมองโดยดูจากค่า correlation มากที่สุด 5 อันดับแรกดังนี้ อายุ , Heart disease, Glucose Level
Hypertension, Married Status.

### Data preparation
การเตรียมข้อมูลก่อน train model เราทำการ drop ค่า outliner ออก หลังจากนั้นจึงจัดการข้อมูล Binary category และ Multicategory โดยใช้ **`One-Hot encoding`** เพื่อเปลี่ยนข้อมูลที่เก็บในลักษณะ categorical ให้อยู่ในรูป Binary values เนื่องจากการทำ Machine leaning นั้น ต้องการข้อมูลในรูปแบบตัวเลขเพื่อใช้ในการ train และ predict โดยแปลงค่าในคอลัมน์ gender, ever_married, work_type, residence_type และ smoking_status เพื่อให้อยู่ในรูปแบบดังกล่าว <br>

เนื่องจากข้อมูลของเรามีความ imbalance เราจึงเลือกใช้ **`SMOTE`** (synthetic minority over-sampling technique) ซึ่งเป็นเทคนิคที่ใช้ในการแก้ปัญหาการจำแนกข้อมูลที่ไม่สมดุลและทำการ normalize ค่าด้วย StandardScaler <br>

และจากข้อมูลทั้งหมด 5,110 มีการแบ่ง Data splitting (train/val/test) ดังนี้<br>
ML: Train 80% และ Test 20%<br>
MLP: Train 80%, validation 20% ของ Train set และ Test 20%<br>


![image](https://user-images.githubusercontent.com/101736826/187707794-38780d34-8cc0-4fd0-95de-48e3eda8c46f.png)

### Data Cleaning: 
เราทำการ drop dataset 1 row จาก 5,110 rows เนื่องจากใน Gender show "Other" ไม่สามารถจำแนกเพศได้
### Data Transformation: 
  1. จัดการข้อมูล Binary category และ Multicategory โดยใช้ **`One-Hot encoding`** เพื่อเปลี่ยนข้อมูลที่เก็บในลักษณะ categorical ให้อยู่ในรูป Binary values เนื่องจากการทำ Machine leaning นั้นต้องการข้อมูลในรูปแบบตัวเลขเพื่อใช้ในการ train และ predict โดยแปลงค่าในคอลัมน์ gender, ever_married <br>
  2. เนื่องจาก dataset ของเรามีปัญหา class imbalance จึงเลือกใช้วิธี **`SMOTE`** (synthetic minority over-sampling technique ซึ่งเป็นเทคนิคที่ใช้ในการแก้ปัญหาการจำแนกข้อมูลที่ไม่สมดุล
  3. ทำการ normalize ค่าด้วย StandardScaler และ OneHotEncoding <br>

## 3. Network architecture
Total params: 7,354<br>
Trainable params: 7,354<br>
Non-trainable params: 0<br>
|Layer (type)|Output Shape|Number of Parameter|Activation function|
|------------|------------|-------------------|-------------------|
|hidden1 (Dense)|	(None, 89)|	801|	tanh|
|hidden2 (Dense)|	(None, 72)|	6,408|	tanh|
|dropout_1 (Dropout)| (None, 72)|	0|	-|
|output (Dense)|	(None, 1)|	73|	sigmoid|

## 4. Training
### 4.1 Traditional Machine Learning (ML)
เราใช้ Scikit-learn ซึ่งเป็น library ใน Python ในการเทรนโมเดลแบบ Traditional Machine Learning ซึ่งประกอบไปด้วย **`RidgeClassifier`**, **`LinearSVC`**, **`SVC`**, **`LogisticRegression`**, **`KNeighborsClassifier`** , **`Xgboost`** และ **`RandomForestClassifier`** <br>
จากนั้นเราใช้ **`K-Fold Cross Validation`** จำนวน 5 รอบในแต่ละโมเดลเพื่อหาค่าเฉลี่ยของ accuracy และเลือกโมเดลที่เหมาะสมกับชุดข้อมูล และนำโมเดลที่ได้ไปปรับหาหาค่า **`Hyperparameter`** โดยใช้ **`GridSearchCV`** เพื่อหาค่าที่เหมาะสมกับโมเดลนั้นๆ <br>
โดยในแต่ละ model มีการ tuning ดังนี้

- RidgeClassifier     : RidgeClassifier(alpha = 0.0001, solver = 'lsqr')
- LinearSVC           : LinearSVC(C= 0.001,multi_class = 'crammer_singer',penalty = 'l2',loss='hinge')
- SVC                 : SVC(C= 1.5, gamma = 'scale', kernel = 'rbf', random_state = 0)
- LogisticRegression  : LogisticRegression(C= 1, max_iter= 100,solver = 'sag', penalty = 'l2')
- KNeighborsClassifier: KNeighborsClassifier(n_neighbors=5, leaf_size = 10, weights='distance')
- Xgboost             : XGBClassifier(eval_metric= 'error', learning_rate=0.1)
- RandomForestClassifier: RandomForestClassifier(criterion= 'gini', random_state = 0, n_estimators= 100)

### 4.2 Multilayer Perceptron (MLP)
ในการเทรนโมเดล Multilayer Perceptron (MLP) เราเลือกใช้ The Keras ecosystem (KerasTuner) โดยจะ trial-and-error ในการปรับหา Hyperparamet เพื่อหาโมเดลที่ดีที่สุดเพื่อพยากรณ์การเป็นโรคหลอดเลือดสมองสำหรับ dataset ข้างต้น<br>
โดยรายละเอียดของข้อมูลในการ trial-and-error มีดังนี้

|Hyperparameter	|List of value	|Best value for dataset|
|---------------|---------------|----------------------|
|Number of layer|[1,2,3]       	|3|
|Number of node	|[21, 22, ..., 100]	|89, 72|
|Learning rate	|[0.01, 0.001, 0.0001, 0.00001]	|ExponentialDecay <br>(itial_learning_rate=1e-2, <br> decay_steps=10000, <br> decay_rate=0.9)|
|Activation	    |[relu, tanh, sigmoid]	|tanh, sigmoid|
|Optimizer      |Adam          | Adam|
|Batch size     |-             |16|
|Epoch          |-             |100|
|Loss           |-             |binary_crossentropy|

## 5. Results
โดยผลลัพธ์ในตารางข้อ 5.1 และ 5.2 เป็นผลลัพธ์ที่ได้จาก test set ทั้ง 2 ตาราง
### 5.1 Traditional Machine Learning (ML)

|Classification algorithm|	Accuracy|	Precision|	Recall	|F1|
|------------------------|----------|----------|----------|--|
|RandomForest👏	|	88.26%|	25.93%|	14.89%|	18.92%|
|KNN|	82.68%	|40.74%|	13.17%|	19.91%|
|XGBClassifier	|76.32%|	57.41%|	12.40%|	20.39%|
|LogisticRegression	|75.83%|	70.37%|	14.13%|	23.53%|
|SVC|	75.24%	|72.22%|	14.08%|	23.57%|
|RidgeClassifier	|73.78%|	72.22%|	13.36%|	22.54%|
|LinearSVC	|72.31%|	77.78%|	13.41%|	22.89%|


### 5.2 Multilayer Perceptron (MLP)
โดยค่าเฉลี่ยของ Accuracy ด้านบนนั้น มากจากการคำนวณค่าเฉลี่ยของ Accuracy ในการเทรนโมเดลลด้วย initial random weights ที่แตกต่างกัน 5 รอบ <br>
ทำการเทรนโดยใช้ <br>
|CPU/GPU/TPU Name|Training Duration (MLP)|
|------------|------------|
|AMD Ryzen 7 4700U |6m|
|Google Colab: Tesla T4 |15m| 
|Google Colab: Tesla P100-PCIE-16GB|14m|  

ได้ค่า Mean และ SD ดังนี้ <br>
|round|Accuracy|Precision|Recall|F1|
|----|--------|----------|----|----|
|1|84.95% |46.30% |16.67% |24.51%|
|2|85.13% |40.74% |15.49% |22.45%|
|3|85.32% |44.44% |16.67% |24.24%|
|4|85.57% |46.30% |20.33% |28.25%|
|5|85.91% |46.30% |17.86% |25.77%|
|AVG|85.77% |44.81%|17.40%|25.04%|

<br>**Mean Accuracy of Test set:** 0.8577299412915853<br>
**STD:** 0.010709705121340223 <br>
**Mean±SD of Accuracy** = ( 0.8684396464129255 , 0.8470202361702451 )

#### กราฟการแสดงผลการเทรนโมเดลด้วย data train set vs. data test set
![image](https://user-images.githubusercontent.com/101736826/189178365-9b99fe9e-53bf-44fb-9ac9-915cee66bf19.png)
<br> จากกราฟสังเกตได้ว่า ค่า accuracy และ loss ของ validation มีการขยับเข้าใกล้ค่า train set เมื่อมีการเทรนโมเดล 500 epoch แสดงให้เห็นว่าโมเดล goodfit
  

## 6.Discussion
- สำหรับการ train model หนึ่งในสิ่งสำคัญคือการเลือกใช้ฟีเจอร์เพื่อไม่ให้ model มีความ overfit มากเกินไป ดังนั้น เราจึงเริ่มจากการดูค่า correlation ของตัวแปรต่างๆ ต่อการเป็นโรคหลอดเลือดสมอง (stroke) ซึ่งหาก correlation มีค่ามาก หมายถึงมีความสัมพันธ์ต่อการเป็น stroke มาก เช่น อายุ การเป็นโรคหัวใจ เป็นต้น
- การ normalization เราใช้ StandardScaler เนื่องจากข้อมูลในแต่ละ Features มีการแจกแจงปกติอยู่แล้ว
- เมื่อลองทำการเทรนข้อมูลที่ imbalance พบว่าค่า accuracy ของ data train set และ test set มีค่าสูงมาก (> 0.9) แต่ค่า Precision, Recall และ f1 มีค่าต่ำมาก (< 0.1) ดังนั้น เราจึงแก้ไขด้วยการทำ **`SMOTE`** (synthetic minority over-sampling technique)
- ข้อมูลในเรื่อง Stroke Prediction มีความ imbalance เราจึงเลือกใช้ SMOTE (synthetic minority over-sampling technique)  จัดการกับข้อมูลในชุดนี้ก่อนจะนำไปใช้สร้าง Model จริง<br>
ในข้อมูล Train Set มีเพียง 195 instances เท่านั้นที่เป็น class1 (หากผู้ป่วยเป็นโรคหลอดเลือดสมอง) แต่ในขณะที่ class0 (หากผู้ป่วยไม่เป็นโรคหลอดเลือดสมอง) มีถึง 3,892 instances<br>
จากเหตุผลด้านบนทำให้ Model ไม่สามารถหา Pattern ที่แน่นอนของ class1 ได้ดีนัก ส่งผลให้ค่า Accuracy, Precision, Recall, F1 ของ class1 น้อยตามไปด้วย
- การเทรนโมเดล MLP จำเป็นต้องทำการ tuning hyperparameter และใช้ทรัพยากรในการเทรนโมเดลมาก อย่างไรก็ตาม เนื่องจาก dataset นี้มีข้อมูลที่ไม่มากนัก ดังนั้น การใช้ traditional ML ซึ่งสามารถ tuning hyperparameter และใช้ทรัพยากรน้อยกว่าจึงเหมาะกับ dataset ชุดนี้มากกว่า

## 7. Conclusion
- Dataset ที่เหมาะกับการ predict ค่าด้วย MLP ควรเป็น Dataset ที่มีขนาดใหญ่กว่าชุดข้อมูลที่ใช้ทำการทดลองในครั้งนี้ ทั้งนี้เพื่อให้มีข้อมูลจำนวนมากพอในการเทรนและเรียนรู้เพื่อนำไปพยากรณ์ผลลัพท์ที่แม่นยำมากยิ่งขึ้น
- ผลจากการเทรนโมเดลพบว่า RandomForestClassifier ซึ่งเป็น traditional ML ให้ค่า Accuracy สูงที่สุดอยู่ที่ 0.8826 ในขณะที่ MLP ให้ค่า Accuracy สูงที่สุดอยู่ที่ 0.8577 เท่านั้น
    
|Performance<br>Measures|MLP|RandomForest👏|KNN|XGBClassifier|LogisticRegression|
|-----------------------|---|------------|---|-------------|------------------|     
|1. Accuracy |85.77% |88.26% |82.68% |76.32% |75.83% |  
|2. Precision |85.77% |25.93% |40.74% |57.41% |70.37% |  
|3. Recall |17.40% |14.89% |13.17% |12.40% |14.13% |  
|4. F1 Score |25.04% |18.92% |19.91% |20.39% |23.53% |

## 8. References
- เทคนิคการเขียนโค้ดด้วยวิธี automl ของ The Keras ecosystem: <br>
    **Authors:** Luca Invernizzi, James Long, Francois Chollet, Tom O'Malley, Haifeng Jin <br>
    **Last modified:** 2021/10/27 <br>
    **Link:** https://keras.io/guides/keras_tuner/getting_started/ <br>
    
- เทคนิคการปรับโมเดล: <br>
    **Authors:** SIDDHESH SAWANT <br>
    **Last modified:** 2021/03/26 <br>
    **Link:** https://www.kaggle.com/code/siddheshera/stroke-eda-smote-9-models-90-accuracy/comments <br>

    **Authors:** OHM-Songpol, nuijth, robinoud, MartRideratGamaGama <br>
    **Last modified:** 2022/02/07 <br>
    **Link:** https://github.com/robinoud/BADS7604_HW3_Deep-Learning
 
 - API Keras  ของ Tensorflow Libary <br>
    **Link:** https://www.tensorflow.org/api_docs/python/tf
    
 - รู้จักกับปัญหา Dying ReLU <br>
    **Authors:** Kenneth Leung <br>
    **Last modified:** 2021/03/31 <br>
    **Link:** https://towardsdatascience.com/the-dying-relu-problem-clearly-explained-42d0c54e0d24
 
## _End Credit_

|Working Percentage|  ID |  Name | Details |
|------------------|-----|-------|---------|
|25% |6410422005     |Metpiya Learakkakorn|Prepare dataset, Data cleaning, EDA, ML, MLP, Report, Conclusion|    
|25% |6410422015     |Khodchapan Vitheethum |Prepare dataset, Data cleaning, EDA, ML, MLP, Report, Conclusion|
|25% |6410422017     |Peerat Pookpanich |Prepare dataset, Data cleaning, EDA, ML, MLP, Report, Conclusion|
|25% |6410422031     |Anyamanee Pornpanvattana |Prepare dataset, Data cleaning, EDA, ML, MLP, Report, Conclusion |

_งานชิ้นนี้เป็นส่วนหนึ่งของ วิชาการเรียนรู้เชิงลึก หลักสูตรวิทยาศาสตรมหาบัณฑิต สาขาวิชาการวิเคราะห์ข้อมูลและวิทยาการข้อมูล มหาวิทยาลัยสถาบันบัณฑิตพัฒนบริหารศาสตร์_
