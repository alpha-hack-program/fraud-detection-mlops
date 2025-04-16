# MLOps Approach for Fraud Detection Example

<table>
    <tr>
        <td><b>Title:</b></td>
        <td>Fraud Detection MLOps</td>
    </tr>
    <tr>
        <td><b>Goal:</b></td>
        <td>Implement MLOps using the Fraud Detection example.</td>
    </tr>
    <tr>
        <td><b>Output:</b></td>
        <td>Learn how to use Data Science Pipelines and Inference.</td>
    </tr>
    <tr>
        <td><b>Timing:</b></td>
        <td>1h</td>
    </tr>
    <tr>
        <td><b>Notes:</b></td>
        <td>We'll start with RHOAI installed.</td>
    </tr>
</table>

---

TODO fix use crt

## Prerequisites:
As part of the demo, you will have to do some changes and commits. So **it is important that you fork the repository and clone it in your local**.

```
git clone https://github.com/alpha-hack-program/fraud-detection-mlops.git
```

Go to the folder where you have cloned your forked repository and create a new branch `testing`
```
cd fraud-detection-mlops
git checkout -b testing
git push origin testing
```

## Steps to Run the Demo


- **Set Up the Data Science Environment**

Apply the following manifest to configure the data science environment:
```bash
oc apply -f gitops/onboard-datascience/app-onboard-datascience.yaml
```

- **Prepare the Training Data (v1.csv)**

Create a file named 'v1.csv' containing the data required for model training in data folder. you can rename 'card_transdata_sort.csv'. Ensure the file adheres to the expected format.

- **Commit the Training Data**

Commit and push the v1.csv file to the Git repository to trigger the model training pipeline:
```bash
git add data/v1.csv
git commit -m "Add data v1.csv for model training"
git push origin main
```

- **Deploy the Inference Service**

Apply the following manifest to deploy the inference service:
```bash
oc apply -f gitops/inference/app-inference.yaml
```

- **Test the Inference Service**

Once the model is trained and deployed, test the inference service by sending test data to the service endpoint.
```bash
host=<YOUR_HOST>
url="https://fraudinference-fraud.apps.$host/v2/models/fraudinference/versions/1/infer" 
data='{
        "id" : "42",
        "inputs": [
                    {
                        "name": "dense_input",
                        "shape": [1, 5],
                        "datatype": "FP32",
                        "data": [0.3111400080477545, 1.9459399775518593, 1.0, 0.0, 0.0]
                    }
                ]
        }'
curl -k -X POST "$url" -H "Content-Type: application/json" -d "$data"
```

- **Update the Model with New Data**

To train a new version of the model. Create a file named 'v2.csv' containing the data required for model training in data folder. you can rename 'card_transdata_full.csv'. Ensure the file adheres to the expected format.

Commit and push the v2.csv file to the Git repository to trigger the model training pipeline again:
```bash
git add data/v2.csv
git commit -m "Add data v2.csv for model training"
git push origin main
```
This will trigger the pipeline to train and deploy the updated model.

- **Inference model v2**

Update the version field in the gitops/onboard-datascience/values-datasciene.yaml file to 2 to use the new model version:
```yaml
# filepath: gitops/inference/values-inference.yaml
inference:
  dataconnections: dataconnection-model
  version: 2
multimodel: false
```

Commit and push the file to the Git repository to inference model v2:
```bash
git add gitops/inference/values-inference.yaml
git commit -m "Inference model v2"
git push origin main
```

- **Test the Inference Service with the Updated Model**

After the updated model is deployed, test the inference service again to validate the new model's performance.
```bash
host=<YOUR_HOST>
url="https://fraudinference-fraud.apps.$host/v2/models/fraudinference/versions/1/infer" 
data='{
        "id" : "42",
        "inputs": [
                    {
                        "name": "dense_input",
                        "shape": [1, 5],
                        "datatype": "FP32",
                        "data": [0.3111400080477545, 1.9459399775518593, 1.0, 0.0, 0.0]
                    }
                ]
        }'
curl -k -X POST "$url" -H "Content-Type: application/json" -d "$data"
```

## Delete environment

```bash
git checkout main
git branch -D testing
git push origin --delete testing
```