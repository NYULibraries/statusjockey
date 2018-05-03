# StatusJockey

[![Coverage Status](https://coveralls.io/repos/github/NYULibraries/statusjockey/badge.svg?branch=master)](https://coveralls.io/github/NYULibraries/statusjockey?branch=master)

Incidents response filter for the [Statuspage.io Incidents API](http://doers.statuspage.io/api/v1/incidents/)

# Features

* Makes API calls to:
  * `/pages/[page_id]/incidents.json`
  * `/pages/[page_id]/incidents/unresolved.json`
  * `/pages/[page_id]/incidents/scheduled.json`
* Can filter and manipulate the response based on the following criteria:
  * `filterByStatus`: only include incidents which match specific statuses.
  * `filterByComponents`: only include incidents which match specific affected components
  * `maps`: map an incident datum to a new key
  * `keys`: only include specific keys (including keys generated by `maps`)
* Returns a JavaScript `Promise`

# Installation

```bash
yarn add statusjockey
```

# Usage

### Configuration

```js
// statusJockeyConfig.js
modules.export = {
    // only return incidents with the following statuses.
    // Otherwise, return all.
    filterByStatus: ["investigating", "completed"],
    // Only return incidents with the following components listed.
    // Otherwise, return all.
    filterByComponents: [null],
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
### Implementation

```js
// main.js
const statusjockey = require('statusjockey');
const API_KEY = process.env["STATUSPAGE_API_KEY"];
const pageFiltersConfig = require('./statusJockeyConfig.js')

statusjockey(
  // perform a fetch with the page id abcd1234, with only scheduled responses
  // limit to the most recent response
  { page_id: 'abcd1234', type: 'scheduled', limit: 1 },
  pageFiltersConfig,
  API_KEY
) // returns a Promise
.then(incidentsList => { // incidentsList is an array of incidents
  // do stuff
});
```
