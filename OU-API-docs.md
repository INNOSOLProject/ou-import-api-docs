# Innotwine University API - OU Access

The Innotwine University API is designed for developers at participating universities to automate the process of importing pre-defined sets of constitutent data files. With the API, you can create new "import sets," upload approved .csv files, query the status of imports, and initiate the actual ingestion of data into the platform.

<br />

# Authentication

Access to the API requires a valid OAuth token associated with a registered Innotwine user with University Admin privileges.

Tokens are expire 2 hours after being issued. You can retrieve an OAuth token by sending an HTTP request with your username and password to https://api.innosolpro.com/import/auth

```bash
curl -X POST https://api.innosolpro.com/import/auth \
-H "Content-Type:application/json" \
-F username=myusername@university.edu \
-F password=mypassword
```

#### Example Response JSON

```js
{
   "success" : true,
   "token" : // OAuth token
}
```

<br />

# Setup

All HTTP requests to the University API must include a special header - "Institution" - containing your intisutition's shortcode, as well as a bearer authentication request header containing the OAuth token described above.

**The institution shortcode for OU is `OU`**

<br />

# Overview

The University API is designed to work in three steps (see below for details about each route):

### 1. Create a new import set

Prepare backend to accept .csv files / get an import ID:

```bash
curl -X POST https://api.innosolpro.com/import \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:$institution_shortcode"
```

### 2. Upload file(s)

Upload individual .csv files with formatted data:

```bash
curl -X POST https://api.innosolpro.com/import/$importId/file \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:$institution_shortcode" \
-H "Content-Type:multipart/form-data" \
-F importType=Gift \ # must be one of established .csv types (e.g. 'Gift', 'Bio')
-F file=@$local_file_name
```

### 3. "Seal" import set

Signal that file uploads are complete and kick off data ingestion:

```bash
curl -X POST https://api.innosolpro.com/import/$importId/complete \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:$institution_shortcode"
```

<br />

# Details

## Creating new import sets

#### POST /import

An "import set" refers to a collection of templated .csv files of constituent-related data that will ultimately be ingested. Creating a new import set prepares the backend to receive files and provides a unique `importId` for subsequent requests.

#### Example Request

```bash
curl -X POST https://api.innosolpro.com/import \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:OU"
```

#### Example Response JSON

```js
{
   "success" : true,
   "importId" : // unique ID string for new import set
}
```

<br />

## Uploading files

#### POST /import/{importId}/file

This route is for uploading individual files and associating them with a specific import set. Files must be in .csv format and contain the appropriate headers/table structure for a given `importType`. This route only accepts `multipart/form-data` encoded request bodies.

Subsequent calls with the same `importType` will overwrrite the previous version of the file in the import set.

| Path Parameters | Description                     |
| --------------- | ------------------------------- |
| `importId`      | The unique id of an import set. |

<br />

| Body Parameters | Description                                                                                                                                              |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `importType`    | The type of pre-defined .csv file type contained in the request. The `importType` must conform to one of the file types associated with your university. |
| `file`          | .csv file to upload                                                                                                                                      |

<br />

The .csv file uploaded on each request must have a valid `importType` value. The local filename does not matter. Valid `importType`s are given below:

| `importType`             |
| ------------------------ |
| `Attribute`              |
| `Attribute_CODES`        |
| `Bio`                    |
| `Bio_CODES`              |
| `Contact_Info`           |
| `Degree`                 |
| `Degree_CODES`           |
| `Event`                  |
| `Event_CODES`            |
| `Free_Text`              |
| `Gift`                   |
| `Gift_CODES`             |
| `Matching_Gift`          |
| `Matching_Gift_CODES`    |
| `Pref_Name`              |
| `Proposal`               |
| `Prospect`               |
| `Prospect_CODES`         |
| `Relationship`           |
| `Relationship_CODES`     |
| `Soft_Credit_Gift`       |
| `Soft_Credit_Gift_CODES` |

<br />

#### Example Request

```bash
curl -X POST https://api.innosolpro.com/import/$importId/file \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:OU" \
-H "Content-Type:multipart/form-data" \
-F importType=Gift \ # must be one of established .csv types (e.g. 'Gift', 'Bio')
-F file=@$local_file_name
```

#### Example Response JSON

```js
{
   "success" : true,
   "fileTypeReceived" : "Gift",
   "importId" : // <import-id>
}
```

<br />

## Finalize import set and begin ingestion

#### POST /import/{importId}/complete

Once all .csv files have been uploaded, an import set must be "sealed." A sealed import set cannot accept additional .csv file uploads. Sealing the import set will initiate the actual ingestion of data into the Innotwine platform.

| Path Parameters | Description                     |
| --------------- | ------------------------------- |
| `importId`      | The unique id of an import set. |

#### Example Request

```bash
curl -X POST https://api.innosolpro.com/import/$importId/complete \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:OU"
```

#### Example Response JSON

```js
{
   "success" : true,
   "importId" : // <import-id>
}
```

<br />

## Query status

#### GET /import/{importId}

At any point after creating the import set, this route retrieves status information about the specified import set.

| Path Parameters | Description                     |
| --------------- | ------------------------------- |
| `importId`      | The unique id of an import set. |

#### Example Request

```bash
curl -X GET https://api.innosolpro.com/import/$importId \
-H "Authorization:Bearer $OAuth_token" \
-H "Institution:OU"
```

#### Example Response JSON

```js
{
   "importSet" : {
      "id" : // <import-id>
      "institution" : // <institution-shortcode>
      "filesReceived" : [
         {
            "fileType" : "Gift",
            "fileSize" : 277, // file size in bytes
            "uploadedAt" : 1548354162162 // UNIX epoch timestamp
         },
         {
            "fileType" : "Bio",
            "uploadedAt" : 1548354460674,
            "fileSize" : 277
         },
         {
            "fileType" : "Activities",
            "fileSize" : 277,
            "uploadedAt" : 1548354914307
         }
      ],
      "sealed" : false, // indicates whether the import set is complete and begun importing
      "createdAt" : 1548353947835 // UNIX epoch timestamp
   }
}
```
