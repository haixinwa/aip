.. Agent-Interaction-Protocol documentation master file, created by
   sphinx-quickstart on Mon Sep 15 10:37:39 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Agent-Interaction-Protocol documentation
========================================

Add your content using ``reStructuredText`` syntax. See the
`reStructuredText <https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html>`_
documentation for details.


.. toctree::
   :maxdepth: 2
   :caption: Contents:

Introduction
--------------
Conception
~~~~~~~~~~~

.. image:: https://raw.githubusercontent.com/haixinwa/aip/07e51e315c03ad32c0af7a8639ab621d53a77821/docs/source/_static/image1.png
   :width: 600
   :height: 300
   :align: center


The ​​Agent Interaction Protocol (AIP)​​ is a distributed agent interaction protocol designed for multi-agent collaboration. AIP supports both agent-to-agent connectivity and agent-to-tool invocation. In addition to point-to-point connections, all agents and tools can be mounted as nodes on a shared gateway to form an interconnected network.

Characteristics
~~~~~~~~~~~~~~~~
- AIP supports both centralized proxy and peer-to-peer communication.
- AIP uses ​​gRPC​​ as the underlying communication protocol, enabling faster serialization/deserialization through binary data transmission.
- Agents communicate via ​​bidirectional asynchronous streams​​, supporting full-duplex real-time interaction.

Architecture
~~~~~~~~~~~~~
.. image:: https://raw.githubusercontent.com/haixinwa/aip/07e51e315c03ad32c0af7a8639ab621d53a77821/docs/source/_static/image2.png
   :width: 300
   :height: 240
   :align: center


AIP adopts a layered design:

- ​​Module Layer​​: Implements business logic distribution and coordination in the form of nodes or gateways.

- Session Layer​​: Manages the lifecycle of each communication session between nodes, using unique session IDs and asynchronous task coordination to ensure real-time message processing and resource safety.

- ​​Transport Layer​​: Handles message dispatching and reception between the Session Layer and gRPC interfaces.

.. image:: https://raw.githubusercontent.com/haixinwa/aip/00f6366ba36cba54b922bb31d942179a37715326/docs/source/_static/image3.png
   :width: 600
   :height: 400
   :align: center


AIP supports two communication modes:

- Request-Response​​: Used for agent-to-tool interactions.

- ​​Bidirectional Asynchronous Streaming​​: Used for real-time full-duplex agent-to-agent communication. The session initiator is responsible for sending session termination requests.

Protocol Data
~~~~~~~~~~~~~~

Mode (Enum)
^^^^^^^^^^^^^^

.. code-block:: python

   #Defines the type of data for transmission or processing.

   TEXT(0): Text data.
   IMAGE(1): Binary image data.
   AUDIO(2): Binary audio data.
   EMBEDDED(3): Custom binary data (e.g. serialized objects).

Examples:

- mode = Mode.text :specifies the current content as TEXT type

-----

SessionStatus (Enum)
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Represents the state of a session.

   START_QUEST(0): Session start, sending the first request (client).
   HOLD_QUEST(1): Subsequent request after session start (client).
   HOLD_RESPONSE(2): Response after session start (server).
   STOP_QUEST(3): Termination request after session start (client).
   STOP_RESPONSE(4): Abnormal termination due to timeout or error, or upon receiving a termination request from the client (server).

Examples:

- status = SessionStatus.START_QUEST # marks the session as "Task Start" status

-----

TaskStatus (Enum)
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Represents the execution status of a task.

   CREATE(0): Task created but not executed.
   EXECUTING(1): Task is executing.
   WAITING(2): Task is waiting for dependencies.
   BREAK(3): Task was interrupted.
   FINISH(4): Task completed.

Examples:

- task_status = TaskStatus.EXECUTING # mark the task as "running"

-----

UpdateType (Enum)
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   ADDED(0): 
   UPDATED(1): 
   REMOVED(2): 

Examples:


-----

AgentSkill
^^^^^^^^^^^^

.. code-block:: python

   #Describes an Agent's skill.

   - skill_id(str): Unique skill identifier (e.g. "translation").
   - capability(str): Description of the skill (e.g. "Multilingual translation").

Examples:

- skill = AgentSkill(skill_id="translation", capability=" multilingual translation")

-----

TaskInfo
^^^^^^^^^^

.. code-block:: python

   #Stores basic information about a task.

   - task_id (str): Unique task identifier (e.g., `"task_001"`).
   - parent_task_ids (List[str]): List of parent task IDs (for dependencies).
   - task_status (TaskStatus): Task status (based on the TaskStatus enum).

