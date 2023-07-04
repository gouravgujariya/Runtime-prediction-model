# -*- coding: utf-8 -*-
"""complexity_computation_model (3).ipynb

Automatically generated by Colaboratory.

Original file is located at
    https://colab.research.google.com/drive/1lZJwaiLX6GyX0k4YdgtjiK3yRf9M_6Yb

# Code TimeComplexity Prediction

This Notebook is Based on creating a model for prediction of the time complexity from the give dataset
<br>
The Dataset is collected from codeforce and is converted into dataset by using Abstract syntax tree process and Graph2vec technique
<br>
Objective:Do Exploratory analysis on the dataset and check if there are any missing values in the dataset and how the features are corelated from each others. Applying neccesary preprocessing to the dataset for creating a best fit model. Using various algorithms to find best for prediction of the Time complexity.
"""

$ ipython nbconvert --to markdown notebook.ipynb

"""---
## Data Analysis and preprocessing

1. loading all necssary libraries in for the preprocessing and analysis
"""
``` python
import pandas as pd
import numpy as np
import seaborn as sns
from sklearn import datasets
from sklearn.ensemble import RandomForestClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
from sklearn.ensemble import VotingClassifier,StackingClassifier
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import GridSearchCV,cross_val_score
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import confusion_matrix
import seaborn as sns
```
```
data=pd.read_csv('/content/coderuntime.csv')

data.head(3)

data.shape
```
"""From the dataset we can see the there are total 16 columns present in the dataset which are:
1. no_of_ifs: no of if presents in algorithm
2. no_of_switches: no of switches present in algorithm
3. no_of_loop: no of loops present in a algorithm
2. no_of_break: no of the breaks in an algorithm
3. priority_queue_present: if priority is present or not
4. no_of_sort:no of sort present
5. hash_set_present:hash set is present or not in algorithm
6. hash_map_present:hash map present or not
7. recursion_present:recursion present or not
8. nested_loop_depth:nested loop present or not
9. noOfVariables:no of variables present in algorithm
10. noOfMethods:no of methods used
11. noOfJumps:no of jumps done between statements in algorithm
12.	noOfStatements:no of statement in an algorithm
13.	complexity:time complexity of the algorithm (target feature)
14.	file_name:filename to which algorithm is related

For the machine learning algorithms we can say file_name is not required as it will no teach anything to the algorithm So lets remove this feature
"""
```
df=data.drop(['file_name'],axis=1)

df.info()
```
"""### categorical feature
As we can see that complexity is objective which is needed to be converted to the numericals before giving it to the algorithm to compute
"""
```
df['complexity'].unique()
```
"""By using sklearn.preprocessing.LabelEncoder we are encoding the complexity and converting into integer"""
```
le=LabelEncoder()
df['complexity']=le.fit_transform(df['complexity'])

df['complexity'].unique()

df.head(3)

df.describe()
```
"""## Data analysis

By the fact of matter we can say that as some features as unbalanced but the are based on facts for example priority queue's worst complexity is logn so when ever present can not be converted to nsquare or some thing else so there is no effect of analysis
"""
```
sns.pairplot(data)
```
"""But we can used pandas correlation feature to compute the correlation between the features"""
```
train = df

def correlation_heatmap(train):
    correlations = train.corr()

    fig, ax = plt.subplots(figsize=(10,10))
    sns.heatmap(correlations, center=0, fmt='.2f', annot=True)
    plt.show();

correlation_heatmap(train)
```
"""as we can see that there is no feature which is 0 or negative so there we cannot also remove the features

# Model Selection

1. Splitting dataset into independent and dependent feature X,Y respectively to feed into model
"""
```
X=df.drop(['complexity'],axis=1)
Y=df['complexity']
```
"""2. standardizing the X so to increase the accuracy and meking features scaler"""
```
ss=StandardScaler()
X_s=ss.fit_transform(X)
```
"""3. Train test split"""
```
xtr,xte, ytr,yte = train_test_split(X_s,Y, test_size=0.2,random_state=True)
```
"""4. Using algorithms to predict create multilabel classifier"""
```
clf1_r = RandomForestClassifier()
clf2 = KNeighborsClassifier()
clf3 = SVC(probability=True)

clf1_r = clf1_r.fit(xtr, ytr)
clf2 = clf2.fit(xtr, ytr)
clf3 = clf3.fit(xtr, ytr)

print(cross_val_score(RandomForestClassifier(),xtr, ytr, cv=5))
print(cross_val_score(KNeighborsClassifier(),xtr, ytr, cv=5))
print(cross_val_score(SVC(),xtr, ytr, cv=5))
```
"""5. form the above reesults we can see that Ensemble model is giving best score so using ensemble model would probebly give  us best model to predict

Using different ensemble models
"""
```
from sklearn.ensemble import RandomForestClassifier
from sklearn.ensemble import AdaBoostClassifier
from sklearn.ensemble import ExtraTreesClassifier
from sklearn.ensemble import BaggingClassifier
from sklearn.ensemble import GradientBoostingClassifier
from lightgbm import LGBMClassifier
import xgboost as xgb
```
"""As manny models as tunned or give good prediction bassed on the different no of estimators so by using for loop and cross val score to get best model"""
```
for i in range(50,500,100):
  res=cross_val_score(RandomForestClassifier(n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):
  res=cross_val_score(ExtraTreesClassifier(n_estimators=i,bootstrap=True,oob_score=True),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score(AdaBoostClassifier(clf1_r,n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score(AdaBoostClassifier(clf_etc,n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score(BaggingClassifier(n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score( xgb.XGBClassifier(n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score(LGBMClassifier(n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)

for i in range(50,500,100):

  res=cross_val_score(GradientBoostingClassifier(n_estimators=i),xtr, ytr, cv=5)
  print(res.max(),i)
```
"""from The above results we can say that Random forest and Extra tree are both giving best cross val score so we wil be selecting them to create model but lets see what are the best possible parameters for the model

### hyper parameter tunning

By using gridsearchcv we are searching for the best possible prameters for the model
"""
```
model_params = {
    'random_forest': {
        'model': RandomForestClassifier(n_estimators=110),
        'params' : {
            'criterion' : ["gini", "entropy"],
            'max_features' : ["auto", "sqrt", "log2"],
            'class_weight' : ["balanced", "balanced_subsample"]
        }
    },
    'extra_tree_forest': {
        'model':ExtraTreesClassifier(n_estimators=50),
        'params' : {
            'criterion' : ["gini", "entropy"],
            'max_features' : ["auto", "sqrt", "log2"],
            'class_weight' : ["balanced", "balanced_subsample"]
        }
    }
}

scores = []

for model_name, mp in model_params.items():
    clf =  GridSearchCV(mp['model'], mp['params'], cv=5, return_train_score=False)
    clf.fit(xtr, ytr)
    scores.append({
        'model': model_name,
        'best_score': clf.best_score_,
        'best_params': clf.best_params_
    })

df_gs = pd.DataFrame(scores,columns=['model','best_score','best_params'])
df_gs

clf_rfc=RandomForestClassifier(n_estimators=110,class_weight='balanced',criterion='entropy')
clf_rfc.fit(xtr,ytr)

clf_etc=ExtraTreesClassifier(n_estimators=50,criterion='entropy',bootstrap=True,oob_score=True)
clf_etc.fit(xtr,ytr)

clf_ada=AdaBoostClassifier(clf_etc,n_estimators=350)
clf_ada=clf_ada.fit(xtr,ytr)

clf_xgb=xgb.XGBClassifier(n_estimators=50)
clf_lgb=LGBMClassifier(n_estimators=50)
clg_gb=GradientBoostingClassifier(n_estimators=50)

clf_xgb=clf_xgb.fit(xtr,ytr)
clf_lgb=clf_lgb.fit(xtr,ytr)
clg_gb=clg_gb.fit(xtr,ytr)

"""stacking
voting hard
voting soft
bagging
sgboost
light gbm
ada etc,rf
rf
etc

"""

ypre_xgb=clf_xgb.predict(xte)
ypre_lgb=clf_lgb.predict(xte)
ypre_gb=clg_gb.predict(xte)
```
"""Now lets try combining the RandomForestClassifier and ExtraForestClassifier by using varous methods like stacking and voting"""
```
from sklearn.ensemble import StackingClassifier

models = [('etc',ExtraTreesClassifier(n_estimators=50,criterion='entropy',bootstrap=True,oob_score=True )),
          ('rf',RandomForestClassifier(n_estimators=110,class_weight='balanced',criterion='entropy')),
          ('ada',clf_ada)]
stacking = StackingClassifier(estimators=models)

eclf_r_soft = VotingClassifier(estimators=[('etc',ExtraTreesClassifier(n_estimators=50,criterion='entropy',bootstrap=True,oob_score=True )),
          ('rf',RandomForestClassifier(n_estimators=110,class_weight='balanced',criterion='entropy')),
          ('ada',clf_ada)],voting='soft')

eclf_r_hard = VotingClassifier(estimators=[('etc',ExtraTreesClassifier(n_estimators=50,criterion='entropy',bootstrap=True,oob_score=True )),
          ('rf',RandomForestClassifier(n_estimators=110,class_weight='balanced',criterion='entropy')),
          ('ada',clf_ada)],voting='soft')

stacking.fit(xtr,ytr)
eclf_r_soft.fit(xtr,ytr)
eclf_r_hard.fit(xtr,ytr)
```
"""lets check for some advanced ensembling models

Here we are predicting all values or the test split for confussion matrix creation and then check for their accuracy.
"""
```
ypre_r=clf_rfc.predict(xte)
ypre_etc=clf_etc.predict(xte)
ypre_st=stacking.predict(xte)
ypre_cs=eclf_r_soft.predict(xte)
ypre_ch=eclf_r_hard.predict(xte)
ypre_ada=clf_ada.predict(xte)

from sklearn.metrics import accuracy_score
accu=[]
pre_y=[ypre_r,ypre_etc,ypre_ada,ypre_ch,ypre_cs,ypre_gb,ypre_lgb,ypre_st,ypre_xgb]
for i in pre_y:
  accu.append(accuracy_score(yte,i))

accu

sns.scatterplot(accu,np.zeros(9))

from sklearn.metrics import confusion_matrix
c_r=confusion_matrix(yte,ypre_r)
c_etc=confusion_matrix(yte,ypre_etc)
c_st=confusion_matrix(yte,ypre_st)
c_vs=confusion_matrix(yte,ypre_cs)
c_vh=confusion_matrix(yte,ypre_ch)
c_ada=confusion_matrix(yte,ypre_ada)

sns.heatmap(c_r/np.sum(c_r), annot=True,
            fmt='.2%', cmap='Blues')

sns.heatmap(c_etc/np.sum(c_etc), annot=True,
            fmt='.2%', cmap='Blues')

sns.heatmap(c_st/np.sum(c_st), annot=True,
            fmt='.2%', cmap='Blues')

sns.heatmap(c_vs/np.sum(c_vs), annot=True,
            fmt='.2%', cmap='Blues')

sns.heatmap(c_vh/np.sum(c_vh), annot=True,
            fmt='.2%', cmap='Blues')

sns.heatmap(c_ada/np.sum(c_ada), annot=True,
            fmt='.2%', cmap='Blues')

from sklearn.metrics import f1_score
y_true=yte
pred_col=[ypre_r,ypre_etc,ypre_ada,ypre_ch,ypre_cs,ypre_gb,ypre_lgb,ypre_st,ypre_xgb]
result_f1=[]
for i in pred_col:
  result_f1.append(f1_score(y_true,i,average=None))

"""From the above array we can state that the best model for prediction is voting_soft classifier which is providing 78 present f1 score"""

from sklearn.metrics import accuracy_score
accu=[]
pre_y=[ypre_r,ypre_etc,ypre_ada,ypre_ch,ypre_cs,ypre_gb,ypre_lgb,ypre_st,ypre_xgb]
for i in pre_y:
  accu.append(accuracy_score(yte,i))

accu

import seaborn as sns

sns.histplot(accu)
