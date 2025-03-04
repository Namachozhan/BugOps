**Error:**

##[error]No agent found in pool ChozhanDevopsAgentpool which satisfies the following demand: java. All demands: maven, java, Agent.Version -gtVersion 3.218.0
Pool: ChozhanDevopsAgentpool

---

The error indicates that the **self-hosted agent** in your **ChozhanDevopsAgentpool** is missing Java, which is required for the build. The pipeline demands Java and Maven, but the agent doesn't satisfy those requirements.  

### **Steps to Fix:**  
#### **1️⃣ Check Installed Java on the Agent Machine**  
Log in to the self-hosted agent VM and run:  
```powershell
java -version
```
If Java is not installed or not found, proceed to install it.

#### **2️⃣ Install Java (If Missing)**  
For Windows:
- Download and install Java from [Adoptium](https://adoptium.net/) or use Chocolatey:
  ```powershell
  choco install openjdk --version=17
  ```
- Restart the agent service after installation:
  ```powershell
  Restart-Service vstsagent.*
  ```

For Linux:
- Install OpenJDK:
  ```bash
  sudo apt update && sudo apt install openjdk-17-jdk -y
  ```
- Restart the agent:
  ```bash
  sudo systemctl restart vstsagent
  ```

#### **3️⃣ Verify Java in Agent Capabilities**  
After installing Java, refresh the **Agent Capabilities** in **Azure DevOps**:
1. Go to **Azure DevOps → Organization Settings → Agent Pools**.
2. Select **ChozhanDevopsAgentpool** → Click on the **agent**.
3. Check the **Capabilities** section to confirm `java` is listed.

#### **4️⃣ Restart the Pipeline**  
After fixing Java, retry the pipeline run.

Let me know if the issue persists! 🚀
