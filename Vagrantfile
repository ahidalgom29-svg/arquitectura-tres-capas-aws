Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "AngelHidalgoMoralesBalanceador" do |bal|
    bal.vm.hostname = "AngelHidalgoMoralesBalanceador"
    bal.vm.provider "virtualbox" do |vb|
      vb.name = "AngelHidalgoMoralesBalanceador"
      vb.memory = "512"
    end
    bal.vm.network "public_network"
    bal.vm.network "private_network", ip: "10.0.1.1", virtualbox__intnet: "red_capa2"

    bal.vm.provision "shell", inline: <<-SHELL
      apt-get update
      DEBIAN_FRONTEND=noninteractive apt-get install -y nginx certbot python3-certbot-nginx iptables-persistent

      cat << 'EOF' > /etc/nginx/sites-available/default
upstream backend_webs {
    server 10.0.1.11;
    server 10.0.1.12;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://backend_webs;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF
      systemctl restart nginx
    SHELL
  end

  config.vm.define "AngelHidalgoMoralesWeb1" do |web1|
    web1.vm.hostname = "AngelHidalgoMoralesWeb1"
    web1.vm.provider "virtualbox" do |vb|
      vb.name = "AngelHidalgoMoralesWeb1"
      vb.memory = "512"
    end
    web1.vm.network "private_network", ip: "10.0.1.11", virtualbox__intnet: "red_capa2"
    web1.vm.network "private_network", ip: "10.0.3.11", virtualbox__intnet: "red_capa3"

    web1.vm.provision "shell", inline: <<-SHELL
      apt-get update
      DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 php php-mysql iptables-persistent
      rm -f /var/www/html/index.html

      cat << 'EOF' > /var/www/html/index.php
<?php
$conn = new mysqli("10.0.3.20", "usuario_web", "1234", "practica");
$estado_bbdd = $conn->connect_error ? "Error de conexion" : "Conectado correctamente a MariaDB (10.0.3.20)";
echo "<h1>Servidor Web 1 - Angel Hidalgo Morales</h1>";
echo "<p>Estado BBDD Capa 3: " . $estado_bbdd . "</p>";
?>
EOF
      systemctl restart apache2
    SHELL
  end

  config.vm.define "AngelHidalgoMoralesWeb2" do |web2|
    web2.vm.hostname = "AngelHidalgoMoralesWeb2"
    web2.vm.provider "virtualbox" do |vb|
      vb.name = "AngelHidalgoMoralesWeb2"
      vb.memory = "512"
    end
    web2.vm.network "private_network", ip: "10.0.1.12", virtualbox__intnet: "red_capa2"
    web2.vm.network "private_network", ip: "10.0.3.12", virtualbox__intnet: "red_capa3"

    web2.vm.provision "shell", inline: <<-SHELL
      apt-get update
      DEBIAN_FRONTEND=noninteractive apt-get install -y apache2 php php-mysql iptables-persistent
      rm -f /var/www/html/index.html

      cat << 'EOF' > /var/www/html/index.php
<?php
$conn = new mysqli("10.0.3.20", "usuario_web", "1234", "practica");
$estado_bbdd = $conn->connect_error ? "Error de conexion" : "Conectado correctamente a MariaDB (10.0.3.20)";
echo "<h1>Servidor Web 2 - Angel Hidalgo Morales</h1>";
echo "<p>Estado BBDD Capa 3: " . $estado_bbdd . "</p>";
?>
EOF
      systemctl restart apache2
    SHELL
  end

  config.vm.define "AngelHidalgoMoralesBBDD" do |bbdd|
    bbdd.vm.hostname = "AngelHidalgoMoralesBBDD"
    bbdd.vm.provider "virtualbox" do |vb|
      vb.name = "AngelHidalgoMoralesBBDD"
      vb.memory = "768"
    end
    bbdd.vm.network "private_network", ip: "10.0.3.20", virtualbox__intnet: "red_capa3"

    bbdd.vm.provision "shell", inline: <<-SHELL
      apt-get update
      DEBIAN_FRONTEND=noninteractive apt-get install -y mariadb-server iptables-persistent

      sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/mariadb.conf.d/50-server.cnf
      systemctl restart mariadb

      mysql -e "CREATE DATABASE IF NOT EXISTS practica;"
      mysql -e "CREATE USER IF NOT EXISTS 'usuario_web'@'10.0.3.%' IDENTIFIED BY '1234';"
      mysql -e "GRANT ALL PRIVILEGES ON practica.* TO 'usuario_web'@'10.0.3.%';"
      mysql -e "FLUSH PRIVILEGES;"
    SHELL
  end
end