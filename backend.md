# Hiring Test for Backend Developers
- NOTE : You can use NodeJs/Python/ (Django/ExpressJS or any other framework of your choice as well)

## **Objective**
The objective of this test is to evaluate the candidate’s ability to:
- Design and implement Django models with **WYSIWYG editor support**.
- Store and manage **FAQs with multi-language translation**.
- Follow **PEP8 conventions** and best practices.
- Write a **clear and detailed README**.
- Use **proper Git commit messages**.

---

## **Task Requirements**

### **1. Model Design**
# models.py
from django.db import models
from ckeditor.fields import RichTextField

class FAQ(models.Model):
    question = models.TextField()
    answer = RichTextField()

    question_hi = models.TextField(null=True, blank=True)  # Hindi translation
    question_bn = models.TextField(null=True, blank=True)  # Bengali translation

    answer_hi = RichTextField(null=True, blank=True)
    answer_bn = RichTextField(null=True, blank=True)

    def get_translated_question(self, lang):
        if lang == 'hi':
            return self.question_hi or self.question
        elif lang == 'bn':
            return self.question_bn or self.question
        return self.question

    def get_translated_answer(self, lang):
        if lang == 'hi':
            return self.answer_hi or self.answer
        elif lang == 'bn':
            return self.answer_bn or self.answer
        return self.answer

    def __str__(self):
        return self.question
        
### **2. WYSIWYG Editor Integration**
- Use **django-ckeditor** to allow users to format answers properly.
- Ensure that the WYSIWYG editor supports **multilingual content**.
- pip install django-ckeditor
- # settings.py
INSTALLED_APPS = [
    ...
    'ckeditor',
]

CKEDITOR_CONFIGS = {
    'default': {
        'toolbar': 'full',
        'height': 300,
        'width': '100%',
    },
}

### **3. API Development**
- Create a ** REST API** for managing FAQs.
- Support **language selection** via `?lang=` query parameter.
- Ensure responses are **fast and efficient** using pre-translation.

- pip install djangorestframework
- # serializers.py
from rest_framework import serializers
from .models import FAQ

class FAQSerializer(serializers.ModelSerializer):
    class Meta:
        model = FAQ
        fields = ['id', 'question', 'answer', 'question_hi', 'question_bn', 'answer_hi', 'answer_bn']
# views.py
from rest_framework import generics
from .models import FAQ
from .serializers import FAQSerializer

class FAQListCreateView(generics.ListCreateAPIView):
    queryset = FAQ.objects.all()
    serializer_class = FAQSerializer

    def get_queryset(self):
        lang = self.request.query_params.get('lang', 'en')
        return FAQ.objects.all().annotate(
            question=models.F('question_' + lang),
            answer=models.F('answer_' + lang)
        )
# urls.py
from django.urls import path
from .views import FAQListCreateView

urlpatterns = [
    path('faqs/', FAQListCreateView.as_view(), name='faq-list-create'),
]

### **4. Caching Mechanism**
- Implement ** cache framework** to store translations.
- Use **Redis** for improved performance.
- pip install django-redis
- # settings.py
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

### **5. Multi-language Translation Support**
- Use **Google Translate API** or `googletrans`.
- Automate translations during object creation.
- Provide **fallback to English** if translation is unavailable.

- pip install googletrans==4.0.0-rc1
- # models.py
from googletrans import Translator

class FAQ(models.Model):
    # fields...
    def save(self, *args, **kwargs):
        translator = Translator()
        if not self.question_hi:
            self.question_hi = translator.translate(self.question, src='en', dest='hi').text
        if not self.question_bn:
            self.question_bn = translator.translate(self.question, src='en', dest='bn').text
        if not self.answer_hi:
            self.answer_hi = translator.translate(self.answer, src='en', dest='hi').text
        if not self.answer_bn:
            self.answer_bn = translator.translate(self.answer, src='en', dest='bn').text
        super().save(*args, **kwargs)

### **6.  Admin Panel**
- Register the **FAQ model** in the  Admin site or create one seperately.
- Enable a **user-friendly admin interface** for managing FAQs.

- # admin.py
from django.contrib import admin
from .models import FAQ

admin.site.register(FAQ)

### **7. Unit Tests & Code Quality**
- Write **unit tests** using `pytest` or `mocha`/`chai`.
- Ensure tests cover **model methods and API responses**.
- Follow **PEP8/ES6 guidelines** and use `flake8/JS tools` for linting.
- pip install pytest pytest-django
- # tests/test_models.py
import pytest
from .models import FAQ

@pytest.mark.django_db
def test_faq_creation():
    faq = FAQ.objects.create(question="What is Django?", answer="A web framework.")
    assert faq.question == "What is Django?"
    assert faq.answer == "A web framework."

### **8. Documentation**
- Write a **detailed README** with:
  - Installation steps
  - API usage examples
  - Contribution guidelines
- Ensure the **README is well-structured and easy to follow**.

