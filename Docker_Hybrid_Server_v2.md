# Docker Hybrid Server

Install Apache and PHP directly on Ubuntu and install MySQL inside a Docker container. This is a hybrid setup.

---

## Step 1 – Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker  # Docker containers listed without sudo
```

**Check that it works:**
```bash
docker --version
docker ps
```

---

## Step 2 – Create a persistent data folder
This keeps your database files safe even if you delete the container.
```bash
sudo mkdir -p /srv/mysql55/data
sudo chmod 777 /srv/mysql55/data
```

---

## Step 3 – Run MySQL 5.5 container
```bash
docker run -d   --name mysql55   -e MYSQL_ROOT_PASSWORD=rootpass   -v /srv/mysql55/data:/var/lib/mysql   -p 3306:3306   mysql:5.5
```

**Check if it’s running:**
```bash
docker ps
```
You should see something like:
```
CONTAINER ID   IMAGE       NAME       PORTS
8ccb6840bb36   mysql:5.5   mysql55    0.0.0.0:3306->3306/tcp
```
`0.0.0.0:3306->3306/tcp` → MySQL port is exposed on all host IPs, ready for remote connection.

---

## Step 4 – Connect to MySQL
```bash
docker exec -it mysql55 mysql -uroot -prootpass
```

---

## Step 5 – (Optionally) Enable auto-start
```bash
docker update --restart always mysql55
```

---

## Allow Remote Access (e.g., from Navicat)
Connect directly via Docker.  
You can skip installing a client on the host and use the container itself:

```bash
docker exec -it mysql55 mysql -uroot -prootpass
```

Then inside MySQL prompt, run:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'noklk'@'10.20.200.4' IDENTIFIED BY 'rootpass' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

**Make sure host port 3306 is exposed:**
```bash
docker ps
```
You should see:
```
0.0.0.0:3306->3306/tcp
```

**Ensure firewall allows port 3306:**
```bash
sudo ufw allow 3306/tcp
sudo ufw status
```

---

## Step 6 – Create the Test PHP File
Create a test PHP file to ensure Apache and MySQL are working correctly.

```bash
nano /var/www/html/dbtest.php
```

Paste this code:
```php
<?php
$mysqli = new mysqli("127.0.0.1", "root", "rootpass");
if ($mysqli->connect_error) {
    die("❌ Connection failed: " . $mysqli->connect_error);
}
echo "✅ Connected to MySQL 5.5 inside Docker successfully!";
?>
```

**Restart Apache**
```bash
sudo systemctl restart apache2
```

**Open in your browser:**
```
http://172.16.0.1/dbtest.php
```

---

## 🧪 Expected Results

✅ If PHP is working and Docker MySQL is accessible, you’ll see:
```
Connected to MySQL 5.5 inside Docker successfully!
```

❌ If you see PHP code as text → PHP module isn’t loaded.  
❌ If you see “Connection failed” → Database isn’t reachable (check `docker ps` and port 3306).
