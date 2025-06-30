
# STX Streaming Payment Contract

## Overview

This Clarity smart contract implements a **Streaming Payment** system on the Stacks blockchain. It allows a sender to create timed payment streams to a recipient, where funds are released incrementally per block over a specified timeframe. The recipient can withdraw their earned balance anytime, and the sender can add more funds or refund excess locked tokens after the stream ends.

---

## Features

* **Create Streams**: Lock an initial balance and define a payment schedule with start and stop blocks.
* **Incremental Payments**: Payments are calculated and released per block within the timeframe.
* **Refuel Streams**: Sender can add more locked STX to an existing stream.
* **Withdraw Payments**: Recipients can withdraw their accumulated payments at any time.
* **Refund Excess**: Sender can refund unused locked tokens after stream expiration.
* **Update Stream Details**: Sender and recipient can update payment rate and timeframe with off-chain signature verification.

---

## Error Codes

| Error Code                     | Description                       |
| ------------------------------ | --------------------------------- |
| `ERR_UNAUTHORIZED` (u0)        | Caller is not authorized          |
| `ERR_INVALID_SIGNATURE` (u1)   | Signature verification failed     |
| `ERR_STREAM_STILL_ACTIVE` (u2) | Stream timeframe has not ended    |
| `ERR_INVALID_STREAM_ID` (u3)   | Provided stream ID does not exist |

---

## Data Structures

### Stream Record

```clojure
{
  sender: principal,
  recipient: principal,
  balance: uint,
  withdrawn-balance: uint,
  payment-per-block: uint,
  timeframe: (tuple (start-block uint) (stop-block uint))
}
```

* **sender**: The principal who funds the stream.
* **recipient**: The beneficiary receiving incremental payments.
* **balance**: Total locked STX for the stream.
* **withdrawn-balance**: Amount already withdrawn by the recipient.
* **payment-per-block**: STX released per block.
* **timeframe**: Start and stop blocks defining the payment period.

---

## Contract Variables

* `latest-stream-id` — Tracks the latest stream ID issued.
* `streams` — A map storing all active and historical streams by ID.

---

## Public Functions

### `stream-to`

Create a new stream.

**Parameters:**

* `recipient` (principal): Receiver of the payment stream.
* `initial-balance` (uint): Amount of STX to lock.
* `timeframe` (tuple): Start and stop block numbers.
* `payment-per-block` (uint): STX released per block.

**Returns:** Stream ID (uint) on success.

---

### `refuel`

Add more locked STX to an existing stream.

**Parameters:**

* `stream-id` (uint): ID of the stream.
* `amount` (uint): Additional STX to add.

**Access:** Only sender can refuel.

---

### `balance-of`

Check the current balance available for withdrawal or refund for a given principal.

**Parameters:**

* `stream-id` (uint)
* `who` (principal)

**Returns:** uint — the balance owed to the principal.

---

### `withdraw`

Recipient withdraws earned STX.

**Parameters:**

* `stream-id` (uint)

**Access:** Only recipient can withdraw.

---

### `refund`

Sender withdraws unused locked STX after the stream timeframe ends.

**Parameters:**

* `stream-id` (uint)

**Access:** Only sender can refund.

---

### `update-details`

Update payment rate and timeframe of an existing stream with off-chain signature verification.

**Parameters:**

* `stream-id` (uint)
* `payment-per-block` (uint)
* `timeframe` (tuple)
* `signer` (principal)
* `signature` (buff 65)

---

## Internal / Read-Only Functions

* `calculate-block-delta`: Calculate how many blocks the stream has been active.
* `hash-stream`: Create a hash of the stream and updated parameters for signature verification.
* `validate-signature`: Verify secp256k1 signature for secure stream updates.

---

## Security Considerations

* Only the sender can add funds or refund locked tokens.
* Only the recipient can withdraw their earned payments.
* Updates to the stream parameters require off-chain signatures from sender or recipient for mutual consent.
* The contract locks STX tokens during the stream, ensuring funds are available before payments.

---

## Deployment & Usage

1. Deploy the contract on Stacks mainnet or testnet.
2. Use `stream-to` to create a payment stream.
3. The sender can add funds anytime with `refuel`.
4. The recipient can withdraw funds with `withdraw`.
5. After the stream ends, sender can call `refund` to reclaim unused funds.
6. Update stream details via `update-details` using signed messages.

---
