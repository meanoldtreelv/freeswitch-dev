#Clustered File System Selection

## CEPH

Ceph is much better suited to a highly avalible cluster compared to other systems, as it also supports the S3 API natively and will not require a retooling of existing codebases to transition.


## Use Cases
One of the main benifits that CEPH brings to the table is the way that it handles file access. By supporting both POSIX compliant filesystem access and object storage protocols (S3/SWIFT) it can act as a bridge between the Freeswitch cluster and the SSP frontend.

- Static File storage
Static audio files can be placed onto the storage cluster from SSP through the use of an API command, the audio file can then be retrieved via the CEPHfs filesystem on the FreeSWITCH cluster

- Voicemail Recordings
Recording files written by FreeSWITCH can be retrieved via an API call in SSP without the need for disk scraping.

- Call Recordings
Currently we are offloading callrecordings to AWS S3, we would be able to pivot the location of those recordings onto CEPH reducing our AWS billing and keeping the files within our own ecosystem (a HUGE requirement for certain certifications such as SOC 2)

### Storage space
Moving forward with a clustered FreeSWITCH solution we should not

## Requirements

CEPH is designed to interface directly with block storage devices, using its own filesystem format (BlueStore). As the base building block of the storage cluster (the OSD) is size independent solutions such as FC SANs and large RAID arrays are in direct opposition to the core design of CEPH.

### Using an XtremeIO SAN
There was initial discussion of using an XtremeIO Xbric SSD SAN as backing storage for the CEPH cluster. This particular SAN is designed for very fast preformance and extremely low latency for medium to large file access (hundreds of megabytes to hundreds of gigabytes). Additionaly it uses internal hashing and bit level deduplication that is transparent to the connected devices.

In the uses cases where CEPH would be deployed, the average file size to be retrieved will be less than 10MB, and a majority of files will be in the KB range.

Additionally, by placing all of our files on a single SAN we introduce a singular point of failure that CEPH is specifically engineered to avoid. In the CEPH ecosystem files are replicated across multiple hosts so that in a loss event (disk, controller, host, rack, datacenter) there remains at minimum one copy of the file which is immediately replicated out to other hosts to prevent complete data loss. 

#### *This particular SAN may be a __MUCH__ better fit to house the CDR databases.*


### Reccomended Configuration
The reccomended configuration for CEPH storage nodes is a server with DAS, in the form of drive bays or a disk shelf, that are able to be addressed directly by the operating system in which CEPH is running. With direct access the CEPH daemon can tune drive performance as well as alert and act on rising failure rates before a hardware failure occours.

### Hardware Lifecycle
Unline traditional SANs, CEPH is intentiionally hardware agnostic and allows for a configured cluster to be upgraded in place, as budget and time allows. Older or less reliable hardware can have their storage resources drained and be removed from the cluster without affecting overall reliability or avalibiliy.

For the initial production cluster buildout hardware can be easily reclaimed and run from Scale LA, Scale SD, or (even) the Poway Office. Age and Runtime of the hardware is not a large concern as new hardware and block devices can be sourced as the project progresses rather than be invested into upfront.
