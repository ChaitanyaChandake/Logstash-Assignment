# Logstash-Assignment

Logstash pipeline to process syslog

For this project, I have designed a Logstash pipeline to convert unstructured syslog messages into a structured JSON format, ensuring that important fields are captured in a structured and easily accessible way. The pipeline leverages Filebeat for log collection and Logstash for parsing, transforming, and outputting the processed data.

**Test Input log**

><14>1 2016-12-25T09:03:52.754646-06:00 acmeinchost1 antivirus 2496 - - alertname="Virus Found" computername="acmeincpc42" computerip="10.58.194.149" severity="1"

**Configure Filebeat**

Filebeat is used to forward the logs collected from a file to Logstash on port 5044. I created custom YAML configuration file(filebeat_test.yml) to ensure new logs are read from input file and forwarded to Logstash for processing.

**Configure Logstash**

Created custom configuration file in config folder of Logstash(config\logstash-sample.conf). This configuration file includes input, filter and output plugins.

**Input Plugin**: The beats input plugin is configured to listen on port 5044. Logstash collects logs sent by filebeat on that port.

**Filter plugin**: After recieveing the logs from filebeat, I have used GROK filter plugin in Logstash to process the incoming logs and extract key meta fields. Steps involve:

1. Using GROK filter to parse and extract structured data from input logs.
   Note: I have handled alertname separately with the GROK filter before applying the KV filter. This is done to ensure proper handling of multi-word or space-separated values that may appear in the alertname 
   field.
3. Using KV(Key-Value) filter to process second part of the log which contains key-value pairs.The KV filter is configured to split the key value pairs based on 
   defined field_split and value_split.This ensures that even if new key value pairs appear in future logs, they are captured dynamically with a prefix(dynamic_) 
   e.g.,dynamic_newkey => value
4. Translated severity from numeric value to a label and captured the label in severity meta key using Translate filter.
5. Mutate filter is used to add important fields that have different intial name before processing and captured them in desired meta names post processing 
   (description,source_ip,computername). Mutate is also used to remove fields that are not needed in final output. These could be added back if needed.
   
**Handled Special Cases**

1. In certain logs, the fields can contain values instead of just hyphens after process id. This configuration parses these fields and captures the corresponding 
   values in msgid and context. These fields can be retained or removed based on the requirements.
2. For logs that contain no key-value pairs at all, the filter plugin is still able to parse the header and capture important fields like timestamp, hostname, 
   application etc.
3. The pipeline is designed to handle new key-value pairs that may be added to logs in the future. As new fields are introduced, they are captured dynamically with 
   the dynamic_ prefix. If any new keys are unnecessary, they can easily be removed using the Mutate plugin.

**Output Plugin**:Finally, I have configured the output plugin to write the processed logs to a file in JSON format using the json_lines codec.

**Steps followed to execute pipeline**

Verifying if configuration for filebeat is correct and properly formatted

>filebeat test config filebeat_test.yml

Running Filebeat

>filebeat -e -c filebeat_text.yml

Logstash configuration test

>bin\logstash.bat -f config\logstash-sample.conf --config.test_and_exit

Running Logstash

>bin\logstash.bat -f config\logstash-sample.conf

**Output JSON log**
```json
{"source_ip":"10.58.194.149","hostname":"acmeinchost1","severity":"critical","timestamp":"2016-12-25T09:03:52.754646-06:00","process_id":"2496","description":"Virus Found","application":"antivirus","computername":"acmeincpc42"}
```
**File Details in the Project** (Refer to Main Directory Folder <mark>Logstash-Solution</mark>)

**Path: Filebeat\filebeat_test.yml**

This file contains configuration for filebeat.

**Path: Logstash_config\logstash-sample.conf**

This file contains configuration file for logstash.

**Path: Test_data\Input_Logs\Added_logs_test**

This file contains input test logs.

**Path: Test_data\Output_Logs\output_file_logs**

This file contains the output generated after Added_logs_test input file is processed.

**Output_console.png**

Screenshot of Logstash console output(Here I have used stdout in output plugin to generate output on console)

**commandline_output.txt** 

Execution output of Logstash during log processing.
