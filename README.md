# CC
# Cloud Computing Practical Record — Expanded Commands and Complete Programs

> **Use this as an execution companion to the theory/write-up notes.** It includes commands, complete sample files, run instructions, and expected outputs.
>
> **Important:** Replace sample usernames, IP addresses, passwords, project IDs, paths, and ports with the values from your lab. Do not paste example passwords into a real system. Some services and older lab platforms may no longer be available; use the version/environment supplied by your institution. Do not write that a deployment succeeded unless you actually verified it.

---

# Experiment 1 — Study of Cloud Computing and Its Architecture

## Practical procedure / what to present

1. Open a browser and visit a cloud-service provider or use the examples given by your teacher.
2. Identify one example each of IaaS, PaaS, and SaaS.
3. Identify public, private, hybrid, and community deployment models.
4. Draw the following architecture in your notebook:

```text
             CLIENT / FRONT END
       +-----------------------------+
       | Browser | Mobile | Desktop  |
       +-----------------------------+
                    |
                 Internet
                    |
       +-----------------------------+
       |       CLOUD BACK END        |
       | Applications / Services     |
       | Compute (VMs)                |
       | Storage / Databases          |
       | Network / Security           |
       | Management / Monitoring      |
       +-----------------------------+
```

5. Explain the five characteristics: on-demand self-service, broad network access, resource pooling, rapid elasticity, and measured service.

## Observation table

| Service model | Meaning | Example |
|---|---|---|
| IaaS | Virtual machines, storage, networking | Amazon EC2 |
| PaaS | Platform to build and deploy apps | Google App Engine |
| SaaS | Ready-to-use software | Gmail |

**Result:** Cloud computing architecture, service models, deployment models, and characteristics were studied.

---

# Experiment 2 — Create a Virtual Machine Using Oracle VirtualBox

## GUI procedure

1. Install and open **Oracle VM VirtualBox**.
2. Click **New**.
3. Enter a name, for example `Ubuntu-Lab`.
4. Select the operating-system type and version.
5. Select the RAM allocation. Use a value supported by your computer and lab instructions.
6. Create a virtual hard disk (VDI), dynamically allocated, and choose the required size.
7. Select the new VM and open **Settings → Storage**.
8. Select the empty optical drive, click the disk icon, and choose the operating-system ISO file.
9. Open **Settings → Network** and use the network mode specified by your teacher (NAT is commonly used for basic internet access).
10. Click **Start** and follow the operating-system installer.
11. Restart the VM after installation and verify that it boots.

## Optional command-line verification (host terminal)

List VMs:

```bash
VBoxManage list vms
```

Show running VMs:

```bash
VBoxManage list runningvms
```

Show details (replace the name):

```bash
VBoxManage showvminfo "Ubuntu-Lab"
```

**Expected observation:** The VM appears in VirtualBox and boots into its installed operating system.

**Result:** A virtual machine was created and configured using VirtualBox.

---

# Experiment 3 — Introduction to OpenStack / DevStack

> DevStack is a development/test installation and normally needs a dedicated supported Linux VM with sufficient RAM, CPU, and disk. Do not run it on a production machine. The exact branch and configuration must match the lab.

## A. Prepare a dedicated Ubuntu lab VM

Update package lists and install Git:

```bash
sudo apt update
sudo apt install -y git
```

Create the DevStack user (run only if your lab image does not already provide it):

```bash
sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/stack
sudo chmod 0440 /etc/sudoers.d/stack
```

Switch to the stack user:

```bash
sudo su - stack
```

Clone DevStack:

```bash
git clone https://opendev.org/openstack/devstack
cd devstack
```

## B. Create the configuration file

Create `local.conf` inside the `devstack` directory:

```bash
nano local.conf
```

Example minimal lab configuration (use the release/branch specified by your instructor):

```ini
[[local|localrc]]
ADMIN_PASSWORD=LabAdminPassword
DATABASE_PASSWORD=LabDatabasePassword
RABBIT_PASSWORD=LabRabbitPassword
SERVICE_PASSWORD=LabServicePassword
HOST_IP=127.0.0.1
```

Save and exit. These are example lab-only values; do not reuse them on an exposed system. Some DevStack releases require additional settings or a non-loopback `HOST_IP`.

