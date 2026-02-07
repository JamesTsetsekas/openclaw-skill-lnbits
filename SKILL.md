---
name: lnbits
description: Manage LNbits Lightning Wallet (Balance, Pay, Invoice)
homepage: https://lnbits.com
metadata: {"clawdbot":{"emoji":"⚡","requires":{"bins":["python3"],"pip":["qrcode[pil]"],"env":["LNBITS_API_KEY", "LNBITS_BASE_URL"]},"primaryEnv":"LNBITS_API_KEY"}}
---

# LNbits Wallet Manager

Enable the assistant to safely and effectively manage an LNbits Lightning Network wallet.

## 🛑 CRITICAL PROTOCOLS 🛑

1.  **NEVER Expose Secrets**: Do NOT display Admin Keys, User IDs, or Wallet IDs.
2.  **Explicit Confirmation**: You MUST ask for "Yes/No" confirmation before paying.
    *   *Format*: "I am about to send **[Amount] sats** to **[Memo/Dest]**. Proceed? (y/n)"
3.  **Check Balance First**: Always call `balance` before `pay` to prevent errors.
4.  **ALWAYS Include Invoice + QR**: When generating an invoice, you MUST: (a) show the `payment_request` text for copying, and (b) output `IMAGE:` followed by the `qr_file` path on ONE line. NEVER skip this.

## Usage

### 0. Setup / Create Wallet
If the user does not have an LNbits wallet, you can create one for them on the demo server.

```bash
python3 {baseDir}/scripts/lnbits_cli.py create --name "My Wallet"
```

**Action**:
1.  Run the command.
2.  Capture the `adminkey` (Admin Key) and `base_url` (defaults to https://demo.lnbits.com).
3.  **IMPORTANT**: Instruct the user to save these credentials securely:
    > "I've created a new wallet! Please add these to your Moltbot configuration or `.env` file:
    > `export LNBITS_BASE_URL=https://demo.lnbits.com`
    > `export LNBITS_API_KEY=<adminkey>`"

### 1. Check Balance
Get the current wallet balance in Satoshis.

```bash
python3 {baseDir}/scripts/lnbits_cli.py balance
```

### 2. Create Invoice (Receive)
Generate a Bolt11 invoice to receive funds. **QR code is always included by default.**
*   **amount**: Amount in Satoshis (Integer).
*   **memo**: Optional description.
*   **--no-qr**: Skip QR code generation (if not needed).

```bash
# Invoice with QR code (default)
python3 {baseDir}/scripts/lnbits_cli.py invoice --amount 1000 --memo "Pizza"

# Invoice without QR code
python3 {baseDir}/scripts/lnbits_cli.py invoice --amount 1000 --memo "Pizza" --no-qr
```

**⚠️ MANDATORY RESPONSE FORMAT**: When generating an invoice, your response MUST include:

1. **Invoice text for copying**: Show the full `payment_request` string so user can copy it
2. **QR code image**: Output `IMAGE:` followed by the `qr_file` path on ONE line

**EXACT FORMAT** (follow precisely):
```
Here is your 100 sat invoice:

lnbc1u1p5abc123...

IMAGE:/tmp/lnbits_qr/invoice_xxx.png
```

**CRITICAL**: The `IMAGE:` and file path MUST be on the SAME LINE. This sends the QR code image to the user.

### 2b. Generate QR Code from Existing Invoice
Convert any Bolt11 string to a QR code image file.

```bash
python3 {baseDir}/scripts/lnbits_cli.py qr <bolt11_string>
```

Returns: `{"qr_file": "/tmp/lnbits_qr/invoice_xxx.png", "bolt11": "..."}`

### 3. Pay Invoice (Send)
**⚠️ REQUIRES CONFIRMATION**: Decode first, verify balance, ask user, then execute.

```bash
# Step 1: Decode to verify amount/memo
python3 {baseDir}/scripts/lnbits_cli.py decode <bolt11_string>

# Step 2: Pay (Only after user CONFIRMS)
python3 {baseDir}/scripts/lnbits_cli.py pay <bolt11_string>
```

## Error Handling
If the CLI returns a JSON error (e.g., `{"error": "Insufficient funds"}`), summarize it clearly for the user. Do not show raw stack traces.