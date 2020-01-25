# Python - Lenses for Apache Kafka

Python library for managing [Lenses](http://www.landoop.com/kafka-lenses) REST and WS APIs.

![Tox Workflow Tests](https://github.com/lensesio/lenses-python/workflows/Tox%20Workflow%20Tests/badge.svg)

# Documentation

See [Lenses Python documentation](https://docs.lenses.io/dev/python-lib/).

### Authentication

There are three different ways that can be used for authentication.

| Authentication Type | Description                    |
|:------------------- |:-------------------------------|
|basic                | Basic (Accont) Authentication  |
|service              | Service Account (Token Based)  |
|krb5                 | Kerberos Authentication        |

#### Basic Auth

Parameters for the **basic_auth** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:---------------------------|:--------:|
|auth_type                   | Authentication Type        | Yes      |
|url                         | Lenses Endpoint            | Yes      |
|username                    | Username                   | Yes      |
|password                    | Password                   | Yes      |

For basic authentication, issue:

    from lensesio.lenses import main

    lenses_lib = main(
        auth_type="basic",
        url=lenses_endpoint,
        username=user,
        password=psk
    )

where `lenses_endpoint`, `user`, `psk` are python variables set by you with the endpoint, username and password

#### Kerberos Auth

Parameters for the **krb_auth** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:---------------------------|:--------:|
|auth_type                   | Authentication Type        | Yes      |
|url                         | Lenses Endpoint            | Yes      |
|krb_service                 | Service                    | Yes      |

**Note**: Kerberos support is only supported for linux platform and is not enabled by default.
To enable Kerberos support follow kerberos dependency step in the `Install` section at the end

        pip3 install dist/lensesio-3.0.0-py3-none-any.whl[kerberos]

For Kerberos authentcation, issue:

    from lensesio.lenses import main
    
    lenses_lib = main(
        auth_type="krb5", 
        url="http://localhost:3030", 
        krb_service="HTTP@primef.dev.local"
    ) 

#### Get User Info after authentication

To get the authenticated user info, issue:

    userInfo = lenses_lib.UserInfo()
    print(userInfo)
    {'permissions': ['ManageConnectors',
      'ViewDataPolicies',
      'ViewTopology',
      ...
     'security': {'http': False, 'kerberos': True, 'ldap': False},
     'token': '***',
     'user': '***'}

### Get Topics List

To get a list with all kafka topics, issue

    topicsList = lenses_lib.LstOfTopicsNames()
    
    print(topicsList)
    [
    'connect-configs',
    ...
    ]

### Get Topics Description

To get detailed description for all kafka topics, issue

    kafkaTopics = lenses_lib.GetAllTopics()
    
    print(kafkaTopics)
    [{'configs': 25,
      'consumers': 1,
      'isCompacted': False,
      ...
      'topicName': 'connect-configs',
     {'configs': 25,
      'consumers': 0,
      ...

### Detailed info for a Topic

To get a detailed description for a particular topic, issue

    topicInfo = lenses_lib.TopicInfo('connect-configs')
    
    print(topicInfo)
    {'applications': [],
     'config': [{'defaultValue': 'more than 1000y',
       'documentation': None,
       'isDefault': True,
       'name': 'message.timestamp.difference.max.ms',
       'originalValue': '9223372036854775807',
       'value': 'more than 1000y'},
      {'defaultValue': '1 MB',
       'documentation': None,
       'isDefault': True,
       'name': 'max.message.bytes',
       'originalValue': '1000012',
       'value': '1 MB'},
       ...

#### Create a Topic

Parameters for the **CreateTopic** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:---------------------------|:--------:|
|topicName                   | Topic's Name               | Yes      |
|partitions                  | Topic's partitions         | Yes      |
|replication                 | Topic's replication number | Yes      |
|config                      |Dict with Topic options     | No       |

To create a topic, first create a dictionary with the options below

        config = {
            "cleanup.policy": "compact",
            "compression.type": "snappy"
        }

The issue the CreateTopic method

        result = lenses_lib.CreateTopic(
            name="test_topic",
            partitions=1,
            replication=1,
            config=config
        )
        
        print(result)
        'Topic [test_topic] created'

#### Update Topic configuration

Parameters for the **UpdateTopicConfig** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:---------------------------|:--------:|
|topicName                   | Topic's Name               | Yes      |
|config                      | Dict with Topic options    | No       |

Create the configuration with the desired options

        config = {"configs": [{"key": "cleanup.policy", "value": "compact"}]}

Use the UpdateTopicConfig method to update the topic's configuration

        result = lenses_lib.UpdateTopicConfig('test_topic', config)
        
        print(result)
        'Topic [test_topic] updated config with [SetTopicConfiguration(...'

#### Publish data to a Topic

Parameters for the **Publish** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:-------------------------- |:--------:|
|topic                       | Topic's Name               | Yes      |
|key                         | Topic's key                | Yes      |
|value                       | Topic's value              | Yes      |

Publishing data to a topic is as easy as

        result = lenses_lib.Publish(
            "test_topic",
            "test_key",
            "{'value':1}"
        )
        
        print(result)
        {'content': None, 'correlationId': 1, 'type': 'SUCCESS'}

#### Delete records from a Topic

Parameters for the **DeleteTopicRecords** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:-------------------------- |:--------:|
|topic                       | Topic's Name               | Yes      |
|partition                   | Topic's partition          | Yes      |
|offset                      | Topic's offset             | Yes      |

Records can be deleted by providing a range of offsets

        result = lenses_lib.DeleteTopicRecords('test_topic', "0", "10")
        
        print(result)
        "Records from topic '%s' and partition '0' up to offset '10'" % 'test_topic'

#### Delete a Topic

Parameters for the **DeleteTopic** method

| Parameter Name             | Description                | Requried |
|:-------------------------- |:-------------------------- |:--------:|
|topicname                   | Topic's Name               | Yes      |


Delete a topic called `test_topic` by using the DeleteTopic method

    result = lenses_lib.DeleteTopic("test_topic")
    
    print(result)
    "Topic 'test_topic' has been marked for deletion"

#### List Kafka ACLs

Example of listing kafka acls. Here we have not set any acls, hence we get an empty list

    result = lenses_lib.GetAcl()
    
    print(result)
    []

#### Set Kafka ACLs

Parameters for the **SetAcl** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|resourceType                | https://kafka.apache.org/documentation/#security_authz_cli     | Yes      |
|resourceName                | -                                                              | Yes      |
|principal                   | -                                                              | Yes      |
|permissionType              | -                                                              | Yes      |
|host                        | -                                                              | Yes      |
|operation                   | -                                                              | Yes      |

    result = lenses_lib.SetAcl("Topic", "transactions", "GROUPA:UserA", "Allow", "*", "Read")
    
    print(result)
    'OK'

#### Delete Kafka ACLs

Parameters for the **DelAcl** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|resourceType                | https://kafka.apache.org/documentation/#security_authz_cli     | Yes      |
|resourceName                | -                                                              | Yes      |
|principal                   | -                                                              | Yes      |
|permissionType              | -                                                              | Yes      |
|host                        | -                                                              | Yes      |
|operation                   | -                                                              | Yes      |

To delete a Kafka ACL, issue:

    result = lenses_lib.DelAcl("Topic", "transactions", "GROUPA:UserA", "Allow", "*", "Read")
    
    print(result)
    'OK'

#### Managing Connect Distributed

Below we include examples for the methods used to manage kafka connect

##### Create a Connector

Parameters for the **CreateConnector** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|configs                     | Connector's config dict                                        | Yes      |

If you have configured Lenses with Connect, then you can use the `CreateConnector` method for creating a new connector.

First, create the configuration that describes the connector

        config = {
            "name": "test_connector",
            "config": {
                "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
                "tasks.max": "1",
                "topic": "test_connector_topic"
            }
        }

The create the connector by issuing:
Note: `dev` is the connect cluster's name.

    result = lenses_lib.CreateConnector('dev', config)

    print(result)
    {'name': 'test_connector',
     'config': {'connector.class': 'org.apache.kafka.connect.file.FileStreamSourceConnector',
      'tasks.max': '1',
      'topic': 'test_connector_topic',
      'name': 'test_connector'},
     'tasks': [{'connector': 'test_connector', 'task': 0}],
     'type': 'source'}

##### List all connectors

Parameters for the **GetConnectors** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |

    result = lenses_lib.GetConnectors('dev')

    print(result)
    ['test_connector', 'logs-broker', 'nullsink']

##### Get information about a connector

Parameters for the **GetConnectorInfo** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.GetConnectorInfo('dev', 'test_connector')

    print(result)
    {'name': 'test_connector',
     'config': {'connector.class': 'org.apache.kafka.connect.file.FileStreamSourceConnector',
      'tasks.max': '1',
      'name': 'test_connector',
      'topic': 'test_connector_topic'},
     'tasks': [{'connector': 'test_connector', 'task': 0}],
     'type': 'source'}

##### Get Connector's Configuration

Parameters for the **GetConnectorConfig** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.GetConnectorConfig('dev', 'test_connector')
    
    print(result)
    {'connector.class': 'org.apache.kafka.connect.file.FileStreamSourceConnector',
     'tasks.max': '1',
     'name': 'test_connector',
     'topic': 'test_connector_topic'}

##### Get Connector's Status

Parameters for the **GetConnectorStatus** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.GetConnectorStatus('dev', 'test_connector')

    print(result)
    {'name': 'test_connector',
     'connector': {'state': 'RUNNING', 'worker_id': '10.15.3.1:8083'},
     'tasks': [{'id': 0, 'state': 'RUNNING', 'worker_id': '10.15.3.1:8083'}],
     'type': 'source'}

##### Get Connector's Tasks

Parameters for the **GetConnectorTasks** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.GetConnectorTasks('dev', 'test_connector')
    
    print(result)
    [{'id': {'connector': 'test_connector', 'task': 0},
      'config': {'task.class': 'org.apache.kafka.connect.file.FileStreamSourceTask',
       'batch.size': '2000',
       'topic': 'test_connector_topic'}}]

##### Get Connector's Task Status

Parameters for the **GetStatusTask** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |
|task_id                     | Connector's task ID                                            | Yes      |

    result = lenses_lib.GetStatusTask('dev', 'test_connector', '0')
    
    print(result)
    {'id': 0, 'state': 'RUNNING', 'worker_id': '10.15.3.1:8083'}

##### Restart a Task

Parameters for the **RestartConnectorTask** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |
|task_id                     | Connector's task ID                                            | Yes      |

    result = lenses_lib.RestartConnectorTask('dev', 'test_connector', '0')

##### Get Connector's plugins

Parameters for the **GetConnectorPlugins** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |

    result = lenses_lib.GetConnectorPlugins('dev')
    
    print(result)
    [{'author': 'Lenses.io',
      'class': 'com.datamountaineer.streamreactor.connect.influx.InfluxSinkConnector',
      'description': 'Store Kafka data into InfluxDB',
      ...
      'version': '2.2.1-L0'},
     {'author': 'Apache Kafka',
      ...
      'version': '2.2.1-L0'}]

##### Pause a Connector

Parameters for the **PauseConnector** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.PauseConnector('dev', 'test_connector')

##### Resume a Connector

Parameters for the **ResumeConnector** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.ResumeConnector('dev', 'test_connector')

##### Restart a Connector

Parameters for the **RestartConnector** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.RestartConnector('dev', 'test_connector')

##### Update Connector's configuration

Parameters for the **SetConnectorConfig** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |
|configs                     | Connector's config dict                                        | Yes      |

        config = {
            "name": "test_connector",
            "connector.class": "org.apache.kafka.connect.file.FileStreamSourceConnector",
            "tasks.max": "5",
            "topic": "test_connector_topic"
        }

        result = lenses_lib.SetConnectorConfig('dev', 'test_connector', config)
        print(result)
        {
            "name":"test_connector",
            "config":{
                "name":"test_connector","connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector",
                "tasks.max":"5","topic":"test_connector_topic"
            },"tasks":[
                {
                    "connector":"test_connector","task":0
                }
            ],
            "type":"source"
    }

##### Delete a Connector

Parameters for the **DeleteConnector** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|cluster                     | Connect cluster                                                | Yes      |
|connector                   | Connector's name                                               | Yes      |

    result = lenses_lib.DeleteConnector('dev', 'test_connector')

##### Get all kafka Consumers

    result = lenses_lib.GetConsumers()
    
    print(result)
    [{'active': False,
      'application': None,
      'consumers': [],
      'consumersCount': 0,
      'coordinator': {'host': '-', 'id': -1, 'port': -1, 'rack': ''},
      'id': 'connect-fast-data',
      'maxLag': None,
      'minLag': None,
      'state': 'CoordinatorNotFound',
      ...
      'topicPartitionsCount': 8}]

##### List all Consumer names

    result = lenses_lib.GetConsumersNames()
    
    print(result)
    ['connect-fast-data',
     'lsql_dc343aa2b9a6441ab2f2e2143771abd7',
     'connect-nullsink',
     'schema-registry',
     'UNKNOWN']

#### SQL Engine

Examples for the methods used for running sql commands for the SQL Engine

#### Create a Topic via SQL Engine

Parameters for the **ExecSQL** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|Query                       | SQL Query                                                      | Yes      |

    query = (
        "CREATE TABLE greetings(_key string, _value string) FORMAT (string, string)"
    )

    result = lenses_lib.ExecSQL(query)
    print(result)
    {
        'data': [
            {
                'value': {
                    'flag': True,
                    'info': 'Topic greetings has been created'
                    ...
    ...
    }

##### Insert records into a Topic

    query = (
        "INSERT INTO greetings(_key, _value) VALUES('Hello', 'World')"
    )
    result = lenses_lib.ExecSQL(query)
    print(result)
    {
        'data': [{'value': {'flag': True, 'info': '1 records inserted'}, 'rownum': 0}], 
        ...
    }

##### Query a Topic

    query = (
        "SELECT * FROM greetings limit 1"
    )
    result = lenses_lib.ExecSQL(query)
    print(result)
    {
     'ERROR': [],
     'data': [{'key': 'Hello',
               'metadata': {'__keysize': 5,
                            '__valsize': 5,
                            'offset': 0,
                            'partition': 0,
                            'timestamp': 1579540609297},
               'rownum': 0,
               'value': 'World'}],
     'metadata': {'fields': ['offset',
                             'partition',
                             ...
    ...
    }


##### Delete a Topic via SQL

    query = (
        "DROP TABLE greetings"
    )
    result = lenses_lib.ExecSQL(query)
    print(result)
    {'ERROR': [],
     'data': [{'rownum': 0, 'value': True}],
     ...
    }

#### SQL Processors

Examples with methods used to manage SQL Processors

##### Create a Processor

Parameters for the **CreateProcessor** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Processors name                                                | Yes      |
|sql                         | SQL Query                                                      | Yes      |
|runners                     | SQL Processor's runners                                        | Yes      |
|clusterName                 | Cluster's name                                                 | Yes      |
|namespace                   | K8 Namespace                                                   | No       |
|pipeline                    | SQL Pipeline tag                                               | No       |


    query = (
        "SET autocreate=true; insert into test_processor_target SELECT * FROM test_processor_source"
    )

    result = lenses_lib.CreateProcessor("test_processor1", query, 1, 'dev', 'ns', '1')

    print(result)
    lsql_fa101b766ec04586b156a1d7f725f771

##### Pause a Processor

Parameters for the **PauseProcessor** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Processors name                                                | Yes      |

First get the processor's ID

    processor_id = lenses_lib.GetProcessorID('test_processor')
    print(processor_id)
    ['lsql_fa101b766ec04586b156a1d7f725f771']

Next use the `PauseProcessor` method to pause the processor

    result = lenses_lib.PauseProcessor(processor_id[0])
    print(result)
    'OK'

##### Resume a processor

Parameters for the **ResumeProcessor** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Processors name                                                | Yes      |

    processor_id = lenses_lib.GetProcessorID('test_processor')
    result = lenses_lib.ResumeProcessor(processor_id[0])

##### Update Processor's Runners

Parameters for the **UpdateProcessorRunners** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Processors name                                                | Yes      |
|runners                     | SQL Processor's runners                                        | Yes      |

    processor_id = lenses_lib.GetProcessorID('test_processor')
    lenses_lib.UpdateProcessorRunners(processor_id[0], '4')

##### Delete a Processor

Parameters for the **DeleteProcessor** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Processors name                                                | Yes      |

    processor_id = lenses_lib.GetProcessorID('test_processor')
    result = lenses_lib.DeleteProcessor(processor_id[0])

#### Data Flows

You can view the data flow in your cluster by using the `GetFlows` method
**Note**: With `GetFlows()` you can view the IDs of all consumers/producers along with any relations to others consumers/producers. Example: { source -> kafka -> sink }

    result = lenses_lib.GetFlows()
    
    print(result)
    {'dev:logs-broker': {'descendants': ['TOPIC-logs_broker'],
        'description': '\nName:logs-broker\nInstance:/var/log/broker.log',
        'label': 'dev:logs-broker',
        'parents': [],
        'type': 'SOURCE',
        'relations': {}},
     'dev:nullsink': {'descendants': [],
        'description': '\nName:nullsink\nInstance:/dev/null',
        'label': 'dev:nullsink',
        'parents': ['TOPIC-nyc_yellow_taxi_trip_data',
            'TOPIC-sea_vessel_position_reports',
            'TOPIC-telecom_italia_data'],
        'type': 'SINK',
        'relations': {}},
     'lsql_585e96e284804792be0875af0559da9e': {'descendants': ['TOPIC-fast_vessel_processor'],
        'description': 'SET autocreate=true;\n\nINSERT INTO fast_vessel_processor\n    SELECT MMSI, Speed, Longitude AS Long, Latitude AS Lat, `Timestamp`\n    FROM sea_vessel_position_reports\n    WHERE Speed > 10;',
        'label': 'filter_fast_vessels',
        'parents': ['TOPIC-sea_vessel_position_reports'],
        'type': 'PROCESSOR',
        'relations': {}}}


#### Data Policy

Examples with methos used to manage data policies in Lenses

##### Set a Policy

Parameters for the **SetPolicy** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Policy name                                                    | Yes      |
|obfuscation                 | Whether to protect messages at a field level                   | Yes      |
|impactType                  | The business impact levels in relation to the data             | Yes      |
|category                    | Category of sensitivity in the data                            | Yes      |
|fields                      | Definition of fields that the data policy will apply to        | No       |

    result = lenses_lib.SetPolicy("test_policy","All","HIGH","test_category",["test_field"])
    print(result)
    'c844ecd4-7cbd-4ec3-82f0-f3750a692efd'

##### View Policies

    policies = lenses_lib.ViewPolicy()
    
    print(policies)
    [{'category': 'test_category',
      'fields': ['test_field'],
      'id': 'c844ecd4-7cbd-4ec3-82f0-f3750a692efd',
      'impact': {'apps': [], 'connectors': [], 'processors': [], 'topics': []},
      'impactType': 'HIGH',
      'lastUpdated': '2020-01-20T17:27:49.368Z',
      'lastUpdatedUser': 'devops@DEV.LOCAL',
      'name': 'test_policy',
      'obfuscation': 'All',
      'versions': 0}]

##### Delete a Policy

Parameters for the **DelPolicy** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|name                        | Policy name                                                    | Yes      |

    lenses_lib.DelPolicy("test_policy")

#### Kafka Quotas

Examples with methods used to manage Kafka quotas

##### Get All Quotas

    lenses_lib.GetQuotas()

##### Set Quotas All Users

Parameters for the **SetQuotasAllUsers** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|config                      | Quota Configuration Dict                                       | Yes      |

    QUOTA_CONFIG = {
        "producer_byte_rate": "100000",
        "consumer_byte_rate": "200000",
        "request_percentage": "75"
    }

    lenses_lib.SetQuotasAllUsers(QUOTA_CONFIG)

##### Set User Quota for a Client

Parameters for the **SetQuotaUserClient** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|user                        | The user to set the quota for                                  | Yes      |
|clientid                    | The client id to set the quota for                             | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    lenses_lib.SetQuotaUserClient('admin', 'admin', QUOTA_CONFIG)

##### Set Quota for User

Parameters for the **SetQuotaUser** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|user                        | The user to set the quota for                                  | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    lenses_lib.SetQuotaUser("admin", QUOTA_CONFIG)

##### Set Quota for all Clients

Parameters for the **SetQuotaAllClient** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|config                      | Quota Configuration Dict                                       | Yes      |

    lenses_lib.SetQuotaAllClient(QUOTA_CONFIG)

##### Set Quota for a Client

Parameters for the **SetQuotaClient** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|clientid                    | The client id to set the quota for                             | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    lenses_lib.SetQuotaClient("admin", QUOTA_CONFIG)

##### Delete Quota for all Users

Parameters for the **DeleteQutaAllUsers** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQutaAllUsers(config)

##### Delete User Quota for all Clients

Parameters for the **DeleteQuotaUserAllClients** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|user                        | The user to delete the quota for                               | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQuotaUserAllClients("admin", config)

##### Delete User Quota for a Client

Parameters for the **DeleteQuotaUserClient** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|user                        | The user to delete the quota for                               | Yes      |
|clientid                    | The client id to delete the quota for                          | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQuotaUserClient("admin", "admin", config)

##### Delete Quota for User

Parameters for the **DeleteQuotaUser** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|user                        | The user to delete the quota for                               | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQuotaUser("admin", config)

##### Delete Quota for all Clients

Parameters for the **DeleteQuotaAllClients** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQuotaAllClients(config)

##### Delete Quota for a Client

Parameters for the **DeleteQuotaClient** method

| Parameter Name             | Description                                                    | Requried |
|:-------------------------- |:-------------------------------------------------------------- |:--------:|
|clientid                    | The client id to delete the quota for                          | Yes      |
|config                      | Quota Configuration Dict                                       | Yes      |

    config = ['consumer_byte_rate', 'producer_byte_rate', 'request_percentage']
    lenses_lib.DeleteQuotaClient('admin', config)

#### Lenses Admin

Examples with methods provided for managing the Lenses Admin interface

##### Create a Group

    group_payload = {
        "name":"test_group",
        "description":"test_description",
        "scopedPermissions":[
            "ViewKafkaConsumers",
            "ManageKafkaConsumers",
            "ViewConnectors",
            "ManageConnectors",
            "ViewSQLProcessors",
            "ManageSQLProcessors",
            "ViewSchemaRegistry",
            "ManageSchemaRegistry",
            "ViewTopology",
            "ManageTopology"
        ],
        "adminPermissions":[
            "ViewDataPolicies",
            "ManageDataPolicies",
            "ViewAuditLogs",
            "ViewUsers",
            "ManageUsers",
            "ViewAlertRules",
            "ManageAlertRules",
            "ViewKafkaSettings",
            "ManageKafkaSettings",
            "ViewLogs"
        ],
        "namespaces":[
            {
                "wildcards":["*"],
                "permissions":[
                    "CreateTopic",
                    "DropTopic",
                    "ConfigureTopic",
                    "QueryTopic",
                    "ShowTopic",
                    "ViewSchema",
                    "InsertData",
                    "DeleteData",
                    "UpdateSchema"
                ],"system":"Kafka","instance":"Dev"
            }
        ]
    }

    lenses_lib.CreateGroup(group_payload)

##### View Groups

    result = lenses_lib.GetGroups()
    
    print(result)
    [{'name': 'devops',
      'description': None,
      'namespaces': [{'wildcards': ['*'],
        'permissions': ['CreateTopic',
        ...
    ...
    ]

##### Update a Group

    group_payload = {
        "name":"test_group","description":"test_description_updated","namespaces":[
            {
                "wildcards":["*"],
                "permissions":[
                    "CreateTopic","DropTopic","ConfigureTopic","QueryTopic","ShowTopic",
                    "ViewSchema","InsertData","DeleteData","UpdateSchema"
                ],
                "system":"Kafka","instance":"Dev"
            }
        ],
        "scopedPermissions":[
            "ViewKafkaConsumers","ManageKafkaConsumers","ViewConnectors","ManageConnectors",
            "ViewSQLProcessors","ManageSQLProcessors","ViewSchemaRegistry","ManageSchemaRegistry",
            "ViewTopology","ManageTopology"
        ],
        "adminPermissions":[
            "ViewDataPolicies","ManageDataPolicies","ViewAuditLogs","ViewUsers","ManageUsers",
            "ViewAlertRules","ManageAlertRules","ViewKafkaSettings","ManageKafkaSettings",
            "ViewLogs"
        ],
        "userAccounts":0,"serviceAccounts":0
    }

    result = lenses_lib.UpdateGroup("test_group", group_payload)

##### Delete a Group

    lenses_lib.DeleteGroup("test_group")

##### Create a User

Note you must create a group prior to creating a user

    lenses_lib.CreateUser(
        acType="BASIC", 
        username="test_user", 
        password="test_user", 
        email=None, 
        groups="test_group_user"
    )

##### Get all Users

    result = lenses_lib.GetUsers()
    
    print(result)
    [{'username': 'test_user',
      'email': None,
      'groups': ['test_group_user'],
      'isActive': False,
      'type': 'BASIC'},
     {'username': 'devops@DEV.LOCAL',
      'email': None,
      'groups': ['devops'],
      'isActive': True,
      'type': 'KERBEROS'}]

##### Update a User

    result = lenses_lib.UpdateUser(
        acType="BASIC", 
        username="test_user", 
        password="test_user", 
        email="test_updated@localhost.localdomain", 
        groups="test_group_user"
    )

##### Change User's Password

    result = lenses_lib.UpdateUserPassword(
        username="test_user",
        password="test_user_updated"
    )

##### Delete a User

    result = lenses_lib.DeleteUser(
        username="test_user",
    )

##### Create a Service Account
Note you must first create a group and a user before creating a service account

    result = lenses_lib.CreateSA(
        name="test_sa",
        groups="test_group_user",
        owner="test_user",
        token="test_sa_token"
    )

##### Update a Service Account

    result = lenses_lib.UpdateSA(
        name="test_sa",
        groups=["test_group_sa", "test_group_user"],
        owner="test_user",
    )

##### Add a new Token to a Service Account
Note adding a new token, automatically expires the old one

    result = lenses_lib.UpdateSAToken(
        name="test_sa",
        token="test_sa_token_updated",
    )

##### Add a new Random Token to a Service Account
Note adding a new token, automatically expires the old one

    result = lenses_lib.UpdateSAToken(name="test_sa",)

##### Delete a Service Account

    result = lenses_lib.DeleteSA(name="test_sa")

##### Get Lenses Configuration

    result = lenses_lib.GetConfig()

    print(result)
    {'lenses.ip': '0.0.0.0',
     'lenses.jmx.port': 9586,
     ...
     'lenses.zookeeper.hosts': [{'jmx': '0.0.0.0:9585', 'url': '0.0.0.0:2181'}]}

##### Get Lenses Audits

    result = lenses_lib.Audits()
    
    print(result[0])
    {'action': 'ADD',
     'content': {'name': 'test_user',
      'groups': 'test_group_user',
      'password': '*****',
      'type': 'BASIC'},
     'resourceId': 'test_user',
     'resourceName': None,
     'timestamp': 1579543219695,
     'type': 'USER_MANAGEMENT_USER',
     'user': 'devops@DEV.LOCAL'}

##### Get Lenses Alerts

    result = lenses_lib.Alerts() 
    
    print(result[0])
    {'alertId': 4001,
     'category': 'Topics',
     'docs': None,
     'instance': 'test_topic',
     'level': 'INFO',
     'map': {'topic': 'test_topic'},
     'summary': "Topic 'test_topic' has been deleted by admin",
     'tags': [],
     'timestamp': 1579542199932}

##### Get Lenses Logs (Java Logs)

    result = lenses_lib.GetLogs()
    
    print(result)
    [
    ...
     {'level': 'INFO',
      'logger': 'akka.actor.ActorSystemImpl',
      'message': 'Request: GET->http://localhost:9991/api/audit?pageSize=999999999 '
                 'returned 200 OK in 43ms',
      'stacktrace': '',
      'thread': 'default-akka.actor.default-dispatcher-137',
      'time': '2020-01-20 18:11:02.924',
      'timestamp': 1579543862924},
     {'level': 'INFO',
      'logger': 'akka.actor.ActorSystemImpl',
      'message': 'Request: GET->http://localhost:9991/api/audit?pageSize=999999999 '
                 'returned 200 OK in 12ms',
      'stacktrace': '',
      'thread': 'default-akka.actor.default-dispatcher-2',
      'time': '2020-01-20 18:11:07.800',
      'timestamp': 1579543867800},
     {'level': 'INFO',
      'logger': 'akka.actor.ActorSystemImpl',
      'message': 'Request: '
                 'GET->http://localhost:9991/api/alerts?pageSize=999999999 '
                 'returned 200 OK in 15ms',
      'stacktrace': '',
      'thread': 'default-akka.actor.default-dispatcher-13',
      'time': '2020-01-20 18:12:06.479',
      'timestamp': 1579543926479},
      ...
    ]

#### Registry

Examples with methods used to manage schemas.

##### Get all Schemas

    result = lenses_lib.GetAllSubjects()
    ['fast_vessel_processor-value',
     'telecom_italia_data-key',
     ...
     'logs_broker-value']

##### Register a new Schema

    SCHEMA_CONFIG = {
        'schema':
            '{"type":"record","name":"reddit_post_key",'
            '"namespace":"com.landoop.social.reddit.post.key",'
            '"fields":[{"name":"testit_id","type":"string"}]}'
    }
    COMPATIBILITY_CONFIG = {'compatibility': 'BACKWARD'}
    COMPATIBILITY_CONFIG_UPDATE = {'compatibility': 'FULL'}

    result = lenses_lib.RegisterNewSchema('test_schema', SCHEMA_CONFIG).keys()

##### List Versions of a Schema

    result = lenses_lib.ListVersionsSubj('test_schema')
    
    print(result)
    [1]

##### Get Schema by ID

    lenses_lib.GetSchemaById(schema_id)

##### Get Global Compatibility

    result = lenses_lib.GetGlobalCompatibility()

##### Update Global Compatibility
Note see under register new schema for the `COMPATIBILITY_CONFIG_UPDATE`

    lenses_lib.UpdateGlobalCompatibility(COMPATIBILITY_CONFIG_UPDATE)

##### Change Compatibility of a Schema

    lenses_lib.ChangeCompatibility('test_schema', COMPATIBILITY_CONFIG_UPDATE)

##### Get Compatibility of a Schema

    lenses_lib.GetCompatibility('test_schema')

##### Update a Schema (If compatible)

    SCHEMA_CONFIG_UPDATE = {
        'schema':
            '{"type":"record","name":"reddit_post_key",'
            '"namespace":"com.landoop.social.reddit.post.key",'
            '"fields":[{"name":"testit_id","type":"string","doc":"desc."}]}'
    }
    lenses_lib.UpdateSchema('test_schema', SCHEMA_CONFIG_UPDATE)

##### Get a certain version of a Schema

    lenses_lib.GetSchemaByVer('test_schema', subj_ver)

##### Delete a Schema version

    lenses_lib.DeleteSchemaByVersion("test_schema", subj_ver)

##### Delete a Schema (all versions)

    lenses_lib.DeleteSubj("test_schema")

# Use Cases and Examples

* CI/CD and Automation
* Jupyter Notebooks
* Machine Learning

## Jupyter Example

<p align="center">
  <img src="https://pbs.twimg.com/media/DbeXsAZXcAAw8uy.jpg" width="400"/>
</p>

# Installation

### Dependencies

Install Pip3:

    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

    python3.8 get-pip.py
    rm -f get-pip.py

##### Virtual Environment (Recommended)

Install virtualenv and virtualenvwrapper:

    pip3.8 install virtualenv virtualenvwrapper

    cat >> ~/.bashrc <<EOF
    export WORKON_HOME=$HOME/VirtEnv/.virtualenvs
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3.8
    export VIRTUALENVWRAPPER_VIRTUALENV_ARGS=' -p /usr/bin/python3.8'
    export PROJECT_HOME=$HOME/VirtEnv
    source /usr/bin/virtualenvwrapper.sh
    EOF

    source ~/.bashrc

All virtualenvs and their packages will be installed under ``~/VirtEnv/.virtualenvs``

    ls ~/VirtEnv/.virtualenvs
    get_env_details  initialize  machinelab  postactivate ...

###### Create a python virtual environment:

    mkvirtualenv myvirtenv

Activate the virtualenv (activated by default after creation):

    # To activate just run workon myvirtenv
    [user@hostname]$ workon myvirtenv
    (myvirtenv)[user@hostname]$

##### Exiting virtualenv:

    (myvirtenv)[user@hostname]$ deactivate
    [user@hostname]$

##### Remove installed virtualenv along with the relevant packages:

    rmvirtualenv myvirtenv

##### Build lensesio lib

    python3 setup.py sdist bdist_wheel

##### Install lensesio lib

You can install by using pip

    pip3 install dist/lensesio-3.0.0-py3-none-any.whl
    
for kerberos support, issue

    pip3 install dist/lensesio-3.0.0-py3-none-any.whl[kerberos]

## Integration Tests

#### Requirements

**Will be handled automatically**

| Automatically installed |
| ------ |
| tox |
| pytest |
| flake8 |

**Must be installed manually**

| Must be Present |
| ------ |
| Docker |
| Virtualenv |

**Storage Requirements**

| Type | Storage |
| ------ | ------ |
| Lenses Box | 1.53 G |
| Virtual 3.8 Env | 139 M |

**Memory Requirements**

Integration tests will run Lenses-Box, which requires ~4G of memory.

#### Create and Activate Virtualenv


```
virtualenv --python=python3 virtenv
source virtenv/bin/activate
```

#### Start Lenses Box

```
make docker
```

#### Run tests

```
make LICENSE_KEY=<YOUR-LICENSE-KEY> docker
make test
```

This command will start a Lenses docker box and when it's ready the integration tests will run.

*Note*: You can find a dev license key [here](https://www.lenses.io/downloads/)

## License

The project is licensed under the Apache 2 license.
