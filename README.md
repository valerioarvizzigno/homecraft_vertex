# Homecraft Retail demo with Elastic ESRE and Google's GenAI


## Configuration steps

1. Setup your Elastic cluster with ML nodes

2. Install python on your local machine. If using Homebew on macOS simply use

```bash
brew install python@3.11  
```

3. (Optional) For better python environment management use Virtual Envs. Create a folder for your project in your favourite location, enter it and create a venv name "homecraft". You will then install all the libraries requirements only inside this venv instead of globally

```bash
python -m venv homecraftenv
```

4. (Optional) If step 3 is followed, activate your virtual env. Check here https://docs.python.org/3/tutorial/venv.html commands depending on your OS. For Unix or macOS use

```bash
source homecraftenv/bin/activate
```

5. Install requirements needed to run the app from the requirements.txt file

```bash
pip install -r requirements.txt 
```

6. Install gcloud SDK. It is needed to connect to VertexAI APIs. (https://cloud.google.com/sdk/docs/install-sdk)
   Follow the instructions at the link depending on your OS. If using Homebrew on macOS you can simply install it with

 ```bash
brew install --cask google-cloud-sdk
```  

7. Init gcloud and follow the CLI instructions. You have to specify the working Google Cloud project

 ```bash
gcloud init
```  

8. Authenticate the VertexAI SDK (it has been installed with requirements.txt). More info here https://googleapis.dev/python/google-api-core/latest/auth.html

 ```bash
gcloud auth application-default login
```  

9. Load the all-distillroberta-v1 (https://huggingface.co/sentence-transformers/all-distilroberta-v1) ML model in you Elastic cluster via Eland client and start it. To run Eland client you need docker installed. An easy way to accomplish this step without python/docker installation is via Google's Cloud Shell.

 ```bash
docker run -it --rm elastic/eland     eland_import_hub_model       --url https://<elastic_user>:<elastic_password>@<your_elastic_endpoint>:9243/ --hub-model-id sentence-transformers/all-distilroberta-v1 --start
 ```

10. Index  general data from a retailer website (I used https://www.ikea.com/gb/en/) with Elastic Enterprise Search's webcrawler and give the index the "search-homecraft-ikea" (for immediate compatibility with this repo code, otherwise change the reference in all homecraft_*.py files). Set an ingest pipeline, named "ml-inference-title-vector", working directly at crawling time, to enrich crawled documents with dense vectors. Use the previously loaded ML model for inference on the "title" field as source, and set "title-vector" as target field for dense vectors.

11. Before launching the crawler, set mappings for the title-vector field on the index

```bash
POST search-homecraft-ikea/_mapping
{
  "properties": {
    "title-vector": {
      "type": "dense_vector",
      "dims": 768,
      "index": true,
      "similarity": "dot_product"
    }
  }
}
```

11. Start crawling

12. Index the Home Depot products dataset (https://www.kaggle.com/datasets/thedevastator/the-home-depot-products-dataset) into elastic.

13. Create a new empty index that will host the dense vectors called "home-depot-product-catalog-vector" (for immediate compatibility with this repo code, otherwise change the reference in all homecraft_*.py files) and specify mappings

```bash
PUT /home-depot-product-catalog-vector 

POST home-depot-product-catalog-vector/_mapping
{
  "properties": {
    "title-vector": {
      "type": "dense_vector",
      "dims": 768,
      "index": true,
      "similarity": "dot_product"
    }
  }
}
```

14. Re-index the product dataset through the same ingest pipeline previously created for the web-crawler. The new index will now have vectors embedded in documents in the title-vector field

```bash
POST _reindex
{
  "source": {
    "index": "home-depot-product-catalog"
  },
  "dest": {
    "index": "home-depot-product-catalog-vector",
    "pipeline": "ml-inference-title-vector"
  }
}
```


15. Set up the environment variables cloud_id, cloud_pass and cloud_user with your deployment credentials and cloud id.

16. Fine-tune text-bison@001 via VertexAI fine-tuning feature, using the fine-tuning/fine_tuning_dataset.jsonl file. This will instruct the model in advertizing partner network when specific questions are asked. For more information about fine-tuning look at https://cloud.google.com/vertex-ai/docs/generative-ai/models/tune-models#generative-ai-tune-model-python

17. Run streamlit app

 ```bash
streamlit run homecraft_home.py
```  


---USE THE HOME PAGE FOR BASE DEMO---

Try a query like: 
"list the 3 top paint primers in the product catalog, specify also product price and product key features. Then explain in bullet points how to use a paint primer"
You can also try asking for related urls and availability

or: could you please list the available stores in UK
or: could you please list the available stores in UK? Please also add a reference to the webpage I can find this list

or: Which are the ways to contact customer support in the UK? What is the webpage url for customer support?

---FOR A DEMO OF FINE-TUNED MODEL USE "HOMECRAFT FINETUNED" WEBPAGE---

Try "Anyone available at Homecraft to assist with painting my house?".
Asking this question in the fine-tuned page should suggest to go with Homecraft's network of professionals

Asking the same to the base model will likely provide a generic or "unable to help" answer.


!!!WORK IN PROGRESS!!! This readme is a quick DRAFT. Full guidelines will be added soon.
