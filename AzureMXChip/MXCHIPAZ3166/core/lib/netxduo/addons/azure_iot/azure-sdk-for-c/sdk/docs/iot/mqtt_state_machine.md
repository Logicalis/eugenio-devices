# Azure IoT Client MQTT State Machine

## High-level Architecture

Device Provisioning and IoT Hub service protocols require additional state management on top of the MQTT protocol. The Azure IoT Hub and Provisioning clients for C provide a common programming model. The clients must be layered on top of an MQTT client selected by the application developer.

The following aspects are being handled by the IoT Clients:

1. Generate MQTT CONNECT credentials.
1. Obtain SUBSCRIBE topic filters and PUBLISH topic strings required by various service features.
1. Parse service errors and output an uniform error object model.
1. Provide the correct sequence of events required to perform an operation.
1. Provide suggested timing information when retrying operations.

The following aspects need to be handled by the application or convenience layers:

1. Ensure secure TLS communication using either server or mutual X509 authentication.
1. Perform MQTT transport-level operations.
1. Delay execution for retry purposes.
1. (Optional) Provide real-time clock information and perform HMAC-SHA256 operations for SAS token generation.

For more information about Azure IoT services using MQTT see [this article](https://docs.microsoft.com/azure/iot-hub/iot-hub-mqtt-support).

## Components

### IoT Hub

![iot_hub_flow](https://www.plantuml.com/plantuml/svg/0/jLTjRzis4FxENt7VcWppDXHhWHb5K5anPgz9x4KADgme25gwsaGaKYMff5wn_pvIBxRZYBAzTlbW1EBTynmVtXsFRxLXofHvHeY-vw9WYkLWlnc4BmJituWbzqibIv66CfFgpPjWFh-u0FjxDGs3U3gxwJQBujkxCBQML-m1HOgA_6CfAk34MT14fbmi6vPw8RfyHuFvTE_BPH07T3RwRHp6iCNTCTg9XOQpop4qGJh65p0Lt93ttts0mU02sD-KIqDNvO8c6KTXABVBGuZYu3OctQlGwRT4GqkDYWgM8pcmxeTeUL9oOBs2A8gC9ynmAMZ-oTXLAGGcDlu_N_tq2uILBGIbMGsXO5e_IfK2ru1vOTQD-3wCZMXMibdXbba6KH21aVSeO8a2LUKrF2hIPoQQwSfCywLWfbG8mmgUsMZ9CUPx-r_bB7dvwZNf-DOVG2iCEMnIfunFiASQiwrO7KgtaUs876E6EHgr9bw6GTQv4zDwri6MYuph5JMou8aDQeCB2HEmGB1PcCBZZn9qkT29uWjNTaO24sked7uWft5qgjAOa9xTTBcrCI7f2C86i9AffWfr8ON6huzbG-Vf1kQH2vhAfCpRj7v2Hqxs-5y2B9X9LiT7vp4_4DeB26MUIm7R8_9qa6tCRNhtRQk3Ks62_C5rXElVunWoljYIfiH7sqIwhIcFNOlUNbrVmEsDPTd2tbcGsaHVMcjR4oicqWpLiBfQ_3T6yZAraYgRZWrokgczs-O3pEz6LzDe0ZbgpPPO8Pw0r4tSbeE7N4W35X4KPbQ7rCFP2zARSrxI1l80iH_5iJgvQpHdju5o9_NoycJvVCUcy5E9shLk9rHYSm1Qruh3qbtNcnd-9_dyfxP3NN_eO3_ekryxzMgTjZlpaDTcKoeLczUgD_dylRXLCBO7U7eEU1El9RJALQdnwpW6gM0UnE5agryzwiaO9kK2QlUXRvuRmW3QxokfZMU2szYc-C0JT3EQvN2tVEnkUKsE6t86zL10yNF1wmcAOY5jGpQyatP_27488wRnzSzda-iW74IJm4uIPnkAxo5QCOLfa2IssRp3zVdvaQZtkAwdLtBgZ37eEAn2oGALuQWtdVXGFkzCyuTi7MVpR_Um_DpSjxN3cNmasI48fqBPeyPkH-gLucdhEYJ-wJFAtdvFdIFstNtxVLhElJlUhtn7q4XjrWKUwT5pWi_DczCwnlG1bRPrxDf1iEsYm8L1zP7PCYGzZmRBXCeYqC_spKQDAhMzwAYIqlHxlM_2gTmITzWVdIr-w9iYdUEPR9vpvn3yHQUpc4xkHZIycevlOcflLnhTZQBmTZeQqtWsduPV0tSzZYUpo-XRGiP_J6_exJYQpswYyJS7hgwHPwYThygcPS9PpjRkNVN8pShUH_QHDOckpmbvYo8jy-nV "iot_hub_flow")

### Device Provisioning Service

![iot_provisioning_flow](https://www.plantuml.com/plantuml/svg/0/hLTjRzis4FxENy5ljOUv68Qrm8mYgEmOZTOazYnb65OK12tICc69L4dAZLVil-yeoR8RlqYscmz68DuzTtpk-F3utbYgRPuc29cba1dLDCNmpNBy3M7u6z1e6MkLIpPKYTBltdcJoSqDXjzxV-Y5_lZni5aGJf-68LZUqnMNZ6lq7uGeW2DdGIB5X8ohAEL2SFkHU1F_nPTRB_J8UF37Q1ZYBUwEqKukP3Y-7U4gIHVn5VQbiEZJJvznhtU3wRz4A4iohLR222KX0n8bWajB1DZrSH1wqn0rc3L1nToIt71D94qvQclO1dMJ1CDz_FURNtx7DBUBYbQQPOc8g0KladwnOAoav96jwMYDNIHo1CrHEixSPvhWm0kmLKikSSmbruLr_umu9_Thg2diZOWbjRdci6SNiDZoMNuljhD8QVJhcxVemBUMAMNKcAzACtI6xKbMOMirvdurDybj2Wab2FAPYRIOSNGKyc0yK_iTQPhQe8xsweJDmHYQHroiNBaU_Wn3d8WPhZsOItxAOs4iI48Jyd5sTGLHKfpF4c4MnAfCHEWYhyCxgKn-96H5pZ7wAuHjgO_ORJZUwF4t3BmMT4-U1hXQPOL8jj0-a7t3DvYiJ9zGoM1XdIZA-llGUAvLYcSQJh_6dIXHESDAun6BURS8UqMuriNNt6HA6lv30ZgU-bWgYtSXOCC9L03wjY76NbBjI5TPfPl4LJetgDrUNher1TlQHohD4cuRETlts66fNiJrOiVaKSIDSLqmfo81uUrU66EtsndGl_ukef2kG37GR7Q6W7sVpH4gSjXRRU3hIyxrkkBsWCOqxEkqzscjAOQMoMu7bg3zngcaZtiuJjIC9WGcham9M5WeOvkcaDfPC68Y8BgLSTdlLVb4eUlkTWxTE63GcVTwhnmZ27_EH0MPI_5Ch83KqJKdkyHkk1TQeLJ_rv9yZXJipArd-RcubIwSxRSzN89TutrfVCqj708wTmk6mrJ2BWBPwJ9OoPFJ1-4GoklNFGp35RGRr7QHpreTYcD0ZAz4n2-Wr3bphSkNpqMX-RDrBob6heGfl5EGJnfV0G15YPRE-XegtFc6_aepTCuexHfZG0r_-rldxi4Rze8x3yRZJYqeF-DWGJLdNd4XVUJhRlTS-2LVbwytTBdCxnr2a4XlA_jmaG4lSmeveXFl0X00hbT0u3Eyzu3r7DXaH2jbJtjzgf8it2AkhPWxM_xvnDJj_Bn_Y3yEoMGw_iiFu4djFtuWmtj_z1YIwUY37uIZAGcU1i5mwWz691nF_24wT65_9m07treLpxYJWWEC_5-5NS3bJYM-2nww2Bs7Ri_Jv7y0 "iot_provisioning_flow")

## Porting the IoT Clients

In order to port the clients to a target platform the following items are required:

- Support for a C99 compiler.
- Types such as `uint8_t` must be defined.
- The target platform supports a stack of several kB (actual requirement depends on features being used and data sizes).
- An MQTT over TLS client supporting QoS 0 and 1 messages.

Optionally, the IoT services support MQTT tunneling over WebSocket Secure which allows bypassing firewalls where port 8883 is not open. Using WebSockets also allows usage of devices that must go through a WebProxy. Application developers are responsible with setting up the wss:// tunnel.

## API

### Connecting

The application code is required to initialize the TLS and MQTT stacks.
Two authentication schemes are currently supported: _X509 Client Certificate Authentication_ and _Shared Access Signature_ authentication.

When X509 client authentication is used, the MQTT password field should be an empty string.

If SAS tokens are used the following APIs provide a way to create as well as refresh the lifetime of the used token upon reconnect.

_Example:_

```C
if(az_result_failed(az_iot_hub_client_sas_get_signature(client, unix_time + 3600, signature, &signature)));
{
    // error.
}

// Application will Base64Encode the HMAC256 of the az_span_ptr(signature) containing az_span_size(signature) bytes with the Shared Access Key.

if(az_result_failed(az_iot_hub_client_sas_get_password(client, NULL, base64_hmac_sha256_signature, password, password_size, &password_length)))
{
    // error.
}
```

Recommended defaults:
    - MQTT Keep-Alive Interval:  `AZ_IOT_DEFAULT_MQTT_CONNECT_KEEPALIVE_SECONDS`
    - MQTT Clean Session: false.

#### MQTT Clean Session

We recommend to always use [Clean Session](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718030) false when connecting to IoT Hub.
Connecting with Clean Session true will remove all enqueued C2D messages.

### Subscribe to Topics

Each service requiring a subscription implements a function similar to the following:

_Example:_

```C
// AZ_IOT_HUB_CLIENT_METHODS_SUBSCRIBE_TOPIC contains the methods topic filter.
MQTTClient_subscribe(mqtt_client, AZ_IOT_HUB_CLIENT_METHODS_SUBSCRIBE_TOPIC, 1);
```

__Note:__ If the MQTT stack allows, it is recommended to subscribe prior to connecting.

### Sending APIs

Each action (e.g. send telemetry, request twin) is represented by a separate public API.
The application is responsible for filling in the MQTT payload with the format expected by the service.

_Example:_

```C
if(az_result_failed(az_iot_hub_client_telemetry_get_publish_topic(client, NULL, topic, topic_size, NULL)))
{
    // error.
}
```

__Note:__ To limit overheads, when publishing, it is recommended to serialize as many MQTT messages within the same TLS record. This feature may not be available on all MQTT/TLS/Sockets stacks.

### Receiving APIs

We recommend that the handling of incoming MQTT PUB messages is implemented by a chain-of-responsibility architecture. Each handler is passed the topic and will either accept and return a response, or pass it to the next handler.

_Example:_

```C
    az_iot_hub_client_c2d_request c2d_request;
    az_iot_hub_client_method_request method_request;
    az_iot_hub_client_twin_response twin_response;

    //az_span received_topic is filled by the application.

    if (az_result_succeeded(az_iot_hub_client_c2d_parse_received_topic(client, received_topic, &c2d_request)))
    {
        // This is a C2D message:
        //  c2d_request.properties contain the properties of the message.
        //  the MQTT message payload contains the data.
    }
    else if (az_result_succeeded(ret = az_iot_hub_client_methods_parse_received_topic(client, received_topic, &method_request)))
    {
        // This is a Method request:
        //  method_request.name contains the method
        //  method_request.request_id contains the request ID that must be used to submit the response using az_iot_hub_client_methods_response_get_publish_topic()
    }
    else if (az_result_succeeded(ret = az_iot_hub_client_twin_parse_received_topic(client, received_topic, &twin_response)))
    {
        // This is a Twin operation.
        switch (twin_response.response_type)
        {
            case AZ_IOT_CLIENT_TWIN_RESPONSE_TYPE_GET:
                // This is a response to a az_iot_hub_client_twin_document_get_publish_topic.
                break;
            case AZ_IOT_CLIENT_TWIN_RESPONSE_TYPE_DESIRED_PROPERTIES:
                // This is received as the Twin desired properties were changed using the service client.
                break;
            case AZ_IOT_CLIENT_TWIN_RESPONSE_TYPE_REPORTED_PROPERTIES:
                // This is a response received after patching the reported properties using az_iot_hub_client_twin_patch_get_publish_topic().
                break;
            default:
                // error.
        }
    }
```

__Important:__ C2D messages are not enqueued until the device establishes the first MQTT session (connects for the first time to IoT Hub). The C2D message queue is preserved (according to the per-message time-to-live) as long as the device connects with Clean Session `false`.

### Retrying Operations

Retrying operations requires understanding two aspects: error evaluation (did the operation fail, should the operation be retried) and retry timing (how long to delay before retrying the operation). The IoT client library is supplying optional APIs for error classification and retry timing.

#### Error Policy

The SDK will not handle protocol-level (WebSocket, MQTT, TLS or TCP) errors. The application-developer is expected to classify and handle errors the following way:

- Operations failing due to authentication errors should not be retried.
- Operations failing due to communication-related errors other than ones security-related (e.g. TLS Alert) may be retried.

Both IoT Hub and Provisioning services will use `MQTT CONNACK` as described in Section 3.2.2.3 of the [MQTT v3 specification](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Table_3.1_-).

##### IoT Service Errors

APIs using `az_iot_status` report service-side errors to the client through the IoT protocols.

The following APIs may be used to determine if the status indicates an error and if the operation should be retried:

```C
az_iot_status status = response.status;
if (az_iot_status_succeeded(status))
{
    // success case
}
else
{
    if (az_iot_status_retriable(status))
    {
        // retry
    }
    else
    {
        // fail
    }
}
```

#### Retry Timing

Network timeouts and the MQTT keep-alive interval should be configured considering tradeoffs between how fast network issues are detected vs traffic overheads. [This document](https://docs.microsoft.com/azure/iot-hub/iot-hub-mqtt-support#default-keep-alive-timeout) describes the recommended keep-alive timeouts as well as the minimum idle timeout supported by Azure IoT services.

For connectivity issues at all layers (TCP, TLS, MQTT) as well as cases where there is no `retry-after` sent by the service, we suggest using an exponential back-off with random jitter function. `az_iot_retry_calc_delay` is available in Azure IoT Common:

```C
// The previous operation took operation_msec.
// The application calculates random_jitter_msec between 0 and max_random_jitter_msec.

int32_t delay_msec = az_iot_calculate_retry_delay(operation_msec, attempt, min_retry_delay_msec, max_retry_delay_msec, random_jitter_msec);
```

_Note 1_: The network stack may have used more time than the recommended delay before timing out. (e.g. The operation timed out after 2 minutes while the delay between operations is 1 second). In this case there is no need to delay the next operation.

_Note 2_: To determine the parameters of the exponential with back-off retry strategy, we recommend modeling the network characteristics (including failure-modes). Compare the results with defined SLAs for device connectivity (e.g. 1M devices must be connected in under 30 minutes) and with the available [IoT Azure scale](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-quotas-throttling) (especially consider _throttling_, _quotas_ and maximum _requests/connects per second_).

In the absence of modeling, we recommend the following default:

```C
    min_retry_delay_msec =     1000;
    max_retry_delay_msec =   100000;
    max_random_jitter_msec =   5000;
```

For service-level errors, the Provisioning Service is providing a `retry-after` (in seconds) parameter:

```C
// az_iot_provisioning_client_received_topic_payload_parse was successful and created a az_iot_provisioning_client_register_response response

int32_t delay_ms;
if ( response.retry_after_seconds > 0 )
{
    delay_ms = response.retry_after_seconds;
}
else
{
    delay_ms = az_iot_calculate_retry_delay(operation_msec, attempt, min_retry_delay_msec, max_retry_delay_msec, random_jitter_msec);
}
```

#### Suggested Retry Strategy

Combining the functions above we recommend the following flow:

![iot_retry_flow](https://www.plantuml.com/plantuml/svg/0/bPNDQkCm4CVlUegvBHIy3n1AQ3RBie7G13TxMefHd6agwaf6abEwfUzUsObSoHedsnnya7xpd-_enbYkRVDSCVCaPCqrVmPtP17U6BZV3ru-xRLgv6wkAgMlhsVhzNGAxhjSp6URnUgMnkus-P_vnf5BVa2vGqrZlsQBfODMciizidV6_bxTGvPDOQtLGHYXf91xTWmeF08Vo1lhX8z4ZdjXBEhY9nv4YHxgY6-nVOvM2xwj451hfKt7UES3dT15A59eBr8yS54r6dr2dS4mcc5QgNbdTXv9LKfUbKtbOYjsMF7NL6C0f0gyhWDR8iyU20jA4rJzO09j7fT3cq3cI5ChQV1xPvBn1tkQdKk6_5yXbEqgzjhT1pbH4t2hPDQN5_wlO_Ba8EqQKJKIZYRaCfw6aAlMKp7Nk4Df1Q_CcF-KXD7-4UoN6ZbYtoxK1AG2PHzHGzbVitTWB6f7Yo_KflZTR9t9NLEMQ0mxhRw_-DpwpvpTUR6gKNFhbA8C_Jf713lDGYj7_Wd4Ujx-NDV9-wZH9D5hKnjCdFSyjQ_HULY4w28jHzHImkcvpGegIImJNSTB6pJA9FKStvVZrEMOrNx0sfV5pz1menowsbek94Xy2KRK3T-DUxdSq_W1 "iot_retry_flow")

When devices are using IoT Hub without Provisioning Service, we recommend attempting to rotate the IoT Credentials (SAS Token or X509 Certificate) on authentication issues.

_Note:_ Authentication issues observed in the following cases do not require credentials to be rotated:

- DNS issues (such as WiFi Captive Portal redirects)
- WebSockets Proxy server authentication
