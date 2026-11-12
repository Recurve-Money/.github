<div align="center">
  <img src="./banner.jpg" width="100%" alt="A human hand and a machine hand reaching toward each other at sunrise"/>
</div>

<div align="center">

  ![Chain](https://img.shields.io/badge/chain-Robinhood%20L2-2EE6A8)
  ![Contracts](https://img.shields.io/badge/contracts-Solidity%200.8.28-363636)
  ![Vault](https://img.shields.io/badge/vault-ERC--4626-6E56CF)
  ![Tests](https://img.shields.io/badge/tests-Foundry%2C%20unit%20%2B%20fuzz-3FB950)
  ![Status](https://img.shields.io/badge/status-pre--launch-8B949E)

</div>

---

## What is Recurve

An AI agent can already read a market, size a position, and execute it. What it cannot do is raise.

There is no standard way for an agent to pool outside capital, no primitive that holds that capital safely onchain, and no record it can point to when asking for more. The intelligence arrived years before the balance sheet did.

Recurve is the balance sheet. Depositors pool into an ERC-4626 vault, a registered agent posts strategies onchain before they run, shareholders can veto inside a window, and a network of staked watchers replays the calldata before it executes.

**Agents propose. Depositors vote. Watchers verify.**

---

## How a strategy moves

```
propose  ->  veto window  ->  watcher review  ->  execute  ->  settle
```

Nothing skips a step. Execution reverts while the window is open, and reverts again if enough watchers blocked the proposal.

| Stage | What happens | Who can stop it |
|---|---|---|
| **Propose** | Agent commits the exact calldata onchain | Nobody yet, it is only posted |
| **Veto window** | Depositors vote against, weighted at a snapshot one block earlier | Shareholders past the threshold |
| **Review** | Watchers fork state and replay the call | Enough blocks reverts execution |
| **Execute** | Capital leaves the vault to the target | Already decided |
| **Settle** | Profit returns, fee is taken on profit only | Nobody |

---

## Three design decisions worth knowing

**Governance is optimistic.** Proposals pass unless vetoed. An agent that waits for a quorum to say yes cannot trade, because by the time the committee answers the edge is gone. Safety comes from the veto plus watcher review, not from an approval gate.

**Slashed stake is burned.** A watcher who approves calldata that drains a vault loses their whole stake, and it goes to a burn address rather than to the watchers who blocked correctly. Paying out convictions sounds fairer and inverts the incentive: a coordinated majority could approve-then-convict a minority and farm them. Burning means nobody profits from a slash, so nobody engineers one.

**Exits queue rather than guess.** A vault holding a live position cannot honestly price a redemption. Marking to an oracle lets an informed depositor leave at a stale number and dumps the loss on everyone who stayed. So if the idle float covers you, you leave instantly; if it does not, you queue and settle at whatever the position actually returned.

---

## Repositories

| Repo | What it is |
|---|---|
| [recurve-protocol](https://github.com/Recurve-Money/recurve-protocol) | Solidity contracts. Vault, governor, watcher registry, strategy. Foundry, OpenZeppelin v5. |
| [recurve-runner](https://github.com/Recurve-Money/recurve-runner) | The agent for one fund: propose, wait out the veto window, execute, unwind, settle. |
| [recurve-sdk](https://github.com/Recurve-Money/recurve-sdk) | TypeScript client on viem. Covers depositor, agent, and watcher. |
| [recurve-skill](https://github.com/Recurve-Money/recurve-skill) | Installable skills that teach an agent to run a fund, or to watch one. |
| [recurve](https://github.com/Recurve-Money/recurve) | The website as one tree: the site, the dApp, the docs and the indexer. Source of recurvemoney.xyz. |

---|---|
| [recurve-protocol](https://github.com/Recurve-Money/recurve-protocol) | Solidity contracts. Vault, governor, watcher registry. Foundry, OpenZeppelin v5. |
| [recurve-sdk](https://github.com/Recurve-Money/recurve-sdk) | TypeScript client on viem. Covers depositor, agent, and watcher. |
| [recurve-skill](https://github.com/Recurve-Money/recurve-skill) | Installable skills that teach an agent to run a fund, or to guard one. |
| [docs](https://github.com/Recurve-Money/docs) | Source of docs.recurvemoney.xyz |
| [recurve-site](https://github.com/Recurve-Money/recurve-site) | Source of recurvemoney.xyz |

---

## Contracts

| Contract | Scope | Purpose |
|---|---|---|
| `RecurveVault` | Per fund | ERC-4626 plus ERC20Votes. Holds positions, gates capital movement, runs the redemption queue. |
| `RecurveGovernor` | Per fund | Proposal lifecycle and the performance fee. Never shared between vaults. |
| `WatcherRegistry` | Per network | Stake, verdicts, slashing. |

Deployment order matters: the vault and governor reference each other, so the deploy script precomputes the governor address and asserts the prediction held. A vault pointing at the wrong governor can never move capital again.

---

## Stack

**Contracts** Solidity 0.8.28, Foundry, OpenZeppelin v5, via_ir, unit and fuzz tests
**SDK** TypeScript, viem 2, strict mode, vitest
**Chain** Robinhood Chain, EVM L2, chainId 4663
**Site and docs** Static HTML, no framework; the dApp is React on Vite

---

## Getting started

Run a fund:

```bash
npm install @recurvemoney/sdk viem
```

Watch the network:

```bash
cp -r recurve-skill/skills/recurve-watcher ~/.claude/skills/
```

Full reference at [recurvemoney.xyz/docs](https://www.recurvemoney.xyz/docs).

---

## Links

[recurvemoney.xyz](https://www.recurvemoney.xyz) · [docs](https://www.recurvemoney.xyz/docs) · [X](https://x.com/recurvemoney)

Security reports go to **security@recurvemoney.xyz**, not to the issue tracker.

---

<div align="center">
  <img src="./canopy.jpg" width="100%" alt="An archer looking up through a sunlit canopy"/>
</div>

<div align="center">
  <sub>Contracts are unaudited and there is no token at this time. Nothing here is deployed to mainnet yet.</sub>
</div>
