# API Documentation - Network Data

This document outlines the available API endpoints for managing network configuration data, including Clients, Assessments, Network Topology (Nodes, Edges), ASNs, Devices, BGP sessions, VSIs, and VSI Peers.

**Base URL:** (Assuming your server runs locally on port 3000) `http://localhost:3000`

**Common Conventions:**
* Request bodies and response bodies are in JSON format.
* Success responses typically return the requested/created/updated resource(s).
* Error responses return a JSON object with an `error` key containing a description.
* Common Error Status Codes:
    * `400 Bad Request`: Invalid input (e.g., missing required fields, invalid ID format).
    * `404 Not Found`: The requested resource (e.g., client, device) does not exist.
    * `409 Conflict`: The operation conflicts with existing data (e.g., unique constraint violation, trying to delete a resource referenced by others).
    * `500 Internal Server Error`: An unexpected error occurred on the server.

---

## Clients API (`/clients`)

Manages client records.

### `POST /clients`

* **Description:** Creates a new client record.
* **Method:** `POST`
* **Path:** `/clients`
* **Request Body:**
    ```json
    {
      "name": "Optional Client Name"
    }
    ```
    * `name` (String, Optional): The name of the client.
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object representing the newly created client.
      ```json
      {
        "id": 1,
        "name": "Optional Client Name"
      }
      ```
* **Error Responses:** `500`

### `GET /clients`

* **Description:** Retrieves a list of all clients, including their associated assessments.
* **Method:** `GET`
* **Path:** `/clients`
* **URL Parameters:** None
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of client objects. Each client object includes an `assessments` array.
      ```json
      [
        {
          "id": 1,
          "name": "Client A",
          "assessments": [
            { "id": 101, "clientId": 1, "question1": "...", ... },
            { "id": 102, "clientId": 1, "question1": "...", ... }
          ]
        },
        { ... }
      ]
      ```
* **Error Responses:** `500`

### `GET /clients/:id`

* **Description:** Retrieves a single client by its ID, including its associated assessments.
* **Method:** `GET`
* **Path:** `/clients/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The unique ID of the client.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the client, including the `assessments` array.
      ```json
       {
         "id": 1,
         "name": "Client A",
         "assessments": [ ... ]
       }
      ```
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /clients/:id`

* **Description:** Updates the details of an existing client.
* **Method:** `PUT`
* **Path:** `/clients/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the client to update.
* **Request Body:**
    ```json
    {
      "name": "Updated Client Name"
    }
    ```
    * `name` (String, Optional): The new name for the client.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object representing the updated client, including assessments.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `DELETE /clients/:id`

* **Description:** Deletes a client. Fails if the client has associated assessments unless cascade delete is configured.
* **Method:** `DELETE`
* **Path:** `/clients/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the client to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `409` (Conflict - Assessments exist), `500`

---

## Assessments API (`/assessments`)

Manages assessment records associated with clients.

### `POST /assessments`

* **Description:** Creates a new assessment, linking it to an existing client.
* **Method:** `POST`
* **Path:** `/assessments`
* **Request Body:**
    ```json
    {
      "clientId": 1, // Required ID of the client this assessment belongs to
      "question1": "Answer 1",
      "question2": "Answer 2",
      // ... other assessment fields (question3-10, phone, email, name, isCompleted)
    }
    ```
    * `clientId` (Integer, **Required**): ID of the associated `Client`.
    * Other fields (String/Boolean, Optional): Assessment details. Defaults to empty strings/false if not provided.
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new assessment, including the associated `client` object.
      ```json
      {
        "id": 101,
        "clientId": 1,
        "question1": "Answer 1", ...,
        "client": { "id": 1, "name": "Client A" }
      }
      ```
* **Error Responses:** `400` (Missing `clientId`), `404` (Client not found), `500`

### `GET /assessments`

* **Description:** Retrieves a list of all assessments, including their associated client.
* **Method:** `GET`
* **Path:** `/assessments`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of assessment objects, each including a `client` object.
      ```json
      [
        { "id": 101, "clientId": 1, ..., "client": { ... } },
        { "id": 102, "clientId": 1, ..., "client": { ... } }
      ]
      ```
* **Error Responses:** `500`

### `GET /assessments/:id`

* **Description:** Retrieves a single assessment by ID, including its associated client.
* **Method:** `GET`
* **Path:** `/assessments/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the assessment.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the assessment, including the `client` object.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /assessments/:id`

