import spacy
import PyPDF2
import docx
from collections import Counter
import re

# Load spaCy NLP model
nlp = spacy.load("en_core_web_sm")

# Function to extract text from PDF resume
def extract_resume_text(pdf_path):
    with open(pdf_path, "rb") as file:
        reader = PyPDF2.PdfReader(file)
        text = ""
        for page in reader.pages:
            text += page.extract_text() + "\n"
    return text

# Function to extract text from Word resume
def extract_resume_docx(docx_path):
    doc = docx.Document(docx_path)
    text = "\n".join([para.text for para in doc.paragraphs])
    return text

# Function to extract keywords from job description
def extract_keywords(job_description):
    doc = nlp(job_description)
    keywords = []
    
    # Extract nouns, proper nouns, and verbs (common in job descriptions)
    for token in doc:
        if token.pos_ in ["NOUN", "PROPN", "VERB"] and not token.is_stop:
            keywords.append(token.text.lower())
    
    # Extract phrases (e.g., "project management")
    for chunk in doc.noun_chunks:
        if not all(word.is_stop for word in chunk):
            keywords.append(chunk.text.lower())
    
    # Count frequency for prioritization
    keyword_counts = Counter(keywords)
    return keyword_counts.most_common(10)  # Top 10 keywords/phrases

# Function to compare resume with job description
def compare_resume_to_job(resume_text, job_keywords):
    resume_text = resume_text.lower()
    missing_keywords = []
    
    for keyword, _ in job_keywords:
        if not re.search(r'\b' + re.escape(keyword) + r'\b', resume_text):
            missing_keywords.append(keyword)
    
    return missing_keywords

# Function to suggest resume improvements
def suggest_resume_changes(missing_keywords):
    suggestions = []
    for keyword in missing_keywords:
        suggestions.append(f"Consider adding '{keyword}' to your resume in a relevant section (e.g., skills, experience).")
    return suggestions

# Example usage
def main():
    # Sample job description (replace with actual job description text)
    job_description = """
    We are seeking a Data Analyst with experience in Python, SQL, and data visualization. 
    The ideal candidate has strong analytical skills, proficiency in Tableau, and knowledge 
    of machine learning. Responsibilities include data cleaning, statistical analysis, 
    and project management.
    """
    
    # Path to your resume (PDF or DOCX)
    resume_path = "resume.pdf"  # Replace with your resume file path
    
    # Extract resume text based on file type
    if resume_path.endswith(".pdf"):
        resume_text = extract_resume_text(resume_path)
    elif resume_path.endswith(".docx"):
        resume_text = extract_resume_docx(resume_path)
    else:
        print("Unsupported resume file format. Use PDF or DOCX.")
        return
    
    # Extract keywords from job description
    job_keywords = extract_keywords(job_description)
    print("Top Job Description Keywords:", job_keywords)
    
    # Compare resume to job description
    missing_keywords = compare_resume_to_job(resume_text, job_keywords)
    
    # Provide suggestions
    if missing_keywords:
        print("\nMissing Keywords in Your Resume:")
        for suggestion in suggest_resume_changes(missing_keywords):
            print(suggestion)
    else:
        print("\nYour resume contains all key keywords from the job description!")

if __name__ == "__main__":
    main()