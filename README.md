from fastapi import FastAPI, File, UploadFile, Form
import uvicorn
import shutil
import os
import zipfile
import pandas as pd

app = FastAPI()

def find_answer(question: str):
    # Placeholder function - Implement logic to fetch the correct answer
    return "Answer not found"

@app.post("/api/")
async def answer_question(question: str = Form(...), file: UploadFile = File(None)):
    extracted_answer = find_answer(question)
    
    if file:
        temp_dir = "temp_uploads"
        os.makedirs(temp_dir, exist_ok=True)
        file_path = os.path.join(temp_dir, file.filename)
        
        try:
            with open(file_path, "wb") as buffer:
                shutil.copyfileobj(file.file, buffer)

            if file.filename.endswith(".zip"):
                with zipfile.ZipFile(file_path, 'r') as zip_ref:
                    zip_ref.extractall(temp_dir)
                    extracted_files = zip_ref.namelist()
                    for extracted_file in extracted_files:
                        csv_path = os.path.join(temp_dir, extracted_file)
                        if extracted_file.endswith(".csv") and os.path.exists(csv_path):
                            df = pd.read_csv(csv_path)
                            if "answer" in df.columns and not df.empty:
                                return {"answer": str(df["answer"].iloc[0])}
        finally:
            shutil.rmtree(temp_dir, ignore_errors=True)
    
    return {"answer": extracted_answer}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

