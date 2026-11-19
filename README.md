# PoC test dApp — Immutable Passport (F-1)

Purpose: read the TEST account's exact addresses + idToken via the Passport SDK,
as page text (the cloud browser cannot use DevTools, clipboard paste, or localStorage).

## Status (19 Nov 2026 ~18:33 WITA)
- Test account created + logged in on play.immutable.com (SecureChipmunk3687,
  sucimaru@bugcrowdninja.com). Wallet shown only truncated: `0x04ab...7778`.
- Address extraction FAILED via: Play UI (truncated), DevTools/localStorage
  (blocked in this browser env), copy-paste into Send field (Ctrl+V refused),
  reading Chromium profile LevelDB from the VM (no leveldb dirs; cloud browser
  runs in a separate sandbox).
- DevHub route STARTED 18:30 (browser task 18530f3e): hub.immutable.com sign-in
  needs a 6-digit email code to sucimaru@bugcrowdninja.com — STOPPED, waiting on
  Suci to send the code or enter it via takeover. No project/client created yet.

## Remaining steps
1. Suci supplies the DevHub email code (or enters it via takeover).
2. Browser creates DevHub project `bugcrowd-poc-test` + Passport OAuth client.
   Record: CLIENT_ID, PUBLISHABLE_KEY, environment (Sandbox vs Production).
3. Fill the `__PLACEHOLDERS__` at the top of `index.html`.
4. Host this page somewhere the cloud browser can reach. Plan: new public GitHub
   repo `Sucimaru/immutable-poc-test-dapp` + GitHub Pages (localhost serving will
   NOT work — the cloud browser is not on this VM). Redirect URI = the Pages URL
   of this page (registered exactly in the DevHub client).
5. Browser opens the Pages URL, clicks Login (SSO, already logged in), then
   "Show getUserInfo + getIdToken". Report the page text: zkevm_eth_address,
   zkevm_user_admin_address, and the raw idToken (idToken is the TEST account's
   short-lived bearer token — handle as a credential: use transiently for the TEE
   replay call only, never store in chat/memory/files).
6. Sanity check: compare returned zkevm_eth_address prefix with `0x04ab...7778`
   (mainnet Play wallet). If Sandbox returns a DIFFERENT address, note which
   environment it came from — the fork step (step 4) needs the MAINNET address,
   so a Production client may be needed instead.

## Then (PoC steps 3–4)
- Step 3 (TEE replay): POST https://tee.express.magiclabs.com/v1/wallet/sign/message
  with `Authorization: Bearer <idToken>` + the hardcoded public headers from
  FINDINGS.md, digest = keccak256("bugcrowd-poc"). Verify returned signature
  ecrecovers to zkevm_user_admin_address. If it fails → F-1 is DEAD, report as
  disproven, do NOT submit.
- Step 4 (fork execute): Anvil fork of https://rpc.immutable.com (chain 13371),
  call SCW execute() with a threshold-1 EOA signature, show Test-IMX/NFT move
  without Guardian. Test-IMX via DevHub faucet (testnet only).
- NO mainnet tx, NO victim funds, ever.
