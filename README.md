# cypress-auto-stub-example [![Build Status](https://travis-ci.org/PinkyJie/cypress-auto-stub-example.svg?branch=master)](https://travis-ci.org/PinkyJie/cypress-auto-stub-example)

This project is an example which shows how to automatically stub all API requests happened in your [Cypress](https://www.cypress.io/) test and how to record/replay them.

## Problem to solve

One concern of running E2E tests is that it takes long time to run, which makes sense because all your API calls are targeted to the real server. To resolve this issue, a single solution is to use mock API in your E2E tests. That's good, but here comes another question: you need to manually update your mocking data according to the API change regularly.

A better solution is:

- Use real API response when you write E2E tests, and **Record these responses and write them into files**.
- Use the recorded API response to run your tests in future.

This project demonstrates how to make this process more smoothly and automatically with Cypress.

## Try it

- Clone this project.
- Install all dependencies by `yarn install` or `npm install`.
- Launch a website by `yarn serve` or `npm run serve`.
- Opn Cypress in recording mode: `yarn cy:open:record`, after running the test `network.spec.ts`, a snapshot file containing all API responses will be generated in `fixture` folder.
- Close Cypress and re-run it without recording: `yarn cy:open`, this time you can see the whole test runs very fast, because all API are stubbed and use the fixture file as responses.
- You can also use `yarn cy:run` to run all Cypress tests in headless mode.

## More details for demo

The website page we are testing in this project contains 3 examples:

- Example 1:
  - Click a button to call API (GET https://reqres.in/api/users) using XHR. API response JSON will be displayed below after it returns.
- Example 2:
  - Click a button to call API (POST https://reqres.in/api/users) using Fetch. API response JSON will be displayed below after it returns.
  - To demonstrate Fetch can be supported here.
- Example 3:
  - Click a button to call 30 **sequential** API (GET https://reqres.in/api/users/1 to https://reqres.in/api/users/10 for 3 times) using Fetch. API response JSON (`userId: 10`) will be displayed below after it returns. **sequential** here mean the 2nd API call will file only after the 1st API call returns.
  - To demonstrate the usage of a new custom command `cy.waitUntilAllAPIFinished()` which will make sure the testing is blocked until all API calls are finished.

## More detail for implementation

> Inspired by [this article](https://medium.com/ax2-inc/dynamic-xhr-responses-recording-stubbing-with-cypress-9257d4f730cd) and Jest snapshot testing.

- Check the main implementation in [`cypress/support/autoStubAPI.ts`](cypress/support/autoStubAPI.ts)

  1. use `cy.route` to monitor all API requests
  2. use callback `onResponse` in `cy.server` to record all API responses in an array
  3. use `cy.writeFile` to write all recorded API responses in a fixture file
  4. while replaying, use `cy.fixture` to load API responses from the fixture file

- Check the implementation for custom command `cy.waitUntilAllAPIFinished` in [`cypress/support/util.ts`](cypress/support/util.ts)
  1. maintain an internal counter for API calls: increase 1 when filing a new API request, decrease 1 when receiving a API response
  2. use Cypress's (Automatic Retry Assertion)[https://docs.cypress.io/guides/core-concepts/introduction-to-cypress.html#Asserting-in-English] to regularly check this counter, block the testing until it equals to 0

There are 4 new configuration parameters introduced in this example:

> All configuration parameters are defined under `env` as environment variables because:
>
> - It's easier for overriding by `cypress open --env foo=bar`
> - All existing configurations provided by Cypress are strongly typed, you can't add new configuration type when you use Typescript.

| Parameter         | Type                                       | Description                                                                                             | Required           | Default Value |
| ----------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------- | ------------------ | ------------- |
| autoRecordEnabled | boolean                                    | enable auto record or not                                                                               |                    | false         |
| apiHost           | string                                     | URL for API endpoint                                                                                    | :white_check_mark: |               |
| stubAPIPattern    | string (will be used in `new RegExp(xxx)`) | API pattern needs to be stubbed                                                                         | :white_check_mark: |               |
| apiMaxWaitingTime | number (in milliseconds)                   | used by `cy.waitUntilAllAPIFinished`, the maximum time when we wait for all API requests to be finished |                    | 60000 (60s)   |