Examples:

- task = TaskInfo(task_id="task_001", task_status=TaskStatus.CREATE)

-----

ContentItem
^^^^^^^^^^^^^

.. code-block:: python

   #Encapsulates message content, supporting multiple data types.

   - _text (Optional[str]): Text content.
   - _image (Optional[bytes]): Binary image data.
   - _audio (Optional[bytes]): Binary audio data.
   - _embedded (Optional[bytes]): Custom binary data.

Methods:

- `write_text(text: str)`: Creates a text content item.
- `write_image(data: bytes)`: Creates an image content item.
- `write_audio(data: bytes)`: Creates an audio content item.
- `write_embedded(data: bytes)`: Creates a custom binary content item.

Examples:

- text_item = ContentItem.write_text("Hello, World!" )
- image_item = ContentItem.write_image(b"image_data")

-----

Peer
^^^^^^^^^^^^^

.. code-block:: python

   #Represents a peer node (Agent or Tool) in communication.

   - agent_info (Optional[AgentInfo]): Detailed information about an Agent.
   - tool_info (Optional[ToolBoxInfo]): Detailed information about a Tool.

Examples:

- peer = Peer()
- peer.agent_info = AgentInfo(agent_id="agent_001")

-----

AgentInfo
^^^^^^^^^^^^^

.. code-block:: python

   #Describes detailed information of an Agent.

   - agent_id (str): Unique identifier of the Agent.
   - address (str): Network address of the Agent.
   - name (str): Name of the Agent.
   - domain (str): Domain the Agent belongs to.
   - input_mode (List[Mode]): List of supported input modes.
   - output_mode (List[Mode]): List of supported output modes.
   - description (str): Functional description.
   - skills (List[AgentSkill]): List of skills.
   - version (str): Version number.

Examples:

- agent = AgentInfo(agent_id="agent_001", input_mode=[Mode.TEXT])

-----

RegisterAgentResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Response result after Agent registration

   - success (bool): Whether the registration was successful(True/False) 
   - peers (List[Peer]): List of known nodes(e.g:Peer(...),Peer(...)).

Examples:

- response = RegisterAgentResponse(success=True, peers=[peer1, peer2])

-----

ToolInfo
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Describes detailed information of a Tool.

   - name (str): Unique identifier of the Tool.
   - description (str): Functional description.
   - arguments (Dict[str, str]): Parameters required to call the Tool.
   - version (str): Version number.

Examples:

- tool = ToolInfo(name ="tool_001", arguments={"lang": "en"})

-----

ToolBoxInfo
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Describes detailed information of a ToolBox.

   - toolbox_id (str): Unique identifier of the ToolBox.
   - address (str):The address (e.g., URL or IP) of the ToolBox service.
   - name (Dict[str, str]): PA dictionary that maps language codes (like 'en', 'zh') to the name of the ToolBox in that language.
   - domain (str): The domain or category that the ToolBox belongs to
   - description (str): A functional description of the ToolBox.
   - tools (str): A list of ToolInfo objects representing the tools contained in the ToolBox.

Examples:

-----

RegisterToolResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #The response result after the tool registration.

   - success (bool) : Whether the registration was successful (True/False).
   

Examples:

- response = RegisterToolResponse(success=True)

-----

DeregisterNodeRequest
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Request the gateway to cancel the node.

   - node_id (str) : The ID of the node (Agent or tool) sending the request (e.g. "agent_001"/"tool_001").
   

Examples:

- request = DeregisterNodeRequest(node_id="agent_001")

-----

DeregisterNodeResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Response result after node deregister.

   - success (bool) : Whether the logout was successful (True/False).
   
Examples:

- response = DeregisterNodeResponse(success=True)

-----

GetNodesRequest
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Request to obtain node information within the specified domain.

   - agent_id (str) : The Agent ID that sent the request (e.g. "agent_001"). 
   - domain (str) : Target domain name (e.g. "translation").
   
Examples:

- request = GetNodesRequest(agent_id="agent_001", domain="translation")

-----

GetNodesResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #To return a list of nodes within the specified domain.

   - peers (List [Peer]) : node List (such as [Peer (...), Peer (...)]).
   
Examples:

- response = GetNodesResponse(peers=[peer1, peer2])

-----

AgentMessage
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Message body for communication between Agents.

   - sender_id (str): Sender ID.
   - receiver_id (str): Receiver ID.
   - session_id (str): Unique session identifier.
   - session_status (SessionStatus): Session status.
   - task_info (TaskInfo): Associated task information.
   - message_id (str): Unique message identifier.
   - reply_to_message_id (str): ID of the message being replied to.
   - content (List[ContentItem]): List of message content items.

