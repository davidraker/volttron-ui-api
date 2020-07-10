# VUI

----

The VOLTTRON User Interface API.

----

----

----

----

## Authentication



The authentication endpoints are provided by the VOLTTRON master web service.  

* Authentication is required before using other API endpoints, which must be accessed using the JWT bearer token provided by the `POST /authenticate` endpoint.

* The token will expire afer some time, and to avoid loss of any session which is established by other endpoints, the session must be renewed using the `PUT /authenticate` endpoint.

* Sessions may be manually ended by calling the `DEL /authenticate` endpoint.

----

#### POST /authenticate

Built-in authorization endpoint for VOLTTRON web subsystem.
Returns JWT bearer token.

**Request:**
* Content Type: `application/json`
* Body:
    ```
    {
        "username": "<username>",
        "password": "<password>"
    }
    ```

**Response:**
* With valid username and password: `200 OK`
    * Content Type: `text/plain`
    * Body:
        ```
        <jwt_token>
        ```

* With invalid username and password: `401 Unauthorized`

----

#### PUT /authenticate

Renew authorization token.

User provides credentials again, with current token to renew the same session.

Returns new JWT bearer token. All subsequent requests should include the new token. The old bearer token is no longer considered valid.

**Request:**
* Content Type: `application/json`
* Authorization: `BEARER <jwt_token>`
* Body:
    ```
    {
        "username": "<username>",
        "password": "<password>"
    }
    ```

**Response:**
* With valid username and password: `200 OK`
    - Content Type: `text/plain`
    - Body:
        ```
        <new_jwt_token>
        ```

* With invalid or mismatched username, password, or token: `401 Unauthorized`

----

#### DELETE /authenticate

Log out of API. Bearer token will be invalidated, and any session will data be cleared.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid username and password: `200 OK`
* With invalid token: `401 Unauthorized`

----

----

----

----

## Platforms



Platforms endpoints expose functionality associated with specific VOLTTRON platforms.

As all functionality of VOLTTRON is the purvue of one or another platform, the /platforms tree forms the core of the VOLTTRON User Interface API. Other top level partitions of the API consist of convenience methods which refer to endpoints within /platforms.

* All endpoints in this tree require authentication using a JWT bearer token provided by the `POST /authenticate` or `PUT /authenticate` endpoints.

----

----

----

### Platforms PubSub



PubSub endpoints expose functionality associated with publication and subscription to topics on the VOLTTRON message bus.

* All PubSub endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/pubsub

Retrieve routes for message bus topics being monitored by this user of the VUI API.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Body:
        ```
            [
                "/platform/:platform/pubsub/:topic",
                "/platform/:platform/pubsub/:topic",
                ...
            ]
        ```

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/pubsub/:topic

Return the subscription to the topic.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token if subscription exists: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
        "topic": "<topic">,
        "push_bind": <bind_object> or null,
        "last_value": <last_value>,
        "number_of_subscribers": <number_of_open_subscriptions_to_topic>
        }       
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        {
            "error": "<Error Message>"
        }
- With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/pubsub/:topic

Create the specified subscription. Returns details. For publishing to the topic, see the `PUT /platforms/:platform/pubsub/:topic` endpoint.

