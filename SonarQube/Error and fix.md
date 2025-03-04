**SonarQube Docker Troubleshooting Guide** 
1. Check If the Container is Running `docker ps`
2. If SonarQube is not listed, it’s not running. 
Start it: 
```docker start <container_id>``` 
3. Restart if needed: 
docker restart <container_id> 


---
**2. Verify Port Mapping**
 
docker inspect <container_id> | grep -i '"HostPort"'
 
Ensures that port 9000 is mapped correctly.
 
If incorrect, stop and remove the container:

 
docker stop <container_id>
docker rm <container_id>
 
Run SonarQube with correct port mapping:

 
docker run -d --name sonarqube -p 9000:9000 sonarqube

---

**3. Check If SonarQube is Listening on Port 9000**
 
ss -tulnp | grep 9000
 
Expected output:
 
LISTEN   0   128   0.0.0.0:9000   0.0.0.0:*
 
If it shows 127.0.0.1:9000, SonarQube is not accessible externally. Restart it with:
 
docker run -d -p 9000:9000 --name sonarqube sonarqube

---
 
**4. Check Firewall Rules (If Active)**
 
sudo ufw status
 
If inactive, no issue here.
 
If active, allow port 9000:
 
sudo ufw allow 9000/tcp
sudo ufw reload

---
 
**5. Check Azure NSG (Network Security Group) Rules**
 
1. Go to Azure Portal → Your VM → Networking.
 
2. Ensure an Inbound Rule exists allowing TCP traffic on port 9000. 
 
3. If missing, create a rule:
 
Source: Any (0.0.0.0/0)
 
Port: 9000
 
Protocol: TCP
 
Action: Allow

---
 
**6. Check Public IP & Access SonarQube**
 
Find your VM’s public IP:
 
curl ifconfig.me
 
Access from browser:
 
http://<public-ip>:9000
 
---
 
**7. Check SonarQube Logs for Errors**
 
docker logs -f sonarqube
 
Look for:
 
Elasticsearch errors → Increase memory.
 
Database errors → Ensure PostgreSQL is running and configured.

---
 
**8. Check Docker Networks (Optional)**
 
docker network ls
 
If using a custom network, ensure SonarQube is correctly attached.
 
