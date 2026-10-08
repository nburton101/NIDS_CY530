CY530 - CICIDS2017 Network Intrusion Detection

This repository contains the code/experiments for the Network Security research paper in SEMO's CY530 Course. These experiments were run on Ubuntu LTS, Python 3.11, and the CICIDS2017 dataset

Baseline Repo: https://github.com/mohakapoor/Network_Anomaly_Detection_CICIDS2017

Step 1: Clone the GitHub repository

	git clone https://github.com/nburton101/NIDS_CY530.git
	cd NIDS_CY530

Step 2: Install Python 3.11
	
	curl -LsSf https://astral.sh/uv/install.sh | sh
	uv python install 3.11
	uv venv --python 3.11
	source .venv/bin/activate

Step 3: Install needed packages
	
	uv pip install -r requirements.txt
	uv pip install nbconvert ipykernel

Step 4: Download CICIDS2017 dataset[https://www.unb.ca/cic/datasets/ids-2017.html] (MachineLearningCSV.zip) from the CICIDS2017 website 

create a raw directory
	bash
	mkdir raw
	
place extracted CSV files in raw/ 

Step 5: Create a 'final' directory and run the preprocessing notebooks from the baseline in the repo

	mkdir final
	
	Run the following notebooks in order:
		1) jupyter nbconvert --to notebook --execute split.ipynb --output split_executed.ipynb
		2) jupyter nbconvert --to notebook --execute cleaning.ipynb --output cleaning_executed.ipynb

Edit preprocessing.ipynb to proper Linux file path
	nano preprocessing.ipynb
	Find these two lines near the end of the notebook:
		mc_train.to_parquet(r'final\\train_mc.parquet')
		mc_test.to_parquet(r'final\\test_mc.parquet')
	Change these lines to:
		mc_train.to_parquet(r'final/train_mc.parquet')
		mc_test.to_parquet(r'final/test_mc.parquet')
	Press Ctrl+O (the letter 'o'), then Enter, then Ctrl+X

	Run final notebook
		3) jupyter nbconvert --to notebook --execute preprocessing.ipynb --output preprocessing_executed.ipynb

	The notebooks will then create:
	final/train_mc.parquet
	final/test_mc.parquet


Step 6: Create Experiment 1 dataset reducing Brute Force (2), DDoS (3), DoS (4) and PortScan (5) classes by 50%
	
	python -c "import pandas as pd; df=pd.read_parquet('final/train_mc.parquet'); keep={2:3708,3:4000,4:4000,5:4000}; 	parts=[df[df.Attack.isin([0,1,6])]]+[df[df.Attack==c].sample(n=n,random_state=42) for c,n in keep.items()]; 		out=pd.concat(parts).sample(frac=1,random_state=42); out.to_parquet('final/train_mc_exp1_majority.parquet'); 		print(out['Attack'].value_counts().sort_index()); print('Total:',len(out))"

** Results should mirror this:
	Attack
	0    8000
	1     974
	2    3708
	3    4000
	4    4000
	5    4000
	6    1718
	Name: count, dtype: int64
	Total: 26400

Step 7: Create Experiment 2 dataset, reducing Brute Force (2), DDoS (3), DoS (4) and PortScan (5) by an additional 25%

	python -c "import pandas as pd; df=pd.read_parquet('final/train_mc.parquet'); keep={2:1854,3:2000,4:2000,5:2000};parts=[df[df.Attack.isin([0,1,6])]]+[df[df.Attack==c].sample(n=n,random_state=42) for c,n in keep.items()]; out=pd.concat(parts).sample(frac=1,random_state=42); out.to_parquet('final/train_mc_exp2_majority.parquet');print(out['Attack'].value_counts().sort_index()); print('Total:',len(out))"

** Results should mirror this:
	Attack
	0    8000
	1     974
	2    1854
	3    2000
	4    2000
	5    2000
	6    1718
	Name: count, dtype: int64
	Total: 18546

Step 8: Verify baseline and experiment class distribution:

	python -c "import pandas as pd; files=['final/train_mc.parquet','final/train_mc_exp1_majority.parquet','final/train_mc_exp2_majority.parquet']; [print('\n'+f, '\n', (lambda df: df['Attack'].value_counts().sort_index())		(pd.read_parquet(f)), '\nTotal:', len(pd.read_parquet(f))) for f in files]"

** Results should mirror this:
final/train_mc.parquet
Attack
0    8000
1     974
2    7417
3    8000
4    8000
5    8000
6    1718
Name: count, dtype: int64
Total: 42109

final/train_mc_exp1_majority.parquet
Attack
0    8000
1     974
2    3708
3    4000
4    4000
5    4000
6    1718
Name: count, dtype: int64
Total: 26400

final/train_mc_exp2_majority.parquet
Attack
0    8000
1     974
2    1854
3    2000
4    2000
5    2000
6    1718
Name: count, dtype: int64
Total: 18546

