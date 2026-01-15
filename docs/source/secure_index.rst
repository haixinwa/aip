.. Agent-Interaction-Protocol documentation master file, created by
   sphinx-quickstart on Mon Sep 15 10:37:39 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Agent-Interaction-Protocol Secure Documentation
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
~~~~~~~~~~

.. image:: https://github.com/user-attachments/assets/b43e74cf-cb86-4d43-91c7-4409d84aa7c8
   :width: 600
   :height: 300
   :align: center


The ​​Agent Interaction Protocol (AIP)​​ is a distributed agent interaction protocol designed for multi-agent collaboration. AIP supports both agent-to-agent connectivity and agent-to-tool invocation. In addition to point-to-point connections, all agents and tools can be mounted as nodes on a shared gateway to form an interconnected network.

Characteristics
~~~~~~~~~~~~~~~~
- AIP supports both centralized proxy and peer-to-peer communication.
- AIP uses ​​gRPC​​ as the underlying communication protocol, enabling faster serialization/deserialization through binary data transmission.
- Agents communicate via ​​bidirectional asynchronous streams​​, supporting full-duplex real-time interaction.
- AIP supports secure communication using mutual TLS (mTLS) authentication.

Architecture
~~~~~~~~~~~~~
.. image:: https://github.com/user-attachments/assets/7e593a88-ab0a-42fc-bca1-fd04fe739755
   :width: 300
   :height: 240
   :align: center


AIP adopts a layered design:

- ​​Module Layer​​: Implements business logic distribution and coordination in the form of nodes or gateways.

- Session Layer​​: Manages the lifecycle of each communication session between nodes, using unique session IDs and asynchronous task coordination to ensure real-time message processing and resource safety.

- ​​Transport Layer​​: Handles message dispatching and reception between the Session Layer and gRPC interfaces, with optional TLS encryption.

.. image:: https://github.com/user-attachments/assets/e674e3c6-b13c-43d2-a8c5-efb306240bd7
   :width: 600
   :height: 400
   :align: center


AIP supports two communication modes:

- Request-Response​​: Used for agent-to-tool interactions.

- ​​Bidirectional Asynchronous Streaming​​: Used for real-time full-duplex agent-to-agent communication. The session initiator is responsible for sending session termination requests.


Security Features
------------------

Authentication Mechanism
~~~~~~~~~~~~~~~~~~~~~~~~
AIP implements **mutual TLS (mTLS)** authentication to ensure secure communication between all components. This means:

- Both clients and servers authenticate each other using certificates
- All communication is encrypted end-to-end
- Only trusted entities with valid certificates can participate in the network

Certificate Hierarchy
~~~~~~~~~~~~~~~~~~~~~~
AIP uses a certificate-based authentication system with the following components:

1. **CA Certificate (Root Certificate)**
   - Self-signed certificate used to sign all other certificates
   - Establishes trust in the entire system

2. **Server Certificates**
   - Used by Gateway, Agent, and ToolBox servers
   - Signed by the CA certificate
   - Contains server-specific information and ``serverAuth`` extension

3. **Client Certificates**
   - Used by Agent clients and Gateway clients
   - Signed by the CA certificate
   - Contains client-specific information and ``clientAuth`` extension

Certificate Generation Guide
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use AIP in secure mode, you need to generate TLS certificates. Follow these steps:

1. **Create a certificate directory**
   .. code-block:: bash
      
      mkdir credentials
      cd credentials

2. **Generate CA certificate**
   .. code-block:: bash
      
      # Generate CA private key
      openssl genrsa -out ca.key 2048
      
      # Generate self-signed CA certificate
      openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt -subj "/CN=AIP-CA"

3. **Generate server certificate**
   .. code-block:: bash
      
      # Generate server private key
      openssl genrsa -out server.key 2048
      
      # Create certificate signing request
      openssl req -new -key server.key -out server.csr -subj "/CN=localhost"
      
      # Create extension configuration
      @"  
      authorityKeyIdentifier=keyid,issuer  
      basicConstraints=CA:FALSE  
      keyUsage = digitalSignature, keyEncipherment  
      subjectAltName = DNS:localhost, IP:127.0.0.1  
      extendedKeyUsage = serverAuth  
      "@ | Out-File server.ext
      
      # Issue server certificate
      openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile server.ext

4. **Generate client certificate**
   .. code-block:: bash
      
      # Generate client private key
      openssl genrsa -out client.key 2048
      
      # Create certificate signing request
      openssl req -new -key client.key -out client.csr -subj "/CN=AIP-Client"
      
      # Create extension configuration
      @"  
      authorityKeyIdentifier=keyid,issuer  
      basicConstraints=CA:FALSE  
      keyUsage = digitalSignature, keyEncipherment  
      extendedKeyUsage = clientAuth  
      "@ | Out-File client.ext
      
      # Issue client certificate
      openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAserial ca.srl -out client.crt -days 365 -sha256 -extfile client.ext

Secure Mode Usage
------------------

Enabling Security in Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
All AIP components support secure mode through the ``--with_auth`` flag. When enabled, components will:

1. Load TLS certificates from specified paths
2. Establish encrypted connections using mutual TLS
3. Authenticate all communication peers

Gateway in Secure Mode
~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash
   
   python run_gateway.py --with_auth --private_key server.key --public_key ca.crt

Agent in Secure Mode
~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash
   
   python run_and_register_agent.py --with_auth --private_key client.key --public_key ca.crt --llm_url <URL> --api_key <KEY> --model <MODEL>

ToolBox in Secure Mode
~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash
   
   python run_and_register_tool.py --with_auth --private_key client.key --public_key ca.crt

Client in Secure Mode
~~~~~~~~~~~~~~~~~~~~~
.. code-block:: bash
   
   python run_client_agent.py --with_auth --private_key client.key --public_key ca.crt

Security API
------------

enable_security Method
~~~~~~~~~~~~~~~~~~~~~~
The core security method implemented by all components:

.. code-block:: python
   
   async def enable_security(self, private_key_path: str, public_key_path: str):
       """
       Enable TLS security for the component.
       
       Args:
           private_key_path: Path to the private key file
           public_key_path: Path to the CA certificate file
       """
       self.private_key_path = private_key_path
       self.public_key_path = public_key_path
       
       with open(private_key_path, 'rb') as f:
           private_key = f.read()
       with open(public_key_path, 'rb') as f:
           certificate_chain = f.read()

       # Set up server credentials for TLS
       self.server_credentials = grpc.ssl_server_credentials(
           [(private_key, certificate_chain)]
       )
       
       # Set up client credentials for TLS
       self.client_credentials = grpc.ssl_channel_credentials(
           root_certificates=certificate_chain
       )
       
       self.with_auth = True

Security Configuration Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All secure components accept the following parameters:

- ``--with_auth``: Enable secure mode (required for secure communication)
- ``--private_key``: Path to the private key file
- ``--public_key``: Path to the CA certificate file
- ``--host_address``: Network address to listen on

Protocol Data
~~~~~~~~~~~~~~

.. include:: index.rst
   :start-after: Protocol Data
   :end-before: PYTHON API

PYTHON API
~~~~~~~~~~~~~~

.. include:: index.rst
   :start-after: PYTHON API