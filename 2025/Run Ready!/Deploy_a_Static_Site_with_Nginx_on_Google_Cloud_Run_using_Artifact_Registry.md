# 📌 Deploy a Static Site with Nginx on Google Cloud Run using Artifact Registry

- 🔗 **Lab Link**: [Click Here](https://www.cloudskillsboost.google/games/6312/labs/39881)  
- 🗓️ **Completion Date**: `2025-08-02`  
## 🛠️ How to Use This Script 

1. 🛡️ Start the lab in a private browsing session.

2. 🔐 Open Google Cloud Console and authorize access.
   
3. 🌍 Set your `PROJECT_ID` and `REGION` using the command provided in Task 1.

4. 📝 Start a new file (e.g., .sh) and open it for editing.
    ```bash
    nano file_name.sh
    ```
5. 📝 Add the following code to your file.

```bash
#!/bin/bash

# === Task 1: Set up your Environment ===

# Get current project ID
PROJECT_ID=$(gcloud config get-value project 2>/dev/null)
if [[ -z "$PROJECT_ID" ]]; then
  echo "❌ Project ID not set. Run: gcloud config set project YOUR_PROJECT_ID"
  exit 1
fi

# Get current run region
REGION=$(gcloud config get-value run/region 2>/dev/null)
if [[ -z "$REGION" ]]; then
  echo "❌ Region not set. Run: gcloud config set run/region REGION"
  exit 1
fi

echo "✅ Using Project: $PROJECT_ID"
echo "✅ Using Region: $REGION"

echo "🔧 Enabling required services..."
gcloud services enable run.googleapis.com artifactregistry.googleapis.com

# === Task 2: Create a Static Website ===

echo "📄 Creating index.html..."
cat > index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>My Static Website</title>
</head>
<body>
    <div>Welcome to My Static Website!</div>
    <p>This website is served from Google Cloud Run using Nginx and Artifact Registry.</p>
</body>
</html>
EOF

# === Task 3: Create an Nginx Configuration ===

echo "⚙️ Creating nginx.conf..."
cat > nginx.conf <<EOF
events {}
http {
    server {
        listen 8080;
        root /usr/share/nginx/html;
        index index.html index.htm;

        location / {
            try_files \$uri \$uri/ =404;
        }
    }
}
EOF

# === Task 4: Create Dockerfile ===

echo "🐳 Creating Dockerfile..."
cat > Dockerfile <<EOF
FROM nginx:latest

COPY index.html /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
EOF

# === Task 5: Build and Push Docker Image to Artifact Registry ===

REPO_NAME="nginx-static-site"
IMAGE_NAME="$REGION-docker.pkg.dev/$PROJECT_ID/$REPO_NAME/nginx-static-site"

echo "📦 Creating Artifact Registry repository..."
gcloud artifacts repositories create $REPO_NAME \
    --repository-format=docker \
    --location="$REGION" \
    --description="Docker repository for static website" || echo "ℹ️ Repository may already exist."

echo "🔐 Configuring Docker authentication..."
gcloud auth configure-docker "$REGION-docker.pkg.dev"

echo "🔨 Building Docker image..."
docker build -t nginx-static-site .

echo "🏷️ Tagging Docker image..."
docker tag nginx-static-site "$IMAGE_NAME"

echo "📤 Pushing Docker image to Artifact Registry..."
docker push "$IMAGE_NAME"

# === Task 6: Deploy to Cloud Run ===

echo "🚀 Deploying to Cloud Run..."
gcloud run deploy nginx-static-site \
    --image "$IMAGE_NAME" \
    --platform managed \
    --region "$REGION" \
    --allow-unauthenticated

echo "🌐 Retrieving service URL..."
SERVICE_URL=$(gcloud run services describe nginx-static-site \
    --platform managed \
    --region "$REGION" \
    --format='value(status.url)')

# === Task 7: Verify Deployment ===

echo "✅ Deployment complete!"
echo "🔗 Visit your static website at: $SERVICE_URL"

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
  - ⌛ Task completion may take around 5 minutes.
