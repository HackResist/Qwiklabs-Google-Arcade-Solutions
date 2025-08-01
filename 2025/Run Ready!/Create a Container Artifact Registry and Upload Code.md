# 📌 Create a Container Artifact Registry and Upload Code

- 🔗 **Lab Link**: [Click Here](https://www.cloudskillsboost.google/games/6312/labs/39883)  
- 🗓️ **Completion Date**: `2025-08-02`  
## 🛠️ How to Use This Script 

1. 🛡️ Start the lab in a private browsing session.

2. 🔐 Open Google Cloud Console and authorize access.
   
3. 🌍 Set your `PROJECT_ID` and `REGION` using the command provided in Task 2.

4. 📝 Start a new file (e.g., .sh) and open it for editing.
    ```bash
    nano file_name.sh
    ```
5. 📝 Add the following code to your file.

```bash
#!/bin/bash

# === Auto-fetch current PROJECT_ID and REGION ===
PROJECT_ID=$(gcloud config get-value project 2>/dev/null)
REGION=$(gcloud config get-value compute/region 2>/dev/null)

# === Check and prompt for missing configs ===
if [[ -z "$PROJECT_ID" ]]; then
  echo "❌ Project ID not set. Run: gcloud config set project YOUR_PROJECT_ID"
  exit 1
fi

if [[ -z "$REGION" ]]; then
  echo "❌ Region not set. Run: gcloud config set compute/region REGION"
  exit 1
fi

echo "✅ Using Project: $PROJECT_ID"
echo "✅ Using Region: $REGION"

# === Task 1: Enable the Artifact Registry API ===
echo "🔧 Enabling Artifact Registry API..."
gcloud services enable artifactregistry.googleapis.com

# === Task 2: Create Artifact Registry Repository ===
REPO_NAME="my-docker-repo"
echo "📦 Creating Artifact Registry repository..."
gcloud artifacts repositories create "$REPO_NAME" \
  --repository-format=docker \
  --location="$REGION" \
  --description="Docker repository" || echo "ℹ️ Repository may already exist."

# === Task 3: Configure Docker Authentication ===
echo "🔐 Configuring Docker authentication..."
gcloud auth configure-docker "$REGION-docker.pkg.dev"

# === Task 4: Build and Tag a Sample Docker Image ===
echo "🐳 Creating sample Docker image..."
mkdir -p sample-app && cd sample-app

echo "FROM nginx:latest" > Dockerfile

IMAGE_NAME="nginx-image"
FULL_TAG="$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:latest"

echo "🔨 Building Docker image..."
docker build -t "$IMAGE_NAME" .

echo "🏷️ Tagging Docker image..."
docker tag "$IMAGE_NAME" "$FULL_TAG"

# === Task 5: Push the Docker Image ===
echo "📤 Pushing Docker image to Artifact Registry..."
docker push "$FULL_TAG"

echo "✅ Docker image pushed successfully!"
echo "📌 Image URI: $FULL_TAG"


```

6. 🔐 Grant execution permission.
```bash
chmod +x file_name.sh
   ```
7. ⚙️ Start the program.
```bash
./file_name.sh
```

## 💡 **Notes**:
  - ⚠️ Do not interrupt or cancel the script once it is running.
  - ⌛ Task completion may take around 8 minutes.