* **Description:** Updates an existing assessment. Cannot change the `clientId`.
* **Method:** `PUT`
* **Path:** `/assessments/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the assessment to update.
* **Request Body:**
    ```json
    {
      "question1": "Updated Answer 1",
      "isCompleted": true
      // ... other updatable assessment fields
    }
    ```
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated assessment, including the `client` object.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `DELETE /assessments/:id`

* **Description:** Deletes an assessment.
* **Method:** `DELETE`
* **Path:** `/assessments/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the assessment to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

---

## Nodes API (`/nodes`)

Manages network topology nodes (e.g., devices, locations in a graph).

### `POST /nodes`

* **Description:** Creates a new node.
* **Method:** `POST`
* **Path:** `/nodes`
* **Request Body:**
    ```json
    {
      "positionX": 100.5,
      "positionY": 200.0,
      "label": "Optional Node Label",
      "type": "Optional Node Type",
      "data": { "custom": "value" } // Optional JSON data
    }
    ```
    * `positionX` (Float, **Required**): X coordinate.
    * `positionY` (Float, **Required**): Y coordinate.
    * `label` (String, Optional)
    * `type` (String, Optional)
    * `data` (JSON, Optional)
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new node.
      ```json
       {
         "id": "clpsl48k3000008l1fh0t9rjw", // Example CUID
         "label": "Optional Node Label",
         "type": "Optional Node Type",
         "data": { "custom": "value" },
         "positionX": 100.5,
         "positionY": 200.0,
         "createdAt": "...",
         "updatedAt": "..."
       }
      ```
* **Error Responses:** `400` (Missing required fields), `500`

### `GET /nodes`

* **Description:** Retrieves a list of all nodes, including incoming and outgoing edges.
* **Method:** `GET`
* **Path:** `/nodes`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of node objects, each including `edgesFrom` and `edgesTo` arrays.
      ```json
      [
        {
          "id": "...", "label": "Node A", ...,
          "edgesFrom": [ { "id": "edge1", "sourceId": "...", "targetId": "...", ... } ],
          "edgesTo": []
        },
        { ... }
      ]
      ```
* **Error Responses:** `500`

### `GET /nodes/:id`

* **Description:** Retrieves a single node by its ID (CUID), including edges.
* **Method:** `GET`
* **Path:** `/nodes/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the node.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the node, including `edgesFrom` and `edgesTo`.
* **Error Responses:** `404` (Not Found), `500`

### `PUT /nodes/:id`

* **Description:** Updates an existing node.
* **Method:** `PUT`
* **Path:** `/nodes/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the node to update.
* **Request Body:**
    ```json
    {
      "positionX": 110.0,
      "label": "Updated Label"
      // ... other updatable fields
    }
    ```
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated node, including edges.
* **Error Responses:** `404` (Not Found), `500`

### `DELETE /nodes/:id`

* **Description:** Deletes a node. **Important:** Requires manually deleting connected edges first.
* **Method:** `DELETE`
* **Path:** `/nodes/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the node to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `404` (Not Found), `500` (Potentially others if edge deletion fails).

---

## Edges API (`/edges`)

Manages network topology edges (connections between nodes).

### `POST /edges`

* **Description:** Creates a new edge connecting two existing nodes.
* **Method:** `POST`
* **Path:** `/edges`
* **Request Body:**
    ```json
    {
      "sourceId": "clpsl48k3000008l1fh0t9rjw", // CUID of the source node
      "targetId": "clpsl48z3000108l19g7b7hdx", // CUID of the target node
      "label": "Optional Edge Label",
      "type": "Optional Edge Type"
    }
    ```
    * `sourceId` (String, **Required**): CUID of the source `Node`.
    * `targetId` (String, **Required**): CUID of the target `Node`.
    * `label` (String, Optional)
    * `type` (String, Optional)
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new edge, including `source` and `target` node objects.
      ```json
      {
        "id": "clxyz...", "label": "...", "type": "...",
        "sourceId": "...", "targetId": "...",
        "source": { "id": "...", "label": "Node A", ... },
        "target": { "id": "...", "label": "Node B", ... },
        "createdAt": "...", "updatedAt": "..."
      }
      ```
* **Error Responses:** `400` (Missing IDs), `404` (Source or Target node not found), `500`

### `GET /edges`

* **Description:** Retrieves a list of all edges, including their source and target nodes.
* **Method:** `GET`
* **Path:** `/edges`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of edge objects, each including `source` and `target`.
* **Error Responses:** `500`

### `GET /edges/:id`

* **Description:** Retrieves a single edge by its ID (CUID), including source and target nodes.
* **Method:** `GET`
* **Path:** `/edges/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the edge.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the edge, including `source` and `target`.
* **Error Responses:** `404` (Not Found), `500`