## C. Start installation

```bash
./stack.sh
```

Wait for the script to finish. Installation can take a while.

## D. Verify OpenStack

Load the generated credentials:

```bash
source openrc admin admin
```

Check services/resources:

```bash
openstack service list
openstack endpoint list
openstack compute service list
openstack network list
openstack image list
openstack flavor list
```

If the lab provides Horizon, open the dashboard URL printed by DevStack or supplied by the instructor.

**Expected observation:** OpenStack service, endpoint, network, image, and flavor information is displayed. Exact output varies by installation.

**Result:** OpenStack services and the DevStack development environment were explored.

---

# Experiment 4 — Installation and Configuration of ownCloud

> ownCloud installation requirements vary by release. If your lab supplies a preinstalled ownCloud server, use the browser setup steps in section B. The Docker example below is a self-contained demonstration where Docker Compose is available.

## A. Start a lab ownCloud stack with Docker Compose

Create a folder:

```bash
mkdir owncloud-lab
cd owncloud-lab
```

Create a file named `compose.yaml`:

```yaml
services:
  db:
    image: mariadb:10.11
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: change_this_root_password
      MYSQL_DATABASE: owncloud
      MYSQL_USER: owncloud
      MYSQL_PASSWORD: change_this_user_password
    volumes:
      - db_data:/var/lib/mysql

  owncloud:
    image: owncloud/server:latest
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      OWNCLOUD_DOMAIN: localhost:8080
      OWNCLOUD_DB_TYPE: mysql
      OWNCLOUD_DB_NAME: owncloud
      OWNCLOUD_DB_USERNAME: owncloud
      OWNCLOUD_DB_PASSWORD: change_this_user_password
      OWNCLOUD_DB_HOST: db
      OWNCLOUD_ADMIN_USERNAME: admin
      OWNCLOUD_ADMIN_PASSWORD: change_this_admin_password
    volumes:
      - owncloud_files:/mnt/data

volumes:
  db_data:
  owncloud_files:
```

Start the containers:

```bash
docker compose up -d
```

Check status and logs:

```bash
docker compose ps
docker compose logs --tail=50 owncloud
```

Open this address in a browser:

```text
http://localhost:8080
```

Sign in using the administrator username/password configured in the compose file.

> This is a demonstration configuration. For a real deployment, use strong unique secrets, supported image versions, HTTPS, backups, and the official installation guidance.

## B. Browser demonstration

1. Sign in to ownCloud.
2. Click the **Files** area.
3. Click **＋ / New** and create a folder named `CloudLab`.
4. Upload a small text file.
5. Verify that the file appears in the file list.
6. If sharing is enabled, use the share control and demonstrate the permission options.

## C. Stop the demonstration

```bash
docker compose down
```

To remove the demonstration data as well (destructive):

```bash
docker compose down -v
```

**Expected observation:** ownCloud login page opens; a test folder/file can be created and displayed.

**Result:** ownCloud file storage and sharing features were demonstrated.

---

# Experiment 5 — User and Project Management Using OpenStack Keystone

## A. Load administrator credentials

In the terminal configured for OpenStack:

```bash
source openrc admin admin
```

Check that authentication works:

```bash
openstack token issue
```

## B. Create a project, user, and role assignment

```bash
openstack project create practical_project
```

Create a user (the CLI prompts for a password if `--password` is omitted):

```bash
openstack user create --password-prompt practical_user
```

List available roles:

```bash
openstack role list
```

Assign the `member` role (some clouds use a different role name):

```bash
openstack role add --project practical_project --user practical_user member
```

## C. Verify the configuration

```bash
openstack project list
openstack user list
openstack role assignment list --user practical_user --project practical_project
```

Show project details:

```bash
openstack project show practical_project
```

**Expected observation:** The created project and user appear in the lists, and the role assignment is displayed.

**Result:** Keystone user, project, role, and authentication operations were performed.

---

# Experiment 6 — Create an RSS Feed Using HTML and PHP

## A. Check/install PHP

On Ubuntu/Debian, if permitted by your lab:

```bash
sudo apt update
sudo apt install -y php-cli php-xml
php -v
```

Create a project folder:

```bash
mkdir rss-practical
cd rss-practical
```

