Logstash requires Java 8. Java 9 is not supported
To check your Java version, run the following command:
#### java -version
On systems with Java installed, this command produces output similar to the following:
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)

### Steps to install Logstash:
1.Add GPG key of ELK's repository,download and install the Public Signing Key:
##### wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

2.Save the repository definition to /etc/apt/sources.list.d/elastic-5.x.list:
#### echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list

3.Run sudo apt-get update and the repository is ready for use
#### sudo apt-get update

4. Install Logstash 5.6.16 version
#### wget https://artifacts.elastic.co/downloads/logstash/logstash-5.6.16.deb
#### sudo dpkg -i logstash-5.6.16.deb

5.Configure logstash as necessary
#### sudo vi /etc/logstash/logstash.yml

6.Configure logstash to automatically start during startup
#### sudo systemctl enable logstash

7.Start logstash service
#### sudo systemctl start logstash


## create logstash config file : eg- sold cars data ingestion example

input {
    file {
        path => "/home/anderson/data/cars.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
    }
}
filter {
    csv {

        separator => ","

        columns => [ "maker", "model", "mileage", "manufacture_year", "engine_displacement",
        "engine_power", "body_type", "color_slug", "stk_year", "transmission", "door_count",
        "seat_count", "fuel_type", "date_created", "date_last_seen", "price_eur" ]
    }
    mutate {convert => ["mileage", "integer"] }
    mutate {convert => ["price_eur", "float"] }
    mutate {convert => ["engine_power", "integer"] }
    mutate {convert => ["door_count", "integer"] }
    mutate {convert => ["seat_count", "integer"] }

}
output {
    elasticsearch {
        hosts => "localhost:9200"
        index => "cars"
        document_type => "sold_cars"
     }
    stdout {}
}

### Ingestion steps:
Run command in bin directory of logstash - bin/logstash -f /path/to/configfile/file.conf 
