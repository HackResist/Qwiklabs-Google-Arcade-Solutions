# 📌 Deploy a Static Site with Nginx on Google Cloud Run using Artifact Registry

- 🔗 **Lab Link**: [Click Here](https://www.cloudskillsboost.google/games/6312/labs/39882)  
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

# === Auto-fetch PROJECT_ID and REGION ===
PROJECT_ID=$(gcloud config get-value project 2>/dev/null)
REGION=$(gcloud config get-value run/region 2>/dev/null)

# === Check for required values ===
if [[ -z "$PROJECT_ID" ]]; then
  echo "❌ Project ID is not set. Run: gcloud config set project PROJECT_ID"
  exit 1
fi

if [[ -z "$REGION" ]]; then
  echo "❌ Region is not set. Run: gcloud config set run/region REGION"
  exit 1
fi

echo "✅ Using Project: $PROJECT_ID"
echo "✅ Using Region: $REGION"

# === Task 1: Set Up Your Environment ===
echo "🔧 Enabling required APIs..."
gcloud services enable run.googleapis.com artifactregistry.googleapis.com cloudbuild.googleapis.com

# === Task 2: Create Artifact Registry Repository ===
echo "📦 Creating Artifact Registry repository..."
gcloud artifacts repositories create traefik-repo \
  --repository-format=docker \
  --location="$REGION" \
  --description="Docker repository for static site images" || echo "ℹ️ Repository may already exist."

# === Task 3: Build and Push the Static Site Image ===

echo "📁 Setting up static site content..."
mkdir -p traefik-site/public
cd traefik-site

cat > public/index.html <<EOF
<html>
<head>
  <title>My Static Website</title>
</head>
<body>
  <p>Hello from my static website on Cloud Run!</p>
</body>
</html>
EOF

echo "⚙️ Creating traefik.yml..."
cat > traefik.yml <<EOF
entryPoints:
  web:
    address: ":8080"

providers:
  file:
    filename: /etc/traefik/dynamic.yml
    watch: true

log:
  level: INFO
EOF

echo "⚙️ Creating dynamic.yml..."
cat > dynamic.yml <<EOF
http:
  routers:
    static-files:
      rule: "PathPrefix(\`/\`)"
      entryPoints:
        - web
      service: static-service

  services:
    static-service:
      loadBalancer:
        servers:
          - url: "http://localhost:8000"
EOF

# === Task 4: Create the Dockerfile ===
echo "🐳 Creating Dockerfile..."
cat > Dockerfile <<EOF
FROM alpine:3.20

RUN apk add --no-cache traefik caddy

COPY traefik.yml /etc/traefik/traefik.yml
COPY dynamic.yml /etc/traefik/dynamic.yml
COPY public/ /public/

EXPOSE 8080

ENTRYPOINT [ "caddy" ]
CMD [ "file-server", "--listen", ":8000", "--root", "/public", "&", "traefik" ]
EOF

# === Authenticate Docker ===
echo "🔐 Authenticating Docker to Artifact Registry..."
gcloud auth configure-docker "$REGION"-docker.pkg.dev

# === Build and Push Docker Image ===
IMAGE_URI="$REGION-docker.pkg.dev/$PROJECT_ID/traefik-repo/traefik-static-site:latest"

echo "🔨 Building Docker image..."
docker build -t "$IMAGE_URI" .

echo "📤 Pushing Docker image to Artifact Registry..."
docker push "$IMAGE_URI"

# === Task 6: Deploy to Cloud Run ===
echo "🚀 Deploying to Cloud Run..."
gcloud run deploy traefik-static-site \
  --image "$IMAGE_URI" \
  --platform managed \
  --allow-unauthenticated \
  --port 8000

# === Task 7: Access your website ===
echo "🌐 Fetching deployed service URL..."
SERVICE_URL=$(gcloud run services describe traefik-static-site \
  --platform managed \
  --region "$REGION" \
  --format='value(status.url)')

echo "✅ Deployment successful!"
echo "🔗 Access your static website here: $SERVICE_URL"

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
  - ⌛ Task completion may take around 3 minutes.
