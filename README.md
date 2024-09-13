# Kafka Distribuído

Lucas Rafael Alves de Souza

Sérgio Henrique Mendes de Assis

Criar 4 instâncias de máquinas virtuais na Nuvem da Oracle: 2 Brokers, 1 Consumer e 1 Producer

<img width="445" alt="image" src="https://github.com/user-attachments/assets/7aa83748-bc1d-4881-85cb-afd2fd360b83">

Geração de chave ssh no terminal para acesso nas máquinas e colocar a chave pública na criação da instância:

```
ssh-keygen -t rsa -b 2048
```

<img width="239" alt="image" src="https://github.com/user-attachments/assets/14eed280-98f3-47fc-9fdf-2bee4108aa7c">

<img width="603" alt="image" src="https://github.com/user-attachments/assets/2fbfce9f-8cf8-4d1d-9859-67885ae42151">

Repetir o processo de criação das máquinas para as outras intâncias atribuindo IPs privados 10.0.0.7-10.0.0.10:

<img width="412" alt="image" src="https://github.com/user-attachments/assets/419c29b4-fd61-4632-bf9c-ca2a56622957">

Editar na lista de segurança da sub-rede abrindo todas as portas das aplicações nas faixas de IPs privados das instâncias (10.0.0.0/24) e permir o acesso por ssh:

<img width="663" alt="image" src="https://github.com/user-attachments/assets/c4288c29-0388-4118-92b5-54cde4d2cf6f">

## Acesso as máquinas virtuais por ssh e instalação das ferramentas

ssh nas máquinas usando o IP público gerado nas instâncias: 

```
ssh -i /caminho/da/chave/privada/gerada/private_key opc@IP_publico_instâncias
```

<img width="759" alt="image" src="https://github.com/user-attachments/assets/7dd39282-a64b-492f-ad00-68a3547cfd7c">


Rodar em todas as instâncias:

```
sudo apt-get update
sudo apt-get install ssh
```

Gerar par de chaves pública/privada nos Brokers 1 e 2:

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```

Nos nós Consumer e Producer copiar chaves geradas:

```
nano .ssh/authorized_keys
```

Em todas as máquinas:

```
sudo nano /etc/hosts
```

Copiar os hostnames de todas as máquinas:

```
10.0.0.9  kafka-cluster
10.0.0.10 kafka-cluster
10.0.0.7   consumer
10.0.0.8   producer
10.0.0.9   broker1
10.0.0.10   broker2
```

Rodar nos Brokers 1 e 2 para instalar o Zookeeper:

```
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.9.2/apache-zookeeper-3.9.2-bin.tar.gz
sudo mkdir -p /data/zookeeper
mv zookeeper/ /usr/local
sudo firewall-cmd --permanent --add-port=2181/tcp
sudo chown -R ubuntu:ubuntu /data/zookeeper
sudo nano /usr/local/zookeeper/conf/zoo.cfg
```

Adicionar:

```
tickTime = 2000

dataDir = /data/zookeeper

clientPort = 2181

initLimit = 5

syncLimit = 2
```

Iniciar o Zookeeper:

```
sudo /usr/local/zookeeper/bin/zkServer.sh start
```

Acessar o servidor no localhost e verificar instalação:

```
sudo /usr/local/zookeeper/bin/zkCli.sh -server 127.0.0.1:2181

sudo /usr/local/zookeeper/bin/zkServer.sh stop
sudo nano /etc/systemd/system/zookeeper.service
```

Adicionar:
```
[Unit]
Description=Zookeeper Daemon
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target
[Service]
Type=forking
WorkingDirectory=/usr/local/zookeeper
User=root
Group=root
ExecStart=/usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo.cfg
ExecStop=/usr/local/zookeeper/bin/zkServer.sh stop /usr/local/zookeeper/conf/zoo.cfg
ExecReload=/usr/local/zookeeper/bin/zkServer.sh restart /usr/local/zookeeper/conf/zoo.cfg
TimeoutSec=30
Restart=on-failure
[Install]
WantedBy=default.target
```

Recarregar os daemons:

```
sudo systemctl daemon-reload
sudo systemctl start zookeeper
sudo systemctl status zookeeper
```

Instalar kafka em todas as máquinas:

```
wget https://downloads.apache.org/kafka/3.7.0/kafka_2.12-3.7.0.tgz
tar xzf kafka_2.12-3.7.0.tgz
mv kafka_2.12-3.7.0 kafka
sudo mv kafka /usr/local/
cd /usr/local/kafka/config
nano /usr/local/kafka/config/server.properties
kafka/config/server.properties
```

Adiionar no fim:

```
delete.topic.enable = true
log.dirs=/usr/local/kafka/logs
```

```
sudo nano /etc/systemd/system/zookeeper.service
```

Editar:

```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=rootc
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/confi>
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
sudo nano /etc/systemd/system/kafka.service
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=root
ExecStart=/bin/sh -c '/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kaf>
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

Iniciar a aplicação:
```
sudo systemctl start kafka
```

Editar em todos os nós:

```
nano ~/.bashrc
```

<img width="305" alt="image" src="https://github.com/user-attachments/assets/b823235a-ee4e-45a8-8b72-648a193fbc52">

```
export KAFKA_HOME="/usr/odp/1.2.2.0-128/kafka"
export PATH=$PATH:$KAFKA_HOME/bin
export PATH=$KAFKA_HOME/bin:$PATH
export ZOOKEEPER_HOME="/usr/odp/1.2.2.0-128/zookeeper"
export PATH=$PATH:$ZOOKEEPER_HOME/bin
export PATH=$ZOOKEEPER_HOME/bin:$PATH
```

Rodar:

```
source ~/.bashrc
```


Rodar comandos para instalar habilitar portas para o Zookeeper e Kafka no firewall:

```
sudo apt install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-port=2181/tcp
sudo firewall-cmd --permanent --add-port=6667/tcp
sudo firewall-cmd --reload
```






