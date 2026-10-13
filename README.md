# ShopWave — PBL-II Experiment 6

Responsive e-commerce demo storefront with product category filters, interactive cart, quantity controls and demo checkout. No real payment or customer data is collected.

## Architecture
Browser → EC2 public IP → Docker container → Nginx
Git repository → Jenkins → validate → build image → deploy container → HTTP smoke test

## Test with Docker on EC2
```bash
docker build -t shopwave-ecommerce .
docker run -d --name shopwave-web -p 8080:80 shopwave-ecommerce
```
Open `http://YOUR_EC2_PUBLIC_IP:8080` after allowing inbound TCP 8080 from your IP. Stop test container with `docker rm -f shopwave-web` before deploying the Jenkins pipeline on port 80.

## Jenkins pipeline
1. Push these files to a GitHub repository.
2. In Jenkins, create a Pipeline job and configure Pipeline script from SCM → Git → your repository URL → Script Path `Jenkinsfile`.
3. Ensure Jenkins has Git, Docker CLI and curl available, and the Jenkins service user has Docker permissions.
4. Allow inbound TCP port 80 in the EC2 security group (preferably only to the intended audience). Keep Jenkins port 8080 restricted to your own IP.
5. Run Build Now; on success open `http://YOUR_EC2_PUBLIC_IP/`.

If Docker permission fails and Jenkins is running on the same EC2 host, a lab setup may use:
```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo -u jenkins docker ps
```
Docker-group access is effectively root-level access, so only do this on a lab host you control.

## Practical screenshots
Capture: successful Jenkins stages, `docker ps` showing `shopwave-web`, website in browser, and cart/filter interaction.

## Limitations
Front-end demo only: no backend, database, accounts, persistent orders, inventory or payment integration. This pipeline replaces the existing container, so it demonstrates automated deployment but not zero-downtime blue/green deployment.
