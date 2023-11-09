# Read Text in Images Lab
[Link to Microsoft Docs](https://microsoftlearning.github.io/AI-102-AIEngineer/Instructions/20-ocr.html)
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

**C#**:
```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```
**Python**:
```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
3. View the contents of the **read-text** folder, and note that it contains a file for configuration settings:
	- **C#**: appsettings.json
	- **Python**: .env
- Open the configuration file and update the configuration values it contains to reflect the **endpoint** and an authentication **key** for your Azure AI services resource. Save your changes.
4. Note that the **read-text** folder contains a code file for the client application:
	- **C#**: Program.cs
	- **Python**: read-text.py
- Open the code file and at the top, under the existing namespace references, find the comment **Import namespaces**. Then, under this comment, add the following language-specific code to import the namespaces you will need to use the Azure AI Vision SDK:
**C#**
```
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```
**Python**
```
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```
5. In the code file for your client application, in the Main function, note that the code to load the configuration settings has been provided. Then find the comment Authenticate Azure AI Vision client. Then, under this comment, add the following language-specific code to create and authenticate a Azure AI Vision client object:

**C#**
```// Authenticate Azure AI Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```
**Python**
```
# Authenticate Azure AI Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
## Use the Read API to read text from an image
The <b>Read</b> API uses a newer text recognition model and generally performs better for larger images that contain a lot of text, but will work for any amount of text. It also supports text extraction from .pdf files, and can recognize both printed text and handwritten text in multiple languages.

The <b>Read</b> API uses an asynchronous operation model, in which a request to start text recognition is submitted; and the operation ID returned from the request can subsequently be used to check progress and retrieve results.

1. In the code file for your application, in the Main function, examine the code that runs if the user selects menu option 1. This code calls the GetTextRead function, passing the path to an image file.
2. In the read-text/images folder, click on Lincoln.jpg to view the file that your code will process.
3. Back in the code file in Visual Studio Code, find the GetTextRead function, and under the existing code that prints a message to the console, add the following code:

**C#**
```
// Use Read API to read text in image
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // Get the async operation ID so we can check for the results
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // Wait for the asynchronous operation to complete
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // If the operation was successfully, process the text line by line
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);

                // Uncomment the following line if you'd like to see the bounding box 
                //Console.WriteLine(line.BoundingBox);
            }
        }
    }
}  
```
**Python**
```
# Use Read API to read text in image
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # Get the async operation ID so we can check for the results
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # Wait for the asynchronous operation to complete
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # If the operation was successfully, process the text line by line
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
                # Uncomment the following line if you'd like to see the bounding box 
                #print(line.bounding_box)
```
4. Examine the code you added to the GetTextRead function. It submits a request for a read operation, and then repeatedly checks status until the operation has completed. If it was successful, the code processes the results by iterating through each page, and then through each line.
5. Save your changes and return to the integrated terminal for the read-text folder, and enter the following command to run the program:

**C#**
```
dotnet run
```
**Python**
```
python read-text.py
```
6. When prompted, enter **1** and observe the output, which is the text extracted from the image.
7. If desired, go back to the code you added to **GetTextRead** and find the comment in the nested for loop at the end, uncomment the last line, save the file, and rerun steps 5 and 6 above to see the bounding box of each line. Be sure to re-comment that line and save the file before moving on.

## Use the Read API to read text from a document
1. In the code file for your application, in the Main function, examine the code that runs if the user selects menu option 2. This code calls the GetTextRead function, passing the path to a PDF document file.
2. In the read-text/images folder, right-click Rome.pdf and select Reveal in File Explorer. Then in File Explorer, open the PDF file to view it.
3. Return to the integrated terminal for the read-text folder, and enter the following command to run the program:

**C#**
```
dotnet run
```
**Python**
```
python read-text.py
```
4. When prompted, enter 2 and observe the output, which is the text extracted from the document.

## Read handwritten text
In addition to printed text, the Read API can extract handwritten text in English.

1. In the code file for your application, in the Main function, examine the code that runs if the user selects menu option 3. This code calls the GetTextRead function, passing the path to an image file.
2. In the read-text/images folder, open Note.jpg to view the image that your code will process.
3. In the integrated terminal for the read-text folder, and enter the following command to run the program:

**C#**
```
dotnet run
```
**Python**
```
python read-text.py
```
4. When prompted, enter 3 and observe the output, which is the text extracted from the document.

**Congratulations!**
For more information about using the Azure AI Vision service to read text, see the [Azure AI Vision documentation](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text).
