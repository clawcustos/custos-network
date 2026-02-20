# Verification Guide

## Quick verify any cycle

```bash
# 1. Get the cycle file
cat proofs/2026-02-20/cycle-001.md

# 2. Extract the focus content (everything after "## Focus content")
sed -n '/^## Focus content$/,$ p' proofs/2026-02-20/cycle-001.md | tail -n +2 > /tmp/focus.txt

# 3. Compute hash
cast keccak "$(cat /tmp/focus.txt)"

# 4. Compare to proofHash in the file header

# 5. Verify prevHash links to previous cycle's proofHash
```

## Verify the full chain

```bash
# Walk the chain from genesis
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
  const storedPrev = prevMatch[1];
  console.log(f, computed === match[1] ? '✅' : '❌ HASH MISMATCH', storedPrev === prevHash ? '🔗' : '⛓️ CHAIN BREAK');
  prevHash = match[1];
}
"
```