**Request:**
* Content Type: `application/json`
* Authorization: `BEARER <jwt_token>`
* Body:
    ```
    {
        "push_bind": <bind_object> or null
    }
    ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Content Type: `application/json`
    * Location: `<resource_location>`
    * Body:
        ```
        {
		    "topic": "<topic>",
            "push_bind": <bind_obaject> or null,
            "last_value": "<last_value>",
            "access_token": <JWT_access_token_for_resource>
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/pubsub/:topic

Unsubscribe to the topic.

NOTE: If multiple subscriptions are open to the same topic, the server should remove this subscriber but keep the subscription resource open.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/pubsub/:topic

Publish to the specified topic on the specified platform and return confirmation details.

The value given in the request body must contain the intended publish body. This may be a single value or dictionary as expected by subscribers to the topic. The publish_type will be used in formatting the publish before it reaches the message bus. If a dictionary is provided for the value and no publish_type is given, the publish will be treated as a record type.

**Request:**
* Content Type: `application/json`
* Authorization: `BEARER <jwt_token>`
* Body:
    ```
    {
        "headers": {<message_bus_headers>},
        "publish_type": "<datalogger|device|analysis|record>"
        "value": <value>
    }
    ```

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "number_of_subsribers": <number_of_subsribers>
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Agents



Agents endpoints expose functionality associted with applications running on a VOLTTRON platform.

* All Agent endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

----

### Platforms Agents RPC



RPC endpoints expose functionality associted with remote procedure calls to agents running on a VOLTTRON platform.

* All RPC endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/agents/:vip_identity/rpc

Get available remote procedure call endpoints for the specified agent.

Success will yield a dictionary with available RPC methods as keys and routes for these as values.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<function_name>": "/platforms/:platform/agents/:vip_identity"/rpc/:function_name",
            "<function_name>": "/platforms/:platform/agents/:vip_identity"/rpc/:function_name",
             ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/agents/:vip_identity/rpc/:function_name

Get parameters for an remote procedure call method.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "param1": <type>,
            "param2": <type>
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/agents/:vip_identity/rpc/:function_name

Send an remote procedure  call to an agent running on a VOLTTRON platform.

The return value of an RPC call is defined by the agent, so this may be a scalar value or another JSON object, for instance a list, dictionary, etc.

**Request:**
* Content Type: `application/json`
* Authorization: `BEARER <jwt_token>`
* Body:
    ```
    {
        "<param_name": <value>,
        "<param_name": <value>,
         ...
    }
    ```

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "return": <value>
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Configs



Agent Configs endpoints expose functionality associted with configuration of agents running on a VOLTTRON platform.

* All Configs endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/agents/:vip_identity/configs

Get routes to available configuration files for the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            ":config_file_name": "/platforms/:platform/agents/:vip_identity"/configs/:config_file_name",
            "<config_file_name>": "/platforms/:platform/agents/:vip_identity"/configs/:config_file_name",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/agents/:vip_identity/configs

Save a new configuration file to the config store.

The file name should be passed in the query parameter `file-name`.

The file should match the content type and contents which the VOLTTRON configuration store expects. The configuration store currently accepts only JSON, CSV, or RAW files. The content type header should match the type of file being sent. Optionally, this endpoint may return `409 Conflict` if the configuration file already exists.  In this case, the user should use `PUT /platforms/:platform/agents/:vip_identity/configs/:file_name` if modification of the existing file is truly intended.

**Request:**
* Content Type: `application/json` or `text/csv` or `text/plain`
* Authorization: `BEARER <jwt_token>`
* Body (shown for JSON conntent):
    ```
    {
        "<setting_name>": <value>,
        "<setting_name>": <value>,
        ...
    }
    ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/agents/:vip_identity/configs/:file_name`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`, `409 Conflict`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/agents/:vip_identity/configs/:file_name

Get a configuration file for the agent from the config store.

The configuration store can currently return JSON, CSV, or RAW files.  If the Accept header is not set, the configuration store will return JSON by default. If the client wishes to restrict the type of file received, it should set the Accept header to the correct MIME type(s).

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Accept: `application/json`, `text/csv`, `text/plain`
    * Body:
        ```
        {
          "<setting_name>": <value>,
          "<setting_name>": <value>,
          ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/agents/:vip_identity/configs/:file_name

Overwrite a configuration file already in the config store.

The file should match the content type and contents which the VOLTTRON configuration store expects. The configuration store currently accepts only JSON, CSV, or RAW files. The content type header should match the type of file being sent.

**Request:**
* Content Type: `application/json` or `text/csv` or `text/plain`
* Authorization: `BEARER <jwt_token>`
* Body:
    ```
    {
        "<setting_name>": <value>,
        "<setting_name>": <value>,
        ...
    }
    ```

**Response:**
* With valid BEARER token on success: `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity/configs/:file_name

Remove an existing configuration file for the agent from the config store.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: 

* With valid BEARER token on failure: `204 No Content`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Frontends



Agent Frontend endpoints expose control panels provided by agents running on a VOLTTRON platform.

* All Frontends endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/agents/:vip_identity/frontends/

Get available front-end applications made available by an agent running on the VOLTTRON platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK` `201 Created` `204 No Content`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<frontend_name>": "/platforms/:platform/agents/:vip_identity/frontends/:frontend_name",
            "<frontend_name>": "/platforms/:platform/agents/:vip_identity/frontends/:frontend_name",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/agents/:vip_identity/frontends/:front_end_name

Load a front-end applications made available by an agent running on the VOLTTRON platform.

The application must have been created by the developer of the application, but possible use cases include configuration front-ends and control panels.  Front-end applications can define addtional routes as necessary within the namespace /platforms/:platform/agents/:vip_identity/frontends/:frontend_name/....  Applications which require interactivity may also open websockets or link to other resources. All applications should provide a schema to load the user interface for the application, but may also provide other resources, for instance an access_token, in this response as needed in the app_data key. If a websocket is opened, the response should use `201 Created` and the URI should be made available in the `Location` header.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK` `201 Created` `204 No Content`
    * Content Type: `application/json`
    * Location: `<resource_location>`
    * Body:
        ```
        {
            "ui_schema": {...},
            "app_data": {...}
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents PubSub



PubSub retrieve informaton about publish and subscription topics used by an agent running on a VOLTTRON platform.

* All PubSub endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/agents/:vip_identity/pubsub

Retrieve routes for message bus topics monitored by and to which are published by an agent running on a VOLTTRON platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Body:
        ```
        {
            "subscribes: [
                         "/platform/:platform/pubsub/:topic",
                         "/platform/:platform/pubsub/:topic",
                         ...
                         ],
            "publishes: [
                         "/platform/:platform/pubsub/:topic",
                         "/platform/:platform/pubsub/:topic",
                         ...   
                        ]
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Status



Agent Status endpoints expose staus information provided for agents by a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.


----

#### GET /platforms/:platform/agents/:vip_identity/status

Get status for the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "name": "<agent_name>",
            "priority": "<priority>",
            "tag": "<tag>",
            "uuid": "<uuid>",
            "running": "<running_status>",
            "enabled": "<enabled_status>"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity/status

Clear status for the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Health



Agent Health endpoints expose health information regarding agents running on a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/agents/:vip_identity/health

Get health for the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": {<agent_health>}
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Running



Agent Running endpoints expose functionality related to starting and stopping agent execution.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/agents/:vip_identity/running

Retrieve the running status of the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": true|false
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/agents/:vip_identity/running

Start the specified agent.

Accepts the query parameter `restart` to restart an agent. If the agent is already running, an error will be returned if the `restart` parameter is not "true".

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success:  `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity/running

Stop the specified agent.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/agents/-/running

Start an agent from the remote platform from a path on the filesystem.

The placeholder `-` is used for the `:vip_identity`. The request body is used to dictate the path on the remote filesystem from which the agent should be run, as well as the vip-identity and tag to use.

The response code will be `201 Created` and provide a `Location` header with the route to the agent.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
    ```
    {
        "run_from_path": "<agent_path>",
        "conig_path": "<config_path>"
        "vip_identity": "<tag>",
    }
    ```

**Response:**
* With valid BEARER token on success:  `201 Created`
    * Content Type: `application/json`
    * Location: `/platforms/:platform/agents/:vip_identity`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Enabled



Agent Enabled endpoints expose functionality regarding the automatic startup of agents running on a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/agents/:vip_identity/enabled

Retrieve the enabled status of the specified agent. If true, the agent will start with the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": true|false
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/agents/:vip_identity/enabled

Enable the specified agent to run on platform start-up.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity/enabled

Disable the specified agent from running on platform start-up.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

### Platforms Agents Tag



Agent Tag endpoints expose functionality regarding tags used to reference agents running on a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/agents/:vip_identity/tag

Retrieve the tag of an agent installed on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "tag": "<tag>"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/agents/:vip_identity/tag

Set the tag to an agent installed on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
	{
        "tag": "<tag>"
	}
	```

**Response:**
* With valid BEARER token on success: `201 Created'
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity/tag

Remove the tag from an agent installed on a VOLTTRON platfrom.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/agents

Return routes for the agents installed on the platform.

Accepts a query parameter `packaged`, which is false by default.  If true, this endpoint will instead return filenames of packaged agents on the platform which can ben installed.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<vip_identity>": "/platforms/:platform/agents/:vip_identity",
            "<vip_identity>": "/platforms/:platform/agents/:vip_identity",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/agents

Install an agent on the VOLTTRON platform.

This request may use the query parameters `vip_identity`, `tag`, `start` and `enable`.  These will be used when installing the agent.

**Request:**
* Authorization: `BEARER <jwt_token>`
* To upload a wheel:
    * Content Type: `application/x-wheel+zip`
    * Content Length: `<file_length>`
    * Body:
        ```
        <file_contents>
        ```
* To install wheel already on the remote platform:
    * Content Type: `application/json`
    * Body:
        ```
        {
            "wheel_path": "<wheel_path>"
        }
        ```

* To upload zipped source for packaging and installation:
    * Content Type: `application/zip`
    * Content Length: `<file_length>`
    * Body:
        ```
        <file_contents>
        ```

* To package and install from source on the remote system:
    * Content Type: `application/json`
    * Body:
        ```
        {
            "source_path": "<source_path>",
            "config_path": "<config_path>"
        }
        ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Content Type: `application/json`
    * Location: `/platforms/:platform/agents/:vip_identity`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/agents/:vip_identity

Get information about one installed agent. Returns routes to get information on health, status, remote procedure calls, monitored pubsub topics, configurations, front-ends (if available), and to perform control actions such as run status (starting/stoping), enabled status (enable/disable), disabling, and any other relevant actions.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": "/platforms/:platform/agents/:vip_identity/status",
            "health": "/platforms/:platform/agents/:vip_identity/health",
            "rpc": "/platforms/:platform/agents/:vip_identity/rpc",
            "pubsub": "/platforms/:platform/agents/:vip_identity/pubsub",
            "configs": "/platforms/:platform/agents/:vip_identity/configs",
            "frontends": "/platforms/:platform/agents/:vip_identity/frontends",
            "running": "/platforms/:platform/agents/:vip_identity/running",
            "enabled": "/platforms/:platform/agents/:vip_identity/enable"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/agents/:vip_identity

Upgrade an agent on the VOLTTRON platform.

**Request:**
* Authorization: `BEARER <jwt_token>`
* To upgrade from an uploaded wheel:
    * Content Type: `application/x-wheel+zip`
    * Content Length: `<file_length>`
    * Body:
        ```
        <file_contents>
        ```
* To upgrade from a wheel already on the remote platform:
    * Content Type: `application/json`
    * Body:
        ```
        {
            "wheel_path": "<wheel_path>"
        }
        ```

* To upload zipped source for packaging and upgrade:
    * Content Type: `application/zip`
    * Content Length: `<file_length>`
    * Body:
        ```
        <file_contents>
        ```

* To package and upgrade from source on the remote system:
    * Content Type: `application/json`
    * Body:
        ```
        {
            "source_path": "<source_path>",
            "config_path": "<config_path>"
        }
        ```

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`
**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<vip_identity>": "<uuid>"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/agents/:vip_identity

Uninstall an agent from the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Configs



Platform Configs endpoints expose functionality associted with configuration files for a VOLTTRON platform.

* All Platform Configs endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/configs

Get routes to available configuration files for the specified platform (not an indiviual agent, but the platform itself).

**Request:**
* Authorization: BEARER <jwt_token>

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<config_name>": "/platforms/:platform/configs/:config_name",
            "<config_name>": "/platforms/:platform/configs/:config_name",
             ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/configs

Save a new platform configuration file.

The file name should be passed in the query parameter `file-name`.

The platform configuration files are currently either JSON, INI files. The MIME type of the response will be either `applciation/json` or `text/plain` in the case of INI files. This endpoint will return an error if the file already exists. To update an existing file, use the `PUT /platforms/:platform/configs/:file_name` endpoint.

> &#x26a0;&#xfe0f; **Editing platform configuration files can affect the ability of the platform to restart.**  It is not currently possible to repair an unstartable platfrom from the API. Fixing mistakes will require direct access to the device or SSH.

**Request:**

* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body (shown for JSON):
    ```
    {
        "<setting_name>": <value>,
        "<setting_name>": <value>,
        ...
    }
    ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/configs/:file_name`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

**Request body:**
```
{"<setting_name>": <value>, ...}
```

**Response body:**
```
 {"return": <value>}
```

----

#### GET /platforms/:platform/configs/:file_name

Get a configuration file for the platform (not for an individual agent).

The platform configuration files are currently either JSON, INI files. The MIME type of the response will be either `applciation/json` or `text/plain` in the case of INI files.

> &#x26a0;&#xfe0f; **Editing platform configuration files can affect the ability of the platform to restart.**  It is not currently possible to repair an unstartable platfrom from the API. Fixing mistakes will require direct access to the device or SSH.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
          "<setting_name>": <value>,
          "<setting_name>": <value>,
          ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/configs/:file_name

Replace an existing platform configuration file.

The platform configuration files are currently either JSON, INI files. The MIME type of the response will be either `applciation/json` or `text/plain` in the case of INI files. This endpoint will return an error if the file does not already exist. To create a new file, use the `POST /platforms/:platform/configs` endpoint.

> &#x26a0;&#xfe0f; **Editing platform configuration files can affect the ability of the platform to restart.**  It is not currently possible to repair an unstartable platfrom from the API. Fixing mistakes will require direct access to the device or SSH.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
    ```
    {
        "<setting_name>": <value>,
        "<setting_name>": <value>,
        ...
    }
    ```

**Response:**
* With valid BEARER token on success: `204 No Content`
    * Content Type: `application/json`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

**Request body:**
```
{"<setting_name>": <value>, ...}
```

**Response body:**
```
 {"return": <value>}
```

----

#### DELETE /platforms/:platform/configs/:file_name

Delete an existing configuration file for the platform (not for an individual agent).


> &#x26a0;&#xfe0f; **Editing platform configuration files can affect the ability of the platform to restart.**  It is not currently possible to repair an unstartable platfrom from the API. Fixing mistakes will require direct access to the device or SSH.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`

* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Status



Platform Status endpoints expose functionality regarding the status of a VOLTTRON platform instance.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/status

Get status for the agents on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        [
            "<vip_identity>": { 
                                "name": "<agent_name>",
                                "priority": "<priority>",
                                "tag": "<tag>",
                                "uuid": "<uuid>",
                                "running": "<running_status>",
                                "enabled": "<enabled_status>"
                               },
            "<vip_identity>": {...},
            ...
        ]
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Health



Platform Health endpoints expose functionality regarding the health of a VOLTTRON platform instance.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/health

Get health for the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": {<platform_health>}
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Devices



Platform Devices endpoints expose functionality associted with devices managed by a VOLTTRON platform.

* All Platform Devices endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

**Note on the use of topics in Device and Platform Device Requests**

There is no special meaning to most segements of a topic in the VOLTTRON platform.  Topics typically, however do follow some convention, for instance `:campus/:building/:device/:point`. In the context of devices, however, the final segement in a full topic denotes a point which can be read or actuated. Partial topics (not including the last segment) denote some collection of these point resources. A topic, for instance which is complete up to the `:device` level in the campus example would yield a single device containing one or more points. A topic complete to the `:building` level would include a set of devices, each containing some set of points. The response to any request containing a full topic may differ, therefore, from a partial topic because it contains a complete path to a resource which can be individually queried or acted upon. The exact difference may vary by endpoint, and as dictated by query parameters as described below.

The `-` character may be used alone to represent any value for a segment: `/:campus/-/:device` will match all devices with the name :device on the :campus in any building.  It is not possible to use `-` as a wildcard within a segment containing other characters: `/campus/foo-bar/:device` will only match a building called "foo-bar", not one called "foobazbar".

> &#x26a0;&#xfe0f; It is possible that more than one device on several connected systems may share the same topic. In this case, as with the `GET /devices` endpoint, the duplicates will be contained within a list. Users should always check the type of the value to determine if a given value in the dictionary is a string, list, or dict.

**Device and Platform Devices endpoints can accept the following query parameters to refine their output:**

* tag (default=null): Filter the result by the provided tag.
* regex (default=null): Filter the result by the provided regular expression (follows python re syntax).

GET requests additionally accept:

* include-points (default=false): 
    * If true, the response will return entries for every point. These will be a dictionary of dictionaries with route, writability, and value unless the result is further filtered by the corresponding parameters. If only one of `route`, `writable`, or `value` are additionally provided, the result will be a dictionary of the corresponding item. If none of these are set to true (the default), all will be returned:
        ```
        {
            "route: <route_to_point>,
            "writable": true|false,
            "value": <value>
        }
        ```
    * If false, the response will consist of a dictionary of routes to devices (the most complete topic without points). `route`, `writable`, and `value` will be ignored.
* route (default=false): IF true, the result will include the route to the points.
* writable (default=false): IF true, the result will include the writability of the points.
* value (default=false): IF true, the result will include the value of the points.



----

#### GET /platforms/:platform/devices

Get routes to all devices for a platform.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": "/platforms/:platform/devices/:topic",
            "<topic>": "/platforms/:platform/devices/:topic",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/devices/:topic/

Returns a collection of points and values for a device topic.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Platform Devices section for information on the use of topics.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": "/platform/:platform/devices/:topic",
            "<topic>": [
                        "/platform/:platform/devices/:topic",
                        "/platform/:platform/devices/:topic",
                       ]
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/devices/:topic/

Sets the value of the specified point and returns its new value and meta-data.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Platform Devices section for information on the use of topics.

If an attempt is made to set a point which is not writable, the response will be `405 Method Not Allowed`.

If the request uses partial topics and/or query parameters to select more than one point to set, the query parameter `write-all` must be set. If `write-all` is not set, the request will fail with `405 Method Not Allowed`. The request will also fail unless all writes are successful, and any points which would otherwise be set will be reverted to their previous value.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
	{
	    "value": <value>
	}
	```

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "value": <new_value>,
            "meta": <meta_data>       
        }
        ```
* With valid BEARER token if any point is not writable: `405 Method Not Allowed`:
    * Conent Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message indicating unwritable points>"
        }
        ```
* With valid BEARER token on any other failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/devices/:topic/

resets the value of the specified point and returns its new value and meta-data.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Platform Devices section for information on the use of topics.

If an attempt is made to set a point which is not writable, the response will be `405 Method Not Allowed`.

If the request uses partial topics and/or query parameters to select more than one point to set, the query parameter `write-all` must be set. If `write-all` is not set, the request will fail with `405 Method Not Allowed`. The request will also fail unless all writes are successful, and any points which would otherwise be set will be reverted to their previous value.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "value": <new_value>,
            "meta": <meta_data>       
        }
        ```
* With valid BEARER token if any point is not writable: `405 Method Not Allowed`:
    * Conent Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message indicating unwritable points>"
        }
        ```
* With valid BEARER token on any other failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Statistics



Platform Statistics endpoints expose functionality regarding the monitoring of the message bus of a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/statistics

Get enabled status of message bus statistics on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": "<enabled|disabled>"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application\json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/statistics

Enable message bus statistics on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application\json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/statistics

Disable message bus statistics on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application\json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Auths



Platform Auths endpoints expose functionality related to authentication/authorization records for agents running within a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/auths

Get routes for authentication/authorization records of all agents running in the VOLTTRON platform.

Request accepts the query parameter `records` which will cause the response to include auth records in addtion to the routes.

> **Note:** `<auth_user_id>` can take any value specified in a `user_id` field in an auth record on the system. For local agents, this should be the actual VIP identity. The string, however is arbitrary and may have other values for remote agents to distinguish them from local versions of the same agent.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body (with record=false):
        ```
        {
            "<auth_user_id>": "/platforms/:platform/auths/:auth_user_id",
            "<auth_user_id>": "/platforms/:platform/auths/:auth_user_id",
            ...
        }
        ```
    * Body (with records=true):
        ```
        {
            "<auth_user_id>": {
                "route": "/platforms/:platform/auths/:auth_user_id",
                "record": {
                    "domain": "<domain>",
                    "address": "<address>",
                    "capabilities": ["<capability>", ...],
                   "roles": ["<role>", ...],
                    "groups": ["<group>", ...],
                    "mechanism": "<mechanism>",
                    "credentials": "<public_key>",
                    "comments": "<comments>",
                    "enabled": true|false
                },
                ...
            }
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/auths

Create authentication/authorization record for an agent running in the VOLTTRON platform.

The `<user_id>` will become the `:auth_user_id` in the route returned in the `Location` header of the response.

> **Note:** A `<user_id>` can take any value but should be unique on the platform. For local agents, this should be the actual VIP identity. The string, however is arbitrary and may have other values for remote agents to distinguish them from local versions of the same agent.  


**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
	{
        "domain": "<domain>",
        "address": "<address>",
        "user_id": "<user_id>",
        "capabilities": ["<capability>", ...],
        "roles": ["<role>", ...],
        "groups": ["<group>", ...],
        "mechanism": "<mechanism>",
        "credentials": "<public_key>",
        "comments": "<comments>",
        "enabled": true|false
	}
	```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/auths/:auth_user_id`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/auths/:auth_user_id

Get authentication/authorization record for the specified agent running in the VOLTTRON platform.

> **Note:** `:auth_user_id` can take any value specified in a `user_id` field in an auth record on the system. For local agents, this should be the actual VIP identity. The string, however is arbitrary and may have other values for remote agents to distinguish them from local versions of the same agent.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
                "domain": "<domain>",
                "address": "<address>",
                "user_id": "<user_id>",
                "capabilities": ["<capability>", ...],
                "roles": ["<role>", ...],
                "groups": ["<group>", ...],
                "mechanism": "<mechanism>",
                "credentials": "<public_key>",
                "comments": "<comments>",
                "enabled": true|false
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/auths/:auth_user_id

Update existing authentication/authorization record for an agent running in the VOLTTRON platform.

> **Note:** `:auth_user_id` can take any value specified in a `user_id` field in an auth record on the system. For local agents, this should be the actual VIP identity. The string, however is arbitrary and may have other values for remote agents to distinguish them from local versions of the same agent.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
	{
        "domain": "<domain>",
        "address": "<address>",
        "user_id": "<user_id>",
        "capabilities": ["<capability>", ...],
        "roles": ["<role>", ...],
        "groups": ["<group>", ...],
        "mechanism": "<mechanism>",
        "credentials": "<public_key>",
        "comments": "<comments>",
        "enabled": true|false
	}
	```

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/auths/:auth_user_id

Remove an authentication/authorization record for an agent running in the VOLTTRON platform.

> **Note:** `:auth_user_id` can take any value specified in a `user_id` field in an auth record on the system. For local agents, this should be the actual VIP identity. The string, however is arbitrary and may have other values for remote agents to distinguish them from local versions of the same agent.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET platforms/:platform/auths/:auth_user_id/publickeys

Retrieve public keys for an agent or the platform.

Passing "-" as the path variable `auth_user_id` will return all public keys, including the server key for the platform.  Passing "platform" for `:auth_user_id` will return only the server key.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "platform": "<server_key>",
            "<vip_identity>": "<public_key>",
            "<vip_identity>": "<public_key>",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms KnownHosts



Platform Known-Hosts endpoints expose functionality regarding authentication of other VOLTTRON instances known to a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/known-hosts

Retrieve routes to the known-host records of platforms known to the specified platform.

In the case where more than one host has the same address on different ports, the value will be a list of routes to the corresponding hosts.

To retrieve the records for all known-hosts on the platform, set the `records` query parameter to true.

> &#x26a0;&#xfe0f; The colon is a reserved character in a URI. The path variable for routes to known hosts is delimited by a dash: e.g.: `<address>-<port>`.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<host_address>": "/platforms/:platform/known-hosts/:host_address-:host_port",
            "<host_address>": [
                "/platforms/:platform/known-hosts/:host_address-:host_port1",
                "/platforms/:platform/known-hosts/:host_address-:host_port2",
                ],
            ...
        }
        ```
    * Body (with records=true):
        ``` 
        {
            <host_address>": {
                    "route": "/platforms/:platform/known-hosts/:host_address-:host_port",
                    "record": {
                        "port": "<port>",
                        "key": "<server_key>"
                    },
                ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/known-hosts

Create a new known-host record for another platform known to the specified platform.

The `Location` header will return a route to the new resource, with `:known-host` having the format `:address-:port`.

> &#x26a0;&#xfe0f; The colon is a reserved character in a URI. The path `known-hosts` variable is delimited by a dash: e.g.: `:address-:port`.

**Request:**
* Authorization: `BEARER <jwt_token>`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "address": "<address>",
            "port": "<port>",
            "key": "<server_key>"
        },
        ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/known-hosts/:known-host`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/known-hosts/:known-host

Retrieve the address, port, and server keys of another platform known to the specified platform.

> &#x26a0;&#xfe0f; The colon is a reserved character in a URI. The path `known-hosts` variable is delimited by a dash: e.g.: `:address-:port`.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "address": "<address>",
            "port": "<port>",
            "key": "<server_key>"
        },
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/known-hosts/:known-host

Update the address, port, and server keys of another platform known to the specified platform.

> &#x26a0;&#xfe0f; The colon is a reserved character in a URI. The path `known-hosts` variable is delimited by a dash: e.g.: `:address-:port`.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
    ```
    {
        "address": "<address>",
        "port": "<port>",
        "key": "<server_key>"
    },
    ```

**Response:**
* With valid BEARER token on success: `200 OK`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/known-hosts/:known-host

Remove the known-host record of another platform known to the specified platform.

> &#x26a0;&#xfe0f; The colon is a reserved character in a URI. The path `known-hosts` variable is delimited by a dash: e.g.: `:address-:port`.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Roles



Platform Roles endpoints expose functionality related to roles which provide authorization for certain functionality exported by VOLTTRON agents.

Roles contain a set of capabilities which grant access to protected resources.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/roles

Retrieve routes to the role records on the specified platform.

To retrieve the records for all roles on the platform, set the `records` query parameter to true.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<role>": "/platforms/:platform/roles/:role",
            "<role>": "/platforms/:platform/roles/:role",
            ...
        }
        ```
    * Body (with records=true):
        ``` 
        {
            "<role>": {
                "route":  "/platforms/:platform/roles/:role",
                "capabilities": [<capability>, <capability>, ...]
                },
            "<role>": {
                "route":  "/platforms/:platform/roles/:role",
                "capabilities": [<capability>, <capability>, ...]
                },
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/roles

Create a new role on the specified platform.

The route to the new resource will be returned in the `Location` header of the response.

**Request:**
* Authorization: `BEARER <jwt_token>`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "role_name": "<role>",
            "capabilities": [
                "<capability>",
                "<capability>",
                ...
            ]
        }
        ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/roles/:role`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/roles/:role

Retrieve the capabilities assigned to a role on the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
            [
                <capability>,
                <capability>,
                ...
            ]
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/roles/:role

Update an existing role on the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`
    * Content Type: `application/json`
    * Body:
        ```
            [
                "<capability>",
                "<capability>",
                ...
            ]
        ```

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/roles/:role

