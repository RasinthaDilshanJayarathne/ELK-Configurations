# Logstash-Config

input {
  file {
    path => ["/u01/CMS/EODFILE/eod_info.log"]
    start_position => "beginning"
    mode => "tail"
  }
  file {
    path => ["/u01/CMS/EODFILE/eod_error.log"]
    start_position => "beginning"
    mode => "tail"
  }
 }

filter { 
   grok { 
     match => {"message" =>"%{DATE:date} %{TIME:time} %{LOGLEVEL:log_level} %{WORD:log_type} \[%{DATA:kafka_msg}\] - \[%{DATA:file_line}.\] %{WORD:module_name} %{GREEDYDATA:msg}"}
   }
  
   mutate { 
     remove_field => ["@timestamp","@version","host","file","message","log","path","event","original"]
   }
}

output {
   stdout {} 
   elasticsearch { 
	hosts => ["https://192.168.1.230:9200"] 
	index => "eod" 
	user => "elastic" 
	password => "password" 
	ssl => true 
	cacert => "/etc/logstash/certs/http_ca.crt"
   }
}
