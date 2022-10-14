# Cypress Test Practice

The goal of this exercise is to explore and learn about the concept of end to end (e2e) testing, specifically using the [cypress](https://www.cypress.io/) testing framework.
The application under test is a sample Next.js with Redis application that can easily be generated using a script (see below).

In our fictitious scenario, we have a set of test cases (documented in the [test cases](#test-cases) section below) that are currently executed manually before each release of the software to ensure that the application does not have any regressions. Your task is to automate these tests.

You will need to

1. Follow the steps below to create and run the application
2. Get familiar with the API of the application
3. [Install](#install-cypress) and run cypress
4. Automate the [test cases](#test-cases) with the help of the [cypress documentation](https://docs.cypress.io/guides/overview/why-cypress)

## Pre requisites

You will need the following software on your machine in order to get the application up and running:

1. [Node.js](https://nodejs.org)
2. [Docker](https://www.docker.com/)
3. [Visual Studio Code](https://code.visualstudio.com/) (or another editor of your choosing)

## Create the application

```
npx create-next-app --example with-redis roadmap
cd roadmap
```

## Run Redis

```
docker run --rm -p 6379:6379 --name roadmap-redis -d redis
```

## Start the Next.js App

```
npm run dev
```

## Update logic for IP Address tracking

In order to successfully use automation to test we have found that you will need to make this change to the code, so it is provided here.

Open the `pages/api/create.ts` file and change the `ip` value to the following. This means replacing line 23 (or thereabouts, in case this has changed) from:

```typescript
      ip: 'NA',
```

to

```typescript
      ip: req.headers['x-forwarded-for'] || 'NA'
```

## Install Cypress

Follow the installation guide at https://docs.cypress.io/guides/getting-started/installing-cypress to get up and running with cypress.

# Using the API

## Create a new request

`POST`
http://localhost:3000/api/create

`BODY`

```
{
    "title": "a cool request"
}
```

`HEADERS`

```
    x-forwarded-for 192.168.0.1
```

## Listing all the features

`GET`
http://localhost:3000/api/features

## Voting on a feature

`POST`
http://localhost:3000/api/vote

```
{
    "id": "1aa12b33-a1af-4d38-8aab-0d85aa044d72",
    "title": "a cool request"
}
```

`HEADERS`

```
    x-forwarded-for 192.168.0.1
```

# Test Cases

This section lists the current manual test cases that we would like to automate.

**Note**: changes may need to be made to the application code to better facilitate the automation, for example:

- Adding data attributes or classes to specific HTML elements
- Adding end points that are only used by testing; for example: an endpoint that can flush redis between test runs

## 1. Adding a request

### Successfully adding a request

```
Actions:
- Type a request into the request input box
- Click the "Request" button

Acceptance Criteria:
- The input box text should be cleared
- The request should appear in the request list
- The request should automatically have your vote 👍
- The vote count on the request should be 1
```

### Request empty validation

```
Actions:
- Ensure the request input box is empty
- Click the "Request" button

Acceptance Criteria:
- A validation message should be shown to indicate that the request input is a mandatory field
```

### Maximum length validation

```
Actions:
- Type a request that exceeds 150 characters
- Click the "Request" button

Acceptance Criteria:
- A error toast should be displayed to indicate that the request should not exceed 150 characters
```

## 2. Subscribing with email address

### Successfully subscribing

```
Actions:
- Type a valid email address in the email address input (e.g. joesoap@gmail.com)
- Click the "OK" button

Acceptance Criteria:
- The email input box text should be cleared
- A success toast should be displayed indicating that the subscription was successful
```

### Email validation: required field

```
Actions:
- Ensure the email input box is empty
- Click the "OK" button

Acceptance Criteria:
- A validation message should be shown to indicate that the email input is a mandatory field
```

### Email validation: @ symbol

```
Actions:
- Type email address without a @ symbol (e.g. joesoap)
- Click the "OK" button

Acceptance Criteria:
- A validation message should be shown to indicate that a '@' is required in the email address
```

### Email validation: part after @ symbol

```
Actions:
- Type email address without the part following the '@' symbol (e.g. joesoap@)
- Click the "OK" button

Acceptance Criteria:
- A validation message should be shown to indicate that the part following the '@' symbol is required
```

## 3. Voting

### Voting on someone elses request

```
Actions:
- Using the 'create' endpoint, add a request using a different ip address to your local ip (see documentation above about the 'vote' endpoint)
- Visit the home page
- The request should appear in the request list
- The vote count on the request should be 1
- Click the vote button to vote for the request

Acceptance Criteria:
- The vote count on the request should increase to 2
- The vote vote button 👍 should indicate that it has been clicked (with a green background)
```

### Others voting on your request

```
Actions:
- Type a request into the request input box
- Click the "Request" button
- Using the 'vote' endpoint, vote for the request using a different ip address to your local ip

Acceptance Criteria:
- The vote count on the request should increase to 2

```

### Feature ordering

```
Actions:
- Using the 'create' endpoint, add 3 different requests
- Visit the home page
- Ensure that each feature was added to the feature list
- Using the 'features' endpoint, query the features and note the 'id' of each feature that was added
- Using the 'vote' endpoint, vote for the 3rd feature in the list
- Reload the web page
- Validate that the 3rd feature is now at the top of the list
- Using the 'vote' endpoint, vote for the 2nd feature twice (with 2 different ip addresses)
- Reload the web page

Acceptance Criteria:
- Validate that the 2nd feature is now first in the list, followed by the 3rd feature and the 1st feature is now last in the list.

```

## 4. Feature released

### Extend the application

```
- Extend the 'create' endpoint to persist a 'released' flag as false by default
- Update the 'Feature' interface to include the 'released' property
- Update the rendering of the requests to use the 'released' flag value for the 'isReleased' property of the 'Item' component
- Create a 'release' endpoint that accepts a 'id' and updates the release flag of the persisted feature to be true
```

### Shows the feature as released

```
Actions:
- Create a request either through the UI or the 'create' endpoint
- Use the 'release' endpoint to flag the request as released
- Reload the web page

Acceptance Criteria:
- The vote button for the feature should now be '✅'
```