## B. Create `index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>RSS Feed Publisher</title>
</head>
<body>
    <h2>Add a News Item</h2>

    <form action="feed.php" method="POST">
        <label>Title:</label><br>
        <input type="text" name="title" required><br><br>

        <label>Description:</label><br>
        <textarea name="description" required></textarea><br><br>

        <button type="submit">Publish Feed Item</button>
    </form>

    <p><a href="feed.xml">View RSS XML</a></p>
</body>
</html>
```

## C. Create `feed.php` (complete program)

```php
<?php
// Accept only form submissions.
if ($_SERVER["REQUEST_METHOD"] !== "POST") {
    http_response_code(405);
    exit("Please submit the form first.");
}

$title = trim($_POST["title"] ?? "");
$description = trim($_POST["description"] ?? "");

if ($title === "" || $description === "") {
    exit("Title and description are required.");
}

// Load existing RSS XML or create a new RSS channel.
$file = __DIR__ . "/feed.xml";

if (file_exists($file)) {
    $xml = simplexml_load_file($file);
    if ($xml === false) {
        exit("Could not read the existing RSS file.");
    }
} else {
    $xml = new SimpleXMLElement(
        '<?xml version="1.0" encoding="UTF-8"?>' .
        '<rss version="2.0"><channel></channel></rss>'
    );
    $xml->channel->addChild("title", "My Practical RSS Feed");
    $xml->channel->addChild("link", "http://localhost:8000/");
    $xml->channel->addChild("description", "RSS feed created in a PHP practical.");
}

// Add the new item. XML special characters are escaped by addChild's value handling.
$item = $xml->channel->addChild("item");
$item->addChild("title", htmlspecialchars($title, ENT_XML1 | ENT_COMPAT, "UTF-8"));
$item->addChild("description", htmlspecialchars($description, ENT_XML1 | ENT_COMPAT, "UTF-8"));
$item->addChild("pubDate", date(DATE_RSS));

if ($xml->asXML($file) === false) {
    exit("Could not save feed.xml. Check folder permissions.");
}

