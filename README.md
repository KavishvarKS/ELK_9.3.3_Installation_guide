#CHANGES TO BE DONE IN ADDITION TO THE DOCUMENT IS;
IN "nano /etc/kibana/kibana.yml"


#server.port: 5601
#server.host: 0.0.0.0

#elasticsearch.hosts: ["https://151.185.42.86:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "ejxtp=YDwUdJ=W9Ytd7X"

elasticsearch.ssl.certificateAuthorities: ["/etc/kibana/certs/http_ca.crt"]
elasticsearch.ssl.verificationMode: certificate



#Fixing "Unable to initialize Fleet" Error

Step 1: Generate a secure 32+ character key
    bashopenssl rand -hex 32

Step 2: Add it to your kibana.yml config file
     yamlxpack.encryptedSavedObjects.encryptionKey: "your-generated-32-char-key-here"

Also recommended to add these while you're at it:
    yamlxpack.security.encryptionKey: "another-32-char-key-here"
    xpack.reporting.encryptionKey: "yet-another-32-char-key-here"

Step 3: Restart Kibana
    sudo systemctl restart kibana

WAIT FOR SOMETIME THE URL WILL  LOAD FOR  A BIT AND THEN THE KIBANA DAASHBOAD WILL BE ACCESSABLE AND YOU CAN ADD FLEET AGENTS.
