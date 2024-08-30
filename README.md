# MICROPROYECTO_1_COMP

##########################################################

# Instalar Consul en los 3 servidores (HAPROXY, web1 y web2):

#Comando 1: 

(Para arquitectura AMD):
curl -O https://releases.hashicorp.com/consul/1.11.3/consul_1.11.3_linux_amd64.zip

# (Para arquitectura ARM):
wget https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_arm64.zip


##########################################################

# Configuracion de Consul en el servidor HAPROXY:

cat <<EOF | sudo tee /etc/consul.d/consul.hcl
data_dir = "/opt/consul"
server = true
bootstrap_expect = 1
bind_addr = "192.168.50.15"
client_addr = "0.0.0.0"
EOF

##########################################################

# Configurar Consul como agente en web 1 y web 2

cat <<EOF | sudo tee /etc/consul.d/consul.hcl
data_dir = "/opt/consul"
retry_join = ["192.168.50.15"]
EOF

##########################################################

# Clonar el repositorio del servicio de node en web1 y web2

git clone https://github.com/omondragon/consulService

# dirigirse a la carpeta donde se clono el repositorio:

cd consulService/app

# Modificar el archivo index.js colocando el la ip del servidor correspondiente

sudo nano index.js

# para web 1 modificar la linea:

conts HOST='192.168.100.3'

# por

conts HOST='192.168.56.11'

# para web 2 modificar la linea:

conts HOST='192.168.100.3'

# por

conts HOST='192.168.56.12'

##########################################################

# Iniciar Consul como servicio para HAPROXY
nohup consul agent -server -bind=192.168.50.15 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -ui &

##########################################################

# Iniciar Consul como servicio para web 1
nohup consul agent -node=web1 -bind=192.168.50.13 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -join=192.168.50.15 -ui &

##########################################################

# Iniciar Consul como servicio para web 2
nohup consul agent -node=web2 -bind=192.168.50.14 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -join=192.168.50.15 -ui &

##########################################################

# Arrancar servidores node en web1 y web2:

# Dirigirse a la ubucacion de inex.js del repositorio clonado:

cd consulService/app

# Arrancar 3 instancias de node:

nohup node index.js 3000 &
nohup node index.js 3001 &
nohup node index.js 3002 &

##########################################################

# Apagar servidores de node:

# Buscar los PID de todos los procesos de node que estan corriendo  :

ps aux | grep node

# Matar los procesos requeridos:

sudo kill -9 <PID_NUMBER>

##########################################################

URLS:
Consul: http://192.168.56.13:8500/ui
HAProxy: http://192.168.56.13/haproxy?stats
Peticiones balanceadas: http://192.168.56.13

##########################################################