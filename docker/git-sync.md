# Use RHEL 9 as the base image
FROM registry.access.redhat.com/ubi9/ubi:latest

# Install git
RUN dnf install -y git && \
    dnf clean all

# Add a script to perform the git sync (as in the previous example)
ADD git-sync.sh /usr/local/bin/git-sync.sh
RUN chmod +x /usr/local/bin/git-sync.sh

CMD ["/usr/local/bin/git-sync.sh"]


#!/bin/sh

# Configuration
REPO_URL="https://your-git-repo-url.git"
CLONE_DIR="/app/repo"
BRANCH="main"
INTERVAL=60 # Interval in seconds

# Clone the repository if it doesn't exist
if [ ! -d "$CLONE_DIR" ]; then
  git clone --branch $BRANCH $REPO_URL $CLONE_DIR
fi

cd $CLONE_DIR

# Periodically pull the latest changes
while true; do
  git fetch origin $BRANCH
  git reset --hard origin/$BRANCH
  sleep $INTERVAL
done

apiVersion: v1
kind: Pod
metadata:
  name: git-sync-pod
spec:
  containers:
  - name: main-app
    image: your-app-image:latest
    volumeMounts:
    - name: git-repo
      mountPath: /app/repo # Mount the shared volume
  - name: git-sync
    image: your-docker-repo/git-sync:latest
    env:
    - name: REPO_URL
      value: "https://your-git-repo-url.git"
    - name: BRANCH
      value: "main"
    - name: INTERVAL
      value: "60"
    volumeMounts:
    - name: git-repo
      mountPath: /app/repo # Mount the shared volume
  volumes:
  - name: git-repo
    emptyDir: {}
