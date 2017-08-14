# Create a new application named "myNewApp" based on the "blue" Stack Template:

	1. Create change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV1",
			ChangeSetType: "CREATE"
			StackName: "myNewApp",
			TemplateURL: "https://s3.amazonaws.com/cloudformations-014066636401/cloudformations/stacks/blue.yaml",
			Capabilities: ["CAPABILITY_NAMED_IAM"]
		};
		cloudformation.createChangeSet(params, function(err, data) {
			if (err) console.log(err, err.stack); // an error occurred
			else     console.log(data);           // successful response
		});

	2. View change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV1",
			StackName: "myNewApp"
		};
		cloudformation.describeChangeSet(params, function(err, data) {
			if (err) console.log(err, err.stack); // an error occurred
			else     console.log(data);           // successful response
		});

		If you review the request from the AWS Cloudformations console, the "myNewApp" 
		stack will remain in a "Reviewing" state until the changes are implemented by 
		the execute-change-set command.

	3. Execute change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV1",
			StackName: "myNewApp"
		};
		cloudformation.executeChangeSet(params, function(err, data) {
			if (err) console.log(err, err.stack); // an error occurred
			else     console.log(data);           // successful response
		});

# Update an existing application w/ different or updated Stack Template:

	1. Update change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV2",
			ChangeSetType: "UPDATE"
			StackName: "myNewApp",
			TemplateURL: "https://s3.amazonaws.com/cloudformations-014066636401/cloudformations/stacks/blue.yaml",
			Capabilities: ["CAPABILITY_NAMED_IAM"]
		};
		cloudformation.createChangeSet(params, function(err, data) {
			if (err) console.log(err, err.stack); // an error occurred
			else     console.log(data);           // successful response
		});

	2. View change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV2",
			StackName: "myNewApp"
		};
		cloudformation.describeChangeSet(params, function(err, data) {
			if (err) console.log(err, err.stack); // an error occurred
			else     console.log(data);           // successful response
		});

		If you review the request from the AWS Cloudformations console, the "myNewApp" 
		stack will remain in a "Reviewing" state until the changes are implemented by 
		the execute-change-set command.	

	3. Execute change set:

		var AWS = require('aws-sdk');
		AWS.config.update({region: 'us-east-2'});
		var cloudformation = new AWS.CloudFormation();

		var params = {
			ChangeSetName: "blueV2",
			StackName: "myNewApp"
		};
		cloudformation.executeChangeSet(params, function(err, data) {
				if (err) console.log(err, err.stack); // an error occurred
				else     console.log(data);           // successful response
		});

# Resource Templates 
	
	Independent cloudformation templates that deploy a single AWS service
	
	Repo location: ./resorces/*
	
	Example:

		./resources/sqs.yaml

		AWSTemplateFormatVersion: '2010-09-09'
		Description: sqs
		Resources:
		  sqs:
		    Type: "AWS::SQS::Queue"
		    
		Outputs:
		  sqsArn:
		    Value: !GetAtt sqs.Arn
		  sqsName:
		    Value: !GetAtt sqs.QueueName

# Transforms

	Parameters of the Resource Templates that are included from any external s3 bucket. 
	These could be defined by a developer or security team..etc and sourced during deployment. 

	Repo location: ./transforms/*

	Example:

		./transforms/lambda.yaml

		MemorySize: 1024
		ZipFile: console.log("hello world and goodbye")
		Timeout: 25
		Mode: Active

# Stack Templates
	
	A grouping of Resource Templates working together for a single application. 
	Upon a Stack update, an changes within any of the independant Resource Templates or external Transform files
	are applied to the overall Stack(s).

	Repo location ./stacks/*

	Example:

		./stacks/blue.yaml

		AWSTemplateFormatVersion: '2010-09-09'
		Description: A Blue Stack Application

		Resources:

		  role:
		    Type: "AWS::CloudFormation::Stack"
		    Properties:
		      TemplateURL: "https://s3.amazonaws.com/cloudformations-014066636401/cloudformations/resources/role.yaml"
		      TimeoutInMinutes: 2
		      Parameters:
			'Fn::Transform':
			  Name: 'AWS::Include'
			  Parameters:
			    Location: "s3://cloudformations-014066636401/cloudformations/transforms/role.yaml"

		  loggroup:
		    Type: "AWS::CloudFormation::Stack"
		    Properties:
		      TemplateURL: "https://s3.amazonaws.com/cloudformations-014066636401/cloudformations/resources/loggroup.yaml"
			      TimeoutInMinutes: 2

			  lambda:
			    Type: "AWS::CloudFormation::Stack"
			    DependsOn:
			      - role
			    Properties:
			      TemplateURL: "https://s3.amazonaws.com/cloudformations-014066636401/cloudformations/resources/lambda.yaml"
			      TimeoutInMinutes: 2
			      Parameters:
				roleArn: 
				  Fn::GetAtt: [role, Outputs.roleArn]
				'Fn::Transform':
				  Name: 'AWS::Include'
				  Parameters:
				    Location: "s3://cloudformations-014066636401/cloudformations/transforms/lambda.yaml"

				...etc