Methods:

- add_content(item: ContentItem): Adds a content item to the message.

Examples:

- msg = AgentMessage(sender_id="agent_001", receiver_id="agent_002")
- msg.add_content(ContentItem.write_text("Hello!" ))

-----

ToolRequest
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Request sent to a Tool.

   - sender_id (str): Sender ID.
   - receiver_id (str): Receiver ID (Tool ID).
   - session_id (str): Session ID.
   - tool_name (str): Name of the tool to call.
   - arguments (Dict[str, Any]): Arguments for the tool call.

Examples:

- request = ToolRequest(tool_name="translator", arguments={"text": "Hello"})

-----

ToolResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Response from a Tool after execution.

   - sender_id (str): Sender ID (Tool ID).
   - receiver_id (str): Receiver ID.
   - session_id (str): Session ID.
   - is_error (bool): Indicates if an error occurred.
   - error_message (str): Error description.
   - content (List[ContentItem]): List of response content items.

Methods:

- add_content(item: ContentItem): Adds a content item to the response.


Examples:

- response = ToolResponse(is_error=False, content=[text_item])

-----

HeartbeatRequest
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Heartbeat request sent periodically to the gateway.

   - sender_id (str): ID of the node (Agent or Tool) sending the request.

Examples:

- request = HeartbeatRequest(sender_id="agent_001")

-----

HeartbeatResponse
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #Response to a heartbeat request

   - success (bool): Indicates if the heartbeat was successful.
   - message (str): Response message.

Examples:

- response = HeartbeatResponse(success=True, message="Heartbeat received")

-----

UpdateNodeInfoRequest 
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - node_info (peer): 

Examples:

-----

UpdateNodeInfoResponse 
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - success (bool): Whether the logout was successful (True/False)
   - message (str): Response message.

Examples:

-----

UpdateSubscriptionRequest
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - subscriber_id (str): 
   - node_ids (str): The ID of the node (Agent or tool) sending the request (e.g. "agent_001"/"tool_001").

Examples:

-----

NodeUpdate
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - update_type (UpdateType): 
   - node_id (str): The ID of the node (Agent or tool) sending the request (e.g. "agent_001"/"tool_001").
   - peer (List[Peer]): node List (such as [Peer (...), Peer (...)]).

Examples:

-----

UnsubscribeRequest 
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - subscriber_id (str): 
   - node_ids (str): The ID of the node (Agent or tool) sending the request (e.g. "agent_001"/"tool_001").

Examples:

-----

UnsubscribeResponse
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

   #

   - success (bool): Whether the logout was successful (True/False)
   - message (str): Response message.

Examples:

-----

PYTHON API
~~~~~~~~~~~~~~

GRPC_SERVICE
^^^^^^^^^^^^^^

Agent_service
++++++++++++++

Gateway_service
++++++++++++++++++

Schema_pb2_grpc
++++++++++++++++++

Schema_pb2
++++++++++++++

Tool_service
++++++++++++++

Type
++++++++++++++

Utils
++++++++++++++

SESSION
^^^^^^^^^^^^^^

Agent_session
++++++++++++++

Gateway_session
+++++++++++++++++

Tool_session
++++++++++++++

MODULE
^^^^^^^^^^^^^^

Gateway
++++++++++++++

.. code-block:: python

   #class moudle.Gateway(host_address: str,service_address: str,gateway_id: str)
   Provides a central proxy service for managing agent registration and communication. As a core component, Gateway maintains information about all nodes in the network and ensures message routing and communication between nodes.
   - address (str): Deployment address of the gateway service (e.g., "localhost:50051")
   - gateway_id (str): Unique identifier for the gateway (auto-generated if not provided)

Methods:

- start()
   Initializes and starts the gateway service to receive registration requests and handle routing.
- stop()
   Stops the gateway server and releases resources.
- connect_to_gateway(gateway_address) 
   Connect to another gateway.

   - Parameter : gateway_address :  Address of the gateway to connect to