### `PUT /edges/:id`

* **Description:** Updates an existing edge (typically only label or type). Changing source/target is usually done by deleting and recreating.
* **Method:** `PUT`
* **Path:** `/edges/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the edge to update.
* **Request Body:**
    ```json
    {
      "label": "Updated Edge Label",
      "type": "Updated Type"
    }
    ```
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated edge, including `source` and `target`.
* **Error Responses:** `404` (Not Found), `500`

### `DELETE /edges/:id`

* **Description:** Deletes an edge.
* **Method:** `DELETE`
* **Path:** `/edges/:id`
* **URL Parameters:**
    * `:id` (String, **Required**): The CUID of the edge to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `404` (Not Found), `500`

---

## ASNs API (`/asns`)

Manages Autonomous System Number (ASN) records.

### `POST /asns`

* **Description:** Creates a new ASN record.
* **Method:** `POST`
* **Path:** `/asns`
* **Request Body:**
    ```json
    {
      "asn": 65001,
      "name": "ASN 65001 Description"
    }
    ```
    * `asn` (Integer, **Required**): The ASN number (must be unique).
    * `name` (String, **Required**): A descriptive name for the ASN.
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new ASN.
* **Error Responses:** `400` (Missing fields, invalid ASN), `409` (ASN already exists), `500`

### `GET /asns`

* **Description:** Retrieves a list of all ASNs, including their associated BGP sessions.
* **Method:** `GET`
* **Path:** `/asns`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of ASN objects, each including a `bgp` array.
      ```json
      [
        {
          "asn": 65001, "name": "...", "createdAt": "...", "updatedAt": "...",
          "bgp": [ { "id": 1, "localAsn": ..., "peerAsn": 65001, ... }, ... ]
        },
        { ... }
      ]
      ```
* **Error Responses:** `500`

### `GET /asns/:asn`

* **Description:** Retrieves a single ASN by its number, including BGP sessions.
* **Method:** `GET`
* **Path:** `/asns/:asn`
* **URL Parameters:**
    * `:asn` (Integer, **Required**): The ASN number.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the ASN, including the `bgp` array.
* **Error Responses:** `400` (Invalid ASN), `404` (Not Found), `500`

### `PUT /asns/:asn`

* **Description:** Updates the name of an existing ASN.
* **Method:** `PUT`
* **Path:** `/asns/:asn`
* **URL Parameters:**
    * `:asn` (Integer, **Required**): The ASN number to update.
* **Request Body:**
    ```json
    {
      "name": "Updated ASN Name"
    }
    ```
    * `name` (String, **Required**): The new name.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated ASN, including BGP sessions.
* **Error Responses:** `400` (Invalid ASN, missing name), `404` (Not Found), `500`

### `DELETE /asns/:asn`

* **Description:** Deletes an ASN. Fails if BGP sessions reference it.
* **Method:** `DELETE`
* **Path:** `/asns/:asn`
* **URL Parameters:**
    * `:asn` (Integer, **Required**): The ASN number to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ASN), `404` (Not Found), `409` (Conflict - BGP sessions exist), `500`

---

## Devices API (`/devices`)

Manages network device records (switches, routers).

### `POST /devices`

* **Description:** Creates a new device record.
* **Method:** `POST`
* **Path:** `/devices`
* **Request Body:**
    ```json
    {
      "name": "core-switch-01",
      "ip": "192.168.1.1"
    }
    ```
    * `name` (String, **Required**): The name of the device.
    * `ip` (String, **Required**): The unique IP address of the device.
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new device.
* **Error Responses:** `400` (Missing fields), `409` (IP already exists), `500`

### `GET /devices`

* **Description:** Retrieves a list of all devices, including their BGP sessions and VSIs.
* **Method:** `GET`
* **Path:** `/devices`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of device objects, each including `bgp` and `vsis` arrays.
      ```json
      [
        {
          "id": 1, "name": "core-switch-01", "ip": "192.168.1.1", ...,
          "bgp": [ ... ],
          "vsis": [ ... ]
        },
        { ... }
      ]
      ```
* **Error Responses:** `500`

### `GET /devices/:id`

* **Description:** Retrieves a single device by ID, including BGP sessions and VSIs.
* **Method:** `GET`
* **Path:** `/devices/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the device.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the device, including `bgp` and `vsis`.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /devices/:id`

* **Description:** Updates an existing device's name or IP address.
* **Method:** `PUT`
* **Path:** `/devices/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the device to update.
* **Request Body:**
    ```json
    {
      "name": "Updated Device Name",
      "ip": "192.168.1.2"
    }
    ```
    * `name` (String, Optional)
    * `ip` (String, Optional)
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated device, including BGP sessions and VSIs.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `409` (IP conflict), `500`

