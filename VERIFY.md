# Verification Guide

**Contract:** CustosNetworkProxy (V5.3) — `0x9B5FD0B02355E954F159F33D7886e4198ee777b9` — Base mainnet

## Quick verify any cycle

```bash
# 1. Get the cycle file
cat proofs/2026-02-20/cycle-001.md

# 2. Extract the focus content (everything after "## Focus content")
sed -n '/^## Focus content$/,$ p' proofs/2026-02-20/cycle-001.md | tail -n +2 > /tmp/focus.txt

# 3. Compute hash
node -e "const {keccak256,toBytes}=require('viem'),fs=require('fs');console.log(keccak256(toBytes(fs.readFileSync('/tmp/focus.txt','utf8'))))"

# 4. Compare to proofHash in the file header

# 5. Verify prevHash links to previous cycle's proofHash
```

## Verify chain head on-chain

```bash
# Read current chain head from proxy
cast call 0x9B5FD0B02355E954F159F33D7886e4198ee777b9 \
  "getChainHeadByWallet(address)(bytes32)" \
  0x0528B8FE114020cc895FCf709081Aae2077b9aFE \
  --rpc-url https://mainnet.base.org
```

## Verify the full chain (local)

```bash
node -e "
const fs = require('fs');
const { keccak256, toBytes } = require('viem');

const files = fs.readdirSync('proofs', {recursive: true})
  .filter(f => f.endsWith('.md'))
  .sort();

let prevHash = '0x' + '0'.repeat(64);
for (const f of files) {
  const content = fs.readFileSync('proofs/' + f, 'utf8');
  const match = content.match(/\*\*Proof hash:\*\* (0x[0-9a-f]+)/);
  const prevMatch = content.match(/\*\*Prev hash:\*\* (0x[0-9a-f]+)/);
  const focusMatch = content.split('## Focus content')[1];
  if (!match || !focusMatch) continue;
  const computed = keccak256(toBytes(focusMatch.trim()));
  const storedPrev = prevMatch?.[1];
  console.log(f, computed === match[1] ? '✅' : '❌ HASH MISMATCH', storedPrev === prevHash ? '🔗' : '⛓️ CHAIN BREAK');
  prevHash = match[1];
}
"
```
