# Sistema de Arquivos Distribuídos para Big Data

Lucas Rafael Alves Souza

Sérgio Henrique Mendes de Assis

## Criação de instâncias na Oracle

Criar 4 instâncias: 2 Brokers, 1 Consumer e 1 Producer

<img width="549" alt="image" src="https://github.com/user-attachments/assets/87ebddb9-194e-4efb-a4ef-2b2704386f1c">

<img width="605" alt="image" src="https://github.com/user-attachments/assets/2e7ad807-4f9c-4655-8a28-47a3e5dd2f38">

Geração de chave ssh no terminal para acesso nas máquinas e colocar a chave pública na criação da instância:

```
ssh-keygen -t rsa -b 2048
```

<img width="239" alt="image" src="https://github.com/user-attachments/assets/14eed280-98f3-47fc-9fdf-2bee4108aa7c">

<img width="603" alt="image" src="https://github.com/user-attachments/assets/2fbfce9f-8cf8-4d1d-9859-67885ae42151">

Repetir o processo de criação das máquinas para as outras intâncias atribuindo IPs privados 10.0.0.2-10.0.0.5:

<img width="1003" alt="image" src="https://github.com/user-attachments/assets/d08a722b-9c0d-43b5-9aad-526e3100bb21">


Editar na lista de segurança da sub-rede abrindo todas as portas das aplicações nas faixas de IPs privados das instâncias (10.0.0.0/24):

<img width="863" alt="image" src="https://github.com/user-attachments/assets/c4288c29-0388-4118-92b5-54cde4d2cf6f">

## Acesso as máquinas virtuais por ssh e instalação das ferramentas

ssh nas máquinas usando o IP público gerado nas instâncias: 

```
ssh -i /caminho/da/chave/privada/gerada/private_key opc@IP_publico_instâncias
```

<img width="822" alt="image" src="https://github.com/user-attachments/assets/cff64b0e-cafc-46a7-8b99-2149570e2da2">


Rodar em todas as instâncias:

```
sudo apt-get update
sudo apt-get install ssh
```

Gerar par de chaves pública/privada no nó mestre:

```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/authorized_keys
```

Nos Worker1, Worker2 e Worker3:

```
nano .ssh/authorized_keys
```

Em todas as máquinas:

```
sudo nano /etc/hosts
```

Copiar os hostnames de todas as máquinas:

```
10.0.0.5      worker1
10.0.0.4      worker2
10.0.0.3      worker3
10.0.0.2      master
```

Rodar para instalar pacotes hadoop e jvm:

```
sudo apt-get -y install openjdk-8-jdk-headless
sudo wget -P ~ https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0-aarch64.tar.gz
tar xzf hadoop-3.4.0-aarch64.tar.gz
mv hadoop-3.4.0 hadoop
nano hadoop/etc/hadoop/hadoop-env.sh
```

Copiar:

```
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-arm64/
```

```
sudo mv hadoop /usr/local/hadoop
sudo nano /etc/environment
```

Copiar:

```
JAVA_HOME=”/usr/lib/jvm/java-8-openjdk-arm64/jre”
```

Editar em todas:

```
nano ~/.bashrc
```

<img width="398" alt="image" src="https://github.com/user-attachments/assets/9147615a-4e44-4adb-9a31-31abc5cb5907">

```
#Hadoop Related Options
export HADOOP_HOME="/usr/local/hadoop"
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

Rodar:

```
source ~/.bashrc
```

Reiniciar todas as máquinas e reconectar. Então:

```
nano /usr/local/hadoop/etc/hadoop/core-site.xml
```
Editar:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://10.0.0.2:9000</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

Editar:

```
configuration>
    <property>
        <name>dfs.replication</name>
        <value>4</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/yarn-site.xml
```

Editar:

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>10.0.0.2</value>
    </property>
</configuration>
```

```
nano /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

Editar:
```
<configuration>
    <property>
        <name>mapreduce.jobtracker.address</name>
        <value>10.0.0.2:54311</value>
    </property>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

Em todas:

```
nano /usr/local/hadoop/etc/hadoop/masters
```


Rodar comandos para instalar habilitar portas para Zookeeper e Kafka no firewall:

```
sudo apt install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-port=/tcp
sudo firewall-cmd --permanent --add-port=/tcp
sudo firewall-cmd --reload
```