Step 9: Train the baseline:
	
	python training_scripts/multiclass_lightgbm.py

**Results should mirror this:

Tuning complete.
Best Hyperparameters Found:
{'class_weight': 'balanced', 'learning_rate': 0.17803838031107141, 'max_depth': 6, 'min_child_samples': 10, 'n_estimators': 10, 'num_leaves': 50}

Best Mean F1-Score (macro) from CV: 0.9830
Saved best model to: models/lightgbm/multiclass_lightgbm.joblib

Test Accuracy: 0.9873
Test F1-weighted: 0.9874

Classification report (test):
              precision    recall  f1-score   support

           0       0.98      0.97      0.97      5000
           1       0.90      0.99      0.94       981
           2       1.00      0.99      0.99      2039
           3       1.00      1.00      1.00      5000
           4       0.99      0.98      0.99      4871
           5       1.00      1.00      1.00      5000
           6       0.93      0.99      0.96       433

    accuracy                           0.99     23324
   macro avg       0.97      0.99      0.98     23324
weighted avg       0.99      0.99      0.99     23324

Classification report (train):
              precision    recall  f1-score   support

           0       1.00      0.98      0.99      8000
           1       0.94      1.00      0.97       974
           2       1.00      1.00      1.00      7417
           3       1.00      1.00      1.00      8000
           4       0.99      1.00      1.00      8000
           5       1.00      1.00      1.00      8000
           6       0.97      1.00      0.99      1718

    accuracy                           0.99     42109
   macro avg       0.98      1.00      0.99     42109
weighted avg       0.99      0.99      0.99     42109


Step 10: Train Experiment 1

	python training_scripts/multiclass_lightgbm_exp1_majority.py

** Results should mirror this:

Tuning complete.
Best Hyperparameters Found:
{'class_weight': 'balanced', 'learning_rate': 0.17803838031107141, 'max_depth': 6, 'min_child_samples': 10, 'n_estimators': 10, 'num_leaves': 50}

Best Mean F1-Score (macro) from CV: 0.9834
Saved best model to: models/lightgbm/multiclass_lightgbm_exp1_majority.joblib

Test Accuracy: 0.9842
Test F1-weighted: 0.9842

Classification report (test):
              precision    recall  f1-score   support

           0       0.98      0.96      0.97      5000
           1       0.90      0.99      0.94       981
           2       0.99      0.99      0.99      2039
           3       0.99      1.00      0.99      5000
           4       0.99      0.98      0.98      4871
           5       1.00      1.00      1.00      5000
           6       0.95      0.99      0.97       433

    accuracy                           0.98     23324
   macro avg       0.97      0.99      0.98     23324
weighted avg       0.98      0.98      0.98     23324

Classification report (train):
              precision    recall  f1-score   support

           0       1.00      0.98      0.99      8000
           1       0.93      1.00      0.97       974
           2       1.00      1.00      1.00      3708
           3       1.00      1.00      1.00      4000
           4       0.99      1.00      0.99      4000
           5       0.99      1.00      1.00      4000
           6       0.99      1.00      0.99      1718

    accuracy                           0.99     26400
   macro avg       0.99      1.00      0.99     26400
weighted avg       0.99      0.99      0.99     26400


Step 11: Train Experiment 2

	python training_scripts/multiclass_lightgbm_exp2_majority.py

** Results should mirror this:
Tuning complete.
Best Hyperparameters Found:
{'class_weight': 'balanced', 'learning_rate': 0.17803838031107141, 'max_depth': 6, 'min_child_samples': 10, 'n_estimators': 10, 'num_leaves': 50}

Best Mean F1-Score (macro) from CV: 0.9815
Saved best model to: models/lightgbm/multiclass_lightgbm_exp2_majority.joblib

Test Accuracy: 0.9838
Test F1-weighted: 0.9839

Classification report (test):
              precision    recall  f1-score   support

           0       0.97      0.97      0.97      5000
           1       0.90      0.99      0.94       981
           2       0.99      0.99      0.99      2039
           3       0.99      1.00      1.00      5000
           4       0.99      0.97      0.98      4871
           5       1.00      1.00      1.00      5000
           6       0.94      0.99      0.97       433

    accuracy                           0.98     23324
   macro avg       0.97      0.99      0.98     23324
weighted avg       0.98      0.98      0.98     23324

Classification report (train):
              precision    recall  f1-score   support

           0       1.00      0.98      0.99      8000
           1       0.93      1.00      0.96       974
           2       0.99      1.00      1.00      1854
           3       1.00      1.00      1.00      2000
           4       0.99      1.00      0.99      2000
           5       0.98      1.00      0.99      2000
           6       0.99      0.99      0.99      1718

    accuracy                           0.99     18546
   macro avg       0.98      1.00      0.99     18546
weighted avg       0.99      0.99      0.99     18546
