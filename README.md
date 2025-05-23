# Online-job-application-optimizer-Python script#

import spaCy
import PyPDF2
import docx
from collections import Counter
import re

# load spaCy NLP model#
nlp = spacy.load("en_core_web_sm")

# function to extract text from pdf resume #

def extract_resume_text(pdf_path):
    with open (pdf_path, "rb") as file:
         reader = PyPDF2.PdfReader(file)
         text = ""
         for page in reader.pages:
             text += page.extract_text()
         return text 

# function to extract text from Word resume#

def extract_resume_docx(docx_path):
    doc = docx.Document(docx_path)
    text = "\n".join([para.text for para in doc.paragraphs])
    return text 

# function to extract from job description #
def extract_keywords(job_description):
    doc = nlp(job description)
    keywords = []

# extract nouns, proper nouns, and verbs (common in job descriptions)
for token in doc:
    if token.pos_ in ["NOUN", "PROPN", "VERB"] and not token.is_stop:
    keywords.append(chunk.text.lower())




         