Remove a role from the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Groups



Platform Groups endpoints expose functionality related to authorization groups within the VOLTTRON platform.

Groups contain a set of roles, which in turn contain sets of capabilities which grant authorization for protected services exported by agents.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /platforms/:platform/groups

Retrieve routes to the group records on the specified platform.

To retrieve the records for all groups on the platform, set the `records` query parameter to true.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<group>": "/platforms/:platform/groups/:group",
            "<group>": "/platforms/:platform/groups/:group",
            ...
        }
        ```
    * Body (with records=true):
        ``` 
        {
            "<group>": {
                "route":  "/platforms/:platform/groups/:group",
                "capabilities": [<role>, <role>, ...]
                },
            "<group>": {
                "route":  "/platforms/:platform/groups/:group",
                "capabilities": [<role>, <role>, ...]
                },
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/groups

Create a new group on the specified platform.

The route to the new resource will be returned in the `Location` header of the response.

**Request:**
* Authorization: `BEARER <jwt_token>`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "group_name": "<group>",
            "roles": {
                <role>,
                <role>,
                ...
            }
        }
        ```

**Response:**
* With valid BEARER token on success: `201 Created`
    * Location: `/platforms/:platform/groups/:group`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/groups/:group

Retrieve the capabilities assigned to a group on the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
            [
                <role>,
                <role>,
                ...
            ]
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /platforms/:platform/groups/:group

Update an existing group on the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`
    * Content Type: `application/json`
    * Body:
        ```
            [
                <role>,
                <role>,
            ...
            ]
        ```

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform/groups/:group