- get_registered_nodes() → Dict[str, Union[AgentInfo, ToolInfo]、
   Gets all registered nodes (Agents and Tools).

   - Returns a dictionary of: {node ID: node information} (Dict[str, Union[AgentInfo, ToolInfo]])
- get_registered_agents() → Dict[str, AgentInfo]
   Gets all registered Agents.

   - Returns a dictionary of: {agent ID: agent information} (Dict[str, AgentInfo])
- get_registered_tools() → Dict[str, ToolInfo]
   Gets all registered Tools.

   - Returns a dictionary of: {tool ID: tool information} (Dict[str, ToolInfo])

-----

Agent
++++++++++++++

.. code-block:: python

   #class module.Agent(host_address: str,service_address: str,agent_id: str,name: str,domain: str,description: str,version: str,input_mode: str,output_mode: str,skills: str)
   Creates, manages, and runs Agents. Supports bidirectional streaming communication with other Agents and synchronous Tool invocation.
   - address (str): Deployment address of the agent service.
   - agent_id (str): Unique identifier of the agent.
   - name (str): Name of the agent.
   - domain (str): Domain or group the agent belongs to.
   - description (str): Detailed functional description of the agent.
   - version (str): Agent version.
   - input_mode (List[Mode]): List of input modalities the agent accepts. Possible values: `TEXT`, `IMAGE`, `AUDIO`, `EMBEDDED`.
   - output_mode (List[Mode]): List of output modalities the agent provides. Possible values: `TEXT`, `IMAGE`, `AUDIO`, `EMBEDDED`.
   - skills (List[AgentSkill]): List of skills the agent possesses.
   - agent_info (AgentInfo): An object containing agent information for registration with the gateway.

Methods:

- _create_agent_info() → AgentInfo
   Create an AgentInfo object for registration with the gateway.
   - Return: An AgentInfo object
- start() → Agent
   Starts the agent server.
   - Returns: self
- stop()
   Stops the agent server and closes all client connections.
- register_to_gateway(gateway_address,timeout: float = 5.0) → Agent
   Registers the agent with a gateway.

   - Parameter: gateway_address (str) : The address of the gateway service
   - timeout: Timeout for registering to the gateway service (default is 5 seconds)

   - Returns: self
- update_peers(domain="default") → Agent
   Queries the gateway for all nodes (Agents and Tools) and updates the locally stored peer information.

   - Parameter: domain (str) : The domain to be queried. Optional parameter, default" default"

   - Returns: self
- get_nodes () -> Dict[str, Union[AgentInfo, Any]] 
   queries the peers node information stored in the current Agent service.

   - Returns: Dictionary of node information
- create_task_info(parent_task_ids=None) → TaskInfo
   Creates a new TaskInfo object.

   - Parameter: parent_task_ids (List[str]) : list of parent task ids

   - Returns: New TaskInfo object
- create_content_items(content, content_mode=None) → List[ContentItem]
   Creates a list of ContentItem objects for message content.

   - Parameters: 
     content (Union[str, bytes, List[Union[str, bytes]]]) : The message content can be either a list or a single content, and the content type supports both string and bytes. Among them, the text mode uses the string type, and the image, audio, and embedding modes use the bytes type.
     content_mode (Optional[Union[Mode, List[Mode]]]) : The modal or modal list of the message content, corresponding one-to-one with the elements of the message list.

   - Returns: List of ContentItem objects
- create_agent_message(receiver_id, content, session_id=None, task_info=None, content_mode=None, session_status=SessionStatus.START_QUEST, message_id=None, reply_to_message_id=None) → AgentMessage
   Creates an AgentMessage object for communication between agents.

   - Parameters: 
     receiver_id (str) : The agent ID that received the message
     content (Union[str, bytes, List[Union[str, bytes]]]) : The content of the message
     session_id (str,) : Session ID
     task_info (TaskInfo, optional) : Task information
     content_mode (Optional[Union[Mode, List[Mode]]]], optional) : The modality of the message content, corresponding one-to-one with the elements of the message list.
     session_status (SessionStatus optional) : Session status
     message_id (str, optional) : the ID of the current message
     reply_to_message_id (str, optional) : used to mark that the current message is a reply to a certain message ID

   - Returns: AgentMessage object

- create_agent_client(receiver_id, stub=GatewayServiceStub, agent_calling="RouteAgentCalling,serve_address=None") → str
   Creates an AgentClient object for sending messages to other agents.

   - Parameters: 
     receiver_id (str) : The agent ID that received the message
     stub(GatewayServiceStub) : Gateway stub class
     agent_calling(str) : Agent message routing method
     serve_address : 

   - Returns: session_id of the created AgentClient object
