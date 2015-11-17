# presto-hands-on

Welcome! This repo contains instructions for setting up Presto on an Amazon EMR cluster, loading some data and playing around with some queries. Feel free to open issues of pull requests if something with the instructions is off.

# Pre-requisites

We assume that you have provisioned a 3 node EMR cluster that you have access to via SSH and that all nodes in the cluster have access to the Internet.

For the Boston Presto hands on meetup, check out this [gist](https://gist.github.com/petroav/1ef757d1673bc6a7436e) for getting credentials.

# Setting up the environment

1. Use the `ec2-user` user to log into your cluster because passwordless ssh for `root` has not been setup. Once logged in, to get a `root` shell you can just do `sudo bash`. Password credentials will be provided to you in person.
2. Upload `default.pem` key (who's contents are included in the gist linked above) to your cluster, preferrably in `/home/ec2-user/default.pem`.
3. We'll use Teradata's `presto-admin` CLI tool for the administrative tasks needed to setup our environment. You should download and set it up with the following instructions on the master node:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/prestoadmin-1.1-offline.tar.bz2
tar -jxf prestoadmin-1.2-offline.tar.bz2
cd prestoadmin/
sudo ./install-prestoadmin.sh
sudo ./presto-admin --help
```
4. Download JRE8u66 from Oracle on your master node:
```
wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u65-b17/jre-8u65-linux-x64.rpm
```
5.  Install the JRE using `presto-admin`:
```
chmod 400 /home/ec2-user/default.pem
sudo cp /home/ec2-user/default.pem /root/.ssh/id_rsa
cd /home/ec2-user/prestoadmin/
# presto-admin will ask you to provide the following: user name for ssh=ec2-user; port=22; *internal* IP of coordinator and workers
sudo ./presto-admin package install /home/ec2-user/prestoadmin/jre-8u65-linux-x64.rpm
java -version
```
If you make a mistake typing in the wrong IP addresses of your nodes when prompted by `presto-admin`, you can edit the settings in `/etc/opt/prestoadmin/config.json`.

# Setting up the data

6. Download the data in the `/mnt` directory by running the commands below. Remember to enter a root shell and then exit after you are done:
```
cd /mnt
sudo bash
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000000
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000001
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000002
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000003
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000004
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000005
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000006
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000007
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000008
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000009
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000010
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000011
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000012
wget --no-check-certificate https://think.big.academy.aws.s3.amazonaws.com/ontime/orc/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-000013
exit
```
7. Upload the newly downloaded data to HDFS:
```
cd /mnt
sudo -u hdfs hdfs dfs -mkdir /ontime
sudo -u hdfs hdfs dfs -copyFromLocal /mnt/b5eb0eae-c2a2-4c3e-850a-f6296905aee7-* /ontime
```
8. Create the `flights` table in Hive and point it to the data you uploaded to HDFS.
```
sudo -u hdfs hive
CREATE EXTERNAL TABLE flights (
    Year INT,
    Quarter INT,
    Month INT,
    DayofMonth INT,
    DayOfWeek INT,
    FlightDate STRING,
    UniqueCarrier STRING,
    AirlineID INT,
    Carrier STRING,
    TailNum STRING,
    FlightNum INT,
    OriginAirportID INT,
    OriginAirportSeqID INT,
    OriginCityMarketID INT,
    Origin STRING,
    OriginCityName STRING,
    OriginState STRING,
    OriginStateFips INT,
    OriginStateName STRING,
    OriginWac INT,
    DestAirportID INT,
    DestAirportSeqID INT,
    DestCityMarketID INT,
    Dest STRING,
    DestCityName STRING,
    DestState STRING,
    DestStateFips INT,
    DestStateName STRING,
    DestWac INT,
    CRSDepTime INT,
    DepTime INT,
    DepDelay INT,
    DepDelayMinutes INT,
    DepDel15 INT,
    DepartureDelayGroups INT,
    DepTimeBlk STRING,
    TaxiOut INT,
    WheelsOff INT,
    WheelsOn INT,
    TaxiIn INT,
    CRSArrTime INT,
    ArrTime INT,
    ArrDelay INT,
    ArrDelayMinutes INT,
    ArrDel15 INT,
    ArrivalDelayGroups INT,
    ArrTimeBlk STRING,
    Cancelled TINYINT,
    CancellationCode STRING,
    Diverted TINYINT,
    CRSElapsedTime INT,
    ActualElapsedTime INT,
    AirTime INT,
    Flights INT,
    Distance INT,
    DistanceGroup INT,
    CarrierDelay INT,
    WeatherDelay INT,
    NASDelay INT,
    SecurityDelay INT,
    LateAircraftDelay INT,
    FirstDepTime INT,
    TotalAddGTime INT,
    LongestAddGTime INT,
    DivAirportLandings INT,
    DivReachedDest INT,
    DivActualElapsedTime INT,
    DivArrDelay INT,
    DivDistance INT,
    Div1Airport STRING,
    Div1AirportID INT,
    Div1AirportSeqID INT,
    Div1WheelsOn INT,
    Div1TotalGTime INT,
    Div1LongestGTime INT,
    Div1WheelsOff INT,
    Div1TailNum STRING,
    Div2Airport STRING,
    Div2AirportID INT,
    Div2AirportSeqID INT,
    Div2WheelsOn INT,
    Div2TotalGTime INT,
    Div2LongestGTime INT,
    Div2WheelsOff INT,
    Div2TailNum STRING,
    Div3Airport STRING,
    Div3AirportID INT,
    Div3AirportSeqID INT,
    Div3WheelsOn INT,
    Div3TotalGTime INT,
    Div3LongestGTime INT,
    Div3WheelsOff INT,
    Div3TailNum STRING,
    Div4Airport STRING,
    Div4AirportID INT,
    Div4AirportSeqID INT,
    Div4WheelsOn INT,
    Div4TotalGTime INT,
    Div4LongestGTime INT,
    Div4WheelsOff INT,
    Div4TailNum STRING,
    Div5Airport STRING,
    Div5AirportID INT,
    Div5AirportSeqID INT,
    Div5WheelsOn INT,
    Div5TotalGTime INT,
    Div5LongestGTime INT,
    Div5WheelsOff INT,
    Div5TailNum STRING
)
STORED AS ORC
LOCATION '/ontime'
tblproperties ("orc.compress"="SNAPPY");

show tables;
select count(*) from flights;
```

# Install and configure Presto

9. Download and install the RPM:
```
cd /home/ec2-user/prestoadmin
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/presto-server-rpm-0.115t-1.x86_64.rpm
sudo ./presto-admin package install presto-server-rpm-0.115t-1.x86_64.rpm
rm -f presto-server-rpm-0.115t-1.x86_64.rpm
```
10. Get the Presto CLI:
```
wget https://s3-us-west-2.amazonaws.com/presto-admin/meetup-2015-11/presto-cli-0.115t-executable.jar
sudo mv presto-cli-0.115t-executable.jar /usr/lib/presto/bin/presto
chmod +x /usr/lib/presto/bin/presto
```
11. Configure Presto coordinator and workers:
```
sudo mkdir /etc/opt/prestoadmin/coordinator
sudo mkdir /etc/opt/prestoadmin/workers

sudo bash -c 'echo "
coordinator=true
node-scheduler.include-coordinator=false
discovery-server.enabled=true
discovery.uri=http://10.0.0.83:8081
http-server.http.port=8081
query.max-memory-per-node=1GB
query.max-memory=40GB
" > /etc/opt/prestoadmin/coordinator/config.properties'

sudo bash -c 'echo "
coordinator=false
discovery.uri=http://10.0.0.83:8081
http-server.http.port=8081
query.max-memory-per-node=1GB
query.max-memory=40GB
" > /etc/opt/prestoadmin/workers/config.properties'

sudo ./presto-admin configuration deploy
```
12. Configure Hive connector:
```
sudo mkdir /etc/opt/prestoadmin/connectors

sudo bash -c 'echo "
connector.name=hive-cdh5
hive.metastore.uri=thrift://10.0.0.83:9083
hive.allow-drop-table=true
hive.allow-rename-table=true
" > /etc/opt/prestoadmin/connectors/hive.properties'

sudo ./presto-admin connector add hive
```
13. Start Presto coordinator and workers and use the CLI:
```
sudo ./presto-admin server start
/usr/lib/presto/bin/presto --server localhost:8081 --catalog hive
```