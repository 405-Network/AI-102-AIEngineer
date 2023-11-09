# Read Text in Images Lab

## Introduction
Optical character recognition (OCR) is a subset of computer vision that deals with reading text in images and documents. In this lab, you will explore two APIs provided by Azure AI Vision service for reading text.

## Provision an Azure AI Services Resource
1. In the Azure portal, use the top search bar to find and create a new Azure AI services multi-service account with the settings below:
	- **Subscription**: Your Azure subscription
	- **Resource Group**: Choose or create a new one as permitted
	- **Region**: Select any available region
	- **Name**: Provide a unique name
	- **Pricing Tier**: Standard S0
1. Follow through the creation process, select the necessary checkboxes and deploy the resource.
2. Once deployed, navigate to the resource's 'Keys and Endpoint' page to retrieve the endpoint and keys for later use.

## Prepare to Use the Azure AI Vision SDK
1. Open Visual Studio Code, navigate to the `20-ocr` folder, and expand either the C-Sharp or Python folder based on your preference.
2. Right-click the `read-text` folder, open an integrated terminal, and install the Azure AI Vision SDK package:  
C#

```dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0```

Python

```pip install azure-cognitiveservices-vision-computervision==0.7.0```
3. View the contents of the **read-text** folder, and note that it contains a file for configuration settings:
	- **C#**: appsettings.json
	- **Python**: .env
- Open the configuration file and update the configuration values it contains to reflect the **endpoint** and an authentication **key** for your Azure AI services resource. Save your changes.
4. Note that the **read-text** folder contains a code file for the client application:
	- **C#**: Program.cs
	- **Python**: read-text.py
- Open the code file and at the top, under the existing namespace references, find the comment **Import namespaces**. Then, under this comment, add the following language-specific code to import the namespaces you will need to use the Azure AI Vision SDK:
C#
```// import namespaces using Microsoft.Azure.CognitiveServices.Vision.ComputerVision; using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;```
Python
```# import namespaces```
```from azure.cognitiveservices.vision.computervision import ComputerVisionClient``` 
```from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes```
```from msrest.authentication import CognitiveServicesCredentials```
