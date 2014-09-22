# Notes #

## To do to implement DataOne Tier1 MemberNode API ##

See accompanying spreadsheet (in open document format): CKAN_DataOne.ods, in this GitHUB repository for more details and links.

Ping-- respond with an HTTP 200, and a current system clock timestamp

Review and use DataOne HTTP error codes in responses

Create node capabilities document according to XML [Types.Node](http://mule1.dataone.org/ArchitectureDocs-current/apis/Types.html#Types.Node) XML schema

Resolve how to do SID (version series, same intention and content model) and PID (individual versions, same bitstream). See discussion of identifies in 'Immutable content model' section, following this.

Harmonize CKAN event logging with required format for DataOne logs; have to distinguish GET and Replication requests and log separately.

generate SHA-1 and MD5 checksums for individual files (identified by PID)

Map CKAN Formats to [DataOne format registry](ent/apis/CN_APIs.html#CNCore.getFormat)

generate DataOne [Types.ObjectList](http://mule1.dataone.org/ArchitectureDocs-current/apis/Types.html#Types.ObjectList) to list packages; 

Generate DataOne SystemMetadata object in CKAN package; probably as an extra

Implement update procedure to update SystemMetadat

Map CKAN /api/3/action/package_search?... (the query) to DataOne Syntax for DataOne query requests

Create [DescribeQueryEngine](http://mule1.dataone.org/ArchitectureDocs-current/apis/Types.html#Types.QueryEngineDescription) XML document for CKAN query engine

Implement BagIt packaging for CKAN package; see https://tools.ietf.org/html/draft-kunze-bagit-10


##  Immutable content model

This information summarized with minor edits from [Mutability of Content in DataONE](https://mule1.dataone.org/ArchitectureDocs-current/design/ContentMutability.html#the-series-identifier).

DataONE currently follows an immutable content model, whereby member nodes are required to save any changes to an object as a new object with a new identifier which formally obsoletes the previous one. The challenge is to find the optimal way to support member nodes using a mutable content model (changes to content overwrite the existing content returned by an identifier), while preserving the reproducibility assurances the current approach offers.

The immutability requirement helps to ensure reproducible results of any use of an object. Any analysis on a data set repeated sometime in the future should yield identical results (within the limits of precision of the analytical tools) and this is one of the major guiding principles in creating DataONE as a long term data repository federation.

**Solution:** a series identifier (**SID** ) to facilitate the semantics of citing an object at the conceptual level, instead of the version level. As content changes over time, new identifiers (the **PID** ) will still be used to mark each change, but the conceptual object can continue to be referred to with an unchanging identifier (SID). *The member node will be responsible for creating each version and assigning a unique PID to it and these objects will be synchronized and replicated to other DataONE member nodes* as they are today. So instead of allowing content to be directly modified, we are allowing strongly-versioned chains to be referenced by an identifier; and relaxing the requirement that all revisions be resolvable forever.

The series identifiers is associated with all versions of an object, is unique in DataONE (assigned to only one version chain), and would be reserved just as PIDs - from the same namespace. The series identifier, once assigned to the version chain, is immutable, and will apply to all new versions of the item. It is also assumed that in order to use one identifier for citations, the cardinality for the series identifier (SID) is 0..1. The semantics for making API calls with a SID would, in general, be to return responses as if the call were made with the most current PID.

Member Nodes that only maintain the latest version of an item would be required to use a new PID for any updated content, and modify the System Metadata appropriately so that the new version can be synchronized with the network. The same SID would typically be used for the updated object, although the the client and/or member node may shift to a new SID for a revision chain at their discretion.

The current DataONE storage model, through the MN_Storage.update method, places responsibility for storing versions squarely on the submitter. Each update to the object requires a new unique identifier (PID) and must state which PID the new version is obsoleting. We will continue to require that unique PIDs are provided for each and every version of an object, but the member node will not be required to maintain a copy of previous revisions if it chooses not to. An optional series identifier (SID) can be provided with object SystemMetadata to group revisions together and to provide a convenient way to refer to the latest version of the object.

The member node must [minimally] maintain system metadata for the current revision of the object. Any updated object is still required to be identified by a new unique PID, but would include the same SID used in the previous version. The obsoletes field should indicate that the new PID replaces the previous PID. The coordinating node learns about the updated content during synchronization because there is:

* a new PID
* an updated dateSystemMetadataUpdated timestamp
* an updated checksum (other fields may also be updated).