### `DELETE /devices/:id`

* **Description:** Deletes a device. Fails if BGP sessions or VSIs reference it.
* **Method:** `DELETE`
* **Path:** `/devices/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the device to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `409` (Conflict - BGP/VSIs exist), `500`

---

## BGP Sessions API (`/bgp`)

Manages BGP peering session records.

### `POST /bgp`

* **Description:** Creates a new BGP session record.
* **Method:** `POST`
* **Path:** `/bgp`
* **Request Body:**
    ```json
    {
      "localAsn": 65000,
      "peerAsn": 65001,
      "peerIp": "10.0.0.1",
      "status": "Established",
      "deviceId": 1, // ID of the local device
      "asnId": 65001 // ASN number of the peer ASN record
    }
    ```
    * All fields (Integer/String, **Required**) as shown.
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new BGP session, including `device` and `asn` objects.
      ```json
      {
        "id": 1, "localAsn": 65000, "peerAsn": 65001, "peerIp": "10.0.0.1", "status": "Established", "deviceId": 1, "asnId": 65001, ...,
        "device": { "id": 1, "name": "...", "ip": "..." },
        "asn": { "asn": 65001, "name": "..." }
      }
      ```
* **Error Responses:** `400` (Missing fields, invalid numbers), `404` (Device or ASN not found), `409` (Session for device/peerIp exists), `500`

### `GET /bgp`

* **Description:** Retrieves a list of all BGP sessions, including the local device and peer ASN details.
* **Method:** `GET`
* **Path:** `/bgp`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of BGP session objects, each including `device` and `asn`.
* **Error Responses:** `500`

### `GET /bgp/:id`

* **Description:** Retrieves a single BGP session by ID, including device and ASN details.
* **Method:** `GET`
* **Path:** `/bgp/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the BGP session.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the BGP session, including `device` and `asn`.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /bgp/:id`

* **Description:** Updates the status of an existing BGP session.
* **Method:** `PUT`
* **Path:** `/bgp/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the BGP session to update.
* **Request Body:**
    ```json
    {
      "status": "Idle"
    }
    ```
    * `status` (String, **Required**): The new status.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated BGP session, including `device` and `asn`.
* **Error Responses:** `400` (Invalid ID, missing status), `404` (Not Found), `500`

### `DELETE /bgp/:id`

* **Description:** Deletes a BGP session record.
* **Method:** `DELETE`
* **Path:** `/bgp/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the BGP session to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

---

## VSIs API (`/vsi`)

Manages Virtual Switch Instance (VSI) records.

### `POST /vsi`

* **Description:** Creates a new VSI record associated with a device.
* **Method:** `POST`
* **Path:** `/vsi`
* **Request Body:**
    ```json
    {
      "deviceId": 1,
      "name": "CUSTOMER_A_VSI",
      "state": "up",
      "type": "static ldp",
      "mtu": 1600,
      "macLearning": "unqualify", // Or "N/A", "Enable", etc.
      "encapsulation": "vlan",    // Or "ethernet", etc.
      "vlanId": 100,            // Optional Integer
      "description": "Optional VSI description" // Optional String
    }
    ```
    * `deviceId`, `name`, `state`, `type`, `mtu`, `macLearning`, `encapsulation` (**Required**).
    * `vlanId`, `description` (Optional).
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new VSI, including `device` and `peers` (empty array initially).
      ```json
       {
         "id": 1, "deviceId": 1, "name": "CUSTOMER_A_VSI", ..., "vlanId": 100, ...,
         "device": { "id": 1, "name": "...", "ip": "..." },
         "peers": []
       }
      ```
* **Error Responses:** `400` (Missing fields, invalid numbers), `404` (Device not found), `409` (VSI name conflict on device), `500`

### `GET /vsi`

* **Description:** Retrieves a list of all VSIs, including their device and associated VSI peers.
* **Method:** `GET`
* **Path:** `/vsi`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of VSI objects, each including `device` and `peers`.
* **Error Responses:** `500`

### `GET /vsi/:id`

* **Description:** Retrieves a single VSI by ID, including device and peers.
* **Method:** `GET`
* **Path:** `/vsi/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the VSI, including `device` and `peers`.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /vsi/:id`

* **Description:** Updates an existing VSI. Cannot update `deviceId` or `name` easily.
* **Method:** `PUT`
* **Path:** `/vsi/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI to update.
* **Request Body:**
    ```json
    {
      "state": "down",
      "mtu": 1500,
      "description": "Updated description"
      // ... other updatable fields (type, macLearning, encapsulation, vlanId)
    }
    ```
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated VSI, including `device` and `peers`.
* **Error Responses:** `400` (Invalid ID, invalid numbers), `404` (Not Found), `500`

