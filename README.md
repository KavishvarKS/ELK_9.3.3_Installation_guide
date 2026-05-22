CHANGES TO BE DONE IN ADDITION TO THE DOCUMENT IS;
IN "nano /etc/kibana/kibana.yml"


#server.port: 5601
#server.host: 0.0.0.0

#elasticsearch.hosts: ["https://151.185.42.86:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "ejxtp=YDwUdJ=W9Ytd7X"

elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/http_ca.crt"]
elasticsearch.ssl.verificationMode: certificate

