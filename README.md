# alexa_hackathon
During this hackathon, we'll take a deep dive into API Gateway, Lambda, DynamoDB, 
and the Alexa Skills Kit by building a skill that invokes a simple RESTful API. This agenda
is intended for a daylong session and a group of 15-30 people, splitting up into two teams. One
team focuses on API development, while the other team focuses on Alexa integration. You could consider
these teams as the *data team* and *integration team* respectively. In a modern DevOps team, both
teams would be combined as one to own the development, enhancement, maintenance, and support of
data and integrations for one or more microservices

> #### Prerequisites
>We'll assume that you have some basic knowledge of AWS 
services like IAM, Cloudformation, DynamoDB, S3, IoT, etc., 
are comfortable using the AWS CLI, and have some knowledge 
of Python. You'll also need to prepare the following <b>prior</b> 
to the workshop: 
>* Laptop running Windows or MacOS (or an EC2 instance with Amazon Linux or Windows2012)
>* An AWS account with Administrator Access
>* The AWS CLI, configured with an Administrator Access ([directions here](https://docs.aws.amazon.com/cli/latest/userguide/installing.html))
>* The ASK CLI ([Alexa Skills Kit CLI](https://developer.amazon.com/docs/smapi/quick-start-alexa-skills-kit-command-line-interface.html#step-1-prerequisites-for-using-ask-cli))
>* Python, Node, and NPM, Virtualenv, git

## Introduction
This Hackathon has two main parts as descroibed above: the API part and the Alexa part. The intent 
of this session is to take a "200-level" dive into microservice design by creating
an Alexa skill and accompanying API. We'll use Lambda, API Gateway, DynamoDB, 
IAM, and Alexa to demonstrate this. 

> If you have established an AWS account within the last 12 months, then this lab will be in the free tier. Otherwise, costs are anticipated to be less than $5

---

## Local Preparation Steps
First, create a *work directory* where you can download the git repository for this hackathon.
save the worksheet, make notes, etc. On my macOS system, I use ~/Developer/hackathons (that is, /Users/dixonaws/Developer/hackathons). 
Second, clone the git repository for this hackathon:
```bash
git clone https://github.com/dixonaws/alexa_hackathon
```

You should now have a new directory, *alexa_hackathon* in your work directory.

Third, complete the worksheet below *or*, if you are on macOS/Linux, you can use a utility in alexa_hackathon to
check versions and create worksheet (called worksheet.txt) for you:
```bash
chmod +x create_worksheet.sh
./create_worksheet.sh
```

---

# Agenda
830-9 Intro to the hackathon: what we'll be doing and how we'll work
9-12 Alexa Team and API Team work on their respective tasks separately
12-1 Lunch
1-130 Checkpoint
130-4 Integration
4-5 Demonstrations: the API layer, the Alexa layer, and the combined skill

# Alexa Team
Create an Alexa Skill that can get data from a RESTful API endpoint. You can use the Alexa Skills Kit CLI (ask-cli)
to create a simple "hello world" skill based on a template. You may choose NodeJS or Python for this skill.
*Approximate time for this task is 2 hours.*

## References
Walkthrough: https://developer.amazon.com/docs/smapi/quick-start-alexa-skills-kit-command-line-interface.html
Youtube video: https://www.youtube.com/watch?v=I-Dw8IpnS2g

# API Team
Create a RESTful API that the Alexa team can use to query data. The API should be backed
by DynamoDB or Aurora, and return a JSON object. For now, the API should be built to respond to HTTP GET requests. 
*Approximate time for this task is 2 hours.*

## References
You can use www.github.com/dixonaws/restful_dynamo or www.github.com/dixonaws/restful_aurora as starting points 
to create your API. 

> Recommendations for both teams: 1) start small and iterate from a "Hello, World!" type of skill to one that returns custom messages, 2) don't worry about the other team's progress during the morning session -- instead, focus on developing your component and understanding how to modify it.


---

## 1. Deploy the CVRA (15 mins.)
Let's deploy the Connected Vehicle Reference Architecture (CVRA).
 Following the [directions here](https://docs.aws.amazon.com/solutions/latest/connected-vehicle-solution/deployment.html), deploy 
 the CVRA in an AWS account where you have administrator access.
The CVRA comes with a Cloudformation template that deploys and configures
all of the AWS services necessary to ingest, store, process, and
analyze data at scale from IoT devices. Automotive use cases aside,
the CVRA provides a useful example of how to secure connect an 
IoT device, perform JITR (Just in Time Registration), use 
Kinesis Analytics to query streams of data, use an IoT rule to 
store data in S3, etc.

The CVRA Cloudformation 
template returns these outputs:

| Key | Value | Description | Associated AWS Service
|:---|:---|:---|:---
UserPool|arn:aws:cognito-idp:us-east-1:000000000:userpool/us-east-1_loAchZlyI|Connected Vehicle User Pool| Cognito
CognitoIdentityPoolId|us-east-1:de4766b0-519a-4030-b036-97a3a2291c98|	Identity Pool ID| Cognito
VehicleOwnerTable|	cvra-demo-VehicleOwnerTable-1TMCCT7LY76B0|	Vehicle Owner table|DynamoDB
CognitoUserPoolId|	us-east-1_loAchZlyI|	Connected Vehicle User Pool ID|Cognito
CognitoClientId|	6rjtru6aur0vni0htpvb49qeuf|	Connected Vehicle Client|Cognito
DtcTable|	cvra-demo-DtcTable-UPJUO460FVYT|	DTC reference table|DynamoDB
VehicleAnomalyTable|	cvra-demo-VehicleAnomalyTable-E3ZR7I8BN41D|	Vehicle Anomaly table|DynamoDB
VehicleTripTable|	cvra-demo-VehicleTripTable-U0C6DSG0JW11|	Vehicle Trip table|DynamoDB
TelemetricsApiEndpoint|	https://abcdef.execute-api.us-east-1.amazonaws.com/prod|RESTful API endpoint to interact with the CVRA (Cognito authZ required)|API Gateway
Telemetrics API ID|(not used)|(not used)|API Gateway
HealthReportTable|	cvra-demo-HealthReportTable-C4VRARO31UZ1|	Vehicle Health Report table|DynamoDB
VehicleDtcTable|	cvra-demo-VehicleDtcTable-76E1UB71GEH3|	Vehicle DTC table|DynamoDB

**Validation**: run the following command from a terminal window:
```bash
aws cloudformation describe-stacks --stack-name cvra-demo --query 'Stacks[*].StackStatus'
```

> We'll refer to the CVRA Cloudformation stack as **cvra-demo** throughout this bootcamp.

The output should resemble something like this:
```json
[
    "CREATE_COMPLETE"
]
```

We're interested in the *VehicleTripTable* -- a table in DynamoDB. You can view the outputs from your 
CVRA deployment through the AWS Console or by using the CLI with something like:
```bash
aws cloudformation describe-stacks --stack-name cvra-demo --output table --query 'Stacks[*].Outputs[*]'
```

---
 
## 2. Deploy the IoT Device Simulator and Generate Trip Data (30 mins.)
In this section, you'll install and configure the AWS IoT Device Simulator to generate 
trip data. [Follow these directions](https://aws.amazon.com/answers/iot/iot-device-simulator/) 
to install the simulator in your own AWS account.

You can provision up to 25 vehicles to simulate trip data. Each simulated
vehicle will travel one of several paths that have been pre-defined by the IoT
device simulator. 

The CVRA expects data to be published to a topic called: `connectedcar/telemetry/<VIN>` The 
device simulator allows you to simulate a number of vehicles and generate trip data.
The payload is of the form:

```json
{
  "timestamp": "2018-08-25 22:38:40.791000000",
  "trip_id": "871be6ea-4ee6-49b8-8a9b-b6ebe5050c8a",
  "vin": "9JVVV63E5NVZWH5UH",
  "name": "odometer",
  "value": 2.267
}
```

---

## 3. Deploy the ConnectedCar Alexa Skill (20 mins.)
In this section, we'll deploy an Alexa skill called ConnectedCar that will read back information about 
the three recent trips that you have taken. You must have the ASK-CLI installed to complete this part of the lab.

### 3.1 Run a Python Program to Test Your Access
First, you can use a Python program included with the reinvent_cvra_bootcamp repo, getRecentTrips.py, to 
test your configuration sofar. The best way to run this program is within a Python Virtual Environment. For the 
impatient, install a virtual environment, install the dependencies from requirements.txt, and run getRecentTrips.py.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary>
<p>
Install and activate Python virtual environment in ./venv (macOS):

```bash
virtualenv venv
source venv/bin/activate
```

Install dependencies (macOS):
```bash
pip install -r requirements.txt
```

Run the program:
```bash
python getRecentTrips.py <TripTable>
```

Or, if you wanted to be very clever using your <i>ninja bash skills</i>, you could do something like this on the bash prompt: 

```bash
python getRecentTrips.py `aws cloudformation describe-stacks --stack-name cvra-demo --output table --query 'Stacks[*].Outputs[*]' |grep 'Vehicle Trip table' |awk -F "|" '{print $4}'`
```
> Quote trifecta: Note the tricky combination of backticks, single quotes, AND double quotes!
</p>
</details>
<br>

You should see output similar to the following after running the program:
```json(venv) f45c898a35bf:reinventCvraBootcamp dixonaws$ python3 getRecentTrips.py cvra-demo-VehicleTripTable-U0C6DSG0JW11
Scanning trip table cvra-demo-VehicleTripTable-U0C6DSG0JW11... done (0).
Found 18 items in the trip table.
dictItems is a <class 'list'>
**** Trip 1 (VIN: 2Z61V6JISOE60EWI8)
{'ignition_status': {'S': 'off'}, 'transmission_gear_position': {'S': 'fourth'}, 'engine_speed_mean': {'N': '3525.2780709525855'}, 'name': {'S': 'aggregated_telemetrics'}, 'driver_safety_score': {'N': '74.82328344340092'}, 'brake_mean': {'N': '0'}, 'high_braking_event': {'N': '0'}, 'fuel_level': {'N': '99.95453355436261'}, 'latitude': {'N': '38.958911'}, 'idle_duration': {'N': '213'}, 'fuel_consumed_since_restart': {'N': '0.019374198537822133'}, 'torque_at_transmission_mean': {'N': '314.3124901722019'}, 'timestamp': {'S': '2018-08-21 23:13:01.589000000'}, 'vehicle_speed_mean': {'N': '79.9034721171209'}, 'start_time': {'S': '2018-08-21T23:12:31.456Z'}, 'end_time': {'S': '2018-08-21T23:13:01.589Z'}, 'trip_id': {'S': '84983a6b-0881-4600-ad20-9db40eb7f868'}, 'oil_temp_mean': {'N': '29.021054075000006'}, 'geojson': {'M': {'bucket': {'S': 'connected-vehicle-trip-us-east-1-477157386854'}, 'key': {'S': 'trip/2Z61V6JISOE60EWI8/84983a6b-0881-4600-ad20-9db40eb7f868.json'}}}, 'accelerator_pedal_position_mean': {'N': '38.609446258882855'}, 'longitude': {'N': '-77.401168'}, 'vin': {'S': '2Z61V6JISOE60EWI8'}, 'brake_pedal_status': {'BOOL': False}, 'high_speed_duration': {'N': '0'}, 'odometer': {'N': '0.6705392939350361'}, 'high_acceleration_event': {'N': '2'}}
...

```

The getRecentTrips.py program simply queries your DynamoDB trip table, which is similar to the ConnectedCar skill that 
we'll deploy in the next section.
<details>
<summary><strong>Code details for getRecentTrips.py (expand for details)</strong></summary>
Have a look at the code listing for getRecentTrips.py. The guts 
are similar to the ConnectedCar skill that we'll deploy in the next
step, particularly the call to <i>scan</i> the DynamoDB trip table:

```python 
dynamoDbClient=boto3.client('dynamodb')

    response=dynamoDbClient.scan(
        TableName='cvra-demo-VehicleTripTable-U0C6DSG0JW11',
        Select='ALL_ATTRIBUTES'
    )

    dictItems=response['Items']

    intRecordCount=json.dumps(response['Count'])
    print("Found " + str(intRecordCount) + " items in the trip table.")

    intTripNumber=1
    for item in dictItems:
        strVin=str(item['vin']['S'])
        print("**** Trip " + str(intTripNumber) + " (VIN: " + strVin + ")")
        print(item)
        print()
        intTripNumber=intTripNumber+1

    print("dictItems is a " + str(type(dictItems)))

```
> An improvement here for production applications would be to 
> query the DynamoDB table instead of scanning it, per the 
> [best practices for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html). 
> You may even elect to create an API layer in front fo the 
> DynamoDB table so that other applications can use the 
> data.
</details>


### 3.2 Deploy the Alexa Skill with the ASK-CL 
In ths step, you'll use the trip data recorded in your DynamoDB table with an Alexa skill called ConnectedCar. First, you'll
need to create an account on developer.amazon.com and initilize the ASK CLI if you haven't already done so.

<details>
<summary><strong>Initialize the ASK CLI (expand for details)</strong></summary>
Issue the following command:
```bash
ask init
```

You should now see this screen in the command prompt. This step isused to select your AWS profile. Choose the default profile.
```bash
(venv) jpdixonmbp:reinvent_cvra_bootcamp jpdixon$ ask init
? Please create a new profile or overwrite the existing profile.
 (Use arrow keys)
  ──────────────
❯ Create new profile 
  ──────────────
  Profile              Associated AWS Profile
  [default]                 "default" 

```

Next, you'll see the following screen to select the AWS profile to use for Lambda function deployment. Choose default:
```bash
(venv) jpdixonmbp:reinvent_cvra_bootcamp jpdixon$ ask init
? Please create a new profile or overwrite the existing profile.
 [default]                 "default"
-------------------- Initialize CLI --------------------
Setting up ask profile: [default]
? Please choose one from the following AWS profiles for skill's Lambda function deployment.
 
❯ default 
  dixonaws@amazon.com 
  jpdixon@gmail.com 
  ccdtw 
  ──────────────
  Skip AWS credential for ask-cli. 
  Use the AWS environment variables. 
  ──────────────


```

Next, the ASK CLI will open a browser window to amazon.com and ask you to sign in using your developer account credentials. Sign in and 
close the browser. 
```bash
(venv) jpdixonmbp:reinvent_cvra_bootcamp jpdixon$ ask init
? Please create a new profile or overwrite the existing profile.
 [default]                 "default"
-------------------- Initialize CLI --------------------
Setting up ask profile: [default]
? Please choose one from the following AWS profiles for skill's Lambda function deployment.
 default
Switch to 'Login with Amazon' page...
  ◝  Listening on http://localhost:9090


```
If all goes well, you should see this on the command prompt:
```bash
(venv) jpdixonmbp:reinvent_cvra_bootcamp jpdixon$ ask init
? Please create a new profile or overwrite the existing profile.
 [default]                 "default"
-------------------- Initialize CLI --------------------
Setting up ask profile: [default]
? Please choose one from the following AWS profiles for skill's Lambda function deployment.
 default
Switch to 'Login with Amazon' page...
Tokens fetched and recorded in ask-cli config.
Vendor ID set as M3D1DAHOJTACU3

Profile [default] initialized successfully.
(venv) jpdixonmbp:reinvent_cvra_bootcamp jpdixon$ 
```


</details>


Once you have confirmed that your DynamoDB Trip table contains trip data withg getRecentTrips.py, and 
initialized the Alexa Skills Kit CLI, issue the following command from your *work* directory 
to clone the ConnectedCar skill. This command will create a new directory called ConnectedCar in the current directory:

```bash
ask new --skill-name "ConnectedCar" --url https://github.com/dixonaws/ConnectedCarAlexa.git  
```

//todo
Modify ConnectedCar's Lambda function to use your DynamoDB table. Then simply deploy it with:
```bash
ask deploy
```

### 3.3 Interact with ConnectedCar
Open developer.amazon.com, login, and browse to you ConnectedCar Alexa Skill. Click on "Developer Console," and then "Alexa Skills Kit." You 
should be able to see the ConnectedCar skill that you deployed in the previous section. Open ConnectedCar and click 
on "Test" near the top of the page. You can use this console to interact with an Alexa skill without using a 
physical Echo devce -- via text or via voice. Try these interactions:

"Alexa, open ConnectedCar"

"Alexa, ask ConnectedCar about my car"

"Alexa, ask ConnectedCar about my trip"

You can also test via the command line with this command:
```bash
ask simulate --text "alexa, open connected car" --locale "en-US"
```

---

## 4. Cleanup (10 mins.)
The last thing to do in this bootcamp is to clean up any resources that were deployed in your account. 
From your worksheet, delete the following Cloudformation stacks:
* Your CVRA Cloudformation stack
* Your vehicle simulator Cloudformation stack

```bash
aws cloudformation delete-stack --stack-name <your CVRA stack>
aws cloudformation delete-stack --stack-name <your vehicle simulator stack>
```

From the AWS console, ensure that any associated S3 buckets, DynamoDB tables, and IoT service is clean.

Also be sure to delete the following:
* The Lambda function for ConnectedCar
* The ConnectedCar skill from developer.amazon.com

---

## 5. Ideas for Customization and Enhancement
Hopefully, you were able to learn how to make use
of the data collected by a simulated connected vehicle (and ultimately any connected 
device). Here are some ideas to make enhancements and improvements from here:
* Adjust the IAM roles for more granular permissions
* Develop account linking for the ConnectedCar skill to read back information only for linked VINs
* Create an authenticated API to access the VehicleTripTable (API Gateway, Lambda, Cognito)
* Enhance ConnectedCar to get the latest fuel prices in a certain location
* Deploy the solution with a real vehicle, using Greengrass and an OBD II dongle!