### `DELETE /vsi/:id`

* **Description:** Deletes a VSI. **Important:** Requires manually deleting associated VSI Peers first.
* **Method:** `DELETE`
* **Path:** `/vsi/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500` (Potentially 409 if peers still exist, depending on DB constraints).

---

## VSI Peers API (`/vsi-peers`)

Manages VSI Peer (Pseudowire) records associated with VSIs.

### `POST /vsi-peers`

* **Description:** Creates a new VSI Peer record, linking it to an existing VSI.
* **Method:** `POST`
* **Path:** `/vsi-peers`
* **Request Body:**
    ```json
    {
      "vsiId": 1, // Required ID of the parent VSI
      "peerAddress": "10.0.0.2",
      "pwId": 1001,
      "pwState": "up",
      "pwMacLearning": "N/A", // Or other status from device
      "pwInLabel": 3000,
      "pwOutLabel": 3001, // Integer or null
      "pwMacLimit": 500,    // Optional Integer or null
      "tunnelPolicy": "OptionalPolicyName" // Optional String or null
    }
    ```
    * `vsiId`, `peerAddress`, `pwId`, `pwState`, `pwMacLearning`, `pwInLabel` (**Required**).
    * `pwOutLabel` (Integer or `null`, Optional - but usually present).
    * `pwMacLimit`, `tunnelPolicy` (Optional).
* **Success Response:**
    * Code: `201 Created`
    * Body: JSON object of the new VSI Peer, including the parent `vsi` object.
      ```json
      {
        "id": 501, "vsiId": 1, "peerAddress": "10.0.0.2", "pwId": 1001, ..., "pwOutLabel": 3001, ...,
        "vsi": { "id": 1, "name": "CUSTOMER_A_VSI", ... }
      }
      ```
* **Error Responses:** `400` (Missing fields, invalid numbers), `404` (VSI not found), `409` (Peer already exists for VSI/Addr/PWID), `500`

### `GET /vsi-peers`

* **Description:** Retrieves a list of all VSI Peers, including their parent VSI.
* **Method:** `GET`
* **Path:** `/vsi-peers`
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON array of VSI Peer objects, each including a `vsi` object.
* **Error Responses:** `500`

### `GET /vsi-peers/:id`

* **Description:** Retrieves a single VSI Peer by ID, including its parent VSI.
* **Method:** `GET`
* **Path:** `/vsi-peers/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI Peer.
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the VSI Peer, including the `vsi` object.
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

### `PUT /vsi-peers/:id`

* **Description:** Updates an existing VSI Peer. Cannot easily change `vsiId`, `peerAddress`, or `pwId`.
* **Method:** `PUT`
* **Path:** `/vsi-peers/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI Peer to update.
* **Request Body:**
    ```json
    {
      "pwState": "down",
      "pwInLabel": 3002,
      "pwOutLabel": null, // Set remote label to null
      "pwMacLimit": 1000
      // ... other updatable fields
    }
    ```
* **Success Response:**
    * Code: `200 OK`
    * Body: JSON object of the updated VSI Peer, including the `vsi` object.
* **Error Responses:** `400` (Invalid ID, invalid numbers), `404` (Not Found), `409` (Update causes unique conflict), `500`

### `DELETE /vsi-peers/:id`

* **Description:** Deletes a VSI Peer record.
* **Method:** `DELETE`
* **Path:** `/vsi-peers/:id`
* **URL Parameters:**
    * `:id` (Integer, **Required**): The ID of the VSI Peer to delete.
* **Success Response:**
    * Code: `204 No Content`
* **Error Responses:** `400` (Invalid ID), `404` (Not Found), `500`

---