Remove a group from the specified platform.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

### Platforms Historians



Platform Historian endpoints expose functionality related to historians running on a VOLTTRON platform.

* All Agent Control endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

----

#### GET /historians

Retrieve routes to historians on the platform.

**Request:**
* Authorization: `BEARER <jwt_token>`


**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<vip_identity>": "/platform/historians/:historian",
            "<vip_identity>": "/platform/historians/:historian",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/historians/:historian

Retrieve routes for an historian.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "topics": "/platforms/:platform/historians/:historian/topics",
            "metatdata": "/platforms/:platform/historians/:historian/metadata",
            "records": "/platforms/:platform/historians/:historian/records"
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/historians/:historian/topics

Query topics from the historian.

By default, the response will contain all topics in the historian. The `pattern` query parameter may be used to filter the results. The `aggregate` query parameter may be used to retreive aggregate topics.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": {"id": <id>, "route": "/platforms/:platform/historians/:historian/topics/:topic"},
            "<topic>": {"id": <id>, "route": "/platforms/:platform/historians/:historian/topics/:topic"},
            ...
        }
        ```
    * Body (with `aggregate`):
        ```
        [
            {"topic_name": <topic>, "aggregation_type": <aggregation_type>, "aggregation_time_period": <aggregation_time_period>, "metadata": <metadata>}
            {"topic_name": <topic>, "aggregation_type": <aggregation_type>, "aggregation_time_period": <aggregation_time_period>, "metadata": <metadata>}
            ...
        ]
        ```

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform/historians/:historian/topics/:topic

Query data for a topic.

Several query parameters may be used to refine the results:

* start: datetime of the start of the query. None for the beginning of time.
* end: datetime of the end of of the query. None for the end of time.
* skip: skip this number of results (for pagination)
* count: return at maximum this number of results (for pagination)
* order: “FIRST_TO_LAST” for ascending time stamps, “LAST_TO_FIRST” for descending time stamps.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "values": [
                {"datetime": <datetime>: "value: <value>},
                {"datetime": <datetime>: "value: <value>},
        '       ...
            ],
        '   "metadata": {
                "key1": value1,
        '       "key2": value2,
        '        ...
            }
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### POST /platforms/:platform/historians/:historian/topics/:topic

Insert records into the historian.

The request body should contain a list of JSON objects matching the format of the record type being inserted (e.g.: record, analysis, datalogger, devices).

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
    [
        {
        <record>
	    }
    ]
	```

