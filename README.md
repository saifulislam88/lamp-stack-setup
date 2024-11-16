
# **LAMP Stack Setup with HTTPS and PHP-Database Integration on Ubuntu 22.04**

This guide will walk you through setting up a fully functional LAMP stack on Ubuntu 22.04. It includes Apache Virtual Hosting with HTTPS (self-signed SSL), a PHP application for inserting and viewing data, and MariaDB integration for dynamic content.

---

## **1. Update System Packages**

Start by updating the package repository and upgrading installed packages:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## **2. Install Required Packages**

Install Apache, MariaDB, PHP, and additional modules:

```bash
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-mysql openssl
```

Enable necessary Apache modules:

```bash
sudo a2enmod rewrite ssl
```

---

## **3. Configure Apache Virtual Hosting**

### 3.1. **Create a Directory for the Application**

Create a directory for your application:

```bash
sudo mkdir -p /var/www/myapp
sudo chown -R $USER:$USER /var/www/myapp
```

### 3.2. **Create a Virtual Host Configuration**

Create a new virtual host configuration file:

```bash
sudo nano /etc/apache2/sites-available/myapp.conf
```

Add the following configuration:

```apache
<VirtualHost *:80>
    ServerName myapp.local
    Redirect permanent / https://myapp.local/
</VirtualHost>

<VirtualHost *:443>
    ServerName myapp.local
    DocumentRoot /var/www/myapp

    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/myapp.crt
    SSLCertificateKeyFile /etc/apache2/ssl/myapp.key

    <Directory /var/www/myapp>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/myapp_error.log
    CustomLog ${APACHE_LOG_DIR}/myapp_access.log combined
</VirtualHost>
```

### 3.3. **Enable the Site and Reload Apache**

Enable the new site and restart Apache:

```bash
sudo a2ensite myapp.conf
sudo systemctl reload apache2
```

---

## **4. Set Up a Self-Signed SSL Certificate**

Create the directory for SSL certificates:

```bash
sudo mkdir -p /etc/apache2/ssl
```

Generate a self-signed SSL certificate:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048     -keyout /etc/apache2/ssl/myapp.key     -out /etc/apache2/ssl/myapp.crt     -subj "/C=US/ST=State/L=City/O=Organization/CN=myapp.local"
```

---

## **5. Configure MariaDB**

### 5.1. **Secure the MariaDB Installation**

Run the secure installation script and follow the prompts:

```bash
sudo mysql_secure_installation
```

### 5.2. **Create a Database and User**

Log in to MariaDB:

```bash
sudo mysql -u root -p
```

Run the following commands to create a database, user, and table:

```sql
CREATE DATABASE myappdb;
CREATE USER 'myappuser'@'localhost' IDENTIFIED BY 'myapppassword';
GRANT ALL PRIVILEGES ON myappdb.* TO 'myappuser'@'localhost';
FLUSH PRIVILEGES;
USE myappdb;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);
EXIT;
```

---

## **6. Create the PHP Application**

### 6.1. **Write the PHP Code**

Create a PHP file in your application directory:

```bash
sudo nano /var/www/myapp/index.php
```

Add the following PHP code:

```php
<?php
\$servername = "localhost";
\$username = "myappuser";
\$password = "myapppassword";
\$dbname = "myappdb";

\$conn = new mysqli(\$servername, \$username, \$password, \$dbname);

if (\$conn->connect_error) {
    die("Connection failed: " . \$conn->connect_error);
}

if (\$_SERVER["REQUEST_METHOD"] == "POST") {
    \$name = \$_POST['name'];
    \$email = \$_POST['email'];
    \$sql = "INSERT INTO users (name, email) VALUES ('\$name', '\$email')";

    if (\$conn->query(\$sql) === TRUE) {
        echo "New record created successfully";
    } else {
        echo "Error: " . \$sql . "<br>" . \$conn->error;
    }
}

\$result = \$conn->query("SELECT * FROM users");
?>
<!DOCTYPE html>
<html>
<head>
    <title>MyApp</title>
</head>
<body>
    <h1>Insert Data</h1>
    <form method="POST">
        Name: <input type="text" name="name" required><br>
        Email: <input type="email" name="email" required><br>
        <button type="submit">Submit</button>
    </form>
    <h1>View Data</h1>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
        </tr>
        <?php
        if (\$result->num_rows > 0) {
            while (\$row = \$result->fetch_assoc()) {
                echo "<tr><td>" . \$row["id"] . "</td><td>" . \$row["name"] . "</td><td>" . \$row["email"] . "</td></tr>";
            }
        }
        ?>
    </table>
</body>
</html>
```

---

### 6.2. **Set Permissions**

Set proper ownership and permissions for the directory:

```bash
sudo chown -R www-data:www-data /var/www/myapp
sudo chmod -R 755 /var/www/myapp
```

---

## **7. Add Local DNS Entry**

To test the application locally, add the hostname to your `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```

Add the following line:

```
127.0.0.1 myapp.local
```

---

## **8. Restart Apache**

Restart Apache to apply all changes:

```bash
sudo systemctl restart apache2
```

---

## **9. Test the Application**

1. Open a web browser and navigate to **[https://myapp.local](https://myapp.local)**.  
2. Insert data into the form.  
3. View the submitted data in the table below the form.

---

### **Congratulations!**  
Your LAMP stack with HTTPS and PHP-MariaDB integration is successfully set up.
