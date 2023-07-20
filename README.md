# Homecraft Retail demo with Elastic ESRE and Google's GenAI


## Configuration steps

1. Setup your Elastic cluster with ML nodes
2. Install python on your local machine
3. Install requirements needed to run the app from the requirements.txt file

```bash
pip install -r requirements.txt 
```

4. Install gcloud SDK. It is needed to connect to VertexAI APIs. (https://cloud.google.com/sdk/docs/install-sdk)
   Follow the instructions at the link depending on your OS. If using Homebrew on macOS you can also install it with

 ```bash
brew install --cask google-cloud-sdk
```  

5. Init gcloud and follow the CLI instructions. You have to specify the working Google Cloud project

 ```bash
gcloud init
```  

6. Authenticate the VertexAI SDK (it has been installed with requirements.txt)

 ```bash
gcloud auth application-default login
```  

7. Load the all-distillroberta-v1 (https://huggingface.co/sentence-transformers/all-distilroberta-v1) ML model in you Elastic cluster via Eland client and start it
8. Index the Home Depot dataset (https://www.kaggle.com/datasets/thedevastator/the-home-depot-products-dataset) into elastic through an ingest pipeline that leverages the previosly loaded model on the title field
9. Set up the environment variables cloud_id, cloud_pass and cloud_user with your deployment credentials and cloud id.
10. Run streamlit app

 ```bash
streamlit run homecraft_home.py
```  
---USE THE SECOND PAGE ON STREAMLIT - HOMECRAFT ASSISTANT---

Try a query like: 
"list the 3 top paint primers in the product catalog, specify also product price and product key features. Then explain in bullet points how to use a paint primer"
You can also try asking for related urls and availability

or: could you please list the available stores in UK
or: could you please list the available stores in UK? Please also add a reference to the webpage I can find this list

or: Which are the ways to contact customer support in the UK? What is the webpage url for customer support?

---FOR A DEMO OF FINE-TUNED MODEL USE "HOMECRAFT FINETUNED" WEBPAGE---
TBD


!!!WORK IN PROGRESS!!! This readme is a quick DRAFT. Full guidelines will be added soon.