echo "RSS item published successfully.<br>";
echo '<a href="feed.xml">Open RSS feed</a>';
?>
```

## D. Run the practical

Start PHP's built-in development server from the project folder:

```bash
php -S localhost:8000
```

Open:

```text
http://localhost:8000/index.html
```

Enter a title and description, then click **Publish Feed Item**.

To inspect the generated XML:

```bash
cat feed.xml
```

**Expected observation:** A success message appears and `feed.xml` contains an RSS `<item>` with title, description, and publication date.

**Result:** An RSS feed was generated using HTML, PHP, and XML.

---

# Experiment 7 — CloudSim Simulation

> CloudSim APIs differ between CloudSim 3.x and CloudSim Plus. The following is a complete **CloudSim 3.x-style** example. Use the JAR/version supplied in your lab; do not mix CloudSim Plus imports with CloudSim 3.x code.

## A. Java program: `CloudSimExample.java`

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

import org.cloudbus.cloudsim.*;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.*;

public class CloudSimExample {

    private static Datacenter createDatacenter(String name) throws Exception {
        List<Pe> peList = new ArrayList<Pe>();
        peList.add(new Pe(0, new PeProvisionerSimple(1000)));

        int hostId = 0;
        int ram = 2048;
        long storage = 1000000;
        int bw = 10000;

        List<Host> hostList = new ArrayList<Host>();
        hostList.add(new Host(
            hostId,
            new RamProvisionerSimple(ram),
            new BwProvisionerSimple(bw),
            storage,
            peList,
            new VmSchedulerTimeShared(peList)
        ));

        String arch = "x86";
        String os = "Linux";
        String vmm = "Xen";
        double timeZone = 10.0;
        double cost = 3.0;
        double costPerMem = 0.05;
        double costPerStorage = 0.001;
        double costPerBw = 0.0;

        DatacenterCharacteristics characteristics =
            new DatacenterCharacteristics(
                arch, os, vmm, hostList, timeZone,
                cost, costPerMem, costPerStorage, costPerBw
            );

        return new Datacenter(
            name,
            characteristics,
            new VmAllocationPolicySimple(hostList),
            new LinkedList<Storage>(),
            0
        );
    }

    private static DatacenterBroker createBroker() throws Exception {
        return new DatacenterBroker("Broker");
    }

    public static void main(String[] args) {
        try {
            int numUsers = 1;
            Calendar calendar = Calendar.getInstance();
            boolean traceFlag = false;

            CloudSim.init(numUsers, calendar, traceFlag);

            createDatacenter("Datacenter_0");
            DatacenterBroker broker = createBroker();
            int brokerId = broker.getId();

            List<Vm> vmList = new ArrayList<Vm>();
            Vm vm = new Vm(
                0, brokerId, 1000, 1, 512,
                1000, 10000, "Xen",
                new CloudletSchedulerTimeShared()
            );
            vmList.add(vm);
            broker.submitVmList(vmList);

            List<Cloudlet> cloudletList = new ArrayList<Cloudlet>();
            UtilizationModel utilization = new UtilizationModelFull();

            Cloudlet cloudlet = new Cloudlet(
                0, 40000, 1, 300, 300,
                utilization, utilization, utilization
            );
            cloudlet.setUserId(brokerId);
            cloudlet.setVmId(0);
            cloudletList.add(cloudlet);
            broker.submitCloudletList(cloudletList);

            CloudSim.startSimulation();

            List<Cloudlet> resultList = broker.getCloudletReceivedList();
            CloudSim.stopSimulation();

            System.out.println("========== CLOUDLET RESULTS ==========");
            for (Cloudlet c : resultList) {
                System.out.println("Cloudlet ID: " + c.getCloudletId());
                System.out.println("Status: " +
                    (c.getStatus() == Cloudlet.SUCCESS ? "SUCCESS" : "FAILED"));
                System.out.println("VM ID: " + c.getVmId());
                System.out.println("Execution time: " + c.getActualCPUTime());
                System.out.println("--------------------------------------");
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## B. Compile and run

1. Add the CloudSim 3.x JAR files and their required dependencies to your Java project's build path.
2. Save the source as `CloudSimExample.java`.
3. Compile and run using your IDE, or use a classpath containing the CloudSim JAR and dependencies.

Generic terminal pattern (replace the JAR path with the actual lab JAR):

```bash
javac -cp ".:cloudsim-3.x.jar" CloudSimExample.java
java -cp ".:cloudsim-3.x.jar" CloudSimExample
```

On Windows, use `;` instead of `:` between classpath entries. If your CloudSim package requires additional JARs, include all of them.

**Expected observation:** The simulation prints the received Cloudlet ID, status, VM ID, and execution time. Exact timings depend on the simulation configuration.

**Result:** A datacenter, broker, VM, and Cloudlet were created and simulated using CloudSim.

---

# Experiment 8 — Hadoop Single-Node Setup and WordCount

> Run only in the Hadoop lab VM. Hadoop/Java compatibility and configuration file syntax depend on the installed release. The following commands assume a Linux pseudo-distributed installation where Hadoop is already extracted and configured.

## A. Set environment variables

Edit `~/.bashrc` and add paths appropriate to your installation:

```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=$HOME/hadoop
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Reload:

```bash
source ~/.bashrc
java -version
hadoop version
```

## B. Configure SSH and Hadoop

Install SSH if required:

```bash
sudo apt update
sudo apt install -y openssh-server
```

Generate a key and authorize local login (only if the lab requires passwordless localhost SSH):

```bash
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh localhost
```

Edit Hadoop configuration files in `$HADOOP_HOME/etc/hadoop`.

`core-site.xml`:

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

`hdfs-site.xml`:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

`mapred-site.xml` (create from the template if needed):

