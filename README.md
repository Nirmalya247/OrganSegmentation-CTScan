# Organ Specific Deep Learning Networks for Anatomical Structure Segmentation

The project consists of two parts
1. Backend (APIs for File conversion, pre and post-processing, prediction) resides in the **organPy.zip**
1. Visualization Application (visualize and interactive UI) resides in **organSegment**

## Backend

The backend is a compressed .zip file and needs to be extracted upon download. It contains portable Python librarys that can run on any system with Windows 8.1 and above

### Run

To run the backend, we just need to run the file:

```
organPy -> run.bat
```

### APIs

The API uses the follwing format:

* All request uses get requests.
* All file path must be in base64 encoded.
* Most requests sends response as soon as it gets the request and creates a thread for further processes and the status is saved in a file whose path needs to be sent as a parameter of the request (logPath)

1.1.	**Creating .vti File**: To visualize the image we need to convert any given image to .vti file we are using constant scale of ```3``` in z axis or Transverse plane and other axis have scale of ```1```. The API description of converting .nii.gz to .vtk is:

```
http://127.0.0.1:2201/getfullvtk?niiPath=<base64 path>&vtkPath=<base64 vti path>&sizePath=<base64 path to save image size>&vtkDataName=<base64 vtk path> &logPath=<base64 path of log file>
```
The API for converting dcm directory to .nii.gz is:
```
http://127.0.0.1:2201/convertdcm?dataRoot=<base64 path of dcm folder>&getMask=<1 if image contains mask else 0>&logPath=<base64 path of log file>
```
This API creates a folder called temp in the dataRoot folder to save the file naming: dataRoot/temp/image.nii.gz

1.2. **Bounding Box**: The API for finding bounding box is the following:
```
http://127.0.0.1:2201/getboundary?niiPath=<base64 path>&boundPath=<base64 sizePath file path>&logPath=<base64 path of log file>
```
2.1. **Prediction**: The API for prediction takes the following data
```
http://127.0.0.1:2201/getprediction?niiPath=<base64 path>&smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>&predictBoundFileName=<base64 bound path to be saved>&logPath=<base64 path of log file>&xMin=<x min>&xMax=<x max>&yMin=<y min>&yMax=<y max>&zMin1=<z min bladder>&zMax1=<z max bladder>&zMin2=<z min rectum>&zMax2=<z max rectum>&zMin3=<z min sigmoid>&zMax3=<z max sigmoid>
```
The above API saves the VTK files in the given file path (smallPredictVTK1, 2, 3) for the respective organs (bladder, rectum and sigmoid), the API also saves .npy files of the same data in the same path for easy post processing if necessary.
all organs are of the dimension: ```(S x C x T)```
```
S => xMax - xMin
C => yMax – yMin
T => max(zMax1, zMax2, zMax3) – min(zMin1, zMin2, zMin3)
```
The dimension is saved in the given file path (predictBoundFileName) in the format:

```{ 'xMin': <int>, 'xMax': <int>, 'yMin': <int>, 'yMax': <int>, 'zMin': <int>, 'zMax': <int> }```

3.1. **Removing disconnected components**: The prediction can contain some small glitches or disconnected components and for removing those disconnected components we can use the API:
```
http://127.0.0.1:2201/getglitches?smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>&logPath=<base64 path of log file>
```
The above API saves all the disconnected components of each organ in its directory, So, it will first create three directories with the given path ```(smallPredictVTK1, 2, 3)``` and saves files for each disconnected components containing the pixels ```[(x1, y1, z1), (x2, y2, z2)…]``` in 16bit binary format. The file names are in the format (index_numberOfPixelInTheComponent), the index starts from ‘0’ and sorted in descending order of number of pixels.
The Visualization software has a feature where the user can select the components needed to be included in the label and remove the other components.
Next, we have another API to recreate the VTK files.
```
http://127.0.0.1:2201/setglitches?smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>& selected1=<base64 comma separated file names for bladder>& selected2=<base64 comma separated file names for rectum>& selected1=<base64 comma separated file names for sigmoid>&logPath=<base64 path of log file>
```
The selected file names do not contain entire path, only the name of the file.

The API will update the VTK files of each organ and we need to reload the predicted organs in the Visualization software.

3.2. **Save**: To save the segmented organs we can use the following APIs

To save as .vtk:
```
http://127.0.0.1:2201/savepredictionvtk?niiPath=<base64 path>&smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>&fullImageSizeFileName=<base64 sizePath file path>&predictBoundFileName=<base64 bound path to be saved>&newFullVTKPath=<base64 path of .vtk file to be created>&clip=<1 or 0  to clip recto sigmoid or not>&logPath=<base64 path of log file>
```
To save as .nii.gz:
```
http://127.0.0.1:2201/savepredictionnii?niiPath=<base64 path>&smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>&fullImageSizeFileName=<base64 sizePath file path>&predictBoundFileName=<base64 bound path to be saved>&newFullNIIPath=<base64 path of .nii file to be created>&clip=<1 or 0  to clip recto sigmoid or not>&logPath=<base64 path of log file>
```
To save as .dcm:
```
http://127.0.0.1:2201/savepredictionnii?niiPath=<base64 path>&smallPredictVTK1=<base64 bladder path>&smallPredictVTK2=<base64 rectum path>&smallPredictVTK3=<base64 sigmoid path>&fullImageSizeFileName=<base64 sizePath file path>&predictBoundFileName=<base64 bound path to be saved>&newFullDCMPath=<base64 path of .dcm file to be created>&clip=<1 or 0  to clip recto sigmoid or not>&logPath=<base64 path of log file>
```
## Visualization

The Visualization software is a desktop apllication written in .NET (c#) 

### Run

To run the software we can open the ```organSegment/organSegment3.sln``` in Visual Studio 2022 or above with .NET Framework 4.7.2 SDK installed or we can run the following command in the folder ```organSegment/organSegment3```:
```
organSegment/organSegment3 dotnet run
```

### Open file

To open a file we can choose appropriate option from the menu:
```
File->Open Image // to open .nii.gz
File->Open DCM->image only // to open .dcm folder with out mask
File->Open DCM->with mask // to open .dcm folder with mask
```

### Visualize Image

To visualize a image we can click ```FullImage``` in the tool strip

### Compute Bounding Box

To compute bounding box we can click ```Compute Bounding Box``` in the tool strip

Then we can modify the area using sliders in the right side tool box (X, Y, Z) and we choose for specific organs (Bladder, Rectm, Sigmoid)

### Prediction

To predict the organs we can click ```Compute Predictions``` in the tool strip

We will get five possible clipping points and all disconnected components for bladder and rectum/sigmoid, we can choose one or multiple junction points or we can set it manually

Also the components we wish to keep and then we can click on ```set``` to visualize the selected components

### Save

To save the prediction we can choose appropriate option from the menu:
```
File->Save As->.vtk // to save as .vtk
File->Save As->.nii.gz // to save as .nii.gz
File->Save As->.dcm // to save as .dcm (clipped recto-sigmoid)
File->Save As->.dcm (without clip) // to save as .dcm (without clipping recto-sigmoid)
```

### Open new project

To close a project we can choose appropriate option from the menu:
```
File->Close Project
```
