# Samples

This directory contains a sample AWS Greengrass PubSub SDK component template that you can use as a skeleton framework to start developing your own AWS Greengrass custom components.

## AWS Greengrass Development Kit
The sample component is built and deployed using the [AWS Greengrass Development Kit](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-development-kit-cli.html), a development tool to create, test, build, publish, and deploy custom Greengrass components.

## Deploying the AWS Greengrass PubSub SDK Component Template 

In the following steps we will copy the AWS Greengrass PubSub SDK component template. Then using the AWS Greengrass Development Kit will build a versioned deployment artifact, publish it to AWS IoT Core and finally deploy it to a remote AWS Greengrass core device. 

### Prerequisites
* An AWS Account, see [How to Create a new AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) if needed.
* A registered [AWS Greengrass V2 core device](https://docs.aws.amazon.com/greengrass/v2/developerguide/setting-up.html)
* Knowledge of [AWS Greengrass Components](https://docs.aws.amazon.com/greengrass/v2/developerguide/create-components.html) and the [AWS Greengrass Developer Guide](https://docs.aws.amazon.com/greengrass/v2/developerguide).
* The [AWS Greengrass Development Kit](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-development-kit-cli.html) installed on the development machine. 
    *  The GDK can be downloaded from the [GDK Repo](https://github.com/aws-greengrass/aws-greengrass-gdk-cli)

### Clone and Copy the Component Template

```
# Clone this GIT Repository
git clone https://github.com/aws-samples/aws-greengrass-pubsub-sdk-for-python.git

# Take a copy of the selected component template and rename to your desired component name.

cp -Rf aws-greengrass-pubsub-sdk-for-python/samples/gg-pubsub-sdk-component-template com.example.greengrass-pubsub-component

# CD into the template component source directory
cd com.example.greengrass-pubsub-component/src
```

### Update the GDK Configuration

Open the src/gdk-config.json config file of the sample component and update the fields in square brackets accordingly.  
**Note:** Remove the brackets / keep the quotes. i.e "[ENTER COMPONENTS NAME]" => "com.example.greengrass-pubsub-sdk"
```
{
  "component" :{
    "[ENTER COMPONENTS NAME]": {
      "author": "[ENTER AUTHORS NAME]",
      "version": "NEXT_PATCH",
      "build": {
        "build_system" :"zip"
      },
      "publish": {
        "bucket": "[ENTER S3 BUCKET FOR DEPLOYMEMT ARTIFACTS]",
        "region": "[ENTER AWS REGON TO PUBLISH COMPONENT]"
      }
    }
  },
  "gdk_version": "1.0.0"
}
```

### Update the Component Recipe
The AWS Greengrass component recipe provides metadata, configuration parameters and PubSub access policies for an AWS Greengrass V2 Component. In the src/recipe.json config file, the **ComponentName** field and the PubSub access policy names must be globally unique on a individual Greengrass Core devices. In the template recipe file, these and the **base-pubsub-topic** fields are set to **COMPONENT_NAME**. 

* Using your preferred IDE or text editor, open the components src/recipe.json file then Find and Replace **COMPONENT_NAME** with the component name used when you copied the template (i.e: **com.example.greengrass-pubsub-component**)

* Update the **ComponentDescription** and **ComponentPublisher** as desired.

Update the **GGV2PubSubSdkConfig** field that enables data injection of application parameters, in particular:
* **base-pubsub-topic**: The base of the PubSub topic schema to be used by the application. This is typically set to the component name and will have been updated if you completed the text Fine and Replace as described above.
* **ipc-subscribe-topics**: An array of user defined PubSub topics the component should subscribe to on the IPC bus.
* **mqtt-subscribe-topics**: An array of user defined PubSub topics the component should subscribe to on the MQTT bus.  

These are passed to the AWS Greengrass component on start-up and can be access programmatically. 
i.e:
```
    "GGV2PubSubSdkConfig": {
        "base-pubsub-topic" : "com.example.greengrass-pubsub-component",
        "ipc-subscribe-topics" : ["ipc/my-app/broadcast", "ipc/my-app/error"],
        "mqtt-subscribe-topics" : ["mqtt/my-app/broadcast", "mqtt/my-app/error"]
    }
```
In this example, we have defined application broadcast and error PubSub topics that may be used to provide differentiated processing based on priority.

You can also add you own parameters in the **GGV2PubSubSdkConfig** field and have them passed into your application logic using the patterns shown in the component template.

### Build and Publish the Component to AWS IoT Core

```
# Build the component:
gdk component build

# Publish the component to AWS IoT Core:
gdk component publish

```

The AWS Greengrass component will now be published to the AWS IoT Core. You can verify in the [AWS IoT Console](https://console.aws.amazon.com/iot/) by selecting your **region** then, selecting **Greengrass** and the **Components** menu as shown below:

![published-component](../images/gg-component-published.png)

### Deploy the Component to an AWS Greengrass Core Device

**Note:** The AWS Greengrass Core device will need permissions to access the S3 bucket with the deployment artifact listed in  src/gdk-config.json as described above. See [Authorize core devices to interact with AWS services](https://docs.aws.amazon.com/greengrass/v2/developerguide/device-service-role.html) in the AWS Greengrass Developer guide for details. 

The final step is to deploy the component to a registered AWS Greengrass Core device:
* In the [AWS IoT Console](https://console.aws.amazon.com/iot/), select **Greengrass** and then the **Core devices** menu item and click on the Greengrass core you will deploy the component on.

* Select the **Deployments** tab and click on the managed deployment to add this component too (You may have to create one if it doesn't exist).
* Click **Revise**, **Revise Deployment** then **Next** and select the name of the component you published.
* Click **Next** leaving all fields default until the final page then click **Deploy**

## Validate the Component

Once the template component and SDK are installed and operational on an AWS Greengrass core device, it will periodically publish a local time message on the devices egress topic. It will also process and respond to well-formatted request messages as described below.

### Verify the Published Time Message

In the [AWS IoT Console](https://console.aws.amazon.com/iot/), select your desired **region** and then **Test** from the menu and subscribe to the components **Egress** topic.  

The Egress topic will be: **[base-pubsub-topic]/[THING_NAME]/egress**  

Where:  
* **base-pubsub-topic** is the value provided for this field in the component recipe (Typically set to the components name) and 
* **[THING_NAME]** is the AWS IoT Thing name associated with the AWS Greengrass core device.

You should see a periodic message being received from the device as below:
```
{
  "sdk_version": "0.1.4",
  "message_id": "20220329121327002764",
  "status": 200,
  "route": "default_message_handler",
  "message": {
    "local-time": "29-Mar-2022 10:13:27"
  }
}
```

For reference, this is generated in the main class [service_loop](/samples/gg-pubsub-sdk-component-template/src/main.py) as per the below code sample:
```
    def service_loop(self):
        '''
        Holds the process up while handling event-driven PubSub triggers.
        Put synchronous application logic here or have the component completely event driven.
        
        This example periodically publishes the local time to IPC and MQTT in a well formatted message.
        '''
        
        while True:
            try:
                # Build and publish a well formatted message displaying local time to IPC and MQTT
                receive_route = "local_time_update"
                my_message = { "local-time" : datetime.now().strftime("%d-%b-%Y %H:%M:%S")  }
                sdk_format_msg = self.message_formatter.get_message(route=receive_route, message=my_message)
                log.info('Publishing message: {}'.format(sdk_format_msg))
                self.publish_message('ipc_mqtt', sdk_format_msg)
            
            except Exception as err:
                # Publish error to IPC and MQTT on default error topic. 
                protocol = 'ipc_mqtt'
                err_msg = 'Exception in main process loop: {}'.format(err)
                self.publish_error(protocol, err_msg)
                
            finally:
                time.sleep(10)
```

### Verify Custom Message Handlers

The template component registers two user defined message handlers. This is to demonstrate how the SDK can be used to route PubSub messages but also how to separate PubSub message processing code by function.

The example message handlers are:
1. [MySystemMessageHandler](/samples/gg-pubsub-sdk-component-template/src/pubsub_message_handlers/my_system_message_handler.py): An example message handler to process system type requests to the custom component including get_health_check_request and get_system_details_request calls

2. [MySensorMessageHandler](/samples/gg-pubsub-sdk-component-template/src/pubsub_message_handlers/my_system_message_handler.py): An example message handler to process data requests to connected sensors or IoT Things including a get_temp_sensor_request call. 

These are registered with the SDK in the main __init__ method:
```
  self.my_system_message_handler = MySystemMessageHandler(self.publish_message, self.publish_error, self.self.message_formatter)
  self.pubsub_client.register_message_handler(self.my_system_message_handler)

  self.my_sensor_message_handler = MySensorMessageHandler(self.publish_message, self.publish_error, self.message_formatter)
  self.pubsub_client.register_message_handler(self.my_sensor_message_handler)
```

While not compulsory, it’s a common pattern for message handlers to operate in a request / response pattern. To make creating well formatted and publishing response messages easier, a reference to the SDK client publish_message, pubish_error and message formatter call-backs is provided to the sample message handler classes. 

### Verify the health-check-request function

In the same AWS IoT Console test facility, publish the below health_check_request message to the component’s ingress topic.

The Ingress topic will be: **[base-pubsub-topic]/[THING_NAME]/ingress**  

Where:  
* **base-pubsub-topic** is the value provided for this field in the component recipe (Typically set to the components name) and 
* **[THING_NAME]** is the AWS IoT Thing name associated with the AWS Greengrass core device.

```
{
  "sdk_version": "0.1.4",
  "message_id": "20220329121327002764",
  "status": 200,
  "route": "MySystemMessageHandler.get_health_check_request",
  "message": { }
}
```

You should receive the below response on the components egress topic. 
```
{
  "sdk_version": "0.1.4",
  "message_id": "20220329121327002764",
  "status": 200,
  "route": "MySystemMessageHandler.get_health_check_response",
  "message": {
    "status": "System OK"
  }
}
```

This is virtue of the SDK routing the request message to the get_health_check_request method in the registered MySystemMessageHandler message handler class. This method is shown below:

```
def get_health_check_request(self, protocol, topic, message_id, status, route, message):
    '''
    [Removed for brevity]
    '''

    try:
        # Log just for Dev / Debug.
        log.info('get_health_check_request received on protocol: {} - topic: {}'.format(protocol, topic))

        # Create a response message using the Greengrass PubSub SDK prefered message_formatter.
        # Reflect the request message_id for tracking, status and other fields as defeult.
        response_route = "MySystemMessageHandler.get_health_check_response"
        msg = {"status" : "System OK"}
        response =  self.message_formatter.get_message(message_id=message_id, route=response_route, message=msg)

        # Publish the message on the protocol (IPC or MQTT) that it was received to default egress topic.
        self.publish_message(protocol=protocol, message=response)

    except Exception as err:
        # Publish error to default error route, will log locally as an error as well. 
        err_msg = 'Exception in get_health_check_request: {}'.format(err)
        self.publish_error(protocol, err_msg)
```

### Validate Supported Message Processors
The template component supports the following PubSub message route values:
1. MySystemMessageHandler.get_health_check_request
2. MySystemMessageHandler.get_system_details_request
3. MySensorMessageHandler.get_temp_sensor_request

Repeat the previous step, replacing the request message **route** value with the above supported values and validate the response. 

## Develop the AWS Greengrass Message Component
You can now develop your custom application logic in the template AWS Greengrass component and redeploy by repeating the GDK build and publish steps then revising the deployment in the AWS IoT console and selecting the latest revision number from the component configuration step. 

### Add a new Function Call

Try adding a new PubSub message call to your component. 

In [MySensorMessageHandler](/samples/gg-pubsub-sdk-component-template/src/pubsub_message_handlers/my_system_message_handler.py) add a new vibration sensor call:
1. Copy the entire **get_temp_sensor_request(...)** method and rename it to **get_vibration_sensor_request(.....)**  
2. In the new get_vibration_sensor_request(.....) method, replace  
   a. "temp_c" : random.randint(30, 50) with 
   b. "g-force" : random.randint(1, 6)

3. Save the file.  

Now repeat the **gdk build** and **gdk publish** command to build a new version of your component. Finally, repeat the deploy steps and update the component version when you get to the component configuration section. 

Using the same **Test** page as above, send the message with the **route** field now set to: **MySensorMessageHandler.get_vibration_sensor_request**  
i.e:
```
{
  "sdk_version": "0.1.4",
  "message_id": "20220329121327002764",
  "status": 200,
  "route": "MySensorMessageHandler.get_vibration_sensor_request",
  "message": { }
}
```

You should see your response return with the new message values you set. 
Use this technique to continue developing your AWS Greengrass Custom component. 

## Monitoring and Debugging the Component

If there are any issues, you can monitor the deployment and the component itself on the Greengrass core in the following logs:
* **Greengrass Core Log:** /greengrass/v2/greengrass.log and 
* **Greengrass Component Log:** /greengrass/v2/[YOUR_GREENGRASS_COMPONENT_NAME].log

## Taxonomy of the SDK Template Component

### Component Recipe
The component recipe in the template component defines a configuration object **GGV2PubSubSdkConfig** that enables data injection of application parameters to the component’s application logic on initialisation.

#### PubSub SDK Config Parameters 
The default GGV2PubSubSdkConfig recipe field is show below: 
```
"GGV2PubSubSdkConfig": {
        "base-pubsub-topic" : "com.example.greengrass-pubsub-component",
        "ipc-subscribe-topics" : ["ipc/my-app/broadcast", "ipc/my-app/error"],
        "mqtt-subscribe-topics" : ["mqtt/my-app/broadcast", "mqtt/my-app/error"]
      }
```

Apart from enforcing consistent design patterns, this config driven approach allows updates of common PubSub application properties and topic schema without the need for application or code updates. You can also add you own parameters in the GGV2PubSubSdkConfig field and have them passed into your application logic using the same patterns described.

#### PubSub Access Policy
The component recipe in the template component defines the PubSub Access Control policy that provides permission for the component to subscribe and publish to both the IPC and MQTT message protocols. By default, this is provided as an Allow All policy and can be restricted to the components least required permissions as needed.

#### Component Lifecycle Events
The template component Lifecycle **Install** event installs the SDK Python package when the component is installed on an AWS Greengrass core device.

The Lifecycle **Run** event sets the components start command and passed the GGV2PubSubSdkConfig JSON field to the component’s application logic.

```
  "Lifecycle": {
    "Install" : {
      "Timeout" : 300,
      "Script" : "python3 -m pip install awsgreengrasspubsubsdk"
    },
    "Run": {
      "Script": "python3 -u {artifacts:decompressedPath}/src/main.py '{configuration:/GGV2PubSubSdkConfig}'",
      "RequiresPrivilege": "false"
    }
```

### Main.py

Provided as the entry point for the template component, the Main module provides the following:

#### Component Config Injection
The main module accepts the GGV2PubSubSdkConfig as a sys.arvg into the application logic and parses from JSON to an object called ggv2_component_config. This is provided to initialise the MyAwsGreengrassV2Component class and finally the components service loop function is started. 

```
if __name__ == "__main__":

    try:
        # Parse config from component recipe into sys.argv[1] 
        ggv2_component_config = sys.argv[1]
        ggv2_component_config = json.loads(ggv2_component_config)
        log.info('GG PubSub SDK Config: {}'.format(ggv2_component_config))

        # Create the component class with the parsed Greengrass recipe config.
        ggv2_component = MyAwsGreengrassV2Component(ggv2_component_config)
        
        # Start the main process loop to hold up the process.
        ggv2_component.service_loop()

    except Exception as err:
        log.error('Exception occurred initialising component. ERROR MESSAGE: {}'.format(err))

```

#### Component Service Loop
Some Greengrass components will be completely event driven with events being triggered in reaction to PubSub messages in which case the service loops only function is to hold the main process up. Its also common for synchronous operations to live within the service loop. 

In the sample template, the service loop periodically publishes a well formatted message containing the local time.

#### MyAwsGreengrassV2Component Class Initialisation
The template component main class is called MyAwsGreengrassV2Component. On initialisation, this class extracts the base-pubsub-topic, ipc-subscribe-topics and mqtt-subscribe-topics parameters from the ggv2_component_config and the  **ingress** and **egress** topics are built as:
* **Ingress Topic**: [base-pubsub-topic]/[AWS_IOT_THING_NAME]/ingress and 
* **Egress Topic**: [base-pubsub-topic]/[AWS_IOT_THING_NAME]/egress

The AWS Greengrass PubSub SDK client message_formatter is initialised and the IPC and MQTT protocols are activated. The SDK doesn't subscribe or allow publishing to any topics until the IPC and / or MQTT protocols are activated as per the example. 

### User Defined Message Handlers
This is the user defined message handlers in the [pubsub_message_handlers](/samples/gg-pubsub-sdk-component-template/src/pubsub_message_handlers) directory as described above. In the template component, two separate message handlers are provided how to route messages and separate PubSub message processors based on functionality.