**Response:**
* With valid BEARER token on success: `201 Created`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms

Get routes for connected platforms.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<platform>": "/platforms/:platform",
            "<platform>": "/platforms/:platform",
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms

Stop all agents on all connected platforms.

> &#x26a0;&#xfe0f; When the request is sent with the `platfroms=true` query parameter, this will stop the platforms themselves. This is not generally recommended, as there is currently no way to restart the platforms through the API.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /platforms/:platform

Return information about the platform and routes to related endpoints.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "status": "/platforms/:platform/status",
            "health": "/platforms/:platform/health",
            "agents": {
                "vip_identity": "/platforms/:platform/agents/:vip_identity",
                "vip_identity": "/platforms/:platform/agents/:vip_identity",
                ...
                },
            "configs": {
                "<config_name>: "/platforms/:platform/configs/:config_name",
                "<config_name>: "/platforms/:platform/configs/:config_name",
                ...
                }
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /platforms/:platform

Stop all agents on the platform.

> &#x26a0;&#xfe0f; When the request is sent with the `platfroms=true` query parameter, this will stop the platform itself. This is not generally recommended, as there is currently no way to restart the platform through the API.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `204 No Content`

* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

----

----

----

## Devices



Devices endpoints expose functionality associted with devices managed by all connected VOLTTRON platforms.

