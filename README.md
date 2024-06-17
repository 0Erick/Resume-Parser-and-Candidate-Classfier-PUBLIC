# Resume Parser and Candidate classifier

This project, developed as college project in collaboration with some colleagues, is a resume parser and candidate classifier. While the code was primarily written by me, the project benefited from the contributions of my team. It was developed based on requirements from a Mexican company, so some information is in Spanish. The project employs Generative AI and Natural Language Processing (NLP) techniques and can be deployed both on the cloud and on-premises

## Resume Parser

The resume parser is responsible for extracting information from resumes. It supports PDF, DOCX, and DOC file formats. Depending on the file content, it decides between using a naive approach or Optical Character Recognition (OCR) to extract text. The extracted information is then passed to a Large Language Model (LLM) to format it in a structured manner. Below is an example of the extracted information:

``` markdown
Type: Work Experience
Management: Yes
Title: S&OP and Demand Planning Leader
Institution: REDACTED
Start Date: January, 2015
End Date: Present
Brief: Candidate led the S&OP process, coordinated the Sales and Operations Plan, and was responsible for Demand Planning. Candidate generated statistical and collaborative forecasts, reviewed production and supply capabilities, and collaborated with strategic suppliers and customers.


Type: Work Experience
Management: Yes
Title: Demand Planning Coordinator
Institution: REDACTED
Start Date: December, 2012
End Date: Present
Brief: Candidate was responsible for Demand Planning, generating sales forecasts and collaborative forecasts. Candidate coordinated the S&OP (Sales and Operations Planning) process, generated orders for suppliers and distribution centers, collaborated with suppliers, and monitored fill rate for over 1100 branches and 5 distribution centers.

Type: Work Experience
Management: No
Title: Planner
Institution: REDACTED
Start Date: July, 2010
End Date: December, 2012
Brief: Candidate was responsible for inventory analysis and control, national and Central American demand planning, internal production planning and control (raw material and finished goods inventory analysis), distribution supervision and material transfers to branches, and master material control (SAP).


Type: Education
Title: Bachelor's Degree in Industrial Engineering
Institution: REDACTED
Start Date: 2002
End Date: 2006
```

After extraction, text embeddings are generated using a pretrained sBERT model (Alibaba-NLP/gte-base-en-v1.5). Some information, such as work experience, is aggregated to create a single embedding. These embeddings are then stored in a MongoDB database.

## Candidate Classifier

The classifier categorizes candidates into three categories: Specialist, Manager, and Director. It was trained on private data from the company, processed by the resume parser. Fifteen features were selected for training the classifier, creating a dataset of shape (N, 15, 768), where N represents the number of observations, 15 the features, and 768 the dimensional space of the sBERT embeddings.  
The features are:

1. Years of work experience
2. Years in leadership positions
3. Average time in each job
4. Information indicating highest level of education (High School, College, Postgraduate, NA)
5. Highest educational qualification (Above college degree, not including name of college or university)
6. Undergraduate degree (Not including the name of the college or university)
7. Last job title (The name of the company is deliberately not included)
8. Summary of responsibilities of last job
9. Categorisation of whether the last job involved leadership
10. Title of penultimate job (Company name is deliberately not included).
11. Summary of responsibilities for the penultimate job
12. Categorisation as to whether the penultimate job involves leadership
13. Title of the penultimate job (Company name deliberately not included)
14. Summary of responsibilities of second to last job
15. Categorisation of whether the antepenultimate job involves leadership

The corresponding labels for the dataset are: 0 (Specialist), 1 (Manager), and 2 (Director). The classifier was trained using a Transformer model with hyperparameters tuned using KerasTuner. The classifier achieved an accuracy of 0.92 on the test set.

## Installation

To install this project, follow these steps:

### Step 1: Install LibreOffice and Tesseract-OCR

You need LibreOffice and Tesseract-OCR installed on your system. On Ubuntu, use the following commands:

```bash
sudo apt-get install libreoffice
sudo apt-get install tesseract-ocr
```

### Step 2 Set Up a Virtual Environment and Install Required Packages

1. Ensure you have Python 3.10 installed.
2. Set up a virtual environment and activate it:

```bash
python3.10 -m venv env
source env/bin/activate
```

3. Install the required packages using the following command:

```bash
pip install -r requirements.txt
```

### Step 3: Set Up MongoDB Databases

1. Tracker Database: We recommend using a MongoDB Atlas free cluster.
2. Embedded/Docs Database: Use a more capable MongoDB Atlas cluster for this database.

### Step 4: Configure Cloud-Based Deployment (Optional)

If you plan to use a cloud-based deployment, set up the following:

1. Anthropic API Key: Obtain from the Anthropic website.
2. MongoDB Atlas Cluster: Set up your clusters as mentioned in Step 3.
3. Azure Document Intelligence Resource: Set up the necessary resources on Azure.

Create a .env file with the following variables:

* CURRENT_ENV
* ANTHROPIC_API_KEY
* AZURE_OCR_ENDPOINT
* AZURE_OCR_KEY
* MONGO_TRACKER_URI
* MONGO_DOCS_URI

## Usage

### Step 1: Organize Resumes for Classification

1. Inside the resumes folder, create subfolders for each label you want to use (*Specialist*, *Manager*, *Director*).
Place the resumes you want to classify inside the respective subfolders.
2. Place the resumes you want to classify inside the respective subfolders.

### Step 2: Run the Parsing Process

Execute the **main.py** script to start the parsing process. The extracted information will be stored in the MongoDB database.

```bash
python main.py
```

### Step 3:  Convert Database to Dataset

Once the parsing process is complete, you can convert the database entries into a dataset using the **bsonToMatrices.ipynb** notebook.

### Step 4: Train the Classifier

Use the **Transformer.ipynb** notebook to train the classifier.
