# Radiology
The radiology project is a collaboration with Boston Children's Hospital, Boston University, and Red Hat to improve the scale and performance of [ChRIS](https://github.com/FNNDSC/).

## ChRIS
ChRIS Research Integration Service is a web-based medical image platform that allows for various forms of medical image processing (for example, MRIs). ChRIS itself is comprised of multiple open source projects hosted on [GitHub](https://github.com/FNNDSC/) with the intention to make the research and capabilities available to other hospitals and institutions.

## Massachusetts Open Cloud
The [MOC](https://massopen.cloud/) is a cloud computing environment built in collaboration with five Massachusetts area universities (Boston University, Harvard, MIT, Northeastern, UMass) used primarily for educational and research purposes. The MOC, however, has the goal of expanding its footprint and usage by providing vertical solutions, such as the ChRIS project, to other institutions with similar needs.

## Goals
Boston Children’s Hospital, Boston University, and Red Hat aim to improve the scale and efficiency of the ChRIS platform using the MOC and various Red Hat technologies, including OpenShift and OpenStack. Today, processing a single set of images takes about ten hours. The goal is to get that down to a few minutes to give patients quicker feedback.

## Architecture
ChRIS is comprised of several components and services, and it is designed to have multiple heterogeneous clusters available for image processing. In the example shown below, the data center providing the image processing is the MOC, which is backed by OpenShift running on OpenStack. The primary interfaces into the MOC are pman and pfioh. These interfaces provide the API for pfcon to communicate with the MOC, where pfioh handles the input/ouput, and pman handles the image processing requests. The image plugins themselves run as container jobs on OpenShift.

![Chris Architecture](chris_architecture.png)

### Explanation of Terms
 * [ChRIS Frontend](https://github.com/FNNDSC/ChRIS_ultron_frontEnd) - The web-based interface for ChRIS
 * [ChRIS Backend](https://github.com/FNNDSC/ChRIS_ultron_backEnd) - The ChRIS REST API
 * [pfcon](https://github.com/FNNDSC/pfcon) - An abstraction API for ChRIS API to manage calls to different datacenters running pfioh and pman
 * [pfioh](https://github.com/FNNDSC/pfioh) - Handles input and output of image data into a data center
 * [pman](https://github.com/FNNDSC/pman) - Handles instantiating and checking the status of image processing jobs
 * Image Plugin(s) - An image processor. Turns raw image data into something meaningful for the radiologists/doctors. Ex: [FreeSurfer](https://surfer.nmr.mgh.harvard.edu/)
 * [ChRIS Store](https://github.com/FNNDSC/ChRIS_store_ui)
 * [ChRIS Frontend](https://github.com/FNNDSC/ChRIS_ui)
 * [ChRIS UI Design Repos](https://github.com/fnndsc/cube-design)


## Secure Multi-Party Computation with ChRIS


### What is Secure Multi-Party Computation (MPC) ?
[Secure MPC is crypto technique that allows parties to jointly compute a function over their inputs while keeping those inputs private](https://en.wikipedia.org/wiki/Secure_multi-party_computation)

### How does secure MPC fit in Health Care?
Secure MPC has wide application whenever data is scarce and collaborating over data is useful but restricted by privacy laws. A few examples are:
* Measuring effectiveness of rehabilitation techniques on gunshot victims
* Studying rare disease

In both the cases hospitals might want to collaborate over data but they can't share patients' data in the clear due to privacy laws and standard hospital practices. Augmenting ChRIS with cryptographically secure MPC allows multiple hospitals to jointly analyze data that they cannot observe individually. As a consequence, each hospital need not entrust other hospitals or the cloud vendor with its patient's data. To achieve this ChRIS interacts with another open source project [Conclave Cloud Dataverse or C2D](https://github.com/cici-conclave/conclave-web). C2D uses OpenShift to create isolated computing environment for MPC job pods.

### How are isolated computing environments created in cloud?
We create several types of isolation by
* Creating machine like virtualisation using namespace system and SELinux policies that comes by default in Red Hat Enterprise Linux distribution.
* Using OpenShift projects to allow a community of users to organize and manage their content in isolation from other communities via API authentication.
* Allowing selective isolation using routes and services. Depending on the level of isolation, pods within a project communicates via services. Communication across different projects are done via routes.


### Secure MPC and ChRIS usecase
[This MPC](https://github.com/FNNDSC/pl-mpcs) plugin compares a patient’s brain volume with the population data collected from two different hospitals.

![PACSPull Plugin](/images/mpc/Feed-Detail-Screencapture-PACS-selected.png)

The above tree illustrates the entire workflow which consists of non MPC component (computed in ChRIS) and MPC component (executed in C2D).
The nodes are various plugins that will be executed in the order of their hierarchy. The first node is for “PACS Pull” plugin. It pulls the MRI scan data for the workflow from BCH database into the computing environment, MOC in this case. The output of this plugin is the input to the whole operation represented in the tree structure, a patient’s brain MRI.

![PACS Pull Output](/images/mpc/PACSPull_Output.png)

Next in chain is “FreeSurfer” plugin. It performs volume and surface based analysis of the MRI data and facilitates the visualization of the functional regions of the highly folded cerebral cortex.

![Free Surfer Plugin 3D Output](/images/mpc/Freesurfer-3D-Screencpature.png)
This is the 3D image output of FreeSurfer plugin. The colored regions depict various segments in the gray matter of the brain.

Next node is for secure MPC. The plugin calls C2D APIs to calculate population mean and population standard deviation using the population data pooled from parties involved in MPC. Once these figures are calculated, the results are pulled in ChRIS by using C2D API. Additional non MPC computation needs to be done using the MPC results. This non MPC computation includes projecting patient’s brain volume against the population mean and calculating number of standard deviations from the mean the patient datapoint is in-order to identify segments that have significant deviation from the population mean in terms of brain volume for the same age group. Both these computation is done within ChRIS.


![MPC output](/images/mpc/Volumet-screencapture-grids.png)

The above graph is one of the outputs and it extrapolates the patient’s data (who is 10 years old) against the population mean for the segment “G_and_S_frontomargin” for various age groups. The blue line is for left hemisphere and the orange line is for right hemisphere. We can infer from the graph that the patient’s brain volume for both hemispheres is lower than the population mean volume for 10 years olds.

![MPC output](/images/mpc/Segment-screencapture-grids.png)

To know which segments have significant deviation from the population mean in terms of brain volume for the same age group this graph is used. One can find that for this patient there are 4 segments which have conspicuous deviation when compared to other 10 year olds.