- FAQ Management System
Introduction
This project is a FAQ management system built with Django. It supports multilingual FAQs with WYSIWYG editor support and provides a REST API for managing FAQs. The project also includes caching mechanisms to improve performance.

Features
Create and manage FAQs with language-specific translations.
Use a WYSIWYG editor for formatting FAQ answers.
Multilingual support for FAQs (English, Hindi, Bengali).
REST API for managing FAQs.
Caching using Redis for improved performance.
Automated translations using Google Translate API.
Installation
Prerequisites
Python 3.8+
Django 3.2+
Redis
Steps
Clone the repository:

sh
git clone https://github.com/your-username/faq-management-system.git
cd faq-management-system
Create a virtual environment and activate it:

sh
python -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
Install dependencies:

sh
pip install -r requirements.txt
Set up Redis:

Ensure Redis is installed and running. You can start Redis with the following command:
sh
redis-server
Apply migrations:

sh
python manage.py migrate
Create a superuser:

sh
python manage.py createsuperuser
Run the development server:

sh
python manage.py runserver
Access the admin panel:

Navigate to http://127.0.0.1:8000/admin and log in with the superuser credentials.
API Usage
Endpoints
Get all FAQs

HTTP
GET /faqs/
Get FAQs with language selection

HTTP
GET /faqs/?lang=<language_code>
Replace <language_code> with en, hi, or bn.
Create a new FAQ

HTTP
POST /faqs/
Request body example:
JSON
{
    "question": "What is Django?",
    "answer": "Django is a high-level Python web framework.",
    "question_hi": "डिज़ांगो क्या है?",
    "answer_hi": "डिज़ांगो एक उच्च-स्तरीय पाइथन वेब फ्रेमवर्क है।",
    "question_bn": "Django কি?",
    "answer_bn": "Django একটি উচ্চ-স্তরের পাইথন ওয়েব ফ্রেমওয়ার্ক।"
}
Contribution Guidelines
We welcome contributions to enhance this project. Please follow these guidelines:

Fork the repository:

Click the 'Fork' button at the top right corner of the repository page.
Clone your forked repository:

sh
git clone https://github.com/your-username/faq-management-system.git
cd faq-management-system
Create a new branch for your feature or bugfix:

sh
git checkout -b feature/your-feature-name
Make your changes and commit them:

Follow PEP8 conventions for Python code.
Write clear and concise commit messages.
sh
git add .
git commit -m "feat: Add new feature description"
Push your changes to your forked repository:

sh
git push origin feature/your-feature-name
Create a pull request:

Go to the original repository and click on 'New Pull Request'.
Provide a clear description of your changes and submit the pull request.
License
This project is licensed under the MIT License. See the LICENSE file for more details.

Contact
If you have any questions or need further assistance, feel free to open an issue or contact the project maintainer.



### **9. Git & Version Control**
- Use **Git for version control**.
- Follow **conventional commit messages**:
  - `feat: Add multilingual FAQ model`
  - `fix: Improve translation caching`
  - `docs: Update README with API examples`
- Ensure **atomic commits** with clear commit messages.

- git commit -m "feat: Add multilingual FAQ model"
git commit -m "fix: Improve translation caching"
git commit -m "docs: Update README with API examples"

### **10. Deployment & Docker Support (Bonus)**
- Provide a **Dockerfile** and **docker-compose.yml**.
- Deploy the application to **Heroku** or **AWS** (optional).

- # Dockerfile
FROM python:3.9

ENV PYTHONUNBUFFERED 1

WORKDIR /app

COPY requirements.txt /app/
RUN pip install -r requirements.txt

COPY . /app/

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

# docker-compose.yml
version: '3'

services:
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - redis

  redis:
    image: "redis:alpine"

---

## **Evaluation Criteria**
Candidates will be evaluated based on:
1. **Code Quality** (PEP8 compliance, readability, and modularity).
2. **Functionality** (API correctness, multilingual support, and caching efficiency).
3. **Documentation** (README completeness and clarity).
4. **Testing** (Coverage and effectiveness of unit tests).
5. **Git Best Practices** (Commit messages and branching strategy).

---

## **Submission Instructions**
- **Attempt the assignment** and complete your solution.
- **Open an issue** in our repository with the relevant tag (`backend` or `frontend`, depending on the test you're applying for).
- **Once done, tag @theakshaydhiman** in the issue, and we will review your code.
- **Include the link to your GitHub repository**, which must be **publicly accessible**.

---

## **Example API Usage**
```bash
# Fetch FAQs in English (default)
curl http://localhost:8000/api/faqs/

# Fetch FAQs in Hindi
curl http://localhost:8000/api/faqs/?lang=hi

# Fetch FAQs in Bengali
curl http://localhost:8000/api/faqs/?lang=bn
```

---

This test ensures the candidate demonstrates **full-stack Django development skills**, covering **models, APIs, caching, internationalization, and documentation**. 🚀