* All Devices endpoints require a JWT bearer token obtained through the `POST /authenticate` or `PUT /authenticate` endpoints.

**Note on the use of topics in Device and Platform Device Requests**

There is no special meaning to most segements of a topic in the VOLTTRON platform.  Topics typically, however do follow some convention, for instance `:campus/:building/:device/:point`. In the context of devices, however, the final segement in a full topic denotes a point which can be read or actuated. Partial topics (not including the last segment) denote some collection of these point resources. A topic, for instance which is complete up to the `:device` level in the campus example would yield a single device containing one or more points. A topic complete to the `:building` level would include a set of devices, each containing some set of points. The response to any request containing a full topic may differ, therefore, from a partial topic because it contains a complete path to a resource which can be individually queried or acted upon. The exact difference may vary by endpoint, and as dictated by query parameters as described below.

The `-` character may be used alone to represent any value for a segment: `/:campus/-/:device` will match all devices with the name :device on the :campus in any building.  It is not possible to use `-` as a wildcard within a segment containing other characters: `/campus/foo-bar/:device` will only match a building called "foo-bar", not one called "foobazbar".

> &#x26a0;&#xfe0f; It is possible that more than one device on several connected systems may share the same topic. In this case, as with the `GET /devices` endpoint, the duplicates will be contained within a list. Users should always check the type of the value to determine if a given value in the dictionary is a string, list, or dict.

