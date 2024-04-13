# Deploy Python App on Google Cloud Run with GitHub Auto-deploy

This README provides detailed instructions for setting up a Python application, containerizing it, and deploying it on Google Cloud Run with automatic deployment from GitHub upon code pushes.

## Setup Instructions

1. **Create a Python Flask Application**

First, create a `app.py` file with a basic Flask application:

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World from Cloud Run!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

Create a `requirements.txt` file for the Python dependencies:

```plaintext
# requirements.txt
Flask==2.0.1
gunicorn==20.1.0
```

2. **Containerization with Docker**

Create a `Dockerfile` to containerize your Flask application:

```Dockerfile
# Dockerfile
FROM python:3.8-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]
```

3. **Set Up Google Cloud Build for Continuous Deployment**

- Enable the Google Cloud Build and Google Container Registry APIs.
- Create a Cloud Build trigger that connects to your GitHub repository.

Example `cloudbuild.yaml` to build and deploy to Cloud Run:

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA']
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'my-app'
      - '--image'
      - 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA'
      - '--region'
      - 'us-central1'
      - '--platform'
      - 'managed'
      - '--allow-unauthenticated'
timeout: '1600s'
```

4. **Configure GitHub Repository**

- Connect your GitHub repository to Google Cloud Build.
- Add the `cloudbuild.yaml` to your repository.
- Set up the necessary permissions for Cloud Build to access your repository.

5. **Deploy Your Application**

Push your code to GitHub. Cloud Build will automatically trigger, build your Docker image, push it to Google Container Registry, and deploy your application to Google Cloud Run.

## Conclusion

This guide provides a detailed workflow for deploying a Python application on Google Cloud Run, with continuous deployment setup via GitHub and Google Cloud Build. Customize the Flask application as needed.
