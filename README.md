# AI-powered-career-guidence
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import declarative_base, sessionmaker

engine = create_engine("sqlite:///users.db")

Base = declarative_base()

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    education = Column(String)
    skills = Column(String)
    interests = Column(String)

Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)

from openai import OpenAI
import os

client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY")
)

def get_career_recommendations(
    education,
    skills,
    interests
):
    prompt = f"""
    Education: {education}
    Skills: {skills}
    Interests: {interests}

    Suggest:
    - Top 5 career paths
    - Required skills
    - Learning roadmap
    - Salary insights
    """

    response = client.chat.completions.create(
        model="gpt-4.1",
        messages=[
            {
                "role": "user",
                "content": prompt
            }
        ]
    )

    return response.choices[0].message.content
    from reportlab.pdfgen import canvas

def create_resume(data, filename):
    pdf = canvas.Canvas(filename)

    pdf.drawString(100, 800, data["name"])
    pdf.drawString(100, 770, "Summary:")
    pdf.drawString(100, 750, data["summary"])

    pdf.drawString(100, 700, "Skills:")
    pdf.drawString(100, 680, data["skills"])

    pdf.drawString(100, 630, "Experience:")
    pdf.drawString(100, 610, data["experience"])

    pdf.save()
    from flask import Flask, request, jsonify
from database import Session, User
from ai_engine import get_career_recommendations
from resume_builder import create_resume

app = Flask(__name__)

@app.route("/")
def home():
    return "AI Career Guidance Platform"

@app.route("/register", methods=["POST"])
def register():

    data = request.json

    session = Session()

    user = User(
        name=data["name"],
        education=data["education"],
        skills=data["skills"],
        interests=data["interests"]
    )

    session.add(user)
    session.commit()

    return jsonify({
        "message": "User registered"
    })

@app.route("/recommend", methods=["POST"])
def recommend():

    data = request.json

    recommendations = get_career_recommendations(
        data["education"],
        data["skills"],
        data["interests"]
    )

    return jsonify({
        "recommendations": recommendations
    })

@app.route("/resume", methods=["POST"])
def resume():

    data = request.json

    filename = f"{data['name']}_resume.pdf"

    create_resume(data, filename)

    return jsonify({
        "message": "Resume generated",
        "file": filename
    })

if __name__ == "__main__":
    app.run(debug=True)
    import requests

data = {
    "name": "Rahul",
    "education": "B.Tech Computer Science",
    "skills": "Python, SQL, ML",
    "interests": "AI, Data Science"
}

response = requests.post(
    "http://127.0.0.1:5000/register",
    json=data
)
import requests

response = requests.post(
    "http://127.0.0.1:5000/recommend",
    json={
        "education": "B.Tech",
        "skills": "Python, Machine Learning",
        "interests": "Artificial Intelligence"
    }
)

print(response.json())

print(response.json())