- submit_inquiry(session_id, receiver_id, content, content_mode=None, session_status=None, task_info=None)
   Sends a request to another agent through the gateway.

   - Parameters: 
     session_id (str) : Session ID
     receiver_id (str) : ID of the agent that received the request
     content (Union[str, bytes, List[Union[str, bytes]]]) : The content of the message
     content_mode (Optional[Union[Mode, List[Mode]]]) : The mode of the message content, corresponding one-to-one with the elements of the message list.
     session_status (SessionStatus optional) : Session status
     task_info (TaskInfo, optional) : Task information

- receive_feedback(session_id, receiver_id) → AgentMessage
   Receives a response from another agent server.

   - Parameters: 
     session_id (str) : Session ID
     receiver_id (str) : Server agent proxy ID

   - Returns: AgentMessage object
- close_agent_client( session_id, receiver_id) -> bool
   Close a specific agent client.

   - Parameters:
     session_id:Session ID
     receiver_id:ID of the agent that received the response

   - Returns: True or False
- submit_feedback(session_id, receiver_id, content, content_mode=None, request_session_status: SessionStatus, task_info=None)
   Sends a response to another agent client.

   - Parameters: 
     session_id (str) : Session ID
     receiver_id (str) : ID of the agent that received the response
     content (Union[str, bytes, List[Union[str, bytes]]]) : The content of the message
     content_mode (Optional[Union[Mode, List[Mode]]]) : The mode of the message content, corresponding one-to-one with the elements of the message list.
     request_session_status (SessionStatus optional) : Session status
     task_info (TaskInfo, optional) : Task information

- receive_inquiry() → AgentMessage
   Receives a request from another agent client.

   - Returns: AgentMessage object
- invoke_session_handlers(session_id, handlers) → List[Any]
   Receives responses from another agent server by invoking custom handlers.

   - Parameters: 
     session_id (str) : Session ID
     handlers (List[Callable]) : A custom list of request handling functions, each handler receives AgentMessage, processes the request, and returns the information of the processed response

   - Return: A list of response messages
- create_tool_client(receiver_id, stub=GatewayServiceStub, tool_calling="RouteToolCalling", serve_address=None) → ToolClient
   Creates a ToolRequest object for invoking a tool.

   - Parameters: 
     receiver_id (str) : The tool ID to be invoked
     stub : GatewayServiceStub
     tool_calling (str, optional) : ool_calling="RouteToolCalling"
     serve_address : address

   - Returns: ToolRequest object
- call_tool(toolbox_id, tool_name=None, arguments=None) → ToolResponse
   Invokes a tool with the given ID through the gateway.

   - Parameters: 
     toolbox_id (str) : The tool ID to be invoked
     tool_name (str, optional) : tool name
     arguments (Dict[str, Any], optional) : tool call parameters

   - Return: The ToolResponse object of the tool
- close_tool_client(toolbox_id) → bool
   Closes a specific tool client.

   - Parameter: toolbox_id (str) : the ID of the toolbox

   - Returns: True if the client is closed, False otherwise

-----

Tool
++++++++++++++

.. code-block:: python

   #class module.Tool
   Creates, manages, and executes Tools. Tools are invoked by Agents to perform specific tasks and return results.
   - address (str): Deployment address of the tool service.
   - tool_id (str): Unique identifier of the tool.
   - name (str): Name of the tool.
   - domain (str): Domain or group the tool belongs to.
   - description (str): Detailed functional description of the tool.
   - version (str): Tool version.
   - input_mode (List[Mode]): List of input modalities the tool accepts. Possible values: `TEXT`, `IMAGE`, `AUDIO`, `EMBEDDED`.
   - output_mode (List[Mode]): List of output modalities the tool provides. Possible values: `TEXT`, `IMAGE`, `AUDIO`, `EMBEDDED`.
   - arguments (Dict[str, str]): Dictionary of parameter names and their values for the tool.
   - tool_info (ToolInfo): An object containing tool information for registration with the gateway.

Methods:

- start() → Tool
   Starts the tool server.

   - Return: self
- stop()
   Stops the tool server.
- register_to_gateway(gateway_address, timeout: float = 5.0)
   Registers the tool with a gateway.

   - Parameter: gateway_address (str) : The address of the gateway service

   - timeout: Timeout for registering to the gateway service (default is 5 seconds)

   - Returns: self
- add_tool(fn, name, description, version)
   Add a tool to the server.

   - Parameters: fn: The function to register as a tool
     name: Optional name for the tool (defaults to function name)
     description: Optional description of what the tool does
     version: Tool version

   - Returns: tool
- tool(name, description, version,)
   Decorator to register a tool.

   - Parameters: 
     name: Optional name for the tool (defaults to function name)
     description: Optional description of what the tool does
     version: Tool version

   - Returns: fn

-----














































































