```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

`yarn-site.xml`:

```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
</configuration>
```

Set `JAVA_HOME` in `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` to the actual Java path.

## C. Start Hadoop

Format only for a fresh HDFS filesystem (formatting an existing NameNode can destroy its namespace metadata):

```bash
hdfs namenode -format
```

Start services:

```bash
start-dfs.sh
start-yarn.sh
jps
```

## D. Create input file and run WordCount

Create `sample.txt`:

```bash
cat > sample.txt <<'EOF'
hello cloud
hello world
cloud computing
EOF
```

Create HDFS input and upload:

```bash
hdfs dfs -mkdir -p /user/$USER/input
hdfs dfs -put -f sample.txt /user/$USER/input/
```

Run the example JAR (the wildcard should match the examples JAR installed in your Hadoop distribution):

```bash
hadoop jar "$HADOOP_HOME"/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/$USER/input /user/$USER/output
```

Display output:

```bash
hdfs dfs -cat /user/$USER/output/part-r-00000
```

Expected output (order may vary):

```text
cloud       2
computing   1
hello       2
world       1
```

Stop services after the practical:

```bash
stop-yarn.sh
stop-dfs.sh
```

**Result:** Hadoop WordCount was executed using HDFS and MapReduce.

---

# Experiment 9 — File Transfer Using SCP and NFS

## A. SCP between two Linux VMs

On the destination VM, find its IP:

```bash
hostname -I
```

Ensure SSH server is installed/running on the destination (Ubuntu/Debian):

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

On the source VM, create a test file:

```bash
echo "File transfer practical" > data.txt
```

Copy it to the destination (replace username and IP):

```bash
scp data.txt username@192.168.1.10:/home/username/
```

Verify on the destination:

```bash
ls -l /home/username/data.txt
cat /home/username/data.txt
```

## B. NFS server setup (Ubuntu/Debian example)

On the **server VM**:

```bash
sudo apt update
sudo apt install -y nfs-kernel-server
sudo mkdir -p /srv/nfs/shared
sudo chmod 777 /srv/nfs/shared
echo "NFS shared folder practical" | sudo tee /srv/nfs/shared/test.txt
```

Edit exports:

```bash
sudo nano /etc/exports
```

Add a line for the client network (example network only; replace with your lab subnet):

```text
/srv/nfs/shared 192.168.1.0/24(rw,sync,no_subtree_check)
```

Apply and start:

```bash
sudo exportfs -rav
sudo systemctl enable --now nfs-kernel-server
sudo exportfs -v
```

On the **client VM**:

```bash
sudo apt update
sudo apt install -y nfs-common
sudo mkdir -p /mnt/nfs
sudo mount -t nfs 192.168.1.10:/srv/nfs/shared /mnt/nfs
```

Verify:

```bash
mount | grep nfs
ls -l /mnt/nfs
cat /mnt/nfs/test.txt
```

Unmount when finished:

```bash
sudo umount /mnt/nfs
```

> The example `chmod 777` is only for a disposable classroom demonstration. Use restricted ownership and permissions in real systems.

**Result:** A file was copied using SCP and a remote directory was mounted using NFS.

---

# Experiment 10 — Deploy a Hello World Application Using Google App Engine

> Use the Google Cloud project and billing/permissions arrangement supplied by your institution. App Engine runtimes change; the sample uses a current-style Python runtime declaration. Confirm the runtime supported by your lab before deployment.

## A. Create the application files

Create a folder and enter it:

```bash
mkdir appengine-hello
cd appengine-hello
```

Create `main.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World from Google App Engine!"

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=8080, debug=True)
```

Create `requirements.txt`:

```text
Flask
gunicorn
```

Create `app.yaml` (runtime must be supported by the lab's App Engine environment):

```yaml
runtime: python312

entrypoint: gunicorn -b :$PORT main:app
```

## B. Test locally

Create/activate a virtual environment and install dependencies:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python main.py
```

Open `http://127.0.0.1:8080` and verify the message. Stop the local server with Ctrl+C.

## C. Deploy (Google Cloud CLI must be installed and authenticated)

```bash
gcloud auth login
gcloud projects list
gcloud config set project YOUR_PROJECT_ID
gcloud app create
gcloud app deploy
gcloud app browse
```

Replace `YOUR_PROJECT_ID` with the project ID assigned to you. `gcloud app create` is generally a one-time initialization step for a project.

**Expected output:** Browser displays `Hello World from Google App Engine!`.

**Result:** A Python web application was tested locally and deployed to App Engine where the lab project permitted.

---

# Experiment 11 — Google Drive, Sheets, and Keep

This practical is mainly performed in the browser; there is no required terminal program.

## A. Google Drive

1. Open Google Drive and sign in with the lab account.
2. Click **New → New folder**.
3. Name it `CloudComputingPractical`.
4. Open the folder and click **New → Google Docs** or upload a test file.
5. Use the **Share** option to demonstrate Viewer, Commenter, and Editor permissions, if permitted by the lab.

