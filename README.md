# Spring Petclinic Energy Testing

See the blog post [Evolution of Energy Usage in Spring Boot](https://medium.com/@mstocker/69c7c372dba3) for more information about this repository.

## Project and Master Thesis

In case you are interested in this forked project due to my project or master thesis, you can `git checkout` the respective tags: `project-thesis` and `master-thesis`.

## Running the Tests on Linux

To run the tests, you can use the `run_single_measurement.sh` script. This script will run the tests for a specific Spring Boot version, JVM version, Java version, and web server. For example, to run the tests for Spring Boot 3.3.1, JVM 21.0.4-tem, Java 21, and Tomcat, you can run the following command:

```bash
./run_single_measurement.sh \
  --spring-boot-version 3.3.1 \
  --jvm-version 21.0.4-tem \
  --java-version 21 \
  --virtual-threads "false"
  --webserver tomcat
```

Note that not all combinations of Spring Boot versions, JVM versions, Java versions, and web servers are supported by the script (e.g., not all JVMs support virtual threads).

## Setup Instructions on Linux

Follow these steps to prepare a Ubuntu 24.04 machine:

```
sudo apt-get install unzip zip sysfsutils docker.io
docker ps
sudo systemctl status docker
sudo gpasswd -a ubuntu docker
newgrp docker

curl -s "https://get.sdkman.io" | bash
source "/home/ubuntu/.sdkman/bin/sdkman-init.sh"

sdk install java 23.0.1-tem 
sdk install java 21.0.5-tem 
sdk install java 17.0.13-tem # set this as default when asked

git clone https://github.com/joular/joularjx.git
cd joularjx/
git reset --hard 3.0.1
./mvnw package

cd 
git clone https://github.com/spring-petclinic/spring-petclinic-rest.git
cd spring-petclinic-rest/
git reset --hard b4c7dd5713dbaaa12742d785f1bab28e7707e186

cd
git clone https://github.com/misto/spring-petclinic-energy-benchmarking.git

wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
tar -xvzf apache-jmeter-5.6.3.tgz 

echo 'export JMETER_HOME=/home/ubuntu/apache-jmeter-5.6.3' | tee -a .bashrc 
echo 'export PATH=$JMETER_HOME/bin:$PATH' | tee -a .bashrc 

source ~/.bashrc
jmeter --version # should print 5.6.3

echo 'mode class/powercap/intel-rapl:0/energy_uj = 0440' | sudo tee -a  /etc/sysfs.conf
echo 'owner class/powercap/intel-rapl:0/energy_uj = root:ubuntu' | sudo tee -a  /etc/sysfs.conf

sudo reboot

cd spring-petclinic-energy-benchmarking/

screen

./run_series.sh 
```

## Setup Instructions on macOS

Follow these steps to prepare a macOS machine (e.g. Mac mini):

Install Docker Desktop.

```
curl -s "https://get.sdkman.io" | bash
source "/Users/m1/.sdkman/bin/sdkman-init.sh"

sdk install java 23.0.1-tem 
sdk install java 21.0.5-tem 
sdk install java 17.0.13-tem

git clone https://github.com/joular/joularjx.git
cd joularjx/
./mvnw package

cd 
git clone https://github.com/j-ruch/spring-petclinic-rest.git
cd spring-petclinic-rest/
git reset --hard 5848de84f424c2cbfc0145f687b43f84a219e36c

cd
git clone https://github.com/j-ruch/spring-petclinic-energy-benchmarking.git

curl https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz -o apache-jmeter-5.6.3.tgz
tar -xvzf apache-jmeter-5.6.3.tgz 

echo 'export JMETER_HOME=/Users/m1/apache-jmeter-5.6.3' | tee -a ~/.zshrc
echo 'export PATH=$JMETER_HOME/bin:$PATH' | tee -a ~/.zshrc 

source ~/.zshrc
jmeter --version # should print 5.6.3

cd spring-petclinic-energy-benchmarking/

screen

./run_series.sh 
```

## Running the Tests on Windows

Follow the installation details for JoularJX on the official GitHub [repository](https://github.com/joular/joularjx) including the Windows RAPL driver and the PowerMonitor.exe file.

### Driver Installation for JoularJX-3.0.1

Download the necessary [Scaphandre driver](https://scaphandre.s3.fr-par.scw.cloud/x86_64/scaphandre_0.5.0_installer.exe), execute the .exe file, and install the Scaphandre service with the following command.
The Prometheus push gateway is not required to run, it is an additional service that can be used to store the data.

```bash
sc.exe create Scaphandre binPath="C:\Program Files (x86)\scaphandre\scaphandre.exe prometheus-push -H localhost -s 45 -S http -p 9091 --no-tls-check" DisplayName=Scaphandre start=auto
```

Open the Windows start menu, search for `services`, and start the service `Scaphandre`.
Then run the following command to check if the service is running.

```bash
driverquery /v | grep -i scaph
```

### PowerMonitor for JoularJX-3.0.1

Download the latest PowerMonitor.exe from the [releases](https://github.com/joular/WinPowerMonitor/releases) page.
Edit the config.properties file and set the correct path to the PowerMonitor.exe file.
In case you want to test different Java or Spring Boot versions, edit your PATH variable and the spring-petclinic-rest/pom.xml file accordingly.

### Running the Tests

The following commands were tested in the **Git Bash (MINGW64)** shell on Windows, they don't run in the PowerShell.
The first command runs the PetClinic application with JoularJX attached.
Adapt the paths to the joularjx.jar, the config.properties, and the petclinic.jar
Be Aware that the application logs are not written to a file, thus leave the window open.

```bash
java -javaagent:/path/to/joularjx/target/*.jar \
-Djoularjx.config=/path/to/spring-petclinic-energy-benchmarking/config.properties \
-Dspring.sql.init.mode=never \
-Dspring.profiles.active=mysql,jpa \
-Dspring.datasource.username=root \
-Dspring.datasource.password=petclinic \
-Dspring.threads.virtual.enabled=false \
-jar /path/to/spring-petclinic-rest/target/*.jar
```

Make sure JMeter is added to the PATH or provide the path in the command, run the JMeter Tests in a separate Shell.

```bash
jmeter -n -t /path/to/spring-petclinic-energy-benchmarking/jmeter-petclinic-server.jmx \
-l /path/to/spring-petclinic-energy-benchmarking/out/$(date +%Y-%m-%d_%H-%M-%S)/log/$(date +%Y-%m-%d_%H-%M-%S)-jmeter.jtl \
-j /path/to/spring-petclinic-energy-benchmarking/out/$(date +%Y-%m-%d_%H-%M-%S)/log/$(date +%Y-%m-%d_%H-%M-%S)-jmeter.log
```

JMeter automatically terminates when the test run is completed.
Terminate the PetClinic application with `CTRL + C`, wait for the application to terminate.
Be Aware that the script for Linux copies the Output files to a specific Folder , this is not the case for Windows.
Copy the application log from the first shell and paste it in a separate log file to store it. 
Note that not all combinations of Spring Boot versions, JVM versions, Java versions, and web servers are supported.
