STEP 1: Create and Test the Application
1. Create the project
Open VS Code → Terminal → New Terminal.

Create a folder and enter it:
mkdir my-project
cd my-project

2. Create these files in VS Code
- app.py
- test_app.py
- requirements.txt
- Dockerfile
- Jenkinsfile
Write the application and tests according to the question. Configure the dependencies in requirements.txt.
3. Install dependencies
python -m pip install -r requirements.txt

4. Run tests
python -m pytest -v

5. Check code quality
python -m flake8 app.py test_app.py --count --statistics

6. Start the application
python app.py

Open http://localhost:5000 if the application uses port 5000. Stop it with Ctrl+C.

Proceed when: The application works, tests pass, and code quality checks succeed.
STEP 2: Upload the Project to GitHub
1. Create a repository
Open GitHub → New repository → Enter the project name → Create repository.

Copy the repository URL.
2. Initialise Git
Run these commands inside your project folder:
git init
git add .
git commit -m "Initial application implementation"
git branch -M version-v1
git remote add origin YOUR_GITHUB_REPOSITORY_URL
git push -u origin version-v1

Replace YOUR_GITHUB_REPOSITORY_URL with your repository URL.
3. Create the v2 branch
git switch -c version-v2
git push -u origin version-v2

Implement the v2 features in VS Code, then:
git add .
git commit -m "Add version 2 features"
git push

4. Create the v3 branch
git switch version-v1
git switch -c version-v3
git push -u origin version-v3

Implement the v3 features, then:
git add .
git commit -m "Add version 3 features"
git push

Important: These commands create the branches. You must implement the appropriate features in each branch. If branches already exist, switch to them instead of creating them again.
STEP 3: Configure Jenkins
1. Start Jenkins
If Jenkins is installed as a WAR file, open a separate PowerShell terminal and run:
java -jar jenkins.war --httpPort=8081

Keep the terminal running.
2. Open Jenkins
Visit:

Jenkins Dashboard
3. Create the pipeline job
1. Click New Item.
2. Enter your project name.
3. Select Pipeline.
4. Click OK.
5. Find the Pipeline section.
6. Set Definition to Pipeline script from SCM.
7. Select Git.
8. Enter your GitHub repository URL.
9. Select the branch containing your Jenkinsfile.
10. Set Script Path to Jenkinsfile.
11. Save.
4. Configure Docker Hub credentials
In Jenkins:
1. Open Manage Jenkins → Credentials.
2. Add or update a Username with password credential.
3. Enter your Docker Hub username.
4. Enter your Docker Hub access token in the password field.
5. Set the credential ID to dockerhub-creds.
6. Save.
5. Save the Jenkinsfile to GitHub
From your project folder:
git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
git push

Ensure the Jenkinsfile is committed to the branch configured as the pipeline source.

Proceed when: Jenkins can load your pipeline and the Docker Hub credential is configured.
STEP 4: Run Tests Through Jenkins
1. Run the pipeline
1. Open your Jenkins job.
2. Click Build with Parameters, if configured.
3. Select the required application version.
4. Select its matching Git branch.
5. Click Build.
2. Inspect the build
1. Open the latest build.
2. Click Console Output.
3. Check the stages in order:
   - Checkout
   - Install Dependencies
   - Run Tests
   - Code Quality
   - Build Docker Image
   - Push to Docker Hub
   - Deploy to Kubernetes
3. If testing fails
Return to your project folder and run:
python -m pytest -v
python -m flake8 app.py test_app.py --count --statistics

Fix the reported errors, commit and push the changes, and rerun Jenkins.

Evidence to save: Successful test output, code-quality output, and Jenkins build results.
STEP 5: Build the Docker Image
1. Create the Dockerfile
Create the file named exactly:

Dockerfile

Configure it for your application, dependencies, and listening port.
2. Build the image
Run from the folder containing the Dockerfile:
docker build -t YOUR_DOCKERHUB_USERNAME/YOUR_IMAGE_NAME:v1 .

Replace the username and image name with your actual details.
3. Verify the image
docker images

4. Optionally test the container locally
docker run -d --name myapp-test -p 5000:5000 YOUR_DOCKERHUB_USERNAME/YOUR_IMAGE_NAME:v1

Check it:
docker ps

Open http://localhost:5000 if your application uses port 5000.

When finished:
docker stop myapp-test
docker rm myapp-test

Evidence to save: Successful Docker image-build output.

Note: If Jenkins has already built the image successfully, you do not need to build it manually again.
STEP 6: Push Images to Docker Hub
1. Create a Docker Hub repository
Open Docker Hub.

Create a repository with the required project name.
2. Verify Jenkins credentials
Confirm the Jenkins credential dockerhub-creds contains the correct Docker Hub username and a valid access token with write permission.
3. Run the pipeline for v1
Select:
- APP_VERSION: v1
- GIT_BRANCH: version-v1
Click Build.

Check that the Docker Hub push stage succeeds.
4. Repeat for v2 and v3
Run the pipeline again with:
Run	APP_VERSION	GIT_BRANCH
1	v1	version-v1
2	v2	version-v2
3	v3	version-v3


5. Verify Docker Hub
Open your repository's Tags tab.

Confirm that v1, v2, and v3 appear if three versions are required.

Evidence to save: A real screenshot showing the published image tags.
STEP 7: Deploy to Kubernetes
1. Start Docker Desktop
Open Docker Desktop and wait for Kubernetes to start.
2. Check the Kubernetes context
kubectl config current-context

If required, switch to Docker Desktop:
kubectl config use-context docker-desktop

3. Check the cluster
kubectl get nodes

The node should be ready.
4. Deploy the application
If Jenkins already deployed the application, skip manual deployment and go to Step 5.

Otherwise, deploy v1 using its published image:
kubectl create deployment library-v1 --image=YOUR_DOCKERHUB_USERNAME/YOUR_IMAGE_NAME:v1

Expose the deployment:
kubectl expose deployment library-v1 --type=NodePort --port=5000 --target-port=5000

Repeat for other versions if required, changing the deployment names and image tags.
5. Verify deployments
kubectl get deployments

6. Verify pods
kubectl get pods

The application pods should reach Running status.

If a pod fails:
kubectl describe pod POD_NAME
kubectl logs POD_NAME

Replace POD_NAME with the actual pod name.
7. Verify services
kubectl get services

Find the NodePort assigned to your application.
8. Open the application
Visit the following address, replacing the port with your actual NodePort:

http://localhost:YOUR_NODEPORT

Confirm the application loads.

Evidence to save:
- kubectl get deployments
- kubectl get pods
- kubectl get services
- Browser screenshot, if required.
