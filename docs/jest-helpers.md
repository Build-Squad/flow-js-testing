---
title: Jest Helpers
sidebar_title: Jest Helpers
description: Helper methods for Jest
---

In order to simplify the process even further we've created several Jest-based methods, which will help you to catch
thrown errors and ensure your code works as intended.

## `shallPass(ix)`

Ensure transaction did not throw and was sealed.

#### Arguments

| Name | Type                                | Description                                          |
| ---- | ----------------------------------- | ---------------------------------------------------- |
| `ix` | [Interaction](./api.md#interaction) | interaction, either in form of a Promise or function |

#### Returns

| Type                                                                        | Description        |
| --------------------------------------------------------------------------- | ------------------ |
| [ResponseObject](https://docs.onflow.org/fcl/reference/api/#responseobject) | Transaction result |

#### Usage

```javascript
import path from "path"
import {
  init,
  emulator,
  shallPass,
  sendTransaction,
  getAccountAddress,
} from "@onflow/flow-js-testing"

// We need to set timeout for a higher number, because some transactions might take up some time
jest.setTimeout(10000)

describe("interactions - sendTransaction", () => {
  // Instantiate emulator and path to Cadence files
  beforeEach(async () => {
    const basePath = path.resolve(__dirname, "./cadence")
    await init(basePath)
    return emulator.start()
  })

  // Stop emulator, so it could be restarted
  afterEach(async () => {
    return emulator.stop()
  })

  test("basic transaction", async () => {
    const code = `
      transaction(message: String){
        prepare(singer: AuthAccount){
          log(message)
        }
      }
    `
    const Alice = await getAccountAddress("Alice")
    const signers = [Alice]
    const args = ["Hello, Cadence"]

    const [txResult, error] = await shallPass(
      sendTransaction({
        code,
        signers,
        args,
      })
    )

    // Transaction result will hold status, events and error message
    console.log(txResult, error)
  })
})
```

## shallRevert(ix, message)

Ensure interaction throws an error. Can test for specific error messages or catch any error message if `message` is not provided.
Returns Promise, which contains result, when resolved.

#### Arguments

| Name                     | Type                                | Description                                                                                                              |
| ------------------------ | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `ix`                     | [Interaction](./api.md#interaction) | transaction, either in form of a Promise or function                                                                     |
| `message` **(optional)** | `string` or `RegExp`                | expected error message provided as either a string equality or regular expression to match, matches any error by default |

#### Returns

| Type                                                                        | Description        |
| --------------------------------------------------------------------------- | ------------------ |
| [ResponseObject](https://docs.onflow.org/fcl/reference/api/#responseobject) | Transaction result |

#### Usage

```javascript
import path from "path"
import {
  init,
  emulator,
  shallPass,
  sendTransaction,
  getAccountAddress,
} from "js-testing-framework"

// We need to set timeout for a higher number, cause some interactions might need more time
jest.setTimeout(10000)

describe("interactions - sendTransaction", () => {
  // Instantiate emulator and path to Cadence files
  beforeEach(async () => {
    const basePath = path.resolve(__dirname, "./cadence")
    await init(basePath)
    return emulator.start()
  })

  // Stop emulator, so it could be restarted
  afterEach(async () => {
    return emulator.stop()
  })

  test("basic transaction", async () => {
    const code = `
      transaction(message: String){
        prepare(singer: AuthAccount){
          panic("You shall not pass!")
        }
      }
    `
    const Alice = await getAccountAddress("Alice")
    const signers = [Alice]
    const args = ["Hello, Cadence"]

    const [txResult, error] = await shallRevert(
      sendTransaction({
        code,
        signers,
        args,
      })
    )

    // Transaction result will hold status, events and error message
    console.log(txResult, error)
  })
})
```

## shallResolve(ix)

Ensure interaction resolves without throwing errors.

#### Arguments

| Name | Type                                | Description                                          |
| ---- | ----------------------------------- | ---------------------------------------------------- |
| `ix` | [Interaction](./api.md#interaction) | interaction, either in form of a Promise or function |

#### Returns

| Type                                                                        | Description        |
| --------------------------------------------------------------------------- | ------------------ |
| [ResponseObject](https://docs.onflow.org/fcl/reference/api/#responseobject) | Transaction result |

#### Usage

```javascript
import path from "path"
import {init, emulator, shallPass, executeScript} from "js-testing-framework"

// We need to set timeout for a higher number, cause some interactions might need more time
jest.setTimeout(10000)

describe("interactions - sendTransaction", () => {
  // Instantiate emulator and path to Cadence files
  beforeEach(async () => {
    const basePath = path.resolve(__dirname, "./cadence")
    await init(basePath)
    return emulator.start()
  })

  // Stop emulator, so it could be restarted
  afterEach(async () => {
    return emulator.stop()
  })

  test("basic script", async () => {
    const code = `
      pub fun main():Int{
        return 42
      }
    `

    const [result, error] = await shallResolve(
      executeScript({
        code,
      })
    )

    expect(result).toBe(42)
    expect(error).toBe(null)
  })
})
```

## shallHavePath(account, path)

Asserts that the given account has the given path enabled.

#### Arguments

| Name      | Type     | Description                                               |
| --------- | -------- | --------------------------------------------------------- |
| `account` | `string` | The address or name of the account to check for the path. |
| `path`    | `string` | The path to check for.                                    |

#### Returns

| Type            | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| `Promise<void>` | A Promise that resolves when the assertion is complete, or rejects with an error if it fails. |

#### Usage

```javascript
import path from "path"
import {init, emulator, shallPass, executeScript} from "js-testing-framework"

// We need to set timeout for a higher number, cause some interactions might need more time
jest.setTimeout(10000)

describe("interactions - sendTransaction", () => {
  // Instantiate emulator and path to Cadence files
  beforeEach(async () => {
    const basePath = path.resolve(__dirname, "./cadence")
    await init(basePath)
    return emulator.start()
  })

  // Stop emulator, so it could be restarted
  afterEach(async () => {
    return emulator.stop()
  })

  describe("check path with Jest helper", () => {
    test("pass account address", async () => {
      const Alice = await getAccountAddress("Alice")
      await shallHavePath(Alice, "/storage/flowTokenVault")
    })

    test("pass account name", async () => {
      await shallHavePath("Alice", "/storage/flowTokenVault")
    })
  })
})
```

## shallHaveStorageValue(account, params)

Asserts that the given account has the expected storage value at the given path.

#### Arguments

| Name              | Type                                            | Description                                                                                            |
| ----------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `account`         | `string`                                        | The address or name of the account to check for the storage value.                                     |
| `params`          | `{pathName: string, key?: string, expect: any}` | An object containing the path name, optional key, and expected value of the storage at the given path. |
| `params.pathName` | `string`                                        | The path of the storage value to retrieve.                                                             |
| `params.key`      | `string` (optional)                             | The key of the value to retrieve from the storage at the given path, if applicable.                    |
| `expect`          | `any`                                           | The expected value of the storage at the given path and key (if applicable).                           |

#### Returns

| Type            | Description                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------- |
| `Promise<void>` | A Promise that resolves when the assertion is complete, or rejects with an error if it fails. |