## B. Google Sheets formulas

1. In Drive, click **New → Google Sheets**.
2. Enter this sample data:

| A | B | C |
|---|---:|---:|
| Student | Practical 1 | Practical 2 |
| Amit | 80 | 90 |
| Neha | 70 | 85 |
| Ravi | 75 | 80 |

3. In `D1`, type `Total`; in `D2`, enter:

```excel
=SUM(B2:C2)
```

4. In `E1`, type `Average`; in `E2`, enter:

```excel
=AVERAGE(B2:C2)
```

5. Drag the fill handle down for the remaining rows.
6. Try these formulas in empty cells:

```excel
=MAX(B2:C4)
=MIN(B2:C4)
=COUNT(B2:C4)
```

Expected values:
- Amit total = 170; average = 85
- Neha total = 155; average = 77.5
- Ravi total = 155; average = 77.5
- Maximum = 90; minimum = 70; count = 6

## C. Google Keep

1. Open Google Keep.
2. Click **Take a note**.
3. Enter the title `Cloud Computing Practical`.
4. Add a short note or use **New list** to create a checklist.
5. Close the note; verify it is saved.

**Result:** Cloud storage, spreadsheet formulas, collaboration/sharing, and online note-taking were demonstrated.

---

# Experiment 12 — Study of Justcloud / Cloud Backup

> Justcloud's availability and client support may vary by region and date. Use the institution's provided account/software if available. Do not download installers from unofficial mirrors.

## Practical procedure

1. Open the official service or the institution-provided backup application.
2. Sign in using the authorized lab account.
3. Locate the backup selection/settings screen.
4. Select a small non-sensitive test folder only.
5. If available, choose the backup schedule and start the backup.
6. Wait for the client to report completion.
7. Open the cloud file listing and confirm that the test file is present.
8. Use the restore/download option to recover the test file to a different folder.
9. Compare the restored file with the original.

## Optional local demonstration of file backup concepts

On Linux, create a sample folder and archive it:

```bash
mkdir -p ~/backup-demo
echo "Cloud backup practical test" > ~/backup-demo/test.txt
tar -czf ~/backup-demo.tar.gz -C ~ backup-demo
ls -lh ~/backup-demo.tar.gz
```

Verify the archive contents:

```bash
tar -tzf ~/backup-demo.tar.gz
```

Extract to a restore folder:

```bash
mkdir -p ~/restore-demo
tar -xzf ~/backup-demo.tar.gz -C ~/restore-demo
cat ~/restore-demo/backup-demo/test.txt
```

**Expected observation:** The selected test file is backed up and can be restored. The local archive commands demonstrate the backup/restore concept only; they do not upload data to Justcloud.

**Result:** Cloud backup, scheduling, and restoration concepts were studied.

---

# Experiment 13 — Cloud IDE: Execute Python and Node.js Programs

> AWS Cloud9 availability for new customers has changed. Use the cloud IDE/workspace supplied by your institution (or the local terminal for the same program demonstration).

## A. Python

Create a file named `script.py`:

```python
print("Hello from the cloud IDE!")

a = 10
b = 20
print("Sum =", a + b)
```

Run:

```bash
python3 script.py
```

Expected output:

```text
Hello from the cloud IDE!
Sum = 30
```

## B. Node.js

Check Node.js:

```bash
node --version
```

Create `server.js`:

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello from Node.js!\n");
});

server.listen(3000, "0.0.0.0", () => {
    console.log("Server running on port 3000");
});
```

Run:

```bash
node server.js
```

In another terminal, test the server:

```bash
curl http://127.0.0.1:3000
```

Expected response:

```text
Hello from Node.js!
```

Stop the server using Ctrl+C.

**Result:** Python and Node.js programs were executed in a cloud-based development environment.

---

# Experiment 14 — Container-Based Workspace and Scaling Using Eclipse Che / Codenvy

> Eclipse Che/Codenvy interfaces and workspace setup differ by version. Use the Kubernetes/container environment supplied by your lab. The Docker example below demonstrates a simple containerized web page; the Kubernetes commands demonstrate scaling where a cluster is available.

## A. Create a web page

Create a folder:

```bash
mkdir container-web
cd container-web
```

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Cloud Lab</title>
</head>
<body>
    <h1>Hello from the container workspace!</h1>
</body>
</html>
```

