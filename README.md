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
IAM, and Alexa to demonstrate this. Both teams are encouraged to track lessons learned throughout the day by
posting a stickynote to the parking lot. Each participant should create their own API and Alexa skill. During the integration,
we'll combine the API with a single Alexa skill that uses multiple intents to call each API endpoint. 

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
| Timeslot | Item |
|:---|:---|
|830-9|Intro to the hackathon: what we'll be doing and how we'll work|
|9-12 |Alexa Team and API Team work on their respective tasks separately|
|12-1 |Lunch|
|1-130 |Checkpoint|
|130-4 |Integration|
|4-430 |Demonstrations: the API layer, the Alexa layer, and the combined skill|
|430-445 |Cleanup|

# Alexa Team
Create an Alexa Skill that can get data from a RESTful API endpoint. You can use the Alexa Skills Kit CLI (ask-cli)
to create a simple "hello world" skill based on a template. You may choose NodeJS or Python for this skill. Note that the
ASK-CLI uses NodeJS for the Lambda function. You can replace the Lambda function with a Python-based function if you choose. For
examples of Pytyhon-based Lambda functions, see https://github.com/dixonaws/alexa-skills-kit-sdk-for-python
*Approximate time for this task is 2 hours.*

### References
Walkthrough: https://developer.amazon.com/docs/smapi/quick-start-alexa-skills-kit-command-line-interface.html
Youtube video: https://www.youtube.com/watch?v=I-Dw8IpnS2g

# API Team
Create a RESTful API that the Alexa team can use to query data. The API should be backed
by DynamoDB or Aurora, and return a JSON object. For now, the API should be built to respond to HTTP GET requests. 
*Approximate time for this task is 2 hours.*

### References
You can use www.github.com/dixonaws/restful_dynamo or www.github.com/dixonaws/restful_aurora as starting points 
to create your API. 

> Recommendations for both teams: 1) start small and iterate from a "Hello, World!" type of skill to one 
that returns custom messages, 2) don't worry about the other team's progress during the 
morning session -- instead, focus on developing your component and understanding how to modify it.

# Integration
During the afternoon, teams should combine and work to integrate, test, and enhance the Alexa skill. This is where
the skill should be tested from the developer console as well as an Echo device.

If time allows, the teams could simulate an authentication system by asking users for the PIN number when they 
launch the Alexa skill. At launch, the skill could play a greeting and then ask the user for a PIN (which would
be hardcoded into the Lambda function). 

---


## Cleanup (10 mins.)
The last thing to do in this hackathon is to clean up any resources that were deployed in your account. 


From the AWS console, ensure that any associated EC2 instances, DynamoDB tables, Lambda functions, or API Gateway resources have
been deleted.


---

## Ideas for Customization and Enhancement
Hopefully, you were able to see how microservices can be developed independently
and integrated (in this hackathon, using Alexa)<br>
Some ideas for enhancement:
* Develop pipelines and deployment automation for each component (CodeCommit, CodePipeline, CodeDeploy)
* Add authentication to the API layer and Alexa layer (Cognito)
* Modify the Alexa skill to get data from another API

