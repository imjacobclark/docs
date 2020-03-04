PubSub provides connectivity with cloud-based message-oriented middleware. You can use PubSub to pass messages between your app instances and your app's backend creating real-time interactive experiences.

PubSub is available with **AWS IoT**. 

Starting with version `12.1.1`, iOS requires that publicly-trusted Transport Layer Security (TLS) server authentication certificates issued after October 15, 2018 meet the Certificate Transparency policy to be evaluated as trusted on Apple platforms. Any existing customer endpoint you have is most likely a VeriSign endpoint. If your endpoint has `-ats` at the end of the first subdomain, then it is an Amazon Trust Services endpoint. You can get an updated endpoint from the AWS console (AWS Console->IoT Core ->Settings page). For more details read: https://aws.amazon.com/blogs/iot/aws-iot-core-ats-endpoints/
{: .callout .callout--info}

## Installation and Configuration

### AWS IoT

When used with `AWSIoTDataManager`, PubSub is capable of signing request according to [Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html). 

The `Podfile` that you configure to install the AWS Mobile SDK must contain the `AWSIoT` pod:

```ruby
    platform :ios, '9.0'

    target :'YOUR-APP-NAME' do
      use_frameworks!

        pod  'AWSIoT', '~> 2.12.0'
        # other pods

    end
```

Run `pod install --repo-update` before you continue.

To use in your app, import the following:

```swift
import AWSIoT
```

Define your unique client ID and endpoint (incl. region) in your configuration:

```swift
// Initialize the AWSIoTDataManager with the configuration
let iotEndPoint = AWSEndpoint(
    urlString: "wss://xxxxxxxxxxxxx-ats.iot.<YOUR-AWS-REGION>.amazonaws.com/mqtt")
let iotDataConfiguration = AWSServiceConfiguration(
    region: AWSRegionType.<YOUR-AWS-REGION>,
    endpoint: iotEndPoint,
    credentialsProvider: AWSMobileClient.default()
)

AWSIoTDataManager.register(with: iotDataConfiguration!, forKey: "MyAWSIoTDataManager")
let iotDataManager = AWSIoTDataManager(forKey: "MyAWSIoTDataManager")
```

You can get the endpoint information from the IoT Core -> Settings page on the AWS Console.  
{: .callout .callout--info}

**Create IAM policies for AWS IoT**

To use PubSub with AWS IoT, you will need to create the necessary IAM policies in the AWS IoT Console, and attach them to your Amazon Cognito Identity. 

Go to IoT Core and choose *Secure* from the left navigation pane. Then navigate to *Create Policy*. The following `myIOTPolicy` policy will allow full access to all the topics.

![Alt text]({%if jekyll.environment == 'production'%}{{site.amplify.docs_baseurl}}{%endif%}/js/images/iot_attach_policy.png?raw=true "Title")


**Attach your policy to your Amazon Cognito Identity**

To attach the policy to your *Cognito Identity*, begin by retrieving the `Cognito Identity Id` from `AWSMobileClient`.

```swift
AWSMobileClient.default().getIdentityId();
```

Then, you need to attach the `myIOTPolicy` policy to the user's *Cognito Identity Id* with the following [AWS CLI](https://aws.amazon.com/cli/) command:

```bash
aws iot attach-principal-policy --policy-name 'myIOTPolicy' --principal '<YOUR_COGNITO_IDENTITY_ID>'
```