Create `Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

## B. Build and run using Docker

```bash
docker build -t cloud-lab-web .
docker run -d --name cloud-lab-container -p 8080:80 cloud-lab-web
```

Test:

```bash
curl http://localhost:8080
```

Expected HTML includes:

```html
<h1>Hello from the container workspace!</h1>
```

View and stop the container:

```bash
docker ps
docker stop cloud-lab-container
docker rm cloud-lab-container
```

## C. Demonstrate scaling with Kubernetes (only if a Kubernetes cluster is configured)

Create `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloud-lab-web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cloud-lab-web
  template:
    metadata:
      labels:
        app: cloud-lab-web
    spec:
      containers:
        - name: web
          image: nginx:alpine
          ports:
            - containerPort: 80
```

Apply and verify:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
```

Scale to four replicas:

```bash
kubectl scale deployment cloud-lab-web --replicas=4
kubectl get pods
```

Clean up:

```bash
kubectl delete deployment cloud-lab-web
```

**Expected observation:** Docker runs the web page in a container. If Kubernetes is available, the deployment changes from two to four pod replicas.

**Result:** Container-based development and application scaling were demonstrated.

---

# Experiment 15 — Study of Eucalyptus Private Cloud

> Eucalyptus is an older private-cloud platform and installation instructions are strongly release/OS dependent. Do not run guessed package installation commands on a modern system. Use the preconfigured Eucalyptus lab environment or virtual machine supplied by your institution.

## Practical procedure in a configured lab

1. Log in to the Eucalyptus cloud controller or lab management machine.
2. Open the Eucalyptus management interface or command-line environment provided by the instructor.
3. Load the Eucalyptus credentials if the lab provides an environment file:

```bash
source eucarc
```

4. List availability zones:

```bash
euca-describe-availability-zones
```

5. List registered images:

```bash
euca-describe-images
```

6. List instances:

```bash
euca-describe-instances
```

7. List storage volumes (if the storage service is configured):

```bash
euca-describe-volumes
```

8. Record the output and identify the Cloud Controller, Cluster Controller, Storage Controller, Walrus, and Node Controller roles.

> If `euca-describe-*` commands are not found, the Euca2ools client is not installed or its environment is not configured. Ask the lab administrator for the correct client package and credentials. Do not invent output.

**Expected observation:** The configured cloud's availability zones, images, instances, or volumes are listed, depending on the services enabled.

**Result:** Eucalyptus private-cloud architecture and its configured resources were studied.

---

# Final Practical Exam Checklist

For each experiment, be ready to write:

1. Aim
2. Requirements / software
3. Theory and important terms
4. Procedure in numbered steps
5. Full program or commands (where applicable)
6. Output / screenshot / observation
7. Result

## Commands worth remembering

| Purpose | Command |
|---|---|
| List VirtualBox VMs | `VBoxManage list vms` |
| OpenStack token | `openstack token issue` |
| OpenStack users | `openstack user list` |
| Start PHP test server | `php -S localhost:8000` |
| Hadoop version | `hadoop version` |
| Start HDFS | `start-dfs.sh` |
| Start YARN | `start-yarn.sh` |
| HDFS list | `hdfs dfs -ls /` |
| SCP copy | `scp file user@IP:/path/` |
| NFS mount | `sudo mount -t nfs SERVER:/share /mnt/nfs` |
| App Engine deploy | `gcloud app deploy` |
| Python run | `python3 script.py` |
| Node run | `node server.js` |
| Docker build | `docker build -t image-name .` |
| Docker run | `docker run -p 8080:80 image-name` |
| Kubernetes scale | `kubectl scale deployment NAME --replicas=4` |
| Eucalyptus zones | `euca-describe-availability-zones` |

## Exam note

The code and commands above are study/practical examples. Use your teacher's prescribed software versions, lab IP addresses, credentials, and project configuration. For GUI-only services, describe and demonstrate the actual interface actions. Record the output you really obtained; if you did not run a command, label the output as **expected output** rather than actual output.
