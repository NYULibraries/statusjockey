# status-jockey

[![Coverage Status](https://coveralls.io/repos/github/NYULibraries/status-jockey/badge.svg?branch=master)](https://coveralls.io/github/NYULibraries/status-jockey?branch=master)
[![npm version](https://img.shields.io/npm/v/status-jockey.svg)](https://www.npmjs.com/package/status-jockey)

Incidents response filter for the [Statuspage.io Incidents API](http://doers.statuspage.io/api/v1/incidents/)

Note: Version 2 makes a huge number of breaking changes to the usage of this package. V1 documentation can be found [in the wiki](https://github.com/NYULibraries/statusjockey/wiki/Version-1-README)

# Features

* Makes API calls to the `Status` and `Manage` APIs at Statuspage.io. See more information comparing these two APIs [here](https://help.statuspage.io/knowledge_base/topics/api-information)
* Can filter and manipulate the response based on the following criteria:
  * `filterByStatus`: only include incidents which match specific statuses.
  * `customFilter`: only include incidents which match a provided filter function
  * `maps`: map an incident datum to a new key
  * `keys`: only include specific keys (including keys generated by `maps`)
* Returns `Promise` from [axios](https://github.com/axios/axios)-based fetch
* Function imports optimized for [tree shaking](https://webpack.js.org/guides/tree-shaking/) for minimized browser use in Webpack.

# Installation

```bash
yarn add status-jockey
# OR
npm install status-jockey
```

# Usage

## Configuration

```js
const config = {
    // only return incidents with the following statuses.
    // Otherwise, return all.
    filterByStatus: ["investigating", "completed"],
    // Only return incidents which return true for a filter function
    customFilter: ({ backfilled }) => backfilled,
    // Maps incident values to custom key values.
    maps: {
      // can map each entry to a new key based on a string identifying old key
      title: "name",
      incident_link: "shortlink",
      // or based on a function which takes the incident item object
      most_recent_message: ({ incident_updates }) => incident_updates[0].body,
      status: ({ status }) => {
        switch (status) {
          case "investigating":
            return "red";
          case "identified":
            return "orange";
          case "monitoring":
            return "green";
          default:
            return "red";
        }
      },
      hashtags: ({ incident_updates }) => {
        const body = incident_updates[0].body;
        return body && body.match(/#\w+/g) ? body.match(/#\w+/g).map(v => v.replace('#', '')) : [];
      }
    },
    // Only return the following keys for each incident
    // Non-mapped keys are given their default values.
    keys: ["id", "most_recent_message", "incident_link", "created_at", "status", "hashtags"],
  }
};
```

## `statusApi`: Public Status API (v2)

`statusApi(config) -> statusApiFetcher({ page_id, type, limit })`

* `page_id` (required): Queries the incidents of the corresponding statuspage.io page id.
* `type` (optional): Queries one `.json` files specified in the [Statuspage public API](https://nyulibraries.statuspage.io/api) (default: `incidents`)
  * `components`
  * `incidents`
  * `incidents/unresolved`
  * `incidents/unresolved`
  * `scheduled-maintenances`
  * `scheduled-maintenances/upcoming`
  * `scheduled-maintenances/active`
* `limit` (optional): Limits to the X most recent incidents in response (default: no limit).

```js
const config = {
  // your configuration
}

const { statusApi } = require('status-jockey');
const fetchStatusApi = statusApi(config)

fetchStatusApi({ page_id: 'abcd123', type: 'incidents', limit: 5 })
  .then(dataList => {
    // do something with dataList (array of incidents/maintenances)
  });

```

## `manageApi`: Manage API (v1)
`manageApi(config) -> manageApiFetcher({ page_id, type, limit })`

* `page_id` (required): Queries the incidents of the corresponding statuspage.io page id.
* `type` (optional): Queries `scheduled`, `unresolved`, or `incidents` (default: `incidents`)
* `limit` (optional): Limits to the X most recent incidents in response (default: no limit).

```js
const config = {
  // your configuration
}

const { manageApi } = require('status-jockey');

const key = '1234567890' // SECRET Statuspage.io API key goes here
const fetchManageApi  = manageApi(config, key)=

fetchManageApi({ page_id: 'abcd123', type: 'incidents', limit: 5 })
  .then(dataList => {
    // do something with dataList (array of incidents/maintenances)
  })
  .catch(err => {
    // handle error
  })
```

## `incidentsFilter`

If you would like to fetch the data yourself but utilize the `status-jockey` response filters. This is useful if would like to exclude [axios](https://github.com/axios/axios) from your browser package.

#### Example (pre-Webpacked JS)
```js
import { incidentsFilter } from 'status-jockey' // tree shaking will exclude other imports from the package
const config = {
  // your configuration
}

const filterIncidents = incidentsFilter(config);

$.ajax('http://abcdefg.statuspage.io/api/v2/incidents/unresolved.json')
  .then({ incidents } => {
    // do something...
    return incidents
  }.then(filterIncidents)
  .catch(err => {
    // handle error
  })
```
