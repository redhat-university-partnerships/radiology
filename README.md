# Radiology
The radiology project is a collaboration with Boston Children's Hospital, Boston University, and Red Hat to improve the scale and performance of [ChRIS](https://github.com/FNNDSC/).

## ChRIS
ChRIS (Children’s Research Integration System) is a web-based medical image platform that allows for various forms of medical image (Ex: MRIs) processing. ChRIS itself is comprised of multiple open source projects hosted on [GitHub](https://github.com/FNNDSC/) with the intention to make the research and capabilities available to other hospitals and institutions.

## Massachusetts Open Cloud
The [MOC](https://massopen.cloud/) is a cloud computing environment built in collaboration with 5 Massachusetts area universities (Boston University, Harvard, MIT, Northeastern, UMass) used primarily for educational and research purposes. The MOC, however, has the goal of expanding its footprint/usage by providing vertical solutions, such as the ChRIS project, to other institutions with similar needs.

## Goals
The desire of Boston Children’s Hospital, Boston University and Red Hat is to help improve the scale and efficiency of the ChRIS platform using the MOC and various Red Hat technologies including OpenShift and OpenStack. Today, processing a single set of images takes ~10 hrs. The goal is to get that down to a few mins to give the patients immediate feedback.

## Architecture
ChRIS is comprised of several components/services and designed to have multiple heterogeneous clusters available for image processing. In the example shown below, the datacenter providing the image processing is the MOC which is backed by OpenShift running on OpenStack. The primary interfaces into the MOC are pman and pfioh which provide the API for pfcon to communicate with where pfioh handles the input/output and pman handles the image processing requests. The image plugins themselves run as container jobs on OpenShift.

![Chris Architecture](chris_architecture.png)

### Explanation of Terms
  * [ChRIS Frontend](https://github.com/FNNDSC/ChRIS_ultron_frontEnd) - The web-based interface for ChRIS
  * [ChRIS Backend](https://github.com/FNNDSC/ChRIS_ultron_backEnd) - The ChRIS REST API
  * [pfcon](https://github.com/FNNDSC/pfcon) - An abstraction API for ChRIS API to manage calls to different datacenters running pfioh and pman
  * [pfioh](https://github.com/FNNDSC/pfioh) - Handles input and output of image data into a data center
  * [pman](https://github.com/FNNDSC/pman) - Handles instantiating and checking the status of image processing jobs
  * Image Plugin(s) - An image processor. Turns raw image data into something meaningful for the radiologists/doctors. Ex: [FreeSurfer](https://surfer.nmr.mgh.harvard.edu/)