**Device and Platform Devices endpoints can accept the following query parameters to refine their output:**

* tag (default=null): Filter the result by the provided tag.
* regex (default=null): Filter the result by the provided regular expression (follows python re syntax).

GET requests additionally accept:

* include-points (default=false): 
    * If true, the response will return entries for every point. These will be a dictionary of dictionaries with route, writability, and value unless the result is further filtered by the corresponding parameters. If only one of `route`, `writable`, or `value` are additionally provided, the result will be a dictionary of the corresponding item. If none of these are set to true (the default), all will be returned:
        ```
        {
            "route: <route_to_point>,
            "writable": true|false,
            "value": <value>
        }
        ```
    * If false, the response will consist of a dictionary of routes to devices (the most complete topic without points). `route`, `writable`, and `value` will be ignored.
* route (default=false): IF true, the result will include the route to the points.
* writable (default=false): IF true, the result will include the writability of the points.
* value (default=false): IF true, the result will include the value of the points.



----

#### GET /devices

Get routes to all devices for all connected platforms.

In the case where a device with the same topic appears on multiple platforms, the value for that topic in the result will be a list of routes where each element is the route to the device topic on on of the platforms where it appears.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": "/platform/:platform/devices/:topic",
            "<topic>": [
                        "/platform/:platform/devices/:topic",
                        "/platform/:platform/devices/:topic",
                       ]
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /devices/:topic

Get routes matching a topic for devices for all connected platforms.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Devices section for information on the use of topics.

Providing a partial topic returns all devices which share the given segements of the provided topic. Using a partial topic: `/:campus/:building` will produce the same dictionary as the response of `GET /devices/` but with only the topics beginning with the segments `:campus` and `:building`.

Providing a full topic will usually produce a single result, except where more than one connected platform has the same topic, in which case the result will be the list of routes to that topic on each containing platform.

In the case where a device with the same topic appears on multiple platforms, the value for that topic in the result will be a list of routes where each element is the route to the device topic on on of the platforms where it appears.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": "/platform/:platform/devices/:topic",
            "<topic>": [
                        "/platform/:platform/devices/:topic",
                        "/platform/:platform/devices/:topic",
                       ]
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### PUT /devices/:topic

Sets the value of the specified point(s) and returns its new value(s) and meta-data.

> **Note:** Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Devices section for information on the use of topics.

If an attempt is made to set a point which is not writable, the response will be `405 Method Not Allowed`.

If the request uses partial topics and/or query parameters to select more than one point to set, the query parameter `write-all` must be set. If `write-all` is not set, the request will fail with `405 Method Not Allowed`. The request will also fail unless all writes are successful, and any points which would otherwise be set will be reverted to their previous value.

**Request:**
* Authorization: `BEARER <jwt_token>`
* Content Type: `application/json`
* Body:
	```
	{
	    "value": <value>
	}
	```

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "value": <new_value>,
            "meta": <meta_data>       
        }
        ```
* With valid BEARER token if any point is not writable: `405 Method Not Allowed`:
    * Conent Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message indicating unwritable points>"
        }
        ```
* With valid BEARER token on any other failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### DELETE /devices/:topic

Resets the value of the specified point(s) and returns its new value(s) and meta-data.

> **Note:** Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Devices section for information on the use of topics.

If an attempt is made to set a point which is not writable, the response will be `405 Method Not Allowed`.

If the request uses partial topics and/or query parameters to select more than one point to set, the query parameter `write-all` must be set. If `write-all` is not set, the request will fail with `405 Method Not Allowed`. The request will also fail unless all writes are successful, and any points which would otherwise be set will be reverted to their previous value.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "value": <new_value>,
            "meta": <meta_data>       
        }
        ```
* With valid BEARER token if any point is not writable: `405 Method Not Allowed`:
    * Conent Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message indicating unwritable points>"
        }
        ```
* With valid BEARER token on any other failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /devices/:topic

Get routes matching a topic for devices for all connected platforms.

> **Note:** Platform Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Platform Devices section for information on the use of topics.

Providing a partial topic returns all devices which share the given segements of the provided topic. Using a partial topic: `/:campus/:building` will produce the same dictionary as the response of `GET /devices/` but with only the topics beginning with the segments `:campus` and `:building`.

Providing a full topic will usually produce a single result, except where more than one connected platform has the same topic, in which case the result will be the list of routes to that topic on each containing platform.

In the case where a device with the same topic appears on multiple platforms, the value for that topic in the result will be a list of routes where each element is the route to the device topic on on of the platforms where it appears.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<topic>": "/platform/:platform/devices/:topic",
            "<topic>": [
                        "/platform/:platform/devices/:topic",
                        "/platform/:platform/devices/:topic",
                       ]
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /devices/hierarchy

Retrieve a topical hierarchy of all devices on all platforms.

The response provides a dictionary organized hierarchically by segments within the device topic. There is no special meaning to the segments of device topics, but the example response shown here assumes that devices are organized in a pattern common for a campus of buildings: `/:campus/:building/:device/:point`.

> &#x26a0;&#xfe0f; It is not necessary that topics all have the same number of segments, therefore some parts of the tree may be deeper than others. Users should not assume a uniform depth to all branches of the tree. It is also possible that a given level of the tree is not uniformly either a dict or a string. For example, in the third building shown in the example, zone level devices have an extra segment to indicate they are served by a particular air handling unit, however the air handling unit device itself has the normal number of segments.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK` `201 Created` `204 No Content`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "<campus>": {
                "<building>": {
                    "<device>": "/platforms/:platform/devices/:topic/",
                    "<device>": "/platforms/:platform/devices/:topic/",
                    ...
                },
                "<building>"" {
                    "<device>": "/platforms/:platform/devices/:topic/",
                    "<device>": [
                        "/platforms/:platform1/devices/:topic/",
                        "/platforms/:platform2/devices/:topic/",
                        ...
                    ],
                    ...
                },
                "<building>": {
                    <ahu_device>: "/platforms/:platform/devices/:topic/",
                    <ahu>: {
                        "<zone_device>": "/platforms/:platform/devices/:topic/",
                        "<zone_device>": "/platforms/:platform/devices/:topic/",
                        ...
                    },
                    ...
                },
                ...
            },
            ...
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /devices/hierarchy/:topic

Retrieve a partial topical hierarchy of all devices on all platforms.

> **Note:** Device endpoints accept query parameters to refine their output, as described in the introduction to the Devices section.

> **Note:** See the introduction to the Devices section for information on the use of topics.

As with the `GET /devices/hierarchy` endpoint, the response provides a dictionary organized hierarchically by segments within the device topic. Providing a partial topic produces a clade of the device tree with its root at the provided topic.

As elaborated on in the introduction to the Devices section, there is no special meaning to the segments of device topics, but the example response shown here assumes that devices are organized in a pattern common for a campus of buildings: `/:campus/:building/:device/:point`.  Using a partial topic: `/:campus/:building` will produce a the same object which could be obtained by indexing the response of `GET /devices/hierarchy` first with `<campus>` and then n`<device>`, e.g.:

    ```
        responseObject['<campus>']['<building>']
    ```
The example shown in the response section below is produced by `GET /devices/hierarcy/:campus/:building`, where `:building` has the same value as the second building in the example shown for `GET /devices/hierarchy`.

Providing a full topic will usually produce a single leaf node, except where more than one connected platform has the same topic, in which case the result will be the list of leaf nodes corresponding to that topic on each containing platform.

> &#x26a0;&#xfe0f; It is not necessary that topics all have the same number of segments, therefore some parts of the tree may be deeper than others. Users should not assume a uniform depth to all branches of the tree. It is also possible that a given level of the tree is not uniformly either a dict or a string. For example, in the third building shown in the example, zone level devices have an extra segment to indicate they are served by a particular air handling unit, however the air handling unit device itself has the normal number of segments.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK` `201 Created` `204 No Content`
    * Content Type: `application/json`
    * Body:
        ```{
            "<device>": "/platforms/:platform/devices/:topic/",
             "<device>": [
                    "/platforms/:platform1/devices/:topic/",
                    "/platforms/:platform2/devices/:topic/",
                    ...
                ],
                ...
            }
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

----

#### GET /keypair

Generate a new public/private keypair.

**Request:**
* Authorization: `BEARER <jwt_token>`

**Response:**
* With valid BEARER token on success: `200 OK`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "public": "<public_key>,
            "private": "<private_key>
        }
        ```
* With valid BEARER token on failure: `400 Bad Request`
    * Content Type: `application/json`
    * Body:
        ```
        {
            "error": "<Error Message>"
        }
        ```
* With invalid BEARER token: `401 Unauthorized`

